# CF Helper Docker

CF Helper Docker provides a self-hosted web interface for the CF Helper system. It runs a local web server that connects to the CF Helper Worker backend, making it ideal for server deployments, internal networks, or environments where a containerized solution is preferred.

**Docker Hub**: [schenanigans/cfhelper](https://hub.docker.com/r/schenanigans/cfhelper)

## What It Does

The Docker container creates a local web server that:
- **Serves a web interface** on port 9876 - access CF Helper from any browser on your network
- **Proxies WebSocket connections** on port 9877 - enables real-time chat with the Worker backend
- **Handles authentication** - manages CF-Access credentials for secure Worker communication
- **Supports enterprise features** - optional mTLS for restricted networks

## How It Works with the Worker

The Docker container acts as a bridge between your local network and the CF Helper Worker:

```
[Your Browser] → [Docker Container] → [CF Helper Worker] → [AI Models]
     ↓                    ↓                    ↓
Local Network      Express Server      Cloudflare Edge
 Port 9876          Proxy Layer         Global Network
```

1. You access the web interface at `http://localhost:9876`
2. The Express server authenticates with the Worker using your CF-Access credentials
3. Your queries are proxied to the Worker which processes them with AI models
4. Responses stream back through the WebSocket connection in real-time

## Key Benefits

### Local Web Server
- **Network Independence**: Once configured, access CF Helper from any device on your network
- **No Installation Required**: Users only need a web browser
- **Centralized Management**: Single configuration point for all users
- **Resource Efficiency**: One container serves multiple users

### Enterprise Ready
- **mTLS Support**: Connect through corporate proxies and restricted networks
- **Docker Integration**: Works with existing container orchestration
- **Health Monitoring**: Built-in health check endpoints
- **Structured Logging**: Winston logger for production debugging

## How the Local Web Server Works

### Architecture Overview

The Docker container runs an Express.js server that provides:

1. **Static Web Hosting** - Serves the React frontend at `http://localhost:9876`
2. **API Proxy Layer** - Routes API calls to the Worker with authentication
3. **WebSocket Bridge** - Maintains real-time connection to the Worker
4. **Local Interface** - No internet access needed after initial setup

### Request Flow

```
1. Browser → http://localhost:9876
   └─> Express serves React app

2. React app → /cfhelper/api/models
   └─> Express proxies to Worker with CF-Access headers
   └─> Returns available AI models

3. User sends message → ws://localhost:9877
   └─> Express WebSocket proxy
   └─> Forwards to wss://cfhelper.taslabs.net
   └─> Worker processes with AI
   └─> Response streams back through WebSocket
```

### Why Use the Docker Container?

**Perfect for:**
- **Home Labs** - Run on your NAS or home server
- **Office Networks** - Deploy once, access from any workstation
- **Development Teams** - Shared instance for the whole team
- **Air-gapped Networks** - With mTLS support for restricted environments

**Benefits over direct Worker access:**
- No need to distribute CF-Access credentials to each user
- Single configuration point for updates
- Local caching of static assets
- Network-level access control
- Integration with existing Docker infrastructure

## Quick Start

### Using Docker Hub (Recommended)

```bash
# 1. Pull the image
docker pull schenanigans/cfhelper:latest

# 2. Create configuration
cat > .env << EOF
CF_ACCESS_CLIENT_ID=your_client_id
CF_ACCESS_CLIENT_SECRET=your_client_secret
CF_DOMAIN=cfhelper.taslabs.net
EOF

# 3. Run the container
docker run -d \
  --name cfhelper \
  -p 9876:1111 \
  -p 9877:8080 \
  --env-file .env \
  schenanigans/cfhelper:latest

# 4. Access the interface
open http://localhost:9876
```

### Using Docker Compose

```yaml
# compose.yaml
services:
  cfhelper:
    image: schenanigans/cfhelper:latest
    container_name: cfhelper
    ports:
      - "9876:1111"  # Web interface
      - "9877:8080"  # WebSocket connection
    environment:
      - CF_ACCESS_CLIENT_ID=${CF_ACCESS_CLIENT_ID}
      - CF_ACCESS_CLIENT_SECRET=${CF_ACCESS_CLIENT_SECRET}
      - CF_DOMAIN=cfhelper.taslabs.net
    restart: unless-stopped
```

```bash
# Start the service
docker compose up -d
```

## Configuration

### Required Settings

The Docker container needs CF-Access credentials to communicate with the Worker:

```bash
# .env file
CF_ACCESS_CLIENT_ID=your_client_id        # From CF Zero Trust
CF_ACCESS_CLIENT_SECRET=your_client_secret # From CF Zero Trust
CF_DOMAIN=cfhelper.taslabs.net            # Worker domain
```

### Optional Settings

```bash
# Server ports (defaults shown)
PORT=9876                    # Web interface port
WS_PORT=9877                 # WebSocket port

# Environment
NODE_ENV=production          # Enable production optimizations

# mTLS for enterprise networks
USE_MTLS=false               # Enable mTLS
MTLS_CERT_PATH=./config/cert.pem
MTLS_KEY_PATH=./config/key.pem
MTLS_CA_PATH=./config/ca.pem
```

## Production Deployment

### Kubernetes Example

```yaml
# cfhelper-deployment.yaml
apiVersion: v1
kind: Secret
metadata:
  name: cfhelper-secrets
stringData:
  CF_ACCESS_CLIENT_ID: "your_client_id"
  CF_ACCESS_CLIENT_SECRET: "your_client_secret"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cfhelper
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cfhelper
  template:
    metadata:
      labels:
        app: cfhelper
    spec:
      containers:
      - name: cfhelper
        image: schenanigans/cfhelper:latest
        ports:
        - containerPort: 1111
          name: web
        - containerPort: 8080
          name: websocket
        env:
        - name: CF_DOMAIN
          value: "cfhelper.taslabs.net"
        envFrom:
        - secretRef:
            name: cfhelper-secrets
---
apiVersion: v1
kind: Service
metadata:
  name: cfhelper
spec:
  selector:
    app: cfhelper
  ports:
  - name: web
    port: 9876
    targetPort: 1111
  - name: websocket
    port: 9877
    targetPort: 8080
```

## Using the Web Interface

### First Time Setup

1. **Access the Interface**
   ```
   http://localhost:9876
   ```

2. **Enter Your Name**
   - Used to identify your messages
   - Stored locally in your browser

3. **Select an AI Model**
   - Models are loaded from the Worker
   - Available models depend on Worker configuration

4. **Start Chatting**
   - Ask questions about Cloudflare
   - Responses stream in real-time
   - Chat history persists during session

### Network Access

Once running, the interface is accessible from:
- **Local machine**: `http://localhost:9876`
- **Other devices**: `http://[server-ip]:9876`
- **Behind proxy**: Configure your reverse proxy to forward ports 9876 and 9877

## Security & Monitoring

### Security Model

```
Users → Docker Container → CF Worker
  ↓          ↓                ↓
No auth   CF-Access      AI Processing
Local     Credentials    No data storage
```

- **Local Access**: No authentication on local interface
- **Worker Communication**: Secured with CF-Access tokens
- **Data Privacy**: No conversation history stored
- **Network Isolation**: Can run in isolated networks with mTLS

### Monitoring

```bash
# Health endpoints
GET /cfhelper/api/containerhealth  # Container status
GET /cfhelper/api/workerhealth     # Worker connectivity

# View logs with Winston formatting
docker logs -f cfhelper | jq

# Monitor resource usage
docker stats cfhelper
```

### Production Checklist

- [ ] Use secrets management for credentials
- [ ] Enable HTTPS with reverse proxy
- [ ] Configure firewall rules for ports
- [ ] Set up log aggregation
- [ ] Monitor container health
- [ ] Regular security updates

## Troubleshooting

### Cannot Access the Web Interface

```bash
# Check if container is running
docker ps | grep cfhelper

# Check container logs
docker logs cfhelper

# Verify ports are exposed
curl http://localhost:9876/cfhelper/api/containerhealth
```

### Connection to Worker Failed

```bash
# Test Worker connectivity
curl http://localhost:9876/cfhelper/api/workerhealth

# Common issues:
# - Invalid CF_ACCESS credentials
# - Incorrect CF_DOMAIN
# - Network firewall blocking outbound HTTPS
```

### WebSocket Connection Issues

- Browser console shows WebSocket errors
- Messages don't appear in real-time
- Check that port 9877 is accessible
- Ensure no proxy is interfering with WebSocket upgrade

## Advanced Configuration

### Enterprise mTLS Setup

```bash
# 1. Place certificates in config directory
mkdir config
cp /path/to/cert.pem config/
cp /path/to/key.pem config/
cp /path/to/ca.pem config/

# 2. Enable mTLS in .env
USE_MTLS=true
MTLS_CERT_PATH=./config/cert.pem
MTLS_KEY_PATH=./config/key.pem
MTLS_CA_PATH=./config/ca.pem

# 3. Mount certificates in Docker
docker run -d \
  --name cfhelper \
  -p 9876:1111 \
  -p 9877:8080 \
  -v ./config:/app/config:ro \
  --env-file .env \
  schenanigans/cfhelper:latest
```

### Reverse Proxy with nginx

```nginx
# /etc/nginx/sites-available/cfhelper
server {
    listen 443 ssl http2;
    server_name cfhelper.internal.company.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Web interface
    location / {
        proxy_pass http://localhost:9876;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # WebSocket connection
    location /ws {
        proxy_pass http://localhost:9877;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
    }
}
```

## Integration

### Reverse Proxy (nginx)

```nginx
server {
    listen 443 ssl;
    server_name cfhelper.example.com;
    
    location / {
        proxy_pass http://localhost:9876;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    location /ws {
        proxy_pass http://localhost:9877;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Docker

```yaml
services:
  cfhelper:
    image: cfhelper:latest
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
    secrets:
      - cf_access_id
      - cf_access_secret
```
