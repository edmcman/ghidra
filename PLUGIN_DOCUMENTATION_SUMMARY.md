# Plugin Enablement Documentation - Summary

## Problem Statement

The original request asked for:
1. A report on how Plugins are enabled or disabled in tools
2. Whether there's a way to have a new extension's plugin be enabled in CodeBrowser automatically

## Solution Provided

This PR adds comprehensive documentation that fully addresses both questions.

### Documentation Added

#### 1. [PluginEnablementGuide.md](GhidraDocs/PluginEnablementGuide.md)
A comprehensive 400+ line guide covering:

**How Plugins Are Enabled or Disabled:**
- Plugin states (loaded/enabled vs. not loaded/disabled)
- Configuration persistence in `.tool` XML files
- Plugin loading process and lifecycle
- Plugin architecture components (PluginInfo, PluginManager, etc.)
- Service system and dependencies

**Extension Plugins and Auto-Enablement:**
- Current behavior: Detection and user prompting
- Implementation details from `GhidraTool.checkForNewExtensions()`
- Why automatic enablement is not supported (user control, stability, performance)
- The `KNOWN_EXTENSIONS` preference mechanism

**Manual Configuration:**
- Using the Configure Tool dialog
- Programmatic plugin management
- Direct tool template editing

**Best Practices for Extension Developers:**
- Plugin metadata and annotations
- Dependency declaration
- Plugin status levels
- Installation documentation
- Testing plugin loading

**Workarounds for Auto-Enablement:**
- Custom tool templates (recommended)
- Modified default tool templates (for controlled deployments)
- User documentation approach
- Programmatic configuration via scripts
- Suggestions for future enhancements

#### 2. [ExtensionPluginQuickStart.md](GhidraDocs/ExtensionPluginQuickStart.md)
A practical quick-start guide with:

**For Users:**
- Step-by-step instructions for enabling extension plugins
- Manual configuration process

**For Extension Developers:**
- Direct answer: "No, there is no automatic enablement mechanism"
- Explanation of what happens when extensions are installed
- Best practices with code examples
- Alternative approaches with examples
- Common questions and answers
- Complete example extension structure

#### 3. README.md Update
Added a new "Extensions and Plugins" section linking to the documentation.

## Key Findings

### Question 1: How are plugins enabled or disabled?

**Answer:** Plugins are enabled/disabled through:

1. **Manual UI Configuration:**
   - `File → Configure` menu opens the plugin management dialog
   - Users check/uncheck plugins to enable/disable them
   - Changes are saved to the tool's `.tool` XML file

2. **Tool Templates:**
   - Plugin configurations are stored in `.tool` XML files
   - These templates persist which plugins are enabled
   - Located in tool configuration directories

3. **Programmatic API:**
   - `tool.addPlugin(className)` - Add a plugin
   - `tool.removePlugins(list)` - Remove plugins
   - Available for scripting and headless mode

### Question 2: Can extension plugins be auto-enabled in CodeBrowser?

**Answer:** No, but there are workarounds.

**Why Not:**
- Ghidra prioritizes user control and tool stability
- Auto-enabling could load unstable code without user knowledge
- Plugins might have unmet dependencies
- Performance impact considerations

**What Happens Instead:**
1. When a tool launches, it detects new extensions
2. User is prompted: "New extension plugins detected. Would you like to configure them?"
3. If user clicks "Yes", a dialog shows available plugins to enable
4. User manually selects which plugins to enable

**Workarounds:**
1. **Distribute custom tool templates** (recommended)
   - Create a `.tool` file with plugins pre-configured
   - Users import it via `File → Import → Ghidra Tool...`

2. **Provide clear documentation**
   - Step-by-step instructions with screenshots
   - Video tutorials
   - Example commands

3. **Headless/scripted configuration**
   - For automated deployments
   - Use Ghidra's API to configure tools programmatically

4. **Modify default templates**
   - Only for controlled/enterprise deployments
   - Requires rebuilding Ghidra from source

## Implementation Details

The extension detection system works as follows:

1. **Preference Tracking:** Tools store a `KNOWN_EXTENSIONS` preference listing extensions they've seen
2. **Detection:** On launch, `GhidraTool.checkForNewExtensions()` compares installed extensions against known ones
3. **Plugin Discovery:** Uses `PluginUtils.findLoadedPlugins()` to find plugins in new extensions
4. **User Prompt:** Shows `PluginInstallerDialog` if new plugins are found
5. **Update Preference:** Adds newly seen extensions to `KNOWN_EXTENSIONS`

Source code references:
- `ghidra/framework/project/tool/GhidraTool.java:233-266`
- `ghidra/framework/plugintool/dialog/PluginInstallerDialog.java`
- `ghidra/framework/plugintool/PluginConfigurationModel.java`

## Usage

### For Users
Read the [Plugin Enablement Guide](GhidraDocs/PluginEnablementGuide.md) to understand how to manage plugins in your tools.

### For Extension Developers
Start with the [Extension Plugin Quick Start](GhidraDocs/ExtensionPluginQuickStart.md) for practical guidance on making your extension's plugins easy to enable.

## Future Enhancements

The documentation suggests potential future features:
- Plugin auto-enable flag in annotations
- Extension manifest files with default enablement preferences
- Post-install hooks for configuration
- Pre-configured plugin profiles

## Files Changed

- `README.md` - Added Extensions and Plugins section
- `GhidraDocs/PluginEnablementGuide.md` - New comprehensive guide
- `GhidraDocs/ExtensionPluginQuickStart.md` - New quick reference

## Testing

No code changes were made, only documentation. The documentation was:
- ✅ Reviewed for technical accuracy against source code
- ✅ Checked against existing Ghidra plugin system implementation
- ✅ Verified with code review tool
- ✅ Based on actual source code analysis

## Conclusion

This PR provides a complete answer to both questions in the problem statement:

1. ✅ **Documented how plugins are enabled/disabled** - Comprehensive coverage of UI, tool templates, and programmatic approaches
2. ✅ **Answered the auto-enable question** - Clear "no" with explanation and practical workarounds

The documentation is ready for users and extension developers to understand and work with Ghidra's plugin system effectively.
