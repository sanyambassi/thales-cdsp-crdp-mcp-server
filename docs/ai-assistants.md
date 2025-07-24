# AI Assistant Integration Guide

This guide shows how to integrate the CRDP MCP Server with various AI assistants to enable secure data protection and revelation capabilities.

## Prerequisites

Before integrating with AI assistants, ensure:

1. **CRDP MCP Server is built and ready**:
   ```bash
   npm install
   npm run build
   ```

2. **CRDP service is running and accessible**

3. **You have the AI assistant installed and configured**

## Supported AI Assistants

### Cursor AI

Cursor AI supports MCP servers through a configuration file.

#### Setup

1. **Create or update your `mcp.json` file** (mcp.json):

```json
{
  "mcpServers": {
    "crdp_mcp_server": {
      "command": "node",
      "args": ["/path/to/your/crdp-mcp-server/dist/crdp-mcp-server.js"],
      "env": {
        "CRDP_SERVICE_URL": "http://your-crdp-server:8090",
        "CRDP_PROBES_URL": "http://your-crdp-server:8080",
        "MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

#### Configuration Details

- **`command`**: Use `node` to run the compiled JavaScript
- **`args`**: Path to your compiled `crdp-mcp-server.js` file
- **`env`**: Environment variables for CRDP service configuration
  - `CRDP_SERVICE_URL`: Your CRDP service endpoint for protect/reveal operations
  - `CRDP_PROBES_URL`: Your CRDP service endpoint for monitoring operations
  - `MCP_TRANSPORT`: Set to `stdio` for AI assistant integration

#### Example Paths

**Windows:**
```json
"args": ["C:\\Users\\YourUsername\\path\\to\\crdp-mcp-server\\dist\\crdp-mcp-server.js"]
```

**macOS/Linux:**
```json
"args": ["/home/username/path/to/crdp-mcp-server/dist/crdp-mcp-server.js"]
```

#### Usage

After configuration, restart Cursor AI. You can then use commands like:

- "Protect my email address john.doe@example.com using the email_policy"
- "Reveal the protected data abc123def456 for user admin"
- "Check the health of my CRDP service"

### Google Gemini

Google Gemini supports MCP servers using the same configuration format as Cursor AI.

#### Setup

1. **Create or update your `settings.json` file** (settings.json):

```json
{
  "mcpServers": {
    "crdp": {
      "command": "node",
      "args": ["/path/to/your/crdp-mcp-server/dist/crdp-mcp-server.js"],
      "env": {
        "CRDP_SERVICE_URL": "http://your-crdp-server:8090",
        "CRDP_PROBES_URL": "http://your-crdp-server:8080",
        "MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

#### Usage

After configuration, restart Google Gemini. You can then use commands like:

- "Protect my email address john.doe@example.com using the email_policy"
- "Reveal the protected data abc123def456 for user admin"
- "Check the health of my CRDP service"

### Claude Desktop

Claude Desktop supports MCP servers using the same configuration format as Cursor AI.

#### Setup

1. **Create or update your `mcp.json` file**:

```json
{
  "mcpServers": {
    "crdp": {
      "command": "node",
      "args": ["/path/to/your/crdp-mcp-server/dist/crdp-mcp-server.js"],
      "env": {
        "CRDP_SERVICE_URL": "http://your-crdp-server:8090",
        "CRDP_PROBES_URL": "http://your-crdp-server:8080",
        "MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

#### Usage

After configuration, restart Claude Desktop. You can then use commands like:

- "Protect my email address john.doe@example.com using the email_policy"
- "Reveal the protected data abc123def for user admin using protection policy ssn_policy"
- "Check the health of my CRDP service"

## Common Configuration Options

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `CRDP_SERVICE_URL` | CRDP service endpoint for protect/reveal operations | `http://10.0.0.168:8090` |
| `CRDP_PROBES_URL` | CRDP service endpoint for monitoring operations | `http://10.0.0.168:8080` |
| `MCP_TRANSPORT` | Transport type (use `stdio` for AI assistants) | `stdio` |

### JWT Authentication

If your CRDP service requires JWT authentication, you can include it in your prompts:

```
Protect my email john.doe@company.com using PPol1 policy. 
JWT: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Troubleshooting

### Common Issues

1. **"Command not found"**
   - Ensure Node.js is installed and accessible in your PATH
   - Verify the path to `crdp-mcp-server.js` is correct

2. **"Connection refused"**
   - Verify your CRDP service is running and accessible
   - Check the URLs in your environment variables

3. **"Permission denied"**
   - Ensure the script has execute permissions
   - Check file ownership and permissions

4. **AI assistant doesn't recognize the tools**
   - Restart your AI assistant after configuration changes
   - Verify the MCP server is starting correctly
   - Check the assistant's logs for connection errors

### Verification

To verify your setup is working:

1. **Test the MCP server directly**:
   ```bash
   node dist/crdp-mcp-server.js
   ```

2. **Check tool availability** in your AI assistant:
   - Try asking for available tools
   - Test a simple protect operation

3. **Monitor logs** for any error messages

## Security Considerations

- **Network Security**: Use HTTPS for production environments
- **Access Control**: Ensure only authorized users can access the CRDP service

## Examples

### Basic Protection
```
User: "Protect my email address test@example.com using the email_policy"
Assistant: "I'll protect your email address using the email_policy. Let me do that for you..."
```

### Protection with JWT
```
User: "Reveal my SSN 123-45-6789 using ssn_policy. JWT: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... and username is authorized_user"
Assistant: "I'll protect your SSN using the ssn_policy with JWT authentication..."
```

### Data Revelation
```
User: "Reveal the protected data abc123def456 for user admin using email_policy"
Assistant: "I'll reveal the protected data for user admin using the email_policy..."
```

### Health Check
```
User: "Check the health of my CRDP service"
Assistant: "I'll check the health status of your CRDP service..."
```

## Getting Help

If you encounter issues:

1. Check the [main troubleshooting guide](../README.md#troubleshooting)
2. Review the [testing documentation](testing.md)
3. Verify your CRDP service is working independently
4. Check your AI assistant's documentation for MCP support
5. Open an issue on the project's GitHub repository

## Contributing

If you've successfully integrated with other AI assistants, please contribute your configuration examples to this documentation. 