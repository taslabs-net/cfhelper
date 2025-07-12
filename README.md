# CF Helper - Cloudflare Documentation Assistant

A multi-architecture Docker container that provides an AI-powered chat interface for Cloudflare documentation using Cloudflare's AI models.

## Quick Start

```bash
# Pull the image
docker pull schenanigans/cfhelper:latest

# Run with basic configuration
docker run -p 9876:1111 -p 9877:8080 --env-file .env schenanigans/cfhelper:latest
```

Access the interface at `http://localhost:9876`

## Multi-Architecture Support

This image supports both:
- **linux/amd64** (Intel/AMD processors)
- **linux/arm64** (Apple Silicon, ARM processors)

## Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `CF_ACCESS_CLIENT_ID` | Cloudflare Access client ID | - |  |
| `CF_ACCESS_CLIENT_SECRET` | Cloudflare Access client secret | - |  |
| `CF_WORKER_DOMAIN` | CF Helper Worker domain | cfhelper.taslabs.net | ❌ |
| `EXTERNAL_PORT` | External HTTP port mapping | 9876 | ❌ |
| `EXTERNAL_WEBSOCKET_PORT` | External WebSocket port mapping | 9877 | ❌ |
| `LOG_LEVEL` | Logging level (ERROR, WARN, INFO, DEBUG) | INFO | ❌ |
| `TZ` | Timezone | UTC | ❌ |
| `TURNSTILE_ENABLED` | Enable Turnstile verification | false | ❌ |
| `TURNSTILE_SITE_KEY` | Turnstile site key | - | ❌ |
| `TURNSTILE_SECRET_KEY` | Turnstile secret key | - | ❌ |
| `CONFIG_PATH` | Configuration directory path | ./config | ❌ |
| `ENABLE_CERT_UPLOAD` | Enable mTLS certificate upload | false | ❌ |
| `CERT_UPLOAD_TOKEN` | Certificate upload authentication token | - | ❌ |

## Docker Compose Example

```yaml
services:
  cfhelper:
    image: schenanigans/cfhelper:latest
    ports:
      - "9876:1111"
      - "9877:8080"
    environment:
      - CF_ACCESS_CLIENT_ID=your_client_id
      - CF_ACCESS_CLIENT_SECRET=your_client_secret
      - LOG_LEVEL=INFO
    volumes:
      - ./config:/app/config
    restart: unless-stopped
```

## Architecture

CF Helper connects to Cloudflare Workers that provide access to multiple AI models through Cloudflare's AI platform. The Worker handles authentication, model routing, and document search.

## Security

- Uses Cloudflare Zero Trust Access for authentication
- Optional Turnstile CAPTCHA protection
- Support for mTLS certificates
- No API keys stored in the container

## Links

- **Source Code**: [GitHub Repository](https://github.com/cloudflare/mcp-server-cloudflare)
- **Documentation**: Full setup and configuration guides available in project repository
- **Cloudflare MCP**: [Cloudflare MCP Server](https://github.com/cloudflare/mcp-server-cloudflare/tree/main/apps/docs-vectorize)
