# NexAuth Quick Start Guide

Get NexAuth up and running in under 10 minutes!

## 🚀 Installation (5 minutes)

### Step 1: Download NexAuth

Visit [NexAuth Releases](https://github.com/xreatlabs/NexAuth/releases) and download:
- **`nexauth-velocity.jar`** - for Velocity proxy
- **`nexauth-bungee.jar`** - for BungeeCord/Waterfall
- **`nexauth-paper.jar`** - for Paper servers (standalone)

### Step 2: Install NexAuth

Place the JAR in your plugins folder:

```
velocity/
└── plugins/
    └── NexAuth/
        └── nexauth-velocity.jar
```

### Step 3: Install NexLimbo (Recommended)

For secure authentication, install NexLimbo:

1. Download from [NexLimbo releases](https://github.com/Xreatlabs/NexLimbo/releases)
2. Place in plugins folder:
   ```
   velocity/
   └── plugins/
       ├── NexAuth/
       │   └── nexauth-velocity.jar
       └── NexLimbo/
           └── NexLimbo-Velocity.jar
   ```

### Step 4: Start Your Server

```bash
java -jar velocity.jar
```

NexAuth will create default configuration files and ask you to restart.

---

## ⚙️ Basic Configuration (3 minutes)

Edit `plugins/NexAuth/config.conf`:

### Option A: MySQL (Recommended)

```hocon
database {
    type = "nexauth-mysql"
    mysql {
        host = "localhost"
        port = 3306
        database = "nexauth"
        username = "nexauth"
        password = "your-password"
    }
}

servers {
    limbo = ["limbo"]         # Your limbo server name
    lobby {
        root = "lobby"        # Your lobby server name
    }
}

crypto {
    default-provider = "argon2id"  # Most secure
}
```

### Option B: SQLite (Simple)

```hocon
database {
    type = "nexauth-sqlite"
    sqlite {
        file = "data/nexauth.db"
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

### Required: Create Servers

Before starting, create these servers:

**Limbo Server**:
- Install NexLimbo on this server
- Or create a minimal server with just NexAuth
- Name: `"limbo"` (or match your config)

**Lobby Server**:
- Your main game server
- Name: `"lobby"` (or match your config)

---

## ✅ Verify Installation (1 minute)

Restart your server and check logs:

```
✅ [INFO] [NexAuth] Enabling NexAuth v0.0.1-beta3
✅ [INFO] [NexAuth] Successfully connected to the database
✅ [INFO] [NexAuth] Schema validated
✅ [INFO] [NexAuth] NexAuth enabled successfully
```

If you see errors, see [Troubleshooting](#troubleshooting) below.

---

## 🎮 Using NexAuth (1 minute)

### For Players

**New Players (Registration)**:
```
/register <password> <confirm>
```
Example: `/register MySecurePass123 MySecurePass123`

**Returning Players (Login)**:
```
/login <password>
```
Example: `/login MySecurePass123`

Players will be:
1. Automatically sent to limbo when joining
2. Able to register/login with commands above
3. Automatically redirected to lobby after successful auth

---

## 🔐 Next Steps

### Enable 2FA (Recommended)

Edit `config.conf`:
```hocon
totp {
    enabled = true
    issuer = "YourServerName"
}
```

Players can then use `/2fa enable` for extra security.

### Setup Email (Optional)

For password reset via email:
```hocon
mail {
    enabled = true
    host = "smtp.gmail.com"
    port = 587
    username = "your-email@gmail.com"
    password = "your-app-password"  # Use Gmail App Password!
}
```

### Configure Permissions

Give yourself admin permissions:
```bash
# In your permission system (e.g., LuckPerms)
/lp user <yourname> permission set nexauth.admin true
```

Now you can use:
- `/nexauth reload` - Reload configuration
- `/nexauth users` - List users

---

## 🔧 Common Commands

### Player Commands
```bash
/register <pass> <confirm>  # Register
/login <password>           # Login
/changepassword <old> <new> # Change password
/2fa enable                 # Enable 2FA
/mail set <email>           # Set email
```

### Admin Commands
```bash
/nexauth reload                    # Reload config
/nexauth users                     # List users
/nexauth user <name>               # View user info
/nexauth migrate <from> <to>       # Migrate data
```

---

## 📊 Migration from Other Plugins

Have existing user data from AuthMe, FastLogin, etc.?

Edit `config.conf`:
```hocon
migration {
    enabled = true
    from = "authme-mysql"  # Change to your plugin

    old-database {
        mysql {
            host = "localhost"
            port = 3306
            database = "authme"
            username = "old_user"
            password = "old_pass"
        }
    }

    options {
        convert-hashes = true
    }
}
```

Restart server to run migration automatically!

---

## 🛠️ Troubleshooting

### Issue: "Database connection failed"

**Solution**:
1. Check MySQL is running: `sudo systemctl status mysql`
2. Verify credentials in config.conf
3. Test connection manually: `mysql -u nexauth -p nexauth`

### Issue: Players stuck in limbo

**Solution**:
1. Verify limbo server exists and is named correctly
2. Check server is online and accepting connections
3. Ensure backend servers are NOT directly accessible

### Issue: "Unknown command"

**Solution**:
1. Give yourself permission: `/lp user <name> permission set nexauth.* true`
2. Reload commands: `/nexauth reload`
3. Restart server if needed

### Issue: Plugin fails to load

**Solution**:
1. Check Java version: Must be Java 21+
2. Check logs for specific errors
3. Ensure all dependencies are installed
4. Verify config.conf syntax (use HOCON format)

---

## 📚 Full Documentation

This is just the basics! For complete information:

- **[User Guide](USER_GUIDE.md)** - Detailed installation, configuration, and usage
- **[Configuration Reference](CONFIGURATION.md)** - All configuration options
- **[API Reference](API_REFERENCE.md)** - API documentation for developers
- **[Developer Guide](DEVELOPER_GUIDE.md)** - Extending and integrating NexAuth
- **[Architecture](ARCHITECTURE.md)** - System design and concepts

---

## 🆘 Need Help?

- **Documentation**: https://github.com/xreatlabs/NexAuth/wiki
- **Discord**: https://discord.gg/m5ptM8X2Du
- **GitHub Issues**: https://github.com/xreatlabs/NexAuth/issues
- **Spigot Resource**: https://www.spigotmc.org/resources/nexauth.101040/

---

## ✅ Checklist

Before going live, make sure:

- [ ] NexAuth installed and enabled
- [ ] Database configured and connected
- [ ] Limbo and lobby servers created
- [ ] Players can register and login
- [ ] Admin permissions configured
- [ ] 2FA enabled (recommended)
- [ ] Email configured (optional)
- [ ] Backup strategy in place
- [ ] Logs being monitored
- [ ] Configuration backed up

---

## 🎯 Quick Tips

**Security**:
- ✅ Use Argon2ID (default - most secure)
- ✅ Enable 2FA for admins
- ✅ Use NexLimbo for secure auth
- ✅ Regular backups of database

**Performance**:
- ✅ Use MySQL/PostgreSQL for large servers
- ✅ Adjust connection pool size if needed
- ✅ Monitor database performance

**Best Practices**:
- ✅ Backup config before changes
- ✅ Keep plugins updated
- ✅ Monitor logs for issues
- ✅ Document custom settings

---

**Ready to go!** 🎉

Your server now has secure, modern authentication. Players can register and login safely with optional 2FA protection.

For advanced configuration and features, check the full documentation linked above!
