# Plugin Starter (Buildable Template)

This is a minimal, buildable plugin template aligned to the server JAR.
It uses Java 21, `manifest.json`, and the server command/event registries.

## Project Layout

```
my-plugin/
├── build.gradle
├── settings.gradle
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── example/
│       │           └── myplugin/
│       │               └── MyPlugin.java
│       └── resources/
│           └── manifest.json
└── lib/
    └── HytaleServer.jar
```

## settings.gradle

```groovy
rootProject.name = 'my-plugin'
```

## build.gradle

```groovy
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    compileOnly files('lib/HytaleServer.jar')
    implementation 'javax.annotation:javax.annotation-api:1.3.2'
}

jar {
    from('src/main/resources') {
        include 'manifest.json'
    }
}
```

## manifest.json

```json
{
  "Group": "Example",
  "Name": "MyPlugin",
  "Version": "1.0.0",
  "Main": "com.example.myplugin.MyPlugin",
  "ServerVersion": "*"
}
```

## Minimal Plugin

```java
package com.example.myplugin;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import javax.annotation.Nonnull;

public final class MyPlugin extends JavaPlugin {
    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        getLogger().info("MyPlugin setup");
    }

    @Override
    protected void start() {
        getLogger().info("MyPlugin start");
    }

    @Override
    protected void shutdown() {
        getLogger().info("MyPlugin shutdown");
    }
}
```

## Minimal Wiring (Command + Event)

This is a compact wiring example that registers a command and a player-ready event.

```java
import com.hypixel.hytale.server.core.event.events.player.PlayerReadyEvent;

@Override
protected void setup() {
    getCommandRegistry().registerCommand(new HelloCommand());
    getEventRegistry().register(PlayerReadyEvent.class, event -> {
        getLogger().info("Player ready: " + event.getPlayer().getName());
    });
}
```

## Command Example (JAR-Aligned)

```java
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import java.util.concurrent.CompletableFuture;

public final class HelloCommand extends AbstractCommand {
    private final RequiredArg<String> nameArg =
        new RequiredArg<>(this, "name", "Player name", ArgTypes.STRING);

    public HelloCommand() {
        super("hello", "Say hello");
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext context) {
        String name = nameArg.get(context);
        context.sendMessage(Message.raw("Hello " + name));
        return CompletableFuture.completedFuture(null);
    }
}
```

Register in `setup()`:

```java
getCommandRegistry().registerCommand(new HelloCommand());
```

## Event Example (Player Ready)

```java
import com.hypixel.hytale.server.core.event.events.player.PlayerReadyEvent;

getEventRegistry().register(PlayerReadyEvent.class, event -> {
    getLogger().info("Player ready: " + event.getPlayer().getName());
});
```

## Optional: Register a Custom Interaction

Short, non-verbatim example based on interaction patterns in the JAR.

```java
import com.hypixel.hytale.server.core.modules.interaction.interaction.config.Interaction;

@Override
protected void setup() {
    getCodecRegistry(Interaction.CODEC)
        .register("Example_Ping", ExamplePingInteraction.class, ExamplePingInteraction.CODEC);
}
```
