# Hytale Server Plugin API Guide (Reverse-Engineered)

This guide is based on inspecting `HytaleServer.jar` (build `2026.01.13-dcad8778f`). It is not an official API reference; verify against the actual server build you are targeting.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Plugin Structure](#plugin-structure)
3. [Lifecycle Methods](#lifecycle-methods)
4. [Event System](#event-system)
5. [Command System](#command-system)
6. [Configuration](#configuration)
7. [Registries Reference](#registries-reference)
8. [Asset Packs](#asset-packs)
9. [Dependencies](#dependencies)
10. [Complete Examples](#complete-examples)

---

## Quick Start

### Requirements

- Java 21 or higher
- `HytaleServer.jar` for compilation

### Project Setup

#### Maven

Create a `pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>MyPlugin</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.hypixel.hytale</groupId>
            <artifactId>HytaleServer</artifactId>
            <version>2026.01.13</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/HytaleServer.jar</systemPath>
        </dependency>
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
</project>
```

Place `HytaleServer.jar` in your project's `lib/` directory.

#### Gradle

Create a `build.gradle`:

```groovy
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    compileOnly files('lib/HytaleServer.jar')
}

jar {
    from('src/main/resources') {
        include 'manifest.json'
    }
}
```

### Minimal Plugin Example

**Directory structure:**

```
MyPlugin/
├── lib/
│   └── HytaleServer.jar
├── src/main/java/com/example/myplugin/
│   └── MyPlugin.java
├── src/main/resources/
│   └── manifest.json
└── pom.xml (or build.gradle)
```

**MyPlugin.java:**

```java
package com.example.myplugin;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import javax.annotation.Nonnull;

public final class MyPlugin extends JavaPlugin {

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    public void setup() {
        getLogger().info("MyPlugin is setting up!");
    }

    @Override
    public void start() {
        getLogger().info("MyPlugin has started!");
    }

    @Override
    public void shutdown() {
        getLogger().info("MyPlugin is shutting down!");
    }
}
```

**manifest.json:**

```json
{
  "Group": "Example",
  "Name": "MyPlugin",
  "Version": "1.0.0",
  "Description": "My first Hytale plugin",
  "Main": "com.example.myplugin.MyPlugin",
  "ServerVersion": "*"
}
```

### Building and Deploying

1. Build your plugin JAR:
   - Maven: `mvn clean package`
   - Gradle: `./gradlew build`

2. Copy the resulting JAR to your server's `mods` directory.

3. Start the server:
   ```bash
   java -jar HytaleServer.jar --mods ./mods
   ```

---

## Plugin Structure

### Base Class

All plugins must extend `JavaPlugin`:

```java
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;

public final class MyPlugin extends JavaPlugin {
    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }
}
```

The constructor receives a `JavaPluginInit` object that must be passed to the superclass.

### JAR Layout

Your plugin JAR must include:

```
MyPlugin.jar
├── manifest.json              # Required: Plugin metadata at JAR root
├── com/
│   └── example/
│       └── myplugin/
│           └── MyPlugin.class # Your compiled plugin class
└── assets/                    # Optional: Asset pack if IncludesAssetPack=true
```

### manifest.json Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Group` | string | Yes | Plugin group/namespace (e.g., "YourStudio") |
| `Name` | string | Yes | Plugin name (unique within group) |
| `Version` | string | Yes | Semantic version (e.g., "1.0.0") |
| `Description` | string | No | Short description |
| `Authors` | array | No | List of author objects |
| `Website` | string | No | Plugin website URL |
| `Main` | string | Yes | Fully qualified main class name |
| `ServerVersion` | string | Yes | Server version range ("*" for any) |
| `Dependencies` | object | No | Required plugin dependencies |
| `OptionalDependencies` | object | No | Optional plugin dependencies |
| `LoadBefore` | array | No | Plugins to load after this one |
| `DisabledByDefault` | boolean | No | If true, plugin must be explicitly enabled |
| `IncludesAssetPack` | boolean | No | If true, plugin contains assets |
| `SubPlugins` | array | No | Nested plugin definitions |

**Author object:**

```json
{
  "Name": "Your Name",
  "Email": "you@example.com",
  "Url": "https://example.com"
}
```

**Complete example:**

```json
{
  "Group": "MyStudio",
  "Name": "AdvancedPlugin",
  "Version": "1.2.0",
  "Description": "An advanced Hytale plugin with dependencies",
  "Authors": [
    { "Name": "Developer", "Email": "dev@example.com" }
  ],
  "Website": "https://example.com/plugin",
  "Main": "com.mystudio.advanced.AdvancedPlugin",
  "ServerVersion": "*",
  "Dependencies": {
    "Hytale:EntityModule": "*",
    "Hytale:BlockStateModule": "*"
  },
  "OptionalDependencies": {
    "Hytale:NPC": ">=1.0.0"
  },
  "IncludesAssetPack": false
}
```

### Plugin Identifier

Plugins are identified by `Group:Name` format (e.g., `MyStudio:AdvancedPlugin`). This is used in dependency declarations.

---

## Lifecycle Methods

Plugins progress through these states:

```
NONE -> SETUP -> START -> ENABLED
                            |
                            v
                      DISABLED or SHUTDOWN
```

| State | Description |
|-------|-------------|
| `NONE` | Initial state before loading |
| `SETUP` | During `setup()` execution |
| `START` | During `start()` execution |
| `ENABLED` | Plugin fully active |
| `DISABLED` | Plugin disabled (can be re-enabled) |
| `SHUTDOWN` | Plugin shutting down |

### setup()

Called first. Use for:
- Registering configurations (must be done here)
- Registering commands
- Registering event listeners
- Initializing data structures

```java
@Override
public void setup() {
    // Register config BEFORE setup completes
    Config<MyConfig> config = withConfig("settings", MyConfig.CODEC);

    // Register commands
    getCommandRegistry().registerCommand(new MyCommand());

    // Register events
    getEventRegistry().register(PlayerConnectEvent.class, this::onPlayerConnect);

    getLogger().info("Setup complete");
}
```

### start()

Called after setup. Use for:
- Starting gameplay systems
- Loading saved data
- Initializing runtime state

```java
@Override
public void start() {
    // Start your systems
    getLogger().info("Plugin started");
}
```

### shutdown()

Called when plugin is being disabled. Use for:
- Saving data
- Stopping tasks
- Releasing resources

```java
@Override
public void shutdown() {
    // Save data, stop tasks
    getLogger().info("Plugin shutting down");
}
```

### cleanup()

Final teardown after shutdown. Use sparingly for final cleanup.

---

## Event System

The event system allows plugins to respond to server events.

### Registering Events

```java
// In setup()
getEventRegistry().register(PlayerConnectEvent.class, this::onPlayerConnect);

// Handler method
private void onPlayerConnect(PlayerConnectEvent event) {
    getLogger().info("Player connected: " + event.getPlayer().getName());
}
```

### With Lambda

```java
getEventRegistry().register(PlayerChatEvent.class, event -> {
    String message = event.getMessage();
    getLogger().info("Chat: " + message);
});
```

### Event Priority

Priority levels (in order of execution):

| Priority | Description |
|----------|-------------|
| `FIRST` | Runs first, before all others |
| `EARLY` | Runs early in the chain |
| `NORMAL` | Default priority |
| `LATE` | Runs after normal handlers |
| `LAST` | Runs last, after all others |

```java
import com.hypixel.hytale.event.EventPriority;

// Register with priority
getEventRegistry().register(EventPriority.EARLY, PlayerConnectEvent.class, this::onConnect);
getEventRegistry().register(EventPriority.LAST, PlayerConnectEvent.class, this::onConnectLast);
```

### Async Events

For events that support async handling:

```java
getEventRegistry().registerAsync(SomeAsyncEvent.class, future -> {
    return future.thenApply(event -> {
        // Process asynchronously
        return event;
    });
});
```

### Unregistering Events

```java
EventRegistration registration = getEventRegistry().register(
    PlayerConnectEvent.class, this::onConnect
);

// Later, to unregister:
registration.unregister();
```

### Common Events Reference

All events are in the `com.hypixel.hytale` package prefix.

**Player Events** (`server.core.event.events.player`):

| Event Class | Description |
|------------|-------------|
| `PlayerConnectEvent` | Player connecting to server |
| `PlayerDisconnectEvent` | Player disconnecting |
| `PlayerChatEvent` | Player sends chat message |
| `PlayerReadyEvent` | Player fully loaded and ready |
| `PlayerInteractEvent` | Player interaction with world |
| `PlayerCraftEvent` | Player crafting item |
| `PlayerMouseButtonEvent` | Mouse button input |
| `PlayerMouseMotionEvent` | Mouse motion input |
| `PlayerSetupConnectEvent` | Player connection setup phase |
| `PlayerSetupDisconnectEvent` | Player disconnect during setup |

**Block/Entity Events** (`server.core.event.events.ecs`):

| Event Class | Description |
|------------|-------------|
| `BreakBlockEvent` | Block being broken |
| `PlaceBlockEvent` | Block being placed |
| `DamageBlockEvent` | Block taking damage |
| `UseBlockEvent` | Block being used/activated (has Pre/Post) |
| `CraftRecipeEvent` | Recipe being crafted (has Pre/Post) |
| `DropItemEvent` | Item being dropped |
| `ChangeGameModeEvent` | Game mode change |
| `DiscoverZoneEvent` | Zone discovery |
| `InteractivelyPickupItemEvent` | Interactive item pickup |
| `SwitchActiveSlotEvent` | Hotbar slot switch |

**Entity Events** (`server.core.event.events.entity`):

| Event Class | Description |
|------------|-------------|
| `EntityEvent` | Base entity event |
| `EntityRemoveEvent` | Entity being removed |
| `LivingEntityInventoryChangeEvent` | Inventory change |
| `LivingEntityUseBlockEvent` | Entity uses block |

**Server Events** (`server.core.event.events`):

| Event Class | Description |
|------------|-------------|
| `BootEvent` | Server boot complete |
| `ShutdownEvent` | Server shutting down |
| `PrepareUniverseEvent` | Universe preparation |

**Asset Events** (`assetstore.event`):

| Event Class | Description |
|------------|-------------|
| `LoadedAssetsEvent` | Assets finished loading |

---

## Command System

### Creating a Command

Extend `AbstractCommand`:

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;

public class HelloCommand extends AbstractCommand {

    public HelloCommand() {
        super("hello", "Says hello to the player");
        // Or with confirmation requirement:
        // super("hello", "Says hello to the player", false);
    }

    // Override execute method to implement command logic
    // Refer to built-in commands for patterns (e.g., MacroCommandPlugin)
}
```

**AbstractCommand constructor signatures:**
- `AbstractCommand(String name, String description)`
- `AbstractCommand(String name, String description, boolean requiresConfirmation)`

### Registering Commands

```java
@Override
public void setup() {
    getCommandRegistry().registerCommand(new HelloCommand());
}
```

### Unregistering Commands

```java
CommandRegistration registration = getCommandRegistry().registerCommand(new HelloCommand());

// Later:
registration.unregister();
```

---

## Configuration

### Creating a Config

Configs use the `BuilderCodec` system for typed serialization:

```java
import com.hypixel.hytale.codec.builder.BuilderCodec;
import com.hypixel.hytale.server.core.util.Config;

public class MyConfig {
    public static final BuilderCodec<MyConfig> CODEC = // define codec

    private String greeting = "Hello";
    private int maxPlayers = 100;

    // getters and setters
}
```

### Registering Config

**Important:** Must be called in `setup()` before it completes.

```java
@Override
public void setup() {
    Config<MyConfig> config = withConfig("settings", MyConfig.CODEC);
    // or with custom name:
    Config<MyConfig> config = withConfig("custom-name", MyConfig.CODEC);
}
```

### Config Location

Configs are stored in the plugin's data directory. Based on current public plugins, the path is:
```
<server>/mods/<Group>_<Name>/settings.json
```

### Accessing Config

```java
private Config<MyConfig> config;

@Override
public void setup() {
    config = withConfig("settings", MyConfig.CODEC);
}

@Override
public void start() {
    MyConfig settings = config.get();
    String greeting = settings.getGreeting();
}
```

---

## Registries Reference

All registries are accessed via getter methods on your plugin:

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `getLogger()` | `HytaleLogger` | Logging with levels (INFO, WARNING, SEVERE) |
| `getManifest()` | `PluginManifest` | Access plugin metadata |
| `getIdentifier()` | `PluginIdentifier` | Get plugin Group:Name |
| `getDataDirectory()` | `Path` | Plugin-specific data folder |
| `getEventRegistry()` | `EventRegistry` | Register event handlers |
| `getCommandRegistry()` | `CommandRegistry` | Register commands |
| `getBlockStateRegistry()` | `BlockStateRegistry` | Block state metadata |
| `getEntityRegistry()` | `EntityRegistry` | Entity type registration |
| `getTaskRegistry()` | `TaskRegistry` | Scheduled task registration |
| `getAssetRegistry()` | `AssetRegistry` | Asset registration |
| `getClientFeatureRegistry()` | `ClientFeatureRegistry` | Client feature flags |

### Logger Usage

```java
getLogger().info("Information message");
getLogger().at(Level.WARNING).log("Warning message");
getLogger().at(Level.SEVERE).withCause(exception).log("Error: %s", message);
```

### Task Registry

For scheduling tasks:

```java
CompletableFuture<Void> task = CompletableFuture.runAsync(() -> {
    // async work
});
getTaskRegistry().registerTask(task);
```

---

## Asset Packs

If your plugin includes custom assets:

1. Set `"IncludesAssetPack": true` in manifest.json

2. Include assets in your JAR under an assets directory

3. The server will automatically register the asset pack on plugin load

### Accessing Asset Packs

```java
// In JavaPlugin
AssetPack pack = getAssetPack("packName");
```

---

## Dependencies

### Declaring Dependencies

In `manifest.json`:

```json
{
  "Dependencies": {
    "Hytale:EntityModule": "*",
    "Hytale:BlockStateModule": ">=1.0.0"
  },
  "OptionalDependencies": {
    "Hytale:NPC": "*"
  }
}
```

### Available Built-in Modules

The server includes built-in modules you can depend on. These are the actual dependency identifiers used in `manifests.json`:

**Core Modules:**

| Module | Description |
|--------|-------------|
| `Hytale:AssetModule` | Asset management and loading |
| `Hytale:BlockModule` | Block system |
| `Hytale:BlockStateModule` | Block state management |
| `Hytale:BlockTypeModule` | Block type definitions |
| `Hytale:DamageModule` | Damage and combat system |
| `Hytale:EntityModule` | Entity system |
| `Hytale:EntityStatsModule` | Entity statistics |
| `Hytale:I18nModule` | Internationalization/localization |
| `Hytale:InteractionModule` | Player/world interaction |
| `Hytale:ItemModule` | Item system |
| `Hytale:LegacyModule` | Legacy compatibility |

**Built-in Plugins (48 total):**

| Plugin | Description |
|--------|-------------|
| `Hytale:Ambience` | Ambient sounds and effects |
| `Hytale:AssetEditor` | In-game asset editing |
| `Hytale:Beds` | Bed mechanics |
| `Hytale:BlockPhysics` | Block physics (gravity, etc.) |
| `Hytale:BlockSpawner` | Block-based spawning |
| `Hytale:BlockTick` | Block tick updates |
| `Hytale:BuilderTools` | Building tools |
| `Hytale:Camera` | Camera controls |
| `Hytale:CommandMacro` | Command macros |
| `Hytale:Crafting` | Crafting system |
| `Hytale:CreativeHub` | Creative mode hub |
| `Hytale:CrouchSlide` | Crouch sliding mechanics |
| `Hytale:Deployables` | Deployable objects |
| `Hytale:Farming` | Farming mechanics |
| `Hytale:Flock` | NPC flocking behavior |
| `Hytale:Fluid` | Fluid/liquid physics |
| `Hytale:HytaleGenerator` | World generation |
| `Hytale:Instances` | World instances |
| `Hytale:LANDiscovery` | LAN server discovery |
| `Hytale:Mantling` | Climbing/mantling |
| `Hytale:Memories` | NPC memories |
| `Hytale:Model` | Model system |
| `Hytale:Mounts` | Mount system |
| `Hytale:NPC` | NPC system (includes Spawning, Flock) |
| `Hytale:NPCCombatActionEvaluator` | NPC combat AI |
| `Hytale:NPCEditor` | NPC editing tools |
| `Hytale:NPCObjectives` | NPC quest objectives |
| `Hytale:NPCReputation` | NPC reputation |
| `Hytale:NPCShop` | NPC shops |
| `Hytale:Objectives` | Quest/objective system |
| `Hytale:ObjectiveReputation` | Objective-based reputation |
| `Hytale:ObjectiveShop` | Objective-based shops |
| `Hytale:Parkour` | Parkour mechanics |
| `Hytale:Path` | Pathfinding |
| `Hytale:Portals` | Portal system |
| `Hytale:Reputation` | Reputation system |
| `Hytale:SafetyRoll` | Safety roll mechanics |
| `Hytale:Shop` | Shop system |
| `Hytale:ShopReputation` | Shop reputation |
| `Hytale:Spawning` | Entity spawning |
| `Hytale:SprintForce` | Sprint mechanics |
| `Hytale:Stash` | Storage/stash system |
| `Hytale:TagSet` | Entity tagging |
| `Hytale:Teleport` | Teleportation |
| `Hytale:Teleporter` | Teleporter blocks |
| `Hytale:Weather` | Weather system |
| `Hytale:WorldGen` | World generation hooks |
| `Hytale:WorldLocationCondition` | Location-based conditions |

---

## Plugin-to-Plugin Integration (Example Pattern)

Public plugins often fetch another plugin at runtime to call its API. For example, the
`Nitrado:Query` plugin retrieves `Nitrado:WebServer` and registers a servlet. This pattern relies
on declaring a dependency in `manifest.json` and then looking up the plugin by identifier:

```java
import com.hypixel.hytale.common.plugin.PluginIdentifier;
import com.hypixel.hytale.server.core.plugin.PluginManager;

var plugin = PluginManager.get().getPlugin(new PluginIdentifier("Nitrado", "WebServer"));
if (plugin instanceof WebServerPlugin webServer) {
    webServer.addServlet(this, "", new QueryServlet());
}
```

See https://github.com/nitrado/hytale-plugin-webserver and
https://github.com/nitrado/hytale-plugin-query for real-world usage.

## Complete Examples

### Example 1: Welcome Plugin

A simple plugin that welcomes players:

**WelcomePlugin.java:**

```java
package com.example.welcome;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.event.events.player.PlayerConnectEvent;
import com.hypixel.hytale.server.core.event.events.player.PlayerDisconnectEvent;
import javax.annotation.Nonnull;

public final class WelcomePlugin extends JavaPlugin {

    public WelcomePlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    public void setup() {
        getEventRegistry().register(PlayerConnectEvent.class, this::onConnect);
        getEventRegistry().register(PlayerDisconnectEvent.class, this::onDisconnect);
        getLogger().info("WelcomePlugin setup complete");
    }

    @Override
    public void start() {
        getLogger().info("WelcomePlugin started");
    }

    @Override
    public void shutdown() {
        getLogger().info("WelcomePlugin shutdown");
    }

    private void onConnect(PlayerConnectEvent event) {
        getLogger().info("Welcome, " + event.getPlayer().getName() + "!");
    }

    private void onDisconnect(PlayerDisconnectEvent event) {
        getLogger().info("Goodbye, " + event.getPlayer().getName() + "!");
    }
}
```

**manifest.json:**

```json
{
  "Group": "Example",
  "Name": "WelcomePlugin",
  "Version": "1.0.0",
  "Description": "Welcomes players when they join",
  "Main": "com.example.welcome.WelcomePlugin",
  "ServerVersion": "*"
}
```

### Example 2: Block Logger Plugin

Logs block breaks and placements:

**BlockLoggerPlugin.java:**

```java
package com.example.blocklogger;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.event.events.ecs.PlaceBlockEvent;
import javax.annotation.Nonnull;

public final class BlockLoggerPlugin extends JavaPlugin {

    public BlockLoggerPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    public void setup() {
        getEventRegistry().register(BreakBlockEvent.class, this::onBreak);
        getEventRegistry().register(PlaceBlockEvent.class, this::onPlace);
    }

    @Override
    public void start() {
        getLogger().info("BlockLogger is now monitoring block activity");
    }

    @Override
    public void shutdown() {
        getLogger().info("BlockLogger stopped");
    }

    private void onBreak(BreakBlockEvent event) {
        getLogger().info("Block broken at " + event.getPosition());
    }

    private void onPlace(PlaceBlockEvent event) {
        getLogger().info("Block placed at " + event.getPosition());
    }
}
```

**manifest.json:**

```json
{
  "Group": "Example",
  "Name": "BlockLogger",
  "Version": "1.0.0",
  "Description": "Logs block breaks and placements",
  "Main": "com.example.blocklogger.BlockLoggerPlugin",
  "ServerVersion": "*",
  "Dependencies": {
    "Hytale:BlockStateModule": "*"
  }
}
```

---

## Troubleshooting

### Plugin Not Loading

1. Ensure `manifest.json` is at the JAR root (not in a subdirectory)
2. Verify `Main` class name matches your actual class
3. Check `ServerVersion` compatibility
4. Verify all dependencies are available

### Dependency Errors

- Check dependency names use `Group:Name` format
- Verify version ranges are valid semver

### Runtime Errors

- Check logs for stack traces
- Ensure all registrations happen in `setup()`
- Configs must be registered before setup completes

### Server Flags

Useful flags for plugin development:

```bash
# Specify mods directory
java -jar HytaleServer.jar --mods ./mods

# Multiple mod directories
java -jar HytaleServer.jar --mods ./mods --mods ./dev-mods

# Early plugins (unstable, requires acknowledgment)
java -jar HytaleServer.jar --accept-early-plugins --early-plugins ./early
```

---

## Next Steps

For more detailed API information, consider:

1. Decompiling specific classes with CFR or Vineflower
2. Examining built-in plugin implementations in the JAR
3. Testing behavior on a running server
4. Reviewing public plugins for real-world patterns (e.g., `Nitrado:WebServer`, `Nitrado:Query`)

This documentation will be updated as more API details are discovered or official documentation is released.
