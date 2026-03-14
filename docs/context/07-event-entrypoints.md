# 07 – Event Entry Points

## Player Login

**Trigger**: Player successfully authenticates and enters the game world.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::onRecvFirstMessage()` → `IOLoginData::loadPlayer()` → `Game::addCreature()` |
| C++ | `src/io/functions/iologindata_load_player.cpp` |
| Lua | `data/scripts/creaturescripts/` → handlers registered for `onLogin` |
| Lua event type | `CreatureEvent::onLogin(player)` |

Steps:
1. `ProtocolGame` receives login packet
2. `IOLoginData::loadPlayer()` reads player from DB (items, spells, storage, conditions)
3. `Game::addCreature(player, tile)` places player on map
4. CreatureEvent `onLogin` fires → Lua handlers run
5. Map description sent to client

---

## Player Logout

**Trigger**: Player disconnects or uses logout.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::logout()` → `Game::removeCreature()` → `IOLoginData::savePlayer()` |
| C++ | `src/io/functions/iologindata_save_player.cpp` |
| Lua | CreatureEvent `onLogout(player)` |

Steps:
1. Disconnect detected or `/logout` talkaction
2. `onLogout` Lua event fires
3. Player saved to DB
4. Creature removed from map

---

## Combat Events

**Damage applied**:

| Layer | Location |
|-------|---------|
| C++ | `Combat::doCombat()` → `Combat::doCombatHealth()` / `doCombatMana()` |
| C++ | `src/creatures/combat/combat.cpp` |
| Lua | `CreatureEvent::onHealthChange(creature, attacker, primaryDamage, primaryType, ...)` |
| Lua | `CreatureEvent::onManaChange(creature, attacker, manaChange, origin)` |

**Death**:

| Layer | Location |
|-------|---------|
| C++ | `Creature::onDeath()` → `Game::executeDeath()` |
| Lua | `CreatureEvent::onPrepareDeath(creature, killer)` |
| Lua | `CreatureEvent::onDeath(creature, corpse, killer, mostDamageKiller, unjustified, ...)` |

**Kill recorded**:

| Layer | Location |
|-------|---------|
| Lua | `CreatureEvent::onKill(creature, target)` |
| DB | Insert into `player_kills`, `player_deaths` |

---

## Spell Casting

**Trigger**: Player sends spell command (voice word or hotkey).

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::parseCastSpell()` → `Spells::getInstantSpellByWords()` → `InstantSpell::cast()` |
| C++ | `src/creatures/combat/spells.hpp/cpp` |
| Lua | `spell:onCastSpell(creature, variant)` in `data/scripts/spells/` |

Rune use:
- `ProtocolGame::parseUseItem()` → checks if item is a rune → `RuneSpell::cast()`
- Lua: `data/scripts/runes/*.lua`

---

## Item Usage

**Trigger**: Player right-clicks and uses an item.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::parseUseItem()` → `Game::playerUseItem()` |
| C++ | `src/game/game.cpp` |
| Lua | `action:onUse(player, item, fromPosition, target, toPosition, isHotkey)` |
| Scripts | `data/scripts/actions/` |

Item use on target:
- `ProtocolGame::parseUseItemEx()` → `Game::playerUseItemEx()`

---

## Movement Events

**Trigger**: Creature steps onto a tile.

| Layer | Location |
|-------|---------|
| C++ | `Game::addCreatureToMapTile()` → fires `MoveEvent` |
| Lua | `moveevent:onStepIn(creature, item, position, fromPosition)` |
| Lua | `moveevent:onStepOut(creature, item, position, fromPosition)` |
| Scripts | `data/scripts/movements/` |

Item added/removed from tile:
- `moveevent:onAddItem(moveitem, tileitem, position)`
- `moveevent:onRemoveItem(moveitem, tileitem, position)`

---

## NPC Interactions

**Trigger**: Player says something near an NPC or opens a trade window.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::parseTalk()` → `Game::playerSay()` → NPC speech handler |
| C++ | `src/creatures/npcs/npc.hpp/cpp` |
| Lua | NPC behavior scripts in `data-crystal/npc/` |
| Lua | Shop API: `npc:openShopWindow(player, items)` |

---

## Quest Triggers

**Trigger**: Varies by quest type – usually item use, step-in, creature death, or NPC dialog.

| Layer | Location |
|-------|---------|
| Lua | `data-crystal/scripts/quests/*.lua` |
| Storage | Quest state stored in `player_storage` table via `player:setStorageValue()` |
| Lua libs | `data-crystal/lib/quests/quest.lua`, `arena.lua`, `demon_oak.lua` |

---

## Skill/Level Advancement

**Trigger**: Player gains enough experience or skill points.

| Layer | Location |
|-------|---------|
| C++ | `Player::addExperience()` → `Player::levelUp()` |
| Lua | `CreatureEvent::onAdvance(player, skill, oldLevel, newLevel)` |

---

## Server Startup

| Layer | Location |
|-------|---------|
| C++ | `CrystalServer::init()` |
| Lua | `GlobalEvent::onStartup()` handlers in `data/scripts/globalevents/` |

---

## Scheduled Global Events

| Layer | Location |
|-------|---------|
| C++ | `GlobalEvents` singleton with timer queue |
| Lua | `globalEvent:onTime(interval)` in `data/scripts/globalevents/` |
| Uses | Server saves, boss resets, boosted creature rotation, raid scheduling |

---

## Raids

**Trigger**: Time-based or manual activation.

| Layer | Location |
|-------|---------|
| C++ | `Raids` class (`src/lua/creature/raids.hpp/cpp`) |
| XML | `data-crystal/raids/` |
| Lua | `data/libs/systems/raids.lua` |

---

## Modal Windows (UI Dialogs)

**Trigger**: Server sends a modal window to player; player responds.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::sendModalWindow()` / `parseModalWindowAnswer()` |
| Lua | `CreatureEvent::onModalWindow(player, modalWindowId, buttonId, choiceId)` |

---

## Extended Opcodes

**Trigger**: Custom client→server packet via opcode `0xDA`.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::parseExtendedOpcode()` |
| Lua | `CreatureEvent::onExtendedOpcode(player, opcode, buffer)` |

---

## Talkactions (Chat Commands)

**Trigger**: Player types a word/command in chat.

| Layer | Location |
|-------|---------|
| C++ | `ProtocolGame::parseTalk()` → `Game::playerSay()` → `TalkAction::checkWord()` |
| C++ | `src/lua/creature/talkaction.hpp/cpp` |
| Lua | `talkaction:onSay(player, words, param, channel)` |
| Scripts | `data/scripts/talkactions/` |

---

## Summary Table

| Event | C++ Entry | Lua Hook |
|-------|-----------|----------|
| Player login | `ProtocolGame::onRecvFirstMessage` | `onLogin(player)` |
| Player logout | `ProtocolGame::logout` | `onLogout(player)` |
| Damage dealt | `Combat::doCombat` | `onHealthChange(...)` |
| Player death | `Creature::onDeath` | `onDeath(...)` |
| Spell cast | `Spells::getInstantSpellByWords` | `onCastSpell(creature, variant)` |
| Item use | `Game::playerUseItem` | `onUse(player, item, ...)` |
| Step on tile | `Game::addCreatureToMapTile` | `onStepIn(creature, ...)` |
| Level up | `Player::levelUp` | `onAdvance(player, skill, old, new)` |
| Chat command | `Game::playerSay` | `onSay(player, words, param, channel)` |
| Server start | `CrystalServer::init` | `onStartup()` |
| Scheduled event | `GlobalEvents` timer | `onTime(interval)` |
| NPC talk | `Npc` speech handler | NPC behavior scripts |
| Modal answer | `ProtocolGame::parseModalWindowAnswer` | `onModalWindow(player, ...)` |
