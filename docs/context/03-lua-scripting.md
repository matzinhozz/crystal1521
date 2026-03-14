# 03 – Lua Scripting System

## Overview

The server uses **LuaJIT** as its scripting engine. Nearly all gameplay logic (spells, quests, NPCs, events, items) is implemented in Lua. The C++ engine exposes a rich API through registered global functions and metatables.

## Script Directory Structure

```
data/
├── core.lua                    # Top-level loader: loads libs/ then scripts/
├── global.lua                  # Global definitions, constants
├── libs/
│   ├── core/
│   │   └── global_storage.lua  # Global KV storage helpers
│   ├── functions/
│   │   ├── combat.lua          # Combat utility functions
│   │   ├── creature.lua        # Creature utility functions
│   │   ├── player.lua          # Player utility functions
│   │   ├── item.lua            # Item utility functions
│   │   ├── game.lua            # Game utility functions
│   │   └── functions.lua       # General helpers
│   ├── systems/
│   │   ├── blessing.lua        # Blessing system
│   │   ├── concoctions.lua     # Concoction system
│   │   ├── daily_reward.lua    # Daily reward system
│   │   ├── exaltation_forge.lua # Forge system
│   │   ├── familiar.lua        # Familiar system
│   │   ├── hireling.lua        # Hireling system
│   │   ├── raids.lua           # Raid helpers
│   │   ├── vip.lua             # VIP system
│   │   └── zones.lua           # Zone system
│   └── tables/                 # Game constant tables
└── scripts/
    ├── actions/                # Item use handlers
    ├── creaturescripts/        # Creature event handlers
    ├── eventcallbacks/         # EventCallback registrations
    ├── globalevents/           # Scheduled/startup global events
    ├── lib/                    # Script-local libraries
    ├── movements/              # Step-on / step-off handlers
    ├── runes/                  # Rune spell logic
    ├── spells/                 # Spell implementations
    │   └── attack/             # Attack spells (30+ files)
    ├── systems/                # Complex game systems in Lua
    ├── talkactions/            # Chat command handlers
    └── weapons/                # Weapon behavior scripts

data-crystal/scripts/           # Crystal-specific overrides and content
    ├── actions/
    ├── creaturescripts/
    ├── quests/
    └── spells/
```

## Event Types

### Actions (`data/scripts/actions/`)

- Triggered when a player **uses an item** (right-click use).
- Registered via `Action` object in Lua.
- Key entry: `action:onUse(player, item, fromPosition, target, toPosition, isHotkey)`

### Movements (`data/scripts/movements/`)

- Triggered when a creature **steps on / off a tile**.
- Registered via `MoveEvent` object.
- Callbacks: `onStepIn`, `onStepOut`, `onAddItem`, `onRemoveItem`

### TalkActions (`data/scripts/talkactions/`)

- Triggered when a player **types a chat command** (e.g., `!commands`).
- Registered via `TalkAction` object.
- Entry: `talkaction:onSay(player, words, param, channel)`

### Spells (`data/scripts/spells/`)

- Registered via `Spell` (InstantSpell or RuneSpell).
- Entry: `spell:onCastSpell(creature, variant)`
- Attack spells examples: `annihilation.lua`, `great_death_beam.lua`, `eternal_winter.lua`, `brutal_strike.lua`, `double_jab.lua`

### CreatureScripts (`data/scripts/creaturescripts/`)

- Attached to creatures via `registerEvent`.
- Available hooks:
  - `onLogin(player)`
  - `onLogout(player)`
  - `onThink(creature, interval)`
  - `onDeath(creature, corpse, killer, mostDamageKiller, unjustified, mostDamageUnjustified)`
  - `onPrepareDeath(creature, killer)`
  - `onKill(creature, target)`
  - `onAdvance(player, skill, oldLevel, newLevel)`
  - `onHealthChange(creature, attacker, primaryDamage, primaryType, secondaryDamage, secondaryType, origin)`
  - `onManaChange(creature, attacker, manaChange, origin)`
  - `onExtendedOpcode(player, opcode, buffer)`

### GlobalEvents (`data/scripts/globalevents/`)

- Server lifecycle and scheduled events.
- Types: `startup`, `shutdown`, `record`, `timer` (cron-like)
- Entry: `globalEvent:onTime(interval)` or `globalEvent:onStartup()`

### EventCallbacks (`data/scripts/eventcallbacks/`)

- More granular callback system layered on top of CreatureEvents.
- **File**: `src/lua/callbacks/event_callback.hpp`
- Allows multiple callbacks per event type.

### NPC Scripts (`data-crystal/npc/`)

- NPC definitions in XML reference Lua behavior scripts.
- NPC interactions handled via shop system and dialog.
- Key C++ file: `src/creatures/npcs/npc.hpp`
- Lua API: `npc:*`, `npcType:*`, `shop:*`

## How Lua Hooks into C++

The bridge is `LuaScriptInterface` (`src/lua/scripts/luascript.hpp`):

```cpp
class LuaScriptInterface : public Lua
  - initState() / reInitState()  // create Lua state
  - loadFile(path)               // load .lua file
  - getEvent(name)               // get event function ref
  - pushFunction() / callFunction() / callVoidFunction()
```

### C++ → Lua Call Pattern

```cpp
// 1. Push function ref
LuaScriptInterface::pushFunction(eventRef);
// 2. Push arguments
lua_push*(L, arg);
// 3. Call
LuaScriptInterface::callFunction(argCount);
// 4. Read result
lua_to*(L, -1);
```

### Lua → C++ Binding Pattern

All C++ functions are registered in `src/lua/functions/` via `luaL_Reg` arrays and `lua_register()` / `Lua::registerClass()`.

Key binding directories:
```
src/lua/functions/
├── core/game/          # Game, Config, GlobalFunctions, Bank, ModalWindow, Zone
├── core/libs/          # DB, KV, Logger, Metrics, Result
├── core/network/       # NetworkMessage, Webhook
├── creatures/combat/   # Combat, Condition, Spell, Variant
├── creatures/monster/  # Monster, MonsterType, Charm, Loot
├── creatures/npc/      # Npc, NpcType, Shop
├── creatures/player/   # Player, Guild, Party, Group, Vocation, Mount
├── events/             # Action, CreatureEvent, EventCallback, GlobalEvent,
│                         MoveEvent, TalkAction, EventsScheduler
├── items/              # Item, ItemType, Container, Weapon, Imbuement
└── map/                # Position, Tile, House, Teleport, Town, Map
```

## Important Entry Points

| System | Entry File |
|--------|-----------|
| Server startup | `data/core.lua` |
| Global constants | `data/global.lua` |
| Player login | `data/scripts/creaturescripts/` → `onLogin` |
| Item use | `data/scripts/actions/` → `onUse` |
| Spells | `data/scripts/spells/` → `onCastSpell` |
| Chat commands | `data/scripts/talkactions/` → `onSay` |
| Daily events | `data/scripts/globalevents/` → `onTime` |
| Quest logic | `data-crystal/scripts/quests/` |
| Forge system | `data/libs/systems/exaltation_forge.lua` |
| Daily reward | `data/libs/systems/daily_reward.lua` |
| VIP system | `data/libs/systems/vip.lua` |

## Lua State Lifecycle

1. `LuaEnvironment::init()` creates global Lua state at server start.
2. All C++ bindings registered once.
3. Scripts loaded top-to-bottom: `data/` → `data-crystal/`.
4. `Game::reloadScripts()` can hot-reload scripts at runtime (via talkaction `/reload`).
5. Each script file runs in the shared global state (no sandboxing per script).
