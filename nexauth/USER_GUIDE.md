# NexAuth User Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Quick Start](#quick-start)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Database Setup](#database-setup)
6. [Using NexAuth](#using-nexauth)
7. [Commands Reference](#commands-reference)
8. [2FA Setup](#2fa-setup)
9. [Migration from Other Plugins](#migration-from-other-plugins)
10. [Troubleshooting](#troubleshooting)
11. [FAQ](#faq)

---

## Introduction

NexAuth is a modern, feature-rich authentication plugin for Minecraft servers running on Velocity, BungeeCord, or Paper. It provides secure authentication, two-factor authentication (2FA), email verification, and supports migration from popular authentication plugins.

### Key Features

✅ **Secure Authentication**
- Strong password hashing (Argon2ID, BCrypt2A)
- Brute force protection
- Session management

✅ **Two-Factor Authentication (2FA)**
- TOTP support (Google Authenticator, Authy, Microsoft Authenticator)
- QR code generation
- Recovery codes

✅ **Premium Account Support**
- Automatic login for Mojang premium accounts
- UUID migration handling

✅ **Multi-Database Support**
- MySQL
- PostgreSQL
- SQLite

✅ **Migration Support**
- AuthMe
- FastLogin
- LogIt
- And many more

✅ **Email Verification**
- Password reset via email
- Email verification for accounts

✅ **Cross-Platform**
- Velocity
- BungeeCord
- Waterfall
- Paper

---

## Quick Start

### 1. Install NexAuth

Download the appropriate version for your platform from the [releases page](https://github.com/xreatlabs/NexAuth/releases).

### 2. Install NexLimbo (Recommended)

For the best experience on proxy setups, install NexLimbo:
- Download from [NexLimbo releases](https://github.com/Xreatlabs/NexLimbo/releases)
- Install `NexLimbo-Velocity.jar` or `NexLimbo-Bungee.jar`

### 3. Configure Database

Edit `config.conf` with your database credentials.

### 4. Restart Server

Restart your proxy or backend server.

### 5. Configure Servers

In `config.conf`, set your limbo and lobby servers.

### Done!

Players can now use `/register` and `/login` commands.

---

## Installation

### Prerequisites

- Java 21 or newer
- MySQL 5.7+, PostgreSQL 10+, or SQLite 3.0+
- Proxy: Velocity or BungeeCord
- Backend: Paper 1.20.1+

### Step 1: Download NexAuth

1. Visit [NexAuth Releases](https://github.com/xreatlabs/NexAuth/releases)
2. Download the appropriate JAR:
   - `nexauth-velocity.jar` - for Velocity proxy
   - `nexauth-bungee.jar` - for BungeeCord/Waterfall
   - `nexauth-paper.jar` - for Paper backend servers

### Step 2: Install to Your Server

#### For Proxy Setup (Velocity/BungeeCord)

1. Place the JAR in your proxy's `plugins` folder:
   ```
   velocity/
   └── plugins/
       └── NexAuth/
           └── nexauth-velocity.jar  (or bungee version)
   ```

2. **Install NexLimbo** (Highly Recommended):
   - Download NexLimbo from [releases](https://github.com/Xreatlabs/NexLimbo/releases)
   - Place in proxy plugins folder:
     ```
     velocity/
     └── plugins/
         ├── NexAuth/
         │   └── nexauth-velocity.jar
         └── NexLimbo/
             └── NexLimbo-Velocity.jar
     ```
   - Configure NexAuth to use NexLimbo in `config.conf`

3. Start your proxy:
   ```bash
   java -jar velocity.jar
   ```

### Step 3: First Launch

On first launch, NexAuth will:
1. Create default configuration files
2. Display setup instructions
3. Request restart to complete initialization

```log
[INFO] [NexAuth] !! A new configuration was generated, please fill it out, if in doubt, see the wiki !!
```

### Step 4: Edit Configuration

Edit the generated `config.conf` file (location depends on setup):
- **Velocity**: `plugins/NexAuth/config.conf`
- **BungeeCord**: `plugins/NexAuth/config.conf`
- **Paper**: `plugins/NexAuth/config.conf`

---

## Configuration

### Basic Configuration

Edit `config.conf` with your settings. Here's a complete example:

```hocon
# NexAuth Configuration File
# For help, visit: https://github.com/xreatlabs/NexAuth/wiki

# Database Configuration
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "your_password"
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
        }
    }
}

# Crypto Settings
crypto {
    default-provider = "argon2id"
    argon2id {
        iterations = 3
        memory = 65536
        parallelism = 1
    }
}

# Authentication Settings
authentication {
    max-login-attempts = 5
    temp-ban-duration = 300000
    session-timeout = 3600000
    minimum-password-length = 8
}

# Server Routing
servers {
    # Limbo servers (where unauthenticated players are sent)
    limbo = ["limbo"]

    # Lobby servers (where authenticated players are sent)
    lobby {
        root = "lobby"
        lobbies = ["lobby-1", "lobby-2"]
    }

    remember-last-server = true
}

# Two-Factor Authentication
totp {
    enabled = true
    issuer = "YourServerName"
}

# Email Configuration
mail {
    enabled = false
    host = "smtp.gmail.com"
    port = 587
    username = "your-email@gmail.com"
    password = "your-app-password"
    from = "NexAuth <your-email@gmail.com>"
}
```

### Configuration Sections

#### Database Configuration

```hocon
database {
    type = "nexauth-mysql"  # or "nexauth-postgresql" or "nexauth-sqlite"

    mysql {
        host = "localhost"           # Database host
        port = 3306                  # Database port
        database = "nexauth"         # Database name
        username = "user"            # Database username
        password = "pass"            # Database password

        # Connection Pool Settings
        pool {
            maximumPoolSize = 10     # Max connections
            minimumIdle = 5          # Min idle connections
            connectionTimeout = 5000 # Connection timeout (ms)
        }
    }

    # PostgreSQL configuration (if using PostgreSQL)
    postgresql {
        host = "localhost"
        port = 5432
        database = "nexauth"
        username = "user"
        password = "pass"
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
        }
    }

    # SQLite configuration (if using SQLite)
    sqlite {
        file = "data/nexauth.db"     # SQLite file path
    }
}
```

**Supported Database Types**:
- `nexauth-mysql` - MySQL/MariaDB
- `nexauth-postgresql` - PostgreSQL
- `nexauth-sqlite` - SQLite
- `authme-mysql` - Migrate from AuthMe (MySQL)
- `fastlogin-mysql` - Migrate from FastLogin (MySQL)
- And many more migration providers

#### Crypto Settings

```hocon
crypto {
    # Default hashing algorithm
    default-provider = "argon2id"

    # Argon2ID settings (RECOMMENDED)
    argon2id {
        iterations = 3      # Number of iterations (higher = slower, safer)
        memory = 65536      # Memory usage in KB (64 MB recommended)
        parallelism = 1     # Parallelism degree
    }

    # BCrypt2A settings
    bcrypt2a {
        cost = 12           # Cost factor (2^cost iterations)
    }

    # SHA-256 (LEGACY - only for migration)
    sha256 {
        # SHA-256 is weak, use only for migration
        # Use argon2id or bcrypt2a for new installations
    }
}
```

**Security Recommendation**: Always use `argon2id` for new installations. It's the most secure option available.

#### Authentication Settings

```hocon
authentication {
    # Maximum login attempts before temporary ban
    max-login-attempts = 5

    # Temporary ban duration in milliseconds (5 minutes = 300000)
    temp-ban-duration = 300000

    # Session timeout in milliseconds (1 hour = 3600000)
    session-timeout = 3600000

    # Minimum password length
    minimum-password-length = 8

    # Maximum password length
    maximum-password-length = 128

    # Require password confirmation on registration
    require-password-confirmation = true
}
```

#### Server Routing

```hocon
servers {
    # Limbo servers - where unauthenticated players are sent
    # Install NexLimbo for best results!
    limbo = ["limbo"]

    # Lobby server configuration
    lobby {
        # Default lobby server
        root = "lobby"

        # Additional lobby servers (for load balancing)
        lobbies = ["lobby-1", "lobby-2"]
    }

    # Remember and return to last server after authentication
    remember-last-server = true

    # Auto-login for premium players
    auto-login-premium = true

    # Allow premium UUID override
    allow-premium-uuid-override = false
}
```

**Important**: Create your limbo and lobby servers before starting:

- **Limbo Server**: A minimal server where players wait during authentication
- **Lobby Server**: Main server where authenticated players are sent

**Recommended**: Use [NexLimbo](https://github.com/Xreatlabs/NexLimbo) for the limbo server.

#### Two-Factor Authentication

```hocon
totp {
    # Enable/disable 2FA
    enabled = true

    # Issuer name (shown in authenticator app)
    issuer = "YourServerName"

    # Code length (usually 6)
    code-length = 6

    # Time window (30 seconds recommended)
    time-window = 30

    # Recovery codes count
    recovery-codes-count = 10
}
```

#### Email Configuration

```hocon
mail {
    # Enable/disable email functionality
    enabled = false

    # SMTP Server Settings
    host = "smtp.gmail.com"
    port = 587
    username = "your-email@gmail.com"
    password = "your-app-password"
    from = "NexAuth <your-email@gmail.com>"

    # Use TLS
    use-tls = true

    # Use SSL
    use-ssl = false

    # Connection timeout
    connection-timeout = 5000

    # Email templates
    templates {
        verification {
            subject = "Verify your account"
            content = "Your verification code is: {code}"
        }

        password-reset {
            subject = "Password Reset"
            content = "Your password reset code is: {code}"
        }
    }
}
```

**Gmail Setup**:
1. Enable 2-Factor Authentication
2. Generate App Password
3. Use app password in configuration

#### Logging

```hocon
logging {
    # Log level: DEBUG, INFO, WARN, ERROR
    level = "INFO"

    # Log to file
    file = "logs/nexauth.log"

    # Max log file size (MB)
    max-file-size = 10

    # Number of backup files to keep
    max-backup-files = 5

    # Debug mode (enables verbose logging)
    debug = false
}
```

### Advanced Configuration

#### MultiProxy Setup

If running multiple proxies:

```hocon
multiproxy {
    enabled = true
    proxy-id = "proxy-1"

    # Shared secret for proxy communication
    shared-secret = "your-secret-key"

    # Proxy list
    proxies = ["proxy-1", "proxy-2", "proxy-3"]
}
```

#### BungeeGuard Integration

If using BungeeGuard:

```hocon
bungeeguard {
    enabled = true
    allowed-ips = ["your-backend-server-ip"]
}
```

#### Session Management

```hocon
session {
    # Enable/disable session system
    enabled = true

    # Session timeout in milliseconds
    timeout = 3600000

    # Remember login for this long
    remember-duration = 2592000000  # 30 days
}
```

#### IP Validation

```hocon
ip-validation {
    # Allow changing IP during session
    allow-ip-change = false

    # Treat different IPs as new session
    strict-mode = true
}
```

### Configuration Migration

NexAuth can migrate configuration from LibreLogin:

```bash
# On first run, existing LibreLogin config will be automatically migrated
# Check the logs for migration status
```

---

## Database Setup

### MySQL Setup

#### Step 1: Install MySQL

**Ubuntu/Debian**:
```bash
sudo apt update
sudo apt install mysql-server
```

**CentOS/RHEL**:
```bash
sudo yum install mysql-server
```

**Windows**: Download from [mysql.com](https://dev.mysql.com/downloads/installer/)

#### Step 2: Secure MySQL

```bash
sudo mysql_secure_installation
```

#### Step 3: Create Database and User

```sql
-- Connect to MySQL
mysql -u root -p

-- Create database
CREATE DATABASE nexauth;

-- Create user
CREATE USER 'nexauth'@'localhost' IDENTIFIED BY 'your-secure-password';

-- Grant permissions
GRANT ALL PRIVILEGES ON nexauth.* TO 'nexauth'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
```

#### Step 4: Configure NexAuth

In `config.conf`:
```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "your-secure-password"
    }
}
```

#### Step 5: Test Connection

Restart NexAuth and check logs:
```log
[INFO] [NexAuth] Connecting to the database...
[INFO] [NexAuth] Successfully connected to the database
[INFO] [NexAuth] Validating schema
[INFO] [NexAuth] Schema validated
```

### PostgreSQL Setup

#### Step 1: Install PostgreSQL

**Ubuntu/Debian**:
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

**CentOS/RHEL**:
```bash
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### Step 2: Create Database and User

```bash
sudo -u postgres psql

-- In PostgreSQL shell:
CREATE DATABASE nexauth;
CREATE USER nexauth WITH ENCRYPTED PASSWORD 'your-secure-password';
GRANT ALL PRIVILEGES ON DATABASE nexauth TO nexauth;
\q
```

#### Step 3: Configure NexAuth

```hocon
database {
    type = "nexauth-postgresql"
    postgresql {
        host = "localhost"
        port = 5432
        database = "nexauth"
        username = "nexauth"
        password = "your-secure-password"
    }
}
```

### SQLite Setup

SQLite is the simplest option - no separate installation needed.

#### Configuration

```hocon
database {
    type = "nexauth-sqlite"
    sqlite {
        file = "plugins/NexAuth/data/nexauth.db"
    }
}
```

**Advantages**:
- No external dependencies
- Easy backup (just copy the file)
- Perfect for small to medium servers

**Disadvantages**:
- Slower than MySQL/PostgreSQL for high traffic
- No concurrent write access
- Not suitable for clustered setups

---

## Using NexAuth

### For Players

#### First Time (Registration)

1. Join your server
2. You'll be automatically sent to the limbo server
3. Open chat and type:
   ```
   /register <password> <confirm-password>
   ```
   Example: `/register MySecurePass123 MySecurePass123`
4. You'll be automatically logged in and sent to the lobby
5. If 2FA is enabled, follow the prompts to set it up

#### Returning Players

1. Join your server
2. You'll be automatically sent to the limbo server
3. Open chat and type:
   ```
   /login <password>
   ```
   Example: `/login MySecurePass123`
4. You'll be automatically logged in and sent to the lobby

#### Changing Password

```
/changepassword <old-password> <new-password>
```

#### Two-Factor Authentication (2FA)

**Enabling 2FA**:
```
/2fa enable
```
- Scan the QR code with your authenticator app
- Enter the code from your app to confirm
- Save your recovery codes in a safe place!

**Confirming 2FA**:
```
/2fa confirm <code>
```

**Disabling 2FA** (requires current 2FA code):
```
/2fa disable <code>
```

#### Email Features

**Setting Email**:
```
/mail set <your-email@example.com>
```
- Check your email for a verification code
- Enter: `/mail verify <code>`

**Resetting Password via Email**:
1. On login screen, click "Forgot Password" or use command
2. Enter your registered email
3. Check email for reset code
4. Enter: `/mail reset <email> <code> <new-password>`

### Premium Players

Premium players can enable auto-login:

```
/premium enable
```

This links your premium account and enables automatic login.

---

## Commands Reference

### Player Commands

| Command | Description | Usage |
|---------|-------------|-------|
| `/register` | Register new account | `/register <password> <confirm>` |
| `/login` | Login to account | `/login <password>` |
| `/changepassword` | Change password | `/changepassword <old> <new>` |
| `/2fa enable` | Enable 2FA | `/2fa enable` |
| `/2fa confirm` | Confirm 2FA setup | `/2fa confirm <code>` |
| `/2fa disable` | Disable 2FA | `/2fa disable <code>` |
| `/mail set` | Set email address | `/mail set <email>` |
| `/mail verify` | Verify email | `/mail verify <code>` |
| `/mail reset` | Reset password via email | `/mail reset <email> <code> <new-pass>` |
| `/premium enable` | Enable premium auto-login | `/premium enable` |
| `/premium confirm` | Confirm premium linking | `/premium confirm <code>` |
| `/premium disable` | Disable premium linking | `/premium disable` |

### Staff Commands

Admin commands (requires `nexauth.admin` permission):

| Command | Description | Usage |
|---------|-------------|-------|
| `/nexauth reload` | Reload configuration | `/nexauth reload` |
| `/nexauth migrate` | Migrate from other plugins | `/nexauth migrate <from> <to>` |
| `/nexauth backup` | Create database backup | `/nexauth backup [location]` |
| `/nexauth users` | List users | `/nexauth users [page]` |
| `/nexauth user <name>` | View user info | `/nexauth user <name>` |
| `/nexauth force-unauth` | Force logout player | `/nexauth force-unauth <player>` |
| `/nexauth import` | Import users from file | `/nexauth import <file>` |
| `/nexauth export` | Export users to file | `/nexauth export <file>` |

### Permission Nodes

```
# Player permissions
nexauth.register - Allow registration
nexauth.login - Allow login
nexauth.changepassword - Allow password change
nexauth.2fa - Allow 2FA management
nexauth.mail - Allow email management
nexauth.premium - Allow premium features

# Admin permissions
nexauth.admin - Full admin access
nexauth.reload - Reload configuration
nexauth.migrate - Database migration
nexauth.backup - Create backups
nexauth.users.manage - Manage users
nexauth.force - Force actions on players
```

---

## 2FA Setup

### What is 2FA?

Two-Factor Authentication (2FA) adds an extra layer of security to your account. Even if someone steals your password, they can't access your account without your authenticator code.

### Supported Authenticator Apps

- Google Authenticator
- Microsoft Authenticator
- Authy
- 1Password
- LastPass Authenticator
- Any app that supports TOTP (Time-based One-Time Password)

### Setting Up 2FA as Admin

#### Step 1: Enable 2FA in Configuration

Edit `config.conf`:
```hocon
totp {
    enabled = true
    issuer = "YourServerName"
}
```

#### Step 2: Reload Configuration

Use command:
```
/nexauth reload
```

#### Step 3: Players Enable 2FA

Players run:
```
/2fa enable
```

### Setting Up 2FA as Player

#### Step 1: Enable 2FA

Run command:
```
/2fa enable
```

#### Step 2: Scan QR Code

1. A QR code will be displayed (in chat or as image)
2. Open your authenticator app
3. Scan the QR code
4. The app will show a 6-digit code

#### Step 3: Confirm Setup

Enter the code from your app:
```
/2fa confirm 123456
```

#### Step 4: Save Recovery Codes

You'll receive 10 recovery codes. Save them somewhere safe!

Recovery codes format: `XXXX-XXXX`

**Important**: Recovery codes are the only way to recover your account if you lose your authenticator!

### Using 2FA

After enabling 2FA, the login process becomes:

1. Player logs in with password
2. System detects 2FA is enabled
3. Player enters code from authenticator app
4. Upon successful verification, player is authenticated

```
/login <password>
# System: "Enter your 2FA code"
/2fa confirm 123456
# System: "Logged in successfully!"
```

### Recovery Codes

If you lose access to your authenticator app:

1. Use recovery code instead of regular 2FA code:
   ```
   /2fa confirm 1234-5678
   ```
2. Disable 2FA and re-enable with new device

**Important**:
- Each recovery code can be used only once
- Generate new recovery codes after use
- Keep them secure and accessible

### Disabling 2FA

If you want to disable 2FA:

```
/2fa disable <current-code>
```

You can also disable via staff command:
```
/nexauth 2fa disable <player>
```

### Troubleshooting 2FA

#### Issue: "Invalid code" error

**Possible causes**:
- Time sync issue on server
- Wrong code entered
- Code expired (codes change every 30 seconds)

**Solution**:
1. Check your device time is correct
2. Wait for new code
3. Try again

#### Issue: Lost authenticator device

**Solution**:
1. Use recovery code: `/2fa confirm 1234-5678`
2. Disable 2FA: `/2fa disable 1234-5678`
3. Re-enable with new device: `/2fa enable`

#### Issue: Can't see QR code

**Solution**:
- Staff can regenerate QR code: `/nexauth 2fa qr <player>`
- Use manual entry code (shown with QR code)
- Ask staff to send QR code via other means

---

## Migration from Other Plugins

NexAuth can migrate user data from many popular authentication plugins.

### Supported Plugins for Migration

| Plugin | Database Type | Identifier |
|--------|---------------|------------|
| AuthMe | MySQL, PostgreSQL, SQLite | `authme-mysql`, `authme-postgresql`, `authme-sqlite` |
| FastLogin | MySQL, SQLite | `fastlogin-mysql`, `fastlogin-sqlite` |
| LogIt | MySQL | `logit-mysql` |
| Aegis | MySQL | `aegis-mysql` |
| DBA | MySQL | `dba-mysql` |
| JPremium | MySQL | `jpremium-mysql` |
| NLogin | MySQL, SQLite | `nlogin-mysql`, `nlogin-sqlite` |
| LimboAuth | MySQL | `limboauth-mysql` |
| Authy | MySQL, SQLite | `authy-mysql`, `authy-sqlite` |
| LoginSecurity | MySQL, SQLite | `loginsecurity-mysql`, `loginsecurity-sqlite` |
| UniqueCodeAuth | MySQL | `uniquecodeauth-mysql` |
| CrazyLogin | MySQL | `crazylogin-mysql` (planned) |

### Migration Process

#### Step 1: Backup Old Plugin Data

**Important**: Always backup your data before migration!

**MySQL/PostgreSQL**:
```bash
# Create SQL dump
mysqldump -u username -p old_database > backup.sql
# OR for PostgreSQL
pg_dump -U username -d old_database > backup.sql
```

**SQLite**:
```bash
# Copy the database file
cp authme.db authme.db.backup
```

#### Step 2: Configure Migration

Edit `config.conf`:
```hocon
# Set up destination database
database {
    type = "nexauth-mysql"  # Your new NexAuth database
    mysql {
        host = "localhost"
        # ... your database settings
    }
}

# Configure migration settings
migration {
    # Enable migration
    enabled = true

    # Source database type
    from = "authme-mysql"  # Change to your plugin

    # Old database connection
    old-database {
        mysql {
            host = "localhost"
            port = 3306
            database = "authme"
            username = "authme_user"
            password = "authme_pass"
        }
    }

    # Hash conversion
    convert-hashes = true  # Convert old hashes to new algorithm

    # Default password for users without password (legacy)
    default-password = "changeme123"

    # Skip premium users during migration
    skip-premium = false
}
```

#### Step 3: Run Migration

**Option A: Automatic (on startup)**

NexAuth will automatically run migration on first startup after configuration:
```log
[INFO] [NexAuth] Starting migration from AuthMe...
[INFO] [NexAuth] Reading data...
[INFO] [NexAuth] Data read, inserting into database...
[INFO] [NexAuth] Migration completed successfully!
```

**Option B: Manual Command**

```
/nexauth migrate authme nexauth-mysql
```

#### Step 4: Verify Migration

Check logs for migration status:
```log
[INFO] [NexAuth] Migration Statistics:
[INFO] [NexAuth] - Users migrated: 1250
[INFO] [NexAuth] - Users skipped: 5
[INFO] [NexAuth] - Errors: 0
```

Verify a few users manually:
```
/nexauth user <username>
```

#### Step 5: Test

1. Test login with old passwords
2. Verify user data (email, last seen, etc.)
3. Check that premium users can auto-login

#### Step 6: Remove Old Plugin

After successful migration:

1. Stop your server/proxy
2. Remove old plugin JAR file
3. Clean old configuration files (optional)
4. Restart server

### Migration Examples

#### Migrate from AuthMe (MySQL)

```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "password"
    }
}

migration {
    enabled = true
    from = "authme-mysql"

    old-database {
        mysql {
            host = "localhost"
            port = 3306
            database = "authme"
            username = "authme_user"
            password = "authme_pass"
        }
    }

    # AuthMe table name (default: authme)
    old-database-table = "authme"

    convert-hashes = true
}
```

#### Migrate from FastLogin (SQLite)

```hocon
database {
    type = "nexauth-postgresql"
    postgresql {
        host = "localhost"
        # ... settings
    }
}

migration {
    enabled = true
    from = "fastlogin-sqlite"

    old-database {
        sqlite {
            file = "/path/to/fastlogin.db"
        }
    }

    convert-hashes = true
}
```

### Troubleshooting Migration

#### Issue: Migration fails with "table not found"

**Solution**: Check that the old database connection settings are correct and the table name is accurate.

#### Issue: Users can't login after migration

**Possible causes**:
1. Password hashes weren't converted
2. Wrong hash algorithm specified
3. UUID mismatch

**Solution**:
1. Enable `convert-hashes` in config
2. Check migration logs for errors
3. Manually verify password hashing

#### Issue: Premium users not auto-logging in

**Solution**:
```hocon
migration {
    skip-premium = false  # Ensure premium users are migrated
}
```

#### Issue: Some users missing after migration

**Solution**:
1. Check migration logs for skipped users
2. Verify old database has no corruption
3. Re-run migration with `--force` flag

#### Custom Migration

For plugins not directly supported:

1. Export user data from old plugin (CSV/JSON)
2. Use `/nexauth import <file>` command
3. Custom format: `uuid,username,password,email,timestamp`

---

## Troubleshooting

### Common Issues

#### Issue: Players stuck in limbo

**Symptoms**: Players join but stay in limbo, can't authenticate

**Causes & Solutions**:

1. **No limbo server configured**
   ```
   servers {
       limbo = ["limbo"]  # Make sure this exists
   }
   ```
   Create a server named "limbo" or update config to match your server name.

2. **Backend servers directly accessible**
   - Players can bypass proxy
   - Configure firewall to block direct access
   - Use BungeeGuard

3. **Event conflicts**
   - Other plugins interfering
   - Check for errors in logs
   - Temporarily disable other plugins to test

#### Issue: "Database connection failed"

**Symptoms**: Plugin shows database errors, crashes on startup

**Causes & Solutions**:

1. **Wrong credentials**
   ```
   mysql {
       username = "nexauth"  # Verify username
       password = "password"  # Verify password
   }
   ```
   Test credentials manually: `mysql -u nexauth -p nexauth`

2. **Database not running**
   ```bash
   sudo systemctl status mysql
   sudo systemctl start mysql
   ```

3. **Wrong host/port**
   ```
   host = "localhost"  # Or remote IP
   port = 3306         # Verify port
   ```

4. **Network/firewall blocked**
   ```bash
   # Test connection
   telnet mysql-host 3306
   ```

5. **Database doesn't exist**
   ```sql
   CREATE DATABASE nexauth;
   ```

#### Issue: "Password validation failed"

**Symptoms**: Players can't register, says password too weak

**Solutions**:

1. **Check minimum length**
   ```hocon
   authentication {
       minimum-password-length = 8  # Adjust if needed
   }
   ```

2. **Check forbidden passwords**
   - Update `forbidden-passwords.txt`
   - Remove too restrictive entries

3. **Check password complexity rules**
   ```hocon
   authentication {
       require-uppercase = false
       require-numbers = false
       require-special = false
   }
   ```

#### Issue: "Player already registered" but account doesn't exist

**Symptoms**: New player can't register, says already exists

**Cause**: UUID conflict from premium migration

**Solution**:
```bash
# Remove stale user
/nexauth user <name>
/nexauth delete <name>
```

Or manually in database:
```sql
DELETE FROM users WHERE last_nickname = 'PlayerName';
```

#### Issue: 2FA codes not working

**Symptoms**: "Invalid code" even with correct code

**Solutions**:

1. **Server time sync**
   ```bash
   # Check server time
   date

   # Sync time (Linux)
   sudo ntpdate -s time.nist.gov

   # Or install NTP
   sudo apt install ntp
   ```

2. **Check TOTP configuration**
   ```hocon
   totp {
       time-window = 30  # Standard window
       code-length = 6   # Standard length
   }
   ```

3. **Generate new 2FA**
   ```
   /nexauth 2fa regenerate <player>
   ```

#### Issue: Email not sending

**Symptoms**: Email verification doesn't work

**Solutions**:

1. **Check email config**
   ```hocon
   mail {
       enabled = true
       host = "smtp.gmail.com"
       username = "your-email@gmail.com"
       password = "app-password"  # NOT regular password
   }
   ```

2. **For Gmail**: Use App Password
   - Enable 2FA on Google account
   - Generate App Password
   - Use app password in config

3. **Check firewall**
   ```bash
   # Allow SMTP port
   sudo ufw allow 587/tcp  # or 465 for SSL
   ```

4. **Test SMTP manually**
   ```bash
   telnet smtp.gmail.com 587
   ```

#### Issue: "You are running an outdated version"

**Solution**:

1. Check releases: https://github.com/xreatlabs/NexAuth/releases
2. Download latest version
3. Replace JAR file
4. Restart server

**To disable update check**:
```hocon
updates {
    check-for-updates = false
}
```

#### Issue: High CPU/Memory usage

**Symptoms**: Server lag, high resource usage

**Solutions**:

1. **Adjust Argon2ID settings**
   ```hocon
   crypto {
       argon2id {
           memory = 32768  # Reduce from 65536
           iterations = 2  # Reduce from 3
       }
   }
   ```

2. **Database connection pool**
   ```hocon
   mysql {
       pool {
           maximumPoolSize = 5  # Reduce from 10
       }
   }
   ```

3. **Enable caching**
   ```hocon
   cache {
       enabled = true
       size = 1000
   }
   ```

#### Issue: Commands not working

**Symptoms**: Commands return "Unknown command"

**Solutions**:

1. **Check permissions**
   ```
   # Give yourself admin
   /lp user <yourname> permission set nexauth.admin true
   ```

2. **Reload commands**
   ```
   /nexauth reload
   ```

3. **Check command framework**
   - Ensure ACF is loaded
   - Check for conflicts with other plugins

4. **View available commands**
   ```
   /help  # or /nexauth help
   ```

### Debug Mode

Enable debug logging for detailed diagnostics:

```hocon
logging {
    level = "DEBUG"
    debug = true
}
```

**Restart server to apply**

Debug logs will show:
- Database queries
- Event firing
- Authentication flow
- Error details

### Getting Help

If you're still stuck:

1. **Check the wiki**: https://github.com/xreatlabs/NexAuth/wiki
2. **Check GitHub Issues**: https://github.com/xreatlabs/NexAuth/issues
3. **Join Discord**: https://discord.gg/m5ptM8X2Du
4. **Provide information**:
   - Server platform (Velocity/Bungee/Paper)
   - NexAuth version
   - Configuration (hide sensitive data)
   - Full error logs
   - Steps to reproduce

### Log Analysis

#### Normal Startup
```
[INFO] [NexAuth] Enabling NexAuth v0.0.1-beta3
[INFO] [NexAuth] Successfully connected to the database
[INFO] [NexAuth] Schema validated
[INFO] [NexAuth] Loaded 0 forbidden passwords
[INFO] [NexAuth] Configuration loaded
[INFO] [NexAuth] NexAuth enabled successfully
```

#### Migration
```
[INFO] [NexAuth] Starting migration from AuthMe
[INFO] [NexAuth] Reading data...
[INFO] [NexAuth] Data read, inserting into database...
[INFO] [NexAuth] Migration completed successfully! (1250 users migrated)
```

#### Authentication Success
```
[INFO] [NexAuth] Player Steve authenticated (LOGIN)
[INFO] [NexAuth] Player Steve redirected to lobby
```

#### Warning Signs

⚠️ `!! THIS IS MOST LIKELY NOT AN ERROR CAUSED BY NEXAUTH !!`
- Usually indicates database/network issues
- Check your configuration

⚠️ `!! YOU ARE RUNNING A DEVELOPMENT BUILD !!`
- Running an unstable version
- Update to release version

⚠️ `!! YOU ARE RUNNING AN OUTDATED VERSION OF NEXAUTH !!`
- Update to latest version
- Security and bug fixes available

⚠️ `!! PLEASE UPDATE TO THE LATEST VERSION !!`
- See above

---

## FAQ

### General

**Q: What is NexAuth?**
A: NexAuth is a modern authentication plugin for Minecraft servers that provides secure login, 2FA, email verification, and more.

**Q: What platforms does NexAuth support?**
A: Velocity, BungeeCord, Waterfall, and Paper (backend servers).

**Q: Is NexAuth free?**
A: Yes! NexAuth is open-source and free to use.

**Q: Can I use NexAuth on a stand-alone server (without proxy)?**
A: Yes, install the Paper version directly on your server.

### Installation

**Q: Do I need to install NexAuth on all servers?**
A: For proxy setups, install only on the proxy. For stand-alone servers, install on the server itself.

**Q: What database should I use?**
A: MySQL or PostgreSQL for production servers. SQLite for small servers or testing.

**Q: Can I change database type later?**
A: Yes, use the migration feature to migrate between database types.

**Q: What is NexLimbo and do I need it?**
A: NexLimbo is a lightweight limbo server optimized for NexAuth. Recommended for proxy setups but not required.

### Security

**Q: Which crypto algorithm should I use?**
A: Use Argon2ID for new installations. It's the most secure option.

**Q: Is 2FA required?**
A: No, but highly recommended for security. Administrators can enforce it.

**Q: Are passwords stored securely?**
A: Yes, passwords are hashed using industry-standard algorithms (Argon2ID/BCrypt2A).

**Q: Can passwords be cracked?**
A: Strong passwords with Argon2ID are extremely resistant to cracking. Use long, complex passwords.

**Q: How do I recover a lost password?**
A: If email is configured, use the password reset feature. Otherwise, administrators can help.

### Migration

**Q: Can I migrate from AuthMe?**
A: Yes, NexAuth supports migration from AuthMe and many other plugins.

**Q: Will my users lose their data?**
A: No, migration preserves all user data including passwords, emails, and statistics.

**Q: Do I need to reinstall after migration?**
A: No, just install NexAuth and configure migration. Old plugin can be removed afterward.

**Q: Can I test migration first?**
A: Yes, you can backup your database and test migration on a copy.

### Features

**Q: Does NexAuth support premium accounts?**
A: Yes, automatic login for Mojang premium accounts.

**Q: Can users change their password?**
A: Yes, users can change their password with `/changepassword` command.

**Q: Does NexAuth support multiple languages?**
A: Yes, messages are configurable and can be customized.

**Q: Can I disable certain features?**
A: Yes, all features can be enabled/disabled in configuration.

**Q: Does NexAuth work with Floodgate (Bedrock)?**
A: Yes, NexAuth has built-in Floodgate support.

**Q: Can I use my own database schema?**
A: NexAuth uses its own schema but migration tools can adapt from other plugins.

### Performance

**Q: Will NexAuth lag my server?**
A: No, NexAuth is optimized for performance. Heavy operations run asynchronously.

**Q: How many users can NexAuth handle?**
A: NexAuth can handle thousands of users. For very large servers, use MySQL/PostgreSQL.

**Q: Does NexAuth cache user data?**
A: Yes, user data is cached for improved performance.

### Support

**Q: Where can I get help?**
A: Check the wiki, GitHub issues, or join our Discord server.

**Q: Can I request features?**
A: Yes, open a feature request on GitHub or discuss in Discord.

**Q: Do you offer commercial support?**
A: Community support is free. For commercial options, contact us on Discord.

**Q: How do I report a bug?**
A: Open an issue on GitHub with details: version, configuration, logs, and steps to reproduce.

### Technical

**Q: What Java version is required?**
A: Java 21 or newer.

**Q: Can I use NexAuth with other authentication plugins?**
A: No, use only one authentication plugin at a time.

**Q: Does NexAuth use a config database (H2/HikariCP)?**
A: No, you need to configure MySQL, PostgreSQL, or SQLite.

**Q: Can I use NexAuth with pmmp/PocketMine?**
A: No, NexAuth is for Java Edition servers only.

**Q: Is the source code available?**
A: Yes, NexAuth is open-source on GitHub.

### Commands

**Q: Players can't use commands in limbo**
A: This is intentional. Commands are disabled in limbo for security. Players can still use `/login` and `/register`.

**Q: How do I give myself admin permissions?**
A: Configure permissions in your proxy's permission system (LuckPerms, etc.).

**Q: Can I disable `/register` for existing users?**
A: Yes, set `allow-registration = false` in configuration.

### Configuration

**Q: Where is the configuration file located?**
A: `plugins/NexAuth/config.conf` (or proxy equivalent).

**Q: Can I use environment variables in config?**
A: Yes, use `${ENV_VAR_NAME}` syntax.

**Q: How do I reload configuration?**
A: Use `/nexauth reload` command.

**Q: Can I backup configuration?**
A: Yes, simply copy the `config.conf` file.

---

## Quick Reference

### Essential Configuration

```hocon
# Minimum viable configuration
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        database = "nexauth"
        username = "user"
        password = "pass"
    }
}

servers {
    limbo = ["limbo"]
    lobby {
        root = "lobby"
    }
}

crypto {
    default-provider = "argon2id"
}
```

### Essential Commands

**Player**:
- `/register <pass> <pass>` - Register
- `/login <pass>` - Login
- `/2fa enable` - Enable 2FA
- `/mail set <email>` - Set email

**Admin**:
- `/nexauth reload` - Reload config
- `/nexauth migrate <from> <to>` - Migrate
- `/nexauth users` - List users
- `/nexauth user <name>` - View user

### Default Ports

- MySQL: 3306
- PostgreSQL: 5432
- SMTP: 587 (TLS) or 465 (SSL)
- Minecraft: 25565

### File Locations

- **Config**: `plugins/NexAuth/config.conf`
- **Messages**: `plugins/NexAuth/messages.conf`
- **Logs**: `plugins/NexAuth/logs/`
- **Database**: Configured in `config.conf`
- **Forbidden Passwords**: `plugins/NexAuth/forbidden-passwords.txt`

### Common File Paths

**Linux**:
```
/etc/mysql/mysql.conf.d/mysqld.cnf  # MySQL config
/etc/postgresql/13/main/postgresql.conf  # PostgreSQL config
```

**Windows**:
```
C:\ProgramData\MySQL\MySQL Server 8.0\my.ini  # MySQL
```

---

## Support Resources

- **Wiki**: https://github.com/xreatlabs/NexAuth/wiki
- **GitHub**: https://github.com/xreatlabs/NexAuth
- **Discord**: https://discord.gg/m5ptM8X2Du
- **Spigot**: https://www.spigotmc.org/resources/nexauth.101040/
- **Issues**: https://github.com/xreatlabs/NexAuth/issues
- **Releases**: https://github.com/xreatlabs/NexAuth/releases

---

## Glossary

**2FA/TOTP**: Two-Factor Authentication / Time-based One-Time Password - Extra security layer using authenticator apps

**Argon2ID**: Modern password hashing algorithm, most secure option

**BCrypt2A**: Password hashing algorithm, proven track record

**HOCON**: Human-Optimized Config Object Notation - Configuration format used by NexAuth

**Limbo Server**: Server where unauthenticated players are sent

**Lobby Server**: Server where authenticated players are sent

**Migration**: Process of moving data from old plugin to NexAuth

**NexLimbo**: Optimized limbo server for NexAuth

**Premium Player**: Player with paid Mojang account

**Session**: Period during which player stays logged in

**TOTP**: Time-based One-Time Password - Code from authenticator app

---

**Version**: 0.0.1-beta3
**Last Updated**: 2025-10-30
**Authors**: Xreatlabs Team
**License**: Mozilla Public License 2.0

For the latest version of this guide, visit: https://github.com/xreatlabs/NexAuth
