# 09 – Known Gotchas & Fragile Areas

## Item Serialization

- **Risk**: Player items are serialized as binary blobs (`attributes` BLOB column) in `player_items`, `player_depotitems`, etc.
- Any change to item attribute encoding (`src/items/functions/item/attribute.hpp`) can corrupt existing saved items.
- Always test item save/load after modifying `ItemAttribute` or `CustomAttribute` serialization.
- Containers are saved as trees (`pid`/`sid` parent-child structure); wrong parentage = invisible items.

## Conditions (Status Effects) Serialization

- Player conditions are serialized as a single BLOB in `players.conditions`.
- Modifying `Condition` class structure without updating serialization breaks saved conditions.
- **File**: `src/creatures/combat/condition.hpp/cpp`

## Lua Global State Sharing

- All Lua scripts share a single global state.
- Variable name collisions between scripts can cause subtle bugs.
- Reload (`/reload scripts`) reinitializes the state; any global persisted state is lost unless in KV/storage.

## Event Registration Order

- Scripts are loaded in filesystem order within each directory.
- Load order can matter when one script depends on a library loaded by another.
- If `onLogin` or `onDeath` doesn't fire, check if the event was registered correctly (`registerCreatureEvent` called before the event fires).

## Dispatcher Task Groups

- Tasks in the `Serial` group run sequentially – a long-running task blocks the queue.
- Do not perform heavy computation in serial tasks; offload to `ThreadPool` group.
- Task expiry is 2000 ms – tasks dropped silently if the queue is backed up.
- **File**: `src/game/scheduling/dispatcher.hpp`

## Database Query Size Limit

- MySQL connection is configured with max 8 MB per query (`src/database/database.hpp`).
- Saving players with very large depots or item sets could hit this limit.
- Large `player_depotitems` saves are a known risk for high-stash accounts.

## Map Loading Performance

- Full `.otbm` map load is single-threaded and synchronous – server is not ready until complete.
- Very large maps increase startup time significantly.
- `MapCache` caches tiles but memory usage scales with map size.

## Thread Safety in Game Logic

- Most game state is accessed from the Dispatcher's serial queue (single-threaded by design).
- `DatabaseTasks` (`src/database/databasetasks.hpp`) runs async – DB callbacks must not directly mutate game state; must re-dispatch to serial queue.
- `ThreadPool` tasks that touch creature or map state without dispatching back to serial = race conditions.

## Bosstiary / Bestiary Data Race

- `IOBestiary` and `IOBosstiary` methods touch shared player charm/bosstiary data.
- If a player logs out while bestiary data is being written async, partial saves can occur.

## House Auction Edge Cases

- `houses.bid_end` expiry is checked at server startup and on login – not continuously polled.
- A crash during bid resolution can leave houses in inconsistent bid state.
- `house_lists` updates must be synchronized with house `owner` field.

## Player Save Failures

- `IOLoginData::savePlayer()` failures are logged but the engine continues; data loss occurs silently.
- **File**: `src/io/functions/iologindata_save_player.cpp`
- No retry mechanism by default.

## NPC Script Complexity

- NPC behavior is tightly coupled to specific dialog state machines in Lua.
- Crystal-specific NPC scripts in `data-crystal/npc/` may rely on storage keys defined elsewhere.
- Changing storage key values for quests can break NPC dialog flow.

## Reload Limitations

- `/reload scripts` reloads Lua but not XML (items, monsters, vocations, mounts).
- Changing item definitions requires a full server restart.
- Reload does not clear the map or respawn creatures.

## Wheel of Destiny Gem Data

- Wheel gem data is stored as binary in `player_wheeldata`.
- The format is tied to the protobuf definitions in `src/protobuf/`.
- Changing protobuf definitions without migration = corrupted wheel data for existing players.

## Market Orphaned Offers

- If a player is deleted while having active market offers, the offers remain with an invalid `player_id`.
- No automatic cleanup; can accumulate dead offers over time.

## `players_online` MEMORY Table

- `players_online` is a MySQL MEMORY table – wiped on MySQL restart.
- If the server crashes without cleaning it, stale records persist until the next server start clears it.
- Queries that JOIN against `players_online` can return incorrect results post-crash.

## Spell Registration Collisions

- Spells are registered by name in the `Spells` singleton.
- Two scripts registering the same spell name silently overwrites the first.
- Always check for duplicate spell names when adding new spells.

## Config Hot-Reload

- `ConfigManager` reads config at startup only – no runtime reload.
- Changes to `config.lua` require a full server restart.

## TalkAction Word Collisions

- TalkActions use exact word matching; two actions registered for the same word → undefined behavior (last registered wins).

## LuaJIT Stack Overflow

- Deep recursive Lua calls (e.g., complex quest chains) can overflow the LuaJIT stack.
- Default Lua stack is limited; avoid deep recursion in NPC dialog or event handlers.

## Augment / Proficiency Systems

- `weapon_proficiencies` and `harmony`/`virtue` columns in `players` table are newer additions.
- These systems may have incomplete Lua coverage or missing config values (confirm).

## Missing Anti-Cheat / Rate Limiting

- `maxPacketsPerSecond = 25` is the only built-in rate limit.
- No server-side pathfinding validation – players can be moved to invalid positions via packet manipulation.
- Speed hacks are detected client-side by Tibia's anti-cheat, not server-side.
