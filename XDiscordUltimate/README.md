# XDiscordUltimate Documentation

Welcome to the comprehensive documentation for XDiscordUltimate! This documentation provides detailed information about the plugin's architecture, configuration, modules, and development.

---

## 📚 Documentation Index

### Quick Start

| Document | Description | Audience |
|----------|-------------|----------|
| [README.md](../README.md) | Project overview and quick start | All users |
| [DEVELOPER.md](DEVELOPER.md) | Development setup and guidelines | Developers |
| [INSTALLATION.md](INSTALLATION.md) | Installation instructions | Administrators |

### Core Documentation

| Document | Description | Audience |
|----------|-------------|----------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | System architecture and design | Developers |
| [API.md](API.md) | Complete API reference | Developers |
| [DATABASE.md](DATABASE.md) | Database schema and configuration | Administrators |

### Configuration & Modules

| Document | Description | Audience |
|----------|-------------|----------|
| [modules/MODULES.md](modules/MODULES.md) | All 18 module documentation | All users |
| [CONFIGURATION.md](CONFIGURATION.md) | Complete configuration guide | Administrators |
| [MESSAGES.md](MESSAGES.md) | Message customization guide | Administrators |

### Feature Guides

| Document | Description | Audience |
|----------|-------------|----------|
| [modules/CHAT-BRIDGE.md](modules/CHAT-BRIDGE.md) | Chat bridge setup and usage | All users |
| [modules/VERIFICATION.md](modules/VERIFICATION.md) | Verification system guide | All users |
| [modules/TICKETS.md](modules/TICKETS.md) | Support ticket system | All users |
| [modules/MODERATION.md](modules/MODERATION.md) | Moderation features | Moderators |
| [modules/SERVER-LOGGING.md](modules/SERVER-LOGGING.md) | Server logging setup | Administrators |

### User Guides

| Document | Description | Audience |
|----------|-------------|----------|
| [FEATURES.md](FEATURES.md) | Feature overview and comparison | All users |
| [COMMANDS.md](COMMANDS.md) | Complete commands reference | All users |
| [PERMISSIONS.md](PERMISSIONS.md) | Permissions reference | Administrators |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Common issues and solutions | All users |

### Developer Resources

| Document | Description | Audience |
|----------|-------------|----------|
| [DEVELOPER.md](DEVELOPER.md) | Development setup | Developers |
| [API.md](API.md) | API reference | Developers |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines | Developers |
| [CHANGELOG.md](CHANGELOG.md) | Version history | All users |

---

## 🎯 Documentation by Audience

### For Server Administrators

**Start Here:**
1. [INSTALLATION.md](INSTALLATION.md) - Install the plugin
2. [CONFIGURATION.md](CONFIGURATION.md) - Configure the plugin
3. [modules/MODULES.md](modules/MODULES.md) - Enable features you need

**Essential Guides:**
- [DATABASE.md](DATABASE.md) - Set up database
- [PERMISSIONS.md](PERMISSIONS.md) - Configure permissions
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Fix common issues

**Module-Specific Setup:**
- [modules/CHAT-BRIDGE.md](modules/CHAT-BRIDGE.md) - Chat integration
- [modules/VERIFICATION.md](modules/VERIFICATION.md) - Account verification
- [modules/TICKETS.md](modules/TICKETS.md) - Support system
- [modules/SERVER-LOGGING.md](modules/SERVER-LOGGING.md) - Logging

### For End Users

**Player Guides:**
1. [FEATURES.md](FEATURES.md) - What the plugin does
2. [COMMANDS.md](COMMANDS.md) - Available commands
3. [modules/VERIFICATION.md](modules/VERIFICATION.md) - How to verify

**Player Features:**
- Chat between Minecraft and Discord
- Account verification
- Support tickets
- Player statistics

### For Developers

**Development Setup:**
1. [DEVELOPER.md](DEVELOPER.md) - Set up dev environment
2. [ARCHITECTURE.md](ARCHITECTURE.md) - Understand the system
3. [API.md](API.md) - API reference

**Contributing:**
- [CONTRIBUTING.md](CONTRIBUTING.md) - How to contribute
- [CHANGELOG.md](CHANGELOG.md) - What's changed

**Customization:**
- [modules/CUSTOM-MODULES.md](modules/CUSTOM-MODULES.md) - Create custom modules
- [API.md](API.md#extension-api) - Extension points

### For Moderators

**Moderation Features:**
- [modules/MODERATION.md](modules/MODERATION.md) - Ban sync, reports
- [modules/SERVER-LOGGING.md](modules/SERVER-LOGGING.md) - Server events
- [modules/ADMIN-ALERTS.md](modules/ADMIN-ALERTS.md) - Server health alerts

---

## 🚀 Quick Start Guide

### New Users

1. **Read the Overview**
   ```markdown
   See: [README.md](../README.md)
   ```

2. **Install the Plugin**
   ```markdown
   See: [INSTALLATION.md](INSTALLATION.md)
   ```

3. **Configure Basic Features**
   ```markdown
   See: [CONFIGURATION.md](CONFIGURATION.md)
   ```

4. **Set Up Discord Bot**
   - Create Discord application
   - Add bot to server
   - Configure bot token and guild ID

5. **Enable Modules**
   - Chat Bridge (chat integration)
   - Verification (account linking)
   - Tickets (support system)

### Returning Users

1. **Check Changelog**
   ```markdown
   See: [CHANGELOG.md](CHANGELOG.md)
   ```

2. **Update Plugin**
   - Backup configuration
   - Replace JAR file
   - Restart server

3. **Review New Features**
   - Check [CHANGELOG.md](CHANGELOG.md)
   - Read new module documentation

### Developers

1. **Set Up Development Environment**
   ```markdown
   See: [DEVELOPER.md](DEVELOPER.md)
   ```

2. **Understand Architecture**
   ```markdown
   See: [ARCHITECTURE.md](ARCHITECTURE.md)
   ```

3. **Study API**
   ```markdown
   See: [API.md](API.md)
   ```

4. **Contribute**
   ```markdown
   See: [CONTRIBUTING.md](CONTRIBUTING.md)
   ```

---

## 📖 Documentation Structure

```
docs/
├── README.md                    # This file - Master index
├── INSTALLATION.md              # Installation guide
├── CONFIGURATION.md             # Configuration reference
├── FEATURES.md                  # Feature overview
├── COMMANDS.md                  # Commands reference
├── PERMISSIONS.md               # Permissions reference
├── TROUBLESHOOTING.md           # Troubleshooting guide
├── ARCHITECTURE.md              # System architecture
├── API.md                       # API documentation
├── DATABASE.md                  # Database documentation
├── DEVELOPER.md                 # Developer guide
├── CONTRIBUTING.md              # Contribution guidelines
├── CHANGELOG.md                 # Version history
└── modules/                     # Module-specific documentation
    ├── MODULES.md               # All modules overview
    ├── CHAT-BRIDGE.md           # Chat bridge module
    ├── VERIFICATION.md          # Verification module
    ├── TICKETS.md               # Tickets module
    ├── MODERATION.md            # Moderation module
    ├── SERVER-LOGGING.md        # Logging module
    ├── ADMIN-ALERTS.md          # Admin alerts module
    ├── BOT-CONSOLE.md           # Bot console module
    ├── ANNOUNCEMENTS.md         # Announcements module
    ├── PLAYER-STATS.md          # Player stats module
    ├── SERVER-STATUS.md         # Server status module
    ├── WELCOME-DM.md            # Welcome DM module
    ├── BOOSTER-PERKS.md         # Booster perks module
    ├── LEADERBOARD.md           # Leaderboard module
    ├── ACTIVITY-ROLES.md        # Activity roles module
    ├── VOICE-CHANNELS.md        # Voice channels module
    ├── ECONOMY-BRIDGE.md        # Economy bridge module
    └── CUSTOM-MODULES.md        # Creating custom modules
```

---

## 🔍 Finding Information

### By Topic

**Setup & Installation:**
- Start: [INSTALLATION.md](INSTALLATION.md)
- Details: [CONFIGURATION.md](CONFIGURATION.md)
- Database: [DATABASE.md](DATABASE.md)

**Features:**
- Overview: [FEATURES.md](FEATURES.md)
- Modules: [modules/MODULES.md](modules/MODULES.md)
- Commands: [COMMANDS.md](COMMANDS.md)

**Development:**
- Setup: [DEVELOPER.md](DEVELOPER.md)
- Architecture: [ARCHITECTURE.md](ARCHITECTURE.md)
- API: [API.md](API.md)

**Troubleshooting:**
- Start: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Database issues: [DATABASE.md#troubleshooting](DATABASE.md#troubleshooting)
- Module issues: [modules/MODULES.md#troubleshooting](modules/MODULES.md#troubleshooting)

### By Feature

**Chat Bridge:**
- Guide: [modules/CHAT-BRIDGE.md](modules/CHAT-BRIDGE.md)
- Config: [CONFIGURATION.md#chat-bridge](CONFIGURATION.md#chat-bridge)
- Troubleshooting: [TROUBLESHOOTING.md#chat-bridge](TROUBLESHOOTING.md#chat-bridge)

**Verification:**
- Guide: [modules/VERIFICATION.md](modules/VERIFICATION.md)
- Config: [CONFIGURATION.md#verification](CONFIGURATION.md#verification)
- Database: [DATABASE.md#verification-module](DATABASE.md#verification-module)

**Tickets:**
- Guide: [modules/TICKETS.md](modules/TICKETS.md)
- Config: [CONFIGURATION.md#tickets](CONFIGURATION.md#tickets)
- Database: [DATABASE.md#tickets-table](DATABASE.md#tickets-table)

**Moderation:**
- Guide: [modules/MODERATION.md](modules/MODERATION.md)
- Config: [CONFIGURATION.md#moderation](CONFIGURATION.md#moderation)
- Database: [DATABASE.md#moderation-logs](DATABASE.md#moderation-logs)

---

## 📝 Documentation Standards

### Writing Style

**Tone:**
- Clear and concise
- Professional but approachable
- Action-oriented
- Consistent terminology

**Structure:**
- Start with overview
- Provide step-by-step instructions
- Include code examples
- Add troubleshooting tips

**Code Examples:**
- Always test before including
- Use proper formatting
- Include comments explaining key parts
- Provide context

### File Naming

- Use PascalCase for file names
- Use hyphens in file names (e.g., `QUICK-START.md`)
- Keep names descriptive
- Avoid abbreviations

### Versioning

- Documentation version matches plugin version
- Breaking changes documented in CHANGELOG.md
- Update all relevant docs when making changes
- Mark new features clearly

---

## 🤝 Contributing to Documentation

### How to Help

1. **Report Issues**
   - Found a typo? Report it!
   - Unclear explanation? Report it!
   - Missing information? Report it!

2. **Submit Improvements**
   - Fork the repository
   - Edit documentation files
   - Submit pull request

3. **Write New Docs**
   - New feature needs documentation
   - Fill gaps in existing docs
   - Translate to other languages

### Documentation Guidelines

1. **Be Clear**
   - Use simple language
   - Avoid jargon
   - Explain acronyms

2. **Be Complete**
   - Cover all use cases
   - Include examples
   - Link to related topics

3. **Be Accurate**
   - Test instructions
   - Update for new versions
   - Verify commands work

4. **Be Consistent**
   - Use same terminology
   - Follow file structure
   - Match existing style

---

## 🔗 Useful Links

### Official Resources

- **GitHub Repository**: https://github.com/xreatlabs/XDiscordUltimate
- **Issue Tracker**: https://github.com/xreatlabs/XDiscordUltimate/issues
- **Discord Server**: https://discord.gg/xreatlabs
- **Spigot Resource**: https://www.spigotmc.org/resources/...

### External Resources

- [Spigot API Documentation](https://hub.spigotmc.org/javadocs/bukkit/)
- [JDA Documentation](https://jda.wiki/)
- [Discord Developer Portal](https://discord.com/developers/applications)
- [YAML Syntax Guide](https://yaml.org/spec/1.2/spec.html)

### Tools

- [YAML Validator](https://yamlvalidator.com/)
- [Discord ID Finder](https://discohook.com/)
- [JSON to YAML Converter](https://www.json2yaml.com/)
- [Markdown Editor](https://stackedit.io/)

---

## 📞 Getting Help

### Community Support

1. **Discord Server**
   - Join: https://discord.gg/xreatlabs
   - Ask questions in #support
   - Share your setup

2. **GitHub Discussions**
   - https://github.com/xreatlabs/XDiscordUltimate/discussions
   - Feature discussions
   - General questions

### Bug Reports

**Before Reporting:**
1. Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
2. Search existing issues
3. Test with debug mode enabled

**Report Bug:**
- GitHub Issues: https://github.com/xreatlabs/XDiscordUltimate/issues
- Include: Minecraft version, plugin version, error logs
- Provide: Steps to reproduce, configuration (sanitized)

### Feature Requests

**Request Feature:**
- GitHub Issues: https://github.com/xreatlabs/XDiscordUltimate/issues
- Label: "enhancement"
- Describe: Use case, benefits, alternatives considered

---

## 📊 Documentation Stats

| Category | Count |
|----------|-------|
| Total Documents | 20+ |
| Module Guides | 16 |
| API Methods | 100+ |
| Configuration Options | 200+ |
| Database Tables | 7 |

---

## 🎓 Learning Path

### New Server Administrator

1. Read [README.md](../README.md) for overview
2. Follow [INSTALLATION.md](INSTALLATION.md)
3. Configure with [CONFIGURATION.md](CONFIGURATION.md)
4. Enable modules from [modules/MODULES.md](modules/MODULES.md)
5. Reference [COMMANDS.md](COMMANDS.md) for usage

### Power User

1. All administrator steps
2. Customize with [MESSAGES.md](MESSAGES.md)
3. Enable advanced modules
4. Set up automated tasks
5. Monitor with [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

### Developer

1. All user steps
2. Study [ARCHITECTURE.md](ARCHITECTURE.md)
3. Set up dev environment from [DEVELOPER.md](DEVELOPER.md)
4. Read [API.md](API.md) thoroughly
5. Contribute via [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 📅 Maintenance

### Documentation Updates

Documentation is maintained alongside code:
- Updated with each release
- Reviewed monthly
- Community feedback incorporated

### Version Compatibility

| Documentation Version | Plugin Version | Status |
|-----------------------|----------------|--------|
| 1.1.0 | 1.1.0 | ✅ Current |
| 1.0.x | 1.0.x | ⚠️ Legacy |
| 0.9.x | 0.9.x | ❌ Deprecated |

---

Thank you for using XDiscordUltimate! This documentation is continuously improved based on community feedback. If you find it helpful, please consider contributing back by reporting issues or submitting improvements.

---

**Need Help?**
- 📧 Email: support@xreatlabs.com
- 💬 Discord: https://discord.gg/xreatlabs
- 🐛 Issues: https://github.com/xreatlabs/XDiscordUltimate/issues
- 📚 Docs: https://github.com/xreatlabs/XDiscordUltimate/wiki

---

*Last updated: December 2024*
*For XDiscordUltimate v1.1.0*