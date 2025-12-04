# AI Service

<cite>
**Referenced Files in This Document**
- [aiService.ts](file://src/services/ai/aiService.ts)
- [modelFactory.ts](file://src/models/modelFactory.ts)
- [modelInterface.ts](file://src/models/modelInterface.ts)
- [appConfig.ts](file://src/config/appConfig.ts)
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts)
- [notificationManager.ts](file://src/services/notification/notificationManager.ts)
- [logger.ts](file://src/utils/logger.ts)
- [retryUtils.ts](file://src/utils/retryUtils.ts)
- [prompts.ts](file://src/i18n/en/prompts.ts)
- [types.ts](file://src/models/types.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Core Components](#core-components)
4. [API Key Validation](#api-key-validation)
5. [Prompt Construction and Formatting](#prompt-construction-and-formatting)
6. [Streaming Response Management](#streaming-response-management)
7. [Integration with ModelFactory](#integration-with-modelfactory)
8. [Suggestion Processing Pipeline](#suggestion-processing-pipeline)
9. [Error Handling and Rate Limiting](#error-handling-and-rate-limiting)
10. [Performance Considerations](#performance-considerations)
11. [Security Aspects](#security-aspects)
12. [Configuration Management](#configuration-management)
13. [Troubleshooting Guide](#troubleshooting-guide)

## Introduction

The AIService serves as the central orchestrator for AI model interactions in CodeKarmic, providing a comprehensive framework for code review automation. It manages the complete lifecycle of AI-powered code analysis, from API key validation and prompt construction to response processing and suggestion generation. The service acts as a bridge between the VS Code extension and various AI model providers, offering seamless integration with multiple model types while maintaining consistent interfaces and robust error handling.

The AIService implements sophisticated caching mechanisms, intelligent retry logic, and performance optimization strategies to ensure reliable and efficient code review operations. It supports both real-time streaming responses and batch processing for large-scale code analysis scenarios.

## Architecture Overview

The AIService follows a layered architecture pattern that separates concerns and promotes modularity:

```mermaid
graph TB
subgraph "VS Code Extension Layer"
UI[User Interface]
Commands[Commands & Actions]
end
subgraph "AI Service Layer"
AIS[AIService]
Config[AppConfig]
Notif[NotificationManager]
end
subgraph "Model Abstraction Layer"
MF[ModelFactory]
MS[ModelService]
MI[ModelInterface]
end
subgraph "Provider Layer"
DS[DeepSeek Provider]
OAI[OpenAI Provider]
end
subgraph "Processing Layer"
SG[SuggestionGenerator]
LP[LargeFileProcessor]
Compress[Compression]
end
subgraph "External Services"
AI[AI Models]
Git[Git APIs]
end
UI --> Commands
Commands --> AIS
AIS --> Config
AIS --> Notif
AIS --> MF
MF --> MS
MS --> MI
MI --> DS
MI --> OAI
AIS --> SG
AIS --> LP
LP --> Compress
AIS --> Git
DS --> AI
OAI --> AI
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L70)
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L44)
- [modelInterface.ts](file://src/models/modelInterface.ts#L39-L62)

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L70)
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L44)

## Core Components

### AIService Singleton

The AIService implements the Singleton pattern to ensure centralized management of AI model interactions:

```mermaid
classDiagram
class AIService {
-instance : AIService
-modelService : AIModelService
-client : OpenAI
-apiKey : string
-modelType : string
-gitService : GitService
-diffCache : Map
-largeFileProcessor : LargeFileProcessor
+getInstance() : AIService
+reviewCode(params) : Promise~CodeReviewResult~
+validateApiKey(apiKey) : Promise~boolean~
+setApiKey(apiKey) : void
+getModel() : string
-generateDiffContent(params) : Promise~string~
-performCodeAnalysis(params, diffContent, options) : Promise~CodeReviewResult~
-makeSpecificApiRequest(prompt, systemPrompt, useStreaming) : Promise~string~
-handleReviewError(error, filePath) : CodeReviewResult
}
class CodeReviewRequest {
+filePath : string
+currentContent : string
+previousContent : string
+useCompression? : boolean
+language? : string
+diffContent? : string
+includeDiffAnalysis? : boolean
+useStreamingOutput? : boolean
}
class CodeReviewResult {
+suggestions : string[]
+diffSuggestions? : string[]
+fullFileSuggestions? : string[]
+score? : number
+diffContent? : string
}
AIService --> CodeReviewRequest : processes
AIService --> CodeReviewResult : produces
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L15-L39)
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L70)

### Model Factory Integration

The AIService integrates with the ModelFactory to manage different AI model providers:

```mermaid
sequenceDiagram
participant Client as Client Code
participant AIS as AIService
participant MF as ModelFactory
participant MS as ModelService
participant Provider as AI Provider
Client->>AIS : reviewCode(params)
AIS->>MF : createModelService()
MF->>MS : new DeepSeekModelService()
MS->>Provider : initialize()
Provider-->>MS : initialized
MS-->>MF : service instance
MF-->>AIS : model service
AIS->>MS : createChatCompletion()
MS->>Provider : API request
Provider-->>MS : response
MS-->>AIS : AIModelResponse
AIS-->>Client : CodeReviewResult
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L51-L61)
- [modelFactory.ts](file://src/models/modelFactory.ts#L58-L110)

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L70)
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L140)

## API Key Validation

The AIService provides robust API key validation capabilities through the ModelFactory integration:

### Validation Process

The API key validation follows a structured approach:

```mermaid
flowchart TD
Start([API Key Validation Request]) --> GetFactory["Get ModelFactory Instance"]
GetFactory --> CreateService["Create Model Service"]
CreateService --> ValidateKey["Call modelService.validateApiKey()"]
ValidateKey --> Success{"Validation Success?"}
Success --> |Yes| CacheService["Cache Model Service"]
Success --> |No| LogError["Log Validation Error"]
CacheService --> ReturnTrue["Return true"]
LogError --> NotifyUser["Notify User"]
NotifyUser --> ReturnFalse["Return false"]
ReturnTrue --> End([Validation Complete])
ReturnFalse --> End
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L712-L723)

### Implementation Details

The validation process involves several key steps:

1. **Factory Initialization**: Creates a temporary model service instance
2. **API Testing**: Makes a lightweight API call to verify credentials
3. **Error Handling**: Comprehensive error logging and user notification
4. **Caching**: Stores validated services for future use

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L712-L723)
- [modelFactory.ts](file://src/models/modelFactory.ts#L58-L110)

## Prompt Construction and Formatting

The AIService constructs sophisticated prompts for code review analysis using a templating system:

### Prompt Templates

The system uses multiple prompt templates for different analysis scenarios:

| Template Type | Purpose | Key Features |
|---------------|---------|--------------|
| `SYSTEM_PROMPT` | General code review guidance | Structured analysis framework |
| `DIFF_PROMPT` | Change-focused analysis | Line-by-line comparison |
| `FULL_FILE_PROMPT` | Complete file analysis | Comprehensive structural review |
| `LARGE_FILE_PROMPT` | Large file compression | Summary-based analysis |
| `FINAL_PROMPT` | Consolidated suggestions | Combined analysis synthesis |

### Prompt Construction Process

```mermaid
flowchart TD
Start([Code Review Request]) --> CheckSize{"File Size > Threshold?"}
CheckSize --> |Yes| LargeFile["Use Large File Template"]
CheckSize --> |No| CheckDiff{"Include Diff Analysis?"}
CheckDiff --> |Yes| DiffTemplate["Apply Diff Template"]
CheckDiff --> |No| FullTemplate["Apply Full File Template"]
LargeFile --> BuildPrompt["Build Combined Prompt"]
DiffTemplate --> BuildPrompt
FullTemplate --> BuildPrompt
BuildPrompt --> AddSystem["Add System Role"]
AddSystem --> FormatMessages["Format Chat Messages"]
FormatMessages --> SendRequest["Send API Request"]
SendRequest --> End([Analysis Complete])
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L308-L332)
- [prompts.ts](file://src/i18n/en/prompts.ts#L10-L107)

### Authentication Headers

The AIService manages authentication headers through the underlying model services:

- **API Key Storage**: Secure storage in AppConfig
- **Header Management**: Automatic header injection for API requests
- **Token Refresh**: Intelligent token refresh mechanisms
- **Security Validation**: Multi-layered security checks

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L308-L332)
- [prompts.ts](file://src/i18n/en/prompts.ts#L10-L107)
- [appConfig.ts](file://src/config/appConfig.ts#L145-L156)

## Streaming Response Management

The AIService supports real-time streaming responses for improved user experience:

### Streaming Architecture

```mermaid
sequenceDiagram
participant Client as Client
participant AIS as AIService
participant MS as ModelService
participant AI as AI Provider
Client->>AIS : reviewCode(useStreamingOutput : true)
AIS->>MS : createChatCompletion(stream : true)
MS->>AI : Stream API Request
loop Streaming Response
AI-->>MS : Stream Chunk
MS-->>AIS : Process Chunk
AIS->>AIS : Update Progress
AIS-->>Client : Real-time Updates
end
AI-->>MS : Stream Complete
MS-->>AIS : Final Response
AIS-->>Client : Complete Analysis
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L269-L275)
- [aiService.ts](file://src/services/ai/aiService.ts#L766-L768)

### Stream Processing Features

1. **Real-time Updates**: Progressive response delivery
2. **Progress Tracking**: Visual progress indicators
3. **Error Recovery**: Graceful handling of stream interruptions
4. **Resource Management**: Efficient memory usage during streaming

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L269-L275)
- [aiService.ts](file://src/services/ai/aiService.ts#L766-L768)

## Integration with ModelFactory

The AIService leverages the ModelFactory for dynamic model service creation and management:

### Factory Pattern Implementation

```mermaid
classDiagram
class AIModelFactory {
<<interface>>
+createModelService() : AIModelService
+getSupportedModelTypes() : string[]
+clearModelServices(type?) : void
}
class AIModelFactoryImpl {
-instance : AIModelFactoryImpl
-modelServices : Map
-factoryConfig : ModelFactoryConfig
+getInstance(config?) : AIModelFactoryImpl
+createModelService() : AIModelService
+updateConfig(config) : void
+clearModelServices(type?) : void
}
class AIModelService {
<<interface>>
+initialize(options) : void
+validateApiKey(apiKey) : Promise~boolean~
+createChatCompletion(params) : Promise~AIModelResponse~
+getModelType() : string
}
class DeepSeekModelService {
+initialize(options) : void
+validateApiKey(apiKey) : Promise~boolean~
+createChatCompletion(params) : Promise~AIModelResponse~
+getModelType() : string
}
AIModelFactory <|-- AIModelFactoryImpl
AIModelFactoryImpl --> AIModelService : creates
AIModelService <|-- DeepSeekModelService
```

**Diagram sources**
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L44)
- [modelInterface.ts](file://src/models/modelInterface.ts#L39-L62)

### Service Caching Strategy

The ModelFactory implements intelligent caching to optimize performance:

- **Cache Key**: Combination of model type and base URL
- **Lifetime Management**: Automatic cache invalidation
- **Memory Optimization**: LRU-based cache eviction
- **Concurrent Access**: Thread-safe cache operations

**Section sources**
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L140)
- [modelInterface.ts](file://src/models/modelInterface.ts#L39-L62)

## Suggestion Processing Pipeline

The AIService integrates with the SuggestionGenerator for structured output processing:

### Processing Workflow

```mermaid
flowchart TD
AIResponse[AI Response] --> Extract["Extract Suggestions"]
Extract --> Parse["Parse Structured Output"]
Parse --> Categorize["Categorize by Type"]
Categorize --> Score["Calculate Quality Score"]
Score --> Format["Format for Display"]
Format --> Validate["Validate Results"]
Validate --> Final["Final Suggestions"]
subgraph "Suggestion Categories"
Structure[Structure]
Performance[Performance]
Security[Security]
Readability[Readability]
Maintainability[Maintainability]
BestPractice[Best Practice]
end
Categorize --> Structure
Categorize --> Performance
Categorize --> Security
Categorize --> Readability
Categorize --> Maintainability
Categorize --> BestPractice
```

**Diagram sources**
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L66-L106)
- [aiService.ts](file://src/services/ai/aiService.ts#L364-L386)

### Structured Output Format

The suggestion generator produces structured suggestions with metadata:

| Field | Type | Description |
|-------|------|-------------|
| `content` | string | The suggestion text |
| `lines` | number[] \| string | Related line numbers or ranges |
| `category` | SuggestionCategory | Classification type |
| `severity` | SuggestionSeverity | Importance level |
| `fixExample` | string | Code example for fixes |

**Section sources**
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L36-L50)
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L183-L248)

## Error Handling and Rate Limiting

The AIService implements comprehensive error handling and rate limiting strategies:

### Error Classification

```mermaid
flowchart TD
Error[API Error] --> Classify{"Error Type"}
Classify --> |Network| NetworkError["Network Timeout<br/>Connection Reset"]
Classify --> |Authentication| AuthError["Invalid API Key<br/>Rate Limit Exceeded"]
Classify --> |Server| ServerError["Internal Server Error<br/>Service Unavailable"]
Classify --> |Client| ClientError["Invalid Request<br/>Bad Request"]
NetworkError --> Retry["Exponential Backoff Retry"]
AuthError --> Fallback["Fallback to Alternative"]
ServerError --> Retry
ClientError --> Log["Log Error & Notify"]
Retry --> Success{"Success?"}
Success --> |Yes| Continue["Continue Processing"]
Success --> |No| Fallback
Fallback --> Alternative["Try Alternative Model"]
Alternative --> Success
Log --> UserNotify["Notify User"]
Continue --> End([Process Complete])
UserNotify --> End
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L388-L410)
- [retryUtils.ts](file://src/utils/retryUtils.ts#L33-L70)

### Retry Mechanisms

The system implements sophisticated retry logic:

- **Exponential Backoff**: Increasing delays between retries
- **Jitter**: Random variation to prevent thundering herd
- **Circuit Breaker**: Temporary service suspension after failures
- **Timeout Management**: Configurable request timeouts

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L388-L410)
- [retryUtils.ts](file://src/utils/retryUtils.ts#L33-L70)

## Performance Considerations

The AIService incorporates multiple performance optimization strategies:

### Optimization Strategies

| Strategy | Implementation | Benefits |
|----------|----------------|----------|
| **Caching** | Model service caching, diff content caching | Reduced API calls, faster responses |
| **Compression** | Large file compression, content summarization | Lower bandwidth, reduced costs |
| **Batch Processing** | Multi-file analysis batching | Improved throughput |
| **Streaming** | Real-time response streaming | Better UX, resource efficiency |
| **Lazy Loading** | On-demand service initialization | Faster startup, lower memory usage |

### Performance Metrics

```mermaid
graph LR
subgraph "Response Times"
RT1[Initial Request: 2-5s]
RT2[Streaming: 1-3s]
RT3[Batch: 5-15s]
end
subgraph "Throughput"
TP1[Single Files: 10/min]
TP2[Batch Files: 50+/min]
TP3[Large Files: 2/min]
end
subgraph "Resource Usage"
RU1[Memory: 50-200MB]
RU2[CPU: 10-30%]
RU3[Network: 1-10MB]
end
```

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L270-L275)
- [aiService.ts](file://src/services/ai/aiService.ts#L444-L446)

## Security Aspects

The AIService implements multiple security measures:

### API Key Security

- **Encrypted Storage**: Secure storage in VS Code settings
- **Memory Protection**: Minimal exposure in memory
- **Access Control**: Restricted access to sensitive operations
- **Audit Logging**: Comprehensive logging of API interactions

### Data Protection

- **Content Sanitization**: Removal of sensitive information
- **Transmission Security**: HTTPS encryption
- **Local Caching**: Secure local storage of processed data
- **Privacy Compliance**: GDPR and data protection standards

**Section sources**
- [appConfig.ts](file://src/config/appConfig.ts#L145-L156)
- [aiService.ts](file://src/services/ai/aiService.ts#L726-L732)

## Configuration Management

The AIService integrates with AppConfig for centralized configuration:

### Configuration Options

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `apiKey` | string | "" | AI provider API key |
| `modelType` | string | "deepseek-reasoner" | Selected AI model |
| `baseUrl` | string | "https://api.deepseek.com/v1" | API endpoint URL |
| `language` | string | "ENGLISH" | UI language preference |

### Dynamic Configuration

The system supports runtime configuration changes:

- **Hot Reloading**: Immediate effect of configuration updates
- **Validation**: Real-time validation of configuration changes
- **Rollback**: Automatic rollback on invalid configurations
- **Persistence**: Automatic saving of configuration changes

**Section sources**
- [appConfig.ts](file://src/config/appConfig.ts#L37-L42)
- [appConfig.ts](file://src/config/appConfig.ts#L95-L110)

## Troubleshooting Guide

Common issues and their solutions:

### API Key Issues

**Problem**: API key validation fails
**Solution**: 
1. Verify API key format and validity
2. Check network connectivity
3. Review rate limiting policies
4. Test with alternative API keys

### Performance Issues

**Problem**: Slow response times
**Solution**:
1. Enable streaming for real-time updates
2. Reduce file sizes through compression
3. Optimize prompt complexity
4. Check network latency

### Error Handling

**Problem**: Unexpected errors during analysis
**Solution**:
1. Check error logs in output panel
2. Verify configuration settings
3. Test with smaller files
4. Contact support with error details

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L691-L710)
- [notificationManager.ts](file://src/services/notification/notificationManager.ts#L79-L121)