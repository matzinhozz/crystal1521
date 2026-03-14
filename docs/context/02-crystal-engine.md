# 02 – Crystal Engine Architecture

## Main Server Loop

**Entry**: `src/main.cpp` → `CrystalServer::init()` → `Game::start()`

```
CrystalServer::init()
  ├── ConfigManager::load("config.lua")
  ├── Logger init (spdlog)
  ├── DatabaseManager::connect()
  ├── DatabaseManager::updateDatabase()  ← runs migrations
  ├── IOMap::loadMap()                   ← loads .otbm map file
  ├── Items::loadFromXml()               ← loads items.xml
  ├── Monsters::loadFromXml()            ← loads monster definitions
  ├── Npcs::loadFromXml()                ← loads NPC definitions
  ├── LuaEnvironment::init()             ← registers all C++ bindings
  ├── Scripts::loadScripts("data/")      ← loads core Lua scripts
  ├── Scripts::loadScripts("data-crystal/") ← loads content scripts
  ├── Server::bind(7171, ProtocolLogin)
  ├── Server::bind(7172, ProtocolGame)
  ├── Server::bind(7171, ProtocolStatus)
  └── Dispatcher::start() / Scheduler::start()
```

## Scheduler & Dispatcher

**Files**: `src/game/scheduling/dispatcher.hpp`, `scheduler.hpp`, `task.hpp`

- **Scheduler** min tick: 50 ms. Queues events with timestamps.
- **Dispatcher** task groups:
  - `ThreadPool` – async background work
  - `Walk` – pathfinding tasks
  - `WalkParallel` – parallel walk tasks
  - `Serial` – sequential, ordered tasks
  - `GenericParallel` – general parallel tasks
- Task expiration: 2000 ms (dropped if too old)
- `SaveManager` – periodic player saves (separate goroutine-equivalent)

## Key Constants

```cpp
// src/game/game.hpp
static constexpr int32_t SERVER_BEAT = 0x32;       // 50 ms
static constexpr int32_t EVENT_MS = 10000;          // 10 s tick
static constexpr int32_t EVENT_DECAYINTERVAL = 250; // ms
static constexpr int32_t EVENT_DECAY_BUCKETS = 4;
static constexpr int32_t CACHE_EXPIRATION_TIME = 10 * 60 * 1000; // 10 min
```

## Game Singleton

**File**: `src/game/game.hpp` / `game.cpp`

The `Game` class is the central controller:

- `Game::getInstance()` – global access
- `loadMainMap()`, `loadCustomMaps()`, `loadMap()`
- `getMapDimensions()`, `setWorldType()`, `getWorldType()`
- `loadBoostedCreature()`, `resetMonsters()`, `resetNpcs()`
- `start()`, `forceRemoveCondition()`
- Holds references to: Map, Creatures list, SpawnManager, event queues

## Creature Hierarchy

```
Thing  (src/items/thing.hpp)
├── Creature  (src/creatures/creature.hpp)
│   ├── Player  (src/creatures/players/player.hpp)
│   ├── Monster (src/creatures/monsters/monster.hpp)
│   └── Npc     (src/creatures/npcs/npc.hpp)
└── Item    (src/items/item.hpp)
    ├── Container (src/items/containers/container.hpp)
    │   ├── DepotChest / DepotLocker
    │   ├── Inbox
    │   ├── Mailbox
    │   └── RewardChest
    ├── BedItem
    ├── Teleport
    └── Weapon
```

## Player Class

**File**: `src/creatures/players/player.hpp`

The `Player` class aggregates all player subsystems:

| Component | File |
|-----------|------|
| `PlayerWheel` | `creatures/players/wheel/player_wheel.hpp` |
| `PlayerAchievement` | `creatures/players/achievement/player_achievement.hpp` |
| `AnimusMastery` | `creatures/players/animus_mastery/animus_mastery.hpp` |
| `Imbuements` | `creatures/players/imbuements/imbuements.hpp` |
| `Party` | `creatures/players/grouping/party.hpp` |
| `Guild` | `creatures/players/grouping/guild.hpp` |
| `PlayerVIP` | `creatures/players/vip/player_vip.hpp` |
| `PlayerBadge` | `creatures/players/cyclopedia/player_badge.hpp` |
| `PlayerTitle` | `creatures/players/cyclopedia/player_title.hpp` |
| `Bank` | `game/bank/bank.hpp` |

- Player also implements `Cylinder` (can contain items) and `Bankable` (can hold bank balance).

## Map Loading

**Files**: `src/map/map.hpp`, `src/io/iomap.hpp`

- Map inherits `MapCache` for tile caching.
- Map files are `.otbm` format, loaded from `data-crystal/world/`.
- `MapSector` divides map into sectors for performance (`src/map/utils/mapsector.hpp`).
- `AStarNodes` provides A* pathfinding (`src/map/utils/astarnodes.hpp`).
- `Spectators` manages creature visibility lists per tile.

## Combat System

**Files**: `src/creatures/combat/combat.hpp`, `condition.hpp`, `spells.hpp`

- `Combat` class handles all damage/heal calculations.
- Callback types:
  - `ValueCallback` – damage values
  - `TileCallback` – area effects on tiles
  - `TargetCallback` – single-target effects
  - `ChainCallback` – chain attack (multi-target)
- `Condition` – status effects (poison, burn, bleed, curse, etc.)
- `Spells` singleton manages `InstantSpell` and `RuneSpell`.

## Item System

**Files**: `src/items/item.hpp`, `items.hpp`

- `Items` singleton loads from `items.xml`.
- Each item has an `ItemType` with attribute map.
- `ItemAttribute` / `CustomAttribute` store per-instance data.
- `Decay` system degrades items over time (`src/items/decay/decay.hpp`).
- Weapons have specialized behavior in `src/items/weapons/weapons.hpp`.

## Event System

**Files**: `src/lua/creature/creatureevent.hpp`, `src/lua/global/globalevent.hpp`, `src/lua/callbacks/event_callback.hpp`

Three parallel event systems:

1. **CreatureEvent** – attached to creatures, triggered by engine hooks
2. **GlobalEvent** – server-wide scheduled or lifecycle events
3. **EventCallback** – flexible, type-parameterized callback registration

CreatureEvent types:
- `onLogin`, `onLogout`, `onThink`, `onPrepareDeath`, `onDeath`
- `onKill`, `onAdvance`, `onModalWindow`, `onTextEdit`
- `onHealthChange`, `onManaChange`, `onExtendedOpcode`

## Server Initialization Flow (Summary)

```
1. main()
2.   CrystalServer::init()
3.     ConfigManager loads config.lua
4.     DB connects → migrations run
5.     Items, Monsters, NPCs loaded from XML
6.     Lua state initialized; all C++ functions registered
7.     data/ scripts loaded (core systems)
8.     data-crystal/ scripts loaded (game content)
9.     Ports bound: 7171 (login), 7172 (game)
10.    Scheduler + Dispatcher start
11.    Server main loop runs (ASIO io_context)
```

## Login & Game Session Flow

```
Client connects to :7171
  ProtocolLogin::onRecvFirstMessage()
    → RSA decrypt key exchange
    → Verify account credentials (Argon2 hash check)
    → Send character list
    → Disconnect

Client connects to :7172
  ProtocolGame::onRecvFirstMessage()
    → XTEA key exchange
    → IOLoginData::loadPlayer()  ← DB read
    → Game::addCreature(player)
    → Send initial map + player data
    → Enter normal packet loop
```

## Key C++ Classes Quick Reference

| Class | File | Purpose |
|-------|------|---------|
| `Game` | `src/game/game.hpp` | Main engine singleton |
| `Dispatcher` | `src/game/scheduling/dispatcher.hpp` | Task queue manager |
| `Creature` | `src/creatures/creature.hpp` | Base entity |
| `Player` | `src/creatures/players/player.hpp` | Player entity |
| `Monster` | `src/creatures/monsters/monster.hpp` | Monster entity |
| `Npc` | `src/creatures/npcs/npc.hpp` | NPC entity |
| `Item` | `src/items/item.hpp` | Item entity |
| `Map` | `src/map/map.hpp` | Map container |
| `Tile` | `src/items/tile.hpp` | Single map tile |
| `Combat` | `src/creatures/combat/combat.hpp` | Combat engine |
| `Spells` | `src/creatures/combat/spells.hpp` | Spell manager |
| `IOLoginData` | `src/io/iologindata.hpp` | Player persistence |
| `ProtocolGame` | `src/server/network/protocol/protocolgame.hpp` | Game protocol |
| `LuaScriptInterface` | `src/lua/scripts/luascript.hpp` | Lua bridge |
| `ConfigManager` | `src/config/configmanager.hpp` | Config reader |
| `Database` | `src/database/database.hpp` | MySQL layer |
