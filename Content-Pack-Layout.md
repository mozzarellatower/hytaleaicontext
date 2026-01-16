# Content Pack Layout (From Sample ZIP Mods)

This guide summarizes common folder conventions found in sample ZIP mods under
`tmp/mods/`. It is based on file layout only (no decompilation) and is intended
as a practical reference.

## Top-Level Layout

Most ZIP mods follow this structure:

```
<mod>.zip
├── manifest.json
├── Common/
└── Server/
```

## Common/ (Client-Visible Assets)

Observed patterns:

- `Common/Blocks/<Namespace>/...`
  - `.blockymodel` and `.blockyanim` files
  - `.png` textures referenced by models
- `Common/Blocks/<Namespace>/Animations/*.blockyanim`
  - Block animation clips (e.g., blinds/curtains)
- `Common/BlockTextures/<Namespace>/*.png`
  - Tile textures for block variants
- `Common/Icons/ItemsGenerated/*.png`
  - Item icons (auto-generated or exported)
- `Common/Icons/CraftingCategories/.../*.png`
  - Crafting category icons (bench tabs)
- `Common/Items/*.png`
  - Standalone item textures (observed in Soulstones)
- `Common/Resources/<Type>/*.blockymodel`
  - Resource models for non-block items (e.g., charms)
- `Common/UI/Custom/*.png`
  - UI texture atlases for custom interfaces

### Common Variants (Examples)

- `Common/Blocks/Decorative_Sets/Paintings/<size>/...` (PixelPaintings)
- `Common/Blocks/Mcw_Windows/Animations/*.blockyanim` (McwHyWindows)
- `Common/BlockTextures/McwPaths/*.png` (McwHyPaths)
- `Common/Resources/Charms/*.blockymodel` (WaybackCharm)

## Server/ (Gameplay Data)

Observed patterns:

- `Server/Item/Items/.../*.json`
  - Item definitions (often nested by namespace and category)
- `Server/Item/Block/Hitboxes/.../*.json`
  - Block hitbox definitions
- `Server/Item/Groups/*.json`
  - Item groupings for catalogs or UI
- `Server/Item/Recipes/.../*.json`
  - Crafting and conversion recipes
- `Server/Languages/<locale>/*.lang`
  - Item and UI localization strings

### Server Variants (Examples)

- `Server/Item/Items/Furniture/Paintings/<size>/...` (PixelPaintings)
- `Server/Item/Items/Spawners/...` (Soulstones)
- `Server/Item/Recipes/Crafting/...` (Soulstones)
- `Server/Item/Recipes/Crystals/...` (Soulstones)

## Example Tree (Non-Verbatim)

```
MyMod.zip
├── manifest.json
├── Common/
│   ├── Blocks/MyMod/Thing.blockymodel
│   ├── Blocks/MyMod/Thing.png
│   ├── Icons/ItemsGenerated/MyMod_Thing.png
│   └── UI/Custom/MyMod_UI.png
└── Server/
    ├── Item/Items/MyMod_Thing.json
    ├── Item/Block/Hitboxes/MyMod_Thing.json
    ├── Item/Recipes/MyMod_Thing.json
    └── Languages/en-US/items.lang
```

## ZIP Mods Referenced

Layout patterns above are drawn from these ZIP mods:

- `tmp/mods/CobbleGens-2026.1.12-32469.zip`
- `tmp/mods/McwHyCarpets_1.0.2.zip`
- `tmp/mods/McwHyPaths_1.1.0.zip`
- `tmp/mods/McwHyWindows_1.0.0.zip`
- `tmp/mods/PixelPaintings-2026.1.12-31659.zip`
- `tmp/mods/Soulstones-2026.1.12-44389.zip`
- `tmp/mods/WaybackCharm-2026.1.5-17015.zip`

## Notes

- Naming patterns often use a `Namespace_ItemName` prefix in filenames.
- Some packs include custom UI atlases in `Common/UI/Custom` (e.g., benches).
- Folder depth varies by mod; multiple tiers of subfolders are common.
- Observed file extensions: `.json`, `.png`, `.blockymodel`, `.blockyanim`, `.lang`.
