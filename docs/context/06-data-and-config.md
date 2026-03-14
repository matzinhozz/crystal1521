# 06 – Data & Configuration

## Main Configuration File

**Template**: `config.lua.dist` (709 lines) → copy to `config.lua` to use.

Parsed by `ConfigManager` (`src/config/configmanager.hpp`). Access in C++ via `g_config.getString(ConfigKey)` or `g_config.getNumber(ConfigKey)`. In Lua: `ConfigManager.getString(key)`.

## Core Settings

```lua
-- Directories
dataPackDirectory = "data-global"   -- or "data-crystal"
coreDirectory = "data"

-- Logging
logLevel = "info"   -- debug | trace | info | warning | error | critical

-- Server identity
serverName = "Crystal"
ip = "127.0.0.1"
```

## Network Settings

```lua
loginProtocolPort = 7171
gameProtocolPort  = 7172
statusProtocolPort = 7171
maxPlayers = 0              -- 0 = unlimited
maxPacketsPerSecond = 25    -- rate limit per connection
packetCompressionLevel = 6  -- 0 = off, 1-9 = zlib level
```

## World / Combat Settings

```lua
worldType = "pvp"           -- pvp | no-pvp | pvp-enforced
protectionLevel = 7         -- no PvP below this level
pzLocked = 60 * 1000        -- ms locked out of PZ after PvP
xpFromPlayersLevelRange = 75  -- level % range for PvP XP
```

## Player Economy

```lua
freeDepotLimit = 2000
premiumDepotLimit = 10000
depotBoxes = 20
```

## Prey System Config

```lua
-- Reroll prices (coins or gold, confirm unit)
preyRerollPriceCoins = true
preyDataBuyUpgradeOneSlot  = 1500
preyDataBuyUpgradeTwoSlot  = 5000
preyDataBuyUpgradeThreeSlot = 10000
preyBonusTime = 2 * 60 * 60  -- 2 hours bonus duration
preyTaxFree = false
```

## Task Hunting Config

```lua
taskHuntingLimitedTasksExhaust = ...
taskHuntingRerollPrice = ...
```

## Forge System Config

```lua
forgeMaxItemTier = 10
forgeCostExaltationDust = ...
forgeBonusRerollPrice = ...
-- (full list in config.lua.dist lines 400+)
```

## Guild Settings

```lua
levelToFormGuild = 8
guildWarsMinimunFrags = 10
createGuildOnlyPremium = true
```

## Animus Mastery

```lua
animusMasteryMaxMonsterXpMultiplier = 4.0
augmentIncreasedDamagePercent = 5
```

## XML Data Files

| File | Purpose |
|------|---------|
| `data/XML/events.xml` | Register event types with the engine |
| `data/XML/groups.xml` | Account group definitions (access levels) |
| `data/XML/vocations.xml` | Vocation stat tables (speed, skills, gain rates) |
| `data/XML/imbuements.xml` | All imbuement definitions |
| `data/XML/mounts.xml` | Mount definitions with IDs and names |
| `data/XML/outfits.xml` | Outfit definitions |
| `data/XML/familiars.xml` | Familiar creature definitions |
| `data/XML/attachedeffects.xml` | Visual effect definitions |
| `data/XML/storages.xml` | Storage key name mappings |
| `data/XML/specialtiles.xml` | Special tile properties |
| `data/items/items.xml` | Item type definitions (IDs, attributes, weights) |
| `data/items/bags.xml` | Bag item definitions |
| `data/items/proficiencies.json` | Weapon proficiency data |

## Monster/NPC Data

- **Monsters**: `data-crystal/monster/` – XML files organized by category (~25 folders)
- **NPCs**: `data-crystal/npc/` – XML files per NPC

Each monster XML contains: health, speed, loot table, combat spells, race, bestiary info.

## Map Files

- **Format**: `.otbm` (Open Tibia Binary Map)
- **Location**: `data-crystal/world/`
- **Loaded by**: `IOMap::loadMap()` → `src/io/iomap.hpp`

## Database Migrations

- **Location**: `data/migrations/` – numbered Lua migration scripts (0–23+)
- **Applied by**: `DatabaseManager::updateDatabase()` at startup
- Each migration file is numbered and applied once (tracked in `server_config` table)

## Key-Value Store

- **Purpose**: Flexible persistent storage for game systems (achievements, custom data)
- **C++ files**: `src/kv/kv.hpp`, `kv_sql.hpp`
- **DB table**: `kv_store`
- **Lua API**: `KV.get(key)`, `KV.set(key, value)`

## Global Storage

- **Purpose**: Server-wide Lua variables persisted between restarts
- **DB table**: `global_storage`
- **Lua lib**: `data/libs/core/global_storage.lua`

## Constants

**C++ constants** in `src/utils/const.hpp` and `src/game/game_definitions.hpp`:
- Map size, creature limits, item limits
- Protocol-level constants

**Lua constants** in `data/libs/tables/` and `data/global.lua`:
- Storage key IDs, item IDs, spell names, creature names

## Important Gameplay-Affecting Parameters

| Parameter | Impact |
|-----------|--------|
| `worldType` | Enables/disables PvP and skull system |
| `protectionLevel` | Level below which PvP is blocked |
| `pzLocked` | Duration players are locked from safe zones after PvP |
| `maxPlayers` | Server capacity |
| `forgeMaxItemTier` | Max forge upgrade level |
| `animusMasteryMaxMonsterXpMultiplier` | XP bonus cap for mastered monsters |
| `dataPackDirectory` | Which content pack is loaded (`data-crystal` vs `data-global`) |
| `freeDepotLimit` / `premiumDepotLimit` | Depot item cap affects progression |
| `preyBonusTime` | Duration of prey bonuses |
| `guildWarsMinimunFrags` | War entry threshold |
