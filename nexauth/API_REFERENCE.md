# NexAuth API Reference

## Table of Contents

1. [Core API](#core-api)
2. [Event System](#event-system)
3. [Database API](#database-api)
4. [Authorization API](#authorization-api)
5. [Crypto API](#crypto-api)
6. [Configuration API](#configuration-api)
7. [Server Management API](#server-management-api)
8. [Premium API](#premium-api)
9. [TOTP API](#totp-api)
10. [Command API](#command-api)
11. [Mail API](#mail-api)

---

## Core API

### NexAuthPlugin

The main plugin interface that all plugin implementations must extend.

#### Methods

##### `getDatabaseProvider() : ReadWriteDatabaseProvider`

Returns the database provider instance.

**Returns**: The configured database provider

**Example**:
```java
ReadWriteDatabaseProvider db = plugin.getDatabaseProvider();
User user = db.getByUUID(playerUUID);
```

---

##### `getEventProvider() : EventProvider<P, S>`

Returns the event provider for subscribing to and firing events.

**Returns**: The event provider instance

**Example**:
```java
EventProvider<P, S> eventProvider = plugin.getEventProvider();
Consumer<AuthenticatedEvent> handler = event -> {
    // Handle authenticated event
};
eventProvider.subscribe(eventProvider.getTypes().authenticated(), handler);
```

---

##### `getAuthorizationProvider() : AuthorizationProvider<P>`

Returns the authorization provider for managing player authentication state.

**Returns**: The authorization provider

**Example**:
```java
AuthorizationProvider<P> authProvider = plugin.getAuthorizationProvider();
boolean isAuthorized = authProvider.isAuthorized(player);
```

---

##### `getLogger() : Logger`

Returns the plugin logger.

**Returns**: The logger instance

---

##### `getConfiguration() : HoconPluginConfiguration`

Returns the plugin configuration.

**Returns**: The configuration instance

**Example**:
```java
HoconPluginConfiguration config = plugin.getConfiguration();
boolean mailEnabled = config.get(MAIL_ENABLED);
```

---

##### `getMessages() : Messages`

Returns the messages manager for localization.

**Returns**: The messages instance

**Example**:
```java
Messages messages = plugin.getMessages();
TextComponent msg = messages.getMessage("login.success", player.getName());
```

---

##### `getCryptoProvider(String id) : CryptoProvider`

Returns a specific crypto provider by identifier.

**Parameters**:
- `id` - The crypto provider identifier (e.g., "argon2id", "bcrypt2a")

**Returns**: The crypto provider, or null if not found

**Example**:
```java
CryptoProvider argon2 = plugin.getCryptoProvider("argon2id");
HashedPassword hashed = argon2.createHash(password);
```

---

##### `getDefaultCryptoProvider() : CryptoProvider`

Returns the default configured crypto provider.

**Returns**: The default crypto provider

---

##### `registerCryptoProvider(CryptoProvider provider)`

Registers a new crypto provider.

**Parameters**:
- `provider` - The crypto provider to register

**Example**:
```java
plugin.registerCryptoProvider(new CustomCryptoProvider());
```

---

##### `migrate(ReadDatabaseProvider from, WriteDatabaseProvider to)`

Migrates user data from one database provider to another.

**Parameters**:
- `from` - Source database provider
- `to` - Destination database provider

**Example**:
```java
ReadDatabaseProvider oldProvider = ...;
ReadWriteDatabaseProvider newProvider = ...;
plugin.migrate(oldProvider, newProvider);
```

---

##### `registerDatabaseConnector(...)`

Registers a database connector.

**Overloads**:

```java
<E extends Exception, C extends DatabaseConnector<E, ?>>
void registerDatabaseConnector(Class<?> clazz, ThrowableFunction<String, C, E> factory, String id)

void registerDatabaseConnector(DatabaseConnectorRegistration<?, ?> registration, Class<?> clazz)
```

**Example**:
```java
plugin.registerDatabaseConnector(
    CustomConnector.class,
    prefix -> new CustomConnector(prefix),
    "custom"
);
```

---

##### `registerReadProvider(ReadDatabaseProviderRegistration<?, ?, ?> registration)`

Registers a read database provider.

**Parameters**:
- `registration` - The provider registration

**Example**:
```java
plugin.registerReadProvider(new ReadDatabaseProviderRegistration<>(
    connector -> new CustomProvider(connector),
    "custom-provider",
    CustomConnector.class
));
```

---

##### `createUser(...) : User`

Creates a new user instance.

**Parameters**:
- `uuid` - The player's UUID
- `premiumUUID` - The premium UUID (if any)
- `hashedPassword` - The hashed password
- `lastNickname` - The last used nickname
- `joinDate` - Join timestamp
- `lastSeen` - Last seen timestamp
- `secret` - TOTP secret (if any)
- `ip` - IP address
- `lastAuthentication` - Last authentication timestamp
- `lastServer` - Last server name
- `email` - Email address

**Returns**: A new User instance

---

##### `getPlatformHandle() : PlatformHandle<P, S>`

Returns the platform handle for platform-specific operations.

**Returns**: The platform handle

---

##### `getParsedVersion() : SemanticVersion`

Returns the parsed version of the plugin.

**Returns**: The semantic version

---

##### `getServerHandler() : ServerHandler<P, S>`

Returns the server handler for server management.

**Returns**: The server handler

---

##### `getPremiumProvider() : PremiumProvider`

Returns the premium provider.

**Returns**: The premium provider

---

##### `getImageProjector() : ImageProjector<P, S>`

Returns the image projector for QR code generation.

**Returns**: The image projector, or null if disabled

---

##### `getTOTPProvider() : TOTPProvider`

Returns the TOTP provider for 2FA operations.

**Returns**: The TOTP provider, or null if disabled

---

##### `getEmailHandler() : EmailHandler`

Returns the email handler.

**Returns**: The email handler, or null if disabled

---

##### `validPassword(String password) : boolean`

Validates a password against configuration rules.

**Parameters**:
- `password` - The password to validate

**Returns**: True if valid, false otherwise

**Example**:
```java
boolean isValid = plugin.validPassword(userPassword);
if (!isValid) {
    // Show password requirements
}
```

---

##### `checkDataFolder()`

Creates the data folder if it doesn't exist.

---

##### `generateNewUUID(String name, UUID premiumID) : UUID`

Generates a new UUID based on configuration.

**Parameters**:
- `name` - The player name
- `premiumID` - The premium UUID (optional)

**Returns**: The generated UUID

**Example**:
```java
UUID newUUID = plugin.generateNewUUID(playerName, premiumUUID);
```

---

##### `floodgateEnabled() : boolean`

Checks if Floodgate integration is enabled.

**Returns**: True if enabled, false otherwise

---

##### `luckpermsEnabled() : boolean`

Checks if LuckPerms integration is enabled.

**Returns**: True if enabled, false otherwise

---

##### `fromFloodgate(UUID uuid) : boolean`

Checks if a UUID is from Floodgate.

**Parameters**:
- `uuid` - The UUID to check

**Returns**: True if from Floodgate, false otherwise

---

---

## Event System

### EventProvider

The event provider manages event subscription, unsubscription, and firing.

#### Methods

##### `subscribe(EventType<P, S, E> type, Consumer<E> handler) : Consumer<E>`

Subscribes to an event.

**Type Parameters**:
- `E` - The event type

**Parameters**:
- `type` - The event type
- `handler` - The event handler

**Returns**: The handler instance (for unsubscription)

**Example**:
```java
EventProvider<P, S> eventProvider = plugin.getEventProvider();
Consumer<AuthenticatedEvent> handler = event -> {
    P player = event.getPlayer();
    User user = event.getUser();

    // Custom logic
    plugin.getLogger().info("Player " + player.getName() + " authenticated!");
};

eventProvider.subscribe(
    eventProvider.getTypes().authenticated(),
    handler
);
```

---

##### `unsubscribe(Consumer<? extends Event<P, S>> handler)`

Unsubscribes from an event.

**Parameters**:
- `handler` - The handler to unsubscribe

**Example**:
```java
eventProvider.unsubscribe(handler);
```

---

##### `fire(EventType<P, S, E> type, E event)`

Fires an event to all subscribers.

**Type Parameters**:
- `E` - The event type

**Parameters**:
- `type` - The event type
- `event` - The event to fire

**Example**:
```java
var event = new AuthenticatedEvent(plugin, player, user, reason);
eventProvider.fire(eventProvider.getTypes().authenticated(), event);
```

---

##### `getTypes() : EventTypes<P, S>`

Returns all available event types.

**Returns**: The event types container

---

### Event

Base interface for all events.

#### Methods

##### `getPlugin() : NexAuthPlugin<P, S>`

Gets the plugin instance.

**Returns**: The plugin instance

---

##### `getPlatformHandle() : PlatformHandle<P, S>`

Gets the platform handle.

**Returns**: The platform handle

---

### CancellableEvent

An event that can be cancelled.

#### Methods

##### `isCancelled() : boolean`

Checks if the event is cancelled.

**Returns**: True if cancelled, false otherwise

---

##### `setCancelled(boolean cancelled)`

Sets the cancellation state.

**Parameters**:
- `cancelled` - The cancellation state

---

### Event Types

#### Available Events

##### AuthenticatedEvent

Fired when a player is successfully authenticated.

**Properties**:
- `player : P` - The player
- `user : User` - The user
- `reason : AuthenticationReason` - The authentication reason

**AuthenticationReason enum**:
- `LOGIN` - Player logged in
- `REGISTER` - Player registered
- `SESSION` - Session was used
- `PREMIUM_AUTO_LOGIN` - Automatic premium login

**Example**:
```java
Consumer<AuthenticatedEvent> handler = event -> {
    P player = event.getPlayer();
    AuthenticationReason reason = event.getReason();

    switch (reason) {
        case LOGIN:
            // Handle login
            break;
        case REGISTER:
            // Handle registration
            break;
        case SESSION:
            // Handle session
            break;
        case PREMIUM_AUTO_LOGIN:
            // Handle auto-login
            break;
    }
};
```

---

##### WrongPasswordEvent

Fired when a player enters wrong password.

**Properties**:
- `player : P` - The player
- `attempts : int` - Number of attempts

**Example**:
```java
Consumer<WrongPasswordEvent> handler = event -> {
    P player = event.getPlayer();
    int attempts = event.getAttempts();

    // Log failed attempt
    plugin.getLogger().warn("Player " + player.getName()
        + " entered wrong password (attempt " + attempts + ")");

    // Custom anti-brute force logic
    if (attempts > 5) {
        // Take action
    }
};
```

---

##### PasswordChangeEvent

Fired when a player changes password.

**Properties**:
- `player : P` - The player
- `oldPassword : String` - The old password
- `newPassword : String` - The new password

---

##### LobbyServerChooseEvent

Fired when choosing lobby server.

**Properties**:
- `player : P` - The player
- `serverName : String` - The server name (mutable)

**Example**:
```java
Consumer<LobbyServerChooseEvent> handler = event -> {
    String currentServer = event.getServerName();

    // Custom lobby selection logic
    String customLobby = selectBestLobby();

    event.setServerName(customLobby);
};
```

---

##### LimboServerChooseEvent

Fired when choosing limbo server.

**Properties**:
- `player : P` - The player
- `serverName : String` - The server name (mutable)

---

##### PremiumLoginSwitchEvent

Fired when switching between premium and offline accounts.

**Properties**:
- `player : P` - The player
- `oldUUID : UUID` - Old UUID
- `newUUID : UUID` - New UUID
- `reason : String` - Switch reason

---

---

## Database API

### ReadDatabaseProvider

Interface for reading data from database.

#### Methods

##### `getByUUID(UUID uuid) : User`

Gets a user by UUID.

**Parameters**:
- `uuid` - The UUID

**Returns**: The user, or null if not found

**Example**:
```java
ReadDatabaseProvider reader = ...;
UUID uuid = plugin.getPlatformHandle().getUUIDForPlayer(player);
User user = reader.getByUUID(uuid);

if (user != null) {
    String lastNickname = user.getLastNickname();
    Timestamp lastSeen = user.getLastSeen();
}
```

---

##### `getByName(String name) : User`

Gets a user by name.

**Parameters**:
- `name` - The name (case-insensitive)

**Returns**: The user, or null if not found

**Example**:
```java
User user = reader.getByName("Steve");
if (user != null) {
    UUID uuid = user.getUUID();
}
```

---

##### `getAllUsers() : List<User>`

Gets all users.

**Returns**: List of all users

**Example**:
```java
List<User> allUsers = reader.getAllUsers();
plugin.getLogger().info("Total users: " + allUsers.size());
```

---

### WriteDatabaseProvider

Interface for writing data to database.

#### Methods

##### `insertUser(User user)`

Inserts a new user.

**Parameters**:
- `user` - The user to insert

**Example**:
```java
WriteDatabaseProvider writer = ...;
User newUser = plugin.createUser(
    uuid,
    premiumUUID,
    hashedPassword,
    nickname,
    joinDate,
    lastSeen,
    totpSecret,
    ip,
    lastAuth,
    lastServer,
    email
);
writer.insertUser(newUser);
```

---

##### `updateUser(User user)`

Updates an existing user.

**Parameters**:
- `user` - The user to update

**Example**:
```java
User user = reader.getByUUID(uuid);
if (user != null) {
    user.setLastSeen(new Timestamp(System.currentTimeMillis()));
    writer.updateUser(user);
}
```

---

##### `deleteUser(UUID uuid)`

Deletes a user.

**Parameters**:
- `uuid` - The UUID of the user to delete

---

##### `deleteUser(User user)`

Deletes a user.

**Parameters**:
- `user` - The user to delete

---

### ReadWriteDatabaseProvider

Extends both ReadDatabaseProvider and WriteDatabaseProvider.

---

### User

Represents a user in the database.

#### Methods

##### `getUUID() : UUID`

Returns the UUID.

---

##### `getPremiumUUID() : UUID`

Returns the premium UUID, or null if not premium.

---

##### `getHashedPassword() : HashedPassword`

Returns the hashed password.

---

##### `getLastNickname() : String`

Returns the last used nickname.

---

##### `getJoinDate() : Timestamp`

Returns the join date.

---

##### `getLastSeen() : Timestamp`

Returns the last seen timestamp.

---

##### `getSecret() : String`

Returns the TOTP secret, or null if not set.

---

##### `getIp() : String`

Returns the IP address.

---

##### `getLastAuthentication() : Timestamp`

Returns the last authentication timestamp.

---

##### `getLastServer() : String`

Returns the last server name.

---

##### `getEmail() : String`

Returns the email address, or null if not set.

---

##### Setters

All fields have corresponding setters:

- `setUUID(UUID uuid)`
- `setPremiumUUID(UUID uuid)`
- `setHashedPassword(HashedPassword password)`
- `setLastNickname(String name)`
- `setJoinDate(Timestamp timestamp)`
- `setLastSeen(Timestamp timestamp)`
- `setSecret(String secret)`
- `setIp(String ip)`
- `setLastAuthentication(Timestamp timestamp)`
- `setLastServer(String server)`
- `setEmail(String email)`

---

### DatabaseConnector

Interface for database connections.

#### Methods

##### `connect()`

Establishes database connection.

---

##### `disconnect()`

Closes database connection.

---

##### `isConnected() : boolean`

Checks if connected.

**Returns**: True if connected, false otherwise

---

##### `getConnection() : Connection`

Gets the underlying connection.

**Returns**: The JDBC connection

---

### Supported Database Providers

#### MySQL

**Identifier**: `nexauth-mysql`

**Configuration**:
```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "user"
        password = "password"
        # Connection pool settings
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
        }
    }
}
```

---

#### PostgreSQL

**Identifier**: `nexauth-postgresql`

**Configuration**:
```hocon
database {
    type = "nexauth-postgresql"
    postgresql {
        host = "localhost"
        port = 5432
        database = "nexauth"
        username = "user"
        password = "password"
        # Connection pool settings
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
        }
    }
}
```

---

#### SQLite

**Identifier**: `nexauth-sqlite`

**Configuration**:
```hocon
database {
    type = "nexauth-sqlite"
    sqlite {
        file = "data/nexauth.db"
    }
}
```

---

### Migration Providers

NexAuth supports migration from various plugins:

| Plugin | Identifier |
|--------|------------|
| AuthMe | `authme-mysql`, `authme-postgresql`, `authme-sqlite` |
| FastLogin | `fastlogin-mysql`, `fastlogin-sqlite` |
| LogIt | `logit-mysql` |
| Aegis | `aegis-mysql` |
| DBA | `dba-mysql` |
| JPremium | `jpremium-mysql` |
| NLogin | `nlogin-mysql`, `nlogin-sqlite` |
| LimboAuth | `limboauth-mysql` |
| Authy | `authy-mysql`, `authy-sqlite` |
| LoginSecurity | `loginsecurity-mysql`, `loginsecurity-sqlite` |
| UniqueCodeAuth | `uniquecodeauth-mysql` |

---

---

## Authorization API

### AuthorizationProvider

Manages player authorization state.

#### Methods

##### `isAuthorized(P player) : boolean`

Checks if player is authorized.

**Parameters**:
- `player` - The player

**Returns**: True if authorized, false otherwise

**Example**:
```java
AuthorizationProvider<P> authProvider = plugin.getAuthorizationProvider();

if (!authProvider.isAuthorized(player)) {
    // Redirect to limbo
    plugin.getServerHandler().sendToLimbo(player);
} else {
    // Already authorized, send to lobby
    plugin.getServerHandler().sendToLobby(player);
}
```

---

##### `isAwaiting2FA(P player) : boolean`

Checks if player is in 2FA setup process.

**Parameters**:
- `player` - The player

**Returns**: True if awaiting 2FA confirmation, false otherwise

---

##### `authorize(User user, P player, AuthenticationReason reason)`

Authorizes a player.

**Parameters**:
- `user` - The user
- `player` - The player
- `reason` - The authentication reason

**Example**:
```java
User user = dbProvider.getByUUID(playerUUID);
authProvider.authorize(user, player, AuthenticationReason.LOGIN);
```

---

##### `confirmTwoFactorAuth(P player, Integer code, User user) : boolean`

Confirms 2FA code.

**Parameters**:
- `player` - The player
- `code` - The TOTP code
- `user` - The user

**Returns**: True if valid, false otherwise

**Example**:
```java
TOTPProvider totpProvider = plugin.getTOTPProvider();
boolean isValidCode = totpProvider.verifyCode(user.getSecret(), code);

if (isValidCode) {
    authProvider.confirmTwoFactorAuth(player, code, user);
} else {
    // Invalid code
}
```

---

---

## Crypto API

### CryptoProvider

Interface for password hashing and verification.

#### Methods

##### `createHash(String password) : HashedPassword`

Creates a hash from password.

**Parameters**:
- `password` - Plaintext password

**Returns**: Hashed password, or null if failed

**Example**:
```java
CryptoProvider argon2 = plugin.getCryptoProvider("argon2id");
HashedPassword hashed = argon2.createHash(password);

if (hashed != null) {
    String hashString = hashed.getHash();
    String saltString = hashed.getSalt();
}
```

---

##### `matches(String input, HashedPassword password) : boolean`

Verifies password against hash.

**Parameters**:
- `input` - Plaintext password
- `password` - Hashed password

**Returns**: True if match, false otherwise

**Example**:
```java
User user = dbProvider.getByUUID(playerUUID);
HashedPassword storedPassword = user.getHashedPassword();

boolean isValid = argon2.matches(inputPassword, storedPassword);
if (isValid) {
    // Password correct
}
```

---

##### `getIdentifier() : String`

Gets the algorithm identifier.

**Returns**: The algorithm name (e.g., "argon2id", "bcrypt2a")

---

### HashedPassword

Represents a hashed password with salt.

#### Methods

##### `getHash() : String`

Returns the hash string.

---

##### `getSalt() : String`

Returns the salt string.

---

##### `getAlgorithm() : String`

Returns the algorithm name.

---

### Supported Algorithms

#### Argon2ID (Recommended)

**Identifier**: `argon2id`

**Security Level**: ⭐⭐⭐⭐⭐ (Highest)

**Features**:
- Memory-hard algorithm
- GPU resistant
- Configurable time/memory cost

**Configuration**:
```hocon
crypto {
    default-provider = "argon2id"
    argon2id {
        iterations = 3
        memory = 65536  # 64 MB
        parallelism = 1
    }
}
```

**Example**:
```java
CryptoProvider argon2 = plugin.getCryptoProvider("argon2id");
// Recommended for new installations
```

---

#### BCrypt2A

**Identifier**: `bcrypt2a`

**Security Level**: ⭐⭐⭐⭐ (High)

**Features**:
- Time-cost adjustable
- Widely supported
- Proven track record

**Configuration**:
```hocon
crypto {
    bcrypt2a {
        cost = 12  # 2^cost iterations
    }
}
```

**Example**:
```java
CryptoProvider bcrypt = plugin.getCryptoProvider("bcrypt2a");
// Good for legacy compatibility
```

---

#### SHA-256 / SHA-512

**Identifiers**: `sha256`, `sha512`

**Security Level**: ⭐⭐ (Low) - Legacy only

**Warning**: Only use for migration purposes. Use Argon2ID or BCrypt2A for new installations.

**Example**:
```java
CryptoProvider sha256 = plugin.getCryptoProvider("sha256");
// Only for migration from old systems
```

---

#### LOGIT-SHA-256

**Identifier**: `logit-sha256`

**Security Level**: ⭐⭐ (Low) - Plugin-specific

**Features**:
- Custom algorithm by LogIt plugin
- Only for migration

---

### Custom Crypto Provider

Implement custom algorithm:

```java
public class CustomCryptoProvider implements CryptoProvider {
    @Override
    public HashedPassword createHash(String password) {
        // Generate salt
        SecureRandom random = new SecureRandom();
        byte[] salt = new byte[16];
        random.nextBytes(salt);

        // Hash password
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hashed = digest.digest((password + Base64.getEncoder().encodeToString(salt)).getBytes());

        return new HashedPassword(
            Base64.getEncoder().encodeToString(hashed),
            Base64.getEncoder().encodeToString(salt),
            "custom"
        );
    }

    @Override
    public boolean matches(String input, HashedPassword password) {
        String inputHash = createHash(input).getHash();
        return inputHash.equals(password.getHash());
    }

    @Override
    public String getIdentifier() {
        return "custom";
    }
}
```

---

---

## Configuration API

### HoconPluginConfiguration

Manages plugin configuration using HOCON format.

#### Methods

##### `get(Key<T> key) : T`

Gets a configuration value.

**Type Parameters**:
- `T` - The value type

**Parameters**:
- `key` - The configuration key

**Returns**: The configuration value

**Example**:
```java
HoconPluginConfiguration config = plugin.getConfiguration();

boolean mailEnabled = config.get(MAIL_ENABLED);
int maxAttempts = config.get(MAX_LOGIN_ATTEMPTS);
String defaultLobby = config.get(DEFAULT_LOBBY);
```

---

##### `set(Key<T> key, T value)`

Sets a configuration value.

**Type Parameters**:
- `T` - The value type

**Parameters**:
- `key` - The configuration key
- `value` - The value to set

**Example**:
```java
config.set(MAIL_ENABLED, true);
config.set(MAX_LOGIN_ATTEMPTS, 5);
config.save(); // Don't forget to save!
```

---

##### `reload(NexAuthPlugin<P, S> plugin) : boolean`

Reloads configuration from file.

**Parameters**:
- `plugin` - The plugin instance

**Returns**: True if new config generated, false otherwise

**Example**:
```java
boolean regenerated = config.reload(plugin);
if (regenerated) {
    plugin.getLogger().warn("New configuration generated. Please edit it!");
}
```

---

##### `save()`

Saves configuration to file.

---

### ConfigurationKeys

Contains all configuration keys.

#### Database Configuration

##### `DATABASE_TYPE : Key<String>`

Database provider type (e.g., "nexauth-mysql")

---

##### `DATABASE_MYSQL_* : Key<*>`

MySQL-specific configuration:
- `DATABASE_MYSQL_HOST : Key<String>`
- `DATABASE_MYSQL_PORT : Key<Integer>`
- `DATABASE_MYSQL_DATABASE : Key<String>`
- `DATABASE_MYSQL_USERNAME : Key<String>`
- `DATABASE_MYSQL_PASSWORD : Key<String>`
- `DATABASE_MYSQL_POOL_SIZE : Key<Integer>`

---

##### Similar keys for PostgreSQL and SQLite

---

#### Authentication Configuration

##### `MAX_LOGIN_ATTEMPTS : Key<Integer>`

Maximum login attempts before temporary ban.

**Default**: `5`

---

##### `TEMP_BAN_DURATION : Key<Long>`

Temporary ban duration in milliseconds.

**Default**: `300000` (5 minutes)

---

##### `SESSION_TIMEOUT : Key<Long>`

Session timeout in milliseconds.

**Default**: `3600000` (1 hour)

---

##### `MINIMUM_PASSWORD_LENGTH : Key<Integer>`

Minimum password length.

**Default**: `8`

---

#### 2FA Configuration

##### `TOTP_ENABLED : Key<Boolean>`

Enable/disable 2FA.

**Default**: `false`

---

##### `TOTP_ISSUER : Key<String>`

TOTP issuer name.

**Default**: `"NexAuth"`

---

#### Email Configuration

##### `MAIL_ENABLED : Key<Boolean>`

Enable/disable email functionality.

**Default**: `false`

---

##### `MAIL_HOST : Key<String>`

SMTP server host.

---

##### `MAIL_PORT : Key<Integer>`

SMTP server port.

---

##### `MAIL_USERNAME : Key<String>`

SMTP username.

---

##### `MAIL_PASSWORD : Key<String>`

SMTP password.

---

##### `MAIL_FROM : Key<String>`

From address.

---

#### Server Configuration

##### `LIMBO : Key<List<String>>`

List of limbo servers.

---

##### `LOBBY : Key<Multimap<String, String>>`

Lobby server configuration.

---

##### `DEFAULT_LOBBY : Key<String>`

Default lobby server.

---

##### `REMEMBER_LAST_SERVER : Key<Boolean>`

Remember last server for each player.

**Default**: `true`

---

#### Crypto Configuration

##### `DEFAULT_CRYPTO_PROVIDER : Key<String>`

Default crypto provider identifier.

**Default**: `"argon2id"`

---

##### See Crypto API for provider-specific settings

---

### HoconMessages

Manages localized messages.

#### Methods

##### `getMessage(String key, String... replacements) : TextComponent`

Gets a message by key with replacements.

**Parameters**:
- `key` - The message key
- `replacements` - Placeholder replacements

**Returns**: The formatted message

**Example**:
```java
Messages messages = plugin.getMessages();

// Simple message
TextComponent msg = messages.getMessage("login.success");

// With replacements
TextComponent msg = messages.getMessage(
    "login.welcome",
    player.getName(),
    serverName
);
```

---

##### `reload(NexAuthPlugin<?, ?> plugin)`

Reloads messages from file.

---

### Message Keys

#### Authentication

- `login.success` - Successful login
- `login.welcome` - Welcome message
- `login.wrong-password` - Wrong password error
- `register.success` - Successful registration
- `register.need-confirm` - Password confirmation required
- `register.password-mismatch` - Passwords don't match

---

#### 2FA

- `totp.enabled` - 2FA enabled message
- `totp.disabled` - 2FA disabled message
- `totp.code-required` - Code required message
- `totp.invalid-code` - Invalid code error

---

#### Email

- `mail.sent` - Email sent confirmation
- `mail.verified` - Email verified confirmation
- `mail.reset-sent` - Password reset email sent

---

#### Errors

- `error.player-not-found` - Player not found
- `error.permission-denied` - Permission denied
- `error.command-disabled` - Command disabled
- `error.database-error` - Database error

---

---

## Server Management API

### ServerHandler

Manages player server routing.

#### Methods

##### `chooseLobby(P player) : String`

Chooses a lobby server for player.

**Parameters**:
- `player` - The player

**Returns**: The chosen lobby server name

**Example**:
```java
ServerHandler<P, S> handler = plugin.getServerHandler();
String lobby = handler.chooseLobby(player);
```

---

##### `chooseLimbo(P player) : String`

Chooses a limbo server for player.

**Parameters**:
- `player` - The player

**Returns**: The chosen limbo server name

---

##### `sendToLobby(P player)`

Sends player to lobby.

**Parameters**:
- `player` - The player

**Example**:
```java
handler.sendToLobby(player);
```

---

##### `sendToLimbo(P player)`

Sends player to limbo.

**Parameters**:
- `player` - The player

---

##### `getBestServer(String serverName) : String`

Gets the best server from a group.

**Parameters**:
- `serverName` - The server name

**Returns**: The best server identifier

---

##### `getPlayersServerName(P player) : String`

Gets the server name for a player.

**Parameters**:
- `player` - The player

**Returns**: The server name, or null if not found

---

### ServerPing

Represents server ping information.

#### Methods

##### `getName() : String`

Returns the server name.

---

##### `getPlayers() : int`

Returns number of players.

---

##### `getMaxPlayers() : int`

Returns maximum players.

---

##### `getLatency() : int`

Returns latency in milliseconds.

---

##### `getMotd() : String`

Returns the MOTD.

---

##### `getVersion() : String`

Returns the version.

---

---

## Premium API

### PremiumProvider

Manages premium account integration.

#### Methods

##### `getUserForName(String username) : PremiumUser`

Gets premium user by name.

**Parameters**:
- `username` - The username

**Returns**: The premium user

**Example**:
```java
PremiumProvider premiumProvider = plugin.getPremiumProvider();
try {
    PremiumUser premiumUser = premiumProvider.getUserForName("Notch");
    UUID premiumUUID = premiumUser.getUUID();
    boolean isOnline = premiumUser.isOnline();
} catch (PremiumException e) {
    // Handle error
}
```

---

##### `getUserForUUID(UUID uuid) : PremiumUser`

Gets premium user by UUID.

**Parameters**:
- `uuid` - The UUID

**Returns**: The premium user

---

##### `isPremium(UUID uuid) : boolean`

Checks if UUID is premium.

**Parameters**:
- `uuid` - The UUID

**Returns**: True if premium, false otherwise

---

### PremiumUser

Represents a premium user.

#### Methods

##### `getUUID() : UUID`

Returns the UUID.

---

##### `getName() : String`

Returns the username.

---

##### `isOnline() : boolean`

Checks if player is online.

**Returns**: True if online, false otherwise

---

##### `getProperties() : Map<String, String>`

Returns player properties.

---

### PremiumException

Exception for premium operations.

#### Methods

##### `getIssue() : Issue`

Returns the issue type.

**Issue enum**:
- `THROTTLED` - Too many requests
- `OFFLINE` - Mojang services offline
- `NOT_PREMIUM` - Account not premium
- `UNKNOWN` - Unknown error

---

---

## TOTP API

### TOTPProvider

Manages TOTP (Time-based One-Time Password) for 2FA.

#### Methods

##### `generateSecret() : String`

Generates a new secret.

**Returns**: The secret string

**Example**:
```java
TOTPProvider totpProvider = plugin.getTOTPProvider();
String secret = totpProvider.generateSecret();
```

---

##### `generateQRCode(String secret, String username, String issuer) : BufferedImage`

Generates QR code for authenticator apps.

**Parameters**:
- `secret` - The TOTP secret
- `username` - The username
- `issuer` - The issuer name

**Returns**: QR code image

**Example**:
```java
ImageProjector<P, S> projector = plugin.getImageProjector();
BufferedImage qrCode = projector.generateQRCode(secret, username, "NexAuth");

// Save QR code
projector.saveImage(qrCode, "totp-qr-" + username + ".png");

// Send to player
projector.showImage(player, qrCode);
```

---

##### `verifyCode(String secret, int code) : boolean`

Verifies a TOTP code.

**Parameters**:
- `secret` - The secret
- `code` - The code to verify

**Returns**: True if valid, false otherwise

**Example**```java
String secret = user.getSecret();
int code = Integer.parseInt(codeString);

boolean isValid = totpProvider.verifyCode(secret, code);
if (isValid) {
    // Code valid
} else {
    // Invalid code
}
```

---

##### `generateCodes(String secret) : List<Integer>`

Generates recovery codes.

**Parameters**:
- `secret` - The secret

**Returns**: List of recovery codes

---

### TOTPData

Represents TOTP data.

#### Fields

- `secret : String` - The secret
- `codes : List<Integer>` - Recovery codes
- `enabled : boolean` - Enabled status

---

---

## Command API

### CommandProvider

Manages command registration.

#### Methods

##### `registerCommands(Object commandClass)`

Registers command class.

**Parameters**:
- `commandClass` - The command class

**Example**:
```java
CommandProvider<P, S> commandProvider = plugin.getCommandProvider();
commandProvider.registerCommands(new CustomCommand());
```

---

##### `unregisterCommands(Object commandClass)`

Unregisters command class.

**Parameters**:
- `commandClass` - The command class

---

### Command Structure

NexAuth uses ACF (Aikar Command Framework) for commands.

#### Base Command Classes

- `AuthorizationCommand` - `/login`, `/register`, `/changepassword`
- `TwoFactorAuthCommand` - `/2fa enable`, `/2fa confirm`
- `PremiumCommand` - `/premium enable`, `/premium confirm`
- `EMailCommand` - `/mail set`, `/mail verify`
- `StaffCommand` - `/nexauth reload`, `/nexauth migrate`
- `ChangePasswordCommand` - Password change

---

#### Custom Commands

Create custom command:

```java
public class CustomCommand {
    @CommandPermission("custom.command")
    @CommandCompletion("@players")
    @Default
    public void onCustomCommand(CommandIssuer issuer, @Optional String target) {
        // Command logic
    }
}
```

**Annotations**:

- `@CommandPermission` - Required permission
- `@CommandCompletion` - Tab completion
- `@Default` - Default command
- `@Subcommand` - Subcommand
- `@Optional` - Optional parameter
- `@Flags` - Command flags

---

---

## Mail API

### EmailHandler

Manages email sending.

#### Methods

##### `sendEmail(String to, String subject, TextComponent content)`

Sends an email.

**Parameters**:
- `to` - Recipient email
- `subject` - Email subject
- `content` - Email content

**Example**:
```java
EmailHandler emailHandler = plugin.getEmailHandler();

TextComponent content = Component.text("Your verification code is: " + code);
emailHandler.sendEmail(userEmail, "NexAuth Verification", content);
```

---

##### `sendPasswordResetEmail(String email, String code)`

Sends password reset email.

**Parameters**:
- `email` - Recipient email
- `code` - Reset code

---

##### `sendVerificationEmail(String email, String code)`

Sends verification email.

**Parameters**:
- `email` - Recipient email
- `code` - Verification code

---

---

## PlatformHandle

Platform-specific operations interface.

#### Methods

##### `getPlatformIdentifier() : String`

Returns platform identifier ("bungeecord", "velocity", "paper")

---

##### `getUUIDForPlayer(P player) : UUID`

Gets player UUID.

**Parameters**:
- `player` - The player

**Returns**: The UUID

---

##### `getNameForPlayer(P player) : String`

Gets player name.

**Parameters**:
- `player` - The player

**Returns**: The name

---

##### `getPlayersServerName(P player) : String`

Gets server name for player.

**Parameters**:
- `player` - The player

**Returns**: The server name

---

##### `sendMessage(P player, TextComponent message)`

Sends message to player.

**Parameters**:
- `player` - The player
- `message` - The message

---

##### `kickPlayer(P player, TextComponent reason)`

Kicks player.

**Parameters**:
- `player` - The player
- `reason` - Kick reason

---

##### `getIP(P player) : String`

Gets player IP address.

**Parameters**:
- `player` - The player

**Returns**: The IP address

---

##### `connect(P player, String server)`

Connects player to server.

**Parameters**:
- `player` - The player
- `server` - Server name

---

---

## Logger

Simple logger interface.

#### Methods

##### `info(String message)`

Logs info message.

---

##### `warn(String message)`

Logs warning message.

---

##### `error(String message)`

Logs error message.

---

##### `error(String message, Throwable throwable)`

Logs error with stack trace.

---

##### `debug(String message)`

Logs debug message (if enabled).

---

---

## Utility Classes

### SemanticVersion

Represents semantic version.

#### Methods

##### `parse(String version) : SemanticVersion`

Parses version string.

---

##### `compare(SemanticVersion other) : int`

Compares versions.

**Returns**: Negative if this < other, 0 if equal, positive if this > other

---

##### `dev() : boolean`

Checks if development build.

**Returns**: True if dev build, false otherwise

---

### ThrowableFunction

Functional interface for functions that throw exceptions.

```java
@FunctionalInterface
public interface ThrowableFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}
```

### ThrowableConsumer

Functional interface for consumers that throw exceptions.

```java
@FunctionalInterface
public interface ThrowableConsumer<T, E extends Exception> {
    void accept(T t) throws E;
}
```

### BiHolder

Simple tuple holding two values.

```java
public record BiHolder<T, U>(T first, U second) {}
```

---

## Integration Guide

### Custom Event Listener

```java
public class CustomEventListener {
    @EventListener
    public void onAuthenticated(AuthenticatedEvent event) {
        P player = event.getPlayer();
        User user = event.getUser();

        // Your custom logic
        plugin.getLogger().info("Player " + player.getName() + " authenticated!");

        // Example: Send to custom server
        if (user.getLastServer() != null) {
            plugin.getServerHandler().connect(player, user.getLastServer());
        }
    }

    @EventListener
    public void onWrongPassword(WrongPasswordEvent event) {
        P player = event.getPlayer();
        int attempts = event.getAttempts();

        // Custom anti-brute force
        if (attempts >= 3) {
            // Warn staff
            // plugin.getStaffProvider().notifyStaff(...)
        }
    }
}
```

### Custom Database Provider

```java
public class CustomReadProvider implements ReadDatabaseProvider {
    private final DatabaseConnector<?> connector;
    private final NexAuthPlugin<P, S> plugin;

    public CustomReadProvider(DatabaseConnector<?> connector, NexAuthPlugin<P, S> plugin) {
        this.connector = connector;
        this.plugin = plugin;
    }

    @Override
    public User getByUUID(UUID uuid) {
        try (var statement = connector.getConnection().prepareStatement(
            "SELECT * FROM users WHERE uuid = ?"
        )) {
            statement.setString(1, uuid.toString());
            var result = statement.executeQuery();

            if (result.next()) {
                return mapResultSet(result);
            }
        } catch (Exception e) {
            plugin.getLogger().error("Failed to get user by UUID", e);
        }
        return null;
    }

    private User mapResultSet(ResultSet rs) throws SQLException {
        return plugin.createUser(
            UUID.fromString(rs.getString("uuid")),
            rs.getObject("premium_uuid", UUID.class),
            parsePassword(rs.getString("password")),
            rs.getString("last_nickname"),
            rs.getTimestamp("join_date"),
            rs.getTimestamp("last_seen"),
            rs.getString("secret"),
            rs.getString("ip"),
            rs.getTimestamp("last_authentication"),
            rs.getString("last_server"),
            rs.getString("email")
        );
    }

    private HashedPassword parsePassword(String passwordString) {
        // Parse your password format
        return new HashedPassword(...);
    }
}
```

---

## Best Practices

### 1. Always Use Async Operations

```java
// Good: Async database query
CompletableFuture.supplyAsync(() -> {
    return dbProvider.getByUUID(uuid);
}).thenAccept(user -> {
    // Handle result on async thread
});
```

### 2. Handle Errors Gracefully

```java
try {
    User user = dbProvider.getByUUID(uuid);
    if (user == null) {
        // Handle user not found
        return;
    }
} catch (Exception e) {
    plugin.getLogger().error("Database error", e);
    // Handle database error
}
```

### 3. Use Event System

```java
// Good: Use events for extensibility
var event = new CustomEvent(plugin, player, data);
eventProvider.fire(eventProvider.getTypes().custom(), event);

// Listen to events instead of hardcoding
eventProvider.subscribe(eventProvider.getTypes().authenticated(), event -> {
    // Handle authentication
});
```

### 4. Validate Inputs

```java
// Always validate user input
if (password == null || password.length() < 8) {
    throw new InvalidCommandArgument("Password too short");
}
```

### 5. Use Type-Safe Configuration

```java
// Good: Use configuration keys
boolean enabled = config.get(MAIL_ENABLED);
int maxAttempts = config.get(MAX_LOGIN_ATTEMPTS);

// Avoid raw configuration access
String value = (String) config.getRaw("some.raw.key"); // Avoid this
```

---

## Common Patterns

### Permission Check

```java
public boolean hasPermission(CommandIssuer issuer, String permission) {
    return issuer.hasPermission(permission);
}
```

### Player Validation

```java
public P validatePlayer(CommandIssuer issuer) {
    P player = plugin.getPlayerFromIssuer(issuer);
    if (player == null) {
        throw new InvalidCommandArgument("Not a player");
    }
    return player;
}
```

### Async Task Execution

```java
plugin.delay(() -> {
    // Execute on async thread pool
    performAsyncOperation();
}, 1000); // Delay in milliseconds
```

### Configuration Reload

```java
@CommandPermission("nexauth.reload")
@Default
public void reload(CommandIssuer issuer) {
    try {
        configuration.reload(plugin);
        messages.reload(plugin);
        issuer.sendMessage(Messages.getMessage("reload.success"));
    } catch (Exception e) {
        issuer.sendMessage(Messages.getMessage("reload.error"));
        plugin.getLogger().error("Reload failed", e);
    }
}
```

---

## Error Codes

| Code | Error | Description |
|------|-------|-------------|
| 1001 | InvalidCredentials | Wrong username or password |
| 1002 | UserNotFound | User doesn't exist |
| 1003 | UserExists | User already exists |
| 1004 | PasswordTooShort | Password doesn't meet requirements |
| 1005 | PasswordWeak | Password is too weak |
| 1006 | TooManyAttempts | Too many login attempts |
| 1007 | 2FARequired | 2FA code required |
| 1008 | 2FAInvalid | Invalid 2FA code |
| 1009 | EmailInvalid | Invalid email address |
| 1010 | DatabaseError | Database operation failed |
| 1011 | PermissionDenied | Missing permission |
| 1012 | CommandDisabled | Command is disabled |
| 1013 | PremiumOnly | Feature only for premium users |
| 1014 | ServerOffline | Server is offline |
| 1015 | NetworkError | Network operation failed |

---

## Examples

### Full Authentication Flow

```java
public class LoginCommand {
    @Default
    public void onLogin(CommandIssuer issuer, String password) {
        P player = plugin.getPlayerFromIssuer(issuer);

        // Check if already authorized
        if (plugin.getAuthorizationProvider().isAuthorized(player)) {
            issuer.sendMessage(plugin.getMessages().getMessage("login.already"));
            return;
        }

        // Get user
        UUID uuid = plugin.getPlatformHandle().getUUIDForPlayer(player);
        User user = plugin.getDatabaseProvider().getByUUID(uuid);

        if (user == null) {
            issuer.sendMessage(plugin.getMessages().getMessage("login.not-registered"));
            return;
        }

        // Verify password
        CryptoProvider crypto = plugin.getDefaultCryptoProvider();
        if (!crypto.matches(password, user.getHashedPassword())) {
            // Fire wrong password event
            var wrongEvent = new WrongPasswordEvent(
                plugin,
                player,
                plugin.getLoginTryListener().getAttempts(player)
            );
            plugin.getEventProvider().fire(
                plugin.getEventProvider().getTypes().wrongPassword(),
                wrongEvent
            );
            return;
        }

        // Check 2FA
        if (user.getSecret() != null) {
            // Send 2FA challenge
            issuer.sendMessage(plugin.getMessages().getMessage("login.2fa-required"));
            // Store partial auth state
            return;
        }

        // Authorize
        plugin.getAuthorizationProvider().authorize(
            user,
            player,
            AuthenticationReason.LOGIN
        );

        // Fire authenticated event
        var authEvent = new AuthenticatedEvent(
            plugin,
            player,
            user,
            AuthenticationReason.LOGIN
        );
        plugin.getEventProvider().fire(
            plugin.getEventProvider().getTypes().authenticated(),
            authEvent
        );
    }
}
```

### Custom Event Handler

```java
public class DatabaseBackupHandler {
    @EventListener
    public void onAuthenticated(AuthenticatedEvent event) {
        // Backup user data after authentication
        P player = event.getPlayer();
        User user = event.getUser();

        // Async backup
        CompletableFuture.runAsync(() -> {
            try {
                backupUser(user);
                plugin.getLogger().debug("Backed up user: " + user.getLastNickname());
            } catch (Exception e) {
                plugin.getLogger().error("Backup failed for user: " + user.getLastNickname(), e);
            }
        });
    }

    private void backupUser(User user) throws IOException {
        // Custom backup logic
        String backupData = GSON.toJson(user);
        Files.write(
            Paths.get("backups/" + user.getUUID() + ".json"),
            backupData.getBytes()
        );
    }
}
```

---

## Performance Tips

### 1. Use Connection Pooling

```java
// Database connection is automatically pooled via HikariCP
// Just configure pool size in config:
database {
    mysql {
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
        }
    }
}
```

### 2. Batch Database Operations

```java
// Instead of individual inserts
for (User user : users) {
    dbProvider.insertUser(user); // Slow
}

// Use batch
dbProvider.insertUsers(users); // Fast
```

### 3. Cache Frequently Used Data

```java
// Cache user data
Map<UUID, User> userCache = new ConcurrentHashMap<>();

// In getByUUID
public User getByUUID(UUID uuid) {
    return userCache.computeIfAbsent(uuid, this::loadUserFromDatabase);
}
```

### 4. Use Async Operations

```java
// Always use async for blocking operations
CompletableFuture.runAsync(() -> {
    // Database queries
    // Network requests
    // File I/O
});
```

### 5. Avoid N+1 Queries

```java
// Bad: N+1 query problem
for (User user : users) {
    user.setServer(getServerName(user.getUUID())); // Query for each user
}

// Good: Batch query
Map<UUID, String> serverMap = batchGetServerNames(userUUIDs);
for (User user : users) {
    user.setServer(serverMap.get(user.getUUID()));
}
```

---

## Security Considerations

### 1. Never Store Plaintext Passwords

```java
// Good: Always hash
CryptoProvider crypto = plugin.getDefaultCryptoProvider();
HashedPassword hashed = crypto.createHash(password);

// Bad: NEVER do this
String plaintext = password; // DON'T!
```

### 2. Use Prepared Statements

```java
// Good: Prepared statement
PreparedStatement stmt = connection.prepareStatement(
    "SELECT * FROM users WHERE uuid = ?"
);
stmt.setString(1, uuid.toString());
ResultSet rs = stmt.executeQuery();

// Bad: SQL injection vulnerable
String query = "SELECT * FROM users WHERE uuid = '" + uuid + "'";
// DON'T!
```

### 3. Validate All Inputs

```java
// Always validate
if (username == null || username.length() < 3) {
    throw new InvalidCommandArgument("Username too short");
}

if (!username.matches("[a-zA-Z0-9_]+")) {
    throw new InvalidCommandArgument("Invalid username");
}
```

### 4. Use Event Cancellation

```java
// Allow other plugins to cancel auth
@EventListener
public void onPreAuthenticate(PreAuthenticateEvent event) {
    if (!isBlacklisted(event.getPlayer())) {
        event.setCancelled(true);
    }
}
```

---

## Troubleshooting

### Common Issues

#### Issue: User is always redirected to limbo

**Cause**: Authorization state not properly set

**Solution**:
```java
// After authentication
authProvider.authorize(user, player, reason);
// Make sure to fire event
eventProvider.fire(types.authenticated(), event);
```

#### Issue: Database connection timeouts

**Cause**: Connection pool exhausted

**Solution**:
```hocon
database {
    mysql {
        pool {
            maximumPoolSize = 20  # Increase pool size
            connectionTimeout = 30000  # Increase timeout
        }
    }
}
```

#### Issue: 2FA codes not working

**Cause**: Time drift between server and authenticator

**Solution**:
```java
// Check time sync
long serverTime = System.currentTimeMillis() / 1000;
// Allow 30 second window for time drift
for (int i = -1; i <= 1; i++) {
    if (verifyCode(secret, code, serverTime + i * 30)) {
        return true;
    }
}
```

#### Issue: Players can join backend servers directly

**Cause**: Backend servers not secured

**Solution**:
1. Use BungeeGuard
2. Configure firewall rules
3. Only allow proxy IP addresses
4. Use NexLimbo for authentication

---

## Changelog

### Version 0.0.1-beta3

**Added**:
- Argon2ID support
- Event system improvements
- MySQL, PostgreSQL, SQLite support
- TOTP 2FA
- Email verification
- Migration from multiple plugins

**For full changelog, see** [CHANGELOG.md](../CHANGELOG.md)

---

## References

- [Official Website](https://nexauth.xreatlabs.xyz)
- [Wiki Documentation](https://github.com/xreatlabs/NexAuth/wiki)
- [GitHub Repository](https://github.com/xreatlabs/NexAuth)
- [Discord Support](https://discord.gg/m5ptM8X2Du)
- [Javadoc](../API/build/docs/javadoc/index.html)

---

## License

NexAuth API is licensed under the Mozilla Public License 2.0.

See [LICENSE](../LICENSE) for details.

---

*This API reference is automatically generated from source code. For the latest updates, visit the [GitHub repository](https://github.com/xreatlabs/NexAuth).*
