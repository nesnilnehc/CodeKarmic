# Configuration API

<cite>
**Referenced Files in This Document**
- [package.json](file://package.json)
- [src/config/appConfig.ts](file://src/config/appConfig.ts)
- [src/models/types.ts](file://src/models/types.ts)
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts)
- [src/services/git/gitService.ts](file://src/services/git/gitService.ts)
- [src/extension.ts](file://src/extension.ts)
- [src/models/modelFactory.ts](file://src/models/modelFactory.ts)
- [src/models/modelValidator.ts](file://src/models/modelValidator.ts)
- [src/constants/constants.ts](file://src/constants/constants.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Configuration Architecture](#configuration-architecture)
3. [AppConfig Class Overview](#appconfig-class-overview)
4. [Configuration Properties](#configuration-properties)
5. [Configuration Schema](#configuration-schema)
6. [Accessing Configuration Values](#accessing-configuration-values)
7. [Event-Driven Configuration Updates](#event-driven-configuration-updates)
8. [Integration with Services](#integration-with-services)
9. [Security and Validation](#security-and-validation)
10. [Best Practices](#best-practices)
11. [Troubleshooting](#troubleshooting)

## Introduction

CodeKarmic implements a centralized configuration management system through the `AppConfig` class, which provides unified access to all runtime settings. The configuration system is built on VS Code's native configuration API and offers type-safe access to application settings with automatic persistence and event-driven updates.

The configuration system manages critical settings including API keys, model selection, file size limits, exclusion patterns, and debugging options. It ensures secure handling of sensitive data while providing seamless integration across all application services.

## Configuration Architecture

The configuration system follows a singleton pattern with event-driven architecture for real-time configuration updates:

```mermaid
classDiagram
class AppConfig {
-instance : AppConfig
-emitter : EventEmitter
-extension : string
+getInstance() : AppConfig
+get(key : ConfigKey) : T
+set(key : ConfigKey, value : T, global : boolean) : Promise~void~
+onChange(event : ConfigChangeEvent, listener : Function) : void
+offChange(event : ConfigChangeEvent, listener : Function) : void
+getApiKey() : string
+setApiKey(apiKey : string) : Promise~void~
+getModelType() : ModelType
+setModelType(modelType : ModelType) : Promise~void~
+getLanguage() : Language
+setLanguage(language : Language) : Promise~void~
}
class ConfigKey {
<<enumeration>>
LANGUAGE
API_KEY
BASE_URL
MODEL_TYPE
}
class ConfigChangeEvent {
<<enumeration>>
LANGUAGE
API_KEY
BASE_URL
MODEL_TYPE
ANY
}
class ModelType {
<<enumeration>>
DEEPSEEK_V3
DEEPSEEK_R1
OPENAI
}
AppConfig --> ConfigKey : uses
AppConfig --> ConfigChangeEvent : emits
AppConfig --> ModelType : validates
```

**Diagram sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L22-L189)
- [src/models/types.ts](file://src/models/types.ts#L10-L14)

**Section sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L49-L189)

## AppConfig Class Overview

The `AppConfig` class serves as the central hub for all application configuration management. It implements a singleton pattern to ensure consistent configuration access across the entire application.

### Key Features

- **Singleton Pattern**: Ensures only one configuration instance exists
- **Event-Driven Updates**: Automatically notifies subscribers of configuration changes
- **Type Safety**: Provides strongly-typed access to configuration values
- **VS Code Integration**: Leverages VS Code's configuration persistence
- **Default Value Management**: Falls back to predefined defaults when values are missing

### Initialization Process

The configuration system initializes automatically when the extension activates:

```mermaid
sequenceDiagram
participant Extension as Extension Activation
participant AppConfig as AppConfig
participant VSCode as VS Code API
participant Services as Application Services
Extension->>AppConfig : getInstance()
AppConfig->>AppConfig : Create singleton instance
AppConfig->>VSCode : Setup configuration listener
Extension->>AppConfig : Get initial values
AppConfig->>VSCode : workspace.getConfiguration()
VSCode-->>AppConfig : Configuration values
AppConfig-->>Extension : Ready for use
Extension->>Services : Initialize with config
```

**Diagram sources**
- [src/extension.ts](file://src/extension.ts#L20-L40)
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L54-L77)

**Section sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L54-L88)

## Configuration Properties

The configuration system manages four primary categories of settings, each serving specific functionality within the application.

### Core Configuration Keys

| Property | Type | Purpose | Default Value |
|----------|------|---------|---------------|
| `language` | `string` | User interface language preference | `'ENGLISH'` |
| `apiKey` | `string` | Authentication credentials for AI services | `''` |
| `baseUrl` | `string` | Endpoint URL for AI model provider | `'https://api.deepseek.com/v1'` |
| `modelType` | `string` | Selected AI model for code analysis | `'deepseek-reasoner'` |

### Advanced Configuration Options

| Property | Type | Purpose | Default Value |
|----------|------|---------|---------------|
| `debugMode` | `boolean` | Enable detailed logging and debugging | `false` |
| `maxFileSizeKb` | `number` | Maximum file size for code review (KB) | `100` |
| `excludeFileTypes` | `string[]` | File patterns to exclude from analysis | Comprehensive list |

**Section sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L37-L42)
- [package.json](file://package.json#L121-L207)

## Configuration Schema

The configuration schema defines the structure and validation rules for all settings through VS Code's contribution points in `package.json`.

### Schema Definition

The configuration schema provides comprehensive metadata for each setting:

```mermaid
erDiagram
CODEKARMIC_CONFIG {
string apiKey "API key for AI service"
boolean debugMode "Enable debug mode"
string modelType "AI model to use for code review"
string openaiHost "OpenAI API host"
number maxFileSizeKb "Maximum file size in KB"
array excludeFileTypes "File types to exclude from code review"
}
MODEL_TYPES {
string gpt_3_5_turbo "GPT-3.5 Turbo model"
string gpt_4_turbo "GPT-4 Turbo model"
string gpt_4 "GPT-4 model"
}
EXCLUDE_PATTERNS {
string image_files "*.png, *.jpg, *.jpeg, *.gif, *.bmp, *.ico, *.svg"
string archive_files "*.zip, *.tar, *.gz, *.rar, *.7z"
string executable_files "*.exe, *.dll, *.bin, *.class"
string build_directories "node_modules/**, .git/**, dist/**"
}
CODEKARMIC_CONFIG ||--|| MODEL_TYPES : contains
CODEKARMIC_CONFIG ||--o{ EXCLUDE_PATTERNS : excludes
```

**Diagram sources**
- [package.json](file://package.json#L118-L209)

### UI Representation

The configuration schema enables VS Code to generate intuitive settings panels with:

- **Type Validation**: Automatic validation based on declared types
- **Enum Selection**: Dropdown menus for model selection
- **File Patterns**: Glob pattern support for exclusions
- **Scope Control**: Machine-wide vs workspace-specific settings

**Section sources**
- [package.json](file://package.json#L118-L209)

## Accessing Configuration Values

The configuration system provides multiple approaches for accessing and managing settings across different contexts.

### Basic Access Pattern

```mermaid
flowchart TD
A[Service Request] --> B{Configuration Needed?}
B --> |Yes| C[AppConfig.getInstance()]
C --> D[get&lt;Type&gt;(ConfigKey)]
D --> E{Value Available?}
E --> |Yes| F[Return Configuration Value]
E --> |No| G[Return Default Value]
F --> H[Use Configuration]
G --> H
B --> |No| I[Continue Operation]
```

**Diagram sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L95-L98)

### Service Integration Examples

#### AIService Configuration Access

The AI service demonstrates comprehensive configuration integration:

```mermaid
sequenceDiagram
participant AIService as AI Service
participant AppConfig as Config Manager
participant ModelFactory as Model Factory
participant Model as AI Model
AIService->>AppConfig : getInstance()
AppConfig-->>AIService : AppConfig instance
AIService->>AppConfig : getApiKey()
AppConfig-->>AIService : API key
AIService->>AppConfig : getModelType()
AppConfig-->>AIService : Model type
AIService->>ModelFactory : createModelService()
ModelFactory->>AppConfig : get configuration values
AppConfig-->>ModelFactory : Configuration data
ModelFactory->>Model : Initialize with config
Model-->>ModelFactory : Initialized service
ModelFactory-->>AIService : Model service
```

**Diagram sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L52-L61)
- [src/models/modelFactory.ts](file://src/models/modelFactory.ts#L58-L62)

#### GitService Configuration Access

The Git service shows how configuration affects service behavior:

```mermaid
flowchart LR
A[GitService] --> B[AppConfig.getInstance()]
B --> C[getMaxFileSizeKb]
B --> D[getExcludePatterns]
C --> E[Apply file size limits]
D --> F[Filter excluded files]
E --> G[Process files]
F --> G
```

**Diagram sources**
- [src/services/git/gitService.ts](file://src/services/git/gitService.ts#L46-L52)

**Section sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L52-L61)
- [src/services/git/gitService.ts](file://src/services/git/gitService.ts#L46-L52)

## Event-Driven Configuration Updates

The configuration system implements a sophisticated event-driven architecture that enables real-time updates across all application components.

### Event Flow Architecture

```mermaid
sequenceDiagram
participant VSCode as VS Code Settings
participant AppConfig as Config Manager
participant Listener1 as Service 1
participant Listener2 as Service 2
participant ListenerN as Service N
VSCode->>AppConfig : Configuration Changed
AppConfig->>AppConfig : Detect affected keys
AppConfig->>Listener1 : Emit ConfigChangeEvent
AppConfig->>Listener2 : Emit ConfigChangeEvent
AppConfig->>ListenerN : Emit ConfigChangeEvent
AppConfig->>Listener1 : Emit ConfigChangeEvent.ANY
AppConfig->>Listener2 : Emit ConfigChangeEvent.ANY
AppConfig->>ListenerN : Emit ConfigChangeEvent.ANY
Listener1->>Listener1 : Update internal state
Listener2->>Listener2 : Update internal state
ListenerN->>ListenerN : Update internal state
```

**Diagram sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L58-L76)

### Event Types and Handlers

The system supports multiple event types for granular control:

| Event Type | Trigger Condition | Use Case |
|------------|-------------------|----------|
| `LANGUAGE` | Language setting changed | Update UI localization |
| `API_KEY` | Authentication credentials updated | Reinitialize services |
| `BASE_URL` | Model endpoint changed | Update API connections |
| `MODEL_TYPE` | AI model selection changed | Switch model providers |
| `ANY` | Any configuration change | General state synchronization |

### Registration Pattern

Services register for configuration changes using the observer pattern:

```mermaid
classDiagram
class ConfigurableService {
+initializeWithConfig()
+onConfigChange()
+dispose()
}
class AppConfig {
+onChange(event : ConfigChangeEvent, listener : Function)
+offChange(event : ConfigChangeEvent, listener : Function)
}
ConfigurableService --> AppConfig : subscribes to
AppConfig --> ConfigurableService : notifies on changes
```

**Diagram sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L117-L128)

**Section sources**
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L58-L76)
- [src/config/appConfig.ts](file://src/config/appConfig.ts#L117-L128)

## Integration with Services

The configuration system seamlessly integrates with all major application services, providing consistent access patterns and automatic updates.

### Service-Specific Configuration Patterns

#### AI Service Configuration

The AI service demonstrates comprehensive configuration integration:

```mermaid
flowchart TD
A[AI Service Startup] --> B[Get API Key]
B --> C{Valid API Key?}
C --> |Yes| D[Initialize Model Service]
C --> |No| E[Wait for Configuration]
D --> F[Set Model Type]
F --> G[Configure Request Options]
G --> H[Ready for Analysis]
E --> I[Listen for API Key Changes]
I --> J[Reinitialize on Change]
```

**Diagram sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L52-L61)

#### Model Factory Configuration

The model factory shows how configuration drives service creation:

```mermaid
sequenceDiagram
participant Factory as Model Factory
participant Config as AppConfig
participant Service as Model Service
Factory->>Config : Get model type
Factory->>Config : Get base URL
Factory->>Config : Get API key
Config-->>Factory : Configuration values
Factory->>Service : Create with config
Service-->>Factory : Initialized service
Factory->>Factory : Cache service instance
```

**Diagram sources**
- [src/models/modelFactory.ts](file://src/models/modelFactory.ts#L58-L62)

### Cross-Service Synchronization

Services maintain consistency through shared configuration access:

```mermaid
graph TB
A[AppConfig Singleton] --> B[AI Service]
A --> C[Git Service]
A --> D[Review Manager]
A --> E[Notification Manager]
B --> F[Model Selection]
C --> G[File Filtering]
D --> H[Analysis Settings]
E --> I[Logging Level]
F --> J[Consistent Behavior]
G --> J
H --> J
I --> J
```

**Diagram sources**
- [src/extension.ts](file://src/extension.ts#L20-L40)

**Section sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L52-L61)
- [src/models/modelFactory.ts](file://src/models/modelFactory.ts#L58-L62)

## Security and Validation

The configuration system implements robust security measures and validation mechanisms to protect sensitive data and ensure system stability.

### API Key Security

The system handles API keys with special security considerations:

```mermaid
flowchart TD
A[API Key Input] --> B{Validation Required?}
B --> |Yes| C[Validate Against Provider]
B --> |No| D[Store Securely]
C --> E{Valid?}
E --> |Yes| D
E --> |No| F[Reject and Notify]
D --> G[Cache in Memory]
G --> H[Clear on Session End]
F --> I[Log Security Event]
```

**Diagram sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L712-L723)

### Configuration Validation

The system implements multiple validation layers:

| Validation Layer | Purpose | Implementation |
|------------------|---------|----------------|
| **Schema Validation** | Type checking | VS Code configuration schema |
| **Business Logic** | Semantic validation | ModelValidator class |
| **Runtime Validation** | Operational checks | Service-specific validation |
| **Security Validation** | Sensitive data protection | API key validation |

### Secure Storage Practices

The configuration system follows VS Code's security guidelines:

- **Memory-only Storage**: API keys stored in memory, not persistent storage
- **Scope Limitation**: Machine-scoped storage for sensitive data
- **Encryption**: VS Code handles encryption of stored values
- **Access Control**: Restricted access through singleton pattern

**Section sources**
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L712-L723)
- [src/models/modelValidator.ts](file://src/models/modelValidator.ts#L4-L14)

## Best Practices

### Configuration Management Guidelines

1. **Use Type-Safe Access**: Always use the typed getters for configuration values
2. **Handle Defaults Gracefully**: Rely on default values for optional settings
3. **Subscribe to Changes**: Register for configuration change events when needed
4. **Validate Early**: Check configuration validity during service initialization
5. **Clean Up Resources**: Dispose of listeners and cached resources appropriately

### Service Integration Patterns

```mermaid
flowchart TD
A[Service Initialization] --> B[Get Required Config]
B --> C[Validate Configuration]
C --> D{Valid?}
D --> |Yes| E[Initialize Service]
D --> |No| F[Use Defaults or Fail]
E --> G[Register for Changes]
G --> H[Ready for Use]
F --> I[Log Warning/Error]
I --> J[Fallback Behavior]
```

### Error Handling Strategies

The system implements comprehensive error handling:

- **Graceful Degradation**: Services continue operating with reduced functionality
- **User Notification**: Clear error messages for configuration issues
- **Fallback Mechanisms**: Default values prevent system failures
- **Logging Integration**: Detailed logs for debugging and monitoring

**Section sources**
- [src/extension.ts](file://src/extension.ts#L37-L66)
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L691-L710)

## Troubleshooting

### Common Configuration Issues

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Missing API Key** | AI service fails to initialize | Configure API key through settings |
| **Invalid Model Type** | Service throws validation error | Select supported model from dropdown |
| **File Size Limits** | Large files not processed | Adjust `maxFileSizeKb` setting |
| **Exclusion Patterns** | Unexpected files included | Review and modify exclusion patterns |

### Diagnostic Commands

The extension provides several diagnostic capabilities:

```mermaid
flowchart LR
A[Diagnostic Request] --> B{Issue Type}
B --> |Configuration| C[Check Settings]
B --> |Connectivity| D[Test API Access]
B --> |Performance| E[Monitor Resource Usage]
C --> F[Validate Configuration]
D --> G[Check Network Connectivity]
E --> H[Profile Service Performance]
F --> I[Generate Report]
G --> I
H --> I
```

### Debug Mode Features

When debug mode is enabled, the system provides enhanced logging:

- **Detailed API Calls**: Complete request/response logging
- **Configuration Tracing**: Real-time configuration change tracking
- **Performance Metrics**: Timing information for service operations
- **Error Context**: Enhanced error messages with stack traces

**Section sources**
- [src/extension.ts](file://src/extension.ts#L37-L66)
- [src/services/ai/aiService.ts](file://src/services/ai/aiService.ts#L691-L710)