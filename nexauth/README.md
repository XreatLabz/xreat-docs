# NexAuth Documentation

Welcome to the NexAuth documentation! This directory contains comprehensive guides and references for NexAuth, a modern Minecraft authentication plugin.

## 📚 Documentation Index

### For Users

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| **[Quick Start Guide](QUICK_START.md)** | Get up and running in under 10 minutes | New users |
| **[User Guide](USER_GUIDE.md)** | Complete installation, configuration, and usage instructions | Server owners, administrators |

### For Developers

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| **[Developer Guide](DEVELOPER_GUIDE.md)** | How to extend, integrate, and contribute to NexAuth | Plugin developers |
| **[API Reference](API_REFERENCE.md)** | Complete API documentation with examples | Developers integrating with NexAuth |
| **[Architecture](ARCHITECTURE.md)** | System architecture, design patterns, and technical details | Contributors, advanced developers |

## 🎯 Choose Your Path

### I'm a Server Owner

👉 Start with: **[Quick Start Guide](QUICK_START.md)** for fast setup

Then read: **[User Guide](USER_GUIDE.md)** for detailed information

You'll learn:
- How to install NexAuth (5 minutes!)
- Basic configuration
- Database setup
- Using commands
- Setting up 2FA
- Migration from other plugins
- Troubleshooting

### I'm a Plugin Developer

👉 Start with: **[Developer Guide](DEVELOPER_GUIDE.md)**

You'll learn:
- How to integrate NexAuth with your plugin
- How to extend NexAuth's functionality
- How to create custom database providers
- How to listen to and fire events
- Best practices and code examples

### I Want to Understand the Architecture

👉 Read: **[Architecture](ARCHITECTURE.md)**

You'll learn:
- System overview
- Design patterns used
- Data flow
- Security architecture
- Database design
- Event system details

### I Need API Details

👉 Reference: **[API Reference](API_REFERENCE.md)**

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

## 🏗️ Project Structure

```
NexAuth/
├── API/                          # API module (interfaces)
├── Plugin/                       # Implementation module
├── NexLimbo/                     # Companion limbo server
├── docs/                         # Documentation (this directory)
│   ├── README.md                 # This file
│   ├── USER_GUIDE.md            # User documentation
│   ├── DEVELOPER_GUIDE.md       # Developer documentation
│   ├── API_REFERENCE.md         # API documentation
│   └── ARCHITECTURE.md          # Architecture documentation
└── README.md                     # Project overview
```

## 🔗 External Resources

- **GitHub Repository**: https://github.com/xreatlabs/NexAuth
- **Wiki**: https://github.com/xreatlabs/NexAuth/wiki
- **Issues**: https://github.com/xreatlabs/NexAuth/issues
- **Discussions**: https://github.com/xreatlabs/NexAuth/discussions
- **Discord Support**: https://discord.gg/m5ptM8X2Du
- **Spigot Resource**: https://www.spigotmc.org/resources/nexauth.101040/
- **Releases**: https://github.com/xreatlabs/NexAuth/releases

## 📋 Table of Contents by Topic

### Getting Started
1. [Read the README](../README.md)
2. **[Quick Start Guide](QUICK_START.md)** - Get up and running in 10 minutes!
3. [User Guide - Installation](USER_GUIDE.md#installation) - Detailed installation
4. [User Guide - Configuration](USER_GUIDE.md#configuration) - Full configuration options

### Configuration
1. [User Guide - Configuration](USER_GUIDE.md#configuration)
2. [API Reference - Configuration](API_REFERENCE.md#configuration-api)
3. [Example configurations in User Guide](USER_GUIDE.md#basic-configuration)

### Database
1. [User Guide - Database Setup](USER_GUIDE.md#database-setup)
2. [Architecture - Database Design](ARCHITECTURE.md#database-design)
3. [API Reference - Database API](API_REFERENCE.md#database-api)
4. [Developer Guide - Custom Database Providers](DEVELOPER_GUIDE.md#custom-database-providers)

### Authentication Flow
1. [Architecture - Data Flow](ARCHITECTURE.md#data-flow)
2. [User Guide - Using NexAuth](USER_GUIDE.md#using-nexauth)
3. [API Reference - Authorization API](API_REFERENCE.md#authorization-api)

### Events
1. [Architecture - Event System](ARCHITECTURE.md#event-system)
2. [API Reference - Event System](API_REFERENCE.md#event-system)
3. [Developer Guide - Custom Event Handlers](DEVELOPER_GUIDE.md#custom-event-handlers)

### 2FA (Two-Factor Authentication)
1. [User Guide - 2FA Setup](USER_GUIDE.md#2fa-setup)
2. [API Reference - TOTP API](API_REFERENCE.md#totp-api)

### Migration
1. [User Guide - Migration](USER_GUIDE.md#migration-from-other-plugins)
2. [Migration providers in API Reference](API_REFERENCE.md#supported-database-providers)

### Security
1. [Architecture - Security Architecture](ARCHITECTURE.md#security-architecture)
2. [API Reference - Crypto API](API_REFERENCE.md#crypto-api)

### Commands
1. [User Guide - Commands Reference](USER_GUIDE.md#commands-reference)
2. [API Reference - Command API](API_REFERENCE.md#command-api)

### Extending NexAuth
1. [Developer Guide - Extending NexAuth](DEVELOPER_GUIDE.md#extending-nexauth)
2. [Developer Guide - Custom Providers](DEVELOPER_GUIDE.md#custom-database-providers)

### Testing
1. [Developer Guide - Testing](DEVELOPER_GUIDE.md#testing)

### Contributing
1. [Developer Guide - Contributing](DEVELOPER_GUIDE.md#contributing)
2. [Best Practices](DEVELOPER_GUIDE.md#best-practices)

## 🎓 Learning Path

### For Server Owners

```
🚀 Follow Quick Start Guide (10 minutes)
   ↓
📖 Read User Guide for details
   ↓
⚙️ Configure NexAuth fully
   ↓
📊 Setup Database properly
   ↓
✅ Test Installation
   ↓
👥 Train Users
   ↓
🔐 Setup 2FA (Recommended)
   ↓
🔄 Setup Migration (If needed)
```

### For Plugin Developers

```
📖 Read README (Overview)
   ↓
📚 Study Architecture
   ↓
🔍 Review API Reference
   ↓
👨‍💻 Read Developer Guide
   ↓
🧪 Setup Development Environment
   ↓
🔌 Create Integration
   ↓
✅ Test Integration
   ↓
🚀 Deploy & Maintain
```

### For Contributors

```
📖 Read README
   ↓
📚 Study Architecture
   ↓
🔍 Review API Reference
   ↓
👨‍💻 Read Developer Guide
   ↓
🧪 Setup Development Environment
   ↓
💡 Identify Area to Contribute
   ↓
🔨 Make Changes
   ↓
✅ Run Tests
   📤 Submit Pull Request
```

## 📊 Documentation Coverage

| Feature | User Guide | API Reference | Architecture | Developer Guide |
|---------|------------|---------------|--------------|-----------------|
| Installation | ✅ | - | - | - |
| Configuration | ✅ | ✅ | - | - |
| Database | ✅ | ✅ | ✅ | ✅ |
| Authentication | ✅ | ✅ | ✅ | - |
| Events | - | ✅ | ✅ | ✅ |
| 2FA | ✅ | ✅ | - | - |
| Migration | ✅ | ✅ | - | - |
| Commands | ✅ | ✅ | - | - |
| API Details | - | ✅ | ✅ | - |
| Customization | - | - | - | ✅ |
| Integration | - | - | - | ✅ |
| Contributing | - | - | - | ✅ |

## 🆘 Getting Help

### Before Asking

1. **Check the User Guide** for common issues
2. **Search existing issues** on GitHub
3. **Check the FAQ** in the User Guide
4. **Review the logs** for error messages

### Where to Ask

| Question Type | Best Place |
|---------------|------------|
| How do I install? | **[Quick Start Guide](QUICK_START.md)** - 10 minute setup! |
| Configuration help | [User Guide](USER_GUIDE.md#configuration) or GitHub Discussions |
| Bug report | [GitHub Issues](https://github.com/xreatlabs/NexAuth/issues) |
| Feature request | [GitHub Issues](https://github.com/xreatlabs/NexAuth/issues) |
| General questions | [GitHub Discussions](https://github.com/xreatlabs/NexAuth/discussions) |
| Quick help | [Discord](https://discord.gg/m5ptM8X2Du) |
| API integration | [Developer Guide](DEVELOPER_GUIDE.md) or Discord |

### When Asking for Help

Please include:
- ✅ NexAuth version (`/nexauth version`)
- ✅ Platform (Velocity/Bungee/Paper)
- ✅ Configuration (hide sensitive data)
- ✅ Full error logs
- ✅ Steps to reproduce
- ✅ What you were trying to do

### Common Solutions

| Issue | Quick Fix |
|-------|-----------|
| Database connection failed | Check credentials in config.conf |
| Players stuck in limbo | Verify limbo server exists and config matches |
| Can't use commands | Check permissions |
| Old plugin migration | Check migration configuration |
| 2FA codes not working | Check server time sync |

## 📝 Documentation Standards

### For Contributors

When updating documentation:

1. **Keep it clear**: Use simple language, avoid jargon
2. **Use examples**: Code samples make concepts concrete
3. **Be consistent**: Follow existing formatting and style
4. **Stay updated**: Sync docs with code changes
5. **Test instructions**: Verify steps work as documented

### File Naming

- Use descriptive names: `USER_GUIDE.md` not `guide.md`
- Use underscores, not hyphens or spaces
- One concept per file

### Sections

Each guide should have:
- Clear introduction
- Table of contents
- Prerequisites
- Step-by-step instructions
- Examples
- Troubleshooting section
- FAQ (if applicable)

## 🔄 Staying Updated

NexAuth documentation is updated with each release. To stay informed:

1. **Watch the repository** on GitHub
2. **Join our Discord** for announcements
3. **Check releases page** for changelogs
4. **Read CHANGELOG.md** in project root

### Documentation Versions

- Documentation matches the latest release
- Check the version number in each document
- Older versions may be available via Git tags

## 📈 Documentation Metrics

We track documentation quality:

- **Coverage**: How much of the API is documented
- **Clarity**: User feedback on understandability
- **Completeness**: Are all features documented?
- **Accuracy**: Does documentation match code?

## 🎨 Contributing to Documentation

Contributions welcome! See our [contribution guidelines](DEVELOPER_GUIDE.md#contributing).

Ways to help:
- 📝 Fix typos and grammar
- 📚 Expand sections
- 🌍 Add translations
- 📊 Improve examples
- 🖼️ Add diagrams
- ❓ Add FAQ items

### Documentation Style Guide

- Use present tense ("Click" not "Clicking")
- Use second person ("you")
- Use active voice
- Capitalize proper nouns
- Use code formatting for commands, config, and code
- Use tables for comparison data
- Use bullet points for lists
- Use **bold** for UI elements and important terms
- Use `code` for technical terms and commands

## 🏆 Acknowledgments

Thanks to all contributors to NexAuth documentation:

- Original plugin authors
- Documentation writers
- Tutorial creators
- Issue reporters
- Community members

## 📜 License

All documentation is licensed under the same license as NexAuth: **Mozilla Public License 2.0**.

## 📞 Contact

- **GitHub**: https://github.com/xreatlabs/NexAuth
- **Discord**: https://discord.gg/m5ptM8X2Du
- **Email**: [Contact via Discord]

---

**Happy coding and thank you for using NexAuth!** 🚀

*For the latest version of this documentation, visit: https://github.com/xreatlabs/NexAuth*
