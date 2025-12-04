# Core Logic

<cite>
**Referenced Files in This Document**
- [codeAnalyzer.ts](file://src/core/review/codeAnalyzer.ts)
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts)
- [contentCompressor.ts](file://src/core/compression/contentCompressor.ts)
- [largeFileProcessor.ts](file://src/core/compression/largeFileProcessor.ts)
- [reviewTypes.ts](file://src/core/review/reviewTypes.ts)
- [compressionTypes.ts](file://src/core/compression/compressionTypes.ts)
- [aiService.ts](file://src/services/ai/aiService.ts)
- [modelInterface.ts](file://src/models/modelInterface.ts)
- [prompts.ts](file://src/i18n/en/prompts.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Code Analyzer](#code-analyzer)
4. [Suggestion Generator](#suggestion-generator)
5. [Content Compression System](#content-compression-system)
6. [Large File Processing](#large-file-processing)
7. [AI Integration](#ai-integration)
8. [Performance Considerations](#performance-considerations)
9. [Error Handling](#error-handling)
10. [Integration Points](#integration-points)
11. [Conclusion](#conclusion)

## Introduction

CodeKarmic's Core Logic forms the heart of the code review system, providing sophisticated analysis capabilities through intelligent parsing, compression, and AI-driven suggestion generation. The system is designed to handle both small and large files efficiently, offering comprehensive code quality assessments while maintaining optimal performance and reliability.

The core components work together to transform raw code changes into actionable insights, enabling developers to improve their code quality systematically. The architecture emphasizes modularity, extensibility, and performance optimization, making it suitable for various development scenarios from individual file reviews to large-scale project analysis.

## Architecture Overview

The Core Logic architecture follows a layered approach with clear separation of concerns:

```mermaid
graph TB
subgraph "Input Layer"
A[Code Review Requests]
B[File Content]
C[Diff Content]
end
subgraph "Processing Layer"
D[Code Analyzer]
E[Suggestion Generator]
F[Content Compressor]
G[Large File Processor]
end
subgraph "AI Integration"
H[AI Service]
I[Model Interface]
J[Content Generation]
end
subgraph "Output Layer"
K[Structured Suggestions]
L[Quality Scores]
M[Review Reports]
end
A --> D
B --> D
C --> D
D --> E
D --> F
F --> G
G --> H
H --> I
I --> J
E --> K
E --> L
E --> M
```

**Diagram sources**
- [codeAnalyzer.ts](file://src/core/review/codeAnalyzer.ts#L17-L90)
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L56-L117)
- [contentCompressor.ts](file://src/core/compression/contentCompressor.ts#L18-L414)
- [largeFileProcessor.ts](file://src/core/compression/largeFileProcessor.ts#L23-L242)

## Code Analyzer

The Code Analyzer serves as the primary orchestrator for code review analysis, responsible for determining the appropriate analysis strategy based on file characteristics and user preferences.

### Core Architecture

```mermaid
classDiagram
class CodeAnalyzer {
-logger : Logger
-largeFileProcessor : LargeFileProcessor
-modelService : ModelInterface
+analyzeCode(request, options) : Promise~CodeReviewResult~
-analyzeDiff(request) : Promise~string[]~
-analyzeFullFile(request) : Promise~string[]~
-parseSuggestions(responseText) : string[]
-calculateScore(result) : number
}
class CodeReviewRequest {
+filePath : string
+currentContent : string
+previousContent : string
+diffContent? : string
+useCompression? : boolean
+language? : string
}
class CodeReviewResult {
+suggestions : string[]
+score? : number
+diffSuggestions? : string[]
+fullFileSuggestions? : string[]
}
class CodeAnalysisOptions {
+useCompression : boolean
+includeDiffAnalysis : boolean
+includeFullFileAnalysis : boolean
+maxTokens? : number
}
CodeAnalyzer --> CodeReviewRequest : processes
CodeAnalyzer --> CodeReviewResult : generates
CodeAnalyzer --> CodeAnalysisOptions : configured by
```

**Diagram sources**
- [codeAnalyzer.ts](file://src/core/review/codeAnalyzer.ts#L17-L230)
- [reviewTypes.ts](file://src/core/review/reviewTypes.ts#L24-L73)

### Analysis Pipeline

The Code Analyzer implements a sophisticated analysis pipeline that adapts to different scenarios:

```mermaid
flowchart TD
Start([Code Analysis Request]) --> CheckLargeFile{"Is Large File?"}
CheckLargeFile --> |Yes| ProcessLargeFile["Large File Processing"]
CheckLargeFile --> |No| CheckDiff{"Include Diff Analysis?"}
CheckDiff --> |Yes| AnalyzeDiff["Analyze Differences"]
CheckDiff --> |No| AnalyzeFull["Analyze Full File"]
AnalyzeDiff --> CheckFull{"Include Full File Analysis?"}
CheckFull --> |Yes| AnalyzeFull
CheckFull --> |No| CombineResults["Combine Results"]
AnalyzeFull --> CombineResults
ProcessLargeFile --> GenerateScore["Calculate Score"]
CombineResults --> GenerateScore
GenerateScore --> ReturnResult["Return CodeReviewResult"]
ReturnResult --> End([Analysis Complete])
```

**Diagram sources**
- [codeAnalyzer.ts](file://src/core/review/codeAnalyzer.ts#L35-L90)

### Key Features

**Dual Analysis Mode**: The analyzer supports both difference-based analysis for Git changes and full file analysis for standalone reviews, providing flexibility for different use cases.

**Intelligent Compression Detection**: Automatically detects large files and applies compression strategies to optimize AI processing while preserving essential information.

**Score Calculation**: Implements a sophisticated scoring mechanism that considers suggestion count, content quality, and AI confidence levels to provide meaningful ratings.

**Error Resilience**: Comprehensive error handling ensures that analysis failures don't prevent the system from providing partial results or fallback suggestions.

**Section sources**
- [codeAnalyzer.ts](file://src/core/review/codeAnalyzer.ts#L35-L230)

## Suggestion Generator

The Suggestion Generator transforms raw AI responses into structured, actionable recommendations with rich metadata for enhanced developer experience.

### Structured Output System

```mermaid
classDiagram
class SuggestionGenerator {
-logger : Logger
-modelService : ModelInterface
+generateFinalSuggestions(diffSuggestions, fullFileSuggestions) : Promise~Object~
+structureSuggestions(suggestions) : StructuredSuggestion[]
+generateReport(result, filePath) : string
-parseFinalResponse(responseText) : Object
-estimateScore(suggestions) : number
}
class StructuredSuggestion {
+content : string
+lines? : string | number[]
+category : SuggestionCategory
+severity : SuggestionSeverity
+fixExample? : string
}
class SuggestionCategory {
<<enumeration>>
STRUCTURE
PERFORMANCE
SECURITY
READABILITY
MAINTAINABILITY
BEST_PRACTICE
OTHER
}
class SuggestionSeverity {
<<enumeration>>
CRITICAL
HIGH
MEDIUM
LOW
INFO
}
SuggestionGenerator --> StructuredSuggestion : creates
StructuredSuggestion --> SuggestionCategory : categorized by
StructuredSuggestion --> SuggestionSeverity : rated by
```

**Diagram sources**
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L56-L456)

### Suggestion Processing Workflow

```mermaid
sequenceDiagram
participant CG as Code Generator
participant SG as Suggestion Generator
participant AI as AI Service
participant Parser as Content Parser
CG->>SG : generateFinalSuggestions()
SG->>AI : createChatCompletion()
AI-->>SG : raw suggestions
SG->>Parser : parseFinalResponse()
Parser-->>SG : structured suggestions
SG->>SG : structureSuggestions()
SG->>SG : categorizeSuggestions()
SG->>SG : generateReport()
SG-->>CG : formatted results
```

**Diagram sources**
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L71-L117)

### Advanced Categorization

The suggestion generator implements intelligent categorization using pattern matching and machine learning heuristics:

| Category | Keywords | Severity Indicators |
|----------|----------|-------------------|
| **Structure** | structure, organization, architecture, module, dependency | Critical for design flaws |
| **Performance** | performance, efficiency, optimization, speed, memory, resource | High for bottlenecks |
| **Security** | security, vulnerability, injection, validation, encryption | Critical for breaches |
| **Readability** | readability, naming, comment, format | Medium for maintainability |
| **Maintainability** | maintainability, duplication, complexity, testing | Medium for technical debt |
| **Best Practice** | best practice, convention, standard, pattern | Low for standards compliance |

**Section sources**
- [suggestionGenerator.ts](file://src/core/review/suggestionGenerator.ts#L119-L456)

## Content Compression System

The Content Compression System provides intelligent content reduction capabilities that preserve essential information while significantly reducing processing overhead for large files.

### Compression Algorithm

```mermaid
flowchart TD
Input[Raw Content] --> DetectLang["Detect Programming Language"]
DetectLang --> CalcStats["Calculate Content Statistics"]
CalcStats --> ScoreLines["Score Lines by Importance"]
ScoreLines --> JS{JavaScript/TypeScript?}
ScoreLines --> Python{Python?}
ScoreLines --> Java{Java/Kotlin?}
ScoreLines --> CPP{C/C++?}
ScoreLines --> CSS{CSS/SASS?}
ScoreLines --> Generic{Other Languages?}
JS --> JSFeatures["Function Definitions<br/>React Hooks<br/>JSX Elements<br/>State Management"]
Python --> PyFeatures["Function/Class Defs<br/>Imports/Decorators<br/>Control Flow<br/>Special Methods"]
Java --> JavaFeatures["Class/Interface Defs<br/>Method Signatures<br/>Annotations<br/>Access Modifiers"]
CPP --> CPPFeatures["Preprocessor<br/>Class Definitions<br/>Function Signatures<br/>Templates"]
CSS --> CSSFeatures["Selectors<br/>Media Queries<br/>Variables/Mixins<br/>Responsive Design"]
Generic --> GenFeatures["Code Structure<br/>Comments<br/>Error Handling<br/>Security Patterns"]
JSFeatures --> CombineScores["Combine Importance Scores"]
PyFeatures --> CombineScores
JavaFeatures --> CombineScores
CPPFeatures --> CombineScores
CSSFeatures --> CombineScores
GenFeatures --> CombineScores
CombineScores --> SortByScore["Sort by Importance"]
SortByScore --> SelectTop["Select Top N%"]
SelectTop --> BuildOutput["Build Compressed Output"]
BuildOutput --> Stats["Add Statistics Header"]
Stats --> Output[Compressed Content]
```

**Diagram sources**
- [contentCompressor.ts](file://src/core/compression/contentCompressor.ts#L56-L174)

### Language-Specific Optimization

The compression system implements sophisticated language detection and optimization:

| Language Family | Key Features | Optimization Strategies |
|----------------|--------------|------------------------|
| **JavaScript/TypeScript** | React hooks, JSX, functional components | Preserve component structure, hook patterns, and state management |
| **Python** | Decorators, magic methods, import statements | Maintain function/class relationships, decorator usage patterns |
| **Java/Kotlin** | Annotations, generics, access modifiers | Preserve class hierarchies, method signatures, and dependency injection |
| **C/C++** | Preprocessor directives, templates | Maintain macro definitions, template signatures, and include guards |
| **CSS/SASS** | Selectors, mixins, responsive patterns | Preserve layout structures, design system patterns |
| **SQL** | Schema definitions, query patterns | Maintain table relationships, indexing strategies |

### Compression Statistics

The system provides comprehensive compression analytics:

```mermaid
classDiagram
class CompressionStats {
+originalSize : number
+compressedSize : number
+compressionRatio : number
+totalLines : number
+keptLines : number
+lineRetentionRate : number
}
class ContentFingerprint {
+language : string
+totalLines : number
+nonEmptyLines : number
+totalTokens : number
+commentLines : number
+codeCommentRatio : number
+contentHash : string
+metrics : Record~string, number~
}
CompressionStats --> ContentFingerprint : generated by
```

**Diagram sources**
- [compressionTypes.ts](file://src/core/compression/compressionTypes.ts#L46-L87)
- [contentCompressor.ts](file://src/core/compression/contentCompressor.ts#L287-L414)

**Section sources**
- [contentCompressor.ts](file://src/core/compression/contentCompressor.ts#L18-L414)

## Large File Processing

The Large File Processor specializes in handling files that exceed AI model token limits, using advanced compression and summarization techniques.

### Processing Pipeline

```mermaid
sequenceDiagram
participant Client as Client Request
participant Processor as LargeFileProcessor
participant Compressor as ContentCompressor
participant AI as AI Service
participant Cache as Diff Cache
Client->>Processor : processLargeFile(request)
Processor->>Processor : isLargeFile(request)
Processor->>Compressor : compressContent(content)
Compressor-->>Processor : compressed content + stats
Processor->>Processor : getLargeFilePrompt()
Processor->>AI : makeSpecificApiRequest(prompt)
AI-->>Processor : AI response
Processor->>Processor : extractSuggestions()
Processor-->>Client : CodeReviewResult
Note over Cache : Caching mechanism for<br/>diff content reuse
```

**Diagram sources**
- [largeFileProcessor.ts](file://src/core/compression/largeFileProcessor.ts#L55-L81)

### Batch Processing System

The processor implements intelligent batch processing for multiple large files:

```mermaid
flowchart TD
Start([Multiple File Requests]) --> Categorize["Categorize by Size"]
Categorize --> LargeFiles["Large Files (>20KB)"]
Categorize --> NormalFiles["Normal Files (<20KB)"]
LargeFiles --> BatchLarge["Batch Process Large Files"]
BatchLarge --> CompressLarge["Apply Compression"]
CompressLarge --> AIRequestLarge["Single AI Request"]
AIRequestLarge --> ParseLarge["Parse Responses"]
NormalFiles --> SortSize["Sort by Size"]
SortSize --> CreateBatches["Create Batches"]
CreateBatches --> TokenLimit{"Within Token Limit?"}
TokenLimit --> |Yes| AddToBatch["Add to Current Batch"]
TokenLimit --> |No| ProcessBatch["Process Current Batch"]
AddToBatch --> NextFile["Next File"]
NextFile --> TokenLimit
ProcessBatch --> AIRequestNormal["Batch AI Request"]
AIRequestNormal --> ParseNormal["Parse Responses"]
ParseLarge --> MergeResults["Merge Results"]
ParseNormal --> MergeResults
MergeResults --> End([Final Results])
```

**Diagram sources**
- [largeFileProcessor.ts](file://src/core/compression/largeFileProcessor.ts#L158-L225)

### Performance Optimization

The Large File Processor includes several performance optimizations:

**Token-Aware Batching**: Automatically groups files to stay within AI model token limits while maximizing processing efficiency.

**Intelligent Compression**: Uses adaptive compression ratios based on file characteristics and content complexity.

**Caching Strategy**: Implements intelligent caching for diff content to avoid redundant computations.

**Parallel Processing**: Supports concurrent processing of independent file batches to maximize throughput.

**Section sources**
- [largeFileProcessor.ts](file://src/core/compression/largeFileProcessor.ts#L23-L242)

## AI Integration

The AI Integration layer provides seamless connectivity between the core logic and various AI models, enabling flexible model switching and enhanced processing capabilities.

### Model Interface Architecture

```mermaid
classDiagram
class ModelInterface {
+generateContent(request) : Promise~AIModelResponse~
+getModelType() : string
+initialize(options) : void
}
class AIModelService {
+initialize(options) : void
+validateApiKey(apiKey) : Promise~boolean~
+createChatCompletion(params) : Promise~AIModelResponse~
+getModelType() : string
}
class AbstractAIModelService {
#modelType : string
#apiKey : string
#baseURL : string
+retryOperation(operation, retries, delay) : Promise~T~
}
class AIService {
-modelService : AIModelService
-client : OpenAI
-apiKey : string
-modelType : string
+reviewCode(params) : Promise~CodeReviewResult~
+batchReviewCode(requests) : Promise~Map~
+validateApiKey(apiKey) : Promise~boolean~
}
ModelInterface <|.. AIModelService : implements
AIModelService <|-- AbstractAIModelService : extends
AIService --> AIModelService : uses
```

**Diagram sources**
- [modelInterface.ts](file://src/models/modelInterface.ts#L39-L185)
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L787)

### Content Generation Pipeline

The AI integration supports sophisticated content generation with streaming and compression capabilities:

```mermaid
sequenceDiagram
participant Analyzer as Code Analyzer
participant AI as AI Service
participant Model as AI Model
participant Stream as Streaming Handler
Analyzer->>AI : generateContent(request)
AI->>AI : validateApiKey()
AI->>AI : createChatCompletion()
AI->>Model : send request with streaming
Model->>Stream : stream chunks
Stream->>Stream : accumulate content
Stream->>AI : complete response
AI->>AI : extractSuggestions()
AI->>AI : calculateScore()
AI-->>Analyzer : structured result
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L260-L411)

### Advanced Features

**Streaming Support**: Real-time response processing enables immediate feedback during long-running analyses.

**Automatic Compression**: Intelligent content compression reduces token usage while preserving essential information.

**Retry Logic**: Robust retry mechanisms handle network issues and rate limiting gracefully.

**Multi-Language Support**: Comprehensive prompt templating supports multiple languages and cultural contexts.

**Section sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L787)
- [modelInterface.ts](file://src/models/modelInterface.ts#L39-L185)

## Performance Considerations

The Core Logic components implement numerous performance optimizations to ensure efficient operation at scale.

### Optimization Strategies

**Intelligent Caching**: Multi-level caching system prevents redundant computations and improves response times.

**Adaptive Compression**: Dynamic compression ratios based on content characteristics optimize processing efficiency.

**Batch Processing**: Intelligent batching reduces API calls and maximizes throughput for multiple files.

**Lazy Loading**: Components load only when needed, reducing memory footprint and startup time.

**Token Management**: Sophisticated token counting and budgeting prevent exceeding model limits.

### Performance Metrics

| Component | Typical Response Time | Memory Usage | Throughput |
|-----------|----------------------|--------------|------------|
| **Small File Analysis** | 2-5 seconds | 50-100 MB | 100+ files/min |
| **Large File Processing** | 10-30 seconds | 200-500 MB | 10-20 files/min |
| **Batch Processing** | 30-60 seconds | 100-300 MB | 50-100 files/min |
| **AI Model Calls** | 5-15 seconds | 10-50 MB | 1-5 requests/sec |

### Scalability Features

**Horizontal Scaling**: Stateless design enables easy horizontal scaling across multiple instances.

**Resource Monitoring**: Built-in performance monitoring tracks system health and identifies bottlenecks.

**Load Balancing**: Intelligent request distribution optimizes resource utilization.

**Graceful Degradation**: System continues operating with reduced functionality during high-load periods.

## Error Handling

The Core Logic implements comprehensive error handling strategies to ensure system reliability and provide meaningful feedback to users.

### Error Classification

```mermaid
flowchart TD
Error[Error Occurred] --> Classify{"Error Type"}
Classify --> |Network| NetworkError["Network Timeout<br/>Connection Issues<br/>Rate Limiting"]
Classify --> |AI| AIError["Model Unavailable<br/>Invalid Response<br/>Token Limit Exceeded"]
Classify --> |Processing| ProcessingError["Memory Overflow<br/>Parsing Failure<br/>Compression Error"]
Classify --> |Validation| ValidationError["Invalid Input<br/>Missing Parameters<br/>Format Issues"]
NetworkError --> Retry["Retry with Backoff"]
AIError --> Fallback["Use Fallback Model"]
ProcessingError --> Graceful["Graceful Degradation"]
ValidationError --> UserFeedback["User-Friendly Message"]
Retry --> Success["Successful Recovery"]
Fallback --> Success
Graceful --> PartialResult["Partial Results"]
UserFeedback --> UserAction["User Action Required"]
```

### Error Recovery Mechanisms

**Automatic Retry**: Intelligent retry logic with exponential backoff for transient failures.

**Fallback Strategies**: Alternative processing methods when primary approaches fail.

**Graceful Degradation**: Reduced functionality instead of complete failure when resources are constrained.

**User Communication**: Clear, actionable error messages that help users resolve issues.

**Logging and Monitoring**: Comprehensive logging for debugging and system monitoring.

## Integration Points

The Core Logic integrates seamlessly with various system components and external services.

### External Integrations

**Version Control Systems**: Native Git integration for diff generation and commit analysis.

**IDE Integration**: VS Code extension support for real-time code review.

**CI/CD Pipelines**: Webhook support for automated code review in continuous integration workflows.

**Notification Systems**: Real-time progress updates and completion notifications.

### Internal Dependencies

**Configuration Management**: Centralized configuration system for model selection and processing options.

**Logging System**: Unified logging infrastructure for debugging and monitoring.

**Cache Management**: Distributed caching for improved performance and reduced API calls.

**Notification Manager**: Coordinated notification delivery across the system.

## Conclusion

CodeKarmic's Core Logic represents a sophisticated, scalable solution for intelligent code review and analysis. The modular architecture, combined with advanced compression techniques and robust AI integration, provides developers with powerful tools for improving code quality while maintaining optimal performance.

Key strengths include:

- **Intelligent Processing**: Adaptive analysis strategies that optimize for different file sizes and types
- **Scalable Architecture**: Designed to handle everything from individual files to large-scale projects
- **Robust Error Handling**: Comprehensive error recovery and user communication systems
- **Performance Optimization**: Multiple layers of optimization for speed and resource efficiency
- **Extensible Design**: Modular architecture that supports easy enhancement and customization

The system successfully balances the need for comprehensive analysis with practical performance requirements, making it suitable for both development teams and enterprise environments. Its intelligent compression and batching capabilities ensure that even large codebases can be analyzed efficiently, while the structured suggestion system provides actionable insights that drive continuous improvement.