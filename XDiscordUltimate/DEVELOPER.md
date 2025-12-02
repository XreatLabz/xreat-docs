# XDiscordUltimate Developer Guide

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Environment Setup](#development-environment-setup)
3. [Building from Source](#building-from-source)
4. [Project Structure](#project-structure)
5. [Contributing Guidelines](#contributing-guidelines)
6. [API Reference](#api-reference)
7. [Custom Module Development](#custom-module-development)
8. [Testing Guidelines](#testing-guidelines)
9. [Debugging](#debugging)
10. [Code Style](#code-style)
11. [Release Process](#release-process)
12. [Troubleshooting](#troubleshooting)

---

## Getting Started

Welcome to the XDiscordUltimate Developer Guide! This guide will help you set up your development environment, understand the codebase, and start contributing to the project.

### Prerequisites

Before you begin, ensure you have:

- **Java 17+** (OpenJDK or Oracle JDK)
- **Git** (for version control)
- **Gradle 7.0+** (build automation)
- **Minecraft Server** (for testing)
- **Discord Developer Account** (for bot creation)
- **IDE** (IntelliJ IDEA recommended, Eclipse also supported)

### Quick Start

1. Clone the repository:
```bash
git clone https://github.com/xreatlabs/XDiscordUltimate.git
cd XDiscordUltimate
```

2. Build the project:
```bash
./gradlew build
```

3. Run tests:
```bash
./gradlew test
```

---

## Development Environment Setup

### Java Installation

#### Option 1: OpenJDK (Recommended)

**Linux (Ubuntu/Debian)**:
```bash
sudo apt update
sudo apt install openjdk-17-jdk
java -version
```

**Windows**:
1. Download OpenJDK 17 from [Adoptium](https://adoptium.net/)
2. Run installer and follow instructions
3. Verify: `java -version` in command prompt

**macOS**:
```bash
brew install openjdk@17
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
java -version
```

#### Option 2: Oracle JDK

Download from [Oracle](https://www.oracle.com/java/technologies/downloads/) and install according to platform instructions.

### IntelliJ IDEA Setup

1. **Install IntelliJ IDEA**:
   - Download from [jetbrains.com](https://www.jetbrains.com/idea/)
   - Install with default settings

2. **Import Project**:
   - Open IntelliJ IDEA
   - Select "Open"
   - Navigate to XDiscordUltimate folder
   - Select `build.gradle` file
   - Click "Open as Project"

3. **Configure SDK**:
   - File → Project Structure (Ctrl+Alt+Shift+S)
   - Project SDK: Select Java 17
   - Project language level: 17

4. **Configure Gradle**:
   - File → Settings → Build, Execution, Deployment → Build Tools → Gradle
   - Use Gradle from: `gradle-wrapper.properties`
   - Gradle JVM: Java 17

### Eclipse Setup

1. **Install Eclipse IDE for Java Developers**

2. **Import Project**:
   - File → Import → Gradle → Existing Gradle Project
   - Select project root directory
   - Follow import wizard

### Environment Variables

Create a `.env` file in the project root (optional, for Discord webhook uploads):

```bash
# Discord webhook for automated builds (optional)
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

---

## Building from Source

### Gradle Tasks

**Build the project**:
```bash
./gradlew build
```

**Build without running tests**:
```bash
./gradlew build -x test
```

**Clean build artifacts**:
```bash
./gradlew clean
```

**Run tests**:
```bash
./gradlew test
```

**Generate Javadoc**:
```bash
./gradlew javadoc
```

**Upload to Discord (requires .env)**:
```bash
./gradlew uploadToDiscord
```

### Shadow JAR

The project uses Gradle Shadow plugin to create an "uber-JAR" that includes all dependencies:

```bash
./gradlew shadowJar
```

This creates `build/libs/XDiscordUltimate-1.1.0.jar` with all dependencies bundled.

### Custom Build Tasks

**Build specific version**:
```bash
./gradlew shadowJar -Pversion=1.2.0
```

**Build with environment-specific config**:
```bash
./gradlew build -Denvironment=production
```

### Continuous Build

For development, use continuous build:

```bash
./gradlew build --continuous
```

This will automatically rebuild when source files change.

---

## Project Structure

```
XDiscordUltimate/
├── build.gradle                    # Build configuration
├── settings.gradle                 # Project settings
├── gradle.properties              # Gradle properties
├── gradlew                        # Gradle wrapper script (Linux/macOS)
├── gradlew.bat                    # Gradle wrapper script (Windows)
├── .gitignore                     # Git ignore rules
├── LICENSE                        # MIT License
├── README.md                      # Project README
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── xreatlabs/
        │           └── xdiscordultimate/
        │               ├── XDiscordUltimate.java       # Main plugin class
        │               ├── LibraryManager.java         # Dependency management
        │               ├── commands/                   # Commands
        │               │   ├── AnnounceCommand.java
        │               │   ├── DiscordConsoleCommand.java
        │               │   ├── HelpCommand.java
        │               │   ├── ReportCommand.java
        │               │   ├── StatsCommand.java
        │               │   ├── SupportCommand.java
        │               │   ├── VerifyCommand.java
        │               │   └── XDiscordCommand.java
        │               ├── database/                   # Database layer
        │               │   └── DatabaseManager.java
        │               ├── discord/                    # Discord integration
        │               │   ├── DiscordListener.java
        │               │   └── DiscordManager.java
        │               ├── hooks/                      # Plugin hooks
        │               │   └── PlaceholderAPIExpansion.java
        │               ├── listeners/                  # Event listeners
        │               │   ├── ChatListener.java
        │               │   ├── PlayerListener.java
        │               │   └── ServerListener.java
        │               ├── modules/                    # Feature modules
        │               │   ├── Module.java             # Base module class
        │               │   ├── ModuleManager.java      # Module management
        │               │   ├── activityroles/          # Activity roles module
        │               │   ├── adminalerts/            # Admin alerts module
        │               │   ├── announcements/          # Announcements module
        │               │   ├── boosters/               # Booster perks module
        │               │   ├── botconsole/             # Bot console module
        │               │   ├── chatbridge/             # Chat bridge module
        │               │   ├── economy/                # Economy bridge module
        │               │   ├── leaderboard/            # Leaderboard module
        │               │   ├── moderation/             # Moderation module
        │               │   ├── servercontrol/          # Server control module
        │               │   ├── serverlogging/          # Server logging module
        │               │   ├── serverstatus/           # Server status module
        │               │   ├── stats/                  # Player stats module
        │               │   ├── tickets/                # Tickets module
        │               │   ├── verification/           # Verification module
        │               │   ├── voicechannel/           # Voice channels module
        │               │   └── welcomedm/              # Welcome DM module
        │               └── utils/                      # Utility classes
        │                   ├── AdminUtils.java
        │                   ├── ColorUtils.java
        │                   ├── ConfigManager.java
        │                   ├── ConsoleAppender.java
        │                   ├── EmbedUtils.java
        │                   ├── HelpGUI.java
        │                   ├── MessageManager.java
        │                   ├── Metrics.java
        │                   ├── TextToSpeechUtils.java
        │                   ├── UpdateChecker.java
        │                   ├── VersionUtils.java
        │                   └── WebhookManager.java
        └── resources/
            ├── config.yml                    # Default configuration
            ├── messages.yml                  # Default messages
            └── plugin.yml                    # Plugin descriptor
```

### Key Directories

- **`commands/`**: All Minecraft commands
- **`database/`**: Database access layer (DatabaseManager)
- **`discord/`**: Discord bot integration (JDA)
- **`hooks/`**: Integrations with other plugins
- **`listeners/`**: Bukkit event listeners
- **`modules/`**: All feature modules
- **`utils/`**: Utility classes and helpers

### File Responsibilities

| File | Responsibility |
|------|---------------|
| `XDiscordUltimate.java` | Main plugin lifecycle |
| `ModuleManager.java` | Module loading and management |
| `DatabaseManager.java` | Database operations and schema |
| `DiscordManager.java` | Discord bot connection and API |
| `ConfigManager.java` | Configuration file management |
| `MessageManager.java` | Localized message handling |

---

## Contributing Guidelines

### Code Contribution Process

1. **Fork the Repository**:
   ```bash
   git clone https://github.com/yourusername/XDiscordUltimate.git
   ```

2. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make Changes**:
   - Follow code style guidelines
   - Write tests for new features
   - Update documentation

4. **Test Your Changes**:
   ```bash
   ./gradlew test
   ./gradlew build
   ```

5. **Commit Changes**:
   ```bash
   git add .
   git commit -m "Add feature: your feature name"
   ```

6. **Push to Fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

7. **Create Pull Request**:
   - Go to GitHub
   - Create pull request from your fork
   - Describe changes and rationale

### Pull Request Guidelines

**Before Submitting**:
- [ ] Code compiles without errors
- [ ] All tests pass
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] Changelog entry added
- [ ] No merge conflicts

**PR Description Template**:
```markdown
## Description
Brief description of changes

## Changes Made
- List specific changes
- Add/remove/modify features

## Testing
- Tested on: Minecraft version, Java version
- Test cases covered

## Breaking Changes
- List any breaking changes (if applicable)

## Additional Notes
- Any additional information
```

### Issue Reporting

**Bug Reports**:
- Use GitHub Issues
- Include Minecraft version
- Include plugin version
- Include steps to reproduce
- Include error logs
- Include configuration (sanitized)

**Feature Requests**:
- Use GitHub Issues with "enhancement" label
- Describe the feature
- Explain use case
- Provide configuration examples

---

## API Reference

### Plugin API

The plugin provides an API for other plugins to interact with it:

#### Getting Plugin Instance

```java
XDiscordUltimate plugin = XDiscordUltimate.getInstance();
```

#### Database Access

```java
DatabaseManager db = plugin.getDatabaseManager();

// Link account
CompletableFuture<Boolean> result = db.linkAccount(
    player.getUniqueId(),
    discordId,
    player.getName(),
    discordName
);

// Get Discord ID from UUID
CompletableFuture<String> discordId = db.getDiscordId(player.getUniqueId());
```

#### Module Access

```java
ModuleManager moduleManager = plugin.getModuleManager();

// Get specific module
ChatBridgeModule chatBridge = moduleManager.getModule(ChatBridgeModule.class);

// Check if module is active
boolean active = moduleManager.isModuleActive("chat-bridge");
```

#### Configuration

```java
ConfigManager configManager = plugin.getConfigManager();

// Check if feature is enabled
boolean enabled = configManager.isFeatureEnabled("verification");

// Get feature config
ConfigurationSection config = configManager.getFeatureConfig("verification");
```

### Extension API

#### Custom Modules

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
        return "Custom module description";
    }

    @Override
    protected void onEnable() {
        // Module initialization
    }

    @Override
    protected void onDisable() {
        // Cleanup
    }

    @Override
    public void onDiscordReady() {
        // Discord is ready, can use bot API
    }
}
```

#### Event Hooks

The plugin fires custom events that can be listened to:

```java
@EventHandler
public void onVerification(VerificationCompleteEvent event) {
    Player player = event.getPlayer();
    String discordId = event.getDiscordId();

    // Handle verification completion
}
```

### PlaceholderAPI Integration

```java
// In your plugin
PlaceholderExpansion expansion = new PlaceholderExpansion() {
    @Override
    public String onRequest(Player player, String identifier) {
        if (identifier.equals("player_verified")) {
            XDiscordUltimate plugin = XDiscordUltimate.getInstance();
            DatabaseManager db = plugin.getDatabaseManager();
            CompletableFuture<String> discordId = db.getDiscordId(player.getUniqueId());

            // Return "true" or "false"
            return discordId.join() != null ? "true" : "false";
        }
        return null;
    }

    @Override
    public String getIdentifier() {
        return "xdiscord";
    }

    @Override
    public String getAuthor() {
        return "YourName";
    }

    @Override
    public String getVersion() {
        return "1.0.0";
    }
};

expansion.register();
```

---

## Custom Module Development

### Step-by-Step Guide

#### 1. Create Module Class

```java
package com.xreatlabs.xdiscordultimate.modules.custom;

import com.xreatlabs.xdiscordultimate.XDiscordUltimate;
import com.xreatlabs.xdiscordultimate.modules.Module;

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
        return "A custom module for custom functionality";
    }

    @Override
    protected void onEnable() {
        info("Custom module enabled!");
        // Load configuration
        loadConfig();
    }

    @Override
    protected void onDisable() {
        info("Custom module disabled!");
        // Cleanup resources
    }

    @Override
    public void onDiscordReady() {
        info("Discord is ready! Starting custom features...");
        // Initialize Discord-dependent features
    }

    private void loadConfig() {
        // Load configuration options
        boolean enabled = getConfig().getBoolean("enabled", true);
        String setting = getConfig().getString("setting", "default");

        debug("Configuration loaded: " + enabled + ", " + setting);
    }
}
```

#### 2. Register Module

Edit `ModuleManager.java`:

```java
// In loadModules() method
registerModule("custom-module", new CustomModule(plugin));
```

#### 3. Add Configuration

Add to `src/main/resources/config.yml`:

```yaml
features:
  custom-module:
    enabled: true
    setting: "value"
    # Add more options as needed
```

#### 4. Add Module Documentation

Create `src/main/resources/modules/CUSTOM-MODULE.md`:

```markdown
# Custom Module

Description of custom module...

## Configuration

```yaml
features:
  custom-module:
    enabled: true
    setting: "value"
```
```

### Module Best Practices

#### 1. Configuration

**Good**:
```java
private String getSetting(String key, String defaultValue) {
    return getConfig().getString("setting-" + key, defaultValue);
}

private boolean isFeatureEnabled(String feature) {
    return getConfig().getBoolean("features." + feature + ".enabled", false);
}
```

**Avoid**:
```java
// Don't hardcode configuration
String setting = "hardcoded-value";
```

#### 2. Database Operations

**Good**:
```java
public CompletableFuture<Void> saveData(Data data) {
    return plugin.getDatabaseManager().executeAsync(() -> {
        String sql = "INSERT INTO custom_table (id, data) VALUES (?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, data.getId());
            stmt.setString(2, data.getValue());
            stmt.executeUpdate();
        }
        return null;
    });
}
```

**Avoid**:
```java
// Don't block main thread
public void saveData(Data data) {
    String sql = "INSERT INTO custom_table ...";
    Statement stmt = conn.createStatement();
    stmt.execute(sql); // This will freeze the server!
}
```

#### 3. Discord Integration

**Good**:
```java
@Override
public void onDiscordReady() {
    if (plugin.getDiscordManager() != null &&
        plugin.getDiscordManager().isReady()) {
        // Initialize Discord features
        setupDiscordListener();
    }
}
```

**Avoid**:
```java
@Override
protected void onEnable() {
    // Don't use Discord in onEnable()
    plugin.getDiscordManager().getJDA() // Will be null!
}
```

#### 4. Event Handling

**Good**:
```java
@EventHandler(ignoreCancelled = true, priority = EventPriority.NORMAL)
public void onPlayerJoin(PlayerJoinEvent event) {
    Player player = event.getPlayer();

    // Process asynchronously if needed
    plugin.getServer().getScheduler().runTaskAsynchronously(plugin, () -> {
        // Long-running operations
    });
}
```

**Avoid**:
```java
@EventHandler
public void onPlayerJoin(PlayerJoinEvent event) {
    // Don't do blocking operations in event handlers
    Thread.sleep(5000); // This will freeze the server!
}
```

#### 5. Resource Management

**Good**:
```java
@Override
protected void onDisable() {
    // Cleanup resources
    if (task != null) {
        task.cancel();
    }
    if (listener != null) {
        plugin.getDiscordManager().getJDA().removeEventListener(listener);
    }
}
```

**Avoid**:
```java
@Override
protected void onDisable() {
    // Don't forget to cleanup!
    // Listeners will leak
    // Tasks will keep running
}
```

---

## Testing Guidelines

### Unit Testing

Create test class in `src/test/java/`:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class ConfigManagerTest {

    @Test
    public void testFeatureEnabled() {
        // Create test configuration
        // Test feature enabled check
        // Assert expected results
    }

    @Test
    public void testDatabaseConnection() throws SQLException {
        // Test database connection
        // Test queries
        // Verify results
    }
}
```

### Running Tests

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests ConfigManagerTest

# Run with coverage
./gradlew test jacocoTestReport
```

### Integration Testing

For integration tests, create test server:

```java
@Test
public void testChatBridge() {
    // Set up mock Discord bot
    // Send test message
    // Verify message bridging
}
```

### Test Configuration

Add to `build.gradle`:

```gradle
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.0'
    testImplementation 'org.mockito:mockito-core:4.6.1'
}

tasks.withType(Test) {
    useJUnitPlatform()
}
```

---

## Debugging

### Enable Debug Mode

In `config.yml`:

```yaml
general:
  debug: true
```

This enables:
- Detailed database query logging
- Discord API interaction logging
- Module lifecycle logging
- Configuration loading logging

### Logging Best Practices

**Use appropriate log levels**:

```java
info("Module enabled successfully");  // Normal operations
warning("Optional dependency not found");  // Warnings
severe("Failed to connect to database");  // Errors
debug("Processing message: " + message);  // Debug only
```

**Include context**:

```java
// Good
error("Failed to send message to channel " + channelId + ": " + e.getMessage(), e);

// Bad
error("Error occurred");
```

### IntelliJ Debugging

**Set Breakpoints**:
1. Click in gutter next to line number
2. Red dot = line breakpoint
3. Green dot = method breakpoint

**Start Debug Session**:
1. Right-click on test class
2. Select "Debug 'ClassName'"
3. Or click bug icon

**Debug Console**:
- Inspect variables
- Evaluate expressions
- Step through code (F8, F7, Shift+F8)

### Common Debugging Scenarios

#### Discord Bot Not Connecting

```java
// Enable debug logging
info("Bot token: " + (token != null ? "present" : "null"));
info("Guild ID: " + guildId);
info("Intents: " + intents);

// Check JDA status
if (jda != null) {
    info("JDA Status: " + jda.getStatus());
    info("Guilds: " + jda.getGuilds().size());
}
```

#### Database Connection Issues

```java
// Test connection
try (Connection conn = dataSource.getConnection()) {
    info("Database connection successful");
    try (Statement stmt = conn.createStatement()) {
        ResultSet rs = stmt.executeQuery("SELECT 1");
        if (rs.next()) {
            info("Database query test: " + rs.getInt(1));
        }
    }
} catch (SQLException e) {
    severe("Database connection failed", e);
}
```

#### Module Not Loading

```java
info("Checking module: " + getName());
info("Is enabled in config: " + isFeatureEnabled(getName().toLowerCase().replace(" ", "-")));
try {
    onEnable();
    info("Module enabled successfully");
} catch (Exception e) {
    error("Failed to enable module", e);
}
```

---

## Code Style

### Java Code Style

#### Formatting

**Indentation**: Use 4 spaces (not tabs)

**Line Length**: Max 120 characters

**Braces**:
```java
// Opening brace on same line
public void method() {
    if (condition) {
        // Code
    }
}
```

**Line Spacing**:
- Blank line between methods
- Blank line between different logical sections
- No blank line after opening brace

#### Naming Conventions

**Classes**: PascalCase
```java
public class ChatBridgeModule { }
public class DatabaseManager { }
```

**Methods**: camelCase
```java
public void onEnable() { }
public CompletableFuture<Boolean> linkAccount() { }
```

**Variables**: camelCase
```java
private String botToken;
private boolean useWebhook;
```

**Constants**: UPPER_SNAKE_CASE
```java
private static final long TYPING_TIMEOUT = 10000;
private static final Pattern WEBHOOK_PATTERN = Pattern.compile("...");
```

**Packages**: lowercase
```java
com.xreatlabs.xdiscordultimate.modules
```

#### Commenting

**Javadoc Comments**:
```java
/**
 * Sends a message to a Discord channel.
 *
 * @param channel The text channel to send to
 * @param message The message content
 * @return CompletableFuture representing the async operation
 */
public CompletableFuture<Void> sendMessage(TextChannel channel, String message) {
    // Implementation
}
```

**Inline Comments**:
```java
// Good
// Check if webhook is enabled before sending
if (useWebhook) {
    sendWebhookMessage(message);
}

// Avoid
// This is a comment that states the obvious
if (true) {
    // doing something
}
```

**Method Comments**:
```java
// Calculate the time difference
long timeDiff = System.currentTimeMillis() - startTime;
```

#### Import Organization

**Order**:
1. Java standard library
2. Third-party libraries
3. Minecraft/Spigot
4. Plugin classes

**No Wildcards**:
```java
// Good
import java.util.List;
import java.util.ArrayList;

// Bad
import java.util.*;
```

#### Code Organization

**Field Ordering**:
1. Static constants
2. Instance variables
3. Constructors
4. Public methods
5. Private methods

**Method Ordering**:
- Public methods first
- Private methods after
- Helper methods at bottom

### XML/HTML Style

**YAML Configuration**:
```yaml
# Use 2 spaces for indentation
features:
  chat-bridge:
    enabled: true
    channel-id: "123456789"
```

**HTML in Embeds**:
```java
// Avoid HTML in descriptions
embedBuilder.appendDescription("Visit <https://example.com> for more info");

// Use markdown formatting
embedBuilder.appendDescription("**Bold text** and *italic text*");
```

---

## Release Process

### Version Numbering

The project uses [Semantic Versioning](https://semver.org/):

- **MAJOR.MINOR.PATCH**
- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

Examples:
- `1.0.0` - Initial release
- `1.1.0` - New features added
- `1.1.1` - Bug fixes
- `2.0.0` - Breaking changes

### Release Checklist

#### Pre-Release

1. **Update Version**:
   - Edit `build.gradle`: `version = 'x.y.z'`
   - Edit `plugin.yml`: `version: x.y.z`

2. **Update Changelog**:
   Create `CHANGELOG.md`:
   ```markdown
   ## [1.1.0] - 2024-01-15

   ### Added
   - New feature: Voice Channels
   - New command: /xdiscord reload

   ### Changed
   - Updated Discord API to JDA 5.0

   ### Fixed
   - Fixed issue with verification codes
   - Fixed memory leak in chat bridge
   ```

3. **Update README**:
   - Update version badges
   - Add any new features

4. **Run Tests**:
   ```bash
   ./gradlew test
   ```

5. **Build Release**:
   ```bash
   ./gradlew clean shadowJar
   ```

6. **Manual Testing**:
   - Test on test server
   - Verify all modules work
   - Check configuration migration

7. **Update Documentation**:
   - API docs
   - Module docs
   - Configuration guide

#### Release

1. **Create Git Tag**:
   ```bash
   git tag -a v1.1.0 -m "Release v1.1.0"
   git push origin v1.1.0
   ```

2. **Create GitHub Release**:
   - Go to GitHub → Releases
   - Click "Create new release"
   - Select tag
   - Upload JAR file
   - Write release notes
   - Publish

3. **Deploy**:
   - Upload to Discord webhook (automatic via Gradle task)
   - Update Spigot resource page (if applicable)

#### Post-Release

1. **Monitor Issues**:
   - Check GitHub Issues
   - Monitor Discord for problems

2. **Release Hotfixes**:
   - For critical bugs, release patch version (1.1.1)
   - Follow same process with expedited testing

3. **Update Documentation**:
   - Update version-specific docs
   - Migrate guides if needed

---

## Troubleshooting

### Build Issues

#### Gradle Download Fails

**Problem**: Gradle wrapper download fails

**Solution**:
```bash
# Delete corrupted wrapper
rm -rf ~/.gradle/wrapper/

# Redownload
./gradlew wrapper --gradle-version 7.6
```

#### Compilation Errors

**Problem**: "Cannot resolve symbol"

**Solution**:
1. Check Java version: `java -version`
2. Refresh Gradle project in IDE
3. Clean and rebuild: `./gradlew clean build`

**Problem**: "Unsupported class file version"

**Solution**:
- Update Java to version 17 or higher
- Check Gradle JDK configuration

#### Test Failures

**Problem**: Tests fail with "ClassNotFoundException"

**Solution**:
```gradle
// Add to build.gradle
test {
    classpath = sourceSets.main.runtimeClasspath
}
```

### Runtime Issues

#### Plugin Won't Load

**Check**:
1. Java version (need 17+)
2. Spigot version (need 1.16.5+)
3. Configuration syntax errors
4. Permission errors

**Solutions**:
```bash
# Check logs
tail -f server.log | grep -i "xdiscord"

# Verify JAR integrity
java -jar XDiscordUltimate-1.1.0.jar
```

#### Discord Bot Won't Connect

**Check**:
1. Bot token is correct
2. Bot is in server
3. Bot has necessary permissions
4. Guild ID is correct

**Debug**:
```java
// Add to DiscordManager.initialize()
plugin.getLogger().info("Bot token: " + (token != null ? "present" : "NULL"));
plugin.getLogger().info("Guild ID: " + guildId);
```

#### Database Connection Fails

**Check**:
1. Database server is running
2. Credentials are correct
3. Database exists
4. User has permissions

**Debug**:
```java
// Test connection manually
try {
    Connection conn = dataSource.getConnection();
    plugin.getLogger().info("Database connection successful!");
    conn.close();
} catch (SQLException e) {
    plugin.getLogger().severe("Database connection failed!", e);
}
```

### Development Issues

#### IntelliJ Won't Recognize Gradle Project

**Solution**:
1. Close IntelliJ
2. Delete `.idea` folder
3. Reopen IntelliJ
4. Open as Gradle project

#### Code Auto-Formatting Messes Up Imports

**Solution**:
1. File → Settings → Editor → Code Style → Java
2. Imports tab
3. Set "Class count to use import with '*'": 999
4. Set "Names count to use import with '*'": 999

#### Gradle Memory Issues

**Solution** (in `gradle.properties`):
```properties
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=512m
org.gradle.parallel=true
org.gradle.daemon=true
```

---

## Additional Resources

### Documentation
- [Spigot API](https://hub.spigotmc.org/javadocs/bukkit/)
- [JDA Documentation](https://jda.wiki/)
- [HikariCP Documentation](https://github.com/brettwooldridge/HikariCP)

### Tools
- [Gradle](https://gradle.org/)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)
- [Minecraft Version Converter](https://minecraft.tools/en/time.php)

### Communities
- [Spigot Forums](https://www.spigotmc.org/)
- [Discord Developer Portal](https://discord.com/developers/applications)
- [XDiscordUltimate Discord](https://discord.gg/xreatlabs)

---

This developer guide provides comprehensive information for developers working on XDiscordUltimate. Keep it updated as the project evolves!