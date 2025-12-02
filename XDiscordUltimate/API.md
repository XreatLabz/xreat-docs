# XDiscordUltimate API Documentation

## Table of Contents

1. [Core Classes](#core-classes)
2. [Module System](#module-system)
3. [Database API](#database-api)
4. [Discord Integration](#discord-integration)
5. [Utility Classes](#utility-classes)
6. [Event Listeners](#event-listeners)
7. [Commands](#commands)

---

## Core Classes

### XDiscordUltimate

Main plugin class that extends `JavaPlugin` and serves as the central entry point.

#### Key Methods

**`public static XDiscordUltimate getInstance()`**
- Returns the singleton instance of the plugin

**`public ConfigManager getConfigManager()`**
- Returns the configuration manager instance

**`public DatabaseManager getDatabaseManager()`**
- Returns the database manager instance

**`public DiscordManager getDiscordManager()`**
- Returns the Discord bot manager instance

**`public ModuleManager getModuleManager()`**
- Returns the module manager instance

**`public void reload()`**
- Reloads all configurations and modules

**`public String parsePlaceholders(String text, Player player)`**
- Parses PlaceholderAPI placeholders in text
- @param text The text to parse
- @param player The player context (can be null)
- @return Parsed text with placeholders replaced

**`public boolean isPlaceholderAPIEnabled()`**
- Checks if PlaceholderAPI is available

**`public boolean isLuckPermsEnabled()`**
- Checks if LuckPerms is available

**`public Object getLuckPerms()`**
- Returns LuckPerms API instance

---

## Module System

### Module (Abstract Base Class)

Base interface for all XDiscordUltimate modules.

#### Abstract Methods

**`public abstract String getName()`**
- Returns the module name

**`public abstract String getDescription()`**
- Returns the module description

**`protected abstract void onEnable()`**
- Called when module is enabled

**`protected abstract void onDisable()`**
- Called when module is disabled

#### Virtual Methods

**`public void onDiscordReady()`**
- Called when Discord bot is ready
- Override in modules that need Discord functionality

**`public void reload()`**
- Reloads the module configuration

#### Helper Methods

**`protected ConfigurationSection getConfig()`**
- Gets the module's configuration section

**`protected boolean isSubFeatureEnabled(String subFeature)`**
- Checks if a sub-feature is enabled

**`protected void debug(String message)`**
- Logs a debug message (only if debug mode enabled)

**`protected void info(String message)`**
- Logs an info message

**`protected void warning(String message)`**
- Logs a warning message

**`protected void error(String message)`**
- Logs an error message

**`protected void error(String message, Throwable throwable)`**
- Logs an error message with exception

**`protected String getCommandFromMessage(String message)`**
- Extracts command from message (removes ! or / prefix)
- @param message The message to check
- @return Command without prefix, or null if no valid prefix

**`protected boolean isCommand(String message)`**
- Checks if message is a command (starts with ! or /)
- @param message The message to check
- @return true if message is a command

### ModuleManager

Manages loading, enabling, disabling, and reloading of modules.

#### Key Methods

**`public void loadModules()`**
- Loads all configured modules from the plugin directory

**`public void disableModules()`**
- Disables all loaded modules

**`public void reloadModules()`**
- Reloads all modules (disables, clears, and re-loads)

**`public void onDiscordReady()`**
- Notifies all enabled modules that Discord is ready

**`public Module getModule(String key)`**
- Gets a module by key

**`public <T extends Module> T getModule(Class<T> moduleClass)`**
- Gets a module by class type

**`public Map<String, Module> getModules()`**
- Returns all loaded modules

**`public boolean isModuleActive(String key)`**
- Checks if a module is loaded and enabled

---

## Database API

### DatabaseManager

Handles all database operations using HikariCP for connection pooling.

#### Connection Management

**`public void initialize() throws SQLException`**
- Initializes database connection pool
- Creates all necessary tables
- Configures based on database type (SQLite, MySQL, PostgreSQL)

**`public Connection getConnection() throws SQLException`**
- Gets a database connection from the pool

**`public void close()`**
- Closes the database connection pool

#### Async Operations

**`public <T> CompletableFuture<T> executeAsync(DatabaseOperation<T> operation)`**
- Executes a database operation asynchronously
- @param operation The operation to execute
- @return CompletableFuture with the result

#### Account Linking

**`public CompletableFuture<Boolean> linkAccount(UUID minecraftUuid, String discordId, String minecraftName, String discordName)`**
- Links a Discord account with Minecraft account
- Uses database-specific upsert syntax

**`public CompletableFuture<String> getDiscordId(UUID minecraftUuid)`**
- Gets Discord ID from Minecraft UUID
- @return Discord ID or null if not linked

**`public CompletableFuture<UUID> getMinecraftUuid(String discordId)`**
- Gets Minecraft UUID from Discord ID
- @return UUID or null if not linked

**`public CompletableFuture<String> getMinecraftName(String discordId)`**
- Gets Minecraft username from Discord ID

**`public CompletableFuture<Boolean> isDiscordLinked(String discordId)`**
- Checks if Discord user has linked their account

#### Verification Codes

**`public void storeVerificationCode(String discordId, String code, String username)`**
- Stores a verification code for Discord user
- Codes expire after 10 minutes by default

**`public VerificationCode getVerificationCode(String code)`**
- Gets verification code info
- @return VerificationCode object or null if not found/expired

**`public void removeVerificationCode(String code)`**
- Removes a used verification code

**`public void cleanExpiredVerificationCodes()`**
- Removes all expired verification codes

#### Tickets

**`public CompletableFuture<Integer> createTicket(UUID playerUuid, String subject)`**
- Creates a new support ticket
- @return Ticket ID or -1 if failed

**`public CompletableFuture<Boolean> updateTicketChannel(int ticketId, String channelId)`**
- Associates Discord channel with ticket

**`public CompletableFuture<Boolean> closeTicket(int ticketId)`**
- Closes a support ticket

**`public CompletableFuture<Boolean> addTicketMessage(int ticketId, UUID senderUuid, String senderType, String message)`**
- Adds a message to a ticket

**`public CompletableFuture<Integer> getOpenTicketsCount(UUID playerUuid)`**
- Gets count of open tickets for a player

#### Moderation

**`public CompletableFuture<Integer> createReport(UUID reporterUuid, UUID targetUuid, String reason)`**
- Creates a player report

**`public CompletableFuture<Void> logModerationAction(String actionType, UUID targetUuid, UUID moderatorUuid, String reason, Timestamp expiresAt)`**
- Logs a moderation action (ban, kick, warn, mute)

**`public CompletableFuture<Boolean> hasRecentReport(UUID reporterUuid, int cooldownMinutes)`**
- Checks if player has recent reports (cooldown check)

#### Player Statistics

**`public CompletableFuture<Void> updatePlayerStats(UUID playerUuid, String statType, int increment)`**
- Updates player statistics
- Supports: joins, messages_sent, deaths, playtime_minutes
- Uses database-specific upsert syntax

#### VerificationCode (Inner Class)

**`public String getDiscordId()`**
- Returns Discord ID

**`public String getCode()`**
- Returns verification code

**`public String getUsername()`**
- Returns Minecraft username

**`public long getCreatedAt()`**
- Returns creation timestamp

**`public long getExpiresAt()`**
- Returns expiration timestamp

**`public boolean isExpired()`**
- Checks if code is expired

---

## Discord Integration

### DiscordManager

Manages Discord bot connection and operations using JDA 5.

#### Initialization

**`public CompletableFuture<Boolean> initialize()`**
- Initializes Discord bot connection
- Configures intents, cache policies, and event listeners
- Waits for ready state before completing
- @return CompletableFuture with success status

**`public void shutdown()`**
- Shuts down Discord bot gracefully

#### Connection Status

**`public boolean isReady()`**
- Checks if bot is connected and ready

**`public JDA getJDA()`**
- Returns JDA instance

**`public Guild getMainGuild()`**
- Returns main Discord guild

#### Channel Management

**`public TextChannel getTextChannel(String name)`**
- Gets text channel by name

**`public TextChannel getTextChannelById(String id)`**
- Gets text channel by ID
- Returns null if ID is invalid or not configured

**`public TextChannel getChatChannel()`**
- Gets configured chat bridge channel

**`public TextChannel getLogsChannel()`**
- Gets configured logs channel

**`public TextChannel getEventsChannel()`**
- Gets configured events channel

**`public TextChannel getTicketsChannel()`**
- Gets configured tickets channel

**`public TextChannel getModerationChannel()`**
- Gets configured moderation channel

**`public TextChannel getConsoleChannel()`**
- Gets configured console channel

**`public TextChannel getWelcomeChannel()`**
- Gets configured welcome channel

#### Activity Management

**`public void updateActivity()`**
- Updates bot activity/status
- Uses placeholders %players% and %max%
- Reads activity type and text from config

**`private void startStatusUpdater()`**
- Starts automatic status updater (runs every 60 seconds)

#### Debugging

**`public void logConfiguredChannels()`**
- Logs all configured channels to console
- Shows which channels are valid/invalid

### DiscordListener

Handles Discord bot events.

#### Event Handlers

**`public void onReady(ReadyEvent event)`**
- Called when Discord bot is ready

**`public void onMessageReceived(MessageReceivedEvent event)`**
- Called when message is received
- Handles commands and chat bridge

**`public void onGuildMemberJoin(GuildMemberJoinEvent event)`**
- Called when member joins guild

**`public void onGuildMemberRemove(GuildMemberRemoveEvent event)`**
- Called when member leaves guild

---

## Utility Classes

### ConfigManager

Manages plugin configuration files and settings.

#### Configuration Files

**`public FileConfiguration getConfig(String name)`**
- Gets a configuration file by name (e.g., "config.yml", "messages.yml")

**`public void saveConfig(String name)`**
- Saves a configuration file

**`public void reload()`**
- Reloads all configuration files

#### Feature Management

**`public boolean isFeatureEnabled(String feature)`**
- Checks if a feature is enabled in config

**`public ConfigurationSection getFeatureConfig(String feature)`**
- Gets configuration section for a feature

**`public boolean isSubFeatureEnabled(String feature, String subFeature)`**
- Checks if a sub-feature is enabled

#### Utility Methods

**`public String getColoredString(String path)`**
- Gets string with color codes translated (& -> §)

**`public String getString(String path, Map<String, String> placeholders)`**
- Gets string with placeholders replaced

**`public List<String> getAdminIDs()`**
- Gets list of admin Discord IDs

**`public String getBotToken()`**
- Gets Discord bot token

**`public String getGuildId()`**
- Gets Discord guild ID

**`public String getDatabaseType()`**
- Gets database type (sqlite, mysql, postgresql)

**`public ConfigurationSection getDatabaseConfig()`**
- Gets database configuration section

**`public boolean isDebugEnabled()`**
- Checks if debug mode enabled

**`public String getLanguage()`**
- Gets configured language

**`public String getMessageFormat(String key)`**
- Gets message format by key

### MessageManager

Manages localized messages and formatting.

#### Key Methods

**`public String getMessage(String key, Player player)`**
- Gets message with placeholders parsed
- @param key Message key
- @param player Player context for placeholders
- @return Formatted message with color codes

**`public String getMessage(String key)`**
- Gets message without player context

### AdminUtils

Provides admin utility functions.

#### Key Methods

**`public boolean isAdmin(Player player)`**
- Checks if player is a plugin admin

**`public boolean isAdmin(String discordId)`**
- Checks if Discord user is a plugin admin

### EmbedUtils

Provides Discord embed building utilities.

#### Key Methods

**`public MessageEmbed buildEmbed(String title, String description, int color, String thumbnailUrl, String imageUrl, String footer)`**
- Builds a Discord embed

**`public MessageEmbed buildPlayerEmbed(Player player)`**
- Builds embed with player information and skin

### ColorUtils

Provides color code conversion utilities.

#### Key Methods

**`public static String parseColors(String text)`**
- Converts & color codes to Minecraft color codes (§)

### VersionUtils

Provides version checking utilities.

#### Key Methods

**`public static boolean isSupported()`**
- Checks if server version is supported

**`public static String getVersionString()`**
- Gets formatted server version

**`public static String getSupportedVersions()`**
- Gets list of supported versions

### UpdateChecker

Checks for plugin updates.

#### Key Methods

**`public void checkForUpdates()`**
- Checks for updates on startup

---

## Event Listeners

### PlayerListener

Handles player-related events.

#### Event Handlers

**`public void onPlayerJoin(PlayerJoinEvent event)`**
- Called when player joins
- Handles first join detection, welcome DMs, stats tracking

**`public void onPlayerQuit(PlayerQuitEvent event)`**
- Called when player quits
- Updates last seen, playtime stats

**`public void onPlayerAdvancement(PlayerAdvancementEvent event)`**
- Called when player makes advancement
- Sends Discord notification if enabled

**`public void onPlayerDeath(PlayerDeathEvent event)`**
- Called when player dies
- Sends Discord notification if enabled

### ChatListener

Handles chat-related events.

#### Event Handlers

**`public void onAsyncPlayerChat(AsyncPlayerChatEvent event)`**
- Called when player sends chat message
- Handles chat bridge, chat filter, stats tracking

### ServerListener

Handles server events.

#### Event Handlers

**`public void onServerCommand(ServerCommandEvent event)`**
- Called when server command is executed
- Handles console command logging

---

## Commands

### VerifyCommand

Handles account verification commands.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /verify command

### SupportCommand

Handles support ticket creation.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /support command

### ReportCommand

Handles player reporting.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /report command

### AnnounceCommand

Handles announcements.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /announce command

### HelpCommand

Provides help GUI.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /help command

### StatsCommand

Displays player statistics.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /stats command

### XDiscordCommand

Main admin command.

**`public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args)`**
- Handles /xdiscord command
- Subcommands: reload, status, help, modules

---

## Database Schema

### verified_users
- minecraft_uuid (VARCHAR(36), PRIMARY KEY)
- discord_id (VARCHAR(20), NOT NULL, UNIQUE)
- verified_at (TIMESTAMP)
- minecraft_name (VARCHAR(16))
- discord_name (VARCHAR(32))

### verification_codes
- discord_id (VARCHAR(20), PRIMARY KEY)
- code (VARCHAR(10), NOT NULL, UNIQUE)
- username (VARCHAR(16))
- created_at (BIGINT)
- expires_at (BIGINT)

### tickets
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- minecraft_uuid (VARCHAR(36))
- discord_channel_id (VARCHAR(20))
- status (VARCHAR(20))
- created_at (TIMESTAMP)
- closed_at (TIMESTAMP)
- subject (TEXT)
- FOREIGN KEY (minecraft_uuid) REFERENCES verified_users(minecraft_uuid)

### ticket_messages
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- ticket_id (INTEGER)
- sender_uuid (VARCHAR(36))
- sender_type (VARCHAR(10))
- message (TEXT)
- sent_at (TIMESTAMP)
- FOREIGN KEY (ticket_id) REFERENCES tickets(id)

### moderation_logs
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- action_type (VARCHAR(20))
- target_uuid (VARCHAR(36))
- moderator_uuid (VARCHAR(36))
- reason (TEXT)
- timestamp (TIMESTAMP)
- expires_at (TIMESTAMP)
- active (BOOLEAN)

### player_stats
- minecraft_uuid (VARCHAR(36), PRIMARY KEY)
- joins (INTEGER)
- messages_sent (INTEGER)
- deaths (INTEGER)
- playtime_minutes (INTEGER)
- last_seen (TIMESTAMP)
- first_join (TIMESTAMP)

### activity_rewards
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- discord_id (VARCHAR(20))
- reward_type (VARCHAR(50))
- reward_data (TEXT)
- claimed (BOOLEAN)
- created_at (TIMESTAMP)
- claimed_at (TIMESTAMP)

---

## PlaceholderAPI Expansion

### PlaceholderAPIExpansion

Provides PlaceholderAPI placeholders for XDiscordUltimate.

#### Available Placeholders

**`%xdiscord_player_joined%`**
- Returns formatted join message

**`%xdiscord_player_left%`**
- Returns formatted leave message

**`%xdiscord_player_deaths%`**
- Returns player death count

**`%xdiscord_player_joins%`**
- Returns player join count

**`%xdiscord_is_verified%`**
- Returns "true" or "false"

**`%xdiscord_verified_role%`**
- Returns Discord verification role name

**`%xdiscord_discord_name%`**
- Returns linked Discord username

**`%xdiscord_server_online%`**
- Returns number of online players

**`%xdiscord_server_max%`**
- Returns max player count

**`%xdiscord_server_tps%`**
- Returns server TPS

---

## Hooks and Dependencies

### LuckPerms Hook

**Integration Points:**
- Prefix/suffix display in Discord
- Group-based verification (verified-group)
- Booster perks (permission-group)
- Activity roles integration

**Detection:**
- Plugin checks if LuckPerms is present on startup
- Automatically hooks if available
- Gracefully degrades if not present

### PlaceholderAPI Hook

**Integration Points:**
- Placeholder parsing for messages
- Custom placeholders for Discord embeds
- Player stats placeholders

**Detection:**
- Plugin checks if PlaceholderAPI is present
- Registers expansion automatically
- Uses reflection for optional dependency

### Vault Hook (Planned)

**Integration Points:**
- Economy bridge module
- Balance tracking
- Transaction logging

---

## Best Practices

### Module Development

1. **Extend the Module class** for new features
2. **Check configuration** before enabling features
3. **Handle Discord readiness** in onDiscordReady()
4. **Use async operations** for database/network calls
5. **Log appropriately** using module's logging methods
6. **Check dependencies** gracefully (LuckPerms, PlaceholderAPI)

### Database Operations

1. **Always use async** for database operations
2. **Use CompletableFuture** for chaining operations
3. **Handle SQLExceptions** appropriately
4. **Use try-with-resources** for connections
5. **Clean up resources** in module disable()

### Discord Integration

1. **Wait for Discord ready** before posting messages
2. **Use configured channels** from DiscordManager
3. **Handle rate limits** appropriately
4. **Cache channel lookups** when possible
5. **Handle missing permissions** gracefully

### Configuration

1. **Provide sensible defaults**
2. **Document all config options**
3. **Validate config values** on load
4. **Handle missing/invalid config** gracefully
5. **Support runtime reload** of config

---

## Error Handling

### Exception Management

**Logging:**
- Use module's built-in logging methods (info, warning, error, debug)
- Log with module name prefix for clarity
- Include stack traces for unexpected exceptions

**Graceful Degradation:**
- Disable specific features rather than crashing
- Log warnings when optional dependencies missing
- Continue plugin load even if non-critical modules fail

**Recovery:**
- Implement cleanup in disable() methods
- Close database connections properly
- Cancel scheduled tasks on disable

### Common Error Scenarios

1. **Discord bot fails to connect**
   - Check bot token validity
   - Verify guild ID is correct
   - Check bot permissions

2. **Database connection fails**
   - Verify database credentials
   - Check database server availability
   - Verify database exists

3. **Channel not found**
   - Verify channel IDs are correct
   - Check bot has access to channel
   - Enable developer mode in Discord

4. **Module fails to enable**
   - Check module configuration
   - Verify required channels are set
   - Check database connectivity

---

## Performance Considerations

### Database

1. **Connection Pooling:** HikariCP manages connections
2. **Async Operations:** All database calls are async
3. **Indexes:** Auto-created on frequently queried columns
4. **Prepared Statements:** Used for all queries
5. **Batch Updates:** Used for bulk operations

### Discord

1. **Caching:** JDA caches members, roles, channels
2. **Rate Limits:** Automatic handling by JDA
3. **Presence Updates:** Throttled to 60 seconds
4. **Message Caching:** Configurable cache size

### Minecraft

1. **Async Listeners:** Chat and events handled async
2. **Task Scheduling:** Use Bukkit async scheduler
3. **Memory:** Regular cleanup of temporary objects
4. **Thread Safety:** Use proper synchronization

---

## Extension Points

### Custom Modules

1. Create class extending Module
2. Implement required methods (getName, getDescription, onEnable, onDisable)
3. Register in ModuleManager.loadModules()
4. Add configuration in config.yml
5. Update feature toggles

### Custom Commands

1. Extend CommandExecutor or implement TabCompleter
2. Register in XDiscordUltimate.registerCommands()
3. Handle permissions and validation
4. Support internationalization

### Custom Listeners

1. Implement Listener interface
2. Use @EventHandler annotation
3. Register in XDiscordUltimate.registerListeners()
4. Handle events asynchronously where possible

### Database Extensions

1. Add table creation to DatabaseManager.createTables()
2. Create repository methods in DatabaseManager
3. Update schema documentation
4. Handle database-specific syntax

---

This API documentation provides comprehensive coverage of XDiscordUltimate's architecture and implementation details. For module-specific documentation, see the individual module documentation files.