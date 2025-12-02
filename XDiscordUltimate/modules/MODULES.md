# XDiscordUltimate Modules Documentation

## Table of Contents

1. [Module Overview](#module-overview)
2. [Core Modules](#core-modules)
3. [Feature Modules](#feature-modules)
4. [Advanced Modules](#advanced-modules)
5. [Module Configuration](#module-configuration)
6. [Module Dependencies](#module-dependencies)
7. [Troubleshooting](#troubleshooting)

---

## Module Overview

XDiscordUltimate uses a modular architecture where each feature is implemented as a separate module. This approach provides:

- **Independent operation**: Each module can be enabled/disabled separately
- **Reduced conflicts**: Features don't interfere with each other
- **Easy customization**: Enable only what you need
- **Improved performance**: Unused features don't consume resources
- **Simplified debugging**: Issues are isolated to specific modules

### Module Structure

All modules extend the base `Module` class and implement:
- `getName()` - Module display name
- `getDescription()` - Brief description
- `onEnable()` - Initialization logic
- `onDisable()` - Cleanup logic

Additional optional methods:
- `onDiscordReady()` - Called when Discord bot connects

---

## Core Modules

### 1. Chat Bridge Module

**Status**: Core Module (Always Required)

**Description**: Bridges chat messages between Minecraft and Discord in real-time

**Key Features**:
- Bidirectional chat synchronization
- Webhook mode for player's Minecraft skin avatars (like DiscordSRV)
- Plain text mode fallback
- Player prefix/suffix support (with LuckPerms)
- Mention handling and notifications
- Emoji conversion
- Typing indicators
- Custom message formatting
- Bot message filtering

#### Configuration

```yaml
features:
  chat-bridge:
    enabled: true
    chat-channel-id: "DISCORD_CHANNEL_ID"
    chat-channel: "minecraft-chat"  # Fallback name
    minecraft-to-discord: true
    discord-to-minecraft: true
    filter-bot-messages: true

    # Webhook Mode (Recommended)
    auto-create-webhook: true
    use-webhook: false
    webhook-url: ""
    skin-provider: "crafatar"  # crafatar, minotar, mc-heads, visage

    # Message Formats
    minecraft-format: "**%player%**: %message%"
    discord-format: "&7[&9Discord&7] &b%user%&7: &f%message%"

    # Enhanced Features
    show-mentions: true
    play-mention-sound: true
    convert-emojis: true
    strip-discord-formatting: false
    show-typing-indicator: true

    # Player Name Format
    show-player-prefix: true
    show-player-suffix: false
    discord-username-format: "%prefix%%player%"
```

#### How It Works

1. **Minecraft to Discord**:
   - Player sends message in Minecraft
   - Plugin captures AsyncPlayerChatEvent
   - Formats message with player's name and skin
   - Sends via webhook to Discord channel
   - Discord displays as if player posted directly

2. **Discord to Minecraft**:
   - User sends message in Discord
   - Plugin captures MessageReceivedEvent
   - Filters bot messages (if enabled)
   - Formats for Minecraft chat
   - Broadcasts to all online players

#### Commands

- **In Minecraft**: No commands (automatic)
- **In Discord**: No commands (automatic)

#### Permissions

- No specific permissions required
- Chat bridge works for all online players
- Use LuckPerms integration for prefix/suffix display

#### Dependencies

- **Required**: Discord bot, configured channel
- **Optional**: LuckPerms (for prefixes), webhooks

---

### 2. Server Logging Module

**Status**: Core Module

**Description**: Logs Minecraft server events to Discord channels

**Key Features**:
- Player join/leave notifications
- Player death messages
- Advancement broadcasts
- Server start/stop logging
- Command usage tracking
- Configurable ignored commands
- Embed or plain text format

#### Configuration

```yaml
features:
  server-logging:
    enabled: true
    logs-channel-id: "DISCORD_CHANNEL_ID"

    # Event Types
    log-server-events: true
    log-join-leave: true
    log-deaths: true
    log-advancements: true
    log-commands: true

    # Ignored Commands
    ignored-commands:
      - "afk"
      - "balancetop"
      - "seen"

    # Message Formats
    use-embeds: true
    join-message: ":arrow_right: **%player%** joined the server"
    leave-message: ":arrow_left: **%player%** left the server"
    death-message: ":skull: **%player%** %death_message%"
    advancement-message: ":trophy: **%player%** has made the advancement **[%advancement%]**"
    command-message: "```[%timestamp%] %player% used command: %command%```"
```

#### Log Events

1. **Server Events**:
   - Server start
   - Server stop
   - Reload

2. **Player Events**:
   - First join
   - Join
   - Leave
   - Death
   - Advancement

3. **Command Events**:
   - All commands (except ignored)
   - Includes timestamp and player

#### Dependencies

- **Required**: Discord bot, logs channel
- **Optional**: PlaceholderAPI (for advancement placeholders)

---

## Feature Modules

### 3. Verification Module

**Status**: Popular Module

**Description**: Code-based system to link Discord accounts with Minecraft accounts

**Key Features**:
- Random verification code generation
- Time-limited codes (default 5 minutes)
- Automatic role assignment
- Permission group assignment (with LuckPerms)
- Whitelist mode support
- Cooldown protection
- DM-based code delivery
- In-game verification command

#### Configuration

```yaml
features:
  verification:
    enabled: true
    kick-after-minutes: 0  # 0 = disabled
    whitelist-mode: false
    verified-role: "Verified"
    verified-group: "verified"  # LuckPerms group
    code-length: 6
    code-expiry-minutes: 5
```

#### Verification Process

1. **Discord User**:
   - Runs `/verify` slash command
   - Receives DM with 6-digit code
   - Gives code to Minecraft player

2. **Minecraft Player**:
   - Runs `/verify <code>`
   - Plugin validates code
   - Links accounts in database
   - Assigns Discord role
   - Adds to permission group (if LuckPerms)

#### Commands

- **Discord**: `/verify` (slash command)
- **Minecraft**: `/verify <code>`

#### Database Tables

- `verified_users` - Linked accounts
- `verification_codes` - Temporary codes

#### Permissions

- **Minecraft**: `xdiscord.verify` (default: true)

---

### 4. Tickets Module

**Status**: Popular Module

**Description**: Full-featured support ticket system with Discord integration

**Key Features**:
- Automatic Discord channel creation
- Ticket categories
- Auto-close after inactivity
- Transcript saving
- Staff notifications
- Player-close permission
- Message logging

#### Configuration

```yaml
features:
  tickets:
    enabled: true
    ticket-category: "Support Tickets"
    auto-close-hours: 48
    player-close: true
    save-transcript: true
    transcript-channel: "ticket-logs"
```

#### Ticket Flow

1. **Create Ticket**:
   - Player runs `/support <message>`
   - Plugin creates ticket in database
   - Creates Discord channel in category
   - Notifies staff
   - Posts ticket creation message

2. **Ticket Conversation**:
   - Player and staff can message
   - All messages logged to database
   - Participants receive notifications

3. **Close Ticket**:
   - Staff closes via button
   - Player can close (if enabled)
   - Transcript saved (if enabled)
   - Channel archived/deleted

#### Commands

- **Minecraft**: `/support <message>`

#### Database Tables

- `tickets` - Ticket records
- `ticket_messages` - Ticket messages

#### Permissions

- **Minecraft**: `xdiscord.support` (default: true)

---

### 5. Moderation Module

**Status**: Popular Module

**Description**: Synchronized moderation between Minecraft and Discord

**Key Features**:
- Ban synchronization
- Chat filtering
- Player reporting
- Moderation logging
- Cooldown protection
- Auto-moderation

#### Configuration

```yaml
features:
  moderation:
    enabled: true
    sync-bans: true
    log-channel: "mod-logs"
    auto-moderate: true
    report-cooldown-minutes: 5
    require-verification-for-reports: true

    # Filtered Words (regex)
    filter-words:
      - "badword1"
      - "badword2"
```

#### Moderation Features

1. **Reporting System**:
   - Players can report others
   - Reports sent to moderation channel
   - Includes timestamps and evidence

2. **Ban Sync**:
   - Bans in Minecraft sync to Discord
   - Discord bans can trigger Minecraft kicks
   - Unbans synchronized

3. **Chat Filter**:
   - Auto-moderates filtered words
   - Can delete messages
   - Can warn/kick offenders

#### Commands

- **Minecraft**: `/report <player> <reason>`

#### Database Tables

- `moderation_logs` - All moderation actions

#### Permissions

- **Minecraft**: `xdiscord.report` (default: true)

---

### 6. Admin Alerts Module

**Status**: Utility Module

**Description**: Monitors server health and sends alerts to Discord

**Key Features**:
- TPS monitoring
- RAM usage tracking
- Customizable thresholds
- DM alerts to admins
- Configurable check interval

#### Configuration

```yaml
features:
  admin-alerts:
    enabled: true
    tps-threshold: 15.0
    ram-threshold: 90
    check-interval: 30  # seconds
    dm-alerts: true
    alert-channel: "admin-alerts"
```

#### Alert Types

1. **Low TPS**:
   - Triggers when TPS drops below threshold
   - Alerts sent to channel and DMs

2. **High RAM**:
   - Monitors memory usage percentage
   - Alerts when exceeding threshold

3. **Custom Checks**:
   - Can be extended for other metrics
   - Plugin exposes API for custom alerts

#### Permissions

- No specific permissions
- Alerts go to configured admins and channel

---

### 7. Bot Console Module

**Status**: Admin Module

**Description**: Execute Minecraft server commands from Discord

**Key Features**:
- Discord command execution
- DM command support (configurable)
- Command history
- Permission-based access
- Safe command filtering

#### Configuration

```yaml
features:
  bot-console:
    enabled: true
    command-prefix: "!"
    allow-dm: false

# Also configure in main config:
discord-console:
  enabled: true
  channel-id: "CONSOLE_CHANNEL_ID"
  allowed-roles:
    - "Admin"
    - "Owner"
```

#### Console Commands

- **Discord**: `!tps` - View server TPS
- **Discord**: `!list` - List online players
- **Discord**: `!restart` - Restart server (if implemented)
- **Discord**: `!stop` - Stop server

#### Security

- Restricted to allowed roles only
- Command prefix prevents accidental execution
- Dangerous commands can be whitelisted

#### Permissions

- **Discord**: Allowed roles configured
- **Minecraft**: Not applicable

---

### 8. Announcements Module

**Status**: Admin Module

**Description**: Broadcast announcements to both Discord and Minecraft

**Key Features**:
- Cross-platform broadcasting
- Scheduled announcements
- Custom sounds and titles
- Embed formatting
- Permission-based

#### Configuration

```yaml
features:
  announcements:
    enabled: true
    default-channel: "announcements"
    show-title: true
    title-stay: 100
    title-fade-in: 10
    title-fade-out: 20
    prefix: "&6[&eAnnouncement&6] &f"
    sound: "ENTITY_EXPERIENCE_ORB_PICKUP"
    discord-color: "#FFD700"

    # Scheduled Announcements
    scheduled:
      server-info:
        enabled: true
        title: "Server Information"
        interval: 600
        minecraft: true
        discord: true
        messages:
          - "&bWelcome to our server!"
          - "&eWebsite: &fexample.com"
```

#### Announcement Types

1. **Manual Announcements**:
   - Triggered by command
   - Sent to both platforms

2. **Scheduled Announcements**:
   - Auto-broadcast at intervals
   - Can target specific platform
   - Permission-gated

#### Commands

- **Minecraft**: `/announce <message>`

#### Permissions

- **Minecraft**: `xdiscord.announce` (default: op)

---

### 9. Player Stats Module

**Status**: Tracking Module

**Description**: Tracks and displays player statistics

**Key Features**:
- Join count tracking
- Message count tracking
- Death count tracking
- Playtime tracking
- Last seen timestamps
- First join tracking

#### Configuration

```yaml
features:
  player-stats:
    enabled: true
```

#### Tracked Statistics

1. **Basic Stats**:
   - Number of joins
   - Messages sent in chat
   - Number of deaths
   - Total playtime (minutes)

2. **Timestamps**:
   - First join date
   - Last seen date

#### Commands

- **Minecraft**: `/stats [player]`
- **Discord**: `/stats [player]`

#### Database Tables

- `player_stats` - Player statistics

#### Permissions

- No specific permissions for viewing own stats
- Can view others' stats without permission

---

### 10. Server Status Module

**Status**: Information Module

**Description**: Auto-updating Discord embed showing server status

**Key Features**:
- Live player count
- TPS display
- Memory usage
- Version information
- Uptime tracking
- Auto-updating embed

#### Configuration

```yaml
features:
  server-status:
    enabled: true
    channel-id: "STATUS_CHANNEL_ID"
    update-interval: 60  # seconds
    message-id: ""  # Auto-filled
```

#### Status Information

1. **Server Metrics**:
   - Online players / max players
   - Current TPS
   - Memory usage
   - Server version

2. **Embed Features**:
   - Color-coded status
   - Auto-updating
   - Customizable format

#### Display Channel

- Dedicated channel for status embed
- Message ID tracked for updates
- Automatically creates and updates message

---

### 11. Welcome DM Module

**Status**: Engagement Module

**Description**: Sends personalized welcome messages to verified Discord users

**Key Features**:
- First-join only option
- Verified users only option
- Embed or plain text
- Customizable fields
- Server information
- Thumbnail support

#### Configuration

```yaml
features:
  welcome-dm:
    enabled: true
    first-join-only: true
    verified-only: true
    use-embed: true

    # Plain Text
    plain-message: "Welcome to our server, %player%! Thanks for linking your Discord account."

    # Embed Settings
    embed:
      title: "Welcome to %server_name%!"
      description: |
        Hey **%player%**! Welcome to our Minecraft server!

        Thanks for linking your Discord account. Here's some useful info to get you started:
      color: "#55FFFF"
      thumbnail: "%player_head%"
      image: ""
      footer: "Sent via XDiscordUltimate"

      fields:
        - "Server IP|play.example.com|true"
        - "Website|https://example.com|true"
        - "Need Help?|Use `/support <message>` in-game or open a ticket in Discord!|false"
```

#### DM Content

1. **Personalization**:
   - Player name
   - Minecraft head avatar
   - Server name
   - Online player count

2. **Server Information**:
   - IP address
   - Website
   - Discord invite
   - Quick start guide

#### Dependencies

- **Required**: Discord bot, verified users
- **Optional**: Player heads API

---

### 12. Booster Perks Module

**Status**: Engagement Module

**Description**: Rewards Discord server boosters with Minecraft perks

**Key Features**:
- Native booster detection
- Custom booster role support
- Permission group assignment
- Automatic sync
- Reward commands
- Remove commands (when unboosting)

#### Configuration

```yaml
features:
  booster-perks:
    enabled: true
    permission-group: "booster"
    sync-interval: 300

    # Detection Options
    use-native-detection: true
    booster-role-id: ""
    additional-booster-roles: []

    # Rewards
    reward-commands:
      - "give %player% diamond 5"
      - "broadcast &d%player% &fis now boosting the Discord server! &d&lThank you!"

    remove-commands: []
```

#### Booster Detection

1. **Native Detection**:
   - Uses Discord's `isBoosting()` method
   - Most reliable method

2. **Role-Based Detection**:
   - Custom booster role ID
   - Additional roles array
   - Backup if native fails

#### Reward System

- **On Boost**: Executes reward commands
- **On Unboost**: Executes remove commands
- **Periodic Sync**: Checks every 5 minutes

#### Dependencies

- **Required**: LuckPerms (for permission groups)
- **Optional**: Custom booster roles

---

## Advanced Modules

### 13. Leaderboard Module

**Status**: Gamification Module

**Description**: Auto-updating leaderboard with interactive buttons

**Key Features**:
- Multiple leaderboard types (KILLS, DEATHS, PLAYTIME, etc.)
- Top 10 players display
- Interactive buttons
- Auto-updating
- Color-coded ranking

#### Configuration

```yaml
features:
  leaderboard:
    enabled: true
    channel-id: "LEADERBOARD_CHANNEL_ID"
    message-id: ""
    update-interval: 300  # seconds
    top-players: 10
    default-type: "KILLS"
```

#### Leaderboard Types

1. **KILLS**: Player kill count
2. **DEATHS**: Player death count
3. **KD_RATIO**: Kills/deaths ratio
4. **PLAYTIME**: Total playtime
5. **MOB_KILLS**: Mob kill count
6. **MESSAGES**: Chat messages sent

#### Interactive Features

- Button to switch between leaderboard types
- Auto-refresh on interval
- Displays player rank and value

---

### 14. Activity Roles Module

**Status**: Engagement Module

**Description**: Auto-assigns Discord roles based on Minecraft playtime

**Key Features**:
- Multi-tier role system
- Automatic promotion
- Playtime tracking
- Color-coded roles
- Announcement support

#### Configuration

```yaml
features:
  activity-roles:
    enabled: true
    sync-interval: 300
    remove-old-roles: true
    announce-promotion: true

    # Tier Configuration
    tiers:
      newcomer:
        display-name: "🌱 Newcomer"
        min-hours: 0
        role-id: ""
        color: "&a"
      regular:
        display-name: "⭐ Regular"
        min-hours: 10
        role-id: ""
        color: "&e"
      veteran:
        display-name: "💎 Veteran"
        min-hours: 50
        role-id: ""
        color: "&b"
      elite:
        display-name: "👑 Elite"
        min-hours: 100
        role-id: ""
        color: "&6"
      legend:
        display-name: "🏆 Legend"
        min-hours: 500
        role-id: ""
        color: "&d"
```

#### Tier System

1. **Automatic Promotion**:
   - Calculates total playtime
   - Assigns appropriate role
   - Removes lower tier roles
   - Announces to player

2. **Sync Process**:
   - Runs every 5 minutes
   - Checks all active players
   - Updates Discord roles
   - Sends notifications

---

### 15. Voice Channels Module

**Status**: Advanced Module

**Description**: Dynamic voice channels and player count status channel

**Key Features**:
- Status channel with live player count
- Dynamic voice channel creation
- Auto-deletion of empty channels
- User limit support
- Custom channel naming

#### Configuration

```yaml
features:
  voice-channels:
    enabled: true

    # Status Channel
    enable-status-channel: true
    status-channel-id: ""
    status-channel-format: "📊 Players: %online%/%max%"
    update-interval: 60

    # Dynamic Channels
    enable-dynamic-channels: true
    category-id: ""
    hub-channel-id: ""
    dynamic-channel-prefix: "🎮 Gaming Room"
    max-dynamic-channels: 10
    user-limit: 0
```

#### Voice Channel Features

1. **Status Channel**:
   - Shows player count
   - Shows TPS
   - Auto-updates
   - Voice channel type

2. **Dynamic Channels**:
   - Created when joining hub
   - Named with prefix + number
   - Deleted when empty
   - User limit support

---

### 16. Economy Bridge Module

**Status**: Integration Module

**Description**: Discord commands for economy integration (requires Vault)

**Key Features**:
- Balance checking
- Top players list
- Pay command support
- Transfer taxes
- Currency customization

#### Configuration

```yaml
features:
  economy-bridge:
    enabled: true
    currency-symbol: "$"
    currency-name: "coins"
    top-players: 10
    allow-transfers: true
    min-transfer: 1.0
    max-transfer: 1000000.0
    transfer-tax: 0.0
```

#### Economy Commands

- **Discord**: `/balance` - Check balance
- **Discord**: `/baltop` - Top 10 richest players
- **Discord**: `/pay <player> <amount>` - Transfer money

#### Dependencies

- **Required**: Vault, Economy plugin (EssentialsX, CMI, etc.)

---

## Module Configuration

### Enabling/Disabling Modules

```yaml
# Disable a module
features:
  chat-bridge:
    enabled: false

# Enable a module
features:
  tickets:
    enabled: true
```

### Module Load Order

1. **Core Modules** (Always loaded):
   - Chat Bridge
   - Server Logging

2. **Feature Modules** (Loaded when enabled):
   - Verification
   - Tickets
   - Moderation
   - Admin Alerts
   - Bot Console
   - Announcements
   - Player Stats
   - Server Status
   - Welcome DM
   - Booster Perks

3. **Advanced Modules** (Loaded when enabled):
   - Leaderboard
   - Activity Roles
   - Voice Channels
   - Economy Bridge

### Module Dependencies

| Module | Required Dependencies | Optional Dependencies |
|--------|----------------------|----------------------|
| Chat Bridge | Discord Bot | LuckPerms, Webhooks |
| Server Logging | Discord Bot | PlaceholderAPI |
| Verification | Discord Bot, Database | LuckPerms |
| Tickets | Discord Bot, Database | - |
| Moderation | Discord Bot, Database | - |
| Admin Alerts | - | - |
| Bot Console | Discord Bot | - |
| Announcements | Discord Bot | - |
| Player Stats | Database | - |
| Server Status | Discord Bot | - |
| Welcome DM | Discord Bot | Player heads API |
| Booster Perks | LuckPerms | - |
| Leaderboard | Database | - |
| Activity Roles | Database, LuckPerms | - |
| Voice Channels | Discord Bot | - |
| Economy Bridge | Vault, Economy Plugin | - |

---

## Module Development

### Creating a Custom Module

1. **Extend Module class**:
```java
public class CustomModule extends Module {
    public CustomModule(XDiscordUltimate plugin) {
        super(plugin);
    }

    @Override
    public String getName() {
        return "Custom Module";
    }

    @Override
    public String getDescription() {
        return "Description of custom module";
    }

    @Override
    protected void onEnable() {
        info("Custom module enabled!");
    }

    @Override
    protected void onDisable() {
        info("Custom module disabled!");
    }
}
```

2. **Register in ModuleManager**:
```java
registerModule("custom-module", new CustomModule(plugin));
```

3. **Add configuration**:
```yaml
features:
  custom-module:
    enabled: true
    custom-setting: "value"
```

### Module Best Practices

1. **Configuration**:
   - Always check if enabled before processing
   - Load configuration in onEnable()
   - Provide sensible defaults

2. **Discord Integration**:
   - Wait for onDiscordReady() before using Discord
   - Remove event listeners in onDisable()
   - Handle missing permissions gracefully

3. **Database Operations**:
   - Use async operations (CompletableFuture)
   - Always handle SQLExceptions
   - Close resources properly

4. **Logging**:
   - Use module's built-in logging methods
   - Include module name in logs
   - Log at appropriate level (info, warning, severe)

5. **Error Handling**:
   - Never let exceptions crash the plugin
   - Catch and log errors gracefully
   - Provide fallback behavior

---

## Troubleshooting

### Module Not Working

1. **Check Enabled Status**:
   ```yaml
   features:
     module-name:
       enabled: true  # Must be true
   ```

2. **Check Configuration**:
   - Verify all required fields are set
   - Check for typos in channel IDs
   - Ensure Discord bot has permissions

3. **Check Dependencies**:
   - Verify optional dependencies are installed
   - Check logs for missing dependency warnings

4. **Check Logs**:
   - Enable debug mode in config
   - Look for module-specific error messages
   - Check for stack traces

### Module Causes Errors

1. **Check Configuration**:
   - Invalid configuration values
   - Missing required fields
   - Wrong data types

2. **Check Permissions**:
   - Discord bot missing channel permissions
   - Minecraft missing bukkit permissions

3. **Check Dependencies**:
   - Optional dependencies not installed
   - Version incompatibilities

### Module Performance Issues

1. **Check Update Intervals**:
   - Reduce update frequency if too high
   - Some modules update every second by default

2. **Check Database Queries**:
   - Enable debug mode to see queries
   - Optimize frequent queries
   - Add indexes if needed

3. **Check Async Operations**:
   - Ensure long operations are async
   - Don't block main thread
   - Use Bukkit scheduler for async tasks

### Module Not Loading

1. **Check Module Registration**:
   - Module must be registered in ModuleManager
   - Check ModuleManager.loadModules()

2. **Check isFeatureEnabled()**:
   - Configuration must have `features.<module>.enabled: true`

3. **Check for Exceptions**:
   - Errors in onEnable() prevent module loading
   - Check logs for stack traces

---

This documentation provides comprehensive coverage of all XDiscordUltimate modules. Each module is designed to work independently while providing powerful features for Discord-Minecraft integration.