# CF Helper Worker

The CF Helper Worker is the core backend service that powers all CF Helper clients. It's deployed as a Cloudflare Worker and provides AI capabilities, session management, and real-time communication.

**Related**: [CF Helper Docker Container](https://hub.docker.com/r/schenanigans/cfhelper)

## Overview

The Worker serves dual purposes:
1. **Backend API**: Handles AI requests from all client platforms
2. **Web Frontend**: Hosts a React-based web interface at https://cfhelper.taslabs.net

## Technical Architecture

### Core Components

- **Cloudflare Worker**: Edge compute runtime
- **Durable Objects**: WebSocket session management (class: `CFHelperChat`)
- **KV Storage**: Model configuration storage
- **Workers AI**: Native AI model execution
- **AI Gateway**: External AI provider integration
- **PartyKit**: WebSocket message routing protocol

### File Structure

```
worker/
├── src/
│   ├── server/
│   │   ├── index.ts      # Main worker & Durable Object
│   │   └── env.d.ts      # TypeScript environment types
│   ├── client/
│   │   └── index.tsx     # React web frontend
│   └── shared.ts         # Shared types
├── public/               # Static assets
├── wrangler.toml        # Cloudflare configuration
└── models.json          # Model configuration
```

## Features

### AI Model Support

The Worker supports three categories of AI models:

1. **Cloudflare Workers AI**
   - Native models via AI binding
   - No additional API keys required
   - Examples: Llama, Qwen, DeepSeek

2. **OpenAI Models**
   - Via OpenAI API with optional AI Gateway
   - Requires `OPENAI_API_KEY`
   - Examples: GPT-4, O4 Mini

3. **Anthropic Models**
   - Via Anthropic API with optional AI Gateway
   - Requires `ANTHROPIC_API_KEY`
   - Examples: Claude Opus 4, Claude Sonnet 4

### Model Management Architecture

The Worker implements a centralized model management system where KV serves as the single source of truth:

1. **KV Storage as Source of Truth**
   - Complete model catalog stored in `Models` key
   - JSON array contains all possible models across providers
   - Updated independently of Worker code

2. **Dynamic Filtering Process**
   ```typescript
   // Fetch complete model list from KV
   const modelsJson = await env.cfhelper_kv.get('Models');
   const allModels = JSON.parse(modelsJson);

   // Filter based on configured API keys
   const availableModels = allModels.filter(model => {
     // Cloudflare models always available
     if (model.provider === 'cloudflare') return true;
     
     // External models require API keys
     if (model.requiresKey) return !!env[model.requiresKey];
     
     return false;
   });
   ```

3. **Unified API Endpoint**
   - All clients use `/cfhelper/api/models`
   - Returns filtered list based on server configuration
   - No client-side model hardcoding

### Platform-Specific Handling

The Worker detects client platforms and applies appropriate logic:

```typescript
// Apple platform bypasses Turnstile
const isApplePlatform = parsed.platform === "apple";
if (isApplePlatform) {
  console.log('[TURNSTILE] Skipping verification for apple platform');
}
```

### MCP Integration

The Worker integrates with Cloudflare's Model Context Protocol (MCP) for enhanced documentation search:

```typescript
// MCP request for documentation search
const mcpResponse = await fetch('https://docs.mcp.cloudflare.com/sse', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    sessionId: sessionId,
    messages: [{
      method: "search_cloudflare_docs",
      params: { query: query }
    }]
  })
});
```

## Configuration

### wrangler.toml

```toml
name = "cfhelper"
main = "src/server/index.ts"
compatibility_date = "2025-07-11"

[ai]
binding = "AI"

[[kv_namespaces]]
binding = "cfhelper_kv"
id = "7ddb6480da3d497eb30b6e3489db4c98"

[[durable_objects.bindings]]
name = "CFHelperChat"
class_name = "CFHelperChat"

[vars]
CLOUDFLARE_OPENAI_GATEWAY = "https://gateway.ai.cloudflare.com/v1/.../openai"
CLOUDFLARE_ANTHROPIC_GATEWAY = "https://gateway.ai.cloudflare.com/v1/.../anthropic"
```

### Environment Secrets

Set via `wrangler secret put`:
- `CF_ACCESS_CLIENT_ID`
- `CF_ACCESS_CLIENT_SECRET`
- `ANTHROPIC_API_KEY`
- `OPENAI_API_KEY`

### KV Model Configuration

Upload models to KV:
```bash
npx wrangler kv key put --namespace-id=... "Models" --path models.json
```

## Deployment

### Initial Setup

```bash
# Install dependencies
npm install

# Create KV namespace
npx wrangler kv namespace create cfhelper-kv

# Set secrets
npx wrangler secret put CF_ACCESS_CLIENT_ID
npx wrangler secret put CF_ACCESS_CLIENT_SECRET

# Deploy
npm run deploy
```

### Model Management

1. Edit `models.json` with available models
2. Upload to KV: `npx wrangler kv key put "Models" --path models.json --remote`
3. Models automatically filter based on configured API keys

### Custom Domain

Configure custom domain in Cloudflare dashboard:
- Route: `cfhelper.taslabs.net/*`
- Worker: `cfhelper`

### Transform Rules (Required)

The Worker serves a React frontend that may conflict with CSP headers. Create an HTTP Response Header Transform Rule:

1. Go to Rules → Transform Rules → HTTP Response Header Modification
2. Create new rule:
   - **Name**: Remove CSP headers for cfhelper worker
   - **If**: `(http.host eq "cfhelper.taslabs.net")`
   - **Then**: Remove headers:
     - `Content-Security-Policy`
     - `Content-Security-Policy-Report-Only`
3. Deploy the rule

This prevents CSP conflicts with the React application served by the Worker.

## API Reference

### GET /cfhelper/api/models
Returns available AI models filtered by API keys.

**Response:**
```json
{
  "models": [
    {
      "id": "@cf/meta/llama-4-scout-17b-16e-instruct",
      "name": "Llama 4 Scout (17B)",
      "description": "Meta's multimodal model"
    }
  ]
}
```

### POST /cfhelper/api/docker
Processes queries from Docker clients.

**Request:**
```json
{
  "query": "What is Cloudflare Workers?",
  "model": "@cf/meta/llama-4-scout-17b-16e-instruct"
}
```

### POST /cfhelper/api/apple
Processes queries from Apple clients (bypasses Turnstile).

**Request:**
```json
{
  "query": "Explain Durable Objects",
  "model": "@cf/meta/llama-4-scout-17b-16e-instruct",
  "platform": "apple"
}
```

### GET /cfhelper/api/health
Returns service health and available models.

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-07-12T14:55:00Z",
  "service": "cfhelper-worker",
  "requestId": "uuid",
  "endpoints": [
    "/cfhelper/api/models",
    "/cfhelper/api/docker",
    "/cfhelper/api/apple",
    "/cfhelper/api/query",
    "/cfhelper/api/health"
  ],
  "models": ["@cf/meta/llama-4-scout-17b-16e-instruct"]
}
```

## Monitoring

### Request Tracking

All requests include tracking headers:
```
X-Client-Type: cf-helper-worker | cf-helper-docker | cf-helper-apple
```

### Logging

Comprehensive logging with request IDs:
```
[WORKER] [uuid] Request received
[WORKER] [uuid] X-Client-Type: cf-helper-docker
[WORKER] [uuid] Routing to Docker API handler
```

### Tail Workers

Configure log aggregation:
```toml
[tail]
consumers = [{ service = "cfhelper-tail" }]
```

## Building and Deployment

### Building and Deploying

```bash
# Build frontend
npm run build

# Deploy to Cloudflare
npm run deploy

# Preview deployment
npx wrangler dev
```

### Testing Models

```bash
# Test model endpoint
curl https://cfhelper.taslabs.net/cfhelper/api/models \
  -H "CF-Access-Client-Id: YOUR_ID" \
  -H "CF-Access-Client-Secret: YOUR_SECRET"
```

## Security

- Zero Trust Access protection on all endpoints
- Turnstile verification for web clients
- Platform-specific bypasses for native apps
- API keys stored as encrypted secrets
- No persistent storage of conversations

## Troubleshooting

### Models Not Showing
1. Check KV has "Models" key: `wrangler kv key list`
2. Verify API keys are set: `wrangler secret list`
3. Check Worker logs for errors

### WebSocket Connection Failed
1. Verify Durable Object binding in wrangler.toml
2. Check CF-Access credentials
3. Ensure correct WebSocket URL format

### AI Provider Errors
1. Verify API keys are correctly set
2. Check AI Gateway configuration
3. Review Worker logs for specific errors
