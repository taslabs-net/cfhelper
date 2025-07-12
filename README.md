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

A multi-platform AI assistant that provides intelligent access to Cloudflare documentation. Built on Cloudflare's global network using Workers AI, Durable Objects, KV storage, and Zero Trust Access.

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
                     |  |   KV Storage     |  | <-- Model Configuration
                     |  | (All Models JSON) |  |
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
                     |  +------------------+  |
                     |  |      MCP          |  | <-- Documentation Search
                     |  +------------------+  |
                     +------------------------+
```

## Key Features

###  Multi-Model AI Support
- Cloudflare Workers AI models
- OpenAI GPT models (via AI Gateway)
- Anthropic Claude models (via AI Gateway)
- Dynamic model selection based on availability

###  Enterprise Security
- Cloudflare Zero Trust Access protection
- Platform-specific authentication
- No data persistence (privacy-first)

###  Real-time Communication
- WebSocket connections via Durable Objects
- Live streaming responses
- Multi-client synchronization

###  Global Performance
- Runs on Cloudflare's global edge network
- Low latency responses worldwide
- Automatic scaling



## Documentation

- **[Worker Setup & Configuration](documentation/cfhelperworker.md)** - Backend deployment
- **[Docker Client Guide](documentation/cfhelperdocker.md)** - Local web server setup
- **[Native Apps Guide](documentation/cfhelpermac.md)** - macOS/iOS development
- **[Architecture Details](documentation/cfhelperoverview.md)** - Technical deep dive

## Links

- **Docker Hub**: [schenanigans/cfhelper](https://hub.docker.com/r/schenanigans/cfhelper)
- **Cloudflare MCP**: [MCP Server Cloudflare](https://github.com/cloudflare/mcp-server-cloudflare)

---

Tim
