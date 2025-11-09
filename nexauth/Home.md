# NexAuth Documentation

Welcome to the NexAuth documentation! This wiki contains comprehensive guides and references for NexAuth, a modern Minecraft authentication plugin.

## 📚 Documentation Index

### For Users

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| **[User Guide](USER_GUIDE)** | Complete installation, configuration, and usage instructions | Server owners, administrators |
| **[Quick Start Guide](QUICK_START)** | Quick overview and basic setup | New users |

### For Developers

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| **[Developer Guide](DEVELOPER_GUIDE)** | How to extend, integrate, and contribute to NexAuth | Plugin developers |
| **[API Reference](API_REFERENCE)** | Complete API documentation with examples | Developers integrating with NexAuth |
| **[Architecture](ARCHITECTURE)** | System architecture, design patterns, and technical details | Contributors, advanced developers |

## 🎯 Choose Your Path

### I'm a Server Owner

👉 Start with: **[User Guide](USER_GUIDE)**

You'll learn:
- How to install NexAuth
- Configuration options
- Database setup
- Using commands
- Setting up 2FA
- Migration from other plugins
- Troubleshooting

### I'm a Plugin Developer

👉 Start with: **[Developer Guide](DEVELOPER_GUIDE)**

You'll learn:
- How to integrate NexAuth with your plugin
- How to extend NexAuth's functionality
- How to create custom database providers
- How to listen to and fire events
- Best practices and code examples

### I Want to Understand the Architecture

👉 Read: **[Architecture](ARCHITECTURE)**

You'll learn:
- System overview
- Design patterns used
- Data flow
- Security architecture
- Database design
- Event system details

### I Need API Details

👉 Reference: **[API Reference](API_REFERENCE)**

You'll find:
- Complete interface documentation
- Method signatures
- Usage examples
- Event reference
- Configuration keys

## 📖 Quick Reference

### Essential Commands

**Player Commands**:
```bash
/register <password> <confirm>  # Register new account
/login <password>               # Login to account
/changepassword <old> <new>     # Change password
/2fa enable                     # Enable 2FA
/mail set <email>               # Set email address
```

**Admin Commands**:
```bash
/nexauth reload                 # Reload configuration
/nexauth migrate <from> <to>    # Migrate from other plugins
/nexauth users                  # List users
```

### Quick Configuration

Minimum viable configuration:

```hocon
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

## 🔗 External Resources

- **GitHub Repository**: https://github.com/xreatlabs/NexAuth
- **Wiki**: https://github.com/xreatlabs/NexAuth/wiki
- **Issues**: https://github.com/xreatlabs/NexAuth/issues
- **Discussions**: https://github.com/xreatlabs/NexAuth/discussions
- **Discord Support**: https://discord.gg/m5ptM8X2Du
- **Spigot Resource**: https://www.spigotmc.org/resources/nexauth.101040/
- **Releases**: https://github.com/xreatlabs/NexAuth/releases

---

**Happy coding and thank you for using NexAuth!** 🚀

*For the complete documentation, visit the [User Guide](USER_GUIDE) or [Developer Guide](DEVELOPER_GUIDE).*
