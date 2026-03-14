# 05 – Global Game Systems

## Bestiary & Charms

**Description**: Tracks player kill counts per monster type; rewards Charm points for completion.

| Item | Detail |
|------|--------|
| Main C++ files | `src/io/iobestiary.hpp/cpp` |
| Lua API | `src/lua/functions/creatures/monster/charm_functions.hpp` |
| DB tables | `player_charms` |
| Key class | `IOBestiary` |

**Methods**:
- `addBestiaryKill(player, monsterType)` – increment kill counter
- `getKillStatus(player, monsterType)` – get progress stage
- `getBestiaryFinished(player, monsterType)` – check completion
- `parseOffensiveCharmCombat()`, `parseDefensiveCharmCombat()`, `parsePassiveCharmCombat()` – apply charm effects during combat
- `parseCharmCarnage()` – Carnage charm on-death proc
- `addCharmPoints()`, `addMinorCharmEchoes()` – progression

**Charm categories**: offensive, defensive, passive. Charm points unlock/upgrade charms.

---

## Bosstiary (Boss Tracking)

**Description**: Tracks boss encounter history; awards Boss Points and bonuses based on boss rarity.

| Item | Detail |
|------|--------|
| Main C++ files | `src/io/io_bosstiary.hpp/cpp` |
| DB tables | `player_bosstiary`, `boosted_boss` |
| Rarity levels | `BANE`, `ARCHFOE`, `NEMESIS` |

**Features**:
- Boss points calculation per rarity level
- Loot bonus multipliers based on boss tier
- Boosted boss system (daily/weekly boost in `boosted_boss` table)
- Config: `bosstiaryDeadBossTime`, boss-specific points in config

---

## Prey System

**Description**: 3 prey slots where players choose monster types for combat bonuses.

| Item | Detail |
|------|--------|
| Main C++ files | `src/io/ioprey.hpp/cpp` |
| Lua functions | `src/lua/functions/` (confirm exact file) |
| DB tables | `player_prey` |
| Slot enum | `PreySlot_One`, `PreySlot_Two`, `PreySlot_Three` |

**Bonus types**:
- `Damage` – increased damage to prey target
- `Defense` – damage reduction from prey target
- `Experience` – bonus XP from prey kills
- `Loot` – improved loot from prey kills

**Features**:
- Wildcard prey selection
- Auto-reroll on expiry
- Slot lock (premium)
- Linked to Task Hunting system

---

## Task Hunting (Hunting Analyzer)

**Description**: Extended prey system with kill quotas and bonus rewards.

| Item | Detail |
|------|--------|
| DB tables | `player_taskhunt` |
| Config keys | `taskHuntingLimitedTasksExhaust`, `taskHuntingRerollPrice` |
| Linked system | Prey System |

---

## Store System

**Description**: Tibia Coins-based in-game shop.

| Item | Detail |
|------|--------|
| DB tables | `store_history`, `coins_transactions` |
| Coin types | Regular coins, Tournament coins |
| Account fields | `coins`, `tournament_coins`, `coins_transferable` |

---

## Daily Rewards

**Description**: Login streak bonus system with escalating rewards.

| Item | Detail |
|------|--------|
| Lua lib | `data/libs/systems/daily_reward.lua` |
| DB tables | `daily_reward_history`, `player_rewards` |
| Reward storage | `player_rewards` (items in reward chest) |

---

## Imbuements

**Description**: Enchantment system for weapons and armor with time-limited charges.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/imbuements/imbuements.hpp/cpp` |
| XML config | `data/XML/imbuements.xml` |
| Lua API | `src/lua/functions/items/imbuement_functions.hpp` |
| DB | Stored in item attributes (serialized) |

**Features**:
- Categories: aggressive, defensive
- Time-based duration (lost on death by default, configurable)
- Skill/stat modifiers per imbuement type
- Pricing and gold cost managed by imbuement XML

---

## Exaltation Forge (Forge System)

**Description**: Item upgrade system using Forge Dust and Sliver currency.

| Item | Detail |
|------|--------|
| Lua lib | `data/libs/systems/exaltation_forge.lua` |
| DB tables | `forge_history` |
| Player fields | `forge_dusts`, `forge_dust_level` (in `players` table) |
| Max tier | 10 (configurable) |
| Config keys | `forgeCostExaltationDust`, `forgeMaxItemTier`, `forgeBonusRerollPrice` |

**History tracking**: `forge_history` table logs all forge attempts.

---

## Wheel of Destiny

**Description**: Skill tree / progression system using Wheel Points and Gems.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/wheel/player_wheel.hpp/cpp`, `wheel_gems.hpp/cpp`, `wheel_definitions.hpp` |
| I/O file | `src/io/io_wheel.hpp/cpp` |
| Lua API | Player wheel functions in `src/lua/functions/creatures/player/` |
| DB tables | `player_wheeldata` |

**Gem system**:
- Gem affinity, quality, modifiers
- Fragment types for gem crafting
- Promotion scrolls
- Gem locking / unlocking

---

## Charms

See **Bestiary & Charms** section above. Charms are part of the Bestiary system and tracked in `player_charms`.

---

## Achievement System

**Description**: Tracks unlocked achievements with timestamps, secret achievements, and points.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/achievement/player_achievement.hpp/cpp` |
| Storage | KV store (`kv_store` table) |
| Features | Points, grades, secret achievements, unlock timestamp |

---

## Market System

**Description**: Player-to-player trading via buy/sell offers.

| Item | Detail |
|------|--------|
| C++ files | `src/io/iomarket.hpp/cpp` |
| DB tables | `market_offers`, `market_history` |
| Lua API | `ProtocolGame::parseMarketRequest()` + Lua market functions |

**Features**:
- Buy and sell offers with item ID, tier, and price
- Anonymous trading option
- Offer history with transaction log
- Market statistics (highest/lowest prices)

---

## House System

**Description**: Player-owned buildings with access control and rent.

| Item | Detail |
|------|--------|
| C++ files | `src/map/house/house.hpp/cpp`, `src/map/house/housetile.hpp/cpp` |
| Lua API | `src/lua/functions/map/house_functions.hpp` |
| DB tables | `houses`, `house_lists`, `tile_store` |

**Features**:
- House ownership (auction/bid system via `houses.bid`, `bid_end` fields)
- Door access control (`house_lists`)
- Beds (`src/items/bed.hpp`)
- House bank balance (`Bankable` interface)
- Guild houses (confirm)
- Rent system (rent field in `houses` table)

---

## Party System

**Description**: Group play with shared experience and statistics.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/grouping/party.hpp/cpp` |
| Lua API | `src/lua/functions/creatures/player/party_functions.hpp` |

**Features**:
- Shared experience (configurable bonus)
- Party Analyzer (tracks loot, damage, healing per member)
- Leadership transfer
- Status/health sharing

---

## Guild System

**Description**: Player organizations with ranks, bank, and wars.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/grouping/guild.hpp/cpp` |
| I/O file | `src/io/ioguild.hpp/cpp` |
| Lua API | `src/lua/functions/creatures/player/guild_functions.hpp` |
| DB tables | `guilds`, `guild_ranks`, `guild_membership`, `guild_wars`, `guildwar_kills`, `guild_invites` |

**Features**:
- Custom ranks with levels
- Member management
- Guild bank (`Bankable` interface)
- Guild wars (war status, kill tracking)
- Minimum level to form guild (`levelToFormGuild = 8`)
- Premium-only guild creation optional

---

## VIP System

**Description**: Player favorite list with groups (Friends, Enemies, Trading Partner).

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/vip/player_vip.hpp/cpp` |
| DB tables | `account_viplist`, `account_vipgroups`, `account_vipgrouplist` |
| Lua lib | `data/libs/systems/vip.lua` |

**Default groups**:
- `Enemies`
- `Friends`
- `Trading Partner`

---

## Familiar System

**Description**: Companion creatures that follow and assist players.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/grouping/familiars.hpp/cpp` |
| XML | `data/XML/familiars.xml` |
| Lua lib | `data/libs/systems/familiar.lua` |

---

## Hireling System

**Description**: Hired NPC servants for player houses.

| Item | Detail |
|------|--------|
| Lua lib | `data/libs/systems/hireling.lua` |
| DB tables | `player_hirelings` |

---

## Animus Mastery (Soul Pit)

**Description**: Monster-specific mastery progression system.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/animus_mastery/animus_mastery.hpp/cpp` |
| Config key | `animusMasteryMaxMonsterXpMultiplier = 4.0` |
| DB | Player field: `animus_mastery` in `players` table |

---

## Blessing System

**Description**: Optional death-penalty reduction blessings.

| Item | Detail |
|------|--------|
| Lua lib | `data/libs/systems/blessing.lua` |
| DB | `players.blessings` bitmask field |
| Enum file | `src/enums/player_blessings.hpp` |

---

## Cyclopedia (Player Encyclopedia)

**Description**: In-game encyclopedia tracking achievements, titles, and badges.

| Item | Detail |
|------|--------|
| C++ files | `src/creatures/players/cyclopedia/player_cyclopedia.hpp/cpp` |
| Badge | `player_badge.hpp/cpp` |
| Title | `player_title.hpp/cpp` |
| Enum | `src/enums/player_cyclopedia.hpp` |

---

## Team Finder

**Description**: UI feature to find parties for specific content.

| Item | Detail |
|------|--------|
| C++ file | `src/creatures/players/grouping/team_finder.hpp` |

---

## Zone System

**Description**: Named map zones with configurable PvP rules and properties.

| Item | Detail |
|------|--------|
| C++ files | `src/game/zones/zone.hpp/cpp` |
| Lua lib | `data/libs/systems/zones.lua` |
| Lua API | `src/lua/functions/core/game/zone_functions.hpp` |
