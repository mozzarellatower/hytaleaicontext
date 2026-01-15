# HytaleServer.jar documentation (reverse-engineered)

This document is derived from inspecting `HytaleServer.jar` contents (manifest metadata, embedded resources, and class string constants). It is not an official spec; verify behavior by running the server or consulting official docs.

## Build metadata

- Main class: `com.hypixel.hytale.Main`
- Java requirement: `Java-Version: 21` (build JDK spec `25`)
- Implementation version: `2026.01.13-dcad8778f` (branch `release`, patchline `release`)
- Artifact: `com.hypixel.hytale:Server`

## External notes (hytalemodding.dev)

These points come from hytalemodding.dev and are not derived from the JAR itself:

- Java runtime: documentation there states server compatibility and modding tooling require Java 25+.
- Server tick rate: a developer Q&A summary claims default 30 TPS and that plugins can change it.

Treat these as external claims and verify against the target server build.

## High-level structure

The JAR is a shaded (fat) server distribution that embeds:

- Core server packages: `com.hypixel.hytale.server.core.*`
- NPC systems: `com.hypixel.hytale.server.npc.*`
- Worldgen modules: `com.hypixel.hytale.server.worldgen.*`
- Spawning and flocking systems: `com.hypixel.hytale.server.spawning.*`, `com.hypixel.hytale.server.flock.*`
- Early plugin loader: `com.hypixel.hytale.plugin.early.*`
- Built-in plugins declared in `manifests.json`
- Block migration maps in `migration/blocks/*.json`
- Third-party runtime libraries (Netty, JLine, BouncyCastle, etc.)

Notable core sub-systems (by package density):

- `modules` (server module framework)
- `command` (command system)
- `asset` (asset handling and registry)
- `universe` (world/universe management)
- `entity`, `inventory`, `auth`, `permissions`, `ui`, `cosmetics`

## Entry point and runtime

Start with:

```bash
java -jar HytaleServer.jar
```

The entry point is `com.hypixel.hytale.Main`. An alternate class `com.hypixel.hytale.LateMain` exists, likely for delayed or staged startup.

## Configuration files

From string constants in `HytaleServerConfig`:

- Config filename: `config.json`
- Sections/keys referenced by name: `Modules`, `ConnectionTimeouts`, `RateLimit`, `AuthCredentialStore`
- Per-mod configuration appears to be keyed by plugin identifier (map of `PluginIdentifier -> ModConfig`)
- A legacy plugin configuration path exists (`legacyPluginConfig`)

These names indicate the server will generate or read a JSON config file in the working directory and merge module/mod settings from it.

### Sample config.json (serverexample)

Observed keys and defaults from `serverexample/config.json`:

| Key | Example value |
|---|---|
| `Version` | `3` |
| `ServerName` | `Hytale Server` |
| `MOTD` | empty string |
| `Password` | empty string |
| `MaxPlayers` | `100` |
| `MaxViewRadius` | `32` |
| `LocalCompressionEnabled` | `false` |
| `Defaults.World` | `default` |
| `Defaults.GameMode` | `Adventure` |
| `ConnectionTimeouts.JoinTimeouts` | `{}` |
| `RateLimit` | `{}` |
| `Modules.PathPlugin.Modules` | `{}` |
| `LogLevels` | `{}` |
| `Mods` | `{}` |
| `DisplayTmpTagsInStrings` | `false` |
| `PlayerStorage.Type` | `Hytale` |
| `AuthCredentialStore.Type` | `Encrypted` |
| `AuthCredentialStore.Path` | `auth.enc` |

### Runtime hints from logs (serverexample)

- Server auth tokens can be supplied by CLI or environment variables, per `ServerAuthManager` log message.
- Plugin and asset packs are loaded from the `mods` directory by default.
- The asset path watcher logs that a restart is required to reload assets (matches `--disable-file-watcher` flag behavior).

## CLI options (observed)

The following flags are present as string constants in `com.hypixel.hytale.server.core.Options`. Some include help text embedded in the binary; those are listed with descriptions. Others are listed without an embedded description and should be treated as provisional.

### Options with embedded descriptions

| Flag | Description (from embedded strings) |
|---|---|
| `--help` | Prints this message. |
| `--version` | Prints version information. |
| `--bare` | Runs the server bare (no world loading, no port binding, no directory creation; plugins may still load). |
| `--log` | Sets the logger level (comma-separated values). |
| `-b`, `--bind` | Port to listen on (default `5520`). |
| `-t`, `--transport` | Transport type (see `com.hypixel.hytale.server.core.io.transport.TransportType`). |
| `--prefab-cache` | Prefab cache directory for immutable assets. |
| `--disable-cpb-build` | Disables building of compact prefab buffers. |
| `--assets` | Asset directory (accepts dir or zip). Default: `../HytaleAssets`. |
| `--mods` | Additional mods directories. |
| `--accept-early-plugins` | Acknowledge that early plugins are unsupported and may be unstable. |
| `--early-plugins` | Additional early plugin directories to load from. |
| `--validate-assets` | Exit with error if any assets are invalid. |
| `--validate-prefabs` | Exit with error if any prefabs are invalid. |
| `--validate-world-gen` | Exit with error if default world gen is invalid. |
| `--shutdown-after-validate` | Automatically shutdown after validation. |
| `--generate-schema` | Generate schema, save into assets directory, then exit. |
| `--world-gen` | World generation directory. |
| `--migrations` | Migrations to run. |
| `--migrate-worlds` | Worlds to migrate. |
| `--boot-command` | Runs command on boot (multiple commands run synchronously in order). |
| `--auth-mode` | Authentication mode. |
| `--session-token` | Session token for Session Service API. |
| `--identity-token` | Identity token (JWT). |
| `--disable-asset-compare` | Disable asset comparison (exact behavior unspecified). |
| `--backup` | Enable backups (details via related flags). |
| `--backup-frequency` | Backup frequency in minutes. |
| `--backup-dir` | Backup directory. |
| `--backup-max-count` | Max number of backups to keep. |

### Other observed flags (no embedded descriptions)

`--allow-op`, `--client-pid`, `--disable-file-watcher`, `--disable-sentry`, `--event-debug`, `--force-network-flush`, `--owner-name`, `--owner-uuid`, `--singleplayer`, `--universe`, `--patchline`, `--environment`, `--release`

## Built-in plugins

The server bundles 46 top-level plugins declared in `manifests.json`, plus the NPC sub-plugins `Spawning` and `Flock`. This table is extracted directly from that manifest and includes declared dependencies.

| Name | Main class | Dependencies | Optional deps | Sub-plugins |
|---|---|---|---|---|
| Ambience | com.hypixel.hytale.builtin.ambience.AmbiencePlugin | Hytale:EntityModule | - | - |
| AssetEditor | com.hypixel.hytale.builtin.asseteditor.AssetEditorPlugin | Hytale:I18nModule, Hytale:ItemModule, Hytale:Model | - | - |
| Beds | com.hypixel.hytale.builtin.beds.BedsPlugin | Hytale:Mounts | - | - |
| BlockPhysics | com.hypixel.hytale.builtin.blockphysics.BlockPhysicsPlugin | Hytale:BlockTick, Hytale:BlockTypeModule | - | - |
| BlockSpawner | com.hypixel.hytale.builtin.blockspawner.BlockSpawnerPlugin | Hytale:BlockModule, Hytale:BlockStateModule, Hytale:I18nModule | - | - |
| BlockTick | com.hypixel.hytale.builtin.blocktick.BlockTickPlugin | Hytale:LegacyModule | - | - |
| BuilderTools | com.hypixel.hytale.builtin.buildertools.BuilderToolsPlugin | Hytale:EntityModule | - | - |
| Camera | com.hypixel.hytale.builtin.adventure.camera.CameraPlugin | Hytale:AssetModule, Hytale:DamageModule, Hytale:InteractionModule | - | - |
| CommandMacro | com.hypixel.hytale.builtin.commandmacro.MacroCommandPlugin | - | - | - |
| Crafting | com.hypixel.hytale.builtin.crafting.CraftingPlugin | Hytale:BlockStateModule, Hytale:EntityModule, Hytale:InteractionModule | - | - |
| CreativeHub | com.hypixel.hytale.builtin.creativehub.CreativeHubPlugin | Hytale:I18nModule, Hytale:Instances | - | - |
| CrouchSlide | com.hypixel.hytale.builtin.crouchslide.CrouchSlidePlugin | - | - | - |
| Deployables | com.hypixel.hytale.builtin.deployables.DeployablesPlugin | - | - | - |
| Farming | com.hypixel.hytale.builtin.adventure.farming.FarmingPlugin | Hytale:BlockPhysics, Hytale:BlockStateModule, Hytale:InteractionModule | - | - |
| Fluid | com.hypixel.hytale.builtin.fluid.FluidPlugin | Hytale:BlockTick, Hytale:LegacyModule | - | - |
| HytaleGenerator | com.hypixel.hytale.builtin.hytalegenerator.plugin.HytaleGenerator | - | - | - |
| Instances | com.hypixel.hytale.builtin.instances.InstancesPlugin | Hytale:BlockPhysics, Hytale:BlockStateModule, Hytale:I18nModule | - | - |
| LANDiscovery | com.hypixel.hytale.builtin.landiscovery.LANDiscoveryPlugin | - | - | - |
| Mantling | com.hypixel.hytale.builtin.mantling.MantlingPlugin | - | - | - |
| Memories | com.hypixel.hytale.builtin.adventure.memories.MemoriesPlugin | Hytale:NPC | - | - |
| Model | com.hypixel.hytale.builtin.model.ModelPlugin | - | - | - |
| Mounts | com.hypixel.hytale.builtin.mounts.MountPlugin | Hytale:DamageModule, Hytale:NPC | - | - |
| NPC | com.hypixel.hytale.server.npc.NPCPlugin | Hytale:AssetModule, Hytale:DamageModule, Hytale:EntityModule, Hytale:EntityStatsModule, Hytale:TagSet | Hytale:Objectives, Hytale:Path, Hytale:Teleport | Flock, Spawning |
| NPCCombatActionEvaluator | com.hypixel.hytale.builtin.npccombatactionevaluator.NPCCombatActionEvaluatorPlugin | Hytale:NPC | - | - |
| NPCEditor | com.hypixel.hytale.builtin.npceditor.NPCEditorPlugin | Hytale:AssetEditor, Hytale:NPC | - | - |
| NPCObjectives | com.hypixel.hytale.builtin.adventure.npcobjectives.NPCObjectivesPlugin | Hytale:NPC, Hytale:Objectives, Hytale:Spawning, Hytale:TagSet | - | - |
| NPCReputation | com.hypixel.hytale.builtin.adventure.npcreputation.NPCReputationPlugin | Hytale:NPC, Hytale:Reputation, Hytale:TagSet | - | - |
| NPCShop | com.hypixel.hytale.builtin.adventure.npcshop.NPCShopPlugin | Hytale:I18nModule, Hytale:NPC, Hytale:Shop | - | - |
| ObjectiveReputation | com.hypixel.hytale.builtin.adventure.objectivereputation.ObjectiveReputationPlugin | Hytale:Objectives, Hytale:Reputation | - | - |
| Objectives | com.hypixel.hytale.builtin.adventure.objectives.ObjectivePlugin | Hytale:BlockStateModule, Hytale:EntityModule, Hytale:I18nModule, Hytale:InteractionModule | - | - |
| ObjectiveShop | com.hypixel.hytale.builtin.adventure.objectiveshop.ObjectiveShopPlugin | Hytale:Objectives, Hytale:Shop | - | - |
| Parkour | com.hypixel.hytale.builtin.parkour.ParkourPlugin | Hytale:EntityModule | - | - |
| Path | com.hypixel.hytale.builtin.path.PathPlugin | Hytale:BuilderTools, Hytale:EntityModule | - | - |
| Portals | com.hypixel.hytale.builtin.portals.PortalsPlugin | Hytale:Ambience, Hytale:Instances, Hytale:NPC | - | - |
| Reputation | com.hypixel.hytale.builtin.adventure.reputation.ReputationPlugin | Hytale:I18nModule | - | - |
| SafetyRoll | com.hypixel.hytale.builtin.safetyroll.SafetyRollPlugin | - | - | - |
| Shop | com.hypixel.hytale.builtin.adventure.shop.ShopPlugin | Hytale:InteractionModule | - | - |
| ShopReputation | com.hypixel.hytale.builtin.adventure.shopreputation.ShopReputationPlugin | Hytale:Reputation, Hytale:Shop | - | - |
| SprintForce | com.hypixel.hytale.builtin.sprintforce.SprintForcePlugin | - | - | - |
| Stash | com.hypixel.hytale.builtin.adventure.stash.StashPlugin | Hytale:BlockStateModule, Hytale:ItemModule | - | - |
| TagSet | com.hypixel.hytale.builtin.tagset.TagSetPlugin | - | - | - |
| Teleport | com.hypixel.hytale.builtin.teleport.TeleportPlugin | Hytale:EntityModule, Hytale:I18nModule | - | - |
| Teleporter | com.hypixel.hytale.builtin.adventure.teleporter.TeleporterPlugin | Hytale:BlockStateModule, Hytale:Teleport | - | - |
| Weather | com.hypixel.hytale.builtin.weather.WeatherPlugin | Hytale:EntityModule | - | - |
| WorldGen | com.hypixel.hytale.builtin.worldgen.WorldGenPlugin | Hytale:EntityModule | - | - |
| WorldLocationCondition | com.hypixel.hytale.builtin.adventure.worldlocationcondition.WorldLocationConditionPlugin | - | - | - |

## Block migration data

The `migration/blocks/` folder contains JSON maps for block ID or block-state migrations across internal versions:

- `migration/blocks/0_1.json`
- `migration/blocks/1_2.json`
- `migration/blocks/2_3.json`
- `migration/blocks/3_4.json`

These appear to map older block names or rotated variants to updated identifiers, used by world migration routines.

---

## Asset Pack Layout (Sample Mods)

Observed in content packs (`.zip`) under `tmp/mods/`:

```
manifest.json
Common/
  Blocks/
  BlockTextures/
  Icons/
  Items/
  Resources/
  UI/
Server/
  Item/
    Block/Hitboxes/
    CustomConnectedBlockTemplates/
    Groups/
    Items/
    Recipes/
  Languages/en-US/
```

Not every pack includes every folder, but most use `Common/*` for art assets
and `Server/Item/*` for data assets and recipes.

Common item JSON keys seen in packs:

- `TranslationProperties`, `Icon`, `IconProperties`
- `Categories`, `Tags`, `Set`, `ResourceTypes`
- `Recipe`
- `BlockType` for placeable blocks

Common `BlockType` keys seen in packs:

- `Material`, `DrawType`, `Opacity`
- `CustomModel`, `CustomModelTexture`
- `HitboxType`, `VariantRotation`
- `BlockParticleSetId`, `BlockSoundSetId`
- `Gathering`, `Flags`, `Support`

Recipe keys vary by pack:

- `Input`, `BenchRequirement`
- `PrimaryOutput` or `OutputQuantity`
- `TimeSeconds` or `Seconds`

## See Also

- [plugins/](./plugins/) - Plugin development documentation
  - [Plugin API Reference](./plugins/HytaleServer-Plugin-API.md)
  - [Plugin Examples](./plugins/HytaleServer-Plugin-Examples.md)
