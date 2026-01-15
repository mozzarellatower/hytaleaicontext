# HytaleServer.jar Dissection Notes (2026.01.13-dcad8778f)

These notes are based on inspecting the JAR contents (class/package names, embedded manifests).
No decompilation was used, so behavior is inferred from naming. Treat this as a map of what exists,
not a full API reference.

## Build Snapshot

- Main class: `com.hypixel.hytale.Main`
- Alternate entry: `com.hypixel.hytale.LateMain`
- Version: `2026.01.13-dcad8778f`
- Java: 21 (build JDK spec 25)

## Plugin System (Observed Classes)

Core plugin types and loaders:

- `com.hypixel.hytale.server.core.plugin.JavaPlugin`
- `com.hypixel.hytale.server.core.plugin.JavaPluginInit`
- `com.hypixel.hytale.server.core.plugin.PluginManager`
- `com.hypixel.hytale.server.core.plugin.PluginClassLoader`
- `com.hypixel.hytale.server.core.plugin.PluginState`
- `com.hypixel.hytale.common.plugin.PluginManifest`
- `com.hypixel.hytale.common.plugin.PluginIdentifier`
- `com.hypixel.hytale.server.core.plugin.event.PluginSetupEvent`

Built-in plugin management commands (from class names):

- `PluginCommand$PluginLoadCommand`
- `PluginCommand$PluginUnloadCommand`
- `PluginCommand$PluginReloadCommand`
- `PluginCommand$PluginListCommand`

Early plugin loader (separate system):

- `com.hypixel.hytale.plugin.early.EarlyPluginLoader`
- `com.hypixel.hytale.plugin.early.TransformingClassLoader`

## Command System (Observed Classes)

The command system appears to be a typed argument framework:

- `com.hypixel.hytale.server.core.command.system.AbstractCommand`
- `com.hypixel.hytale.server.core.command.system.CommandRegistry`
- `...arguments.system.RequiredArg`, `OptionalArg`, `FlagArg`, `Argument`
- `...arguments.types.ArgTypes` and multiple `ArgumentType` classes

Built-in command families (selected examples):

- Access control: `BanCommand`, `UnbanCommand`, `Whitelist*Command`
- Permissions: `PermCommand`, `PermGroupCommand`, `PermUserCommand`
- Worldgen: `WorldGenCommand`, `WorldGenReloadCommand`, `WorldGenBenchmarkCommand`
- Prefabs: `PrefabSpawnerCommand`, `PrefabSpawnerSetCommand`, `PrefabSpawnerWeightCommand`

## Event System (Observed Classes)

Player events:

- `PlayerConnectEvent`, `PlayerDisconnectEvent`, `PlayerReadyEvent`
- `PlayerChatEvent`, `PlayerInteractEvent`
- `AddPlayerToWorldEvent`, `DrainPlayerFromWorldEvent`
- `PlayerSetupConnectEvent`, `PlayerSetupDisconnectEvent`

Block/entity events:

- `BreakBlockEvent`, `PlaceBlockEvent`, `UseBlockEvent` (Pre/Post)
- `DamageBlockEvent`, `DropItemEvent`, `CraftRecipeEvent` (Pre/Post)
- `EntityRemoveEvent`, `LivingEntityUseBlockEvent`

Permissions events:

- `PlayerPermissionChangeEvent`
- `PlayerGroupEvent`
- `GroupPermissionChangeEvent`

Server lifecycle events:

- `BootEvent`, `PrepareUniverseEvent`, `ShutdownEvent`

Asset events:

- `assetstore.event.LoadedAssetsEvent`

## Permissions and Access Control

Permissions appear to be a first-class module with events and commands:

- `PermissionsModule`, `HytalePermissionsProvider`
- `HytalePermissions` and `PermissionHolder`
- Commands for ops, groups, and users (see command section)

## Prefabs and Structure Handling

Prefab systems and events:

- `PrefabStore`, `PrefabEntry`, `PrefabRotation`
- `PrefabPasteEvent`, `PrefabPlaceEntityEvent`
- Selection buffers and serializers: `BinaryPrefabBufferCodec`, `PrefabBuffer`
- Prefab spawner module: `PrefabSpawnerModule`

This strongly suggests a stable internal pipeline for saving, loading, and pasting structures.

## Assets and Asset Packs

Asset systems are extensive:

- `assetstore.AssetStore`, `AssetRegistry`, `AssetPack`
- `AssetUpdateQuery`, `AssetValidationResults`
- Events: `GenerateAssetsEvent`, `LoadedAssetsEvent`, `RegisterAssetStoreEvent`

This aligns with the `IncludesAssetPack` manifest flag and asset registries.

## Universe, World, and Worldgen

Universe and storage:

- `Universe`, `PlayerRef`
- `DiskDataStore`, `DiskPlayerStorageProvider`

World access and chunk internals:

- `BlockAccessor`, `ChunkAccessor`
- `BlockChunk`, `EntityChunk`, `EnvironmentChunk`

Worldgen entry points:

- `IWorldGen`, `IWorldGenProvider`
- `FlatWorldGenProvider`, `VoidWorldGenProvider`
- `WorldGenTimingsCollector`, `WorldGenBenchmarkCommand`

## Built-in Plugin Families (From manifests.json)

The server ships many built-in plugins. These names are good clues for extension points:

- Systems: `AssetEditor`, `Model`, `TagSet`, `LegacyModule`
- Gameplay: `Crafting`, `Farming`, `Weather`, `Mounts`, `Teleport`, `Portals`
- NPC/AI: `NPC`, `NPCObjectives`, `NPCReputation`, `NPCShop`, `NPCEditor`
- Economy/Goals: `Shop`, `Reputation`, `Objectives`, `ObjectiveShop`, `ObjectiveReputation`
- World: `Instances`, `WorldGen`, `HytaleGenerator`, `Teleporter`
- Tools: `BuilderTools`, `CommandMacro`, `AssetEditor`, `CreativeHub`

## Subsystem Map (From Package Names)

These packages exist in the JAR and suggest specific modding areas:

- Access control: `server.core.modules.accesscontrol` (ban/whitelist commands)
- Collision: `server.core.modules.collision` (hitboxes, block collisions)
- Entity UI: `server.core.modules.entityui` (combat text, entity UI components)
- Permissions: `server.core.permissions` (groups, user permissions, op commands)
- Prefab tools: `server.core.modules.prefabspawner` (prefab weights/spawn rules)
- UI toolkit: `server.core.ui` and `server.core.ui.browser` (UI builders, file browser)

## Subsystem Index (Packages -> Plugin Ideas)

| Package / Area | What It Suggests | Practical Plugin Ideas |
|---|---|---|
| `server.core.command.system` | Typed commands with argument types | Admin tooling, structured CLI, debug helpers |
| `server.core.event.events.player` | Player lifecycle hooks | Join rewards, onboarding, hub teleports |
| `server.core.event.events.ecs` | Block/entity actions | Region protection, logging, minigame rules |
| `server.core.event.events.permissions` | Permission changes | Audit logs, role sync, web admin |
| `server.core.prefab` | Prefab storage and rotation | Build tools, prefab limits, paste validation |
| `server.core.modules.prefabspawner` | Prefab spawner rules | Spawn balancing, seasonal structure swaps |
| `server.core.permissions` | Groups, ops, permission system | Moderation panels, auto-role policies |
| `server.core.universe` | World and player storage | World backups, per-world settings |
| `server.core.universe.world.worldgen` | World generation providers | Custom generators, benchmark tooling |
| `server.core.modules.entityui` | Entity UI components | Combat text, in-world HUDs |
| `server.core.ui` / `server.core.ui.browser` | UI pages, file browser | In-game admin UI, asset tools |
| `assetstore.*` | Asset pack registry | Custom asset packs, validation tools |

## UI and In-Game Tools

There are core UI and UI builder classes:

- `UICommandBuilder`, `UIEventBuilder`
- `ServerFileBrowser`, `FileBrowserConfig`
- `EntityUIModule` and UI components like `CombatTextUIComponent`

Builder tools include prefab editor UI pages under
`com.hypixel.hytale.builtin.buildertools.prefabeditor.ui`.

## Example Plugin Ideas (Grounded in Observed Classes)

These examples are derived from class names above; confirm APIs before implementation.

1. **Permission audit log**  
   Listen for `PlayerPermissionChangeEvent`, `PlayerGroupEvent`, `GroupPermissionChangeEvent` and write to a JSON log.

2. **Prefab paste limiter / validator**  
   Hook `PrefabPasteEvent` to enforce size limits or block lists before placement.

3. **Anti-grief region rules**  
   Use `BreakBlockEvent`, `PlaceBlockEvent`, `UseBlockEvent` to enforce region protection.

4. **Chat formatting and moderation**  
   Listen to `PlayerChatEvent` and apply filters or templated formatting (presence of `PlayerChatEvent$Formatter`).

5. **Worldgen experiment toggles**  
   Add admin commands that swap worldgen providers (`IWorldGenProvider` classes are present).

6. **Entity combat text overlay**  
   Leverage `EntityUIModule` and UI components like `CombatTextUIComponent`.

7. **Asset pack validator**  
   On `LoadedAssetsEvent`, run a custom validation pass and emit warnings.

8. **Prefab spawner editor**  
   Wrap `PrefabSpawnerCommand` functionality in a plugin command or UI page.

9. **Plugin health dashboard**  
   Use `PluginManager` and built-in plugin command patterns to report load state and dependencies.

10. **Player world transition hooks**  
    `AddPlayerToWorldEvent` and `DrainPlayerFromWorldEvent` suggest a clean place for teleport rules.

11. **Permission mirror to external systems**  
    Watch `PlayerPermissionChangeEvent` and sync roles to a Discord or web panel.

12. **Prefab spawn balancing**  
    Build a command that edits `PrefabSpawner` weights based on live server conditions.

13. **Collision testing tools**  
    Use collision module command classes as patterns for custom hitbox diagnostics.

14. **Worldgen benchmark runner**  
    Wrap `WorldGenBenchmarkCommand` in a plugin command for remote admins.

15. **UI-driven admin tools**  
    Leverage `UICommandBuilder` and `ServerFileBrowser` to create in-game dashboards.

## Known Limits of This Dissection

- No decompilation performed; signatures and behavior are inferred.
- Some classes may be internal-only and not exposed to plugins.
- Names can change between server builds; verify against your target JAR.
