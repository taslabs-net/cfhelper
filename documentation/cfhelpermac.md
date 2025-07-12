# CF Helper Native Apps

CF Helper Native Apps provide platform-specific experiences for macOS, iOS, and iPadOS. Built with SwiftUI, these apps connect directly to the CF Helper Worker backend, offering native performance and features.

**Platforms**: macOS 15.0+, iOS 18.0+, iPadOS 18.0+

## Why Native Apps?

While the Worker provides a web interface and Docker offers a containerized solution, native apps deliver:

- **Native Performance** - Optimized for Apple Silicon and iOS devices
- **Platform Integration** - Uses system features like secure storage and notifications
- **Offline Capability** - Configuration persists across sessions
- **Native UI/UX** - Follows Apple's Human Interface Guidelines
- **Direct Connection** - No intermediate proxy needed

## How It Works with the Worker

The native apps communicate directly with the CF Helper Worker:

```
[Native App] → [CF Helper Worker] → [AI Models]
     ↓               ↓                   ↓
SwiftUI       WebSocket/HTTPS      Cloudflare AI
URLSession    Durable Objects      OpenAI/Anthropic
```

1. **Configuration** - Store CF-Access credentials locally
2. **Authentication** - Send credentials with each request
3. **Platform Detection** - Identify as "apple" to bypass Turnstile
4. **Direct Connection** - WebSocket connects straight to Worker
5. **Real-time Chat** - Messages stream through Durable Objects

## Key Features

### Universal App Design

**macOS**:
- Sidebar with model selection and configuration
- Resizable window with persistent layout
- Native menu bar integration
- Keyboard shortcuts for common actions

**iOS/iPadOS**:
- Adaptive layout for all screen sizes
- Sheet-based settings interface
- Gesture support and haptic feedback
- Split view support on iPad

### Real-time Communication

- **WebSocket Connection** - Direct connection to Worker's Durable Objects
- **Streaming Responses** - See AI responses as they're generated
- **Connection Status** - Visual indicators for connection state
- **Auto-reconnect** - Handles network interruptions gracefully

### Smart Features

- **Prompt Suggestions** - Quick-start prompts when chat is empty
- **Thinking Indicator** - Shows when AI is processing
- **HTML Rendering** - Rich text responses with proper formatting
- **Copy Support** - Long-press to copy messages
- **Model Switching** - Change AI models on the fly

## Getting Started

### Quick Setup

1. **Download/Build the App**
   - Clone the repository
   - Open in Xcode 15+
   - Build and run

2. **Configure on First Launch**
   - Enter your CF-Access credentials
   - Set your display name
   - Save configuration

3. **Start Chatting**
   - Select an AI model
   - Ask questions about Cloudflare
   - Get instant responses

### Configuration Required

```
Domain: cfhelper.taslabs.net
CF-Access-Client-Id: [Your Client ID]
CF-Access-Client-Secret: [Your Client Secret]
Username: [Your Display Name]
```

## User Interface

### macOS Interface

```
┌─────────────────────────────────────────┐
│ ┌─────────┬─────────────────────────┐  │
│ │         │                         │  │
│ │ Sidebar │     Chat Messages       │  │
│ │         │                         │  │
│ │ Models  │                         │  │
│ │ Config  │                         │  │
│ │         │ ┌─────────────────────┐ │  │
│ │         │ │   Message Input    │ │  │
│ └─────────┴─┴─────────────────────┴─┘  │
└─────────────────────────────────────────┘
```

- **Sidebar** - Always visible with model selection and config
- **Chat Area** - Full conversation history with scrolling
- **Input Field** - Type and send messages

### iOS/iPadOS Interface

```
┌─────────────────────┐
│  CF Helper    ⚙️    │  <- Navigation bar
├─────────────────────┤
│                     │
│   Chat Messages     │
│                     │
│                     │
├─────────────────────┤
│ Type a message...📤 │  <- Input area
└─────────────────────┘
```

- **Navigation Bar** - Model selection and settings
- **Chat View** - Full-screen conversation
- **Settings Sheet** - Slide-up configuration

### Prerequisites

- macOS 14.0 or later
- Xcode 15.0 or later
- Git

### Configuration Files

```
apple/Cf Helper/
├── ContentView.swift        # Main UI
├── ConfigurationView.swift  # Settings
├── WebSocketManager.swift   # Networking
├── ChatMessage.swift        # Data model
├── Info.plist              # App config
└── Cf_HelperApp.swift      # Entry point
```

## Technical Details

### Network Communication

**WebSocket Connection**:
```swift
// Direct connection to Worker
wss://cfhelper.taslabs.net/parties/chat/{roomId}

// Headers sent with connection
CF-Access-Client-Id: {your-id}
CF-Access-Client-Secret: {your-secret}
X-Client-Type: cf-helper-apple
```

**API Endpoints**:
```swift
// Fetch available models
GET https://cfhelper.taslabs.net/cfhelper/api/models

// Send queries (via WebSocket)
{
  "type": "add",
  "content": "user message",
  "model": "selected-model",
  "platform": "apple"
}
```

### Platform-Specific Code

```swift
// Cross-platform color handling
#if os(macOS)
Color(NSColor.controlBackgroundColor)
#else
Color(UIColor.systemBackground)
#endif

// Platform detection for Worker
"platform": "apple"  // Bypasses Turnstile
```

## Security & Privacy

### Data Storage

- **Credentials** - Stored in UserDefaults (consider Keychain for production)
- **Messages** - Only in memory during session
- **No Analytics** - No tracking or data collection
- **No Persistence** - Conversations not saved

### Network Security

- **TLS/WSS Only** - All connections encrypted
- **CF-Access** - Zero Trust authentication
- **No Proxies** - Direct connection to Worker
- **Platform Bypass** - Skip Turnstile for native apps

## Troubleshooting

### Common Issues

**"Unknown error connecting"**
- Check CF-Access credentials are correct
- Verify domain is cfhelper.taslabs.net
- Ensure network connectivity

**Models not loading**
- Credentials may be incorrect
- Worker may be unreachable
- Check Xcode console for errors

**Messages not appearing**
- WebSocket may have disconnected
- Check the connection indicator (green = connected)
- Try refreshing models

**Build errors in Xcode**
- Clean build folder (Cmd+Shift+K)
- Update to Xcode 15+
- Check deployment target matches your OS

### Debug Mode

Enable debug logging in `WebSocketManager.swift`:
```swift
print("[DEBUG] WebSocket state: \(state)")
print("[DEBUG] Received: \(text)")
```

## Distribution Options

### For Personal Use

1. **Build and Run** - Use Xcode to run on your devices
2. **Ad Hoc** - Share with up to 100 devices
3. **TestFlight** - Beta testing for teams

### For Organizations

1. **Enterprise Distribution** - Internal app stores
2. **MDM Deployment** - Managed device distribution
3. **Direct Installation** - Signed and notarized apps

### App Store (Future)

Requirements for public distribution:
- App Store Connect account
- App review compliance
- Privacy policy
- Terms of service

## Contributing

The native apps are part of the CF Helper ecosystem. To contribute:

1. Fork the repository
2. Create a feature branch
3. Test on all platforms
4. Submit a pull request

### Code Style

- Follow SwiftUI best practices
- Use Swift's async/await where appropriate
- Add comments for complex logic
- Test on both macOS and iOS
