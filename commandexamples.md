# Custom command examples (JAR-style patterns)

These examples use the JAR command system (`AbstractCommand`, `CommandRegistry`,
`CommandContext`, `ArgTypes`). Replace placeholder module calls with the APIs
for your server build.

Registration (JAR):

```java
import com.hypixel.hytale.server.core.command.system.CommandRegistry;

CommandRegistry registry = ...; // Obtain from your plugin entry point.
registry.registerCommand(new PingCommand());
```

## Example 1: Simple ping command

Goal: add `/myping` that replies with a latency message.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import java.util.concurrent.CompletableFuture;

public final class PingCommand extends AbstractCommand {
  public PingCommand() {
    super("myping", "Replies with latency");
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    if (!context.isPlayer()) {
      context.sendMessage(Message.raw("Player-only command."));
      return CompletableFuture.completedFuture(null);
    }

    // TODO: replace with your ping accessor.
    context.sendMessage(Message.raw("Pong"));
    return CompletableFuture.completedFuture(null);
  }
}
```

In-game:

```
/myping
```

## Example 2: Give coins command

Goal: add `/givecoins <player> <amount>`.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.concurrent.CompletableFuture;

public final class GiveCoinsCommand extends AbstractCommand {
  private final RequiredArg<PlayerRef> targetArg;
  private final RequiredArg<Integer> amountArg;

  public GiveCoinsCommand() {
    super("givecoins", "Give coins to a player");
    targetArg = withRequiredArg("target", "Target player", ArgTypes.PLAYER_REF);
    amountArg = withRequiredArg("amount", "Amount", ArgTypes.INTEGER);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    PlayerRef target = targetArg.get(context);
    int amount = amountArg.get(context);

    // TODO: apply coins via your inventory/economy module.
    context.sendMessage(Message.raw("Queued " + amount + " coins for " + target));
    return CompletableFuture.completedFuture(null);
  }
}
```

In-game:

```
/givecoins Alice 250
```

## Example 3: Teleport command with permissions

Goal: add `/tpto <player>` requiring `mymod.tpto`.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.concurrent.CompletableFuture;

public final class TpToCommand extends AbstractCommand {
  private final RequiredArg<PlayerRef> targetArg;

  public TpToCommand() {
    super("tpto", "Teleport to a player");
    requirePermission("mymod.tpto");
    targetArg = withRequiredArg("target", "Target player", ArgTypes.PLAYER_REF);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    if (!context.isPlayer()) {
      context.sendMessage(Message.raw("Player-only command."));
      return CompletableFuture.completedFuture(null);
    }

    PlayerRef target = targetArg.get(context);
    // TODO: teleport the sender to the target via your world/teleport module.
    context.sendMessage(Message.raw("Teleporting to " + target));
    return CompletableFuture.completedFuture(null);
  }
}
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

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import java.util.concurrent.CompletableFuture;

public final class MotdCommand extends AbstractCommand {
  private final String motd;

  public MotdCommand(String motd) {
    super("motd", "Show the message of the day");
    this.motd = motd;
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    context.sendMessage(Message.raw(motd));
    return CompletableFuture.completedFuture(null);
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
import com.hypixel.hytale.event.EventRegistry;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.event.events.player.PlayerConnectEvent;

EventRegistry events = ...; // From your plugin base, for example getEventRegistry().
events.registerGlobal(PlayerConnectEvent.class, event -> {
  Player player = event.getPlayer();
  player.sendMessage(Message.raw("Welcome " + player.getDisplayName()));
});
```

## Example 6: Cooldown command

Goal: add `/roll` with a 10 second cooldown per sender.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.CompletableFuture;

public final class RollCommand extends AbstractCommand {
  private final Map<UUID, Long> lastRoll = new HashMap<>();

  public RollCommand() {
    super("roll", "Roll a random number");
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    UUID senderId = context.sender().getUuid();
    long now = System.currentTimeMillis();
    long last = lastRoll.getOrDefault(senderId, 0L);

    if (now - last < 10_000) {
      long waitMs = 10_000 - (now - last);
      context.sendMessage(Message.raw("Cooldown: try again in " + (waitMs / 1000) + "s"));
      return CompletableFuture.completedFuture(null);
    }

    int roll = 1 + (int) (Math.random() * 100);
    lastRoll.put(senderId, now);
    context.sendMessage(Message.raw("You rolled " + roll));
    return CompletableFuture.completedFuture(null);
  }
}
```

In-game:

```
/roll
```

## Example 7: Admin-only reload

Goal: add `/mymod reload` to reload config at runtime.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import java.util.concurrent.CompletableFuture;

public final class MyModCommand extends AbstractCommand {
  private final RequiredArg<String> actionArg;

  public MyModCommand() {
    super("mymod", "Admin commands for MyMod");
    actionArg = withRequiredArg("action", "Action", ArgTypes.STRING);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    String action = actionArg.get(context);
    if (!"reload".equalsIgnoreCase(action)) {
      context.sendMessage(Message.raw("Usage: /mymod reload"));
      return CompletableFuture.completedFuture(null);
    }

    if (!context.sender().hasPermission("mymod.admin")) {
      context.sendMessage(Message.raw("No permission."));
      return CompletableFuture.completedFuture(null);
    }

    // TODO: reload your config.
    context.sendMessage(Message.raw("Reloaded config."));
    return CompletableFuture.completedFuture(null);
  }
}
```

In-game:

```
/mymod reload
```

## Example 8: Multi-level command with subcommands (JAR-style)

Goal: add `/bw setup addteam red` with subcommand routing.

Registered-arg approach (subcommand owns its args):

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.OptionalArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import java.util.concurrent.CompletableFuture;

public final class BwCommand extends AbstractCommand {
  public BwCommand() {
    super("bw", "BedWars");
    addSubCommand(new SetupCommand());
  }
}

private static final class SetupCommand extends AbstractCommand {
  private final RequiredArg<String> actionArg;
  private final OptionalArg<String> teamArg;

  private SetupCommand() {
    super("setup", "Arena setup");
    actionArg = withRequiredArg("action", "Action", ArgTypes.STRING);
    teamArg = withOptionalArg("team", "Team id", ArgTypes.STRING);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    String action = actionArg.get(context);
    String team = teamArg.provided(context) ? teamArg.get(context) : "";
    // Dispatch to your setup handler.
    return CompletableFuture.completedFuture(null);
  }
}
```

Manual-parse fallback when you need nested routing under `setup`:

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import java.util.concurrent.CompletableFuture;

private static final class SetupCommand extends AbstractCommand {
  private SetupCommand() {
    super("setup", "Arena setup");
    setAllowsExtraArguments(true);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    String[] args = argsAfter(context, 1); // Skip the "setup" token; adjust if needed.
    // Parse args and dispatch (for example, "addteam red").
    return CompletableFuture.completedFuture(null);
  }
}
```

## Example 9: Transfer player to another server (ClientReferral)

Goal: add `/transfer <host> <port> [data]` to send a client referral packet.
Referral data is limited to 4096 bytes in the JAR.

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.OptionalArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.CompletableFuture;

public final class TransferCommand extends AbstractCommand {
  private final RequiredArg<String> hostArg;
  private final RequiredArg<Integer> portArg;
  private final OptionalArg<String> dataArg;

  public TransferCommand() {
    super("transfer", "Send a player to another server");
    hostArg = withRequiredArg("host", "Target host", ArgTypes.STRING);
    portArg = withRequiredArg("port", "Target port", ArgTypes.INTEGER);
    dataArg = withOptionalArg("data", "Referral data", ArgTypes.STRING);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    if (!context.isPlayer()) {
      context.sendMessage(Message.raw("Player-only command."));
      return CompletableFuture.completedFuture(null);
    }

    String host = hostArg.get(context);
    int port = portArg.get(context);
    byte[] data = dataArg.provided(context)
        ? dataArg.get(context).getBytes(StandardCharsets.UTF_8)
        : null;

    Ref<EntityStore> ref = context.senderAsPlayerRef();
    PlayerRef playerRef = ref.getStore().getComponent(ref, PlayerRef.getComponentType());
    playerRef.referToServer(host, port, data);
    return CompletableFuture.completedFuture(null);
  }
}
```

## Example 10: Transfer with referral data + receiving redirect

Goal: send referral data from a command, then read and route on the target server
using `PlayerSetupConnectEvent`.

Source server command:

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.OptionalArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.CompletableFuture;

public final class TransferLobbyCommand extends AbstractCommand {
  private final RequiredArg<String> hostArg;
  private final RequiredArg<Integer> portArg;
  private final OptionalArg<String> tagArg;

  public TransferLobbyCommand() {
    super("transferlobby", "Send a player to a lobby shard");
    hostArg = withRequiredArg("host", "Target host", ArgTypes.STRING);
    portArg = withRequiredArg("port", "Target port", ArgTypes.INTEGER);
    tagArg = withOptionalArg("tag", "Referral tag", ArgTypes.STRING);
  }

  @Override
  protected CompletableFuture<Void> execute(CommandContext context) {
    if (!context.isPlayer()) {
      return CompletableFuture.completedFuture(null);
    }
    String host = hostArg.get(context);
    int port = portArg.get(context);
    byte[] data = tagArg.provided(context)
        ? tagArg.get(context).getBytes(StandardCharsets.UTF_8)
        : null;

    Ref<EntityStore> ref = context.senderAsPlayerRef();
    PlayerRef playerRef = ref.getStore().getComponent(ref, PlayerRef.getComponentType());
    playerRef.referToServer(host, port, data);
    return CompletableFuture.completedFuture(null);
  }
}
```

Receiving server redirect:

```java
import com.hypixel.hytale.server.core.event.events.player.PlayerSetupConnectEvent;
import java.nio.charset.StandardCharsets;

getEventRegistry().registerGlobal(PlayerSetupConnectEvent.class, event -> {
  byte[] data = event.getReferralData();
  if (data == null) {
    return;
  }
  String tag = new String(data, StandardCharsets.UTF_8);
  if ("lobby-1".equals(tag)) {
    event.referToServer("lobby.example.com", 25565);
  }
});
```

## Helper: parse extra args safely

If you need to read `CommandContext.getInputString()` and handle quoted args,
use a small tokenizer. Skip the called command token; for nested routes, skip
any parent tokens included in the input string.

```java
import com.hypixel.hytale.server.core.command.system.CommandContext;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

static String[] argsAfter(CommandContext context, int skipTokens) {
  List<String> tokens = tokenizeInput(context.getInputString());
  if (tokens.size() <= skipTokens) {
    return new String[0];
  }
  return Arrays.copyOfRange(tokens.toArray(new String[0]), skipTokens, tokens.size());
}

static List<String> tokenizeInput(String input) {
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

## Notes

- Command helpers here are JAR-based (`AbstractCommand`, `CommandContext`,
  `ArgTypes`), but other modules are placeholders.
- Replace types like `Server`, `Player`, and `Event` with actual SDK classes.
- Use `/help` or `/commands` in-game to confirm actual command registration.
