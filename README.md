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

A multi-platform AI assistant that provides intelligent access to Cloudflare documentation.

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

### Web Interface / Docker

<p align="center">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_web_docker_main.png" width="600" alt="CF Helper Web/Docker Main Screen">
  <img src="https://github.com/taslabs-net/cfhelper/blob/main/screenshots/cfhelper_web_docker_response.png" width="600" alt="CF Helper Web/Docker Response">
</p>

## Overview

CF Helper demonstrates enterprise Cloudflare capabilities through a practical AI documentation assistant. Users can ask questions about Cloudflare products and receive intelligent responses powered by multiple AI models.

### Available Clients

- **[Web Interface](documentation/cfhelperworker.md)** - Direct access through your browser
- **[Docker Container](documentation/cfhelperdocker.md)** - Local web server with enhanced features ([Docker Hub](https://hub.docker.com/r/schenanigans/cfhelper))
- **[Native Apps](documentation/cfhelpermac.md)** - macOS, iOS, and iPadOS applications

## Architecture

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
                     |  | Zero Trust Access|  | <-- Service Auth Tokens
                     |  | (CF-Access-*)    |  |     (Client ID/Secret)
                     |  +------------------+  |
                     |  |    Turnstile     |  | <-- Bot Protection
                     |  | (Web clients only)|  |     (Bypassed for native)
                     |  +------------------+  |
                     +-----------+------------+
                                 |
                     +-----------v------------+
                     |   CF Helper Worker     |
                     |  +------------------+  |
                     |  | Durable Objects  |  | <-- WebSocket Sessions
                     |  | (Chat Sessions)  |  |     Real-time messaging
                     |  +------------------+  |
                     |  +------------------+  |
                     |  |   KV Storage     |  | <-- Model Configuration
                     |  | (Models JSON)    |  |     Dynamic model list
                     |  +------------------+  |
                     |  +------------------+  |
                     |  |  Workers AI      |  | <-- Cloudflare Models
                     |  |    Binding       |  |     Native AI execution
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
                     |  +------------------+  |
                     |  |      MCP          |  | <-- Documentation Search
                     |  | (Model Context)  |  |     Cloudflare docs
                     |  +------------------+  |
                     +------------------------+
```

### Cloudflare Products Used

- **Workers** - Edge compute runtime for the backend
- **Workers AI** - Native AI model execution
- **AI Gateway** - Unified API for external AI providers
- **Durable Objects** - Stateful WebSocket session management
- **KV Storage** - Distributed key-value store for configuration
- **Zero Trust Access** - Service authentication with tokens
- **Turnstile** - Bot protection for web clients
- **Transform Rules** - HTTP header manipulation for CSP removal
- **MCP (Model Context Protocol)** - Enhanced documentation search
- **mTLS** - Optional mutual TLS for enterprise Docker deployments

## Key Features

###  Multi-Model AI Support
- **Cloudflare Workers AI** - Native models (Llama, Qwen, DeepSeek)
- **OpenAI Integration** - GPT-4 and O4 models via AI Gateway
- **Anthropic Integration** - Claude Opus 4 and Sonnet 4 via AI Gateway
- **Dynamic Model Loading** - Models filtered by available API keys

###  Enterprise Security
- **Zero Trust Access** - Service auth tokens (CF-Access-Client-Id/Secret)
- **Turnstile Protection** - Bot prevention for web interface
- **Platform Authentication** - Native apps bypass Turnstile
- **No Data Persistence** - Privacy-first, no conversation storage

###  Real-time Communication
- **Durable Objects** - Stateful WebSocket session management
- **Live Streaming** - See AI responses as they're generated
- **PartyKit Protocol** - Efficient message routing
- **Multi-client Support** - Synchronized across all platforms

###  Global Performance
- **Edge Computing** - Runs on 300+ Cloudflare locations
- **Smart Routing** - Automatic latency optimization
- **KV Storage** - Globally distributed configuration
- **Auto-scaling** - Handles any traffic volume

## Links

- **Docker Hub**: [schenanigans/cfhelper](https://hub.docker.com/r/schenanigans/cfhelper)
- **Cloudflare MCP**: [MCP Server Cloudflare](https://github.com/cloudflare/mcp-server-cloudflare)

---

Built with ❤️ using Cloudflare's global network
