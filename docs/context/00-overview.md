# 00 – Project Overview

## Description

Crystal Server is an open-source C++ RPG game server engine compatible with **Tibia client version 15**.
It is a fork/continuation of the Crystal engine lineage, implementing the full game protocol and Lua-scripted gameplay systems.

## Technologies

| Layer | Technology |
|-------|-----------|
| Core engine | C++20 |
| Scripting | LuaJIT |
| Database | MySQL / MariaDB |
| Networking | ASIO (async TCP) |
| Encryption | XTEA (in-game), RSA (login), Argon2 (passwords) |
| Serialization | Protobuf, PugiXML |
| Logging | spdlog |
| Metrics | Prometheus (optional) |
| Build | CMake + vcpkg |

## Supported Client Version

- **Tibia 15.x** (protocol identifiers match client 15 handshake)
- Login protocol identifier: `0x01`
- Game protocol uses XTEA encryption + optional zlib compression (`packetCompressionLevel = 6`)

## How the Server is Structured

```
crystalserver/
├── src/          # C++ engine (248 headers, 186 sources)
├── data/         # Core Lua scripts, XML configs, migrations
├── data-crystal/ # Crystal-specific game content (monsters, NPCs, maps)
├── data-global/  # Alternative global datapack
├── schema.sql    # Full database schema
└── config.lua.dist  # Configuration template
```

The engine separates **content** (data-crystal/) from **core scripting libraries** (data/) and the **C++ engine** (src/).

## General Architecture

```
main.cpp
  └── CrystalServer::init()
        ├── ConfigManager::load(config.lua)
        ├── DatabaseManager::connect()
        ├── Game::loadMainMap()
        ├── Lua environment init (LuaScriptInterface)
        ├── Monster/NPC definitions loaded from XML
        ├── ProtocolLogin / ProtocolGame / ProtocolStatus bound to ports
        └── Game::start() → main scheduler loop
```

- **Game** is a singleton (`Game::getInstance()`) that drives the entire server.
- The **Dispatcher** handles task queues: Walk, Serial, GenericParallel, ThreadPool.
- **Scheduler** ticks every 50 ms minimum; server beat is `0x32` (50 ms).
- Lua scripts are loaded at startup; most gameplay logic lives in Lua.

## How to Run Locally

```bash
# 1. Install dependencies via vcpkg (handled by CMake)
cmake --preset linux-release
cmake --build --preset linux-release

# 2. Configure
cp config.lua.dist config.lua
# Edit config.lua: set ip, database credentials, dataPackDirectory

# 3. Database
mysql -u root -p < schema.sql

# 4. Run
./crystalserver
```

Docker support available in `docker/`.

## Required Dependencies

Managed via **vcpkg** (`vcpkg.json`):

- `asio`, `luajit`, `libmariadb`, `pugixml`, `spdlog`, `curl`
- `protobuf`, `parallel-hashmap`, `magic-enum`, `mio`
- `mpir`, `abseil`, `bshoshany-thread-pool`

## Port Defaults

| Service | Port |
|---------|------|
| Login | 7171 |
| Game | 7172 |
| Status | 7171 |
