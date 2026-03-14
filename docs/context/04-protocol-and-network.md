# 04 – Protocol & Network

## Protocol Version

- **Client**: Tibia 15.x
- **Login protocol identifier**: `0x01` (`ProtocolLogin::PROTOCOL_IDENTIFIER`)
- **Checksum**: enabled (`USE_CHECKSUM = true`)
- **Encryption**: XTEA symmetric encryption on game packets
- **Compression**: zlib (level configured by `packetCompressionLevel`, default `6`)
- **RSA**: used for initial key exchange on login

## Network Handling Classes

| Class | File | Role |
|-------|------|------|
| `Server` | `src/server/server.hpp` | Binds ports, accepts connections |
| `Connection` | `src/server/network/connection/connection.hpp` | Wraps a single TCP connection (ASIO) |
| `Protocol` | `src/server/network/protocol/protocol.hpp` | Base protocol (XTEA, checksum, compression) |
| `ProtocolLogin` | `src/server/network/protocol/protocollogin.hpp` | Handles login handshake |
| `ProtocolGame` | `src/server/network/protocol/protocolgame.hpp` | Handles all in-game packets |
| `ProtocolStatus` | `src/server/network/protocol/protocolstatus.hpp` | Server status queries |
| `NetworkMessage` | `src/server/network/message/networkmessage.hpp` | Incoming packet reader |
| `OutputMessage` | `src/server/network/message/outputmessage.hpp` | Outgoing packet builder |

## Base Protocol Class

**File**: `src/server/network/protocol/protocol.hpp`

```cpp
class Protocol : public std::enable_shared_from_this<Protocol>
  - Connection_ptr connection
  - uint32_t[] xteaKey             // XTEA encryption key
  - bool encryptionEnabled
  - bool compressionEnabled
  - z_stream* zStream              // zlib state

  virtual parsePacket(msg)         // override in subclasses
  virtual onSendMessage(msg)
  virtual onRecvMessage(msg)

  - encryptMessage(OutputMessage&)
  - decryptMessage(NetworkMessage&)
  - XTEA_encrypt() / XTEA_decrypt()
  - enableChecksum()
```

## Login Protocol

**File**: `src/server/network/protocol/protocollogin.hpp`

```
ProtocolLogin::onRecvFirstMessage(msg)
  1. Read client version / OS
  2. RSA decrypt → extract XTEA key
  3. Validate account + password (Argon2)
  4. Call getCharacterList()
  5. Send character list packet
  6. Disconnect

Constants:
  PROTOCOL_IDENTIFIER = 0x01
  USE_CHECKSUM = true
```

## Game Protocol

**File**: `src/server/network/protocol/protocolgame.hpp`

Key type definitions at top of file:
```cpp
static constexpr uint16_t MAX_KNOWN_CREATURES = 1300;
using UsersMap = std::map<uint32_t, std::shared_ptr<ProtocolGame>>;
using MarketOfferList = std::vector<MarketOffer>;
using ItemsTierCountList = std::vector<std::pair<uint16_t, uint8_t>>;
struct TextMessage { MessageType type; std::string text; uint16_t channelId; Color_t color; };
```

`ProtocolGame` handles:
- Player session lifecycle
- Sending map data
- Parsing all client→server opcodes
- Sending all server→client packets

## Packet Flow

### Client → Server (Incoming)

```
TCP data arrives
  Connection reads bytes
  NetworkMessage parses header (length + checksum)
  XTEA decrypt (if enabled)
  Protocol::parsePacket(msg) called
    ProtocolGame reads opcode byte
    Dispatches to handler method
    Handler may call Game::* or Player::*
```

### Server → Client (Outgoing)

```
Engine calls ProtocolGame::send*(...)
  OutputMessage built (opcode + data)
  XTEA encrypt
  zlib compress (if enabled)
  Connection::send(outputMsg)
  ASIO async_write to socket
```

## Key Packet Handlers (ProtocolGame)

Common handler method naming convention: `parse*` for incoming, `send*` for outgoing.

**Incoming (client → server) examples:**
- `parseTalk()` – chat message
- `parseUseItem()` – item use
- `parseMoveCreature()` / `parseWalk()` – movement
- `parseAttack()` – combat initiation
- `parseCastSpell()` – spell cast
- `parseMarketRequest()` – market operations
- `parseExtendedOpcode()` – extended custom opcodes

**Outgoing (server → client) examples:**
- `sendAddCreature()` – add creature to known list
- `sendMoveCreature()` – creature moved
- `sendMapDescription()` – full/partial map update
- `sendTextMessage()` – chat/system message
- `sendStats()` – player stats update
- `sendInventoryItem()` – inventory change
- `sendMarketEnter()` / `sendMarketLeave()`
- `sendModalWindow()` – UI dialog
- `sendBestiaryCharms()` – charm window
- `sendPreyData()` – prey slot data
- `sendWheelOfDestinyData()` – wheel UI

## Extended Opcodes

- Mechanism for custom client↔server communication without official Tibia opcodes.
- Parsed by `ProtocolGame::parseExtendedOpcode()`.
- Dispatched to Lua via `CreatureEvent::onExtendedOpcode(player, opcode, buffer)`.
- Used for custom UI overlays, analytics, or third-party client features.
- **Lua API**: `player:sendExtendedOpcode(opcode, buffer)`

## Where to Add New Packets

### New server → client packet:
1. Add `send*()` method to `ProtocolGame` (`protocolgame.hpp` / `.cpp`)
2. Create `OutputMessage`, write opcode byte + data
3. Call `writeToOutputBuffer(msg)`

### New client → server packet:
1. Add opcode constant (verify against client version 15)
2. Add case to `ProtocolGame::parsePacket()` switch
3. Implement `parse*()` handler method
4. Call appropriate `Game::*` or `Player::*` methods

### New extended opcode (no client modification):
1. Use existing `parseExtendedOpcode()` path
2. Register handler in Lua via `CreatureEvent` `onExtendedOpcode`
3. Client must send opcode `0xDA` + your custom byte + buffer

## Network Configuration

From `config.lua.dist`:

```lua
ip = "127.0.0.1"
loginProtocolPort = 7171
gameProtocolPort = 7172
statusProtocolPort = 7171
maxPacketsPerSecond = 25
packetCompressionLevel = 6    -- 0 = disabled, 1-9 zlib levels
```

## Webhook Support

**File**: `src/server/network/webhook/webhook.hpp`

- HTTP POST webhooks (via libcurl) for external integrations.
- Accessible from Lua via `webhook:*` functions.
- Useful for Discord notifications, monitoring, etc.
