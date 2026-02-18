# Ghidra Plugin Enablement Guide

## Table of Contents
1. [Overview](#overview)
2. [How Plugins Are Enabled or Disabled](#how-plugins-are-enabled-or-disabled)
3. [Plugin Architecture](#plugin-architecture)
4. [Extension Plugins and Auto-Enablement](#extension-plugins-and-auto-enablement)
5. [Manually Configuring Plugins](#manually-configuring-plugins)
6. [Best Practices for Extension Developers](#best-practices-for-extension-developers)

---

## Overview

Ghidra uses a flexible plugin architecture that allows tools (like CodeBrowser) to be configured with different sets of plugins. This guide explains how plugins are enabled or disabled in Ghidra tools and describes the mechanisms available for automatically enabling plugins from new extensions.

---

## How Plugins Are Enabled or Disabled

### Plugin States

Plugins in Ghidra can be in one of two states relative to a tool:
- **Loaded (Enabled)**: The plugin is instantiated and active in the tool
- **Not Loaded (Disabled)**: The plugin is available but not currently active

### Configuration Persistence

Plugin configurations are persisted in tool template files (`.tool` files) stored as XML. These files contain:
- The list of enabled plugins
- Individual plugin configuration state
- Tool layout and window positions
- Known extensions

Example location: `Ghidra/Configurations/Public_Release/src/main/resources/defaultTools/CodeBrowser.tool`

### Plugin Loading Process

When a tool starts, the following happens:

1. **Tool Template Loading**: The tool reads its configuration from the `.tool` XML file
2. **Plugin Discovery**: The `PluginClassManager` discovers all available plugin classes marked with `@PluginInfo` annotations
3. **Plugin Instantiation**: Only plugins listed in the tool configuration are instantiated
4. **Dependency Resolution**: The `PluginManager` ensures all required services are available
5. **Initialization**: Plugins are initialized in dependency order via their `init()` method

---

## Plugin Architecture

### Plugin Base Class

All plugins must extend the `Plugin` abstract base class:

```java
@PluginInfo(
    status = PluginStatus.RELEASED,
    packageName = CorePluginPackage.NAME,
    category = PluginCategoryNames.CODE_VIEWER,
    shortDescription = "Brief description",
    description = "Detailed description",
    servicesRequired = { SomeService.class },
    servicesProvided = { MyService.class }
)
public class MyPlugin extends Plugin {
    public MyPlugin(PluginTool tool) {
        super(tool);
    }
}
```

### Key Components

#### 1. PluginInfo Annotation
Provides metadata about the plugin:
- `status`: Plugin lifecycle stage (RELEASED, STABLE, UNSTABLE, HIDDEN)
- `packageName`: Plugin category/package
- `category`: UI category for organization
- `description`: User-facing description
- `servicesRequired`: Services this plugin depends on
- `servicesProvided`: Services this plugin provides to others
- `eventsConsumed`/`eventsProduced`: Plugin event types

#### 2. PluginManager
Manages the plugin lifecycle:
- Constructs plugins
- Registers services
- Resolves dependencies
- Initializes plugins
- Handles plugin disposal

#### 3. PluginConfigurationModel
Tracks plugin state:
- Loaded vs. available plugins
- Plugin packages and dependencies
- Plugin stability (stable vs. unstable)

---

## Extension Plugins and Auto-Enablement

### Current Behavior

When a new extension is installed, Ghidra **does not automatically enable its plugins** in existing tools. Instead:

1. **Extension Detection**: When a tool launches, it checks for newly installed extensions by comparing the `KNOWN_EXTENSIONS` preference against currently installed extensions
   
2. **User Notification**: If new extensions are detected, the user is prompted:
   ```
   "New extension plugins detected. Would you like to configure them?"
   ```

3. **Manual Configuration**: If the user clicks "Yes", the `PluginInstallerDialog` opens, showing the new plugins available for installation

4. **Preference Update**: After the dialog is dismissed, the tool's `KNOWN_EXTENSIONS` preference is updated to include the new extensions

### Implementation Details

The detection and notification process is handled in `GhidraTool.checkForNewExtensions()`:

```java
public void checkForNewExtensions() {
    // 1. Remove any extensions that are no longer installed
    removeUninstalledExtensions();
    
    // 2. Find newly installed extensions
    Set<ExtensionDetails> newExtensions = 
        ExtensionUtils.getExtensionsInstalledSinceLastToolLaunch(this);
    
    // 3. Get plugins from those extensions
    List<Class<?>> newPlugins = PluginUtils.findLoadedPlugins(newExtensions);
    if (newPlugins.isEmpty()) {
        return;
    }
    
    // 4. Notify user
    int option = OptionDialog.showYesNoDialog(getActiveWindow(), 
        "New Plugins Found!",
        "New extension plugins detected. Would you like to configure them?");
    
    if (option == OptionDialog.YES_OPTION) {
        // Show plugin installer dialog
        List<PluginDescription> pluginDescriptions = 
            PluginUtils.getPluginDescriptions(this, newPlugins);
        PluginInstallerDialog pluginInstaller = 
            new PluginInstallerDialog("New Plugins Found!", this, 
                new PluginConfigurationModel(this), pluginDescriptions);
        showDialog(pluginInstaller);
    }
    
    // 5. Update known extensions
    addInstalledExtensions(newExtensions);
}
```

### Why Plugins Are Not Auto-Enabled

Ghidra does not automatically enable extension plugins for several important reasons:

1. **User Control**: Users should explicitly choose which plugins to enable
2. **Stability**: Extensions may contain unstable plugins that could affect tool stability
3. **Performance**: Loading unnecessary plugins can impact performance
4. **Dependencies**: Plugins may have service dependencies that aren't met
5. **Configuration**: Some plugins require configuration before use

---

## Manually Configuring Plugins

### Using the Configure Tool Dialog

1. **Open the Tool**: Launch the tool (e.g., CodeBrowser)
2. **Access Configuration**: Navigate to `File → Configure`
3. **Select Plugins**: 
   - The dialog shows all available plugins organized by package
   - Check boxes to enable plugins
   - Uncheck boxes to disable plugins
4. **Dependencies**: The UI will warn if you try to disable a plugin that others depend on
5. **Apply Changes**: Click "OK" to apply the configuration

### Managing Plugins Programmatically

Tools can be configured programmatically through the plugin API:

```java
// Add a plugin
tool.addPlugin("com.example.MyPlugin");

// Remove a plugin
tool.removePlugins(Collections.singletonList(myPlugin));

// Get current plugins
List<Plugin> plugins = tool.getManagedPlugins();
```

### Editing Tool Templates Directly

Advanced users can manually edit `.tool` XML files to configure plugins:

```xml
<TOOL>
  <TOOL_NAME>My Tool</TOOL_NAME>
  <PLUGINS>
    <PLUGIN>
      <NAME>com.example.MyPlugin</NAME>
    </PLUGIN>
  </PLUGINS>
  <PREFERENCES>
    <PREFERENCE_STATE NAME="KNOWN_EXTENSIONS">
      <ARRAY NAME="KNOWN_EXTENSIONS" TYPE="string">
        <STRING>MyExtension</STRING>
      </ARRAY>
    </PREFERENCE_STATE>
  </PREFERENCES>
</TOOL>
```

---

## Best Practices for Extension Developers

### 1. Provide Clear Plugin Metadata

Use descriptive `@PluginInfo` annotations:

```java
@PluginInfo(
    status = PluginStatus.RELEASED,  // Use appropriate status
    packageName = "MyExtension",      // Use a unique package name
    category = PluginCategoryNames.COMMON,
    shortDescription = "Brief, clear description",
    description = "Detailed description of what this plugin does " +
                  "and when users should enable it"
)
```

### 2. Declare Dependencies Explicitly

Always declare service dependencies:

```java
@PluginInfo(
    servicesRequired = { 
        ProgramManager.class,
        GoToService.class 
    },
    servicesProvided = { 
        MyCustomService.class 
    }
)
```

### 3. Choose Appropriate Plugin Status

- **RELEASED**: Production-ready, stable plugins
- **STABLE**: Well-tested but may have minor issues
- **UNSTABLE**: Experimental or in-development plugins
- **HIDDEN**: Internal plugins not meant for direct user access

### 4. Provide Installation Documentation

Include documentation with your extension explaining:
- What plugins are included
- Which plugins users should enable for specific workflows
- Any configuration steps required after enabling plugins
- Dependencies on other extensions or services

### 5. Consider Default Tool Templates

For extensions that should be enabled by default in specific tools:

1. **Document the Process**: Provide clear instructions for users to manually add your plugins
2. **Provide Tool Templates**: Distribute custom `.tool` files that include your plugins pre-configured
3. **Create Installation Scripts**: Provide scripts that help users configure their tools

### 6. Test Plugin Loading

Ensure your plugin:
- Initializes properly when loaded
- Handles missing dependencies gracefully
- Disposes of resources properly when unloaded
- Works correctly when loaded/unloaded multiple times

---

## Enabling Extension Plugins in CodeBrowser Automatically

### Current Limitations

There is **no built-in mechanism** to automatically enable extension plugins in CodeBrowser (or any other tool) without user intervention. This is by design to maintain user control and tool stability.

### Workarounds and Alternatives

#### Option 1: Provide Custom Tool Template

Create a custom tool template that includes your extension's plugins:

1. Configure CodeBrowser with your plugins enabled
2. Export the tool configuration (`File → Export → Tool...`)
3. Distribute this `.tool` file with your extension
4. Instruct users to import the tool template (`File → Import → Ghidra Tool...`)

#### Option 2: Modify Default Tool Template

For deployment scenarios where you control the Ghidra installation:

1. Locate the default tool template:
   ```
   Ghidra/Configurations/Public_Release/src/main/resources/defaultTools/CodeBrowser.tool
   ```
2. Add your plugin entries to the `<PLUGINS>` section
3. Add your extension to the `KNOWN_EXTENSIONS` preference
4. Rebuild and redistribute Ghidra

**Note**: This approach requires rebuilding Ghidra and is not suitable for distributing standalone extensions.

#### Option 3: Provide User Documentation

The simplest and most maintainable approach:

1. Include clear installation instructions with your extension
2. Document which plugins to enable and why
3. Provide screenshots or step-by-step guides
4. Consider creating a video tutorial

Example documentation:

```markdown
## Enabling MyExtension Plugins

After installing MyExtension, follow these steps to enable the plugins:

1. Launch CodeBrowser
2. When prompted about new plugins, click "Yes"
3. In the plugin dialog, enable the following plugins:
   - MyMainPlugin: Core functionality
   - MyHelperPlugin: Additional features
4. Click "OK" to save the configuration
5. Restart CodeBrowser if prompted

Alternatively, you can enable plugins manually:
1. Go to File → Configure
2. Find "MyExtension" in the plugin list
3. Check the boxes for the plugins you want to enable
4. Click "OK"
```

#### Option 4: Programmatic Configuration (Advanced)

For scripted deployments or testing, you can use Ghidra's headless mode or scripting API:

```java
// In a Ghidra script or headless analyzer
import ghidra.framework.plugintool.PluginTool;

// Get or create a tool
PluginTool tool = // ... obtain tool reference

// Add your extension's plugins
tool.addPlugin("com.mycompany.MyMainPlugin");
tool.addPlugin("com.mycompany.MyHelperPlugin");

// Save the tool configuration
tool.saveTool();
```

### Future Enhancements

Consider requesting the following features in future Ghidra versions:

1. **Plugin Auto-Enable Flag**: An annotation or configuration option to mark plugins as "auto-enable in specific tools"
2. **Extension Manifest**: A manifest file in extensions that specifies default plugin enablement preferences
3. **Post-Install Hooks**: Scripts that run after extension installation to configure tools
4. **Plugin Profiles**: Pre-configured plugin sets that users can apply with one click

---

## Summary

### Key Points

1. **Manual Control**: Plugins are enabled/disabled manually through the Configure Tool dialog
2. **Detection**: New extension plugins are detected and users are prompted to configure them
3. **No Auto-Enable**: Extensions cannot automatically enable their plugins in existing tools
4. **Persistence**: Plugin configurations are saved in tool template XML files
5. **Dependencies**: The plugin system enforces service dependencies

### Recommendations

**For Users:**
- Review and enable new extension plugins when prompted
- Use `File → Configure` to manage plugins
- Create custom tool templates for different workflows

**For Extension Developers:**
- Provide clear documentation for plugin enablement
- Use appropriate plugin status levels
- Declare all dependencies explicitly
- Consider distributing custom tool templates
- Test plugin loading/unloading thoroughly

**For Organizations:**
- Consider creating standardized tool templates
- Document plugin configuration policies
- Use headless/scripted approaches for deployment
- Maintain tool templates in version control

---

## Additional Resources

- **Plugin Development**: See the Ghidra Plugin API documentation
- **Extension Development**: Refer to the Extension Development guide
- **Source Code**: 
  - `ghidra/framework/project/tool/GhidraTool.java`
  - `ghidra/framework/plugintool/PluginManager.java`
  - `ghidra/framework/plugintool/dialog/PluginInstallerDialog.java`
  - `ghidra/framework/plugintool/PluginConfigurationModel.java`

---

*Last Updated: February 2026*
