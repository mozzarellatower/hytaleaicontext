# Hytale server commands (local notes)

This server exposes commands through the console and (for players with permission) in chat. Commands are typically written with a leading slash and space-separated arguments.

## Where to type commands

- **Server console**: Type the command directly into the terminal running `HytaleServer.jar`. You can usually omit the leading slash here, but keeping it is fine.
- **In-game chat**: Type commands with a leading `/` (for example, `/auth login ...`). Only players with permission can run most commands.

## Basic command syntax

```
/<command> [arg1] [arg2] ...
```

- Arguments are space-separated.
- If an argument contains spaces, try quoting it with `"` or `'` (server implementations vary).
- If a command fails, check the server console log for the error message.

## Finding available commands

Because Hytale is built for modding, the command list depends on the core server version and any installed mods.

To discover commands:

1. Check the server console output on startup for modules/plugins and any registered commands.
2. Try a built-in help command if available (commonly `/help` or `/commands`).
3. Check mod documentation or any `manifest.json`/docs bundled with the mod.
4. If available, try `/dumpcommands` to export a full list (class `DumpCommandsCommand` exists in the jar).

## Auth command (confirmed from logs)

The log indicates authentication uses a command like:

```
/auth login <token-or-credentials>
```

This appears in the server log message: "Use /auth login to authenticate".

## Permissions

Permissions are stored in `serverexample/permissions.json`. In this sample, the `OP` group has `"*"` (all permissions). If you add players or groups there, restart the server to apply changes.

From local sample mods (not included in this repo), permissions APIs appear to include `PermissionsModule` and helpers like `setPermissionGroup(s)` (names observed in class strings). Use these as search anchors when exploring the server JAR or decompiled sources.

## Mods and custom commands

Mods can register their own commands. Look for mod folders under `serverexample/mods` and any mod-specific config or docs. If a mod has no manifest or docs, you can still inspect the server log on startup to see whether it loaded and what commands it adds.

### Patterns seen in sample mods (heuristic)

From string scans of local sample `.jar` mods (not included in this repo):

- Commands are registered through `CommandRegistry` and implemented with `AbstractCommand` or `AbstractAsyncCommand`.
- Command execution uses `CommandContext` and `CommandSender`.
- Some mods build UI-driven commands via `UICommandBuilder`.
- Argument helpers like `OptionalArg` appear in command classes.
- Several mods use `CommandBuffer` (likely for batched or scripted command execution).

These are class-level observations, not confirmed command names. Use `/commands` or logs to discover actual labels.

### Observed mod commands (decompiled local samples)

The following command labels and permission gates are taken from decompiled local mods. Treat as examples and verify on your server build.

- AdminUI: `admin` command plus shortcut labels `wl`, `whitelists`, `m`, `mute`, `b`, `bans`, `p`, `players`, `w`, `warps`, `bk`, `backup`, `st`, `stats`.
  Uses `AbstractAsyncCommand`, `setPermissionGroups("OP")`, and `setPermissionGroup(GameMode.Creative)` on shortcuts.
- AdvancedItemInfo: `advancedinfo` with aliases `aii`, `iteminfo`; uses `AbstractCommand`, `setPermissionGroup(GameMode.Adventure)`,
  and an optional arg `s` ("Default Search") via `SingleArgumentType` (examples include `iron`, `stone`).
- BetterModlist: `modlist`; uses `AbstractAsyncCommand` and `setPermissionGroup(GameMode.Adventure)`.
- Hybrid: `hybrid`; uses `setPermissionGroup(GameMode.Creative)`.

Permissions module usage observed in local mods:

- Some mods call `PermissionsModule.get().getGroupsForUser(uuid)` and check for `"OP"` group membership for access checks.

### Observed mod behaviors (non-command)

- LuckyMining registers an `EntityEventSystem` for `BreakBlockEvent` and uses
  `ParticleUtil.spawnParticleEffect` plus `SoundUtil.playSoundEvent2dToPlayer` for
  feedback.
- AdminUI uses `registerGlobal(PlayerConnectEvent)` for player tracking and
  `registerGlobal(PlayerChatEvent)` to implement mute checks, and listens to
  `LoadedAssetsEvent<ModelAsset>` for model lists.
- ThePickaxesPlaceTorches registers a custom `Interaction` codec and uses
  `BlockPlaceUtils.placeBlock` to place torches based on `InteractionContext`.
- Hybrid registers `EntityEventSystem` handlers for `BreakBlockEvent`,
  `PlaceBlockEvent`, and `UseBlockEvent.Pre`, and registers global
  `PlayerReadyEvent` plus `LoadedAssetsEvent<Item>`/`LoadedAssetsEvent<BlockType>`
  listeners (classes `HybridBreakBlockEventSystem`, `HybridPlaceBlockEventSystem`,
  `HybridUseBlockEventSystem`, `_RegisterHybridEvents`).
- InfiniteMana registers `PlayerTickCallback.ON_PLAYER_TICK` and sets
  `HybridEntityStatType.MANA` to max each tick via `EntityStatFunctions`, gated by
  config flag `givePlayersInfiniteMana` (classes `PlayerCallbacks`,
  `ConfigHandler`).
- TreeHarvester registers `EntityBreakBlockCallback.ENTITY_BREAK_BLOCK` and, on
  log breaks, gathers connected tree blocks to drop logs/leaves, optionally
  replants saplings, and optionally decreases tool durability (classes
  `EntityCallbacks`, `Util`, `ConfigHandler`).
- Ymmersive Melodies registers `Interaction` codec
  `Ymmersive_Melodies_Melody_Playback` (plays note sound events in `tick0`) and
  `OpenCustomUIInteraction` page codec `Ymmersive_Melodies_Selection` for custom
  UI (classes `YmmersiveMelodies`, `MelodyPlaybackInteraction`,
  `MelodySelectionSupplier`).
- Spellbook registers `Interaction` codecs `DarkhaxSpellbookWarpHome` (teleport
  to nearest respawn) and `DarkhaxSpellbookBlockPush` (applies `Velocity`), plus
  `BlockState` types `ItemGeneratorState` and `ConveyorState` (classes
  `Spellbook`, `WarpHomeInteraction`, `BlockPushInteraction`).

## Core commands (inferred from the server jar)

I scanned `serverexample/Server/HytaleServer.jar` for command classes. The exact command labels and arguments can differ from class names, so treat this as a likely set and use `/help` or `/commands` to confirm syntax.

Examples below are best-effort guesses based on class names. If one fails, try `/help <command>` or `/commands` to see the real shape.

### Utility and meta

- `/help`
- `/commands`
- `/version`
- `/ping`
- `/backup`
- `/notify`
- `/sleep` (plus offset/test variants)

Examples:

```
/help
/help teleport
/commands
/dumpcommands
/version
/ping
/backup
/sleep
/sleep offset 120
```

### Server/admin

- `/auth` (subcommands: `login`, `loginbrowser`, `logindevice`, `logout`, `status`, `select`, `cancel`, `persistence`)
- `/stop`
- `/kick`
- `/who`
- `/maxplayers`

Examples:

```
/auth login <token-or-credentials>
/auth loginbrowser
/auth logindevice
/auth status
/auth select <profile>
/auth persistence set <on|off>
/auth logout
/auth cancel
/kick <player> [reason]
/who
/maxplayers 50
/stop
```

### Player and inventory

- `/gamemode`
- `/give`
- `/givearmor`
- `/inventory` (clear/see/item/backpack)
- `/kill`
- `/damage`
- `/hide` (show/hide players)
- `/respawn`
- `/reset`
- `/viewradius` (get/set)
- `/whereami`
- `/whoami`
- `/sudo`

Examples:

```
/gamemode <adventure|creative|survival> [player]
/give <player> <item-id> [count]
/givearmor <player> <set-id>
/inventory clear [player]
/inventory see <player>
/kill [player]
/damage <amount> [player]
/hide <player>
/hide show <player>
/respawn [player]
/reset [player]
/viewradius get [player]
/viewradius set <radius> [player]
/whereami [player]
/whoami
/sudo <player> <command...>
```

### Teleport and warp

- `/teleport` (to player/coords, back, forward, home, top, world, all)
- `/spawn` (set/go)
- `/warp` (set/go/list/remove/reload)

Examples:

```
/teleport <player>
/teleport <x> <y> <z>
/teleport <player> <x> <y> <z>
/teleport back
/teleport forward
/teleport home
/teleport top
/teleport world <world-id>
/teleport all <player>
/spawn
/spawn set
/warp list
/warp set <name>
/warp go <name>
/warp remove <name>
/warp reload
```

### World, weather, and lighting

- `/weather` (get/set/reset)
- `/lighting` (get/info/invalidate/send/toggle)

Examples:

```
/weather get
/weather set <weather-id>
/weather reset
/lighting get
/lighting info
/lighting invalidate
/lighting send global
/lighting send local
/lighting toggle
```

### Gameplay systems (likely admin/debug)

- `/objective` (start/complete/history/panel/markers)
- `/reputation` (add/set/rank/value)
- `/memories` (capacity/level/unlock/clear)
- `/npc` (spawn/debug/path/role/blackboard/etc.)
- `/spawn` (markers/beacons/suppression/stats)
- `/instances` (edit/load/list/migrate)
- `/model` (set/reset)
- `/camera` (player camera demos/effects)
- `/mount` (mount/dismount)
- `/parkour` (checkpoint add/remove/reset)
- `/ambience`
- `/portals`

Examples:

```
/objective start <objective-id> [player]
/objective complete <objective-id> [player]
/reputation add <faction-id> <amount> [player]
/reputation set <faction-id> <amount> [player]
/memories clear [player]
/memories unlock <memory-id> [player]
/npc spawn <npc-id>
/npc debug toggle
/spawn markers add
/spawn suppression dump
/instances edit <instance-id>
/instances list
/model set <model-id> [player]
/model reset [player]
/camera demo activate
/camera demo deactivate
/mount [mount-id]
/dismount [player]
/parkour checkpoint add
/parkour checkpoint remove
/ambience clear
/portals leave
```

### Builder tools (mostly creative/debug)

- `/brushconfig`
- `/prefabedit`
- `/select`
- `/copy` `/paste` `/undo` `/redo` (and other builder tool variants)

Examples:

```
/brushconfig list
/brushconfig load <name>
/prefabedit create <name>
/prefabedit save
/select
/copy
/paste
/undo [count]
/redo [count]
```
