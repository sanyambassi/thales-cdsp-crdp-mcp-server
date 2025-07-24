# Testing Guide

This guide provides instructions for testing the CRDP MCP Server.

## Prerequisites

Before testing, ensure:

1. **Server is built and ready**: `npm install && npm run build`
2. **CRDP service is accessible**: Test with `curl -X GET http://your-crdp-server:8090/healthz`
3. **Valid protection policies** are configured in your CRDP service

## Manual Testing Methods

### 1. JSON-RPC over HTTP

#### Start the Server in HTTP Mode

```bash
export CRDP_SERVICE_URL="http://your-crdp-server:8090"
export CRDP_PROBES_URL="http://your-crdp-server:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"
npm start
```

#### Test Individual Tools

**Test protect_data**:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "john.doe@example.com",
        "protection_policy_name": "email_policy"
      }
    }
  }'
```

**Test reveal_data**:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "reveal_data",
      "arguments": {
        "protected_data": "enc_abc123def456",
        "username": "authorized_user",
        "protection_policy_name": "email_policy"
      }
    }
  }'
```

**Test JWT Authorization**:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "test@example.com",
        "protection_policy_name": "email_policy",
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
      }
    }
  }'
```

**Test get_metrics**:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0", 
    "id": 4, 
    "method": "tools/call", 
    "params": {
      "name": "get_metrics",
      "arguments": {}
    }
  }'
```

### 2. JSON-RPC over stdio

Start the server in stdio mode (default):

```bash
npm start
```

In a separate process or script, communicate using stdin/stdout:

**Initialization (Client -> Server)**:
```json
{"jsonrpc":"2.0","id":1,"method":"server/initialize","params":{"client_info":{"name":"TestClient","version":"1.0.0"}}}
```

**Initialized Notification (Server -> Client)**:
```json
{"jsonrpc":"2.0","method":"server/initialized","params":{"server_info":{"name":"crdp-mcp-server","version":"1.0.0","protocol_version":"2025-06-18"}}}
```

**Get Tools List (Client -> Server)**:
```json
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
```

**Execute Health Check (Client -> Server)**:
```json
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"check_health","arguments":{}}}
```

### 3. Automated Testing

Use the provided test script:

```bash
node scripts/test-server.js
```

This script:
1. Checks CRDP server health
2. Optionally tests protect/reveal operations
3. Provides interactive prompts for testing

## Testing Different Scenarios

### 1. Versioning Testing

Test protection policies with different versioning configurations:

- **External Versioning**: Observe `external_version` in response
- **Internal Versioning**: Version embedded in `protected_data`
- **No Versioning**: Simple `protected_data` response

### 2. Authentication Testing

- **Username-based**: Use `username` parameter for reveal operations
- **JWT-based**: Use `jwt` parameter (required if CRDP has JWT verification)
- **Both**: Provide both parameters for compatibility

## Common Errors

| Error | Possible Cause | Solution |
|-------|----------------|----------|
| Connection refused | CRDP container not running | Start CRDP container |
| 404 Not Found | Invalid protection policy | Check policy name |
| Validation Error | Missing required parameters | Check parameter requirements |

## Integration Testing

### MCP Inspector Testing

1. **Install MCP Inspector**:
   ```bash
   npm install -g @modelcontextprotocol/inspector
   ```

2. **Run Inspector**:
   ```bash
   npx @modelcontextprotocol/inspector
   ```

3. **Configure Connection**:
   - Transport: `stdio` or `streamable-http` 
   - For HTTP: `http://localhost:3000/mcp`
   - For stdio: Choose the binary path

4. **Test Operations**:
   - Browse available tools
   - Execute tools with sample data
   - Check response formats

## Troubleshooting

- **Server not responding**: Check environment variables and CRDP service
- **Tool execution fails**: Verify protection policy exists and parameters are correct
- **JWT errors**: Ensure token is valid and CRDP is configured for JWT auth
- **Version mismatch**: For external versioning, ensure you're passing the correct version 