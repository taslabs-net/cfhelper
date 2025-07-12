```
 ██████╗███████╗    ██╗  ██╗███████╗██╗     ██████╗ ███████╗██████╗ 
██╔════╝██╔════╝    ██║  ██║██╔════╝██║     ██╔══██╗██╔════╝██╔══██╗
██║     █████╗      ███████║█████╗  ██║     ██████╔╝█████╗  ██████╔╝
██║     ██╔══╝      ██╔══██║██╔══╝  ██║     ██╔═══╝ ██╔══╝  ██╔══██╗
╚██████╗██║         ██║  ██║███████╗███████╗██║     ███████╗██║  ██║
 ╚═════╝╚═╝         ╚═╝  ╚═╝╚══════╝╚══════╝╚═╝     ╚══════╝╚═╝  ╚═╝
                                                                     
                     Built by Tim Schneider
```

# CF Helper - Cloudflare Documentation Assistant

A multi-platform AI assistant system that provides intelligent access to Cloudflare documentation using Cloudflare's enterprise capabilities including Workers AI, Durable Objects, KV storage, Zero Trust Access, and AI Gateway integration.

## Screenshots

### iOS
<p align="center">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_iOS_main.png" width="300" alt="CF Helper iOS Main Screen">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_iOS_model_selection.png" width="300" alt="CF Helper iOS Model Selection">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_iOS_response.png" width="300" alt="CF Helper iOS Response">
</p>

### macOS
<p align="center">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_macOS_main.png" width="600" alt="CF Helper macOS Main Screen">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_macOS_response.png" width="600" alt="CF Helper macOS Response">
</p>

## 🚀 Quick Start

### Docker (Recommended for Quick Setup)
```bash
# Pull the image
docker pull schenanigans/cfhelper:latest

# Run with basic configuration
docker run -p 9876:1111 -p 9877:8080 \
  -e CF_ACCESS_CLIENT_ID=your_client_id \
  -e CF_ACCESS_CLIENT_SECRET=your_client_secret \
  -e CF_ENDPOINT=cfhelper.yourdomain.com \
  -e CF_WORKER_DOMAIN=your-worker.workers.dev \
  schenanigans/cfhelper:latest
```

Access the interface at `http://localhost:9876`

## 🏗️ Architecture

CF Helper consists of three client applications that connect to a centralized Cloudflare Worker backend:

1. **[Web Client](documentation/cfhelperworker.md)** - React app hosted on Cloudflare Workers
2. **[Docker Client](documentation/cfhelperdocker.md)** - Containerized Node.js/React app ([Docker Hub](https://hub.docker.com/r/schenanigans/cfhelper))
3. **[Native Apps](documentation/cfhelpermac.md)** - SwiftUI apps for macOS/iOS/iPadOS

### System Architecture

```
+------------------+     +------------------+     +------------------+
|   Mac/iOS App    |     | Docker Container |     |   Web Browser    |
|   (SwiftUI)      |     |  (Node + React)  |     |     (React)      |
+--------+---------+     +--------+---------+     +--------+---------+
         |                        |                         |
         |    X-Client-Type:      |    X-Client-Type:       |    X-Client-Type:
         |    cf-helper-apple     |    cf-helper-docker     |    cf-helper-worker
         |                        |                         |
         +------------------------+-------------------------+
                                  |
                                  v
                     +------------------------+
                     |  Cloudflare Edge (CDN) |
                     |  +------------------+  |
                     |  |  Zero Trust      |  |
                     |  |  Access Gateway  |  |
                     |  +------------------+  |
                     +-----------+------------+
                                 |
                     +-----------v------------+
                     |   CF Helper Worker     |
                     |  +------------------+  |
                     |  | Durable Objects  |  | <-- WebSocket Sessions
                     |  | (Chat Sessions)  |  |
                     |  +------------------+  |
                     |  +------------------+  |
                     |  |   KV Storage     |  | <-- Source of Truth
                     |  | (All Models JSON) |  |     for Model Config
                     |  +------------------+  |
                     |  +------------------+  |
                     |  |  Workers AI      |  | <-- Cloudflare Models
                     |  |    Binding       |  |
                     |  +------------------+  |
                     +-----------+------------+
                                 |
                     +-----------v------------+
                     |     AI Gateway         |
                     |  +------------------+  |
                     |  |   OpenAI API     |  | <-- GPT Models
                     |  +------------------+  |
                     |  +------------------+  |
                     |  |  Anthropic API   |  | <-- Claude Models
                     |  +------------------+  |
                     +------------------------+
```

## ✨ Key Features

### 🤖 Unified AI Backend
- Single Cloudflare Worker handles all AI requests
- Supports multiple AI providers (Cloudflare Workers AI, OpenAI, Anthropic)
- Dynamic model selection based on available API keys
- Centralized model configuration in KV storage

### 🔐 Security & Authentication
- Cloudflare Zero Trust Access protection
- Turnstile verification for web clients
- Platform-specific authentication strategies
- Optional mTLS support for enterprise deployments

### 💬 Real-time Communication
- WebSocket support via Cloudflare Durable Objects
- PartyKit-compatible message routing
- Session persistence and state management
- Multi-client chat synchronization

### 📊 Observability
- Custom `X-Client-Type` headers for traffic analysis
- Cloudflare Analytics integration
- Request tracking with unique IDs
- Tail Workers for log aggregation

## 🛠️ Technical Stack

### Backend (Cloudflare Worker)
- **Runtime**: Cloudflare Workers (V8 isolates)
- **Storage**: KV (configuration), Durable Objects (sessions)
- **AI**: Workers AI binding, OpenAI/Anthropic via AI Gateway
- **Security**: Zero Trust Access, Turnstile
- **Language**: TypeScript

### Frontend Clients

#### Web
- React with TypeScript
- Turnstile integration
- Hosted directly on Cloudflare Workers

#### Docker
- Node.js/Express backend
- React frontend
- Multi-architecture support (AMD64/ARM64)
- Available on [Docker Hub](https://hub.docker.com/r/schenanigans/cfhelper)

#### Mac/iOS
- Native SwiftUI application
- Universal app supporting macOS, iOS, and iPadOS
- Features:
  - Real-time WebSocket chat
  - Dynamic model selection
  - Cloudflare Zero Trust authentication
  - Native performance and UI

## 📋 Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `CF_ACCESS_CLIENT_ID` | Cloudflare Access client ID | ✅ |
| `CF_ACCESS_CLIENT_SECRET` | Cloudflare Access client secret | ✅ |
| `CF_ENDPOINT` | CF Helper Worker custom domain | ✅ |
| `CF_WORKER_DOMAIN` | CF Helper Worker .workers.dev domain | ✅ |
| `ANTHROPIC_API_KEY` | Anthropic API key | ❌ |
| `OPENAI_API_KEY` | OpenAI API key | ❌ |

### Model Configuration

Models are configured in KV storage with the following structure:

```json
[
  {
    "id": "@cf/meta/llama-4-scout-17b-16e-instruct",
    "name": "Llama 4 Scout (17B)",
    "description": "Meta's multimodal model with 16 experts - Default",
    "provider": "cloudflare",
    "default": true
  },
  {
    "id": "claude-opus-4-20250514",
    "name": "Claude Opus 4",
    "description": "Anthropic's most capable model",
    "provider": "anthropic",
    "requiresKey": "ANTHROPIC_API_KEY"
  }
]
```

## 🚦 API Endpoints

### Public Endpoints
- `GET /cfhelper/api/health` - Worker health check
- `GET /cfhelper/api/models` - List available AI models

### Platform-Specific Endpoints
- `POST /cfhelper/api/docker` - Docker client queries
- `POST /cfhelper/api/apple` - Apple client queries

### WebSocket Endpoints
- `wss://cfhelper.taslabs.net/parties/chat/{roomId}` - Real-time chat

## 🚀 Deployment

### 1. Deploy the Worker
Follow the instructions in [CF Helper Worker documentation](documentation/cfhelperworker.md)

### 2. Configure Models
Upload your models configuration to KV:
```bash
wrangler kv key put --namespace-id YOUR_NAMESPACE_ID Models < models.json
```

### 3. Set Up Access
Configure Cloudflare Zero Trust Access policies for your domain

### 4. Choose Your Client

#### Docker Deployment
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
      - CF_ENDPOINT=cfhelper.yourdomain.com
      - CF_WORKER_DOMAIN=your-worker.workers.dev
    restart: unless-stopped
```

#### Native App Installation
See [CF Helper Mac documentation](documentation/cfhelpermac.md) for building and deploying the native apps.

## 🔒 Security Considerations

- All traffic protected by Cloudflare Zero Trust
- API keys stored as encrypted Worker secrets
- Platform-specific authentication strategies
- No persistent chat storage (privacy-first design)
- Optional mTLS certificate support

## 📚 Documentation

- [CF Helper Worker](documentation/cfhelperworker.md) - Backend Worker setup
- [CF Helper Docker](documentation/cfhelperdocker.md) - Docker deployment guide
- [CF Helper Mac](documentation/cfhelpermac.md) - Native app documentation
- [Architecture Overview](documentation/cfhelperoverview.md) - Detailed system design

## 🔗 Links

- **Source Code**: This repository
- **Docker Hub**: [schenanigans/cfhelper](https://hub.docker.com/r/schenanigans/cfhelper)
- **Cloudflare MCP**: [MCP Server Cloudflare](https://github.com/cloudflare/mcp-server-cloudflare)

## 🎯 Future Enhancements

- Additional AI provider integrations
- Enhanced model routing logic
- Persistent chat history (opt-in)
- Multi-language support
- Voice input/output capabilities

---

Built with ❤️ using Cloudflare's global network
