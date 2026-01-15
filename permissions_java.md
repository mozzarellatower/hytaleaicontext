# Setting permissions in Java (modding guide)

This is a generic Java modding guide for permission checks and registration. Replace the API calls with the ones from your Hytale SDK version.

## 1) Define permission keys

Pick stable, namespaced keys so server admins can grant access:

- `mymod.use`
- `mymod.admin`
- `mymod.command.reload`

## 2) Register permissions (if the SDK supports it)

Some modding SDKs let you register permission metadata (name, description, default). If available, do it during plugin enable:

```java
public void onEnable(Server server) {
  Permissions perms = server.getPermissions();
  perms.register("mymod.use", "Use MyMod commands", PermissionDefault.FALSE);
  perms.register("mymod.admin", "Admin access for MyMod", PermissionDefault.OP);
}
```

If the SDK does not support registration, just use the keys consistently and document them.

## 3) Gate commands with permissions

Check permission keys in your command handlers:

```java
server.getCommands().register("mymod", (ctx) -> {
  if (!ctx.hasPermission("mymod.use")) {
    ctx.replyError("No permission.");
    return;
  }

  // Command logic...
});
```

## 4) Gate events and features

Use the same permission keys to protect features outside commands:

```java
server.getEvents().onPlayerJoin((event) -> {
  Player p = event.getPlayer();
  if (p.hasPermission("mymod.admin")) {
    p.sendMessage("Admin features enabled.");
  }
});
```

## 5) Add permissions to groups on the server

In this workspace, server permissions are stored in `serverexample/permissions.json`.

Example group config:

```json
{
  "groups": {
    "Default": [
      "mymod.use"
    ],
    "OP": [
      "*"
    ],
    "MyModAdmin": [
      "mymod.admin",
      "mymod.command.reload"
    ]
  }
}
```

Then assign a player UUID to the group:

```json
{
  "users": {
    "player-uuid-here": {
      "groups": ["MyModAdmin"]
    }
  }
}
```

Restart the server after changing the file.

## 6) Provide a permissions reference

Add a short section to your mod README listing permission keys and what they do. This reduces support requests and makes admin setup easy.

## 7) Common mistakes

- Checking permissions only client‑side instead of server‑side.
- Using inconsistent keys across commands and features.
- Forgetting to update `permissions.json` and restart.

## Next step

If you share the exact SDK/API, I can rewrite this with the real classes, method names, and any registration mechanics it provides.
