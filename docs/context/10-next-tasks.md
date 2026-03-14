# 10 – Next Tasks & Improvement Opportunities

## Refactoring Opportunities

### Player Class Decomposition
- `Player` class (`src/creatures/players/player.hpp`) aggregates too many systems directly.
- Consider extracting more subsystems into their own components following the existing pattern (Wheel, Achievement, etc.).
- The `players` DB table has 155+ columns – some systems could be normalized into separate tables.

### IOLoginData Split
- `iologindata_load_player.cpp` and `iologindata_save_player.cpp` are large and monolithic.
- Could be split per-system (items, skills, storage, charms) for easier maintenance.

### Combat Callback Hierarchy
- `Combat` class has multiple callback types (Value, Tile, Target, Chain) that could benefit from a unified callback interface.
- **File**: `src/creatures/combat/combat.hpp`

### Spell System
- `InstantSpell` and `RuneSpell` share much logic – potential for better base class abstraction.
- **File**: `src/creatures/combat/spells.hpp`

### Config Access
- Direct `g_config.getString(key)` calls scattered throughout the engine.
- Consider grouping config reads into typed structs per system (already partially done with `config_enums.hpp`).

---

## Code Organization Improvements

### Event System Unification
- Three parallel event systems (CreatureEvent, GlobalEvent, EventCallback) have overlapping functionality.
- A unified event bus would reduce the mental overhead of knowing which system to use.

### Lua Binding Organization
- `src/lua/functions/` has 60+ files – difficult to navigate.
- Adding a `README.md` per subdirectory listing exported functions would help.

### Monster and NPC Definitions
- Monster definitions in XML spread across 25+ directories in `data-crystal/monster/`.
- No index file; finding a specific monster requires `grep`.
- Consider a catalog Lua file that maps monster names to file paths.

### Migrations Naming
- Migration files in `data/migrations/` are numbered but not named descriptively.
- Rename with descriptive suffixes (e.g., `15-add-forge-history.lua`).

---

## Performance Improvements

### Player Save Bottleneck
- Player saves during logout are synchronous in the serial task queue.
- Large player saves (big depots) can delay the queue.
- Move saves fully to `DatabaseTasks` async queue with proper state snapshot.

### Map Sector Queries
- `Spectators::getSpectators()` is called frequently for every packet broadcast.
- Benchmark and profile – may benefit from spatial index or sector pre-computation.

### Item Attribute Deserialization
- `ItemAttribute` deserialization happens on every item load from DB.
- A lazy-loading or cache-on-first-access pattern could reduce startup time.

### Index Review
- Several queries on `player_storage (player_id, key)` – verify composite index exists.
- `market_offers` queries by `itemtype` and `sale` – verify index coverage.
- `player_bosstiary` lookups by `player_id, bossid` – verify composite index.

### Dispatcher Serial Queue
- Any task that sleeps or blocks in the serial queue stalls the entire server.
- Audit all serial tasks for blocking DB calls that should use `DatabaseTasks`.

---

## Missing Documentation

- No inline Doxygen comments on most C++ classes (Doxyfile.in exists but docs are sparse).
- `data-crystal/scripts/quests/` has no quest index or dependency map.
- No documentation on the item attribute binary format.
- The protobuf schema in `src/protobuf/` is undocumented.
- No documented list of all Lua global functions exported by the engine.
- `data/libs/tables/` constant tables are not documented.

---

## Potential New Features / Systems

### Server-Side Anti-Cheat
- Add server-side position validation for movement packets.
- Track movement speed and flag suspicious deltas.

### Hot Config Reload
- Allow `config.lua` to be reloaded at runtime (at least for non-critical settings like rates, pvp type).

### Structured Quest System
- Replace scattered `player_storage` quest flags with a dedicated `player_quests` table.
- Would enable better progress reporting and easier debugging.

### Market Cleanup Job
- Add a scheduled GlobalEvent to clean up orphaned market offers (no valid `player_id`).

### Player Audit Log
- Add a `player_audit` table for tracking significant actions (level up, item forge, house purchase).
- Useful for GM investigation and anti-cheat.

### Metric Dashboards
- Prometheus metrics (`src/lib/metrics/`) are available but may lack game-specific metrics.
- Add metrics for: active players, combat events/sec, DB query latency, scheduler queue depth.

### Lua Type Safety
- Current Lua bindings use raw `lua_push*` / `lua_to*` with no type safety.
- Consider adding a thin typed wrapper layer to catch Lua→C++ type mismatches at runtime.

---

## Areas Requiring Immediate Attention

| Issue | Priority | File |
|-------|----------|------|
| Silent player save failures | High | `src/io/functions/iologindata_save_player.cpp` |
| `players_online` MEMORY table crash recovery | Medium | `schema.sql` |
| Market orphaned offers | Low | `src/io/iomarket.cpp` |
| Missing composite DB indexes | Medium | `schema.sql` |
| Wheel/protobuf migration path | High (if changing protobuf) | `src/protobuf/` |
