# CRDP MCP Server

A Model Context Protocol (MCP) server that allows interacting with the CipherTrust RestFul Data Protection (CRDP) service.

## Overview

This MCP server enables AI applications and LLMs to securely protect and reveal sensitive data through the CipherTrust CRDP service. It supports both individual and bulk protect and reveal operations with comprehensive versioning support.

## Features

- **Data Protection**: Protect sensitive data using various CRDP policies
- **Data Revelation**: Securely reveal protected data with proper authorization
- **Bulk Operations**: Process multiple data items in single batch operations
- **Versioning Support**: Handle external, internal, and no-versioning scenarios
- **JWT Authentication**: Optional JWT bearer token support for secure operations
- **Monitoring**: Health checks and metrics collection
- **Multiple Transports**: Support for stdio and HTTP transports
- **Compliance Ready**: Built-in support for GDPR, HIPAA, PCI-DSS compliance

## Prerequisites

Before installing and running the CRDP MCP Server, ensure you have the following prerequisites installed:

- **Node.js** (v18 or higher)
- **npm** (comes with Node.js)
- **TypeScript** (installed globally)
- **CRDP Service** (running and accessible)

See [prerequisites](docs/prerequisites.md) for detailed installation instructions.

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/sanyambassi/thales-cdsp-crdp-mcp-server.git
cd thales-cdsp-crdp-mcp-server
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Build the Project

```bash
npm run build
```

### 4. Start the Server

#### For stdio transport (default):
```bash
npm start
```

#### For HTTP transport:
```bash
MCP_TRANSPORT=streamable-http npm start
```

#### For development with auto-reload:
```bash
npm run dev
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CRDP_SERVICE_URL` | CRDP service endpoint for protect/reveal operations | `http://localhost:8090` |
| `CRDP_PROBES_URL` | CRDP service endpoint for monitoring operations | `http://localhost:8090` |
| `MCP_TRANSPORT` | Transport type (`stdio` or `streamable-http`) | `stdio` |
| `MCP_PORT` | HTTP port (when using streamable-http) | `3000` |

### Example Configuration

```bash
export CRDP_SERVICE_URL="http://crdp-server:8090"
export CRDP_PROBES_URL="http://crdp-server:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"
```

## Available Tools

### Data Protection Tools

#### `protect_data`
Protect a single piece of sensitive data.

**Parameters:**
- `data` (required): The sensitive data to protect
- `protection_policy_name` (required): CRDP protection policy name
- `jwt` (optional): JWT token for authorization

**Example:**
```json
{
  "name": "protect_data",
  "arguments": {
    "data": "john.doe@example.com",
    "protection_policy_name": "email_policy",
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### `protect_bulk`
Protect multiple data items in a single batch operation.

**Parameters:**
- `request_data` (required): Array of protection request objects
- `jwt` (optional): JWT token for authorization

**Example:**
```json
{
  "name": "protect_bulk",
  "arguments": {
    "request_data": [
      {
        "protection_policy_name": "email_policy",
        "data": "john.doe@example.com"
      },
      {
        "protection_policy_name": "ssn_policy",
        "data": "123-45-6789"
      }
    ]
  }
}
```

### Data Revelation Tools

#### `reveal_data`
Reveal a single piece of protected data.

**Parameters:**
- `protected_data` (required): The protected data to reveal
- `username` (required): User identity for authorization
- `protection_policy_name` (required): Policy name used for protection
- `external_version` (optional): Version information for the protected data
- `jwt` (optional): JWT token for authorization

**Example:**
```json
{
  "name": "reveal_data",
  "arguments": {
    "protected_data": "enc_abc123def456",
    "username": "authorized_user",
    "protection_policy_name": "email_policy",
    "external_version": "1003000",
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### `reveal_bulk`
Reveal multiple protected data items in a single batch operation.

**Parameters:**
- `protected_data_array` (required): Array of reveal request objects
- `username` (required): User identity for authorization
- `jwt` (optional): JWT token for authorization

**Example:**
```json
{
  "name": "reveal_bulk",
  "arguments": {
    "username": "authorized_user",
    "protected_data_array": [
      {
        "protection_policy_name": "email_policy",
        "protected_data": "enc_abc123"
      },
      {
        "protection_policy_name": "ssn_policy",
        "protected_data": "enc_def456"
      }
    ],
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### Monitoring Tools

#### `get_metrics`
Get CRDP service metrics.

#### `check_health`
Check CRDP service health status.

#### `check_liveness`
Check CRDP service liveness.

## Versioning Support

The server supports three types of versioning scenarios:

### 1. External Versioning
Returns both protected data and external version:
```
Data protected successfully. Protected data: abcdefcLJTrU0Y8FKC
External version: 1003000
```

### 2. Internal Versioning
Returns protected data with embedded version:
```
Data protected successfully. Protected data: 1001000Y57IlQvok1Ke
```

### 3. No Versioning
Returns protected data only:
```
Data protected successfully. Protected data: BcmX5McZK6BB
```

## Testing

For comprehensive testing instructions, see [testing](docs/testing.md).

## n8n Integration

This project includes n8n workflow templates for creating conversational AI interfaces to the CRDP service:

### **n8n Templates**

- **`crdp_demo_mcp_server.json`**: MCP Server workflow that exposes CRDP tools
- **`crdp_demo_mcp_client.json`**: MCP Client workflow with conversational AI interface

### **Features**

- **Conversational Interface**: Protect and reveal data using natural language
- **JWT Authorization**: Secure operations with optional JWT bearer tokens
- **Conversational Memory**: Maintains context across chat sessions
- **Intelligent Tool Selection**: Automatically uses bulk operations for multiple data items
- **Strict Security**: Always requires proper authorization parameters

### **Quick Setup**

1. **Import Workflows**: Import both JSON files into your n8n instance
2. **Configure Credentials**: Add your OpenAI (or other LLM) credentials to the MCP Client
3. **Activate Workflows**: Enable both workflows
4. **Start Chatting**: Use the chat interface to interact with CRDP

For detailed n8n setup instructions, see [n8n/README.md](n8n/README.md).

### Quick Test

Test the server using curl:

```bash
# Test HTTP transport
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "test@example.com",
        "protection_policy_name": "email_policy"
      }
    }
  }'
```

## Development

### Project Structure

```
crdp-mcp-server/
├── src/
│   └── crdp-mcp-server.ts    # Main server implementation
├── dist/                     # Compiled JavaScript output
├── docs/                     # Documentation
├── n8n/                      # n8n workflow templates
├── package.json              # Project configuration
└── tsconfig.json            # TypeScript configuration
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `npm start` | Start the server (stdio transport) |
| `npm run dev` | Start development server with auto-reload |
| `npm run build` | Compile TypeScript to JavaScript |
| `npm run clean` | Clean the dist directory |

### Building from Source

```bash
# Clone the repository
git clone https://github.com/sanyambassi/thales-cdsp-crdp-mcp-server.git
cd thales-cdsp-crdp-mcp-server

# Install dependencies
npm install

# Build the project
npm run build

# Start the server
npm start
```

## Security Considerations

- All sensitive data is processed through the secure CRDP service
- User authorization is required for all reveal operations
- The server does not store sensitive data locally
- All communications should be over secure channels (HTTPS/WSS)
- Proper access controls should be implemented for the CRDP service

## Compliance

The server is designed to support various compliance frameworks:

- **GDPR**: Data minimization and right to be forgotten
- **HIPAA**: Protected health information safeguards
- **PCI-DSS**: Payment card data protection
- **CCPA/CPRA**: California consumer privacy requirements

## Troubleshooting

### Common Issues

1. **"tsc is not recognized"**: Install TypeScript globally with `npm install -g typescript`
2. **Connection refused**: Ensure CRDP service is running and accessible
3. **Authentication errors**: Verify CRDP service credentials and permissions

### Logs

The server outputs logs to stderr. Check for:
- CRDP service connection status
- Tool execution results
- Error messages and stack traces

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License (c) 2025 Thales Group. See the [LICENSE](LICENSE) file for details.

## Support

For issues and questions:
- Check the [troubleshooting section](#troubleshooting)
- Review the [testing documentation](docs/testing.md)
- Open an issue on GitHub

## Changelog

### v1.0.0
- Initial release
- Support for protect/reveal operations
- Bulk operations support
- Versioning support (external, internal, none)
- JWT authorization support
- HTTP and stdio transport support
- Monitoring tools
- Compliance framework support 