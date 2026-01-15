<<<<<<< HEAD
# hytaleassetcontext
=======
# Hytale Server Documentation (Unofficial)

Community-driven documentation for Hytale dedicated servers, reverse-engineered from `HytaleServer.jar`.

> **Disclaimer:** This is unofficial documentation based on reverse engineering. It is not affiliated with or endorsed by Hypixel Studios. API details may change between server versions.

---

## Quick Start

### Running a Server

```bash
# Basic server start
java -jar HytaleServer.jar

# With custom mods directory
java -jar HytaleServer.jar --mods ./mods

# Specify bind address/port (build 2026.01.13)
java -jar HytaleServer.jar --bind 0.0.0.0:25565
```

### Creating Your First Plugin

1. Set up a Java 21 project with `HytaleServer.jar` as a dependency
2. Create a class extending `JavaPlugin`
3. Add a `manifest.json` to your JAR root
4. Place the JAR in your server's `mods` folder

```java
public final class MyPlugin extends JavaPlugin {
    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    public void setup() {
        getLogger().info("Hello, Hytale!");
    }

    @Override
    public void start() {}

    @Override
    public void shutdown() {}
}
```

```json
{
  "Group": "MyName",
  "Name": "MyPlugin",
  "Version": "1.0.0",
  "Main": "com.example.MyPlugin",
  "ServerVersion": "*"
}
```

---

## Documentation

### Server Documentation

| Document | Description |
|----------|-------------|
| [HytaleServer.md](./HytaleServer.md) | Server CLI options, configuration, and built-in plugins |
| [HytaleServer-Dissection.md](./HytaleServer-Dissection.md) | JAR package map and inferred mod/plugin use cases |

### Plugin Development

| Document | Description |
|----------|-------------|
| [Plugin API Reference](./plugins/HytaleServer-Plugin-API.md) | Complete API documentation for plugin development |
| [Plugin Examples](./plugins/HytaleServer-Plugin-Examples.md) | 21 ready-to-use plugin examples |
| [Structures Guide](./plugins/Structures-Guide.md) | Saving and loading structures (prefabs) |

---

## Server Features

### CLI Options (Observed in 2026.01.13)

| Flag | Description |
|------|-------------|
| `--bind <host:port>` | Port/address to listen on |
| `--mods <path>` | Additional mods directories |
| `--universe <path>` | Universe data directory |
| `--assets <path>` | Asset directory (dir or zip) |
| `--world-gen <path>` | World generation directory |
| `--accept-early-plugins` | Allow early plugins (unstable) |
| `--early-plugins <path>` | Additional early plugin directories |

See [HytaleServer.md](./HytaleServer.md) for the complete list.

### Built-in Plugins (46 + 2 sub-plugins)

The server includes extensive built-in functionality (plus NPC sub-plugins `Spawning` and `Flock`):

- **Core Systems:** Entity, Block, Item, Interaction, Damage, I18n modules
- **Gameplay:** Crafting, NPCs, Mounts, Weather, Farming, Combat
- **Building:** Builder Tools, Prefab system, World Generation
- **Multiplayer:** LAN Discovery, Instances, Teleportation

See [HytaleServer.md](./HytaleServer.md) for the complete list.

---

## Plugin API Overview

### Plugin Lifecycle

```
NONE → SETUP → START → ENABLED → SHUTDOWN
```

| Method | Purpose |
|--------|---------|
| `setup()` | Register configs, commands, events |
| `start()` | Initialize gameplay systems |
| `shutdown()` | Save data, cleanup resources |

### Available Registries

| Registry | Purpose |
|----------|---------|
| `getLogger()` | Logging |
| `getEventRegistry()` | Event handlers |
| `getCommandRegistry()` | Commands |
| `getBlockStateRegistry()` | Block states |
| `getEntityRegistry()` | Entities |
| `getTaskRegistry()` | Scheduled tasks |
| `getAssetRegistry()` | Assets |
| `getDataDirectory()` | Plugin data folder |

### Event System

```java
// Register in setup()
getEventRegistry().register(PlayerConnectEvent.class, event -> {
    getLogger().info("Player joined: " + event.getPlayer().getName());
});
```

**Common Events:**
- `PlayerConnectEvent`, `PlayerDisconnectEvent`, `PlayerChatEvent`
- `BreakBlockEvent`, `PlaceBlockEvent`, `UseBlockEvent`
- `BootEvent`, `ShutdownEvent`

### Command System

```java
getCommandRegistry().registerCommand(new MyCommand());

class MyCommand extends AbstractCommand {
    public MyCommand() {
        super("mycommand", "Description here");
    }
}
```

---

## Example Plugins Included

The [examples documentation](./plugins/HytaleServer-Plugin-Examples.md) includes complete, working code for:

| Category | Examples |
|----------|----------|
| **Essentials** | Commands, Events, Config, Scheduled Tasks |
| **Player Systems** | Play Time, Stats, AFK Detection, Daily Rewards |
| **Chat** | Filter, Prefixes, Private Messaging |
| **Economy** | Balance, Payments, Shop integration |
| **Teleportation** | Warps, Homes, Spawn system |
| **Admin** | Player Management, Reports, Announcements |
| **Gameplay** | Kits, Cooldowns, Combat Logging, Polls |

---

## Project Structure

```
hytaledocs/
├── README.md                    # This file
├── HytaleServer.jar             # Server JAR (for reference)
├── HytaleServer.md              # Server documentation
└── plugins/
    ├── README.md                # Plugin docs index
    ├── HytaleServer-Plugin-API.md      # API reference
    ├── HytaleServer-Plugin-Examples.md # 21 examples
    └── Structures-Guide.md      # Prefab/structure guide
```

---

## Requirements

- **Java:** 21 or higher
- **OS:** Windows, macOS, or Linux

### Plugin Development

**Maven:**
```xml
<dependency>
    <groupId>com.hypixel.hytale</groupId>
    <artifactId>HytaleServer</artifactId>
    <version>2026.01.13</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/HytaleServer.jar</systemPath>
</dependency>
```

**Gradle:**
```groovy
compileOnly files('lib/HytaleServer.jar')
```

---

## Key Packages

| Package | Contents |
|---------|----------|
| `com.hypixel.hytale.server.core.plugin` | `JavaPlugin`, `PluginBase`, lifecycle |
| `com.hypixel.hytale.server.core.command.system` | Command framework |
| `com.hypixel.hytale.server.core.event.events` | Event classes |
| `com.hypixel.hytale.server.core.prefab` | Structure/prefab system |
| `com.hypixel.hytale.server.core.modules` | Core game modules |
| `com.hypixel.hytale.common.plugin` | `PluginManifest`, `PluginIdentifier` |
| `com.hypixel.hytale.event` | `EventRegistry`, `EventPriority` |

---

## Contributing

This documentation is community-maintained. Contributions welcome:

1. **Report inaccuracies** - API changes between versions
2. **Add examples** - New plugin patterns and use cases
3. **Improve clarity** - Better explanations and code samples
4. **Document new features** - As the game updates

---

## Reverse Engineering Notes

This documentation was created by analyzing:

- Class and method signatures via `strings` extraction
- Embedded `manifests.json` for plugin structure
- Package organization and naming conventions
- Built-in plugin implementations as reference
- Public example plugins from the community (see below)

**Tools used:**
- `unzip` - JAR inspection
- `strings` - Binary string extraction
- Class structure analysis

**Not used (unavailable):**
- Full decompilation (Fernflower/CFR not available)
- Runtime debugging

---

## Version Information

| Item | Version |
|------|---------|
| Server Build | `2026.01.13` |
| Build Hash | `dcad8778f` |
| Java Target | 21 |
| Documentation | v1.0.0 |

---

## Resources

### Official
- [Hytale Website](https://hytale.com)
- [Hypixel Studios](https://hypixelstudios.com)

### Community
- Report documentation issues via pull requests
- Share your plugins with the community

### Example Plugins (Public Repos)
- https://github.com/nitrado/hytale-plugin-webserver (HTTP server plugin, shared servlet layer)
- https://github.com/nitrado/hytale-plugin-query (server info API, content negotiation)
- https://github.com/apexhosting/hytale-plugin-prometheus (Prometheus metrics exporter)
- https://github.com/nitrado/hytale-plugin-performance-saver (TPS + view radius throttling)

---

## License

This documentation is provided for educational purposes. Hytale and related trademarks are property of Hypixel Studios.

---

*Last updated: January 2026*
>>>>>>> f4bfcd4 (First commit)
