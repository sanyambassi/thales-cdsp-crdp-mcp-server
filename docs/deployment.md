# Deployment Guide

This guide covers deploying the CRDP MCP Server in various environments.

## Prerequisites

Before deployment, ensure you have:

1. **Completed the prerequisites** from [prerequisites.md](prerequisites.md)
2. **Built the project** successfully
3. **Configured your CRDP service** endpoints
4. **Tested the server** using the [testing guide](testing.md)

## Local Deployment

### Development Environment

```bash
# Clone the repository
git clone <your-repo-url>
cd crdp-mcp-server

# Install dependencies
npm install

# Build the project
npm run build

# Start in development mode
npm run dev
```

### Production Environment

```bash
# Install dependencies
npm install

# Build the project
npm run build

# Start the server
npm start
```

## Docker Deployment

### Create Dockerfile

Create a `Dockerfile` in the project root:

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Change ownership
RUN chown -R nodejs:nodejs /app
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"

# Start the application
CMD ["npm", "start"]
```

### Build and Run Docker Container

```bash
# Build the image
docker build -t crdp-mcp-server .

# Run the container
docker run -d \
  --name crdp-mcp-server \
  -p 3000:3000 \
  -e CRDP_SERVICE_URL=http://your-crdp-server:8090 \
  -e CRDP_PROBES_URL=http://your-crdp-server:8080 \
  -e MCP_TRANSPORT=streamable-http \
  crdp-mcp-server
```

### Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  crdp-mcp-server:
    build: .
    ports:
      - "3000:3000"
    environment:
      - CRDP_SERVICE_URL=http://crdp-service:8090
      - CRDP_PROBES_URL=http://crdp-service:8080
      - MCP_TRANSPORT=streamable-http
    depends_on:
      - crdp-service
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"]
      interval: 30s
      timeout: 10s
      retries: 3

  crdp-service:
    image: your-crdp-service-image
    ports:
      - "8090:8090"
      - "8080:8080"
    environment:
      - CRDP_CONFIG_PATH=/etc/crdp/config.yaml
    volumes:
      - ./crdp-config:/etc/crdp
    restart: unless-stopped
```

Run with:
```bash
docker-compose up -d
```

## Kubernetes Deployment

### Create Kubernetes Manifests

Create `k8s/namespace.yaml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: crdp-mcp
```

Create `k8s/configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: crdp-mcp-config
  namespace: crdp-mcp
data:
  CRDP_SERVICE_URL: "http://crdp-service:8090"
  CRDP_PROBES_URL: "http://crdp-service:8080"
  MCP_TRANSPORT: "streamable-http"
  MCP_PORT: "3000"
```

Create `k8s/deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crdp-mcp-server
  namespace: crdp-mcp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: crdp-mcp-server
  template:
    metadata:
      labels:
        app: crdp-mcp-server
    spec:
      containers:
      - name: crdp-mcp-server
        image: your-registry/crdp-mcp-server:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: crdp-mcp-config
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

Create `k8s/service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: crdp-mcp-service
  namespace: crdp-mcp
spec:
  selector:
    app: crdp-mcp-server
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
```

Create `k8s/ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: crdp-mcp-ingress
  namespace: crdp-mcp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: crdp-mcp.your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: crdp-mcp-service
            port:
              number: 80
```

### Deploy to Kubernetes

```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Apply configurations
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml

# Check deployment status
kubectl get pods -n crdp-mcp
kubectl get services -n crdp-mcp
```

## Cloud Platform Deployment

### AWS ECS

Create `aws/task-definition.json`:
```json
{
  "family": "crdp-mcp-server",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::your-account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::your-account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "crdp-mcp-server",
      "image": "your-account.dkr.ecr.region.amazonaws.com/crdp-mcp-server:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "CRDP_SERVICE_URL",
          "value": "http://your-crdp-service:8090"
        },
        {
          "name": "CRDP_PROBES_URL",
          "value": "http://your-crdp-service:8080"
        },
        {
          "name": "MCP_TRANSPORT",
          "value": "streamable-http"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/crdp-mcp-server",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

### Google Cloud Run

```bash
# Build and push to Google Container Registry
gcloud builds submit --tag gcr.io/your-project/crdp-mcp-server

# Deploy to Cloud Run
gcloud run deploy crdp-mcp-server \
  --image gcr.io/your-project/crdp-mcp-server \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars CRDP_SERVICE_URL=http://your-crdp-service:8090,CRDP_PROBES_URL=http://your-crdp-service:8080,MCP_TRANSPORT=streamable-http
```

### Azure Container Instances

```bash
# Build and push to Azure Container Registry
az acr build --registry your-registry --image crdp-mcp-server .

# Deploy to Container Instances
az container create \
  --resource-group your-rg \
  --name crdp-mcp-server \
  --image your-registry.azurecr.io/crdp-mcp-server:latest \
  --ports 3000 \
  --environment-variables CRDP_SERVICE_URL=http://your-crdp-service:8090 CRDP_PROBES_URL=http://your-crdp-service:8080 MCP_TRANSPORT=streamable-http
```

## Environment-Specific Configuration

### Development Environment

```bash
# Environment variables for development
export CRDP_SERVICE_URL="http://localhost:8090"
export CRDP_PROBES_URL="http://localhost:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"
export NODE_ENV="development"
```

### Staging Environment

```bash
# Environment variables for staging
export CRDP_SERVICE_URL="http://staging-crdp-service:8090"
export CRDP_PROBES_URL="http://staging-crdp-service:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"
export NODE_ENV="staging"
```

### Production Environment

```bash
# Environment variables for production
export CRDP_SERVICE_URL="http://prod-crdp-service:8090"
export CRDP_PROBES_URL="http://prod-crdp-service:8080"
export MCP_TRANSPORT="streamable-http"
export MCP_PORT="3000"
export NODE_ENV="production"
```

## Monitoring and Logging

### Health Checks

The server provides health check endpoints:

```bash
# Basic health check
curl http://localhost:3000/health

# Server information
curl http://localhost:3000/
```

### Logging

Configure logging for your environment:

```bash
# Enable debug logging
export DEBUG=*

# Enable structured logging
export NODE_ENV=production
```

### Metrics

Monitor server performance:

```bash
# Get server metrics
curl http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "get_metrics",
      "arguments": {}
    }
  }'
```

## Security Considerations

### Network Security

1. **Use HTTPS/WSS** for all external communications
2. **Implement proper firewall rules**
3. **Use VPN or private networks** for internal communications
4. **Restrict access** to CRDP service endpoints

### Authentication and Authorization

1. **Implement JWT authentication** if required by CRDP service
2. **Use service accounts** with minimal required permissions
3. **Rotate credentials** regularly
4. **Monitor access logs** for suspicious activity

### Data Protection

1. **Encrypt data in transit** using TLS
2. **Encrypt data at rest** if storing any sensitive information
3. **Implement proper access controls**
4. **Regular security audits**

## Troubleshooting

### Common Deployment Issues

1. **Container won't start**
   - Check container logs: `docker logs crdp-mcp-server`
   - Verify environment variables are set correctly
   - Ensure CRDP service is accessible

2. **Health checks failing**
   - Verify the health endpoint is responding
   - Check network connectivity
   - Review application logs

3. **Performance issues**
   - Monitor resource usage
   - Check CRDP service performance
   - Review application metrics

### Getting Help

1. Check the [troubleshooting section](../README.md#troubleshooting)
2. Review application logs
3. Test connectivity to CRDP service
4. Open an issue on GitHub with detailed information

## Backup and Recovery

### Backup Strategy

1. **Configuration backups**: Store environment configurations in version control
2. **Docker images**: Tag and store important versions
3. **Deployment manifests**: Version control all deployment configurations

### Recovery Procedures

1. **Rollback deployment**: Use previous working version
2. **Restore configuration**: Apply backed-up configurations
3. **Verify connectivity**: Test CRDP service connectivity
4. **Run health checks**: Ensure all endpoints are responding

This deployment guide provides comprehensive instructions for deploying the CRDP MCP Server in various environments. 