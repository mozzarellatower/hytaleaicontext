# Custom plugins (modding overview)

This is a practical, engine-agnostic guide for writing your own Hytale server plugins. It avoids hardcoding API details because the official SDK and APIs can change by build. Use this as a checklist and adapt it to the SDK docs that match your server jar.

## 1) Get the right SDK

- Find the SDK that matches your server build/version.
- Prefer the official modding docs for the exact API names.
- If you have a modding template or sample project from the SDK, start there.

## 2) Typical plugin structure

Most Hytale-style mods/plugins follow a similar layout:

- A `manifest.json` (or equivalent) describing the mod: name, version, entry point.
- A compiled artifact (often a `.jar`) placed in `serverexample/mods`.
- Optional assets/resources bundled or placed alongside the mod.

Example layout (generic):

```
mods/
  MyMod/
    manifest.json
    MyMod.jar
    assets/
      ...
```

## 3) Common components to implement

- **Entry point**: class or module loaded by the server.
- **Lifecycle hooks**: enable/disable callbacks to register and unregister features.
- **Command registration**: add your custom commands.
- **Event listeners**: react to player join, block break, chat, etc.
- **Config**: load your JSON/YAML config on startup.
- **Permissions**: define permission keys so server admins can control access.

## 4) Installation workflow

1. Build the mod with the SDK toolchain.
2. Drop the output into `serverexample/mods`.
3. Restart the server (many servers do not hot-reload code).
4. Verify in logs that your mod loaded and your commands registered.

## 5) Permissions and access control

- Add permission keys to your mod (e.g., `mymod.admin`, `mymod.use`).
- Grant them to groups in `serverexample/permissions.json`.
- Restart after permission changes.

## 6) Logging and debugging

- Log on enable and when commands run.
- If a command is not found, check the server log for load errors.
- Keep a small debug command that prints internal state.

## 7) Server-safe design tips

- Validate user input in commands.
- Avoid heavy work on the main thread; queue or schedule background tasks.
- Guard against null/invalid player references.

## 8) Next steps

If you can provide the SDK docs or a sample mod template, I can write a version of this file with exact API names, code, and build steps that match your server.
