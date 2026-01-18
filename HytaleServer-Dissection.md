# HytaleServer.jar Dissection Notes (2026.01.13-dcad8778f)

These notes are based on inspecting the JAR contents (class/package names, embedded manifests).
No decompilation was used, so behavior is inferred from naming. Treat this as a map of what exists,
not a full API reference.

## Build Snapshot

- Main class: `com.hypixel.hytale.Main`
- Alternate entry: `com.hypixel.hytale.LateMain`
- Version: `2026.01.13-dcad8778f`
- Java: 21 (build JDK spec 25)
- Maven coords: `com.hypixel.hytale:Server:2026.01.13-dcad8778f`
- Manifest revision: `dcad8778f19e4e56af55d74b58575c91c50a018d`

## Package Map (Top-Level Namespaces)

Top-level package roots under `com.hypixel.hytale` (from `jar tf`):

- `assetstore` (asset pack registry, codecs, iterators)
- `builtin` (built-in plugins such as worldgen, builder tools, NPC systems)
- `codec` (data codecs, schema, validation, lookup)
- `common` (collections, threading, semver, util)
- `component` (ECS-style component system)
- `event` (event buses and registries)
- `function` (functional helpers)
- `logger` (logger facade, backends, sentry)
- `math` (vectors, matrices, random, shapes)
- `metrics` (executor metrics, init tracing)
- `plugin` (early plugin loader)
- `procedurallib` (procedural generation helpers)
- `protocol` (network protocol classes)
- `registry` (registries and keys)
- `server` (core server runtime)
- `sneakythrow` (exception helper)
- `storage` (storage and persistence)
- `unsafe` (unsafe helpers)

## Package Map (Server Runtime)

Top-level server runtime packages under `com.hypixel.hytale.server`:

- `core` (main server runtime and subsystems)
- `flock` (flocking AI systems)
- `migrations` (data migrations)
- `npc` (NPC systems and built-in NPC plugin)
- `spawning` (spawner systems)
- `worldgen` (world generation support)

Core subsystems under `com.hypixel.hytale.server.core`:

- `asset`, `auth`, `blocktype`, `client`, `codec`, `command`, `console`
- `cosmetics`, `entity`, `event`, `inventory`, `io`, `meta`, `modules`
- `permissions`, `plugin`, `prefab`, `receiver`, `registry`, `task`
- `ui`, `universe`, `util`

## Package Map (Protocol and Data Types)

Protocol packages under `com.hypixel.hytale.protocol`:

- `io` (packet I/O, varints, validation, netty encoder/decoder)
- `packets` (network packet groupings)
  - `asseteditor`, `assets`, `auth`, `buildertools`, `camera`, `connection`
  - `entities`, `interaction`, `interface_`, `inventory`, `machinima`
  - `player`, `serveraccess`, `setup`, `window`, `world`, `worldmap`

The protocol root also contains many data classes (e.g., assets, blocks, items, interactions).

Packet group samples (class prefixes under `com.hypixel.hytale.protocol.packets.*`):

- `asseteditor` (73): `AssetEditor*` (create/update/delete assets, asset packs, schemas)
- `assets` (47): `Update*` (asset registry updates for blocks, items, audio, fx, etc.)
- `auth` (10): `Auth*`, `Password*`, `ConnectAccept`
- `buildertools` (47): `BuilderTool*`, `Prefab*`, `EntityToolAction`
- `camera` (5): `SetServerCamera`, `SetFlyCameraMode`
- `connection` (8): `Connect`, `Disconnect`, `Ping`, `Pong`
- `entities` (8): `EntityUpdates`, `ApplyKnockback`, `PlayAnimation`
- `interaction` (7): `SyncInteractionChain*`, `PlayInteractionFor`
- `interface_` (42): `CustomPage*`, `CustomUI*`, `ChatMessage`, `ServerInfo`
- `inventory` (11): `InventoryAction`, `MoveItemStack`, `UpdatePlayerInventory`
- `machinima` (5): `UpdateMachinimaScene`, `SetMachinimaActorModel`
- `player` (21): `JoinWorld`, `ClientMovement`, `SetGameMode`, `UpdateMovementSettings`
- `serveraccess` (5): `RequestServerAccess`, `SetServerAccess`, `UpdateServerAccess`
- `setup` (17): `RequestAssets`, `AssetInitialize`, `WorldLoadProgress`, `ViewRadius`
- `window` (17): `OpenWindow`, `CraftItemAction`, `UpdateWindow`
- `world` (36): `SetChunk`, `UpdateTime`, `ServerSetBlock`, `UpdateWeather`
- `worldmap` (12): `UpdateWorldMap`, `MapChunk`, `MapMarker`

## Server Networking (JAR)

Key server-side networking classes:

- `com.hypixel.hytale.server.core.io.PacketHandler` (base handler)
- `com.hypixel.hytale.server.core.io.netty.PlayerChannelHandler` (Netty inbound bridge)
- `com.hypixel.hytale.server.core.io.adapter.PacketAdapters` (inbound/outbound filter registry)

Packet handler implementations observed in the JAR:

- Core: `InitialPacketHandler`, `AuthenticationPacketHandler`, `PasswordPacketHandler`,
  `SetupPacketHandler`, `GamePacketHandler`, `InventoryPacketHandler`,
  `GenericConnectionPacketHandler`, `GenericPacketHandler`, `SubPacketHandler`
- Built-in: `AssetEditorPacketHandler`, `AssetEditorGamePacketHandler`,
  `BuilderToolsPacketHandler`, `MountGamePacketHandler`

## Package Map (Procedural Library)

`com.hypixel.hytale.procedurallib` subpackages:

- `condition`, `json`, `logic`, `property`, `random`, `supplier`, `util`
- Noise helpers: `NoiseFunction`, `NoiseFunction2d`, `NoiseFunction3d`, `NoiseType`

## Package Map (Registry and Storage)

- `com.hypixel.hytale.registry` (Registry, Registration)
- `com.hypixel.hytale.storage` (IndexedStorageFile, legacy `IndexedStorageFile_v0`)

## Package Map (UI)

Server UI tooling in `com.hypixel.hytale.server.core.ui`:

- `browser` (server file browser)
- `builder` (UI command/event builders)

## Built-in Plugin Package Roots

Package roots under `com.hypixel.hytale.builtin`:

- `adventure` (objectives, reputation, shop, stash, teleporter, etc.)
- `ambience`, `asseteditor`, `beds`, `blockphysics`, `blockspawner`
- `blocktick`, `buildertools`, `commandmacro`, `crafting`, `creativehub`
- `crouchslide`, `deployables`, `fluid`, `hytalegenerator`, `instances`
- `landiscovery`, `mantling`, `model`, `mounts`, `npccombatactionevaluator`
- `npceditor`, `parkour`, `path`, `portals`, `safetyroll`, `sprintforce`
- `tagset`, `teleport`, `weather`, `worldgen`

## Core Module Packages

Modules under `com.hypixel.hytale.server.core.modules`:

- `accesscontrol`, `block`, `blockhealth`, `blockset`, `camera`
- `collision`, `debug`, `entity`, `entitystats`, `entityui`
- `i18n`, `interaction`, `item`, `migrations`, `physics`
- `prefabspawner`, `projectile`, `serverplayerlist`
- `singleplayer`, `splitvelocity`, `time`

## Built-in Plugin Manifests (manifests.json)

The JAR includes `manifests.json` with 46 plugin entries and one plugin with two sub-plugins.

Sub-plugins:
- `Hytale:NPC` -> `Spawning`, `Flock`

Plugin list (Group:Name -> Main):
- Hytale:AssetEditor -> com.hypixel.hytale.builtin.asseteditor.AssetEditorPlugin
- Hytale:BlockSpawner -> com.hypixel.hytale.builtin.blockspawner.BlockSpawnerPlugin
- Hytale:BlockTick -> com.hypixel.hytale.builtin.blocktick.BlockTickPlugin
- Hytale:BlockPhysics -> com.hypixel.hytale.builtin.blockphysics.BlockPhysicsPlugin
- Hytale:BuilderTools -> com.hypixel.hytale.builtin.buildertools.BuilderToolsPlugin
- Hytale:Crafting -> com.hypixel.hytale.builtin.crafting.CraftingPlugin
- Hytale:CommandMacro -> com.hypixel.hytale.builtin.commandmacro.MacroCommandPlugin
- Hytale:Instances -> com.hypixel.hytale.builtin.instances.InstancesPlugin
- Hytale:LANDiscovery -> com.hypixel.hytale.builtin.landiscovery.LANDiscoveryPlugin
- Hytale:NPC -> com.hypixel.hytale.server.npc.NPCPlugin
- Hytale:NPCObjectives -> com.hypixel.hytale.builtin.adventure.npcobjectives.NPCObjectivesPlugin
- Hytale:ObjectiveReputation -> com.hypixel.hytale.builtin.adventure.objectivereputation.ObjectiveReputationPlugin
- Hytale:Objectives -> com.hypixel.hytale.builtin.adventure.objectives.ObjectivePlugin
- Hytale:ObjectiveShop -> com.hypixel.hytale.builtin.adventure.objectiveshop.ObjectiveShopPlugin
- Hytale:Path -> com.hypixel.hytale.builtin.path.PathPlugin
- Hytale:Reputation -> com.hypixel.hytale.builtin.adventure.reputation.ReputationPlugin
- Hytale:NPCReputation -> com.hypixel.hytale.builtin.adventure.npcreputation.NPCReputationPlugin
- Hytale:Shop -> com.hypixel.hytale.builtin.adventure.shop.ShopPlugin
- Hytale:ShopReputation -> com.hypixel.hytale.builtin.adventure.shopreputation.ShopReputationPlugin
- Hytale:NPCShop -> com.hypixel.hytale.builtin.adventure.npcshop.NPCShopPlugin
- Hytale:NPCEditor -> com.hypixel.hytale.builtin.npceditor.NPCEditorPlugin
- Hytale:Stash -> com.hypixel.hytale.builtin.adventure.stash.StashPlugin
- Hytale:TagSet -> com.hypixel.hytale.builtin.tagset.TagSetPlugin
- Hytale:Teleport -> com.hypixel.hytale.builtin.teleport.TeleportPlugin
- Hytale:Fluid -> com.hypixel.hytale.builtin.fluid.FluidPlugin
- Hytale:Weather -> com.hypixel.hytale.builtin.weather.WeatherPlugin
- Hytale:WorldGen -> com.hypixel.hytale.builtin.worldgen.WorldGenPlugin
- Hytale:Farming -> com.hypixel.hytale.builtin.adventure.farming.FarmingPlugin
- Hytale:Camera -> com.hypixel.hytale.builtin.adventure.camera.CameraPlugin
- Hytale:WorldLocationCondition -> com.hypixel.hytale.builtin.adventure.worldlocationcondition.WorldLocationConditionPlugin
- Hytale:NPCCombatActionEvaluator -> com.hypixel.hytale.builtin.npccombatactionevaluator.NPCCombatActionEvaluatorPlugin
- Hytale:Model -> com.hypixel.hytale.builtin.model.ModelPlugin
- Hytale:Mantling -> com.hypixel.hytale.builtin.mantling.MantlingPlugin
- Hytale:SafetyRoll -> com.hypixel.hytale.builtin.safetyroll.SafetyRollPlugin
- Hytale:SprintForce -> com.hypixel.hytale.builtin.sprintforce.SprintForcePlugin
- Hytale:CrouchSlide -> com.hypixel.hytale.builtin.crouchslide.CrouchSlidePlugin
- Hytale:Parkour -> com.hypixel.hytale.builtin.parkour.ParkourPlugin
- Hytale:Mounts -> com.hypixel.hytale.builtin.mounts.MountPlugin
- Hytale:HytaleGenerator -> com.hypixel.hytale.builtin.hytalegenerator.plugin.HytaleGenerator
- Hytale:Teleporter -> com.hypixel.hytale.builtin.adventure.teleporter.TeleporterPlugin
- Hytale:Memories -> com.hypixel.hytale.builtin.adventure.memories.MemoriesPlugin
- Hytale:Deployables -> com.hypixel.hytale.builtin.deployables.DeployablesPlugin
- Hytale:Portals -> com.hypixel.hytale.builtin.portals.PortalsPlugin
- Hytale:Beds -> com.hypixel.hytale.builtin.beds.BedsPlugin
- Hytale:Ambience -> com.hypixel.hytale.builtin.ambience.AmbiencePlugin
- Hytale:CreativeHub -> com.hypixel.hytale.builtin.creativehub.CreativeHubPlugin

## Built-in Plugin Dependency Graph (manifests.json)

Each entry lists required and optional dependencies as declared in the manifest.

- Hytale:Ambience -> required: Hytale:EntityModule; optional: none
- Hytale:AssetEditor -> required: Hytale:I18nModule, Hytale:ItemModule, Hytale:Model; optional: none
- Hytale:Beds -> required: Hytale:Mounts; optional: none
- Hytale:BlockPhysics -> required: Hytale:BlockTick, Hytale:BlockTypeModule; optional: none
- Hytale:BlockSpawner -> required: Hytale:BlockModule, Hytale:BlockStateModule, Hytale:I18nModule; optional: none
- Hytale:BlockTick -> required: Hytale:LegacyModule; optional: none
- Hytale:BuilderTools -> required: Hytale:EntityModule; optional: none
- Hytale:Camera -> required: Hytale:AssetModule, Hytale:DamageModule, Hytale:InteractionModule; optional: none
- Hytale:CommandMacro -> required: none; optional: none
- Hytale:Crafting -> required: Hytale:BlockStateModule, Hytale:EntityModule, Hytale:InteractionModule; optional: none
- Hytale:CreativeHub -> required: Hytale:I18nModule, Hytale:Instances; optional: none
- Hytale:CrouchSlide -> required: none; optional: none
- Hytale:Deployables -> required: none; optional: none
- Hytale:Farming -> required: Hytale:BlockPhysics, Hytale:BlockStateModule, Hytale:InteractionModule; optional: none
- Hytale:Fluid -> required: Hytale:BlockTick, Hytale:LegacyModule; optional: none
- Hytale:HytaleGenerator -> required: none; optional: none
- Hytale:Instances -> required: Hytale:BlockPhysics, Hytale:BlockStateModule, Hytale:I18nModule; optional: none
- Hytale:LANDiscovery -> required: none; optional: none
- Hytale:Mantling -> required: none; optional: none
- Hytale:Memories -> required: Hytale:NPC; optional: none
- Hytale:Model -> required: none; optional: none
- Hytale:Mounts -> required: Hytale:DamageModule, Hytale:NPC; optional: none
- Hytale:NPC -> required: Hytale:AssetModule, Hytale:DamageModule, Hytale:EntityModule, Hytale:EntityStatsModule, Hytale:TagSet; optional: Hytale:Objectives, Hytale:Path, Hytale:Teleport
- Hytale:NPCCombatActionEvaluator -> required: Hytale:NPC; optional: none
- Hytale:NPCEditor -> required: Hytale:AssetEditor, Hytale:NPC; optional: none
- Hytale:NPCObjectives -> required: Hytale:NPC, Hytale:Objectives, Hytale:Spawning, Hytale:TagSet; optional: none
- Hytale:NPCReputation -> required: Hytale:NPC, Hytale:Reputation, Hytale:TagSet; optional: none
- Hytale:NPCShop -> required: Hytale:I18nModule, Hytale:NPC, Hytale:Shop; optional: none
- Hytale:ObjectiveReputation -> required: Hytale:Objectives, Hytale:Reputation; optional: none
- Hytale:ObjectiveShop -> required: Hytale:Objectives, Hytale:Shop; optional: none
- Hytale:Objectives -> required: Hytale:BlockStateModule, Hytale:EntityModule, Hytale:I18nModule, Hytale:InteractionModule; optional: none
- Hytale:Parkour -> required: Hytale:EntityModule; optional: none
- Hytale:Path -> required: Hytale:BuilderTools, Hytale:EntityModule; optional: none
- Hytale:Portals -> required: Hytale:Ambience, Hytale:Instances, Hytale:NPC; optional: none
- Hytale:Reputation -> required: Hytale:I18nModule; optional: none
- Hytale:SafetyRoll -> required: none; optional: none
- Hytale:Shop -> required: Hytale:InteractionModule; optional: none
- Hytale:ShopReputation -> required: Hytale:Reputation, Hytale:Shop; optional: none
- Hytale:SprintForce -> required: none; optional: none
- Hytale:Stash -> required: Hytale:BlockStateModule, Hytale:ItemModule; optional: none
- Hytale:TagSet -> required: none; optional: none
- Hytale:Teleport -> required: Hytale:EntityModule, Hytale:I18nModule; optional: none
- Hytale:Teleporter -> required: Hytale:BlockStateModule, Hytale:Teleport; optional: none
- Hytale:Weather -> required: Hytale:EntityModule; optional: none
- Hytale:WorldGen -> required: Hytale:EntityModule; optional: none
- Hytale:WorldLocationCondition -> required: none; optional: none

## Embedded Third-Party Libraries (Package Roots)

The JAR is shaded and includes common libraries under these package roots:

- `ch.randelshofer.fastdoubleparser`
- `com.github.luben.zstd`
- `com.google.gson`, `com.google.protobuf`, `com.google.crypto.tink`
- `com.nimbusds`
- `io.netty`, `io.sentry`
- `it.unimi` (fastutil)
- `joptsimple`
- `org.bouncycastle`, `org.bson`
- `org.checkerframework`
- `org.fusesource`
- `org.jline`

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

## Mod Packaging Conventions (From Local Sample Mods)

From 18 sample mods (11 `.jar`, 7 `.zip`), each contained a root-level `manifest.json`.

Observed patterns:
- JAR mods include a `Main` entry in `manifest.json`.
- ZIP mods (content packs) omit `Main` and list only metadata.
- Dependencies are declared by `Group:Name` identifiers (including mod-to-mod deps).

Examples of dependency strings seen:
- `Hytale:AccessControlModule`, `Hytale:EntityModule`, `Hytale:InteractionModule`
- `Serilum:Hybrid` (mod-to-mod)
- `Darkhax:Spellbook` (mod-to-mod)

These samples suggest:
- `manifest.json` is expected at the archive root.
- ZIP mods are likely content packs (data/asset), not code plugins.

## Mod Load Order Notes (Sample Mods)

Observed mod-to-mod dependencies from local sample mods indicate load order requirements:

- `Serilum:Hybrid` is required by `Serilum:InfiniteMana` and `Serilum:TreeHarvester`.
- `Darkhax:Spellbook` is required by `Darkhax:WaybackCharm`.

Treat these as mod-specific requirements and ensure dependency mods load first.

## Practical Patterns From Sample Mods (Non-verbatim)

The following guidance is derived from file layouts and JSON shapes seen in sample mods. It is a practical starting point, not an official API reference.

### Add Blocks (Data Pack Style)

Typical layout:
- `Server/Item/Items/...` for item/block definitions (JSON)
- `Server/Item/Block/Hitboxes/...` for hitbox definitions (JSON)
- `Server/Item/Groups/...` for block grouping (JSON)
- `Server/Item/Recipes/...` for crafting recipes (JSON)
- `Server/Languages/...` for localized strings (LANG)
- Textures/models under `Blocks/`, `BlockTextures/`, `Items/`, and `Icons/ItemsGenerated`

Minimal approach:
1. Create an item JSON under `Server/Item/Items/<your_folder>/Your_Block.json`.
2. Include a `BlockType` section with material, draw type, sounds, particles, and textures or models.
3. Reference a hitbox JSON under `Server/Item/Block/Hitboxes/...` if the block is not a basic cube.
4. Add an icon texture and localized name/description keys.
5. Optionally add a recipe JSON under `Server/Item/Recipes/...` and group entry under `Server/Item/Groups/...`.

### Add Tools or Usable Items

Typical layout:
- Item definition JSON under `Server/Item/Items/...`
- Interaction definitions under `Server/Item/Interactions/Item/...`
- Root interaction mapping under `Server/Item/RootInteractions/...`

Minimal approach:
1. Define the item with `Categories` like `Items.Tools`, model/texture, and icon properties.
2. Add an `Interactions` map on the item that points to interaction IDs.
3. Create interaction JSON files with a `Type` pointing at the plugin interaction handler.
4. Add a root interaction JSON to allow the item to trigger actions in a specific context (e.g., creative).

This pattern lets a code plugin handle behavior while the content pack defines how the item is presented and invoked.

### Add UI Pages (Custom UI)

Typical layout:
- UI page definitions under `Common/UI/Custom/Pages/...` (`.ui` files)
- UI images under `Common/UI/Custom/...` (PNG)
- Plugin classes bind UI pages at runtime (code-side)

Minimal approach:
1. Create `.ui` layout files under `Common/UI/Custom/Pages/...` with standard layout containers.
2. Reference shared UI components (e.g., a common stylesheet/ui include) via relative imports.
3. Provide UI textures under `Common/UI/Custom/...` and reference them in the UI files.
4. In code, load and show the UI page when an interaction or command is triggered.

UI data files define layout, while the plugin provides the data and event handling.

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

### Confirmed by Decompiled Server Sources

The extracted server sources confirm these event classes and ECS types:

- Core lifecycle: `BootEvent`, `PrepareUniverseEvent`, `ShutdownEvent`
- Player events: `PlayerConnectEvent`, `PlayerDisconnectEvent`, `PlayerReadyEvent`, `PlayerChatEvent`, `PlayerRefEvent`
- ECS events: `BreakBlockEvent`, `PlaceBlockEvent`, `UseBlockEvent`, `CraftRecipeEvent`, `DropItemEvent`, `InteractivelyPickupItemEvent`, `SwitchActiveSlotEvent`
- Entity events: `EntityRemoveEvent`, `LivingEntityInventoryChangeEvent`
- ECS core: `Store`, `Ref`, `ArchetypeChunk`, `Query`, `EntityEventSystem`, `EntityStore`, `PlayerRef`

## ECS and Entity/Player Core (Observed Classes)

Core ECS types:

- `com.hypixel.hytale.component.Store` (add/remove entities, components, invoke ECS events)
- `com.hypixel.hytale.component.Ref` (store + index reference with validation)
- `com.hypixel.hytale.component.ArchetypeChunk`
- `com.hypixel.hytale.component.system.EntityEventSystem`

Entity storage and player bridge:

- `com.hypixel.hytale.server.core.universe.world.storage.EntityStore`
- `com.hypixel.hytale.server.core.universe.PlayerRef` (Component + IMessageReceiver)

Common entity components (examples):

- `server.core.modules.entity.component.TransformComponent`
- `server.core.modules.entity.component.DisplayNameComponent`
- `server.core.modules.entity.player.PlayerSkinComponent`
- `server.core.modules.entity.item.ItemComponent`

From `javap`, `PlayerRef` exposes identity, messaging, and movement helpers:
`getUuid()`, `getUsername()`, `getReference()`, `getTransform()`, `updatePosition(...)`,
`sendMessage(Message)`.

Decompiled event field examples (selected):

- `PlayerChatEvent` is async + cancellable; holds `sender`, `targets`, `content`, and a `Formatter`.
- `BreakBlockEvent` carries `itemInHand`, `targetBlock`, `blockType`.
- `PlaceBlockEvent` carries `itemInHand`, `targetBlock`, `rotation`.
- `UseBlockEvent.Pre` is cancellable with `interactionType`, `context`, `targetBlock`, `blockType`.

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

Player transfer (JAR):

- `PlayerRef.referToServer(host, port[, data])` sends a `ClientReferral` packet
  (`com.hypixel.hytale.protocol.packets.auth.ClientReferral`) with a
  `HostAddress` target and optional referral data (max 4096 bytes).
- Incoming referrals surface on `PlayerSetupConnectEvent`
  (`getReferralData()`, `getReferralSource()`).

World access and chunk internals:

- `BlockAccessor`, `ChunkAccessor`
- `BlockChunk`, `EntityChunk`, `EnvironmentChunk`

Worldgen entry points:

- `IWorldGen`, `IWorldGenProvider`
- `FlatWorldGenProvider`, `VoidWorldGenProvider`
- `WorldGenTimingsCollector`, `WorldGenBenchmarkCommand`

### Worldgen: Zones, Biomes, Caves (JAR)

The JAR exposes a zone/biome/cave stack under `com.hypixel.hytale.server.worldgen.*`:

- Zones: `ZonePatternGenerator.generate(seed, x, z)` returns `ZoneGeneratorResult`.
  - `ZoneGeneratorResult.getZone()` and `getBorderDistance()`.
  - `Zone` is a record with `biomePatternGenerator()` and `caveGenerator()`.
  - Discovery metadata lives in `ZoneDiscoveryConfig`.
- Biomes: `BiomePatternGenerator.generateBiomeAt(zoneResult, seed, x, z)` and
  `getCustomBiomeAt(...)` (`Biome`, `CustomBiome`, `TileBiome`).
- Caves: `CaveGenerator.getCaveTypes()` returns `CaveType[]`.
  - `CaveType.getBiomeMask()` exposes an `Int2FlagsCondition` used for biome masks.
  - `CaveBiomeMaskFlags` exists under `server.worldgen.cave`.

External tutorials describe zones, biomes, and caves as linked systems with
zone borders and biome masks; these concepts align with the class structure
above but should still be validated on your target build.

## Dimensions (Multiple Worlds) - Inferred Guidance

No explicit "dimension" API is visible in the JAR metadata. The closest observable concept is multiple worlds/instances managed by the universe/instances systems. Treat "dimensions" as multiple worlds with their own worldgen and settings.

Observed clues:
- `com.hypixel.hytale.server.core.universe` (Universe/World management)
- `com.hypixel.hytale.builtin.instances.InstancesPlugin` (instance system)
- `com.hypixel.hytale.server.core.universe.world.worldgen.*` (per-world generation)
- `IWorldGenProvider` and related worldgen classes

Suggested setup (unverified, based on class names):
1. Define additional worlds/instances via server configuration (per-world entries).
2. Assign a worldgen provider to each world (e.g., `FlatWorldGenProvider`, `VoidWorldGenProvider`, or a custom provider).
3. Use teleport/instance APIs to move players between worlds.
4. Ensure per-world data storage is isolated in the universe directory.

Where to look (class/package anchors to inspect):
- `com.hypixel.hytale.server.core.universe.Universe`
- `com.hypixel.hytale.server.core.universe.world.*` (world objects and storage)
- `com.hypixel.hytale.server.core.universe.world.worldgen.*` (worldgen provider interfaces)
- `com.hypixel.hytale.builtin.instances.InstancesPlugin` (instance lifecycle)
- `com.hypixel.hytale.builtin.teleport.TeleportPlugin` and `com.hypixel.hytale.builtin.adventure.teleporter.TeleporterPlugin`

This section is inferred from naming and package layout, not from decompiled code. Validate on the target server build before relying on it.

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
- Entity stats: `server.core.modules.entitystats` (stat tracking and modifiers)
- Interaction system: `server.core.modules.interaction` (interaction chains, rules)
- Items: `server.core.modules.item` (item definitions and item systems)
- Permissions: `server.core.permissions` (groups, user permissions, op commands)
- Prefab tools: `server.core.modules.prefabspawner` (prefab weights/spawn rules)
- Projectiles: `server.core.modules.projectile` (projectile movement and impacts)
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

16. **Server player list sync**  
    Use `server.core.modules.serverplayerlist` plus `UpdateServerPlayerList` packets for live staff visibility.

17. **Localized UI strings**  
    Build UI pages with `LocalizableString` to enforce i18n-ready admin panels.

18. **Inventory action audit**  
    Track `InventoryAction`/`MoveItemStack` flows to flag duping patterns or suspicious trades.

19. **Custom HUD modules**  
    Emit `CustomHud` / `HudComponent` updates for RPG stats or timed buffs.

20. **World map markers**  
    Use `MapMarker` and `UpdateWorldMap` for quests, POIs, or admin telemetry.

21. **Time and weather controls**  
    Expose `UpdateTime`, `UpdateWeather`, and `SetTimeDilation` toggles to admins.

22. **Entity stat balancing**  
    Wire `entitystats` module events with `UpdateEntityStatTypes` for live tuning.

23. **Projectile sandbox**  
    Build a range test using `server.core.modules.projectile` and `UpdateProjectileConfigs`.

24. **Interaction chain visualizer**  
    Use `SyncInteractionChain*` packets to trace and display interaction chains for QA.

25. **Prefab preview and catalog**  
    Render prefab lists with `Prefab*` packets and a UI page for selection/paste.

26. **Asset pack rollout gates**  
    Combine `AssetEditor*` and `Update*` assets to stage new packs for selected users.

27. **Crash-safe save hooks**  
    Use `server.core.universe` events plus `IndexedStorageFile` to checkpoint world data.

28. **Camera director tools**  
    Leverage `SetServerCamera` and `SetFlyCameraMode` for machinima or admin control.

29. **Debug shape overlays**  
    Use `DisplayDebug` / `ClearDebugShapes` to draw paths, spawn volumes, or nav hints.

30. **Access control workflows**  
    Extend `server.core.modules.accesscontrol` and `RequestServerAccess` for approval queues.

31. **Player position audit**  
    Read `PlayerRef.getTransform()` and `TransformComponent` to log or gate movement patterns.

32. **Nameplate overrides**  
    Use `DisplayNameComponent` to apply role-based display names in-world.

33. **Entity inventory watcher**  
    Listen for `LivingEntityInventoryChangeEvent` and inspect `ItemComponent` for balance checks.

34. **Loot filter rules**  
    Use `InteractivelyPickupItemEvent` and `DropItemEvent` for auto-pickup filters or anti-theft logic.

35. **Hotbar ability switching**  
    Use `SwitchActiveSlotEvent` to toggle tool modes or ability bars.

36. **Crafting gatekeeper**  
    Use `CraftRecipeEvent` to restrict recipes by zone, role, or progression.

37. **Unique item gating**  
    Track `UniqueItemUsagesComponent` to limit one-time rewards or items.

38. **Dropped item tuning**  
    Adjust pickup/merge timing using `ItemComponent` delays and `DropItemEvent` data.

39. **Movement-state rules**  
    Use `MovementStatesComponent` to gate actions while sprinting, crouching, or gliding.

## Known Limits of This Dissection

- Partial decompilation performed; coverage is limited to extracted source subsets.
- Some classes may be internal-only and not exposed to plugins.
- Names can change between server builds; verify against your target JAR.

## Changelog (Doc Updates)

- Added mod load-order notes based on local sample mods (not included in this repo).
- Added practical patterns for blocks, tools, and UI from sample mod layouts.
