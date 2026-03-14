# 08 – Database Model

**File**: `schema.sql` (914 lines) | **Engine**: MySQL / MariaDB

Total tables: **51**

---

## Account & Auth Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `accounts` | `id`, `name`, `password`, `email`, `premdays`, `coins`, `tournament_coins`, `coins_transferable`, `house_bid_id` | Player accounts, VIP, coins |
| `account_sessions` | `id`, `account_id`, `expires` | Session tokens |
| `account_bans` | `account_id`, `reason`, `expires_at`, `active` | Active bans |
| `account_ban_history` | `account_id`, `reason`, `removed_at` | Historical bans |
| `account_viplist` | `account_id`, `player_id` | VIP player list |
| `account_vipgroups` | `id`, `account_id`, `name` | VIP groups (Friends/Enemies/Trading) |
| `account_vipgrouplist` | `account_id`, `player_id`, `groupid` | VIP group membership |
| `coins_transactions` | `account_id`, `type`, `coins`, `description` | Coin purchase/spend log |
| `ip_bans` | `ip`, `reason`, `expires_at` | IP-level bans |

---

## Player Core Tables

### `players` (critical table – 155+ columns)

Core character data:

| Column Group | Columns |
|-------------|---------|
| Identity | `name`, `account_id`, `vocation`, `sex`, `pronoun` |
| Stats | `level`, `experience`, `health`, `healthmax`, `mana`, `manamax`, `soul` |
| Position | `posx`, `posy`, `posz`, `town_id`, `lookdir` |
| Skills | `skill_fist`..`skill_fishing` (8 skills), `skill_*_tries`, `skill_*_percent` |
| Leech | `manashield`, `manavampirism_percent`, `life_leeching` |
| Critical | `critical_hit_chance`, `critical_hit_percent` |
| Systems | `stamina`, `blessings`, `skull`, `skull_time` |
| Social | `marriage_status`, `marriage_spouse` |
| Modern | `forge_dusts`, `forge_dust_level`, `boss_points`, `loyalty_points` |
| New | `animus_mastery`, `weapon_proficiencies`, `harmony`, `virtue` |
| Timestamps | `lastlogin`, `lastlogout`, `deletion` |
| Misc | `balance`, `offlinetraining_skill`, `offlinetraining_time`, `conditions` (BLOB) |

### Related Player Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `player_items` | `player_id`, `pid`, `sid`, `itemtype`, `count`, `attributes` | Inventory items (serialized) |
| `player_depotitems` | `player_id`, `pid`, `sid`, `itemtype`, `count`, `attributes` | Depot items |
| `player_inboxitems` | `player_id`, `pid`, `sid`, `itemtype`, `count`, `attributes` | Inbox items |
| `player_rewards` | `player_id`, `pid`, `sid`, `itemtype`, `count`, `attributes` | Reward chest items |
| `player_stash` | `player_id`, `item_id`, `item_count` | Stash system items |
| `player_outfits` | `player_id`, `outfit_id`, `addons` | Unlocked outfits |
| `player_mounts` | `player_id`, `mount_id` | Unlocked mounts |
| `player_spells` | `player_id`, `name` | Learned spells |
| `player_storage` | `player_id`, `key`, `value` | Lua storage values (quest flags, etc.) |
| `player_deaths` | `player_id`, `time`, `level`, `killed_by`, `is_player` | Death records |
| `player_kills` | `player_id`, `target`, `time`, `unavenged` | PvP kill records |
| `player_namelocks` | `player_id`, `reason`, `namelocked_at` | Namelock records |
| `player_oldnames` | `player_id`, `name`, `date` | Name change history |
| `player_statements` | `player_id`, `channel_id`, `date`, `statement` | Chat log |

---

## Gameplay System Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `player_charms` | `player_guid`, `charm_points`, `charms` (JSON/BLOB), `used_charm_budget` | Bestiary charm system |
| `player_bosstiary` | `player_id`, `bossid`, `kill_count`, `last_kill`, `boss_points` | Boss encounter tracking |
| `player_prey` | `player_id`, `slot`, `state`, `raceid`, `option`, `bonus_type`, `bonus_value`, `bonus_grade`, `task_points`, `reroll_price`, `lock` | Prey slots |
| `player_taskhunt` | `player_id`, `slot`, `state`, `raceid`, `currentkills`, `goalkills`, `finished` | Task hunting |
| `player_wheeldata` | `player_id`, `slot`, `data` | Wheel of Destiny data |
| `player_hirelings` | `player_id`, `name`, `sex`, `type`, `equipment`, `pos` | Hired servants |
| `daily_reward_history` | `player_id`, `rewardDay`, `daystreak`, `timestamp` | Daily login streak |
| `forge_history` | `player_id`, `action_type`, `description`, `is_success`, `cost`, `gained_tier`, `reduced_tier`, `increased_tier`, `date` | Forge operation log |
| `store_history` | `account_id`, `product_type`, `description`, `coins`, `coins_transferable`, `timestamp`, `category` | Store purchase history |

---

## Guild Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `guilds` | `id`, `name`, `ownerid`, `creationdata`, `motd`, `description`, `logo_gfx_name`, `balance` | Guild data + bank balance |
| `guild_ranks` | `id`, `guild_id`, `name`, `level` | Guild rank definitions |
| `guild_membership` | `player_id`, `guild_id`, `rank_id`, `nick` | Member ↔ guild mapping |
| `guild_wars` | `id`, `guild1`, `guild2`, `name1`, `name2`, `status`, `started`, `ended` | Active and past wars |
| `guildwar_kills` | `id`, `killer`, `target`, `killerguild`, `targetguild`, `time` | War kill records |
| `guild_invites` | `player_id`, `guild_id` | Pending invitations |

---

## Market Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `market_offers` | `id`, `player_id`, `sale`, `itemtype`, `amount`, `price`, `created`, `anonymous`, `tier` | Active buy/sell offers |
| `market_history` | `id`, `player_id`, `sale`, `itemtype`, `amount`, `price`, `expires_at`, `inserted`, `state`, `tier` | Transaction log |

---

## Map / World Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `houses` | `id`, `owner`, `paid`, `warnings`, `name`, `town_id`, `rent`, `size`, `guildid`, `bid`, `bid_end`, `last_bid`, `highest_bidder` | House ownership + auction |
| `house_lists` | `house_id`, `listid`, `list` | Door access control lists |
| `tile_store` | `house_id`, `data` | Serialized house tile contents |
| `towns` | `town_id`, `town_name`, `posx`, `posy`, `posz` | Town definitions |
| `boosted_boss` | `id`, `date`, `raceid`, `name`, `kills`, `lootQ` | Current boosted boss |
| `boosted_creature` | `id`, `date`, `raceid`, `name` | Current boosted creature |

---

## Server / Meta Tables

| Table | Key Columns | Purpose |
|-------|-------------|---------|
| `server_config` | `config`, `value` | Server key-value config (version tracking) |
| `global_storage` | `key`, `value` | Global Lua storage (persisted variables) |
| `kv_store` | `key`, `value` | Generic KV store for game systems |
| `players_online` | `player_id` | **MEMORY table** – tracks online players |

---

## Critical Tables for Gameplay

| Table | Why Critical |
|-------|-------------|
| `players` | All character state – loss = complete data corruption |
| `player_items` | Player inventory – wrong serialization = item loss |
| `player_storage` | Quest flags and system state – corruption breaks quests |
| `player_charms` | Bestiary progress – large data loss if corrupted |
| `player_depotitems` | Depot items – item loss if corrupted |
| `accounts` | Auth – corruption locks out all players |
| `houses` | Ownership – corruption causes house disputes |
| `market_offers` | Active trades – orphaned offers = coin loss |

---

## Default Data (from schema.sql)

Pre-seeded on schema install:

- **Account**: `god` with password hash `21298df8a3277357ee55b01df9530b535cf08ec1`
- **Players**: `Rook Sample`, `Sorcerer Sample`, `Druid Sample`, `Paladin Sample`, `Knight Sample`, `Monk Sample`, `GOD`
- **Towns**: seeded from initial data
