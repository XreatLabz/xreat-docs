# XDiscordUltimate Database & Configuration Reference

## Table of Contents

1. [Database Overview](#database-overview)
2. [Database Setup](#database-setup)
3. [Database Schema](#database-schema)
4. [Configuration Overview](#configuration-overview)
5. [Configuration Reference](#configuration-reference)
6. [Message Configuration](#message-configuration)
7. [Migration Guide](#migration-guide)
8. [Backup and Restore](#backup-and-restore)
9. [Troubleshooting](#troubleshooting)

---

## Database Overview

XDiscordUltimate uses a relational database to store:
- Linked Discord and Minecraft accounts
- Verification codes
- Support tickets and messages
- Player statistics
- Moderation logs
- Activity rewards

### Supported Databases

| Database | Status | Recommended For |
|----------|--------|-----------------|
| **SQLite** | ✅ Built-in | Small servers (1-50 players) |
| **MySQL** | ✅ Supported | Medium-Large servers (50+ players) |
| **PostgreSQL** | ✅ Supported | Large servers (200+ players) |

### Database Selection Guide

**SQLite** (Default)
- ✅ No external dependencies
- ✅ Easy setup (file-based)
- ✅ Good for small servers
- ❌ Limited concurrent connections
- ❌ Not suitable for high-load scenarios

**MySQL**
- ✅ Production-ready
- ✅ Good performance
- ✅ Widely supported
- ❌ Requires MySQL server
- ❌ More complex setup

**PostgreSQL**
- ✅ Advanced features
- ✅ Excellent performance
- ✅ ACID compliance
- ❌ Requires PostgreSQL server
- ❌ Higher resource usage

---

## Database Setup

### SQLite (Default)

No setup required! SQLite database is created automatically on first run.

**Database Location**: `{plugin-folder}/data.db`

**Configuration**:
```yaml
database:
  type: "sqlite"
  sqlite:
    file: "data.db"
```

### MySQL Setup

#### Step 1: Create MySQL Database

```sql
-- Login to MySQL
mysql -u root -p

-- Create database
CREATE DATABASE xdiscord;

-- Create user
CREATE USER 'xdiscord'@'localhost' IDENTIFIED BY 'your_password';

-- Grant permissions
GRANT ALL PRIVILEGES ON xdiscord.* TO 'xdiscord'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
```

#### Step 2: Configure Plugin

```yaml
database:
  type: "mysql"
  mysql:
    host: "localhost"
    port: 3306
    database: "xdiscord"
    username: "xdiscord"
    password: "your_password"
    ssl: false
```

#### Step 3: Connection Pool Settings

For high-load servers, adjust HikariCP settings in code or create `hikari.properties`:

```properties
# Connection Pool Settings
dataSource.maximumPoolSize=20
dataSource.minimumIdle=5
dataSource.idleTimeout=300000
dataSource.connectionTimeout=30000
dataSource.leakDetectionThreshold=60000
dataSource.validationTimeout=5000
```

### PostgreSQL Setup

#### Step 1: Create PostgreSQL Database

```sql
-- Login to PostgreSQL
psql -U postgres

-- Create database
CREATE DATABASE xdiscord;

-- Create user
CREATE USER xdiscord WITH PASSWORD 'your_password';

-- Grant permissions
GRANT ALL PRIVILEGES ON DATABASE xdiscord TO xdiscord;

-- Connect to database
\c xdiscord

-- Grant schema permissions
GRANT ALL ON SCHEMA public TO xdiscord;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO xdiscord;
```

#### Step 2: Configure Plugin

```yaml
database:
  type: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "xdiscord"
    username: "xdiscord"
    password: "your_password"
```

---

## Database Schema

### Table: verified_users

**Purpose**: Maps Minecraft accounts to Discord accounts for verification

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| minecraft_uuid | VARCHAR(36) | PRIMARY KEY | PK | Player UUID (hyphenated) |
| discord_id | VARCHAR(20) | NOT NULL, UNIQUE | UNIQUE | Discord user ID |
| verified_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Verification timestamp |
| minecraft_name | VARCHAR(16) | - | - | Minecraft username |
| discord_name | VARCHAR(32) | - | - | Discord username |

**DDL**:
```sql
CREATE TABLE verified_users (
    minecraft_uuid VARCHAR(36) PRIMARY KEY,
    discord_id VARCHAR(20) NOT NULL UNIQUE,
    verified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    minecraft_name VARCHAR(16),
    discord_name VARCHAR(32)
);

CREATE INDEX idx_verified_users_discord_id ON verified_users(discord_id);
```

**Sample Data**:
```sql
INSERT INTO verified_users (minecraft_uuid, discord_id, minecraft_name, discord_name)
VALUES ('550e8400-e29b-41d4-a716-446655440000', '123456789012345678', 'Steve', 'Steve#1234');
```

### Table: verification_codes

**Purpose**: Temporary storage for verification codes

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| discord_id | VARCHAR(20) | PRIMARY KEY | PK | Discord user ID |
| code | VARCHAR(10) | NOT NULL, UNIQUE | UNIQUE | Generated code |
| username | VARCHAR(16) | - | - | Minecraft username |
| created_at | BIGINT | NOT NULL | INDEX | Creation timestamp (ms) |
| expires_at | BIGINT | NOT NULL | INDEX | Expiration timestamp (ms) |

**DDL**:
```sql
CREATE TABLE verification_codes (
    discord_id VARCHAR(20) PRIMARY KEY,
    code VARCHAR(10) NOT NULL UNIQUE,
    username VARCHAR(16),
    created_at BIGINT NOT NULL,
    expires_at BIGINT NOT NULL
);

CREATE INDEX idx_verification_code ON verification_codes(code);
CREATE INDEX idx_verification_expires ON verification_codes(expires_at);
```

**Sample Data**:
```sql
INSERT INTO verification_codes (discord_id, code, username, created_at, expires_at)
VALUES ('123456789012345678', 'ABC123', 'Steve', 1640000000000, 1640000300000);
```

### Table: tickets

**Purpose**: Support ticket management

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | PK | Ticket ID |
| minecraft_uuid | VARCHAR(36) | NOT NULL, FOREIGN KEY | INDEX | Player UUID |
| discord_channel_id | VARCHAR(20) | - | - | Discord channel ID |
| status | VARCHAR(20) | DEFAULT 'OPEN' | INDEX | OPEN, CLOSED, PENDING |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Creation time |
| closed_at | TIMESTAMP | NULL | - | Close time |
| subject | TEXT | - | - | Ticket subject |

**DDL**:
```sql
CREATE TABLE tickets (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    minecraft_uuid VARCHAR(36) NOT NULL,
    discord_channel_id VARCHAR(20),
    status VARCHAR(20) DEFAULT 'OPEN',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    closed_at TIMESTAMP NULL,
    subject TEXT,
    FOREIGN KEY (minecraft_uuid) REFERENCES verified_users(minecraft_uuid)
);

CREATE INDEX idx_tickets_status ON tickets(status);
CREATE INDEX idx_tickets_player ON tickets(minecraft_uuid);
```

**Sample Data**:
```sql
INSERT INTO tickets (minecraft_uuid, discord_channel_id, subject, status)
VALUES ('550e8400-e29b-41d4-a716-446655440000', '987654321098765432', 'Need help', 'OPEN');
```

### Table: ticket_messages

**Purpose**: Messages within tickets

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | PK | Message ID |
| ticket_id | INTEGER | NOT NULL, FOREIGN KEY | INDEX | Ticket ID |
| sender_uuid | VARCHAR(36) | - | - | Sender UUID (if player) |
| sender_type | VARCHAR(10) | - | - | PLAYER or STAFF |
| message | TEXT | - | - | Message content |
| sent_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Send time |

**DDL**:
```sql
CREATE TABLE ticket_messages (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    ticket_id INTEGER NOT NULL,
    sender_uuid VARCHAR(36),
    sender_type VARCHAR(10),
    message TEXT,
    sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES tickets(id)
);

CREATE INDEX idx_ticket_messages_ticket ON ticket_messages(ticket_id);
```

### Table: moderation_logs

**Purpose**: Moderation action logging

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | PK | Log ID |
| action_type | VARCHAR(20) | NOT NULL | INDEX | BAN, KICK, WARN, MUTE, REPORT |
| target_uuid | VARCHAR(36) | NOT NULL | INDEX | Target UUID |
| moderator_uuid | VARCHAR(36) | - | INDEX | Moderator UUID |
| reason | TEXT | - | - | Action reason |
| timestamp | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Action time |
| expires_at | TIMESTAMP | NULL | - | Expiration time (for temp bans) |
| active | BOOLEAN | DEFAULT TRUE | INDEX | Still active flag |

**DDL**:
```sql
CREATE TABLE moderation_logs (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    action_type VARCHAR(20) NOT NULL,
    target_uuid VARCHAR(36) NOT NULL,
    moderator_uuid VARCHAR(36),
    reason TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NULL,
    active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_mod_logs_action ON moderation_logs(action_type);
CREATE INDEX idx_mod_logs_target ON moderation_logs(target_uuid);
CREATE INDEX idx_mod_logs_active ON moderation_logs(active);
```

**Sample Data**:
```sql
INSERT INTO moderation_logs (action_type, target_uuid, moderator_uuid, reason)
VALUES ('BAN', '550e8400-e29b-41d4-a716-446655440000', '650e8400-e29b-41d4-a716-446655440001', 'Cheating');
```

### Table: player_stats

**Purpose**: Player statistics tracking

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| minecraft_uuid | VARCHAR(36) | PRIMARY KEY | PK | Player UUID |
| joins | INTEGER | DEFAULT 0 | - | Join count |
| messages_sent | INTEGER | DEFAULT 0 | - | Chat messages |
| deaths | INTEGER | DEFAULT 0 | - | Death count |
| playtime_minutes | INTEGER | DEFAULT 0 | INDEX | Total playtime (minutes) |
| last_seen | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Last activity |
| first_join | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | First join time |

**DDL**:
```sql
CREATE TABLE player_stats (
    minecraft_uuid VARCHAR(36) PRIMARY KEY,
    joins INTEGER DEFAULT 0,
    messages_sent INTEGER DEFAULT 0,
    deaths INTEGER DEFAULT 0,
    playtime_minutes INTEGER DEFAULT 0,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    first_join TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_player_stats_playtime ON player_stats(playtime_minutes);
```

**Sample Data**:
```sql
INSERT INTO player_stats (minecraft_uuid, joins, messages_sent, deaths, playtime_minutes)
VALUES ('550e8400-e29b-41d4-a716-446655440000', 5, 150, 2, 120);
```

### Table: activity_rewards

**Purpose**: Discord activity reward tracking

| Column | Type | Constraints | Index | Description |
|--------|------|-------------|-------|-------------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT | PK | Reward ID |
| discord_id | VARCHAR(20) | NOT NULL | INDEX | Discord user ID |
| reward_type | VARCHAR(50) | - | - | Type of reward |
| reward_data | TEXT | - | - | Reward data (JSON) |
| claimed | BOOLEAN | DEFAULT FALSE | INDEX | Claimed flag |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | - | Creation time |
| claimed_at | TIMESTAMP | NULL | - | Claim time |

**DDL**:
```sql
CREATE TABLE activity_rewards (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    discord_id VARCHAR(20) NOT NULL,
    reward_type VARCHAR(50),
    reward_data TEXT,
    claimed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    claimed_at TIMESTAMP NULL
);

CREATE INDEX idx_activity_rewards_discord ON activity_rewards(discord_id);
CREATE INDEX idx_activity_rewards_claimed ON activity_rewards(claimed);
```

### Database Indexes

**Purpose of Indexes**:
- Improve query performance
- Speed up lookups
- Enable efficient sorting

**Indexes Created**:
```sql
-- verified_users table
CREATE INDEX idx_verified_users_discord_id ON verified_users(discord_id);

-- verification_codes table
CREATE INDEX idx_verification_code ON verification_codes(code);
CREATE INDEX idx_verification_expires ON verification_codes(expires_at);

-- tickets table
CREATE INDEX idx_tickets_status ON tickets(status);

-- ticket_messages table
CREATE INDEX idx_ticket_messages_ticket ON ticket_messages(ticket_id);

-- moderation_logs table
CREATE INDEX idx_mod_logs_action ON moderation_logs(action_type);
CREATE INDEX idx_mod_logs_target ON moderation_logs(target_uuid);
CREATE INDEX idx_mod_logs_active ON moderation_logs(active);

-- player_stats table
CREATE INDEX idx_player_stats_playtime ON player_stats(playtime_minutes);

-- activity_rewards table
CREATE INDEX idx_activity_rewards_discord ON activity_rewards(discord_id);
CREATE INDEX idx_activity_rewards_claimed ON activity_rewards(claimed);
```

---

## Configuration Overview

XDiscordUltimate uses YAML configuration files:

1. **config.yml** - Main plugin configuration
2. **messages.yml** - Customizable messages and text

### Configuration File Locations

```
plugin-folder/
├── XDiscordUltimate/
│   ├── config.yml
│   ├── messages.yml
│   └── data.db  (SQLite database)
```

### Configuration Loading Order

1. Plugin loads defaults from JAR
2. Creates files if they don't exist
3. Loads user's configuration
4. Merges with defaults
5. Validates configuration
6. Logs warnings for invalid values

---

## Configuration Reference

### config.yml Structure

```yaml
# Admin Discord IDs
adminIDs: []

# General Settings
general:
  debug: false
  language: "en_US"
  check-updates: true

# Feature Toggles
features:
  # Core Modules
  chat-bridge: { }
  server-logging: { }

  # Feature Modules
  verification: { }
  tickets: { }
  moderation: { }
  admin-alerts: { }
  bot-console: { }
  announcements: { }
  player-stats: { }
  server-status: { }
  welcome-dm: { }
  booster-perks: { }

  # Advanced Modules
  leaderboard: { }
  activity-roles: { }
  voice-channels: { }
  economy-bridge: { }

# Discord Bot Settings
discord:
  bot-token: "YOUR_BOT_TOKEN"
  guild-id: "YOUR_GUILD_ID"
  activity: { }
  commands: { }
  channels: { }

# Database Settings
database:
  type: "sqlite"
  sqlite: { }
  mysql: { }

# Message Formats
messages:
  minecraft-to-discord: "**%player%**: %message%"
  discord-to-minecraft: "&7[&9Discord&7] &b%user%&7: &f%message%"
  join-message: ":arrow_right: **%player%** joined the server!"
  leave-message: ":arrow_left: **%player%** left the server!"
  death-message: ":skull: **%player%** %death_message%"

# Advanced Settings
advanced:
  thread-pool-size: 4
  cache: { }
  rate-limit: { }

# Discord Console
discord-console:
  enabled: true
  channel-id: ""
  allowed-roles: []
```

### Feature Configuration Details

#### Chat Bridge Module

```yaml
features:
  chat-bridge:
    enabled: true  # Master enable

    # Channel Configuration
    chat-channel-id: "1234567890123456789"  # Discord channel ID
    chat-channel: "minecraft-chat"  # Fallback: channel name

    # Direction Controls
    minecraft-to-discord: true  # Send MC chat to Discord
    discord-to-minecraft: true  # Send Discord chat to MC

    # Webhook Settings
    skin-provider: "crafatar"  # crafatar, minotar, mc-heads, visage
    auto-create-webhook: true  # Auto-create webhook (recommended)
    use-webhook: false  # Use custom webhook URL
    webhook-url: ""  # Custom webhook URL

    # Message Formatting
    minecraft-format: "**%player%**: %message%"
    discord-format: "&7[&9Discord&7] &b%user%&7: &f%message%"

    # Enhanced Features
    filter-bot-messages: true  # Ignore Discord bot messages
    show-mentions: true  # Convert @mentions
    play-mention-sound: true  # Play sound for mentions
    convert-emojis: true  # Convert custom emojis
    strip-discord-formatting: false  # Remove **, *, ~~
    show-typing-indicator: true  # Show Discord typing

    # Player Name Formatting (requires LuckPerms)
    show-player-prefix: true
    show-player-suffix: false
    discord-username-format: "%prefix%%player%"
```

**Placeholders**:
- `%player%` - Minecraft player name
- `%prefix%` - Player prefix (LuckPerms)
- `%suffix%` - Player suffix (LuckPerms)
- `%message%` - Chat message
- `%user%` - Discord username

#### Verification Module

```yaml
features:
  verification:
    enabled: true

    # Verification Requirements
    kick-after-minutes: 0  # 0 = disabled, kick unverified after X minutes
    whitelist-mode: false  # Only verified players can join

    # Role Assignment
    verified-role: "Verified"  # Discord role name to give
    verified-group: "verified"  # Minecraft group (LuckPerms)

    # Code Settings
    code-length: 6  # Verification code length
    code-expiry-minutes: 5  # Code expires after X minutes
```

**Commands**:
- `/verify <code>` - Verify account

#### Tickets Module

```yaml
features:
  tickets:
    enabled: true

    # Ticket Creation
    ticket-category: "Support Tickets"  # Discord category name
    auto-close-hours: 48  # Auto-close after X hours (0 = disabled)
    player-close: true  # Allow players to close tickets
    save-transcript: true  # Save transcript on close
    transcript-channel: "ticket-logs"  # Channel to save transcripts

    # Optional: Category ID instead of name
    # ticket-category-id: "1234567890123456789"
```

**Commands**:
- `/support <message>` - Create support ticket

#### Moderation Module

```yaml
features:
  moderation:
    enabled: true

    # Ban Synchronization
    sync-bans: true  # Sync bans between MC and Discord
    log-channel: "mod-logs"  # Moderation log channel

    # Auto-Moderation
    auto-moderate: true  # Enable chat filter
    report-cooldown-minutes: 5  # Cooldown between reports
    require-verification-for-reports: true  # Only verified players can report

    # Chat Filter
    filter-words:
      - "badword1"
      - "badword2"

    # Advanced Filtering (regex)
    # filter-patterns:
    #   - ".*\\b(free money|click here)\\b.*"
```

**Commands**:
- `/report <player> <reason>` - Report a player

#### Database Configuration

**SQLite**:
```yaml
database:
  type: "sqlite"
  sqlite:
    file: "data.db"  # Database file name
```

**MySQL**:
```yaml
database:
  type: "mysql"
  mysql:
    host: "localhost"  # Database host
    port: 3306  # Database port
    database: "xdiscord"  # Database name
    username: "root"  # Database username
    password: "password"  # Database password
    ssl: false  # Use SSL connection
```

**PostgreSQL**:
```yaml
database:
  type: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "xdiscord"
    username: "postgres"
    password: "password"
```

#### Discord Bot Configuration

```yaml
discord:
  bot-token: "YOUR_BOT_TOKEN"  # Get from Discord Developer Portal
  guild-id: "YOUR_GUILD_ID"  # Your Discord server ID

  # Bot Activity/Status
  activity:
    type: "PLAYING"  # PLAYING, WATCHING, LISTENING, COMPETING
    text: "with %players%/%max% players"  # Activity text
    status: "ONLINE"  # ONLINE, DND, IDLE, INVISIBLE

  # Command Settings
  commands:
    use-slash-commands: true  # Enable slash commands
    legacy-prefix: "!"  # Legacy command prefix

  # Channel Configuration
  channels:
    welcome: "1234567890123456789"  # Welcome channel
    logs: "1234567890123456789"  # General logs
    chat: "1234567890123456789"  # Chat bridge channel
    console: "1234567890123456789"  # Console output
    events: "1234567890123456789"  # Player events
    tickets: "1234567890123456789"  # Tickets channel
    moderation: "1234567890123456789"  # Moderation logs
```

**How to Get Channel IDs**:
1. Enable Developer Mode in Discord (Settings → Advanced → Developer Mode)
2. Right-click channel → Copy ID
3. Paste into config.yml

#### Admin Console Configuration

```yaml
discord-console:
  enabled: true
  channel-id: "1234567890123456789"  # Console channel ID
  allowed-roles:
    - "Admin"
    - "Owner"
```

**Console Commands**:
- `!tps` - Check server TPS
- `!list` - List online players
- `!restart` - Restart server (requires restart script)
- `!stop` - Stop server (requires shutdown script)

---

## Message Configuration

### messages.yml Structure

```yaml
# Command Messages
command:
  verify:
    success: "&aYou have successfully verified your account!"
    already-verified: "&eYou are already verified!"
    invalid-code: "&cInvalid or expired verification code!"
    usage: "&cUsage: /verify <code>"

  support:
    created: "&aSupport ticket created! Check Discord for details."
    already-have: "&cYou already have an open support ticket!"
    usage: "&cUsage: /support <message>"

  report:
    success: "&aPlayer reported successfully!"
    cooldown: "&cYou must wait before reporting again!"
    usage: "&cUsage: /report <player> <reason>"

# Chat Messages
chat:
  join: ":arrow_right: **%player%** joined the server!"
  leave: ":arrow_left: **%player%** left the server!"
  death: ":skull: **%player%** %death_message%"
  minecraft-to-discord: "**%player%**: %message%"
  discord-to-minecraft: "&7[&9Discord&7] &b%user%&7: &f%message%"

# Error Messages
error:
  not-verified: "&cYou must verify your Discord account first!"
  no-permission: "&cYou don't have permission to use this command!"
  player-not-found: "&cPlayer not found!"
  unknown-command: "&cUnknown command!"

# Success Messages
success:
  reloaded: "&aConfiguration reloaded successfully!"
  announcement-sent: "&aAnnouncement sent!"
  ticket-closed: "&aTicket closed!"
```

### Customizing Messages

**Color Codes**:
- `&0` - Black
- `&1` - Dark Blue
- `&2` - Dark Green
- `&3` - Dark Aqua
- `&4` - Dark Red
- `&5` - Dark Purple
- `&6` - Gold
- `&7` - Gray
- `&8` - Dark Gray
- `&9` - Blue
- `&a` - Green
- `&b` - Aqua
- `&c` - Red
- `&d` - Light Purple
- `&e` - Yellow
- `&f` - White

**Formatting Codes**:
- `&l` - Bold
- `&o` - Italic
- `&n` - Underline
- `&m` - Strikethrough
- `&k` - Obfuscated

**Placeholders**:
- `%player%` - Player name
- `%target%` - Target player name
- `%reason%` - Reason text
- `%user%` - Discord username
- `%message%` - Message content
- `%server%` - Server name

### Example Custom Configuration

```yaml
# Custom Spanish Messages
language: "es_ES"

command:
  verify:
    success: "&a¡Has verificado tu cuenta exitosamente!"
    invalid-code: "¡Código inválido o expirado!"

chat:
  join: "➜ **%player%** se unió al servidor"
  leave: "➜ **%player%** salió del servidor"

error:
  no-permission: "¡No tienes permiso para usar este comando!"
```

---

## Migration Guide

### Upgrading from 1.0.x to 1.1.0

**Breaking Changes**:
- Database schema updated (new tables added)
- Configuration structure changed
- Module names updated

**Migration Steps**:

1. **Backup Database**:
```bash
cp data.db data.db.backup
```

2. **Update Plugin**:
   - Replace JAR file
   - Start server

3. **New Tables Created Automatically**:
   - `activity_rewards` (for new modules)

4. **Configuration Updated**:
   - Old config backed up as `config.yml.old`
   - New config template loaded
   - Merge custom settings manually

### Migrating from SQLite to MySQL

#### Step 1: Export SQLite Data

```bash
sqlite3 data.db ".dump" > xdiscord_backup.sql
```

#### Step 2: Create MySQL Database

```sql
CREATE DATABASE xdiscord_new;
```

#### Step 3: Import to MySQL

```bash
mysql -u root -p xdiscord_new < xdiscord_backup.sql
```

#### Step 4: Update Configuration

```yaml
database:
  type: "mysql"
  mysql:
    host: "localhost"
    port: 3306
    database: "xdiscord_new"
    username: "xdiscord"
    password: "your_password"
```

#### Step 5: Restart Plugin

Restart server or reload plugin to apply changes.

### Database Schema Migrations

**v1.0.0 → v1.1.0**:

```sql
-- New table: activity_rewards
CREATE TABLE IF NOT EXISTS activity_rewards (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    discord_id VARCHAR(20) NOT NULL,
    reward_type VARCHAR(50),
    reward_data TEXT,
    claimed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    claimed_at TIMESTAMP NULL
);

-- New indexes
CREATE INDEX idx_activity_rewards_discord ON activity_rewards(discord_id);
CREATE INDEX idx_activity_rewards_claimed ON activity_rewards(claimed);
```

---

## Backup and Restore

### Automated Backup Script

Create `backup-xdiscord.sh`:

```bash
#!/bin/bash

# Configuration
PLUGIN_FOLDER="/path/to/plugins/XDiscordUltimate"
BACKUP_FOLDER="/path/to/backups/xdiscord"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup folder
mkdir -p $BACKUP_FOLDER

# Backup database
cp $PLUGIN_FOLDER/data.db $BACKUP_FOLDER/xdiscord_$DATE.db

# Backup configuration
cp $PLUGIN_FOLDER/config.yml $BACKUP_FOLDER/config_$DATE.yml
cp $PLUGIN_FOLDER/messages.yml $BACKUP_FOLDER/messages_$DATE.yml

# Compress backup
tar -czf $BACKUP_FOLDER/xdiscord_backup_$DATE.tar.gz \
    $BACKUP_FOLDER/xdiscord_$DATE.db \
    $BACKUP_FOLDER/config_$DATE.yml \
    $BACKUP_FOLDER/messages_$DATE.yml

# Clean up uncompressed files
rm $BACKUP_FOLDER/xdiscord_$DATE.db
rm $BACKUP_FOLDER/config_$DATE.yml
rm $BACKUP_FOLDER/messages_$DATE.yml

# Keep only last 7 days of backups
find $BACKUP_FOLDER -name "xdiscord_backup_*.tar.gz" -mtime +7 -delete

echo "Backup completed: xdiscord_backup_$DATE.tar.gz"
```

**Schedule with Cron**:
```bash
# Run daily at 2 AM
0 2 * * * /path/to/backup-xdiscord.sh
```

### MySQL Backup

**Manual Backup**:
```bash
mysqldump -u xdiscord -p xdiscord > xdiscord_backup.sql
```

**Automated Backup**:
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
mysqldump -u xdiscord -p'your_password' xdiscord > /backups/xdiscord_$DATE.sql
gzip /backups/xdiscord_$DATE.sql
```

### Restore from Backup

#### SQLite Restore

```bash
# Stop server
# Restore database
cp /backups/xdiscord_backup_20240115_020000.tar.gz /tmp/
cd /tmp
tar -xzf xdiscord_backup_20240115_020000.tar.gz
cp xdiscord_20240115_020000.db /path/to/plugins/XDiscordUltimate/data.db
cp config_20240115_020000.yml /path/to/plugins/XDiscordUltimate/config.yml
cp messages_20240115_020000.yml /path/to/plugins/XDiscordUltimate/messages.yml

# Start server
```

#### MySQL Restore

```bash
# Create database
mysql -u root -p -e "CREATE DATABASE xdiscord;"

# Restore data
gunzip -c /backups/xdiscord_20240115_020000.sql.gz | mysql -u xdiscord -p xdiscord

# Verify
mysql -u xdiscord -p -e "USE xdiscord; SHOW TABLES;"
```

---

## Troubleshooting

### Database Connection Issues

#### Error: "Database not initialized"

**Cause**: Database connection failed during initialization

**Solution**:
1. Check database credentials
2. Verify database server is running
3. Check network connectivity
4. Verify user has permissions

**Debug Steps**:
```java
// Add to DatabaseManager.initialize()
plugin.getLogger().info("Database type: " + type);
plugin.getLogger().info("JDBC URL: " + config.getJdbcUrl());
plugin.getLogger().info("Username: " + config.getUsername());
```

#### Error: "Table doesn't exist"

**Cause**: Plugin hasn't created tables yet or table creation failed

**Solution**:
1. Delete database file (SQLite)
2. Restart plugin (tables will be recreated)
3. Or manually run DDL from logs

**Manual Table Creation**:
```sql
-- Check which tables exist
.tables

-- Create missing tables
CREATE TABLE IF NOT EXISTS verified_users (...);
CREATE TABLE IF NOT EXISTS verification_codes (...);
```

#### Error: "Connection timeout"

**Cause**: Database server too slow or network issues

**Solution**:
1. Increase connection timeout:
```yaml
advanced:
  database-timeout: 60000
```
2. Check database server load
3. Optimize database queries
4. Add indexes to frequently queried columns

### Configuration Issues

#### Error: "Invalid configuration value"

**Cause**: Configuration value is wrong type or format

**Solution**:
1. Enable debug mode:
```yaml
general:
  debug: true
```
2. Check console for specific errors
3. Validate YAML syntax (use online YAML validator)
4. Compare with default config

**Common Issues**:
- Missing quotes around strings with special characters
- Wrong indentation
- Typo in key names
- Wrong data type (e.g., string instead of boolean)

#### Error: "Channel not found"

**Cause**: Discord channel ID is incorrect or bot doesn't have access

**Solution**:
1. Verify channel ID is correct
2. Enable Developer Mode in Discord
3. Right-click channel → Copy ID
4. Check bot has access to channel
5. Verify channel is in configured guild

### Migration Issues

#### Error: "Column doesn't exist" after migration

**Cause**: Database schema not updated

**Solution**:
1. Check if migration scripts ran
2. Manually run schema updates
3. Verify database version

**Check Schema Version**:
```sql
-- Add version table
CREATE TABLE schema_version (
    version VARCHAR(20) PRIMARY KEY,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert current version
INSERT INTO schema_version (version) VALUES ('1.1.0');
```

#### Data Loss During Migration

**Prevention**:
1. Always backup before migration
2. Test migration on development server first
3. Verify data integrity after migration

**Recovery**:
1. Restore from backup
2. Identify what went wrong
3. Retry migration with fixes

### Performance Issues

#### Slow Database Queries

**Diagnosis**:
1. Enable query logging:
```yaml
general:
  debug: true
```
2. Check slow queries in logs
3. Analyze query execution plans

**Solutions**:
1. Add indexes to slow columns
2. Optimize queries
3. Use connection pooling
4. Increase pool size:
```yaml
advanced:
  database-pool-size: 20
```

#### High Memory Usage

**Diagnosis**:
1. Check database connection count
2. Monitor cache size
3. Check for memory leaks

**Solutions**:
1. Close database connections properly
2. Clear caches periodically
3. Reduce pool size
4. Increase server memory

---

This database and configuration reference provides comprehensive information for managing XDiscordUltimate's data layer and configuration. Keep this guide handy for troubleshooting and setup tasks!