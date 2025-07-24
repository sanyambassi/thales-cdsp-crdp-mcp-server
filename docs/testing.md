# Testing Guide

This document provides comprehensive testing instructions for the CRDP MCP Server, including manual testing with JSON-RPC and MCP Inspector.

## Prerequisites for Testing

Before running tests, ensure:

1. **CRDP MCP Server is built and ready**:
   ```bash
   npm install
   npm run build
   ```

2. **CRDP service is running and accessible**:
   ```bash
   # Test CRDP service connectivity
   curl -X GET http://your-crdp-server:8090/healthz
   curl -X GET http://your-crdp-server:8080/healthz
   ```

3. **Valid protection policies are configured** in your CRDP service

## Testing Methods

### 1. Manual Testing with JSON-RPC

#### Start the Server in HTTP Mode

```bash
# Set environment variables
export CRDP_SERVICE_URL="http://your-crdp-server:8090"
export CRDP_PROBES_URL="http://your-crdp-server:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"

# Start the server
npm start
```

#### Test Individual Tools

**1. Test protect_data Tool**

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

**1a. Test protect_data Tool with JWT**

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
        "protection_policy_name": "email_policy",
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data protected successfully. Protected data: enc_abc123def456ghi789\nExternal version: 2024001"
      }
    ]
  }
}
```

**2. Test reveal_data Tool**

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
        "protected_data": "enc_abc123def456ghi789",
        "username": "authorized_user",
        "protection_policy_name": "email_policy",
        "external_version": "2024001"
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data revealed successfully. Revealed data: john.doe@example.com"
      }
    ]
  }
}
```

**3. Test reveal_data Tool with External Version**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "reveal_data",
      "arguments": {
        "protected_data": "enc_xyz789abc123def456",
        "username": "admin_user",
        "protection_policy_name": "ssn_policy",
        "external_version": "2024002"
      }
    }
  }'
```

**3a. Test reveal_data Tool with JWT and External Version**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "reveal_data",
      "arguments": {
        "protected_data": "enc_xyz789abc123def456",
        "username": "admin_user",
        "protection_policy_name": "ssn_policy",
        "external_version": "2024002",
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data revealed successfully. Revealed data: 123-45-6789"
      }
    ]
  }
}
```

**4. Test protect_bulk Tool**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 4,
    "method": "tools/call",
    "params": {
      "name": "protect_bulk",
      "arguments": {
        "request_data": [
          {
            "protection_policy_name": "email_policy",
            "data": "jdoe@company.com"
          },
          {
            "protection_policy_name": "ssn_policy",
            "data": "987-65-4321"
          }
        ]
      }
    }
  }'
```

**5. Test reveal_bulk Tool**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "tools/call",
    "params": {
      "name": "reveal_bulk",
      "arguments": {
        "username": "authorized_user",
        "protected_data_array": [
          {
            "protection_policy_name": "email_policy",
            "protected_data": "enc_abc123def456ghi789"
          },
          {
            "protection_policy_name": "ssn_policy",
            "protected_data": "enc_xyz789abc123def456"
          }
        ]
      }
    }
  }'
```

**6. Test Monitoring Tools**

```bash
# Test get_metrics
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "tools/call",
    "params": {
      "name": "get_metrics",
      "arguments": {}
    }
  }'

# Test check_health
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 7,
    "method": "tools/call",
    "params": {
      "name": "check_health",
      "arguments": {}
    }
  }'

# Test check_liveness
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 8,
    "method": "tools/call",
    "params": {
      "name": "check_liveness",
      "arguments": {}
    }
  }'
```

#### Test Error Handling

**1. Test Missing Parameters**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 9,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "test@example.com"
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 9,
  "error": {
    "code": -32602,
    "message": "Missing required parameters for protect_data: protection_policy_name..."
  }
}
```

**2. Test Invalid Tool Name**

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 10,
    "method": "tools/call",
    "params": {
      "name": "invalid_tool",
      "arguments": {}
    }
  }'
```

### 2. Testing with MCP Inspector

#### Install and Setup MCP Inspector

1. **Install MCP Inspector**:
   ```bash
   # Install globally
   npm install -g @modelcontextprotocol/inspector
   
   # Or use npx (recommended)
   npx @modelcontextprotocol/inspector
   ```

2. **Start the CRDP MCP Server**:
   ```bash
   # For stdio transport (default)
   npm start
   
   # For HTTP transport
   export MCP_TRANSPORT="streamable-http"
   npm start
   ```

#### Using MCP Inspector

1. **Launch MCP Inspector**:
   ```bash
   # Using npx (recommended)
   npx @modelcontextprotocol/inspector
   
   # Or if installed globally
   mcp-inspector
   ```

2. **Configure Server Connection**:
   - **For stdio transport**: Select "stdio" and provide the path to your server executable
   - **For HTTP transport**: Select "HTTP" and enter `http://localhost:3000/mcp`

3. **Test Server Initialization**:
   - Click "Initialize" to establish connection
   - Verify server capabilities are listed

4. **Test Tools**:
   - Navigate to the "Tools" tab
   - Select a tool (e.g., `protect_data`)
   - Fill in the required parameters
   - Click "Call" to execute
   - Verify the response

5. **Test Prompts**:
   - Navigate to the "Prompts" tab
   - Select a prompt (e.g., `secure_data_processing`)
   - Fill in optional parameters
   - Click "Get" to retrieve the prompt
   - Verify the response

6. **Test Resources**:
   - Navigate to the "Resources" tab
   - Select a resource (e.g., `crdp_protection_policies`)
   - Click "Read" to retrieve the resource
   - Verify the content

#### MCP Inspector Test Scenarios

**Scenario 1: Basic Protection Flow**
1. Initialize server
2. Call `protect_data` with valid parameters
3. Verify response includes protected data and external version (if applicable)
4. Call `reveal_data` with the protected data
5. Verify original data is revealed correctly

**Scenario 2: Bulk Operations**
1. Initialize server
2. Call `protect_bulk` with multiple data items
3. Verify all items are protected
4. Call `reveal_bulk` with the protected data
5. Verify all items are revealed correctly

**Scenario 3: Error Handling**
1. Initialize server
2. Call tools with missing parameters
3. Verify appropriate error messages
4. Call tools with invalid data
5. Verify error handling

**Scenario 4: Monitoring**
1. Initialize server
2. Call `check_health`
3. Call `check_liveness`
4. Call `get_metrics`
5. Verify all monitoring tools return valid responses

### 3. Automated Testing Scripts

#### Create Test Script

Create a file `test-server.js`:

```javascript
const axios = require('axios');

const MCP_ENDPOINT = 'http://localhost:3000/mcp';

async function testTool(name, arguments) {
  try {
    const response = await axios.post(MCP_ENDPOINT, {
      jsonrpc: "2.0",
      id: Date.now(),
      method: "tools/call",
      params: {
        name,
        arguments
      }
    });
    
    console.log(`✅ ${name}:`, response.data.result);
    return response.data.result;
  } catch (error) {
    console.error(`❌ ${name}:`, error.response?.data || error.message);
    return null;
  }
}

async function runTests() {
  console.log('🧪 Starting CRDP MCP Server Tests...\n');

  // Test protect_data
  await testTool('protect_data', {
    data: 'john_doe@hiscompany.com',
    protection_policy_name: 'email_policy'
  });

  // Test check_health
  await testTool('check_health', {});

  // Test get_metrics
  await testTool('get_metrics', {});

  console.log('\n🏁 Tests completed!');
}

runTests();
```

#### Run Automated Tests

```bash
# Install axios if not already installed
npm install axios

# Run tests
node test-server.js
```

### 2. Automated Testing with test-server.js

You can use the provided automated test script to quickly verify your MCP server and CRDP connectivity.

**Location:** `scripts/test-server.js`

#### How to Use

1. **Ensure your MCP server is running** (e.g., `npm start`)
2. **Install dependencies** (if not already):
   ```bash
   npm install axios
   ```
3. **Run the test script:**
   ```bash
   node scripts/test-server.js
   ```
   Or, if you have the executable flag set:
   ```bash
   ./scripts/test-server.js
   ```
4. **(Optional) Test a different endpoint:**
   By default, the script tests `http://localhost:3000/mcp`. To test a different endpoint, set the `MCP_ENDPOINT` environment variable:
   ```bash
   MCP_ENDPOINT=http://your-server:3000/mcp node scripts/test-server.js
   ```

#### What It Does
- By default, tests only monitoring tools: `check_health`, `check_liveness`, and `get_metrics`.
- After those, it will prompt you if you want to test protect/reveal operations.
- If you answer 'y', it will ask for:
  - Data to protect
  - Protection policy name
  - Username for reveal
- It will then run `protect_data` and, using the protected value, run `reveal_data`.
- If you answer 'n', it exits after the monitoring tests.

**Example Output:**
```
🧪 Starting CRDP MCP Server Tests...
📡 Testing endpoint: http://localhost:3000/mcp

🔍 Testing monitoring tools...
✅ check_health: { ... }
✅ check_liveness: { ... }
✅ get_metrics: { ... }

Do you want to test protect/reveal operations? (y/n): y
Enter data to protect: test@example.com
Enter protection policy name: email_policy
✅ protect_data: { ... }
Enter username for reveal: testuser
✅ reveal_data: { ... }

🏁 Tests completed!
```

---

## Testing Different Versioning Scenarios

### 1. External Versioning Test

Test with a policy that returns external version:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 11,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "john.doe@company.com",
        "protection_policy_name": "PPol1"
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 11,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data protected successfully. Protected data: enc_abc123def456ghi789\nExternal version: 2024001"
      }
    ]
  }
}
```

### 2. Internal Versioning Test

Test with a policy that embeds version in protected data:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 12,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "jdoe@company.com",
        "protection_policy_name": "PPol2"
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data protected successfully. Protected data: 2024001Y57IlQvok1Ke"
      }
    ]
  }
}
```

### 3. No Versioning Test

Test with a policy that doesn't use versioning:

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 13,
    "method": "tools/call",
    "params": {
      "name": "protect_data",
      "arguments": {
        "data": "john_doe@hiscompany.com",
        "protection_policy_name": "PPol3"
      }
    }
  }'
```

**Expected Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 13,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Data protected successfully. Protected data: enc_BcmX5McZK6BB"
      }
    ]
  }
}
```

## Performance Testing

### Load Testing

Use a tool like Apache Bench (ab) or wrk to test server performance:

```bash
# Install Apache Bench (Ubuntu/Debian)
sudo apt-get install apache2-utils

# Test with 100 requests, 10 concurrent
ab -n 100 -c 10 -p test-payload.json -T application/json http://localhost:3000/mcp/
```

Create `test-payload.json`:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "check_health",
    "arguments": {}
  }
}
```

## Troubleshooting Test Issues

### Common Test Failures

1. **Connection Refused**
   - Ensure server is running
   - Check port configuration
   - Verify firewall settings

2. **CRDP Service Errors**
   - Verify CRDP service is accessible
   - Check network connectivity
   - Validate protection policies

3. **Authentication Errors**
   - Verify CRDP service credentials
   - Check JWT token configuration
   - Ensure proper permissions

4. **Parameter Validation Errors**
   - Check required parameters are provided
   - Verify parameter types and formats
   - Review error messages for guidance

### Debug Mode

Enable debug logging by setting environment variables:

```bash
export DEBUG=*
export NODE_ENV=development
npm start
```

### Log Analysis

Check server logs for:
- CRDP service connection status
- Tool execution results
- Error messages and stack traces
- Performance metrics

## Test Results Validation

### Success Criteria

- ✅ All tools return expected responses
- ✅ Error handling works correctly
- ✅ Versioning scenarios are handled properly
- ✅ Performance meets requirements
- ✅ Security measures are enforced

### Test Report Template

Create a test report with:

1. **Test Environment**: OS, Node.js version, CRDP service details
2. **Test Scenarios**: List of tests performed
3. **Results**: Pass/fail status for each test
4. **Issues**: Any problems encountered
5. **Performance**: Response times and throughput
6. **Recommendations**: Suggested improvements

## Continuous Testing

### Integration with CI/CD

Add testing to your CI/CD pipeline:

```yaml
# Example GitHub Actions workflow
name: Test CRDP MCP Server
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm install
      - run: npm run build
      - run: npm test  # Add test script to package.json
```

This comprehensive testing guide ensures your CRDP MCP Server is thoroughly validated before deployment. 