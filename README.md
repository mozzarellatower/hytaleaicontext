# Hytale Server Documentation (Unofficial)

Community-driven documentation for Hytale dedicated servers, reverse-engineered from `HytaleServer.jar`.

> Disclaimer: This is unofficial documentation based on reverse engineering. It is not affiliated with or endorsed by Hypixel Studios. API details may change between server versions.

---

## Source Boundaries

This repo separates sources:

- JAR-derived facts: extracted from `serverexample/Server/HytaleServer.jar` or server logs.
- Sample mods: observations from files in `tmp/mods/`.
- External claims: sourced from public docs or community sources and explicitly labeled.

---

## Agent Pointers (Read First)

If you are an agent, start with:

- `INSTRUCTIONS.MD` (workflow rules and scope)
- Check for `AGENTS.md` locally (task splits used by Codex/agent runs; may not exist in the public repo)
- `HytaleServer-Dissection.md` (JAR map and inferred guidance)

---

## Who This Is For

- Server admins who need CLI/config references.
- Plugin developers who need lifecycle, events, and registries.
- AI agents that need a clear map of the JAR and docs.

---

## Where To Start

- Server CLI/config and built-in plugins: `HytaleServer.md`
- JAR map and built-in plugin manifests: `HytaleServer-Dissection.md`
- Plugin API reference: `plugins/HytaleServer-Plugin-API.md`
- Plugin examples: `plugins/HytaleServer-Plugin-Examples.md`
- Structures guide (prefabs): `plugins/Structures-Guide.md`

Additional docs:
- Commands overview: `commands.md`
- Command examples: `commandexamples.md`
- Command mod notes: `commandmod.md`
- Permissions notes: `permissions_java.md`
- Sample mods scan (content packs): `tmp/mods-scan-assets.md`

---

## Quick Start

### Running a Server

The server JAR in this repo is located at `serverexample/Server/HytaleServer.jar`.

```bash
# Basic server start
java -jar serverexample/Server/HytaleServer.jar

# With custom mods directory
java -jar serverexample/Server/HytaleServer.jar --mods ./mods

# Specify bind address/port (build 2026.01.13)
java -jar serverexample/Server/HytaleServer.jar --bind 0.0.0.0:25565
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
  "Authors": [
    {
      "Name": "Your Name",
      "Email": "you@example.com",
      "Url": "https://example.com"
    }
  ],
  "Version": "1.0.0",
  "Main": "com.example.MyPlugin",
  "ServerVersion": "*"
}
```

---

## Server Features (High Level)

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

See `HytaleServer.md` for the complete list.

### Built-in Plugins (46 + 2 sub-plugins)

The server includes extensive built-in functionality (plus NPC sub-plugins `Spawning` and `Flock`):

- Core systems: Entity, Block, Item, Interaction, Damage, I18n modules
- Gameplay: Crafting, NPCs, Mounts, Weather, Farming, Combat
- Building: Builder Tools, Prefab system, World Generation
- Multiplayer: LAN Discovery, Instances, Teleportation

See `HytaleServer.md` for the complete list.

---

## Plugin API Overview

### Plugin Lifecycle

```
NONE -> SETUP -> START -> ENABLED -> SHUTDOWN
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

Common events:
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

## Modding Patterns (From Sample Mods)

These are high-level, non-verbatim patterns observed from sample mods in `tmp/mods/`:

- Blocks: JSON item definitions under `Server/Item/Items/...` with `BlockType`, plus optional hitboxes, groups, and recipes.
- Tools/usable items: item JSON with `Interactions` + separate interaction JSONs and root interaction mapping.
- UI: `.ui` files under `Common/UI/Custom/Pages/...` with textures in `Common/UI/Custom/...`, bound by plugin code at runtime.
- Packaging: `.jar` mods act as code plugins; `.zip` mods act as data/content packs (observed).

See `HytaleServer-Dissection.md` for expanded guidance.

Additional references:
- `plugins/Structures-Guide.md` (content pack flow + layout patterns)
- `HytaleServer-Dissection.md` (mod load-order notes)

---

## Example Plugins Included

The examples doc includes complete, working code for:

| Category | Examples |
|----------|----------|
| Essentials | Commands, Events, Config, Scheduled Tasks |
| Player Systems | Play Time, Stats, AFK Detection, Daily Rewards |
| Chat | Filter, Prefixes, Private Messaging |
| Economy | Balance, Payments, Shop integration |
| Teleportation | Warps, Homes, Spawn system |
| Admin | Player Management, Reports, Announcements |
| Gameplay | Kits, Cooldowns, Combat Logging, Polls |

---

## Project Structure

```
.
├── README.md
├── HytaleServer.md
├── HytaleServer-Dissection.md
├── commands.md
├── commandexamples.md
├── commandmod.md
├── permissions_java.md
├── plugins/
│   ├── README.md
│   ├── HytaleServer-Plugin-API.md
│   ├── HytaleServer-Plugin-Examples.md
│   └── Structures-Guide.md
├── serverexample/
│   └── Server/
│       └── HytaleServer.jar
└── tmp/  # scratch, ignored
```

---

## Requirements

- Java: 21 or higher
- OS: Windows, macOS, or Linux

### Plugin Development

Maven:
```xml
<dependency>
    <groupId>com.hypixel.hytale</groupId>
    <artifactId>HytaleServer</artifactId>
    <version>2026.01.13</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/HytaleServer.jar</systemPath>
</dependency>
```

Gradle:
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

1. Report inaccuracies (API changes between versions)
2. Add examples (new plugin patterns and use cases)
3. Improve clarity (better explanations and code samples)
4. Document new features (as the game updates)

---

## Reverse Engineering Notes

This documentation was created by analyzing:

- Class and method signatures via `strings` extraction
- Embedded `manifests.json` for plugin structure
- Package organization and naming conventions
- Built-in plugin implementations as reference

Tools used:
- `unzip` (JAR inspection)
- `strings` (binary string extraction)
- Class structure analysis

Not used (unavailable):
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

Official:
- https://hytale.com
- https://hypixelstudios.com

---

## License

This documentation is provided for educational purposes. Hytale and related trademarks are property of Hypixel Studios.

---

Last updated: January 2026
