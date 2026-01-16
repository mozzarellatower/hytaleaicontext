# Structures and Prefabs Guide (Reverse-Engineered + Site Cross-Check)

This guide focuses on prefab/structure workflows exposed by the server tooling and
the public docs at https://hytalemodding.dev/en/docs/guides/prefabs.

## Overview

Hytale prefabs are structure files saved and loaded by the server. The built-in
commands focus on saving, loading, and editing prefabs inside a dedicated editing
world.

## Command Summary

Source: https://hytalemodding.dev/en/docs/guides/prefabs

There are two main commands for prefabs: `/prefab` and `/editprefab`. Use
`--help` on any subcommand (example: `/prefab save --help`) or `/help` in-game
to inspect arguments and flags.

### /prefab

This command interacts with prefab files on disk.

| Subcommand | Description |
|---|---|
| `save` | Saves the prefab to the file system. |
| `load` | Loads the prefab into the game. |
| `delete` | Deletes the prefab from the file system. |
| `list` | Lists all prefabs in the file system. |

### /editprefab

This command creates or edits the physical structure in an editing world.

| Subcommand | Description |
|---|---|
| `exit` | Exits the current prefab editing world. |
| `load` | Creates a new prefab editing world to paste an existing prefab into. |
| `new` | Creates a new prefab editing world to create a new prefab from scratch. |
| `select` | Selects the area of the prefab the user is looking at (within 200 blocks). |
| `save` | Saves the current prefab using the existing area or selected area. |
| `saveui` | Opens the save UI for the current world, allowing the user to interact with all prefabs in the world. |
| `kill` | Despawns all entities in the currently selected prefab. |
| `saveas` | Saves the selected prefab into a new file. |
| `setbox` | Sets the bounding box of the currently selected prefab. |
| `info` | Shows general information about the selected prefab. |
| `tp` | Opens the teleport UI to teleport to a prefab in the editing world. |
| `modified` | Lists all modified prefabs with unsaved changes. |

## Known Issues (from hytalemodding.dev)

Source: https://hytalemodding.dev/en/docs/guides/prefabs

- After saving edits to an existing prefab, the prefab may not reflect changes
  until you exit and re-enter the world.
- When pasting a prefab, the material preview can be inaccurate; toggling the
  material view (key `t`) usually fixes one of the views.
- `/prefab delete` can error with `Assert not in thread`.

## Related Internals (Jar Dissection)

See `HytaleServer-Dissection.md` for package-level references to prefab tooling:

- `com.hypixel.hytale.server.core.prefab`
- `com.hypixel.hytale.server.core.modules.prefabspawner`
- `com.hypixel.hytale.builtin.buildertools.prefabeditor.ui`

## JAR-Backed Prefab APIs

The server JAR exposes concrete prefab APIs you can target from plugins:

- `com.hypixel.hytale.server.core.prefab.PrefabStore`
  - Separate namespaces for server, worldgen, and asset-pack prefabs.
  - Read/write via `getServerPrefab`, `getWorldGenPrefab`, `getAssetPrefab` and
    `saveServerPrefab`, `saveWorldGenPrefab`, `saveAssetPrefab`.
  - Asset-pack helpers include `getAssetPrefabsPathForPack` and
    `findAssetPackForPrefabPath`.
- `com.hypixel.hytale.server.core.prefab.selection.standard.BlockSelection`
  - Core structure buffer with block/fluid/entity counts and placement helpers.
  - Supports `place(...)`, `rotate(...)`, `flip(...)`, `relativize(...)`, and
    comparison helpers like `matches(...)`.
  - Masked placement uses `com.hypixel.hytale.server.core.prefab.selection.mask.BlockMask`.
- `com.hypixel.hytale.server.core.prefab.selection.mask.BlockMask`
  - Parses masks from strings (`parse(...)`) and supports invert/exclude logic.
- `com.hypixel.hytale.server.core.prefab.event.PrefabPasteEvent`
  - Cancellable ECS event with `getPrefabId()` and `isPasteStart()`.
- `com.hypixel.hytale.server.core.prefab.event.PrefabPlaceEntityEvent`
  - ECS event for entity placement with `getPrefabId()` and `getHolder()`.

## Worldgen + Prefab Integration

Classes that show prefabs are used during world generation:

- `com.hypixel.hytale.server.worldgen.chunk.populator.PrefabPopulator`
- `com.hypixel.hytale.server.worldgen.cave.CavePrefabPlacement`
- `com.hypixel.hytale.server.worldgen.container.PrefabContainer`
- `com.hypixel.hytale.server.worldgen.container.UniquePrefabContainer`

Worldgen providers are defined under
`com.hypixel.hytale.server.core.universe.world.worldgen.provider` with
`FlatWorldGenProvider`, `VoidWorldGenProvider`, and `DummyWorldGenProvider`
implementing `IWorldGenProvider`.

## Asset Pack Layout (Sample Mods)

Examples from local content packs (`.zip`) (not included in this repo) show a common layout:

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
and `Server/Item/*` for data assets.

### Data Asset Patterns (JSON)

Observed in sample mods:

- Item definitions live under `Server/Item/Items/.../*.json`.
- Hitbox definitions live under `Server/Item/Block/Hitboxes/.../*.json` with
  a simple `Boxes` array.
- Connected block templates live under
  `Server/Item/CustomConnectedBlockTemplates/*.json` and use keys like
  `ConnectsToOtherMaterials`, `MaterialName`, `DefaultShape`, `Shapes`.
- Group listings live under `Server/Item/Groups/*.json` with keys like `Blocks`.
- Recipes appear under `Server/Item/Recipes/.../*.json`.

### Common Item Keys

Typical item JSON keys observed in multiple packs:

- `TranslationProperties` (string keys for language files)
- `Icon`, `IconProperties`
- `PlayerAnimationsId`
- `Categories`, `Tags`, `Set`, `ResourceTypes`
- `BlockType` (for placeable blocks)
- `Recipe`

`BlockType` commonly includes:

- `Material`, `DrawType`, `Opacity`
- `CustomModel`, `CustomModelTexture`
- `HitboxType`, `VariantRotation`
- `BlockParticleSetId`, `BlockSoundSetId`
- `Gathering`, `Flags`, `Support`

### Recipe Shape Examples

Recipes vary by pack, but the following keys appear:

- `Input` (list of resource types or items with `Quantity`)
- `BenchRequirement` (crafting bench id/type/categories)
- `PrimaryOutput` or `OutputQuantity`
- `TimeSeconds` or `Seconds`

### Practical Content Pack Flow (Non-verbatim)

Add a new block:
1. Define the item JSON under `Server/Item/Items/...` with a `BlockType`.
2. Add textures/models under `Common/BlockTextures/` or `Common/Blocks/`.
3. Add an icon under `Common/Icons/ItemsGenerated/`.
4. Add localization keys under `Server/Languages/en-US/`.
5. Optionally add `Recipes` and `Groups` entries.

Add a usable item with interaction:
1. Add the item JSON with an `Interactions` map.
2. Define the interaction JSON in `Server/Item/Interactions/Item/`.
3. Map root interactions in `Server/Item/RootInteractions/` if needed.
4. Implement the interaction handler in a plugin.

Add custom UI:
1. Add `.ui` files under `Common/UI/Custom/Pages/...`.
2. Add UI textures under `Common/UI/Custom/...`.
3. Load and show the UI from plugin code in response to interactions or commands.
