# Hytale Server Plugin API Guide (Reverse-Engineered)

This guide is based on inspecting `HytaleServer.jar` (build `2026.01.13-dcad8778f`). It is not an official API reference; verify against the actual server build you are targeting.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Plugin Structure](#plugin-structure)
3. [Lifecycle Methods](#lifecycle-methods)
4. [Event System](#event-system)
5. [Component System](#component-system)
6. [Command System](#command-system)
7. [Configuration](#configuration)
8. [Registries Reference](#registries-reference)
9. [Asset Packs](#asset-packs)
10. [Dependencies](#dependencies)
11. [External Example Guides](#external-example-guides)
12. [Complete Examples](#complete-examples)

---

## Quick Start

### Requirements

- Java 21 or higher (server manifest `Java-Version: 21`)
- Hytalemodding.dev setup guide recommends JDK 25+ (Build-Jdk-Spec is 25 in this JAR)
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

### Verified Lifecycle Hooks (javap)

`PluginBase` exposes lifecycle hooks and helpers that match the runtime order:

- `protected void setup()` / `protected void setup0()`
- `protected void start()` / `protected void start0()`
- `protected void shutdown()` / `protected void shutdown0(boolean)`
- `public CompletableFuture<Void> preLoad()`

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
    getEventRegistry().registerGlobal(PlayerConnectEvent.class, this::onPlayerConnect);

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
getEventRegistry().registerGlobal(PlayerConnectEvent.class, this::onPlayerConnect);

// Handler method
private void onPlayerConnect(PlayerConnectEvent event) {
    getLogger().info("Player connected: " + event.getPlayer().getName());
}
```

### With Lambda

```java
getEventRegistry().registerGlobal(PlayerChatEvent.class, event -> {
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
getEventRegistry().registerGlobal(EventPriority.EARLY, PlayerConnectEvent.class, this::onConnect);
getEventRegistry().registerGlobal(EventPriority.LAST, PlayerConnectEvent.class, this::onConnectLast);
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
EventRegistration registration = getEventRegistry().registerGlobal(
    PlayerConnectEvent.class, this::onConnect
);

// Later, to unregister:
registration.unregister();
```

### Registering Global Events (Site Docs)

The hytalemodding.dev event guide uses `registerGlobal(...)` in examples:

```java
getEventRegistry().registerGlobal(PlayerReadyEvent.class, ExampleEvent::onPlayerReady);
```

### ECS Event Systems (Site Docs)

The hytalemodding.dev events guide notes that ECS events should be handled by an
`EntityEventSystem` registered via `getEntityStoreRegistry()`.

```java
getEntityStoreRegistry().registerSystem(new ExampleCancelCraft());
```

`EntityEventSystem` exists in `com.hypixel.hytale.component.system`.

### ECS Event System Template (JAR)

Use ECS systems for block/crafting events like `BreakBlockEvent`,
`PlaceBlockEvent`, `UseBlockEvent.Pre`, and `CraftRecipeEvent.Pre`.

```java
import com.hypixel.hytale.component.Archetype;
import com.hypixel.hytale.component.ArchetypeChunk;
import com.hypixel.hytale.component.CommandBuffer;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.component.query.Query;
import com.hypixel.hytale.component.system.EntityEventSystem;
import com.hypixel.hytale.server.core.asset.type.item.config.CraftingRecipe;
import com.hypixel.hytale.server.core.event.events.ecs.CraftRecipeEvent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;

public final class CancelRecipeSystem
    extends EntityEventSystem<EntityStore, CraftRecipeEvent.Pre> {

    public CancelRecipeSystem() {
        super(CraftRecipeEvent.Pre.class);
    }

    @Override
    public void handle(int index,
                       ArchetypeChunk<EntityStore> archetypeChunk,
                       Store<EntityStore> store,
                       CommandBuffer<EntityStore> commandBuffer,
                       CraftRecipeEvent.Pre event) {
        CraftingRecipe recipe = event.getCraftedRecipe();
        // TODO: inspect recipe input and cancel when needed.
        if (shouldCancel(recipe)) {
            event.setCancelled(true);
        }
    }

    @Override
    public Query<EntityStore> getQuery() {
        return Archetype.empty();
    }

    private boolean shouldCancel(CraftingRecipe recipe) {
        return false;
    }
}
```

Register in your plugin:

```java
getEntityStoreRegistry().registerSystem(new CancelRecipeSystem());
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

### Observed in Local Mods (Decompiled)

Local decompiled mods show these concrete `EventRegistry` usages:

- `registerGlobal(PlayerConnectEvent.class, ...)`
- `registerGlobal(PlayerChatEvent.class, ...)`
- `registerGlobal(PlayerReadyEvent.class, ...)`
- `register(LoadedAssetsEvent.class, Item.class, ...)`
- `register(LoadedAssetsEvent.class, CraftingRecipe.class, ...)`
- `register(LoadedAssetsEvent.class, ModelAsset.class, ...)`
- `register(RemovedAssetsEvent.class, CraftingRecipe.class, ...)`

Some mods wrap event handling in their own callback layers (for example, `PlayerTickCallback` and `EntityBreakBlockCallback`) rather than directly using `EventRegistry`.

**Deprecations noted on hytalemodding.dev (see `https://hytalemodding.dev/en/docs/server/events`):**

- `LivingEntityUseBlockEvent`
- `PlayerCraftEvent`
- `PlayerInteractEvent`
- `PrepareUniverseEvent`

### Extended Event List (Site Reference)

From `https://hytalemodding.dev/en/docs/server/events`:

```
## IEvent
- AddPlayerToWorldEvent
- AllNPCsLoadedEvent
- AllWorldsLoadedEvent
- AssetMonitorEvent
  - AssetStoreMonitorEvent
  - CommonAssetMonitorEvent
- AssetPackRegisterEvent
- AssetPackUnregisterEvent
- AssetStoreEvent
  - RegisterAssetStoreEvent
  - RemoveAssetStoreEvent
- AssetsEvent
  - GenerateAssetsEvent
  - LoadedAssetsEvent
  - RemovedAssetsEvent
- BootEvent
- ChunkEvent
  - ChunkPreLoadProcessEvent
- DrainPlayerFromWorldEvent
- EditorClientEvent
  - AssetEditorActivateButtonEvent
  - AssetEditorAssetCreatedEvent
  - AssetEditorClientDisconnectEvent
  - AssetEditorSelectAssetEvent
  - AssetEditorUpdateWeatherPreviewLockEvent
- EntityEvent
  - EntityRemoveEvent
  - LivingEntityInventoryChangeEvent
- GenerateDefaultLanguageEvent
- GenerateSchemaEvent
- GenerateServerStateEvent
- ItemContainerChangeEvent
- ~~LivingEntityUseBlockEvent~~ DEPRECATED
- LoadAssetEvent
- LoadedNPCEvent
- MessagesUpdated
- PlayerConnectEvent
- PlayerEvent
  - ~~PlayerCraftEvent~~ DEPRECATED
  - ~~PlayerInteractEvent~~ DEPRECATED
  - PlayerMouseButtonEvent
  - PlayerMouseMotionEvent
  - PlayerReadyEvent
  - RemovePlayerFromWorldEvent
- PlayerRefEvent
  - PlayerDisconnectEvent
- PlayerSetupConnectEvent
- PlayerSetupDisconnectEvent
- PluginEvent
  - PluginSetupEvent
- ~~PrepareUniverseEvent~~ DEPRECATED
- ShutdownEvent
- SingleplayerRequestAccessEvent
- TreasureChestOpeningEvent
- WindowCloseEvent
- WorldEvent
  - AddWorldEvent
  - RemoveWorldEvent
  - StartWorldEvent
- WorldPathChangedEvent
## IAsyncEvent
- AssetEditorFetchAutoCompleteDataEvent
- AssetEditorRequestDataSetEvent
- PlayerChatEvent
- SendCommonAssetsEvent
## EcsEvent
- CancellableEcsEvent
  - BreakBlockEvent
  - ChangeGameModeEvent
  - ChunkSaveEvent
  - ChunkUnloadEvent
  - CraftRecipeEvent
  - Damage
  - DamageBlockEvent
  - DropItemEvent
  - InteractivelyPickupItemEvent
  - PlaceBlockEvent
  - PrefabPasteEvent
  - SwitchActiveSlotEvent
- DiscoverInstanceEvent
  - Display
- DiscoverZoneEvent
  - Display
- MoonPhaseChangeEvent
- UseBlockEvent
  - Post
  - Pre
```

---

### Decompiled Event Details (Selected)

These are pulled from decompiled classes in the server JAR:

- `PlayerChatEvent` implements `IAsyncEvent` and `ICancellable`.
  - Fields: `sender` (`PlayerRef`), `targets` (`List`), `content` (`String`).
  - Formatter: `PlayerChatEvent.Formatter` + `DEFAULT_FORMATTER` uses translation key
    `server.chat.playerMessage` with `username` and `message` params.
- `PlayerConnectEvent` stores `Holder`, `PlayerRef`, and optional `World`.
  - `getPlayer()` is deprecated; prefer `getPlayerRef()` / `getHolder()`.
- `PlayerDisconnectEvent` exposes `getDisconnectReason()` from `PacketHandler`.
- `PlayerReadyEvent` includes a `readyId` integer.
- `BreakBlockEvent` includes `itemInHand`, `targetBlock` (`Vector3i`), `blockType`.
- `PlaceBlockEvent` includes `itemInHand`, `targetBlock`, and `rotation` (`RotationTuple`).
- `UseBlockEvent` includes `interactionType`, `context`, `targetBlock`, `blockType`.
  - `UseBlockEvent.Pre` implements `ICancellableEcsEvent`; `Post` is non-cancellable.
- `CraftRecipeEvent` includes `craftedRecipe` and `quantity` with `Pre`/`Post` subclasses.
- `DropItemEvent.PlayerRequest` includes `inventorySectionId` and `slotId`.
- `DropItemEvent.Drop` includes `itemStack` and `throwSpeed`.
- `SwitchActiveSlotEvent` includes `inventorySectionId`, `previousSlot`, `newSlot`, `serverRequest`.

---

## Event Registry Notes (javap)

`com.hypixel.hytale.event.EventRegistry` supports additional registration patterns beyond the basic examples:

- `registerGlobal(...)` and `registerAsyncGlobal(...)` for global handlers
- `registerUnhandled(...)` and `registerAsyncUnhandled(...)` for unhandled events
- Keyed events via `register(Class, KeyType, Consumer)` and async equivalents

## Component System

The ECS lives under `com.hypixel.hytale.component` in the server JAR. Core types you will see in signatures:

- `Store<ECS_TYPE>` and `Ref<ECS_TYPE>` for component storage and references
- `Holder<ECS_TYPE>` for grouped component data
- `Component`, `ComponentType`, `ComponentRegistry`, and `ComponentRegistryProxy`
- `ReadWriteQuery` and helpers under `com.hypixel.hytale.component.query`
- System base classes under `com.hypixel.hytale.component.system` (`System`, `StoreSystem`, `RefSystem`, `HolderSystem`, `RefChangeSystem`)

Entity components are stored in `com.hypixel.hytale.server.core.universe.world.storage.EntityStore` and typically accessed via `Store<EntityStore>` and `Ref<EntityStore>`. Plugin registries expose `getEntityStoreRegistry()` and `getChunkStoreRegistry()` for component systems (see Registries Reference).

Key APIs observed via `javap`:

- `Store` entity helpers: `addEntity(...)`, `removeEntity(...)`, `addComponent(...)`,
  `getComponent(...)`, `invoke(EcsEvent)`, `forEachChunk(...)`, `tick(float)`.
- `Ref` helpers: `getStore()`, `getIndex()`, `validate()`, `isValid()`.
- `EntityStore` helpers: `getStore()`, `getRefFromUUID(UUID)`, `getRefFromNetworkId(int)`,
  `takeNextNetworkId()`, `getWorld()`.

`PlayerRef` is a `Component<EntityStore>` and `IMessageReceiver` with player identity and
movement helpers, including `getUuid()`, `getUsername()`, `getReference()`, `getTransform()`,
`updatePosition(World, Transform, Vector3f)`, and `sendMessage(Message)`.

ECS events are handled in systems derived from:

- `com.hypixel.hytale.component.system.EntityEventSystem` with
  `handle(int, ArchetypeChunk<ECS_TYPE>, Store<ECS_TYPE>, CommandBuffer<ECS_TYPE>, EventType)`.

Practical notes from the JAR:

- `Store` component APIs use `ComponentType` (not Java classes). Many built-in components expose a `getComponentType()` static.
- `Store.ensureComponent(...)` returns void; use `ensureAndGetComponent(...)` to retrieve a component after ensuring it exists.
- `Store.addEntity(...)` expects an `Archetype` and `AddReason`; the store is supplied by the server, not constructed manually.
- `Store.assertThread()` throws if called off the owning store thread; `assertWriteProcessing()` fails if invoked from an active system.
- `PlayerRef.getComponent(...)` is deprecated; it will log and marshal to the world thread if called asynchronously.

Decompiled component details (selected):

- `TransformComponent` stores `position` (`Vector3d`), `rotation` (`Vector3f`), and a `ModelTransform` for sent state.
  - Helpers: `teleportPosition(...)`, `teleportRotation(...)`, `markChunkDirty(...)`.
- `DisplayNameComponent` stores an optional `Message` display name.
- `PlayerSkinComponent` wraps `PlayerSkin` and tracks `isNetworkOutdated`.
- `MovementStatesComponent` holds current and sent `MovementStates`.
- `UUIDComponent` supports `generateVersion3UUID()` and `randomUUID()`.
- `UniqueItemUsagesComponent` tracks a set of used unique item IDs.
- `ItemComponent` exposes pickup/merge delays and drop helpers.
  - Constants: `DEFAULT_PICKUP_DELAY = 0.5`, `PICKUP_DELAY_DROPPED = 1.5`,
    `PICKUP_THROTTLE = 0.25`, `DEFAULT_MERGE_DELAY = 1.5`.
  - Helpers: `generateItemDrops(...)`, `generatePickedUpItem(...)`, `addToItemContainer(...)`.

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

### Observed in Sample Mods (Heuristic)

String scans of local `.jar` sample mods (not included in this repo) show common use of:

- `CommandRegistry` for registration
- `CommandContext` and `CommandSender` during execution
- `AbstractAsyncCommand` for off-thread command handling
- `UICommandBuilder` for UI-bound command flows
- `OptionalArg` for argument parsing

This list is based on class/string presence and does not guarantee API stability.

### UI-Triggered Command Flow (Inferred Example)

Observed in a sample admin UI mod (non-verbatim pattern):

1. Define a tool item in `Server/Item/Items/...` with an `Interactions` map that points to an interaction ID.
2. Map that interaction ID to a root interaction in `Server/Item/RootInteractions/...`.
3. Define the interaction JSON in `Server/Item/Interactions/Item/...` with a `Type` that matches a plugin handler class.
4. Provide UI layouts under `Common/UI/Custom/Pages/...` (`.ui` files) plus textures under `Common/UI/Custom/...`.
5. In plugin code, handle the interaction to open the UI and register any related commands through `CommandRegistry` or `UICommandBuilder`.

This is a practical wiring pattern: data files expose the item and interaction, while the plugin handles UI and behavior.

### Alternative Command Bases (Site Docs)

hytalemodding.dev documents `AbstractPlayerCommand` under
`com.hypixel.hytale.server.core.command.system.basecommands`. This extends
`AbstractAsyncCommand`, meaning these commands run off the main server thread.

The documented execute signature includes a player-bound context:

```java
protected void execute(CommandContext commandContext,
                       Store<EntityStore> store,
                       Ref<EntityStore> ref,
                       PlayerRef playerRef,
                       World world) {
    // ...
}
```

**AbstractCommand constructor signatures:**
- `AbstractCommand(String name, String description)`
- `AbstractCommand(String name, String description, boolean requiresConfirmation)`
- `AbstractCommand(String name)`

**Permission helpers on AbstractCommand (javap):**
- `setPermissionGroup(GameMode)` for gamemode-gated commands
- `setPermissionGroups(String...)` for named permission groups (for example, `OP`)

**AbstractCommand execute signature (javap):**

```java
protected abstract CompletableFuture<Void> execute(CommandContext context);
```

### Arguments and Suggestions (JAR Example)

This example uses the JAR argument system (`RequiredArg`, `OptionalArg`, `ArgTypes`) and the built-in suggestion API.
Arguments must be registered on the command via `withRequiredArg(...)` / `withOptionalArg(...)` (or related helpers);
constructing `RequiredArg` / `OptionalArg` directly does not register them with the parser.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.OptionalArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.command.system.suggestion.SuggestionProvider;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public final class WarnCommand extends AbstractCommand {
    private final RequiredArg<String> targetArg;
    private final OptionalArg<String> reasonArg;

    public WarnCommand() {
        super("warn", "Warn a player");
        targetArg = withRequiredArg("target", "Target name", ArgTypes.STRING)
            .suggest((sender, input, argIndex, result) -> {
                result.fuzzySuggest(input, List.of("Alice", "Bob"), s -> s);
            });
        reasonArg = withOptionalArg("reason", "Reason", ArgTypes.STRING);
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext context) {
        if (!context.sender().hasPermission("example.warn")) {
            context.sendMessage(Message.raw("No permission."));
            return CompletableFuture.completedFuture(null);
        }

        String target = targetArg.get(context);
        String reason = reasonArg.provided(context) ? reasonArg.get(context) : "No reason";
        context.sendMessage(Message.raw("Warned " + target + ": " + reason));
        return CompletableFuture.completedFuture(null);
    }
}
```

### Subcommands and extra arguments (JAR notes)

- Subcommand arguments must be registered on the subcommand itself (for example,
  `withRequiredArg(...)` inside the subcommand constructor).
- If a subcommand needs to accept extra tokens for manual routing, call
  `setAllowsExtraArguments(true)` and parse `CommandContext.getInputString()`.

Pattern example (non-verbatim):

```java
public final class BwCommand extends AbstractCommand {
    public BwCommand() {
        super("bw", "BedWars root");
        addSubCommand(new SetupCommand());
    }
}

private static final class SetupCommand extends AbstractCommand {
    private SetupCommand() {
        super("setup", "Arena setup");
        setAllowsExtraArguments(true);
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext context) {
        String input = context.getInputString();
        // Parse tokens after "setup" and dispatch to your handler.
        return CompletableFuture.completedFuture(null);
    }
}
```

Helper for manual parsing (handles quoted args):

```java
private static String[] argsAfter(CommandContext context, int skipTokens) {
    List<String> tokens = tokenizeInput(context.getInputString());
    if (tokens.size() <= skipTokens) {
        return new String[0];
    }
    return tokens.subList(skipTokens, tokens.size()).toArray(new String[0]);
}

private static List<String> tokenizeInput(String input) {
    List<String> out = new ArrayList<>();
    StringBuilder cur = new StringBuilder();
    boolean inQuotes = false;
    char quote = 0;
    boolean escape = false;

    for (int i = 0; i < input.length(); i++) {
        char c = input.charAt(i);

        if (escape) {
            cur.append(c);
            escape = false;
            continue;
        }

        if (c == '\\') {
            escape = true;
            continue;
        }

        if (inQuotes) {
            if (c == quote) {
                inQuotes = false;
            } else {
                cur.append(c);
            }
            continue;
        }

        if (c == '"' || c == '\'') {
            inQuotes = true;
            quote = c;
            continue;
        }

        if (Character.isWhitespace(c)) {
            if (cur.length() > 0) {
                out.add(cur.toString());
                cur.setLength(0);
            }
            continue;
        }

        cur.append(c);
    }

    if (cur.length() > 0) {
        out.add(cur.toString());
    }

    return out;
}
```

Sender resolution: prefer `CommandContext.isPlayer()` and `CommandContext.senderAsPlayerRef()`
when you need a player sender instead of casting `CommandSender`.

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

## Observed Runtime Classes (Server Logs)

These class names appear in `serverexample/logs/*.log` stack traces and are useful anchors when searching the server JAR:

- `com.hypixel.hytale.server.core.command.system.CommandManager`
- `com.hypixel.hytale.server.core.command.system.AbstractCommand`
- `com.hypixel.hytale.server.core.command.system.basecommands.CommandBase`
- `com.hypixel.hytale.server.core.command.commands.server.auth.AuthPersistenceCommand`
- `com.hypixel.hytale.server.core.auth.ServerAuthManager`
- `com.hypixel.hytale.server.core.auth.EncryptedAuthCredentialStore`

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

`PluginBase` also exposes `withConfig(BuilderCodec)` for default-named configs.

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
| `getEntityStoreRegistry()` | `ComponentRegistryProxy<EntityStore>` | Entity store components |
| `getChunkStoreRegistry()` | `ComponentRegistryProxy<ChunkStore>` | Chunk store components |
| `getCodecRegistry(StringCodecMapCodec)` | `CodecMapRegistry` | Codec registry (string-keyed) |
| `getCodecRegistry(AssetCodecMapCodec)` | `CodecMapRegistry.Assets` | Codec registry for JSON assets |
| `getCodecRegistry(MapKeyMapCodec)` | `MapKeyMapRegistry` | Codec registry for map-keyed codecs |
| `getBasePermission()` | `String` | Base permission node for plugin |

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

## Site Notes: Server Code Insights (Summary)

Source: https://hytalemodding.dev/en/docs/established-information/server-plugin-insights

- Server source is expected to ship as shared source with no obfuscation.
- License is described as allowing Hytale-related usage (no reuse for unrelated games).
- Built-in permission system exists and supports alternative backends (e.g., database).
- Worldgen can be fully replaced by implementing `IWorldGenProvider` / `IWorldGen`.
- Chunk storage provider can be customized (e.g., empty provider or database-backed).

These notes are from a public Discord discussion and should be treated as
informational context until confirmed by official release notes.

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

See public example plugins for real-world usage patterns.

## External Example Guides

Claude-generated guides are external and not JAR-derived. Use them as conceptual scaffolding and verify API names against `serverexample/Server/HytaleServer.jar` or the decompiled repo.

Key corrections when adapting those guides:

- Replace `plugin.json` with `manifest.json` at the JAR root (see Plugin Structure).
- Use `com.hypixel.hytale.server.core.command.system.*` (`AbstractCommand`, `CommandRegistry`) instead of `com.hypixel.hytale.server.command.*`.
- Confirm ECS types with `com.hypixel.hytale.component.*` and `EntityStore` paths described in Component System.

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
        getEventRegistry().registerGlobal(PlayerConnectEvent.class, this::onConnect);
        getEventRegistry().registerGlobal(PlayerDisconnectEvent.class, this::onDisconnect);
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

import com.hypixel.hytale.component.Archetype;
import com.hypixel.hytale.component.ArchetypeChunk;
import com.hypixel.hytale.component.CommandBuffer;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.component.query.Query;
import com.hypixel.hytale.component.system.EntityEventSystem;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.event.events.ecs.PlaceBlockEvent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import javax.annotation.Nonnull;

public final class BlockLoggerPlugin extends JavaPlugin {

    public BlockLoggerPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    public void setup() {
        getEntityStoreRegistry().registerSystem(new BreakBlockLogger());
        getEntityStoreRegistry().registerSystem(new PlaceBlockLogger());
    }

    @Override
    public void start() {
        getLogger().info("BlockLogger is now monitoring block activity");
    }

    @Override
    public void shutdown() {
        getLogger().info("BlockLogger stopped");
    }

    private final class BreakBlockLogger extends EntityEventSystem<EntityStore, BreakBlockEvent> {
        private BreakBlockLogger() {
            super(BreakBlockEvent.class);
        }

        @Override
        public void handle(int index,
                           ArchetypeChunk<EntityStore> archetypeChunk,
                           Store<EntityStore> store,
                           CommandBuffer<EntityStore> commandBuffer,
                           BreakBlockEvent event) {
            getLogger().info("Block broken at " + event.getTargetBlock());
        }

        @Override
        public Query<EntityStore> getQuery() {
            return Archetype.empty();
        }
    }

    private final class PlaceBlockLogger extends EntityEventSystem<EntityStore, PlaceBlockEvent> {
        private PlaceBlockLogger() {
            super(PlaceBlockEvent.class);
        }

        @Override
        public void handle(int index,
                           ArchetypeChunk<EntityStore> archetypeChunk,
                           Store<EntityStore> store,
                           CommandBuffer<EntityStore> commandBuffer,
                           PlaceBlockEvent event) {
            getLogger().info("Block placed at " + event.getTargetBlock());
        }

        @Override
        public Query<EntityStore> getQuery() {
            return Archetype.empty();
        }
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
5. Reading the hytalemodding.dev plugin guides:
   - https://hytalemodding.dev/en/docs/guides/plugin/creating-commands
   - https://hytalemodding.dev/en/docs/guides/plugin/creating-events
   - https://hytalemodding.dev/en/docs/guides/plugin/permission-management
   - https://hytalemodding.dev/en/docs/guides/plugin/inventory-management
   - https://hytalemodding.dev/en/docs/guides/plugin/spawning-entities
   - https://hytalemodding.dev/en/docs/guides/plugin/player-input-guide

This documentation will be updated as more API details are discovered or official documentation is released.
