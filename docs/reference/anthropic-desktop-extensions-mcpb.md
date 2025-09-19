# Anthropic Desktop Extensions (MCPB) Documentation

**Source URL:** https://www.anthropic.com/engineering/desktop-extensions
**Fetch Date:** 2025-09-19
**Description:** Complete documentation for Anthropic's Desktop Extensions (.mcpb files) - a packaging format for Model Context Protocol (MCP) servers

## Overview

Desktop Extensions (.mcpb files) are a packaging format that simplifies installing Model Context Protocol (MCP) servers for Claude Desktop, making powerful local AI tools accessible to non-technical users.

## Key Features

- **Single-click installation** of MCP servers
- **Bundled dependencies** - all required packages included
- **Cross-platform support** - works across different operating systems
- **Secure configuration management** - sensitive data stored securely
- **Automatic updates** - seamless update process
- **Enterprise-friendly controls** - management and security features for organizations

## Technical Architecture

### Extension Structure

A typical Desktop Extension (.mcpb) contains:

- `manifest.json` - Metadata and configuration
- `server/` - MCP server implementation files
- `dependencies/` - Required packages/libraries
- Optional `icon.png` - Extension icon

### Manifest Configuration

The `manifest.json` is the core configuration file with key sections:

- **Metadata** - name, version, description
- **Server configuration** - how to run the MCP server
- **User configuration options** - customizable settings
- **Platform compatibility** - supported operating systems
- **Tools and prompt declarations** - available functionality

## Example Manifest

```json
{
  "mcpb_version": "0.1",
  "name": "my-extension",
  "version": "1.0.0",
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"]
    }
  }
}
```

## Development Process

1. **Initialize manifest**
   ```bash
   npx @anthropic-ai/mcpb init
   ```

2. **Configure user inputs** - Set up any required configuration options

3. **Package extension**
   ```bash
   npx @anthropic-ai/mcpb pack
   ```

4. **Test locally** - Drag .mcpb file into Claude Desktop for testing

## Security Considerations

- **Sensitive data protection** - Stored in OS keychain for security
- **Enterprise controls** - Management features for organizational deployment
- **Group Policy and MDM support** - Integration with enterprise management systems
- **Extension blocklisting** - Ability to prevent specific extensions from running

## Open Source Commitment

Anthropic is open-sourcing the complete Desktop Extensions ecosystem:

- **Complete MCPB specification** - Full technical specification
- **Packaging tools** - Development and build tools
- **Reference implementation code** - Example implementations
- **TypeScript types and schemas** - Type definitions for development

## Future Vision

The specification is currently versioned 0.1, indicating:

- **Ongoing development** - Active specification evolution
- **Community collaboration** - Open to community input and contributions
- **Iterative improvement** - Regular updates and enhancements

## Installation and Usage

Desktop Extensions can be installed by:

1. **Direct installation** - Drag .mcpb file into Claude Desktop
2. **Enterprise deployment** - Through organizational management systems
3. **Future app store** - Planned centralized distribution (coming soon)

## Benefits for Users

- **Simplified installation** - No technical setup required
- **Bundled dependencies** - Everything included in one package
- **Secure configuration** - Sensitive data handled safely
- **Easy updates** - Automatic update mechanism
- **Cross-platform** - Works on all supported desktop platforms

## Benefits for Developers

- **Simplified distribution** - Single file packaging
- **Dependency management** - Bundled dependencies eliminate version conflicts
- **User-friendly** - Non-technical users can easily install
- **Enterprise-ready** - Built-in enterprise management features
- **Open ecosystem** - Open source tools and specifications