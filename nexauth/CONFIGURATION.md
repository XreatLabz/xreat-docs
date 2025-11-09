# NexAuth Configuration Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Configuration File Location](#configuration-file-location)
3. [Configuration Format](#configuration-format)
4. [Complete Configuration Reference](#complete-configuration-reference)
5. [Database Configuration](#database-configuration)
6. [Authentication Configuration](#authentication-configuration)
7. [Server Configuration](#server-configuration)
8. [Crypto Configuration](#crypto-configuration)
9. [2FA Configuration](#2fa-configuration)
10. [Email Configuration](#email-configuration)
11. [Logging Configuration](#logging-configuration)
12. [Advanced Configuration](#advanced-configuration)
13. [Environment Variables](#environment-variables)
14. [Configuration Migration](#configuration-migration)
15. [Validation Rules](#validation-rules)
16. [Troubleshooting](#troubleshooting)

---

## Introduction

This document provides a comprehensive reference for all configuration options available in NexAuth. Use this as a reference when setting up or customizing your NexAuth installation.

### Configuration Types

NexAuth uses HOCON (Human-Optimized Config Object Notation) format for configuration files. HOCON is:
- Human-readable
- Supports comments
- Allows comments on same line
- Supports includes and substitutions

### Key Features

- **Type-safe**: Values are validated against expected types
- **Default values**: Sensible defaults for all options
- **Validation**: Configuration is validated on load
- **Reloadable**: Can be reloaded without restart (`/nexauth reload`)

---

## Configuration File Location

The configuration file location depends on your setup:

### Velocity Proxy
```
plugins/NexAuth/config.conf
```

### BungeeCord/Waterfall Proxy
```
plugins/NexAuth/config.conf
```

### Paper Backend Server
```
plugins/NexAuth/config.conf
```

### Custom Location

You can specify a custom location using system property:
```bash
java -Dnexauth.config=/path/to/config.conf -jar velocity.jar
```

---

## Configuration Format

### Basic Syntax

```hocon
# Comments start with #

# Key-value pairs
key = value

# Nested objects
database {
    host = "localhost"
    port = 3306
}

# Arrays
servers = ["lobby-1", "lobby-2"]

# Strings
name = "value"

# Numbers
port = 3306

# Booleans
enabled = true

# Placeholders
username = "${DATABASE_USERNAME}"  # From environment variable
```

### String Values

```hocon
# Regular string
name = "Steve"

# String with special characters
motd = "Welcome to \"My Server\"!"

# Multiline string
description = """
This is a
multiline string
"""

# Raw string (no escaping)
regex = '''[a-zA-Z0-9_]+'''
```

### Arrays and Lists

```hocon
# Simple array
servers = ["lobby-1", "lobby-2", "lobby-3"]

# Array of objects
servers = [
    {
        name = "lobby-1"
        ip = "127.0.0.1"
        port = 25565
    }
]
```

### Includes

```hocon
# Include another file
include "database.conf"
include "servers.conf"
```

### Substitutions

```hocon
# Use value from another key
default_port = 3306
mysql_port = ${default_port}

# Use environment variable
db_password = "${DB_PASSWORD}"

# Use system property
config_path = "${?CUSTOM_CONFIG_PATH}"  # Optional, won't error if not set
```

---

## Complete Configuration Reference

Here's the complete structure of the NexAuth configuration file:

```hocon
# NexAuth Configuration
# Full reference: https://github.com/xreatlabs/NexAuth/wiki

# ==========================================
# DATABASE CONFIGURATION
# ==========================================
database {
    # Database provider type
    # Options: nexauth-mysql, nexauth-postgresql, nexauth-sqlite
    # Migration providers: authme-mysql, fastlogin-mysql, etc.
    type = "nexauth-mysql"

    # MySQL configuration
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "your-password"

        # Connection pool settings
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
            idleTimeout = 600000
            maxLifetime = 1800000
            leakDetectionThreshold = 60000
        }

        # SSL/TLS settings
        ssl {
            enabled = false
            # ssl-cert, ssl-key, ssl-ca paths if needed
        }

        # Additional connection properties
        properties {
            useSSL = false
            allowPublicKeyRetrieval = true
        }
    }

    # PostgreSQL configuration
    postgresql {
        host = "localhost"
        port = 5432
        database = "nexauth"
        username = "nexauth"
        password = "your-password"

        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
            idleTimeout = 600000
            maxLifetime = 1800000
        }

        ssl {
            enabled = false
            mode = "prefer"  # disable, require, verify-ca, verify-full
            cert = "/path/to/cert"
        }

        properties {
            ssl = false
        }
    }

    # SQLite configuration
    sqlite {
        file = "data/nexauth.db"
        # WAL mode for better concurrency
        wal = true
        # Timeout for database locks (ms)
        busyTimeout = 30000
    }
}

# ==========================================
# CRYPTO CONFIGURATION
# ==========================================
crypto {
    # Default hashing algorithm
    # Options: argon2id, bcrypt2a, sha256, sha512, logit-sha256
    default-provider = "argon2id"

    # Argon2ID settings (RECOMMENDED)
    argon2id {
        # Number of iterations (higher = more secure, slower)
        # Recommended: 3-4 for modern servers
        iterations = 3

        # Memory usage in KB (higher = more secure, more memory)
        # Recommended: 65536 (64 MB) for servers with 8GB+ RAM
        memory = 65536

        # Parallelism degree (CPU threads to use)
        # Recommended: 1-2
        parallelism = 1

        # Hash length (bytes)
        hash-length = 32
    }

    # BCrypt2A settings
    bcrypt2a {
        # Cost factor (2^cost iterations)
        # Recommended: 12 for modern servers
        # Higher = more secure, slower
        cost = 12

        # Minor version (2a, 2b, 2y)
        variant = "2a"
    }

    # SHA-256 settings (LEGACY - for migration only)
    sha256 {
        # Iterations (for salted SHA)
        iterations = 1

        # WARNING: SHA-256 is not secure for passwords!
        # Only use for migration from old plugins
        # Use argon2id or bcrypt2a for new installations
    }

    # SHA-512 settings (LEGACY - for migration only)
    sha512 {
        iterations = 1

        # WARNING: SHA-512 is not secure for passwords!
        # Only use for migration from old plugins
    }

    # LOGIT-SHA-256 settings
    logit-sha256 {
        iterations = 10000

        # Custom algorithm used by LogIt plugin
        # Use only for migration
    }
}

# ==========================================
# AUTHENTICATION SETTINGS
# ==========================================
authentication {
    # Maximum login attempts before temporary ban
    max-login-attempts = 5

    # Temporary ban duration in milliseconds
    # Default: 300000 (5 minutes)
    temp-ban-duration = 300000

    # Session timeout in milliseconds
    # After this time, player must re-authenticate
    # Default: 3600000 (1 hour)
    session-timeout = 3600000

    # Minimum password length
    minimum-password-length = 8

    # Maximum password length
    maximum-password-length = 128

    # Require password confirmation on registration
    require-password-confirmation = true

    # Allow registration
    allow-registration = true

    # Auto-login premium players
    auto-login-premium = true

    # Allow changing username after registration
    allow-username-change = false

    # Password validation rules
    password-rules {
        # Require uppercase letters
        require-uppercase = false

        # Require lowercase letters
        require-lowercase = false

        # Require numbers
        require-numbers = false

        # Require special characters
        require-special = false

        # Maximum repeated characters
        max-repeated-chars = 3
    }

    # Name validation
    name-validation {
        # Minimum name length
        min-length = 3

        # Maximum name length
        max-length = 16

        # Regex pattern for valid names
        pattern = "[a-zA-Z0-9_]+"

        # Check case sensitivity
        case-sensitive = false

        # Allow digits
        allow-digits = true
    }
}

# ==========================================
# SERVER CONFIGURATION
# ==========================================
servers {
    # Limbo servers (for unauthenticated players)
    # Create these servers before starting!
    limbo = ["limbo"]

    # Lobby server configuration
    lobby {
        # Default lobby server
        root = "lobby"

        # Additional lobby servers (for load balancing)
        # Format: "server-name" = "server-display-name"
        lobbies = ["lobby-1", "lobby-2"]
    }

    # Remember and return to last server after authentication
    remember-last-server = true

    # Default server for new players (if remember-last-server is false)
    default-server = "lobby"

    # Server selection strategy
    # Options: round-robin, least-players, random, hash
    selection-strategy = "round-robin"

    # Backend server protection
    protection {
        # Enable BungeeGuard
        bungeeguard-enabled = false

        # Allowed proxy IPs (for backend servers)
        allowed-proxy-ips = []

        # Verify player authenticity
        verify-players = true
    }

    # MultiProxy configuration
    multiproxy {
        # Enable MultiProxy support
        enabled = false

        # This proxy's ID
        proxy-id = "proxy-1"

        # Shared secret for proxy communication
        shared-secret = "your-secret"

        # List of all proxies
        proxies = ["proxy-1", "proxy-2"]
    }
}

# ==========================================
# TWO-FACTOR AUTHENTICATION (2FA)
# ==========================================
totp {
    # Enable/disable 2FA
    enabled = false

    # Issuer name (shown in authenticator app)
    # Recommended: Your server name
    issuer = "NexAuth"

    # Code length (6 is standard)
    code-length = 6

    # Time window in seconds (30 is standard)
    # Allows for slight time drift
    time-window = 30

    # Number of recovery codes
    recovery-codes-count = 10

    # Allow users to disable 2FA
    allow-disable = true

    # Enforce 2FA for all users (dangerous!)
    enforce = false

    # QR code generation
    qr {
        # Enable QR code display
        enabled = true

        # QR code size in pixels
        size = 300

        # Image format: PNG, JPEG
        format = "PNG"

        # QR code error correction level
        # Options: L, M, Q, H
        error-correction = "M"
    }
}

# ==========================================
# EMAIL CONFIGURATION
# ==========================================
mail {
    # Enable/disable email functionality
    enabled = false

    # SMTP Server Settings
    host = "smtp.gmail.com"
    port = 587
    username = "your-email@gmail.com"
    password = "your-app-password"

    # From address
    from = "NexAuth <your-email@gmail.com>"

    # Use TLS (recommended for port 587)
    use-tls = true

    # Use SSL (for port 465)
    use-ssl = false

    # Connection timeout in milliseconds
    connection-timeout = 5000

    # Read timeout in milliseconds
    read-timeout = 10000

    # Email templates
    templates {
        # Verification email template
        verification {
            subject = "Verify your NexAuth account"
            content = """
            <html>
            <body>
                <h2>Verify Your Account</h2>
                <p>Hello,</p>
                <p>Your verification code is: <strong>{code}</strong></p>
                <p>Enter this code in the game to verify your email.</p>
                <p>Best regards,<br>NexAuth</p>
            </body>
            </html>
            """
            format = "HTML"  # Options: TEXT, HTML
        }

        # Password reset email template
        password-reset {
            subject = "NexAuth Password Reset"
            content = """
            <html>
            <body>
                <h2>Password Reset</h2>
                <p>Hello,</p>
                <p>Your password reset code is: <strong>{code}</strong></p>
                <p>Use this code to reset your password in the game.</p>
                <p>If you did not request this, please ignore this email.</p>
            </body>
            </html>
            """
            format = "HTML"
        }

        # Custom variables available:
        # {code} - The verification/reset code
        # {username} - The player's username
        # {server} - Server name
        # {ip} - Player's IP address
    }

    # Rate limiting
    rate-limiting {
        # Max emails per hour per IP
        max-per-hour = 10

        # Cooldown between emails in seconds
        cooldown = 60
    }
}

# ==========================================
# LOGGING CONFIGURATION
# ==========================================
logging {
    # Log level
    # Options: TRACE, DEBUG, INFO, WARN, ERROR, OFF
    level = "INFO"

    # Log to file
    file = "logs/nexauth.log"

    # Max log file size in MB
    max-file-size = 10

    # Number of backup files to keep
    max-backup-files = 5

    # Enable/disable console logging
    console = true

    # Debug mode (enables verbose logging)
    debug = false

    # Log patterns
    patterns {
        # Console log pattern
        console = "[{yyyy-MM-dd HH:mm:ss}] [{LEVEL}] [{COMPONENT}] {MESSAGE}{EXCEPTION}"

        # File log pattern
        file = "{yyyy-MM-dd HH:mm:ss.SSS} [{LEVEL}] [{COMPONENT}] {MESSAGE}{EXCEPTION}"
    }

    # Component names for logging
    components {
        database = "Database"
        auth = "Auth"
        events = "Events"
        config = "Config"
    }

    # Colored console output
    console-colors = true
}

# ==========================================
# SESSION MANAGEMENT
# ==========================================
session {
    # Enable/disable session system
    enabled = true

    # Session timeout in milliseconds
    # After this time, player must re-authenticate
    timeout = 3600000

    # Remember login duration in milliseconds
    # Max time to remember login (30 days = 2592000000)
    remember-duration = 2592000000

    # IP validation
    ip-validation {
        # Allow changing IP during session
        allow-ip-change = false

        # Treat different IPs as new session
        strict-mode = true
    }

    # Session storage
    storage {
        # Session backend: database, file
        backend = "database"

        # File location (if using file backend)
        file = "data/sessions.json"

        # Clean expired sessions
        cleanup-enabled = true

        # Cleanup interval in minutes
        cleanup-interval = 60
    }
}

# ==========================================
# UPDATES
# ==========================================
updates {
    # Check for updates on startup
    check-for-updates = true

    # Update check interval in hours
    check-interval = 24

    # Notify about dev builds
    notify-dev-builds = false

    # Update channel
    # Options: release, beta, dev
    channel = "release"
}

# ==========================================
# PERFORMANCE
# ==========================================
performance {
    # Async operation settings
    async {
        # Thread pool size
        # Default: CPU cores
        pool-size = 4

        # Queue size
        queue-size = 10000

        # Keep alive time in seconds
        keep-alive = 60
    }

    # Caching
    cache {
        # Enable caching
        enabled = true

        # User cache size
        user-cache-size = 1000

        # Server cache size
        server-cache-size = 100

        # Cache expiration in milliseconds
        expiration = 300000
    }

    # Database query optimization
    database-optimization {
        # Use prepared statements
        prepared-statements = true

        # Query timeout in seconds
        query-timeout = 30

        # Enable query logging
        query-logging = false
    }
}

# ==========================================
# SECURITY
# ==========================================
security {
    # Encryption settings
    encryption {
        # Enable config encryption
        enabled = false

        # Encryption key (if enabled)
        key = "your-encryption-key"
    }

    # Password security
    password-security {
        # Enable forbidden passwords list
        forbidden-passwords-enabled = true

        # Forbidden passwords file location
        forbidden-passwords-file = "forbidden-passwords.txt"

        # Check password strength
        check-strength = true

        # Minimum strength score (0-4)
        min-strength-score = 2
    }

    # Rate limiting
    rate-limiting {
        # Enable rate limiting
        enabled = true

        # Max requests per minute per IP
        max-requests-per-minute = 60

        # Burst capacity
        burst-capacity = 10

        # Block duration in seconds
        block-duration = 300
    }

    # CSRF protection
    csrf {
        # Enable CSRF tokens
        enabled = true

        # Token timeout in minutes
        timeout = 10
    }
}

# ==========================================
# INTEGRATIONS
# ==========================================
integrations {
    # Floodgate (Bedrock) integration
    floodgate {
        # Enable Floodgate support
        enabled = false

        # Use Floodgate UUIDs
        use-floodgate-uuids = true

        # Prefix for Bedrock player names
        prefix = ""
    }

    # LuckPerms integration
    luckperms {
        # Enable LuckPerms context provider
        enabled = false

        # Context prefix
        context-prefix = "nexauth:"
    }

    # PlaceholderAPI integration
    placeholderapi {
        # Enable PlaceholderAPI placeholders
        enabled = false

        # Update interval in milliseconds
        update-interval = 1000
    }

    # Vault integration
    vault {
        # Enable Vault economy integration
        enabled = false
    }

    # Discord integration
    discord {
        # Enable Discord webhook notifications
        enabled = false

        # Webhook URL
        webhook-url = ""

        # Notify on authentication
        notify-auth = true

        # Notify on failed login
        notify-failed-login = true

        # Notify on password change
        notify-password-change = true
    }
}

# ==========================================
# MIGRATION CONFIGURATION
# ==========================================
migration {
    # Enable migration
    enabled = false

    # Source database type
    from = "authme-mysql"

    # Migration batch size
    batch-size = 100

    # Old database connection
    old-database {
        mysql {
            host = "localhost"
            port = 3306
            database = "old_db"
            username = "old_user"
            password = "old_pass"
        }

        postgresql { /* ... */ }

        sqlite {
            file = "/path/to/old.db"
        }
    }

    # Migration options
    options {
        # Convert hashes to new algorithm
        convert-hashes = true

        # Default password for users without password
        default-password = "changeme123"

        # Skip premium users during migration
        skip-premium = false

        # Keep old database intact
        keep-old-database = true

        # Validate data during migration
        validate-data = true
    }
}

# ==========================================
# LOCALIZATION
# ==========================================
localization {
    # Default language
    # Options: en, de, fr, es, pt, ru, zh-CN, etc.
    default-language = "en"

    # Enable fallback to English for missing translations
    fallback-to-english = true

    # Message cache size
    cache-size = 1000

    # Reload messages from file
    auto-reload = false
}

# ==========================================
# METRICS
# ==========================================
metrics {
    # Enable bStats metrics
    enabled = true

    # Report player count
    report-player-count = true

    # Report plugin version
    report-version = true

    # Custom metrics
    custom {
        enabled = false
        metric-id = "your-metric-id"
    }
}
```

---

## Database Configuration

### MySQL

```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "password"

        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
            idleTimeout = 600000
            maxLifetime = 1800000
            leakDetectionThreshold = 60000
        }

        ssl {
            enabled = false
            # Provide certificate files if needed
            # ssl-cert = "/path/to/client-cert.pem"
            # ssl-key = "/path/to/client-key.pem"
            # ssl-ca = "/path/to/ca-cert.pem"
        }

        properties {
            useSSL = false
            allowPublicKeyRetrieval = true
            serverTimezone = "UTC"
            characterEncoding = "utf8"
        }
    }
}
```

**Pool Settings Explained**:
- `maximumPoolSize`: Maximum number of connections in pool
- `minimumIdle`: Minimum number of idle connections
- `connectionTimeout`: Timeout when getting connection (ms)
- `idleTimeout`: Close connections after idle time (ms)
- `maxLifetime`: Maximum connection lifetime (ms)
- `leakDetectionThreshold`: Detect connection leaks (ms)

### PostgreSQL

```hocon
database {
    type = "nexauth-postgresql"
    postgresql {
        host = "localhost"
        port = 5432
        database = "nexauth"
        username = "nexauth"
        password = "password"

        pool {
            maximumPoolSize = 10
            minimumIdle = 5
            connectionTimeout = 5000
            idleTimeout = 600000
            maxLifetime = 1800000
        }

        ssl {
            enabled = false
            mode = "prefer"  # disable, require, verify-ca, verify-full
            cert = "/path/to/cert"
        }

        properties {
            ssl = false
            # PostgreSQL specific properties
        }
    }
}
```

### SQLite

```hocon
database {
    type = "nexauth-sqlite"
    sqlite {
        file = "data/nexauth.db"

        # Enable WAL (Write-Ahead Logging) for better concurrency
        wal = true

        # Busy timeout in milliseconds
        busyTimeout = 30000

        # Page size (default: 4096)
        pageSize = 4096

        # Cache size in pages (default: 2000)
        cacheSize = 2000
    }
}
```

**SQLite Recommendations**:
- Best for small to medium servers (< 1000 users)
- WAL mode improves concurrency
- Regular backups (just copy the .db file)

---

## Authentication Configuration

```hocon
authentication {
    # Maximum login attempts before temporary ban
    # Recommended: 5
    max-login-attempts = 5

    # Temporary ban duration in milliseconds
    # Default: 300000 (5 minutes)
    temp-ban-duration = 300000

    # Session timeout in milliseconds
    # Default: 3600000 (1 hour)
    session-timeout = 3600000

    # Password length requirements
    minimum-password-length = 8
    maximum-password-length = 128

    # Require password confirmation on registration
    require-password-confirmation = true

    # Allow registration
    allow-registration = true

    # Auto-login premium players
    auto-login-premium = true

    # Password complexity requirements
    password-rules {
        require-uppercase = false
        require-lowercase = false
        require-numbers = false
        require-special = false
        max-repeated-chars = 3
    }

    # Name validation
    name-validation {
        min-length = 3
        max-length = 16
        pattern = "[a-zA-Z0-9_]+"
        case-sensitive = false
        allow-digits = true
    }
}
```

**Security Recommendations**:
- Set `min-login-attempts` to 5 or lower
- Use `temp-ban-duration` of at least 5 minutes
- Set `minimum-password-length` to at least 8
- Use `require-password-confirmation` to prevent typos

---

## Server Configuration

```hocon
servers {
    # Limbo servers - create these before starting!
    limbo = ["limbo"]

    # Lobby servers
    lobby {
        root = "lobby"
        lobbies = ["lobby-1", "lobby-2"]
    }

    # Remember last server for each player
    remember-last-server = true

    # Default server (if remember-last-server is false)
    default-server = "lobby"

    # Server selection strategy
    selection-strategy = "round-robin"  # round-robin, least-players, random, hash

    # Backend protection
    protection {
        bungeeguard-enabled = false
        allowed-proxy-ips = []
        verify-players = true
    }

    # MultiProxy setup
    multiproxy {
        enabled = false
        proxy-id = "proxy-1"
        shared-secret = "your-secret"
        proxies = ["proxy-1", "proxy-2"]
    }
}
```

**Setup Instructions**:

1. **Create Limbo Server**:
   - Install [NexLimbo](https://github.com/Xreatlabs/NexLimbo)
   - Or create a minimal server with just NexAuth
   - Name it "limbo" (or update config to match)

2. **Create Lobby Server**:
   - Create your main game server
   - Name it "lobby" (or update config to match)
   - Players will be sent here after authentication

3. **Update Server Names**:
   - If your servers have different names, update `limbo` and `lobby` arrays
   - Example: `limbo = ["auth-server"]`

---

## Crypto Configuration

```hocon
crypto {
    # Default hashing algorithm
    default-provider = "argon2id"

    # Argon2ID (RECOMMENDED - Highest Security)
    argon2id {
        iterations = 3
        memory = 65536
        parallelism = 1
        hash-length = 32
    }

    # BCrypt2A (High Security)
    bcrypt2a {
        cost = 12
        variant = "2a"
    }

    # SHA-256/SHA-512 (LEGACY - Only for Migration)
    sha256 {
        iterations = 1
    }
}
```

### Choosing an Algorithm

| Algorithm | Security | Speed | Recommendation |
|-----------|----------|-------|----------------|
| Argon2ID | ⭐⭐⭐⭐⭐ | Medium | **RECOMMENDED** - Best for new installations |
| BCrypt2A | ⭐⭐⭐⭐ | Slow | Good alternative to Argon2ID |
| SHA-256 | ⭐ | Fast | **NOT RECOMMENDED** - Only for migration |

### Argon2ID Tuning

```hocon
argon2id {
    # For high-security environments
    iterations = 5
    memory = 131072  # 128 MB
    parallelism = 2

    # For low-resource servers
    iterations = 2
    memory = 32768   # 32 MB
    parallelism = 1
}
```

**Memory Usage**: Argon2ID will use approximately `memory` KB per hash. With default settings (64 MB), NexAuth can use 64 MB per concurrent authentication.

---

## 2FA Configuration

```hocon
totp {
    enabled = true
    issuer = "YourServerName"
    code-length = 6
    time-window = 30
    recovery-codes-count = 10
    allow-disable = true
    enforce = false

    qr {
        enabled = true
        size = 300
        format = "PNG"
        error-correction = "M"
    }
}
```

**Options Explained**:
- `issuer`: Name shown in authenticator app (e.g., "MyMinecraftServer")
- `code-length`: Usually 6 digits
- `time-window`: Seconds of time drift allowed (30 is standard)
- `recovery-codes-count`: How many recovery codes to generate (default: 10)
- `enforce`: Require ALL users to use 2FA (dangerous!)

**Supported Authenticator Apps**:
- Google Authenticator
- Microsoft Authenticator
- Authy
- 1Password
- LastPass Authenticator

---

## Email Configuration

```hocon
mail {
    enabled = false
    host = "smtp.gmail.com"
    port = 587
    username = "your-email@gmail.com"
    password = "your-app-password"
    from = "NexAuth <your-email@gmail.com>"
    use-tls = true
    use-ssl = false
    connection-timeout = 5000
    read-timeout = 10000

    templates {
        verification {
            subject = "Verify your NexAuth account"
            content = "Your verification code is: {code}"
            format = "TEXT"  # or HTML
        }
        password-reset {
            subject = "NexAuth Password Reset"
            content = "Your password reset code is: {code}"
            format = "TEXT"
        }
    }

    rate-limiting {
        max-per-hour = 10
        cooldown = 60
    }
}
```

### Gmail Setup

1. Enable 2-Factor Authentication on your Google account
2. Generate an App Password:
   - Go to Google Account settings
   - Security > 2-Step Verification
   - App passwords > Generate
   - Copy the generated password
3. Use the app password in `password` field (NOT your regular password)

### Other SMTP Providers

**Outlook/Hotmail**:
```hocon
mail {
    host = "smtp-mail.outlook.com"
    port = 587
    use-tls = true
}
```

**Yahoo**:
```hocon
mail {
    host = "smtp.mail.yahoo.com"
    port = 587
    use-tls = true
}
```

**Custom SMTP Server**:
```hocon
mail {
    host = "your-smtp-server.com"
    port = 25
    username = "your-username"
    password = "your-password"
    use-tls = false  # or true depending on server
}
```

---

## Logging Configuration

```hocon
logging {
    level = "INFO"
    file = "logs/nexauth.log"
    max-file-size = 10
    max-backup-files = 5
    console = true
    debug = false

    patterns {
        console = "[{yyyy-MM-dd HH:mm:ss}] [{LEVEL}] [{COMPONENT}] {MESSAGE}{EXCEPTION}"
        file = "{yyyy-MM-dd HH:mm:ss.SSS} [{LEVEL}] [{COMPONENT}] {MESSAGE}{EXCEPTION}"
    }

    console-colors = true
}
```

**Log Levels**:
- `OFF`: Disable logging
- `ERROR`: Only errors
- `WARN`: Warnings and errors
- `INFO`: General information, warnings, and errors
- `DEBUG`: Detailed debug information
- `TRACE`: Extremely detailed (usually not needed)

**Debug Mode**: Enable for troubleshooting. Remember to disable in production!

---

## Advanced Configuration

### MultiProxy Setup

```hocon
multiproxy {
    enabled = true
    proxy-id = "proxy-1"
    shared-secret = "your-secret-key-here"
    proxies = ["proxy-1", "proxy-2", "proxy-3"]
}
```

**Requirements**:
1. Same shared secret on all proxies
2. Unique `proxy-id` for each proxy
3. NexAuth installed on all proxies

### BungeeGuard Integration

```hocon
servers {
    protection {
        bungeeguard-enabled = true
        allowed-proxy-ips = ["127.0.0.1"]
        verify-players = true
    }
}
```

### Session Management

```hocon
session {
    enabled = true
    timeout = 3600000  # 1 hour
    remember-duration = 2592000000  # 30 days

    ip-validation {
        allow-ip-change = false
        strict-mode = true
    }

    storage {
        backend = "database"
        cleanup-enabled = true
        cleanup-interval = 60
    }
}
```

### Performance Tuning

```hocon
performance {
    async {
        pool-size = 4
        queue-size = 10000
        keep-alive = 60
    }

    cache {
        enabled = true
        user-cache-size = 1000
        server-cache-size = 100
        expiration = 300000
    }
}
```

---

## Environment Variables

You can use environment variables in your configuration:

```hocon
database {
    host = "${DB_HOST}"
    port = ${DB_PORT}
    username = "${DB_USER}"
    password = "${DB_PASSWORD}"
}

mail {
    username = "${MAIL_USERNAME}"
    password = "${MAIL_PASSWORD}"
}
```

**Setting Environment Variables**:

**Linux/macOS**:
```bash
export DB_HOST="localhost"
export DB_USER="nexauth"
export DB_PASSWORD="password"
```

**Windows**:
```cmd
set DB_HOST=localhost
set DB_USER=nexauth
set DB_PASSWORD=password
```

**Systemd**:
```ini
[Service]
Environment=DB_HOST=localhost
Environment=DB_USER=nexauth
Environment=DB_PASSWORD=password
```

**Docker**:
```yaml
services:
  velocity:
    image: velocity
    environment:
      - DB_HOST=db
      - DB_USER=nexauth
      - DB_PASSWORD=password
```

---

## Configuration Migration

### From LibreLogin

NexAuth automatically migrates LibreLogin configuration on first startup.

**Migration Process**:
1. NexAuth detects existing LibreLogin config
2. Migrates configuration values
3. Logs migration status
4. Creates new config with migrated values

### Manual Migration

If automatic migration fails, manually copy settings:

```hocon
# Old LibreLogin config
authme {
    max-attempts = 3
    session-timeout = 3600
}

# Migrated to NexAuth
authentication {
    max-login-attempts = 3
    session-timeout = 3600000  # Convert to milliseconds
}
```

### Config Versioning

NexAuth tracks configuration versions:

```hocon
config {
    version = 3
    last-updated = "2025-10-30"
}
```

When upgrading NexAuth:
1. Old config is backed up
2. New default config is created
3. Your settings are migrated if possible

---

## Validation Rules

NexAuth validates configuration on load:

### Database Validation

```hocon
# Required fields
database {
    type = "nexauth-mysql"  # Must be valid type
    mysql {
        host = "localhost"   # Must not be empty
        port = 3306          # Must be 1-65535
        database = "nexauth" # Must not be empty
        username = "user"    # Must not be empty
        password = "pass"    # Must not be empty
    }
}
```

**Validation Checks**:
- Type must be a valid provider
- Host must not be empty
- Port must be valid (1-65535)
- Database name must not be empty
- Username must not be empty

### Authentication Validation

```hocon
authentication {
    max-login-attempts = 5    # Must be 1-20
    temp-ban-duration = 300000 # Must be > 0
    session-timeout = 3600000  # Must be > 0
    minimum-password-length = 8 # Must be 4-128
    maximum-password-length = 128 # Must be >= minimum
}
```

**Validation Checks**:
- Login attempts: 1-20
- Timeouts: > 0 milliseconds
- Password length: 4-128 characters
- Max length >= Min length

### Server Validation

```hocon
servers {
    limbo = ["limbo"]  # List must not be empty
    lobby {
        root = "lobby"  # Must not be empty
    }
}
```

**Validation Checks**:
- Limbo list must not be empty
- Lobby root must not be empty
- Server names must be valid

### Crypto Validation

```hocon
crypto {
    default-provider = "argon2id"  # Must be registered provider
    argon2id {
        iterations = 3     # Must be 1-10
        memory = 65536     # Must be 1024-1048576
        parallelism = 1    # Must be 1-4
    }
}
```

**Validation Checks**:
- Provider must be registered
- Iterations: 1-10
- Memory: 1024-1048576 KB
- Parallelism: 1-4

---

## Troubleshooting

### Common Configuration Errors

#### Error: "Database type doesn't exist"

**Cause**: Invalid database type in `database.type`

**Solution**:
```hocon
# Wrong
database {
    type = "mysql"  # Invalid!
}

# Correct
database {
    type = "nexauth-mysql"  # Valid
}
```

#### Error: "Invalid port number"

**Cause**: Port outside valid range

**Solution**:
```hocon
# Wrong
database {
    mysql {
        port = 70000  # Too high!
    }
}

# Correct
database {
    mysql {
        port = 3306  # Valid MySQL port
    }
}
```

#### Error: "Memory must be between 1024 and 1048576"

**Cause**: Argon2ID memory setting invalid

**Solution**:
```hocon
# Wrong
crypto {
    argon2id {
        memory = 100  # Too low!
    }
}

# Correct
crypto {
    argon2id {
        memory = 65536  # Valid (64 MB)
    }
}
```

#### Error: "Lobby server is also a limbo server"

**Cause**: Server listed in both limbo and lobby

**Solution**:
```hocon
# Wrong
servers {
    limbo = ["limbo"]
    lobby {
        root = "limbo"  # Same as limbo!
    }
}

# Correct
servers {
    limbo = ["limbo"]
    lobby {
        root = "lobby"  # Different server
    }
}
```

### Checking Configuration

#### Verify Config Syntax

Run NexAuth and check logs for validation errors:
```log
[INFO] [NexAuth] Configuration loaded successfully
```

#### Test Database Connection

```bash
# Start NexAuth
# Check logs for:
[INFO] [NexAuth] Connecting to the database...
[INFO] [NexAuth] Successfully connected to the database
```

#### Validate Configuration

Use `/nexauth reload` to reload and validate config:
```log
[INFO] [NexAuth] Configuration reloaded successfully
```

### Debugging Configuration

#### Enable Debug Logging

```hocon
logging {
    level = "DEBUG"
    debug = true
}
```

This will show:
- Configuration loading process
- Database connection attempts
- Validation errors
- Each config value being applied

#### Check Loaded Configuration

Logs show config values:
```log
[INFO] [NexAuth] Using database: nexauth-mysql
[INFO] [NexAuth] Default crypto provider: argon2id
[INFO] [NexAuth] Max login attempts: 5
```

### Resetting Configuration

To reset to defaults:

1. Stop your server
2. Delete `config.conf`
3. Start server (creates new default config)
4. Edit with your settings

```bash
# Backup current config first!
cp plugins/NexAuth/config.conf config.conf.backup

# Reset
rm plugins/NexAuth/config.conf

# Start server - new default config will be created
java -jar velocity.jar
```

### Using Includes

Split large config into multiple files:

**config.conf**:
```hocon
database = ${?env.DATABASE_CONFIG}
crypto = ${?env.CRYPTO_CONFIG}
authentication = ${?env.AUTH_CONFIG}
```

**database.conf** (external file):
```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        # ... rest of database config
    }
}
```

Then set environment variables:
```bash
export DATABASE_CONFIG="file:/path/to/database.conf"
```

---

## Best Practices

### 1. Use Environment Variables for Secrets

```hocon
database {
    username = "${DB_USERNAME}"
    password = "${DB_PASSWORD}"
}

mail {
    password = "${MAIL_PASSWORD}"
}
```

### 2. Keep Config Version Controlled

```bash
# Git
git add config.conf

# But exclude sensitive data!
# Use environment variables for passwords
```

### 3. Document Custom Settings

```hocon
# Custom setting for my server
custom {
    # This enables premium rewards
    premium-rewards = true
    # Contact: admin@myserver.com for questions
}
```

### 4. Use Includes for Modular Config

```hocon
# config.conf
include "database.conf"
include "servers.conf"
include "features.conf"
```

### 5. Validate Before Production

1. Test config with debug logging
2. Check all required fields
3. Verify database connection
4. Test authentication flow
5. Disable debug logging in production

### 6. Backup Before Changes

```bash
# Before editing
cp config.conf config.conf.backup.$(date +%Y%m%d)

# After changes
cp config.conf config.conf.new
# Test new config
# If good, keep it
# If bad, restore backup
```

---

## Quick Reference

### Essential Settings

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

### Recommended Settings

```hocon
# For production servers
crypto {
    default-provider = "argon2id"
    argon2id {
        iterations = 3
        memory = 65536
        parallelism = 1
    }
}

authentication {
    max-login-attempts = 5
    temp-ban-duration = 300000
    minimum-password-length = 8
}

mail {
    enabled = true  # If using email features
}
```

### Performance Settings

```hocon
performance {
    cache {
        enabled = true
        user-cache-size = 1000
    }
}

database {
    mysql {
        pool {
            maximumPoolSize = 10
            minimumIdle = 5
        }
    }
}
```

### Security Settings

```hocon
security {
    password-security {
        forbidden-passwords-enabled = true
        check-strength = true
    }

    rate-limiting {
        enabled = true
        max-requests-per-minute = 60
        block-duration = 300
    }
}
```

---

## Support

### Getting Help

- **Configuration issues**: Check [User Guide - Troubleshooting](USER_GUIDE.md#troubleshooting)
- **Community support**: [Discord](https://discord.gg/m5ptM8X2Du)
- **Bug reports**: [GitHub Issues](https://github.com/xreatlabs/NexAuth/issues)
- **Questions**: [GitHub Discussions](https://github.com/xreatlabs/NexAuth/discussions)

### Providing Configuration Help

When asking for configuration help, include:

1. **Full configuration** (hide sensitive data)
   ```hocon
   database {
       type = "nexauth-mysql"
       mysql {
           host = "localhost"
           port = 3306
           database = "nexauth"
           username = "nexauth"
           password = "***HIDDEN***"  # Hide password
       }
   }
   ```

2. **Server platform**: Velocity/Bungee/Paper
3. **Database type**: MySQL/PostgreSQL/SQLite
4. **Full error logs**
5. **What you're trying to achieve**

---

**Version**: 0.0.1-beta3
**Last Updated**: 2025-10-30

For more help, visit: https://github.com/xreatlabs/NexAuth
