# Prerequisites

This document outlines all the prerequisites required to run the CRDP MCP Server.

## System Requirements

- **Operating System**: Windows 2016 or higher, or Linux (Ubuntu 18.04+, CentOS 7+)
- **Memory**: Minimum 2GB RAM, recommended 4GB+
- **Storage**: At least 500MB free disk space
- **Network**: Access to CRDP service endpoints

## Required Software

### 1. Node.js

**Version**: 18.0.0 or higher

#### Installation Options:

**Option A: Official Installer**
1. Visit [nodejs.org](https://nodejs.org/)
2. Download the LTS version for your platform
3. Run the installer and follow the setup wizard

**Option B: Package Manager (Linux/macOS)**
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS (using Homebrew)
brew install node@18

# CentOS/RHEL
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
```

**Option C: Node Version Manager (nvm)**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal or run
source ~/.bashrc

# Install Node.js 18
nvm install 18
nvm use 18
```

#### Verify Installation:
```bash
node --version
npm --version
```

### 2. TypeScript

**Version**: 5.0.0 or higher

#### Installation:
```bash
npm install -g typescript
```

#### Verify Installation:
```bash
tsc --version
```

### 3. npx (Node Package Execute)

**Comes with npm 5.2.0+**

npx allows you to run packages without installing them globally. It's useful for running tools like MCP Inspector.

#### Verify Installation:
```bash
npx --version
```

#### Usage Examples:
```bash
# Run MCP Inspector without installing globally
npx @modelcontextprotocol/inspector

# Run other tools
npx ts-node src/crdp-mcp-server.ts
```

### 4. Git (Optional but Recommended)

**For cloning the repository and version control**

#### Installation:

**Windows:**
1. Download from [git-scm.com](https://git-scm.com/)
2. Run the installer

**macOS:**
```bash
# Using Homebrew
brew install git

# Or using Xcode Command Line Tools
xcode-select --install
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt-get install git

# CentOS/RHEL
sudo yum install git
```

#### Verify Installation:
```bash
git --version
```

## CRDP Service Requirements

### 1. CRDP Service Access

You need access to a running CipherTrust RestFul Data Protection (CRDP) service with:

- **Service URL**: Endpoint for protect/reveal operations (default: `http://localhost:8090`)
- **Probes URL**: Endpoint for monitoring operations (default: `http://localhost:8080`)
- **Valid Protection Policies**: Configured policies for data protection

### 2. Network Connectivity

Ensure your system can reach the CRDP service endpoints:

```bash
# Test connectivity to CRDP service
curl -X GET http://your-crdp-server:8090/healthz

# Test connectivity to probes endpoint
curl -X GET http://your-crdp-server:8080/healthz
```

### 3. Authentication (if required)

If your CRDP service requires authentication:
- Obtain valid credentials
- Configure JWT tokens if needed
- Ensure proper permissions for protect/reveal operations

## Development Tools (Optional)

### 2. Code Editor

Recommended editors:
- **Visual Studio Code** with TypeScript extensions
- **WebStorm** or **IntelliJ IDEA**
- **Vim** or **Emacs** with TypeScript plugins

### 3. HTTP Client

For testing API endpoints:
- **Postman**
- **Insomnia**
- **curl** (command line)

### 4. MCP Inspector

For testing MCP protocol:
- **Model Context Protocol Inspector**
- Install with: `npm install -g @modelcontextprotocol/inspector`
- Or use: `npx @modelcontextprotocol/inspector`

## Platform-Specific Setup

### Windows

1. **Install Node.js** using the official installer
2. **Install TypeScript** globally:
   ```cmd
   npm install -g typescript
   ```
3. **Verify installations**:
   ```cmd
   node --version
   npm --version
   tsc --version
   ```

### macOS

1. **Install Homebrew** (if not already installed):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install Node.js and TypeScript**:
   ```bash
   brew install node@18
   npm install -g typescript
   ```

3. **Verify installations**:
   ```bash
   node --version
   npm --version
   tsc --version
   ```

### Linux (Ubuntu/Debian)

1. **Update package list**:
   ```bash
   sudo apt update
   ```

2. **Install Node.js**:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

3. **Install TypeScript**:
   ```bash
   sudo npm install -g typescript
   ```

4. **Verify installations**:
   ```bash
   node --version
   npm --version
   tsc --version
   ```

### Linux (CentOS/RHEL)

1. **Install Node.js**:
   ```bash
   curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
   sudo yum install -y nodejs
   ```

2. **Install TypeScript**:
   ```bash
   sudo npm install -g typescript
   ```

3. **Verify installations**:
   ```bash
   node --version
   npm --version
   tsc --version
   ```

## Verification Checklist

Before proceeding with the CRDP MCP Server installation, verify:

- [ ] Node.js 18+ is installed and accessible
- [ ] npm is installed and accessible
- [ ] npx is available (comes with npm 5.2.0+)
- [ ] TypeScript is installed globally
- [ ] CRDP service is running and accessible
- [ ] Network connectivity to CRDP endpoints is confirmed
- [ ] Required protection policies are configured in CRDP
- [ ] Authentication credentials are available (if required)

## Troubleshooting

### Common Issues

1. **"tsc is not recognized"**
   - Ensure TypeScript is installed globally: `npm install -g typescript`
   - Check PATH environment variable includes npm global packages

2. **"node is not recognized"**
   - Reinstall Node.js and ensure it's added to PATH
   - Restart terminal/command prompt after installation

3. **Permission errors on Linux/macOS**
   - Use `sudo` for global npm installations
   - Or configure npm to use a different directory: `npm config set prefix ~/.npm-global`

4. **Network connectivity issues**
   - Check firewall settings
   - Verify CRDP service is running
   - Test with curl or telnet

### Getting Help

If you encounter issues:

1. Check the [troubleshooting section](#troubleshooting)
2. Verify all prerequisites are met using the checklist above
3. Check the [testing documentation](testing.md) for verification steps
4. Open an issue on GitHub with detailed error information 