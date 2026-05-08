# UserSettings Plugin Architecture

## Overview

The UserSettings plugin is a Thunder/WPEFramework-based service that provides a centralized management system for user preferences and settings in RDK-based devices. It exposes JSON-RPC APIs for applications to retrieve and modify user settings while persisting data through the PersistentStore plugin.

## System Architecture

### Component Structure

```
┌─────────────────────────────────────────────────────────┐
│                  Client Applications                     │
│          (Web Apps, Native Apps, Services)              │
└────────────────────┬────────────────────────────────────┘
                     │ JSON-RPC API
                     ▼
┌─────────────────────────────────────────────────────────┐
│              UserSettings Plugin                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │         UserSettings (JSONRPC Handler)           │   │
│  │  - Manages JSON-RPC API endpoints                │   │
│  │  - Validates input parameters                    │   │
│  │  - Forwards requests to implementation           │   │
│  └────────────────┬────────────────────────────────┘   │
│                   │                                      │
│  ┌────────────────▼────────────────────────────────┐   │
│  │    UserSettingsImplementation                    │   │
│  │  - Business logic and data management            │   │
│  │  - Default value handling                        │   │
│  │  - Event notification management                 │   │
│  │  - Interface: IUserSettings, IUserSettingsInspector │
│  └────────────────┬────────────────────────────────┘   │
└───────────────────┼────────────────────────────────────┘
                    │
                    ▼ COM-RPC Interface
┌─────────────────────────────────────────────────────────┐
│             PersistentStore Plugin                       │
│  - Key-value storage (IStore2 interface)                │
│  - Change notifications                                  │
│  - Persistent data management                            │
└─────────────────────────────────────────────────────────┘
```

### Key Components

1. **UserSettings Class** (UserSettings.h/cpp)
   - Inherits from `PluginHost::IPlugin` and `PluginHost::JSONRPC`
   - Implements JSON-RPC endpoint handlers
   - Manages plugin lifecycle (Initialize, Deinitialize)
   - Handles client connection notifications

2. **UserSettingsImplementation Class** (UserSettingsImplementation.h/cpp)
   - Implements `IUserSettings`, `IUserSettingsInspector`, and `IConfiguration` interfaces
   - Manages settings storage and retrieval via PersistentStore
   - Maintains default values for all settings
   - Broadcasts change notifications to registered clients
   - Provides singleton instance for internal access

3. **Notification System**
   - Two-tier notification architecture:
     - Store notifications: Listens to PersistentStore changes
     - Client notifications: Broadcasts changes to registered clients
   - Event-driven architecture ensures consistency across components
   - Change events include: `OnAudioDescriptionChanged`, `OnCaptionsChanged`,
     `OnVoiceGuidanceChanged`, `OnScreenReaderSpeedChanged`, and others — one per setting

## Data Flow

### Setting Retrieval Flow
```
Client Request → JSON-RPC Handler → UserSettingsImplementation 
→ PersistentStore (IStore2) → Return Value or Default
```

### Setting Update Flow
```
Client Update → JSON-RPC Handler → UserSettingsImplementation 
→ Validation → PersistentStore (IStore2) → Store Notification 
→ Broadcast to Registered Clients
```

## Plugin Framework Integration

### Thunder Framework Integration
- Uses Thunder's COM-RPC mechanism for inter-plugin communication
- Implements standard Thunder plugin interfaces (`IPlugin`, `JSONRPC`)
- Follows Thunder configuration and lifecycle management
- Leverages Thunder's security token mechanism (when enabled)

### Dependencies
- **PersistentStore Plugin**: Required for data persistence (queried via `org.rdk.PersistentStore` callsign)
- **Thunder Core**: Provides COM-RPC, JSONRPC, and plugin infrastructure
- **Thunder Interfaces**: Uses standardized interfaces (IUserSettings, IStore2)

### Configuration
- Plugin name: `UserSettings`
- Namespace: `UserSettings`
- Config file: `UserSettings.config`
- Startup order: 51 (configurable)
- Auto-start: Enabled by default

## Technical Implementation

### Settings Management
The plugin manages 19 different user settings categorized into:

1. **Audio Settings**
   - Audio description (boolean)
   - Preferred audio languages (string)

2. **Caption Settings**
   - Captions enabled (boolean)
   - Preferred caption languages (string)
   - Closed caption service preference (string)

3. **Parental Control Settings**
   - PIN control (boolean)
   - Viewing restrictions (string)
   - Viewing restrictions window (string)
   - Live/playback watershed (boolean)
   - Block not-rated content (boolean)
   - PIN on purchase (boolean)
   - Content PIN (string)

4. **Accessibility Settings**
   - Presentation language (string)
   - High contrast mode (boolean)
   - Voice guidance enabled (boolean)
   - Voice guidance rate (numeric: 0.1–10)
   - Voice guidance hints (boolean)
   - Screen reader speed (uint8_t) — storage key: `screenReaderSpeed`; fires `OnScreenReaderSpeedChanged` notification on change

5. **Privacy Settings**
   - Privacy mode (string)

### Thread Safety
- Uses mutex locks (`_adminLock`) to protect shared data structures
- Thread-safe notification registration and broadcasting
- Atomic operations for client list management

### Error Handling
- Returns Thunder error codes (`Core::ERROR_*`)
- Validates input parameters before processing
- Gracefully handles PersistentStore unavailability
- Default value fallback mechanism

## Inspector Interface

The UserSettingsInspector interface (`IUserSettingsInspector`) provides:
- Bulk retrieval of multiple settings in a single call
- Efficient querying for system monitoring and diagnostics
- Reduced API overhead for applications needing multiple settings

## Build System

### CMake Configuration
- Plugin library: `WPEFrameworkUserSettings.so`
- Implementation library: `WPEFrameworkUserSettingsImplementation.so`
- C++11 standard compliance
- Integrates with Thunder's build system via `write_config()`

### Helper Dependencies
- `UtilsJsonRpc.h`: JSON-RPC utility macros
- `UtilsLogging.h`: Logging infrastructure

## Testing Infrastructure

### L1 Unit Tests
- Component-level testing of implementation logic
- Mock-based testing framework
- Code coverage tracking via lcov

### L2 Integration Tests
- End-to-end API testing
- Multi-component interaction validation
- Workflow and scenario-based testing
