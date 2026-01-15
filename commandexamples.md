# Custom command examples (template)

These examples are intentionally generic so they can be adapted to the official Hytale SDK. Replace API names and types with the ones in your SDK version.

## Example 1: Simple ping command

Goal: add `/myping` that replies with a latency message.

Pseudo-code:

```java
public final class MyMod {
  public void onEnable(Server server) {
    server.getCommands().register("myping", (ctx) -> {
      Player player = ctx.getPlayer();
      int ms = player.getPingMs();
      player.sendMessage("Pong: " + ms + "ms");
    });
  }
}
```

In-game:

```
/myping
```

## Example 2: Give item command

Goal: add `/givecoins <player> <amount>`.

```java
server.getCommands().register("givecoins", (ctx) -> {
  String targetName = ctx.arg(0);
  int amount = ctx.argInt(1);

  Player target = server.getPlayerByName(targetName);
  if (target == null) {
    ctx.replyError("Player not found: " + targetName);
    return;
  }

  Coins coins = target.getInventory().getCoins();
  coins.add(amount);
  ctx.replySuccess("Added " + amount + " coins to " + targetName);
});
```

In-game:

```
/givecoins Alice 250
```

## Example 3: Teleport command with permissions

Goal: add `/tpto <player>` requiring `mymod.tpto`.

```java
server.getCommands().register("tpto", (ctx) -> {
  if (!ctx.hasPermission("mymod.tpto")) {
    ctx.replyError("No permission.");
    return;
  }

  Player sender = ctx.getPlayer();
  Player target = server.getPlayerByName(ctx.arg(0));
  if (target == null) {
    ctx.replyError("Player not found.");
    return;
  }

  sender.teleport(target.getLocation());
  ctx.replySuccess("Teleported to " + target.getName());
});
```

In-game:

```
/tpto Bob
```

## Example 4: Config-driven message command

Goal: add `/motd` that reads text from a config file.

Config (example `config.json`):

```json
{
  "motd": "Welcome to the server!"
}
```

Pseudo-code:

```java
public final class MyMod {
  private String motd;

  public void onEnable(Server server) {
    this.motd = loadConfig("config.json").getString("motd");

    server.getCommands().register("motd", (ctx) -> {
      ctx.replyInfo(motd);
    });
  }
}
```

In-game:

```
/motd
```

## Example 5: Event listener

Goal: announce when a player joins.

```java
server.getEvents().onPlayerJoin((event) -> {
  Player p = event.getPlayer();
  server.broadcast(p.getName() + " joined the server.");
});
```

## Example 6: Cooldown command

Goal: add `/roll` with a 10 second cooldown per player.

```java
Map<UUID, Long> lastRoll = new HashMap<>();

server.getCommands().register("roll", (ctx) -> {
  Player p = ctx.getPlayer();
  long now = System.currentTimeMillis();
  long last = lastRoll.getOrDefault(p.getUuid(), 0L);

  if (now - last < 10_000) {
    ctx.replyError("Cooldown: try again in " + ((10_000 - (now - last)) / 1000) + "s");
    return;
  }

  int roll = 1 + (int)(Math.random() * 100);
  lastRoll.put(p.getUuid(), now);
  ctx.replySuccess("You rolled " + roll);
});
```

In-game:

```
/roll
```

## Example 7: Admin-only reload

Goal: add `/mymod reload` to reload config at runtime.

```java
server.getCommands().register("mymod", (ctx) -> {
  String sub = ctx.arg(0);

  if ("reload".equals(sub)) {
    if (!ctx.hasPermission("mymod.admin")) {
      ctx.replyError("No permission.");
      return;
    }

    reloadConfig();
    ctx.replySuccess("Reloaded config.");
  }
});
```

In-game:

```
/mymod reload
```

## Notes

- Command parsing helpers (`ctx.arg`, `ctx.replyInfo`, etc.) are placeholders.
- Replace types like `Server`, `Player`, and `Event` with actual SDK classes.
- Use `/help` or `/commands` in-game to confirm actual command registration.
