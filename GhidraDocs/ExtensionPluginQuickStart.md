# Extension Plugin Quick Start Guide

## For Users: Enabling Extension Plugins

### When You Install a New Extension

1. **Launch your tool** (e.g., CodeBrowser)
2. **Watch for the prompt**: "New extension plugins detected. Would you like to configure them?"
3. **Click "Yes"** to open the plugin installer
4. **Select the plugins** you want to enable
5. **Click "OK"** to save

### Manual Plugin Configuration

1. Open your tool (CodeBrowser, etc.)
2. Navigate to `File → Configure`
3. Browse the plugin list to find your extension's plugins
4. Check the boxes for plugins you want to enable
5. Click "OK" to apply changes
6. Restart the tool if prompted

---

## For Extension Developers: Auto-Enabling Plugins

### The Short Answer

**No**, there is currently no way to automatically enable extension plugins in CodeBrowser or other tools without user intervention. This is intentional to give users control over their tool configuration.

### What Happens When You Install an Extension

1. Extension files are copied to the Ghidra installation
2. Ghidra must be **restarted** for the extension classes to be loaded
3. When a tool launches, it detects new extensions
4. The user is **prompted** to configure the new plugins
5. The user must **manually select** which plugins to enable

### Best Practices for Your Extension

#### 1. Provide Clear Documentation

Create an `README.md` in your extension with:

```markdown
## Installation

1. Install this extension via File → Install Extensions
2. Restart Ghidra
3. Launch CodeBrowser
4. When prompted, enable these plugins:
   - **MyMainPlugin**: Core functionality
   - **MyHelperPlugin**: Optional helper features
```

#### 2. Use Descriptive Plugin Metadata

```java
@PluginInfo(
    status = PluginStatus.RELEASED,
    packageName = "MyExtension",
    category = PluginCategoryNames.COMMON,
    shortDescription = "Clear one-line description",
    description = "Detailed description explaining:\n" +
                  " - What this plugin does\n" +
                  " - When users should enable it\n" +
                  " - Any prerequisites or dependencies"
)
```

#### 3. Distribute a Pre-Configured Tool Template

Create a tool template with your plugins already enabled:

1. Configure CodeBrowser with your plugins enabled
2. Export: `File → Export → Tool...`
3. Include the `.tool` file with your extension
4. Document how users can import it: `File → Import → Ghidra Tool...`

### Alternative Approaches

#### Custom Tool Template (Recommended)

```
MyExtension/
├── extension.properties
├── lib/
│   └── MyExtension.jar
└── tools/
    └── CodeBrowserWithMyExtension.tool  ← Pre-configured tool
```

**Instruct users:**
"After installation, import the custom tool via File → Import → Ghidra Tool, then select `tools/CodeBrowserWithMyExtension.tool`"

#### Headless Configuration Script

For automated deployments:

```java
// Script to configure a tool programmatically
import ghidra.framework.model.*;
import ghidra.framework.plugintool.*;

// This requires access to the tool via scripting or headless mode
PluginTool tool = ... // obtain tool reference
tool.addPlugin("com.example.MyPlugin");
tool.saveTool();
```

#### Documentation-First Approach (Simplest)

Focus on making enablement easy:

1. **Clear screenshots** in your README
2. **Step-by-step video** (1-2 minutes)
3. **One-click copy-paste** commands where applicable
4. **FAQ section** addressing common questions

---

## Common Questions

### Q: Why can't my extension auto-enable its plugins?

**A:** Ghidra prioritizes user control and stability. Auto-enabling plugins could:
- Enable unstable or experimental code without user knowledge
- Impact tool performance unexpectedly
- Load plugins with unmet dependencies
- Violate user preferences or security policies

### Q: Can I modify the default CodeBrowser tool template?

**A:** Yes, but this requires rebuilding Ghidra from source:

1. Edit `Ghidra/Configurations/Public_Release/src/main/resources/defaultTools/CodeBrowser.tool`
2. Add your plugin entries to the XML
3. Rebuild Ghidra: `gradle buildGhidra`

This is **not recommended** for distributing extensions to other users.

### Q: Will users have to re-enable my plugins after each Ghidra update?

**A:** No. Once enabled in a tool, plugin configurations persist across Ghidra updates as long as:
- The tool configuration is saved
- The extension remains installed
- The plugin class names don't change

### Q: Can I use a post-install hook or script?

**A:** Not directly. Ghidra doesn't support post-install hooks for extensions. However, you can:
- Provide a Ghidra script users can run after installation
- Create a custom tool template users can import
- Document the manual enablement process clearly

### Q: What about enterprise/corporate deployments?

**A:** For controlled deployments:

1. **Pre-configure tool templates** with required plugins enabled
2. **Distribute configured tools** through your deployment system
3. **Use headless mode** to programmatically configure tools
4. **Version control** your tool configurations
5. **Document policies** for plugin management

---

## Quick Reference: Plugin States

| Status | Meaning | When to Use |
|--------|---------|-------------|
| `RELEASED` | Production-ready | Stable, fully-tested plugins |
| `STABLE` | Well-tested | Mostly stable, minor issues possible |
| `UNSTABLE` | Experimental | Development, testing, experimental features |
| `HIDDEN` | Internal only | Infrastructure plugins, not for direct use |

---

## Example: Complete Extension Setup

### Minimal Extension Structure

```
MyExtension/
├── extension.properties
├── README.md
├── lib/
│   └── MyExtension.jar
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── MyPlugin.java
```

### extension.properties

```properties
name=MyExtension
description=Provides awesome features
author=Your Name
version=1.0.0
```

### MyPlugin.java

```java
package com.example;

import ghidra.app.plugin.PluginCategoryNames;
import ghidra.app.plugin.core.Plugin;
import ghidra.framework.plugintool.PluginInfo;
import ghidra.framework.plugintool.PluginTool;
import ghidra.framework.plugintool.util.PluginStatus;

@PluginInfo(
    status = PluginStatus.RELEASED,
    packageName = "MyExtension",
    category = PluginCategoryNames.COMMON,
    shortDescription = "My awesome plugin",
    description = "This plugin provides awesome features. " +
                  "Enable it to access X, Y, and Z functionality."
)
public class MyPlugin extends Plugin {
    
    public MyPlugin(PluginTool tool) {
        super(tool);
    }
    
    @Override
    protected void init() {
        super.init();
        // Initialize your plugin
    }
    
    @Override
    protected void dispose() {
        // Clean up resources
        super.dispose();
    }
}
```

### README.md

```markdown
# MyExtension for Ghidra

## Installation

1. Download MyExtension.zip
2. In Ghidra: File → Install Extensions
3. Select the MyExtension.zip file
4. Restart Ghidra

## Enabling Plugins

After installation and restart:

1. Launch CodeBrowser
2. You'll see: "New extension plugins detected. Would you like to configure them?"
3. Click "Yes"
4. Enable "MyPlugin" in the dialog
5. Click "OK"

Or enable manually:
1. File → Configure
2. Find "MyExtension" in the list
3. Check "MyPlugin"
4. Click "OK"

## Usage

After enabling MyPlugin:
- Access features via: Tools → MyExtension → ...
- Right-click context menu includes new options
- etc.
```

---

## See Also

- [Full Plugin Enablement Guide](PluginEnablementGuide.md) - Complete documentation
- Ghidra Plugin Development - API documentation
- Extension Development Guide - Building extensions

---

*This guide provides practical quick-start information. For comprehensive details, see the [Plugin Enablement Guide](PluginEnablementGuide.md).*
