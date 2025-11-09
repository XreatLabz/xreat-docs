# NexAuth Developer Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Development Environment Setup](#development-environment-setup)
3. [Project Structure](#project-structure)
4. [Building from Source](#building-from-source)
5. [API Overview](#api-overview)
6. [Extending NexAuth](#extending-nexauth)
7. [Custom Database Providers](#custom-database-providers)
8. [Custom Event Handlers](#custom-event-handlers)
9. [Custom Crypto Providers](#custom-crypto-providers)
10. [Plugin Integration](#plugin-integration)
11. [Testing](#testing)
12. [Contributing](#contributing)
13. [Best Practices](#best-practices)

---

## Introduction

This guide is for developers who want to:
- Extend NexAuth with custom functionality
- Integrate NexAuth with other plugins
- Contribute to the NexAuth project
- Build custom features on top of NexAuth

### Prerequisites

- Java 21 or newer
- Maven or Gradle
- Git
- Basic understanding of Minecraft plugin development
- Knowledge of design patterns (Observer, Strategy, Factory)

### What You'll Learn

- How to set up development environment
- How to build NexAuth from source
- How to extend NexAuth's functionality
- How to create custom database providers
- How to listen to and fire events
- How to integrate with other plugins

---

## Development Environment Setup

### Step 1: Install Java 21

**Linux (Ubuntu/Debian)**:
```bash
sudo apt update
sudo apt install openjdk-21-jdk

# Verify
java -version
javac -version
```

**Windows**:
1. Download OpenJDK 21 from [Adoptium](https://adoptium.net/)
2. Run installer
3. Verify in Command Prompt: `java -version`

**macOS**:
```bash
brew install openjdk@21
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
```

### Step 2: Install Git

**Linux**:
```bash
sudo apt install git
```

**Windows**: Download from [git-scm.com](https://git-scm.com/download/win)

**macOS**:
```bash
# Git comes with Xcode Command Line Tools
xcode-select --install
```

### Step 3: Clone Repository

```bash
git clone https://github.com/xreatlabs/NexAuth.git
cd NexAuth
```

### Step 4: Set Up IDE

#### IntelliJ IDEA (Recommended)

1. Download IntelliJ IDEA Community Edition
2. Open project:
   ```
   File > Open > Select NexAuth folder
   ```
3. Import Gradle project when prompted
4. Wait for indexing to complete

#### Eclipse

1. Install BuildShip Gradle plugin
2. Import as Gradle project
3. Select project root directory

### Step 5: Verify Setup

Build the project:
```bash
# Using Gradle
./gradlew build

# On Windows
gradlew.bat build
```

Should see:
```
BUILD SUCCESSFUL
```

---

## Project Structure

```
NexAuth/
├── API/                          # API module - interfaces and abstractions
│   └── src/main/java/
│       └── xyz/xreatlabs/nexauth/api/
│           ├── event/            # Event system interfaces
│           ├── database/         # Database abstractions
│           ├── authorization/    # Authorization interfaces
│           ├── crypto/          # Crypto interfaces
│           ├── configuration/   # Configuration interfaces
│           ├── server/          # Server management
│           └── provider/        # Provider interfaces
│
├── Plugin/                       # Plugin module - implementations
│   └── src/main/java/
│       └── xyz/xreatlabs/nexauth/
│           ├── common/          # Shared implementation
│           │   ├── AuthenticNexAuth.java
│           │   ├── authorization/
│           │   ├── database/    # Database implementations
│           │   ├── config/      # Configuration system
│           │   ├── command/     # Command system
│           │   ├── crypto/      # Crypto implementations
│           │   ├── mail/
│           │   ├── totp/
│           │   ├── image/
│           │   └── listener/
│           ├── bungeecord/      # BungeeCord-specific code
│           └── velocity/        # Velocity-specific code (if present)
│
├── NexLimbo/                     # Companion limbo server project
│   ├── api/                      # Limbo API
│   ├── bungee/                   # BungeeCord implementation
│   └── velocity/                 # Velocity implementation
│
├── docs/                         # Documentation
├── settings.gradle               # Gradle settings
├── build.gradle                  # Root build file
└── README.md
```

### Module Breakdown

#### API Module

Contains **interfaces only** - no implementations. This is what other plugins use to integrate with NexAuth.

**Key Interfaces**:
- `NexAuthPlugin` - Main plugin interface
- `EventProvider` - Event system
- `ReadDatabaseProvider` - Database read operations
- `WriteDatabaseProvider` - Database write operations
- `AuthorizationProvider` - Authentication management
- `CryptoProvider` - Password hashing

#### Plugin Module

Contains **implementations** for specific platforms (BungeeCord, Velocity, Paper).

**Key Classes**:
- `AuthenticNexAuth` - Main implementation
- `AuthenticDatabaseProvider` - Database implementation
- `AuthenticEventProvider` - Event implementation
- `AuthenticAuthorizationProvider` - Authorization implementation

---

## Building from Source

### Using Gradle

```bash
# Build all modules
./gradlew build

# Build specific module
./gradlew :API:build
./gradlew :Plugin:build

# Build without tests (faster)
./gradlew build -x test

# Clean build
./gradlew clean build
```

### Build Outputs

After successful build:

```
API/build/libs/
├── nexauth-api-0.0.1-beta3.jar
└── nexauth-api-0.0.1-beta3-sources.jar

Plugin/build/libs/
├── nexauth-bungee-0.0.1-beta3.jar
├── nexauth-bungee-0.0.1-beta3-sources.jar
├── nexauth-velocity-0.0.1-beta3.jar
└── nexauth-velocity-0.0.1-beta3-sources.jar
```

### Running Tests

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests "xyz.xreatlabs.nexauth.database.AuthenticDatabaseProviderTest"

# Run tests with coverage
./gradlew jacocoTestReport
```

### Publishing to Local Maven

```bash
# Publish to local Maven repository
./gradlew publishToMavenLocal

# Use in your project
# In your build.gradle:
repositories {
    mavenLocal()
}

dependencies {
    implementation 'xyz.xreatlabs.nexauth:nexauth-api:0.0.1-beta3-SNAPSHOT'
}
```

---

## API Overview

### Core Interfaces

#### NexAuthPlugin

Main interface that all plugin implementations extend.

**Key Methods**:
```java
public interface NexAuthPlugin<P, S> {
    ReadWriteDatabaseProvider getDatabaseProvider();
    EventProvider<P, S> getEventProvider();
    AuthorizationProvider<P> getAuthorizationProvider();
    Logger getLogger();
    HoconPluginConfiguration getConfiguration();
    Messages getMessages();
    // ... many more methods
}
```

#### EventProvider

Manages event system (Observer pattern).

```java
public interface EventProvider<P, S> {
    <E extends Event<P, S>> Consumer<E> subscribe(EventType<P, S, E> type, Consumer<E> handler);
    void unsubscribe(Consumer<? extends Event<P, S>> handler);
    <E extends Event<P, S>> void fire(EventType<P, S, E> type, E event);
}
```

#### ReadDatabaseProvider

Read operations from database.

```java
public interface ReadDatabaseProvider {
    User getByUUID(UUID uuid);
    User getByName(String name);
    List<User> getAllUsers();
}
```

#### WriteDatabaseProvider

Write operations to database.

```java
public interface WriteDatabaseProvider {
    void insertUser(User user);
    void updateUser(User user);
    void deleteUser(UUID uuid);
    void deleteUser(User user);
}
```

#### ReadWriteDatabaseProvider

Combined read/write interface.

```java
public interface ReadWriteDatabaseProvider extends ReadDatabaseProvider, WriteDatabaseProvider {
}
```

#### AuthorizationProvider

Manages player authorization state.

```java
public interface AuthorizationProvider<P> {
    boolean isAuthorized(P player);
    boolean isAwaiting2FA(P player);
    void authorize(User user, P player, AuthenticationReason reason);
    boolean confirmTwoFactorAuth(P player, Integer code, User user);
}
```

#### CryptoProvider

Password hashing interface.

```java
public interface CryptoProvider {
    @Nullable
    HashedPassword createHash(String password);
    boolean matches(String input, HashedPassword password);
    String getIdentifier();
}
```

### Event System

#### Event Types

NexAuth provides numerous event types:

- `AuthenticatedEvent` - Player authenticated
- `WrongPasswordEvent` - Wrong password entered
- `PasswordChangeEvent` - Password changed
- `LobbyServerChooseEvent` - Choosing lobby server
- `LimboServerChooseEvent` - Choosing limbo server
- `PremiumLoginSwitchEvent` - Premium UUID switch

#### Event Structure

```java
public interface Event<P, S> {
    NexAuthPlugin<P, S> getPlugin();
    PlatformHandle<P, S> getPlatformHandle();
}
```

#### CancellableEvent

Events that can be cancelled by listeners.

```java
public interface CancellableEvent extends Event<P, S> {
    boolean isCancelled();
    void setCancelled(boolean cancelled);
}
```

---

## Extending NexAuth

### Creating a Custom Plugin Extension

#### Step 1: Create Project

```bash
mkdir NexAuth-Extension
cd NexAuth-Extension
```

#### Step 2: Setup build.gradle

```groovy
plugins {
    id 'java'
    id 'com.github.johnrengelman.shadow' version '8.1.1'
}

group = 'com.yourname'
version = '1.0.0'

repositories {
    mavenCentral()
    maven { url 'https://jitpack.io' }
}

dependencies {
    // Use NexAuth API
    implementation 'xyz.xreatlabs.nexauth:nexauth-api:0.0.1-beta3'

    // Platform-specific dependencies
    implementation 'xyz.xreatlabs.nexauth:nexauth-bungee:0.0.1-beta3'  // for BungeeCord
    // OR
    implementation 'xyz.xreatlabs.nexauth:nexauth-velocity:0.0.1-beta3'  // for Velocity

    // Your other dependencies
}

shadowJar {
    relocate 'xyz.xreatlabs.nexauth.api', 'shadow.nexauth.api'
    relocate 'xyz.xreatlabs.nexauth.common', 'shadow.nexauth.common'
}
```

#### Step 3: Implement Integration

```java
package com.yourname.nexauth.extension;

import xyz.xreatlabs.nexauth.api.NexAuthPlugin;
import xyz.xreatlabs.nexauth.api.event.events.AuthenticatedEvent;
import xyz.xreatlabs.nexauth.api.authorization.AuthorizationProvider;

public class CustomNexAuthIntegration {

    private final NexAuthPlugin<?, ?> nexAuth;

    public CustomNexAuthIntegration(NexAuthPlugin<?, ?> nexAuth) {
        this.nexAuth = nexAuth;

        // Subscribe to events
        registerEventHandlers();
    }

    private void registerEventHandlers() {
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onPlayerAuthenticated
        );
    }

    private void onPlayerAuthenticated(AuthenticatedEvent event) {
        // Your custom logic here
        System.out.println("Player authenticated: " + event.getPlayer());

        // Example: Send custom message
        // sendToDiscord(event.getPlayer());

        // Example: Give rewards
        // giveReward(event.getPlayer());
    }

    // Your custom methods
    public void customMethod() {
        // Access NexAuth features
        AuthorizationProvider<?> authProvider = nexAuth.getAuthorizationProvider();
        // Your logic
    }
}
```

### Creating a BungeeCord Plugin with NexAuth Integration

#### Step 1: Create bungee.yml

```yaml
name: NexAuthExtension
version: 1.0.0
author: YourName
main: com.yourname.nexauth.extension.BungeeExtension

depends: [NexAuth]

api-version: 1.20
```

#### Step 2: Create Plugin Class

```java
package com.yourname.nexauth.extension;

import net.md_5.bungee.api.plugin.Plugin;
import xyz.xreatlabs.nexauth.api.NexAuthPlugin;

public class BungeeExtension extends Plugin {

    private NexAuthPlugin<?, ?> nexAuth;
    private CustomNexAuthIntegration integration;

    @Override
    public void onEnable() {
        // Get NexAuth instance
        this.nexAuth = getNexAuth();

        if (nexAuth == null) {
            getLogger().severe("NexAuth not found! Disabling...");
            onDisable();
            return;
        }

        // Initialize integration
        this.integration = new CustomNexAuthIntegration(nexAuth);

        getLogger().info("NexAuth Extension enabled!");
    }

    @Override
    public void onDisable() {
        getLogger().info("NexAuth Extension disabled!");
    }

    private NexAuthPlugin<?, ?> getNexAuth() {
        // Get NexAuth plugin instance
        Plugin plugin = getProxy().getPluginManager().getPlugin("NexAuth");
        if (plugin instanceof NexAuthPlugin) {
            return (NexAuthPlugin<?, ?>) plugin;
        }
        return null;
    }
}
```

### Creating a Paper Plugin with NexAuth Integration

#### Step 1: Create plugin.yml

```yaml
name: NexAuthExtension
version: 1.0.0
author: YourName
main: com.yourname.nexauth.extension.PaperExtension
api-version: 1.20

depend: [NexAuth]

commands:
  mycommand:
    description: Custom command
    permission: mycommand.use
    permission-message: You don't have permission!
    usage: /mycommand
    permission-nodes:
      - mycommand.use

permissions:
  mycommand.use:
    description: Allows the user to use the custom command
    default: true
```

#### Step 2: Create Plugin Class

```java
package com.yourname.nexauth.extension;

import org.bukkit.plugin.java.JavaPlugin;
import xyz.xreatlabs.nexauth.api.NexAuthPlugin;
import xyz.xreatlabs.nexauth.api.event.events.AuthenticatedEvent;

public class PaperExtension extends JavaPlugin {

    private NexAuthPlugin<?, ?> nexAuth;

    @Override
    public void onEnable() {
        // Get NexAuth instance
        this.nexAuth = getServer().getPluginManager().getPlugin("NexAuth");

        if (nexAuth == null) {
            getLogger().severe("NexAuth not found! Disabling...");
            onDisable();
            return;
        }

        // Subscribe to events
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onPlayerAuthenticated
        );

        // Register commands
        getCommand("mycommand").setExecutor((sender, command, label, args) -> {
            // Your command logic
            sender.sendMessage("NexAuth is enabled: " + (nexAuth != null));
            return true;
        });

        getLogger().info("NexAuth Extension enabled!");
    }

    private void onPlayerAuthenticated(AuthenticatedEvent event) {
        // Your custom logic
        getLogger().info("Player authenticated: " + event.getPlayer());
    }
}
```

---

## Custom Database Providers

Create a custom database provider for unsupported databases or specific requirements.

### Example: Redis-Based Provider

#### Step 1: Implement DatabaseConnector

```java
public class RedisDatabaseConnector implements DatabaseConnector<String, Redis> {

    private final String prefix;
    private final Redis connection;

    public RedisDatabaseConnector(String prefix) {
        this.prefix = prefix;
        this.connection = new Redis(); // Your Redis client
    }

    @Override
    public void connect() throws Exception {
        connection.connect();
    }

    @Override
    public void disconnect() throws Exception {
        connection.disconnect();
    }

    @Override
    public boolean isConnected() {
        return connection.isConnected();
    }

    @Override
    public Redis getConnection() {
        return connection;
    }
}
```

#### Step 2: Implement ReadDatabaseProvider

```java
public class RedisReadDatabaseProvider implements ReadDatabaseProvider {

    private final Redis connector;
    private final NexAuthPlugin<?, ?> plugin;

    public RedisReadDatabaseProvider(DatabaseConnector<String, Redis> connector, NexAuthPlugin<?, ?> plugin) {
        this.connector = (Redis) connector;
        this.plugin = plugin;
    }

    @Override
    public User getByUUID(UUID uuid) {
        try {
            String key = "user:uuid:" + uuid.toString();
            String data = connector.getConnection().get(key);

            if (data == null) {
                return null;
            }

            return deserializeUser(data);
        } catch (Exception e) {
            plugin.getLogger().error("Failed to get user by UUID", e);
            return null;
        }
    }

    @Override
    public User getByName(String name) {
        try {
            String lowerName = name.toLowerCase();
            String uuidKey = "name:uuid:" + lowerName;
            String uuid = connector.getConnection().get(uuidKey);

            if (uuid == null) {
                return null;
            }

            return getByUUID(UUID.fromString(uuid));
        } catch (Exception e) {
            plugin.getLogger().error("Failed to get user by name", e);
            return null;
        }
    }

    @Override
    public List<User> getAllUsers() {
        List<User> users = new ArrayList<>();

        try {
            Set<String> keys = connector.getConnection().keys("user:uuid:*");

            for (String key : keys) {
                String data = connector.getConnection().get(key);
                if (data != null) {
                    users.add(deserializeUser(data));
                }
            }
        } catch (Exception e) {
            plugin.getLogger().error("Failed to get all users", e);
        }

        return users;
    }

    private User deserializeUser(String data) {
        // Deserialize JSON data to User object
        return GSON.fromJson(data, User.class);
    }
}
```

#### Step 3: Implement WriteDatabaseProvider

```java
public class RedisWriteDatabaseProvider implements WriteDatabaseProvider {

    private final Redis connector;
    private final NexAuthPlugin<?, ?> plugin;

    public RedisWriteDatabaseProvider(DatabaseConnector<String, Redis> connector, NexAuthPlugin<?, ?> plugin) {
        this.connector = (Redis) connector;
        this.plugin = plugin;
    }

    @Override
    public void insertUser(User user) {
        try {
            // Store user data
            String userKey = "user:uuid:" + user.getUUID().toString();
            connector.getConnection().setex(userKey, 3600, serializeUser(user));

            // Store name index
            String nameKey = "name:uuid:" + user.getLastNickname().toLowerCase();
            connector.getConnection().setex(nameKey, 3600, user.getUUID().toString());

        } catch (Exception e) {
            plugin.getLogger().error("Failed to insert user", e);
        }
    }

    @Override
    public void updateUser(User user) {
        try {
            insertUser(user); // Redis overwrites existing keys
        } catch (Exception e) {
            plugin.getLogger().error("Failed to update user", e);
        }
    }

    @Override
    public void deleteUser(UUID uuid) {
        try {
            User user = getByUUID(uuid);
            if (user != null) {
                // Delete user data
                connector.getConnection().del("user:uuid:" + uuid.toString());

                // Delete name index
                connector.getConnection().del("name:uuid:" + user.getLastNickname().toLowerCase());
            }
        } catch (Exception e) {
            plugin.getLogger().error("Failed to delete user", e);
        }
    }

    @Override
    public void deleteUser(User user) {
        deleteUser(user.getUUID());
    }

    private String serializeUser(User user) {
        // Serialize User object to JSON
        return GSON.toJson(user);
    }
}
```

#### Step 4: Combine Providers

```java
public class RedisReadWriteDatabaseProvider extends RedisReadDatabaseProvider
        implements ReadWriteDatabaseProvider {

    private final RedisWriteDatabaseProvider writeProvider;

    public RedisReadWriteDatabaseProvider(DatabaseConnector<String, Redis> connector,
                                          NexAuthPlugin<?, ?> plugin) {
        super(connector, plugin);
        this.writeProvider = new RedisWriteDatabaseProvider(connector, plugin);
    }

    @Override
    public void insertUser(User user) {
        writeProvider.insertUser(user);
    }

    @Override
    public void updateUser(User user) {
        writeProvider.updateUser(user);
    }

    @Override
    public void deleteUser(UUID uuid) {
        writeProvider.deleteUser(uuid);
    }

    @Override
    public void deleteUser(User user) {
        writeProvider.deleteUser(user);
    }
}
```

#### Step 5: Register Provider

```java
public class YourPlugin extends Plugin {

    @Override
    public void onEnable() {
        // Get NexAuth
        NexAuthPlugin<?, ?> nexAuth = getNexAuth();

        // Register database connector
        nexAuth.registerDatabaseConnector(
            RedisDatabaseConnector.class,
            prefix -> new RedisDatabaseConnector(prefix),
            "redis"
        );

        // Register database provider
        nexAuth.registerReadProvider(
            new ReadDatabaseProviderRegistration<>(
                connector -> new RedisReadWriteDatabaseProvider(connector, nexAuth),
                "nexauth-redis",
                RedisDatabaseConnector.class
            )
        );
    }
}
```

#### Step 6: Configure in config.conf

```hocon
database {
    type = "nexauth-redis"
    redis {
        host = "localhost"
        port = 6379
        password = "redis-password"
        database = 0
    }
}
```

### Example: File-Based Provider (JSON)

For simple setups or testing:

```java
public class JsonFileDatabaseProvider implements ReadWriteDatabaseProvider {

    private final File dataFile;
    private final Map<UUID, User> users;
    private final NexAuthPlugin<?, ?> plugin;

    public JsonFileDatabaseProvider(File dataFile, NexAuthPlugin<?, ?> plugin) {
        this.dataFile = dataFile;
        this.plugin = plugin;
        this.users = new ConcurrentHashMap<>();
        loadData();
    }

    private void loadData() {
        if (!dataFile.exists()) {
            return;
        }

        try (FileReader reader = new FileReader(dataFile)) {
            Type type = new TypeToken<Map<UUID, User>>(){}.getType();
            Map<UUID, User> loaded = GSON.fromJson(reader, type);
            if (loaded != null) {
                users.putAll(loaded);
            }
        } catch (Exception e) {
            plugin.getLogger().error("Failed to load data", e);
        }
    }

    private void saveData() {
        try (FileWriter writer = new FileWriter(dataFile)) {
            GSON.toJson(users, writer);
        } catch (Exception e) {
            plugin.getLogger().error("Failed to save data", e);
        }
    }

    @Override
    public User getByUUID(UUID uuid) {
        return users.get(uuid);
    }

    @Override
    public User getByName(String name) {
        String lowerName = name.toLowerCase();
        return users.values().stream()
            .filter(user -> user.getLastNickname().equalsIgnoreCase(lowerName))
            .findFirst()
            .orElse(null);
    }

    @Override
    public List<User> getAllUsers() {
        return new ArrayList<>(users.values());
    }

    @Override
    public void insertUser(User user) {
        users.put(user.getUUID(), user);
        saveData();
    }

    @Override
    public void updateUser(User user) {
        users.put(user.getUUID(), user);
        saveData();
    }

    @Override
    public void deleteUser(UUID uuid) {
        users.remove(uuid);
        saveData();
    }

    @Override
    public void deleteUser(User user) {
        deleteUser(user.getUUID());
    }
}
```

---

## Custom Event Handlers

Listen to NexAuth events to integrate with your plugin.

### Example: Reward System

```java
public class RewardSystem {

    private final NexAuthPlugin<?, ?> nexAuth;

    public RewardSystem(NexAuthPlugin<?, ?> nexAuth) {
        this.nexAuth = nexAuth;
        registerEvents();
    }

    private void registerEvents() {
        // Authenticated event
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onAuthenticated
        );

        // Wrong password event
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().wrongPassword(),
            this::onWrongPassword
        );

        // Password change event
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().passwordChange(),
            this::onPasswordChange
        );
    }

    private void onAuthenticated(AuthenticatedEvent event) {
        P player = event.getPlayer();
        User user = event.getUser();

        // Give welcome reward
        giveWelcomeReward(player);

        // Track statistics
        updateStatistics(player, user);

        // Send custom message
        sendCustomMessage(player, "Welcome back, " + user.getLastNickname() + "!");
    }

    private void onWrongPassword(WrongPasswordEvent event) {
        P player = event.getPlayer();
        int attempts = event.getAttempts();

        // After 3 attempts, send warning
        if (attempts >= 3) {
            sendCustomMessage(player, "Warning: Multiple failed login attempts!");
        }

        // After 5 attempts, alert staff
        if (attempts >= 5) {
            notifyStaff(player, attempts);
        }
    }

    private void onPasswordChange(PasswordChangeEvent event) {
        P player = event.getPlayer();

        // Announce password change to player
        sendCustomMessage(player, "Your password has been changed successfully!");

        // Log security event
        plugin.getLogger().info("Player " + player.getName() + " changed password");
    }

    // Your custom methods
    private void giveWelcomeReward(P player) {
        // Your reward logic
    }

    private void updateStatistics(P player, User user) {
        // Your statistics logic
    }
}
```

### Example: Integration with Discord

```java
public class DiscordIntegration {

    private final NexAuthPlugin<?, ?> nexAuth;
    private final DiscordBot discordBot;

    public DiscordIntegration(NexAuthPlugin<?, ?> nexAuth, DiscordBot discordBot) {
        this.nexAuth = nexAuth;
        this.discordBot = discordBot;
        registerEvents();
    }

    private void registerEvents() {
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onAuthenticated
        );

        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().wrongPassword(),
            this::onWrongPassword
        );

        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().passwordChange(),
            this::onPasswordChange
        );
    }

    private void onAuthenticated(AuthenticatedEvent event) {
        String playerName = getPlayerName(event.getPlayer());
        String message = "✅ " + playerName + " joined the server";
        discordBot.sendToLogChannel(message);
    }

    private void onWrongPassword(WrongPasswordEvent event) {
        String playerName = getPlayerName(event.getPlayer());
        String message = "⚠️ " + playerName + " failed login (attempt " + event.getAttempts() + ")";
        discordBot.sendToLogChannel(message);
    }

    private void onPasswordChange(PasswordChangeEvent event) {
        String playerName = getPlayerName(event.getPlayer());
        String message = "🔑 " + playerName + " changed their password";
        discordBot.sendToLogChannel(message);
    }

    private String getPlayerName(Object player) {
        // Platform-specific implementation
        return player.toString();
    }
}
```

### Creating Custom Events

Create custom events for your extension:

```java
public class CustomEvent implements Event<P, S> {

    private final NexAuthPlugin<P, S> plugin;
    private final P player;
    private final String data;

    public CustomEvent(NexAuthPlugin<P, S> plugin, P player, String data) {
        this.plugin = plugin;
        this.player = player;
        this.data = data;
    }

    @Override
    public NexAuthPlugin<P, S> getPlugin() {
        return plugin;
    }

    @Override
    public PlatformHandle<P, S> getPlatformHandle() {
        return plugin.getPlatformHandle();
    }

    public P getPlayer() {
        return player;
    }

    public String getData() {
        return data;
    }
}

// Fire the event
CustomEvent event = new CustomEvent(nexAuth, player, "custom data");
nexAuth.getEventProvider().fire(customEventType, event);
```

---

## Custom Crypto Providers

Create custom password hashing algorithms.

### Example: Custom Pepper-Based Hashing

```java
public class PepperCryptoProvider implements CryptoProvider {

    private final String pepper;
    private final MessageDigest digest;

    public PepperCryptoProvider(String pepper) throws NoSuchAlgorithmException {
        this.pepper = pepper;
        this.digest = MessageDigest.getInstance("SHA-256");
    }

    @Override
    public HashedPassword createHash(String password) {
        try {
            // Generate salt
            SecureRandom random = new SecureRandom();
            byte[] salt = new byte[16];
            random.nextBytes(salt);

            // Hash password + pepper + salt
            String input = password + pepper + Base64.getEncoder().encodeToString(salt);
            byte[] hash = digest.digest(input.getBytes(StandardCharsets.UTF_8));

            return new HashedPassword(
                Base64.getEncoder().encodeToString(hash),
                Base64.getEncoder().encodeToString(salt),
                "pepper-sha256"
            );
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public boolean matches(String input, HashedPassword password) {
        try {
            String salt = password.getSalt();
            String inputHash = createHash(input).getHash();
            return inputHash.equals(password.getHash());
        } catch (Exception e) {
            return false;
        }
    }

    @Override
    public String getIdentifier() {
        return "pepper-sha256";
    }
}
```

### Register Custom Crypto Provider

```java
// In your plugin
public class YourPlugin extends Plugin {

    @Override
    public void onEnable() {
        NexAuthPlugin<?, ?> nexAuth = getNexAuth();

        try {
            // Create provider with pepper from config
            String pepper = getConfig().getString("pepper");
            CryptoProvider provider = new PepperCryptoProvider(pepper);

            // Register
            nexAuth.registerCryptoProvider(provider);

            getLogger().info("Custom crypto provider registered: " + provider.getIdentifier());
        } catch (Exception e) {
            getLogger().severe("Failed to register crypto provider", e);
        }
    }
}
```

### Advanced: Memory-Hard KDF

```java
public class CustomKDFProvider implements CryptoProvider {

    private final int iterations;
    private final int memory;
    private final int parallelism;

    public CustomKDFProvider(int iterations, int memory, int parallelism) {
        this.iterations = iterations;
        this.memory = memory;
        this.parallelism = parallelism;
    }

    @Override
    public HashedPassword createHash(String password) {
        try {
            // Generate salt
            byte[] salt = new byte[16];
            new SecureRandom().nextBytes(salt);

            // Apply KDF
            byte[] hash = applyKDF(password, salt, iterations, memory, parallelism);

            return new HashedPassword(
                Base64.getEncoder().encodeToString(hash),
                Base64.getEncoder().encodeToString(salt),
                "custom-kdf"
            );
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public boolean matches(String input, HashedPassword password) {
        try {
            byte[] salt = Base64.getDecoder().decode(password.getSalt());
            byte[] inputHash = applyKDF(input, salt, iterations, memory, parallelism);
            return Arrays.equals(inputHash, Base64.getDecoder().decode(password.getHash()));
        } catch (Exception e) {
            return false;
        }
    }

    @Override
    public String getIdentifier() {
        return "custom-kdf";
    }

    private byte[] applyKDF(String password, byte[] salt, int iterations,
                           int memory, int parallelism) {
        // Your KDF implementation
        // Consider using existing KDF libraries like Google Tink, Bouncy Castle, etc.
        return new byte[32]; // Placeholder
    }
}
```

---

## Plugin Integration

### Integration Checklist

When integrating with NexAuth:

1. **Add Dependency**: Depend on NexAuth API
2. **Check for NexAuth**: Verify NexAuth is present and enabled
3. **Subscribe to Events**: Listen to relevant events
4. **Handle Errors**: Gracefully handle NexAuth not being present
5. **Unregister**: Clean up event listeners on disable

### Example: ChestShop Integration

```java
public class ChestShopNexAuthIntegration {

    private final ChestShop plugin;
    private final NexAuthPlugin<?, ?> nexAuth;

    public ChestShopNexAuthIntegration(ChestShop plugin) {
        this.plugin = plugin;
        this.nexAuth = getNexAuth();

        if (nexAuth != null) {
            registerEvents();
        }
    }

    private void registerEvents() {
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onPlayerAuthenticated
        );
    }

    private void onPlayerAuthenticated(AuthenticatedEvent event) {
        P player = event.getPlayer();

        // Send shop message
        sendMessage(player, "You can now use ChestShop!");
    }

    public boolean canPlayerShop(P player) {
        if (nexAuth == null) {
            return true; // Allow if NexAuth not installed
        }

        return nexAuth.getAuthorizationProvider().isAuthorized(player);
    }

    // Other integration methods...
}
```

### Integration Example: Vault Economy

```java
public class VaultNexAuthIntegration {

    private final NexAuthPlugin<?, ?> nexAuth;
    private final Economy economy;

    public VaultNexAuthIntegration(NexAuthPlugin<?, ?> nexAuth, Economy economy) {
        this.nexAuth = nexAuth;
        this.economy = economy;
        registerEvents();
    }

    private void registerEvents() {
        nexAuth.getEventProvider().subscribe(
            nexAuth.getEventProvider().getTypes().authenticated(),
            this::onAuthenticated
        );
    }

    private void onAuthenticated(AuthenticatedEvent event) {
        P player = event.getPlayer();

        // Check balance for premium benefits
        if (economy.hasAccount(player)) {
            double balance = economy.getBalance(player);

            if (balance > 1000) {
                grantPremiumBenefits(player);
            }
        }
    }
}
```

### Testing Integration

```java
public class IntegrationTest {

    @Test
    public void testNexAuthPresent() {
        // Test that NexAuth is properly detected
        Plugin plugin = getServer().getPluginManager().getPlugin("NexAuth");
        assertNotNull(plugin, "NexAuth should be installed");
        assertTrue(plugin.isEnabled(), "NexAuth should be enabled");
    }

    @Test
    public void testEventSubscription() {
        // Test that events are properly subscribed
        NexAuthPlugin<?, ?> nexAuth = getNexAuth();
        assertNotNull(nexAuth);

        // Verify event providers are available
        assertNotNull(nexAuth.getEventProvider());
        assertNotNull(nexAuth.getAuthorizationProvider());
    }

    @Test
    public void testAuthenticationCheck() {
        // Test authentication checking
        // ... your test code
    }
}
```

---

## Testing

### Unit Testing Database Providers

```java
public class CustomDatabaseProviderTest {

    private TestNexAuthPlugin plugin;
    private CustomDatabaseProvider provider;
    private InMemoryDatabaseConnector connector;

    @Before
    public void setUp() {
        plugin = new TestNexAuthPlugin();
        connector = new InMemoryDatabaseConnector();
        provider = new CustomDatabaseProvider(connector, plugin);
    }

    @Test
    public void testInsertAndGetUser() {
        // Create test user
        User user = plugin.createUser(
            UUID.randomUUID(),
            null,
            new HashedPassword("hash", "salt", "argon2id"),
            "TestUser",
            new Timestamp(System.currentTimeMillis()),
            new Timestamp(System.currentTimeMillis()),
            null,
            "127.0.0.1",
            new Timestamp(System.currentTimeMillis()),
            "lobby",
            null
        );

        // Insert user
        provider.insertUser(user);

        // Retrieve user
        User retrieved = provider.getByUUID(user.getUUID());

        // Verify
        assertNotNull(retrieved);
        assertEquals(user.getUUID(), retrieved.getUUID());
        assertEquals(user.getLastNickname(), retrieved.getLastNickname());
    }

    @Test
    public void testUpdateUser() {
        // Test user update
        // ... test code
    }

    @Test
    public void testDeleteUser() {
        // Test user deletion
        // ... test code
    }
}
```

### Mock NexAuth Plugin for Testing

```java
public class MockNexAuthPlugin implements NexAuthPlugin<Object, Object> {

    private final Map<String, CryptoProvider> cryptoProviders = new HashMap<>();
    private EventProvider<Object, Object> eventProvider;
    private ReadWriteDatabaseProvider databaseProvider;
    private AuthorizationProvider<Object> authProvider;
    private Logger logger;

    public MockNexAuthPlugin() {
        this.logger = new SystemOutLogger();
        this.eventProvider = new MockEventProvider();
        // Initialize other mock providers
    }

    // Implement required methods...

    @Override
    public ReadWriteDatabaseProvider getDatabaseProvider() {
        return databaseProvider;
    }

    @Override
    public EventProvider<Object, Object> getEventProvider() {
        return eventProvider;
    }

    // ... other methods
}
```

### Integration Testing

```java
public class NexAuthIntegrationTest {

    private static final String TEST_DB = "test.db";

    @BeforeClass
    public static void setUpClass() {
        // Set up test database
        File dbFile = new File(TEST_DB);
        if (dbFile.exists()) {
            dbFile.delete();
        }
    }

    @AfterClass
    public static void tearDownClass() {
        // Clean up test database
        File dbFile = new File(TEST_DB);
        if (dbFile.exists()) {
            dbFile.delete();
        }
    }

    @Test
    public void testFullAuthenticationFlow() {
        // Test complete auth flow
        // ... test code
    }
}
```

### Testing Utilities

```java
public class TestUtils {

    public static User createTestUser(String name) {
        return new AuthenticUser(
            UUID.randomUUID(),
            null,
            new HashedPassword("passwordHash", "salt", "argon2id"),
            name,
            new Timestamp(System.currentTimeMillis()),
            new Timestamp(System.currentTimeMillis()),
            null,
            "127.0.0.1",
            new Timestamp(System.currentTimeMillis()),
            "lobby",
            null
        );
    }

    public static HashedPassword createTestPassword(String password, String algorithm) {
        CryptoProvider provider = getProvider(algorithm);
        return provider.createHash(password);
    }

    public static CryptoProvider getProvider(String identifier) {
        // Get crypto provider for testing
        return new Argon2IDCryptoProvider(new SystemOutLogger());
    }
}
```

---

## Contributing

### Contribution Guidelines

We welcome contributions to NexAuth! Here's how to contribute:

#### Types of Contributions

- 🐛 Bug fixes
- ✨ New features
- 📚 Documentation improvements
- 🧪 Test coverage improvements
- 💡 Performance optimizations
- 🔧 Code refactoring

#### Before You Start

1. Check existing issues and PRs
2. Create an issue to discuss large changes
3. Ask questions in Discord if unsure

#### Development Workflow

1. **Fork** the repository
2. **Clone** your fork:
   ```bash
   git clone https://github.com/yourname/NexAuth.git
   cd NexAuth
   ```
3. **Create** a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make** your changes
5. **Test** thoroughly:
   ```bash
   ./gradlew test
   ```
6. **Commit** with clear messages:
   ```bash
   git commit -m "feat: add feature X (fixes #123)"
   ```
7. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
8. **Open** a Pull Request

#### Code Style

- Follow [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- Use meaningful variable names
- Write JavaDoc for public methods
- Keep methods focused and small
- Use interfaces for abstraction

#### Commit Message Format

Use conventional commits:

```
type(scope): subject

body (optional)

footer (optional)
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Test changes
- `chore`: Build process, dependency updates

Examples:
```bash
feat(database): add support for MongoDB provider
fix(event): prevent NPE on player disconnect
docs(api): update API documentation for v0.0.1-beta3
refactor(crypto): simplify hash verification logic
```

#### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Code refactoring
- [ ] Performance improvement

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Code is properly commented
- [ ] Changes are documented
- [ ] No breaking changes (or documented)

## Related Issues
Closes #123
```

#### Running Tests

```bash
# Run all tests
./gradlew test

# Run tests with coverage
./gradlew jacocoTestReport

# Run specific test class
./gradlew test --tests "*DatabaseProviderTest"

# Run integration tests
./gradlew integrationTest
```

#### Building Documentation

If you make changes to the API:

```bash
# Generate Javadoc
./gradlew :API:javadoc

# Open docs/index.html
```

#### Code Review Process

1. **Automated checks** must pass (CI)
2. **At least one** reviewer approval required
3. **Address feedback** and push changes
4. **Squash commits** before merging (if requested)

### Getting Help

- **Discord**: https://discord.gg/m5ptM8X2Du
- **GitHub Issues**: https://github.com/xreatlabs/NexAuth/issues
- **GitHub Discussions**: https://github.com/xreatlabs/NexAuth/discussions

---

## Best Practices

### Design Patterns

#### Use Observer Pattern for Events

```java
// Good: Use events for loose coupling
eventProvider.subscribe(types.authenticated(), event -> {
    // Handle event
});

// Avoid: Hard-coded logic
if (player.isAuthenticated()) {
    // Hard-coded
}
```

#### Use Strategy Pattern for Providers

```java
// Good: Pluggable providers
ReadWriteDatabaseProvider provider = new MySQLProvider(connector);
// Can easily switch to PostgreSQL provider

// Avoid: Hard-coded implementation
MySQLDatabaseProvider provider = new MySQLDatabaseProvider();
```

#### Use Factory Pattern for Object Creation

```java
// Good: Factory for creating objects
User user = userFactory.create(uuid, username, password);

// Avoid: Direct instantiation everywhere
User user = new AuthenticUser(...);
```

### Error Handling

#### Handle Exceptions Gracefully

```java
// Good: Handle errors properly
try {
    User user = databaseProvider.getByUUID(uuid);
    if (user == null) {
        // Handle not found
        return;
    }
    // Process user
} catch (SQLException e) {
    plugin.getLogger().error("Database error", e);
    // Handle database error
    return;
}

// Avoid: Swallowing exceptions
try {
    databaseProvider.getByUUID(uuid);
} catch (Exception e) {
    // Silent failure
}
```

#### Use Specific Exceptions

```java
// Good: Specific exceptions
try {
    // Operation
} catch (SQLException e) {
    throw new DatabaseException("Failed to fetch user", e);
} catch (JsonParseException e) {
    throw new ConfigurationException("Invalid config format", e);
}

// Avoid: Generic exceptions
try {
    // Operation
} catch (Exception e) {
    throw new RuntimeException(e);
}
```

### Performance

#### Use Async Operations

```java
// Good: Async database query
CompletableFuture.supplyAsync(() -> {
    return databaseProvider.getByUUID(uuid);
}).thenAccept(user -> {
    // Process result on async thread
    processUser(user);
});

// Avoid: Blocking main thread
User user = databaseProvider.getByUUID(uuid); // Blocks!
processUser(user);
```

#### Batch Database Operations

```java
// Good: Batch insert
List<User> users = getUsersToInsert();
databaseProvider.insertUsers(users); // Single operation

// Avoid: Individual inserts
for (User user : users) {
    databaseProvider.insertUser(user); // N operations
}
```

#### Cache Frequently Used Data

```java
// Good: Cache data
private final Map<UUID, User> userCache = new ConcurrentHashMap<>();

public User getUser(UUID uuid) {
    return userCache.computeIfAbsent(uuid, this::loadUserFromDatabase);
}

// Avoid: Database query every time
public User getUser(UUID uuid) {
    return databaseProvider.getByUUID(uuid); // Expensive!
}
```

### Security

#### Never Store Plaintext Passwords

```java
// Good: Always hash passwords
CryptoProvider provider = plugin.getDefaultCryptoProvider();
HashedPassword hashed = provider.createHash(password);

// Avoid: NEVER store plaintext
String password = "userPassword"; // NEVER!
```

#### Validate All Inputs

```java
// Good: Validate inputs
if (username == null || username.length() < 3) {
    throw new IllegalArgumentException("Username too short");
}

if (!username.matches("[a-zA-Z0-9_]+")) {
    throw new IllegalArgumentException("Invalid username");
}

// Avoid: No validation
String username = input; // Risky!
```

#### Use Prepared Statements

```java
// Good: Prepared statement
PreparedStatement stmt = connection.prepareStatement(
    "SELECT * FROM users WHERE uuid = ?"
);
stmt.setString(1, uuid.toString());
ResultSet rs = stmt.executeQuery();

// Avoid: SQL injection vulnerable
String query = "SELECT * FROM users WHERE uuid = '" + uuid + "'";
// NEVER do this!
```

### Code Organization

#### Keep Methods Small and Focused

```java
// Good: Focused methods
public void authenticatePlayer(Player player, String password) {
    validateInput(player, password);
    verifyCredentials(player, password);
    authorizePlayer(player);
}

private void validateInput(Player player, String password) {
    // Validation logic
}

private void verifyCredentials(Player player, String password) {
    // Verification logic
}

private void authorizePlayer(Player player) {
    // Authorization logic
}

// Avoid: Large monolithic method
public void authenticatePlayer(Player player, String password) {
    // Everything in one method
}
```

#### Use Dependency Injection

```java
// Good: Dependency injection
public class AuthService {
    private final DatabaseProvider db;
    private final EventProvider eventProvider;
    private final Logger logger;

    public AuthService(DatabaseProvider db, EventProvider eventProvider, Logger logger) {
        this.db = db;
        this.eventProvider = eventProvider;
        this.logger = logger;
    }
}

// Avoid: Hard dependencies
public class AuthService {
    private DatabaseProvider db = new MySQLDatabaseProvider(...); // Hard-coded!
}
```

#### Document Public APIs

```java
// Good: Well documented
/**
 * Authenticates a player with the given password.
 *
 * @param player the player to authenticate
 * @param password the plaintext password
 * @return true if authentication successful, false otherwise
 * @throws AuthException if authentication fails
 * @throws IllegalArgumentException if player or password is null
 */
public boolean authenticatePlayer(Player player, String password) throws AuthException {
    // Implementation
}

// Avoid: No documentation
public boolean auth(Player p, String pass) {
    // What does this do?
}
```

### Testing

#### Write Unit Tests

```java
// Good: Test coverage
@Test
public void testPasswordValidation() {
    // Arrange
    String password = "SecurePass123!";

    // Act
    boolean isValid = validator.isValid(password);

    // Assert
    assertTrue(isValid);
}

// Test edge cases
@Test
public void testPasswordTooShort() {
    String password = "short";
    assertFalse(validator.isValid(password));
}

@Test
public void testPasswordNull() {
    assertThrows(IllegalArgumentException.class, () -> {
        validator.isValid(null);
    });
}
```

#### Use Mocks for Testing

```java
// Good: Use mocks
@Test
public void testAuthenticationSuccess() {
    // Arrange
    DatabaseProvider mockDb = mock(DatabaseProvider.class);
    User mockUser = createTestUser();
    when(mockDb.getByUUID(any())).thenReturn(mockUser);

    AuthService service = new AuthService(mockDb, ...);

    // Act
    boolean result = service.authenticate(player, "password");

    // Assert
    assertTrue(result);
    verify(mockDb).getByUUID(any());
}
```

#### Test Integration Points

```java
// Good: Test integration
@Test
public void testDatabaseProviderIntegration() {
    // Test that your provider works with NexAuth
    DatabaseProvider provider = new CustomProvider(connector);

    // Verify it implements the interface
    assertTrue(provider instanceof ReadDatabaseProvider);
    assertTrue(provider instanceof WriteDatabaseProvider);

    // Test basic operations
    User user = createTestUser();
    provider.insertUser(user);

    User retrieved = provider.getByUUID(user.getUUID());
    assertNotNull(retrieved);
}
```

### Logging

#### Use Appropriate Log Levels

```java
// Good: Correct log levels
logger.info("Player authenticated: " + player.getName());  // Info
logger.warn("Multiple failed attempts for: " + player.getName());  // Warning
logger.error("Database connection failed", exception);  // Error
logger.debug("Processing user data for: " + uuid);  // Debug

// Avoid: Wrong levels
logger.info("Database connection failed");  // Should be error!
logger.error("Player joined");  // Should be info!
```

#### Include Context in Logs

```java
// Good: Contextual logging
logger.info("User {} authenticated via {} (IP: {})",
    user.getLastNickname(),
    reason.name(),
    user.getIp());

// Avoid: Unclear logs
logger.info("User logged in");
```

### Configuration

#### Use Type-Safe Configuration

```java
// Good: Type-safe
boolean enabled = config.get(MAIL_ENABLED);
int maxAttempts = config.get(MAX_LOGIN_ATTEMPTS);

// Avoid: Raw access
String value = (String) config.get("mail.enabled");  // Not type-safe!
```

#### Validate Configuration

```java
// Good: Validate config
if (config.get(DATABASE_POOL_SIZE) < 1) {
    throw new ConfigurationException("Database pool size must be positive");
}

// Avoid: No validation
int poolSize = config.get(DATABASE_POOL_SIZE);  // Could be 0 or negative!
```

---

## Example Projects

Looking for examples? Check out these projects:

### Official Extensions

1. **NexLimbo**: https://github.com/Xreatlabs/NexLimbo
   - Optimized limbo server
   - Official NexAuth companion

### Community Extensions

*(Feel free to add your project here by opening a PR)*

---

## Resources

### Documentation
- [NexAuth Wiki](https://github.com/xreatlabs/NexAuth/wiki)
- [API Reference](API_REFERENCE.md)
- [User Guide](USER_GUIDE.md)
- [Architecture](ARCHITECTURE.md)

### APIs and Libraries
- [Mojang API](https://wiki.vg/Mojang_API)
- [ACF Command Framework](https://github.com/aikar/commands)
- [HOCON Configuration](https://github.com/lightbend/config)
- [Adventure Text API](https://github.com/KyoriAdventure/adventure)

### Tools
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)
- [Eclipse](https://www.eclipse.org/)
- [Gradle](https://gradle.org/)
- [Maven](https://maven.apache.org/)

### Testing
- [JUnit 5](https://junit.org/junit5/)
- [Mockito](https://site.mockito.org/)
- [TestContainers](https://www.testcontainers.org/)

---

## Support

### Getting Help

- **GitHub Issues**: [Report bugs or request features](https://github.com/xreatlabs/NexAuth/issues)
- **Discord**: [Join our community](https://discord.gg/m5ptM8X2Du)
- **GitHub Discussions**: [Ask questions](https://github.com/xreatlabs/NexAuth/discussions)

### Professional Support

For commercial support or consulting:
- Contact us on Discord
- Email: [contact information]

---

**Version**: 0.0.1-beta3
**Last Updated**: 2025-10-30
**Maintainers**: Xreatlabs Team

Happy coding! 🚀
