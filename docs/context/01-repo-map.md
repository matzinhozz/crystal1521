# 01 – Repository Map

## Root Structure

```
crystalserver/
├── src/                    # C++ engine source
├── data/                   # Core engine scripts and configs
├── data-crystal/           # Crystal-specific game content
├── data-global/            # Alternative datapack (configured via dataPackDirectory)
├── docs/                   # Documentation (Doxyfile.in, python-scripts)
├── cmake/                  # CMake modules and toolchains
├── docker/                 # Docker Compose and Dockerfile
├── metrics/                # Prometheus config
├── tests/                  # C++ test suite
├── vcproj/                 # Visual Studio project files
├── CMakeLists.txt          # Top-level build config
├── CMakePresets.json       # Build presets (linux-release, windows-release, etc.)
├── config.lua.dist         # Configuration template (709 lines)
├── schema.sql              # Full MySQL schema (914 lines)
├── vcpkg.json              # Dependency manifest
├── GitVersion.yml          # Semantic versioning config
└── Jenkinsfile             # CI pipeline
```

## C++ Source Tree (`src/`)

```
src/
├── main.cpp                        # Entry point
├── crystalserver.hpp/cpp           # Server bootstrap
├── pch.hpp/cpp                     # Precompiled headers
├── core.hpp / declarations.hpp     # Global includes and forward declarations
│
├── game/                           # Game engine core
│   ├── game.hpp/cpp                # Game singleton (main loop)
│   ├── game_definitions.hpp
│   ├── bank/                       # Banking interface
│   ├── scheduling/                 # Dispatcher, Scheduler, SaveManager
│   ├── movement/                   # Position, Teleport
│   ├── zones/                      # Zone system (PvP, safe zones)
│   ├── functions/                  # game_reload
│   └── modal_window/
│
├── creatures/                      # Entity system
│   ├── creature.hpp/cpp            # Base Creature class
│   ├── creatures_definitions.hpp
│   ├── players/                    # Player entity + subsystems
│   │   ├── player.hpp/cpp
│   │   ├── achievement/
│   │   ├── animus_mastery/
│   │   ├── attached_effects/
│   │   ├── cyclopedia/             # Badge, Title, Cyclopedia entries
│   │   ├── grouping/               # Guild, Party, Familiars, TeamFinder
│   │   ├── imbuements/
│   │   ├── management/             # Ban, Waitlist
│   │   ├── proficiencies/
│   │   ├── storages/
│   │   ├── vip/
│   │   ├── vocations/
│   │   └── wheel/                  # Wheel of Destiny + Gems
│   ├── monsters/                   # Monster entity + spawning
│   ├── npcs/                       # NPC entity + spawning
│   ├── combat/                     # Combat, Conditions, Spells
│   ├── appearance/                 # Mounts, Outfits, AttachedEffects
│   └── interactions/               # Chat system
│
├── items/                          # Item system
│   ├── item.hpp/cpp                # Base Item class
│   ├── items.hpp/cpp               # Items manager
│   ├── thing.hpp/cpp               # Base Thing (parent of Item/Creature)
│   ├── tile.hpp/cpp                # Map tile
│   ├── cylinder.hpp/cpp            # Container base
│   ├── containers/                 # Container, Depot, Inbox, Mailbox, Rewards
│   ├── weapons/
│   ├── decay/
│   └── functions/item/             # Attribute, CustomAttribute, ItemParse
│
├── map/                            # Map system
│   ├── map.hpp/cpp                 # Map container (inherits MapCache)
│   ├── mapcache.hpp/cpp
│   ├── spectators.hpp/cpp
│   ├── town.hpp/cpp
│   ├── house/                      # House, HouseTile
│   └── utils/                      # MapSector, AStarNodes
│
├── server/                         # Network layer
│   ├── server.hpp/cpp              # Main server (port binding)
│   ├── signals.hpp/cpp
│   └── network/
│       ├── connection/             # TCP connection wrapper
│       ├── message/                # NetworkMessage, OutputMessage
│       ├── protocol/               # Protocol, ProtocolGame, ProtocolLogin, ProtocolStatus
│       └── webhook/                # HTTP webhook
│
├── io/                             # Data I/O layer
│   ├── iologindata.hpp/cpp         # Player load/save
│   ├── iomap.hpp/cpp               # Map loading (.otbm)
│   ├── iobestiary.hpp/cpp          # Bestiary + Charm system
│   ├── io_bosstiary.hpp/cpp        # Bosstiary
│   ├── ioprey.hpp/cpp              # Prey system
│   ├── iomarket.hpp/cpp            # Market
│   ├── ioguild.hpp/cpp             # Guild data
│   ├── io_wheel.hpp/cpp            # Wheel of Destiny data
│   └── functions/                  # iologindata_load/save_player
│
├── database/                       # DB layer
│   ├── database.hpp/cpp            # MySQL connection (max 8 MB queries)
│   ├── databasemanager.hpp/cpp
│   └── databasetasks.hpp/cpp       # Async DB tasks
│
├── lua/                            # Lua scripting system
│   ├── scripts/                    # LuaScriptInterface, LuaEnvironment
│   ├── global/                     # BaseEvents, GlobalEvent, LuaVariant
│   ├── callbacks/                  # CreatureCallback, EventCallback
│   ├── creature/                   # CreatureEvent, Actions, TalkAction, Raids
│   ├── modules/                    # Module loader
│   └── functions/                  # All Lua↔C++ bindings (60+ files)
│
├── config/                         # ConfigManager, config_enums
├── account/                        # Account entity + repository (DB-backed)
├── security/                       # RSA, Argon2
├── kv/                             # Key-value store (SQL-backed)
├── enums/                          # All enum definitions
├── lib/                            # DI container, logging, messaging, metrics, threading
├── utils/                          # Tools, hash, SIMD, wildcard tree, etc.
└── protobuf/                       # Protobuf definitions
```

## Lua Scripts (`data/scripts/`)

```
data/scripts/
├── actions/            # Item use handlers
├── creaturescripts/    # Creature event handlers (onDeath, onLogin, etc.)
├── eventcallbacks/     # EventCallback registrations
├── globalevents/       # Server-wide scheduled events
├── lib/                # Shared Lua libraries
├── movements/          # Step-on / step-off handlers
├── runes/              # Rune spell implementations
├── spells/             # Spell scripts (attack/, healing/, support/, etc.)
├── systems/            # Game system implementations
├── talkactions/        # Chat command handlers
└── weapons/            # Weapon-specific combat scripts
```

## Lua Libraries (`data/libs/`)

```
data/libs/
├── core/               # global_storage.lua
├── functions/          # combat.lua, creature.lua, player.lua, item.lua, game.lua, functions.lua
├── systems/            # blessing.lua, daily_reward.lua, exaltation_forge.lua,
│                       # hireling.lua, raids.lua, vip.lua, zones.lua, concoctions.lua
└── tables/             # Game constant tables
```

## Game Content (`data-crystal/`)

```
data-crystal/
├── lib/                # quests.lua, storages.lua, town.lua
├── monster/            # Monster XML definitions (~25 categories)
│   ├── bosses/
│   ├── demons/
│   ├── dragons/
│   ├── humanoids/
│   ├── undeads/
│   └── [20+ more]
├── npc/                # NPC XML definitions
├── raids/              # Raid XML (ferumbras/, the_old_widow/)
├── scripts/            # Crystal-specific Lua scripts
│   ├── actions/
│   ├── creaturescripts/
│   ├── quests/
│   └── spells/
├── startup/            # Startup tables and config
└── world/              # Map files (.otbm)
```

## Configuration Files

| File | Purpose |
|------|---------|
| `config.lua.dist` | Server configuration template |
| `data/XML/events.xml` | Event type registrations |
| `data/XML/groups.xml` | Account group definitions |
| `data/XML/vocations.xml` | Vocation stat definitions |
| `data/XML/imbuements.xml` | Imbuement definitions |
| `data/XML/mounts.xml` | Mount definitions |
| `data/XML/outfits.xml` | Outfit definitions |
| `data/XML/familiars.xml` | Familiar definitions |
| `data/items/items.xml` | Item type definitions |
| `data/items/proficiencies.json` | Proficiency data |
| `data/migrations/` | Numbered DB migration scripts (0–23+) |

## Database Schema

- **File**: `schema.sql` (914 lines)
- **51 tables** – see `08-database-model.md` for full breakdown
