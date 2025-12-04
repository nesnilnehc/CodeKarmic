# System Relationships

<cite>
**Referenced Files in This Document**
- [extension.ts](file://src/extension.ts)
- [reviewManager.ts](file://src/services/review/reviewManager.ts)
- [aiService.ts](file://src/services/ai/aiService.ts)
- [gitService.ts](file://src/services/git/gitService.ts)
- [modelFactory.ts](file://src/models/modelFactory.ts)
- [commitExplorer.ts](file://src/ui/components/commitExplorer.ts)
- [fileExplorer.ts](file://src/ui/components/fileExplorer.ts)
- [reviewTypes.ts](file://src/core/review/reviewTypes.ts)
- [versionControlTypes.ts](file://src/services/git/versionControlTypes.ts)
- [types.ts](file://src/models/types.ts)
- [modelInterface.ts](file://src/models/modelInterface.ts)
- [baseModel.ts](file://src/models/baseModel.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Extension Entry Point](#extension-entry-point)
4. [Core Component Relationships](#core-component-relationships)
5. [Domain Models](#domain-models)
6. [Invocation Workflows](#invocation-workflows)
7. [Integration Patterns](#integration-patterns)
8. [Common Issues and Solutions](#common-issues-and-solutions)
9. [Performance Considerations](#performance-considerations)
10. [Conclusion](#conclusion)

## Introduction

The CodeKarmic architecture demonstrates a sophisticated integration of multiple specialized services to provide comprehensive code review capabilities within Visual Studio Code. The system follows a modular design pattern where the extension entry point serves as the orchestrator, coordinating between the ReviewManager, AIService, GitService, and UI components through well-defined interfaces and domain models.

This architecture emphasizes loose coupling between components while maintaining strong cohesion within each module. The system supports multiple review modes including Git commit-based reviews, file/folder exploration, real-time editing, and domain-specific analysis, all unified under a common framework.

## Architecture Overview

The CodeKarmic system follows a layered architecture with clear separation of concerns:

```mermaid
graph TB
subgraph "Extension Layer"
EXT[extension.ts<br/>Entry Point]
end
subgraph "Service Layer"
RM[ReviewManager<br/>Orchestrator]
AIS[AIService<br/>AI Coordinator]
GS[GitService<br/>Version Control]
MF[ModelFactory<br/>Model Provider]
end
subgraph "UI Layer"
CE[CommitExplorer<br/>Commit Browser]
FE[FileExplorer<br/>File Browser]
RP[ReviewPanel<br/>Review Display]
end
subgraph "Model Layer"
CI[CommitInfo<br/>Commit Metadata]
CF[CommitFile<br/>File Changes]
CR[CodeReviewResult<br/>Analysis Results]
MT[ModelType<br/>AI Models]
end
subgraph "External Services"
AI[AI Models<br/>DeepSeek/OpenAI]
GC[Git Repository<br/>Local/Remote]
end
EXT --> RM
EXT --> GS
EXT --> CE
EXT --> FE
RM --> GS
RM --> AIS
AIS --> MF
MF --> AI
CE --> GS
FE --> GS
FE --> RM
RM --> CI
RM --> CF
AIS --> CR
MF --> MT
GS --> GC
AIS --> AI
```

**Diagram sources**
- [extension.ts](file://src/extension.ts#L1-L50)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L80-L120)
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L80)
- [gitService.ts](file://src/services/git/gitService.ts#L45-L85)

## Extension Entry Point

The extension entry point serves as the central coordination hub, initializing and wiring up all core components during activation. The initialization process follows a specific sequence to ensure proper dependency resolution and service availability.

```mermaid
sequenceDiagram
participant Ext as Extension.activate()
participant NM as NotificationManager
participant Config as AppConfig
participant ModelV as ModelValidator
participant Git as GitService
participant RM as ReviewManager
participant UI as UI Components
Ext->>NM : startSession(true)
Ext->>Config : getInstance()
Ext->>ModelV : validateModel(modelType)
ModelV-->>Ext : validation result
Ext->>Git : new GitService()
Ext->>RM : new ReviewManager(gitService)
Ext->>UI : register tree data providers
Note over Ext,UI : Commands registration
Ext->>Ext : registerCommand('codekarmic.startReview')
Ext->>Ext : registerCommand('codekarmic.reviewCode')
Ext->>Ext : registerCommand('codekarmic.generateReport')
```

**Diagram sources**
- [extension.ts](file://src/extension.ts#L20-L70)

The extension initialization ensures that:
- Notification management is established for user feedback
- Model configuration is validated before proceeding
- Shared Git service instances are created for consistency
- Review manager coordinates between Git and AI services
- UI components are registered for user interaction

**Section sources**
- [extension.ts](file://src/extension.ts#L20-L70)

## Core Component Relationships

### ReviewManager Orchestration

The ReviewManager acts as the primary orchestrator, managing the review workflow and coordinating between GitService and AIService. It maintains state for selected commits, review data, and manages the overall review lifecycle.

```mermaid
classDiagram
class ReviewManager {
-gitService : GitService
-selectedCommit : CommitInfo
-reviews : Map~string, ReviewData~
-notificationManager : NotificationManager
+initialize(repoPath : string) : Promise~void~
+selectCommit(commitId : string) : Promise~void~
+reviewFile(filePath : string) : Promise~ReviewData~
+generateReport() : Promise~string~
+addComment(filePath : string, lineNumber : number, content : string) : Promise~void~
+addAISuggestion(filePath : string, suggestion : string) : Promise~void~
+setCodeQualityScore(filePath : string, score : number) : Promise~void~
}
class GitService {
-git : SimpleGit
-repoPath : string
-commits : CommitInfo[]
+setRepository(repoPath : string) : Promise~void~
+getCommits(filter? : CommitFilter) : Promise~CommitInfo[]~
+getCommitFiles(commitId : string) : Promise~CommitFile[]~
+getFileDiff(commitHash : string, filePath : string) : Promise~string~
}
class AIService {
-modelService : AIModelService
-apiKey : string
+reviewCode(params : CodeReviewRequest) : Promise~CodeReviewResult~
+batchReviewCode(requests : CodeReviewRequest[]) : Promise~Map~string, CodeReviewResult~~
+validateApiKey(apiKey : string) : Promise~boolean~
}
class ModelFactory {
-modelServices : Map~string, AIModelService~
+createModelService() : AIModelService
+getSupportedModelTypes() : string[]
}
ReviewManager --> GitService : uses
ReviewManager --> AIService : coordinates
AIService --> ModelFactory : creates models
ReviewManager --> ReviewData : manages
GitService --> CommitInfo : provides
GitService --> CommitFile : provides
```

**Diagram sources**
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L79-L120)
- [gitService.ts](file://src/services/git/gitService.ts#L45-L85)
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L80)
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L50)

### AIService Coordination

The AIService serves as a facade for AI model interactions, delegating to dynamically created model instances via the ModelFactory. It handles API key validation, request batching, and response processing.

```mermaid
sequenceDiagram
participant RM as ReviewManager
participant AIS as AIService
participant MF as ModelFactory
participant MS as ModelService
participant AI as AI Model
RM->>AIS : reviewCode(request)
AIS->>AIS : validate model service
alt Direct file review
AIS->>MS : createChatCompletion()
else Git commit review
AIS->>AIS : generateDiffContent()
AIS->>MS : createChatCompletion()
end
MS->>AI : API call
AI-->>MS : response
MS-->>AIS : formatted response
AIS-->>RM : CodeReviewResult
```

**Diagram sources**
- [aiService.ts](file://src/services/ai/aiService.ts#L74-L120)
- [modelFactory.ts](file://src/models/modelFactory.ts#L58-L95)

**Section sources**
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L79-L120)
- [aiService.ts](file://src/services/ai/aiService.ts#L40-L80)
- [modelFactory.ts](file://src/models/modelFactory.ts#L19-L50)

## Domain Models

### Core Data Structures

The system relies on several key domain models that flow between components, ensuring type safety and clear data contracts.

| Model | Purpose | Key Properties | Flow Direction |
|-------|---------|----------------|----------------|
| **CommitInfo** | Git commit metadata | hash, date, message, author | GitService → ReviewManager |
| **CommitFile** | File changes in commit | path, content, status, diffs | GitService → ReviewManager |
| **CodeReviewRequest** | AI analysis input | filePath, currentContent, diffContent | ReviewManager → AIService |
| **CodeReviewResult** | AI analysis output | suggestions, score, diffContent | AIService → ReviewManager |
| **ReviewData** | Complete review state | commitId, filePath, comments, suggestions | ReviewManager internal |

### Model Relationships

```mermaid
erDiagram
COMMITINFO {
string hash PK
string date
string message
string author
string authorEmail
array files
}
COMMITFILE {
string path PK
string content
string previousContent
string status
int insertions
int deletions
}
CODEREVIEWREQUEST {
string filePath PK
string currentContent
string previousContent
string diffContent
boolean useCompression
string language
}
CODEREVIEWRESULT {
array suggestions
float score
string diffContent
array diffSuggestions
array fullFileSuggestions
}
REVIEWDATA {
string commitId
string filePath PK
array comments
array aiSuggestions
float codeQualityScore
string reviewId PK
}
COMMITINFO ||--o{ COMMITFILE : contains
COMMITINFO ||--o{ REVIEWDATA : reviews
REVIEWDATA ||--|| CODEREVIEWREQUEST : generates
CODEREVIEWREQUEST ||--|| CODEREVIEWRESULT : produces
```

**Diagram sources**
- [versionControlTypes.ts](file://src/services/git/versionControlTypes.ts#L8-L25)
- [reviewTypes.ts](file://src/core/review/reviewTypes.ts#L24-L75)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L11-L27)

**Section sources**
- [versionControlTypes.ts](file://src/services/git/versionControlTypes.ts#L8-L25)
- [reviewTypes.ts](file://src/core/review/reviewTypes.ts#L24-L75)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L11-L27)

## Invocation Workflows

### Starting a Code Review

The workflow for initiating a code review demonstrates the coordinated interaction between multiple components:

```mermaid
sequenceDiagram
participant User
participant Ext as Extension
participant RM as ReviewManager
participant GS as GitService
participant UI as UI Components
User->>Ext : codekarmic.startReview
Ext->>GS : setRepository(rootPath)
GS-->>Ext : repository ready
Ext->>UI : refresh commit explorer
UI-->>User : display commits
User->>UI : select commit
UI->>Ext : codekarmic.selectCommit(hash)
Ext->>RM : selectCommit(commitHash)
RM->>GS : getCommitFiles(commitHash)
GS-->>RM : commit files
RM-->>UI : refresh file explorer
UI-->>User : display files
```

**Diagram sources**
- [extension.ts](file://src/extension.ts#L102-L139)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L149-L207)
- [commitExplorer.ts](file://src/ui/components/commitExplorer.ts#L36-L115)

### Processing a Commit

When processing a specific commit, the system follows a structured workflow:

```mermaid
flowchart TD
Start([Select Commit]) --> ValidateRepo["Validate Repository"]
ValidateRepo --> GetFiles["Get Commit Files"]
GetFiles --> CreateReviews["Create Review Entries"]
CreateReviews --> ParallelProcess["Parallel File Processing"]
ParallelProcess --> GenerateDiff["Generate Diff Content"]
GenerateDiff --> CallAI["Call AI Service"]
CallAI --> ProcessResult["Process AI Results"]
ProcessResult --> UpdateState["Update Review State"]
UpdateState --> MoreFiles{"More Files?"}
MoreFiles --> |Yes| ParallelProcess
MoreFiles --> |No| Complete([Complete])
```

**Diagram sources**
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L289-L370)
- [aiService.ts](file://src/services/ai/aiService.ts#L74-L120)

### Generating a Report

The report generation workflow showcases batch processing and result aggregation:

```mermaid
sequenceDiagram
participant User
participant RM as ReviewManager
participant GS as GitService
participant AIS as AIService
participant WV as WebView
User->>RM : generateReport()
RM->>GS : getCommitFiles(selectedCommit.hash)
GS-->>RM : commit files
RM->>WV : create report view
RM->>AIS : batchReviewCode(files)
loop For each file batch
AIS->>AIS : process batch
AIS-->>RM : batch results
RM->>WV : update progress
end
RM->>RM : generateMarkdownReport()
RM-->>WV : final report
WV-->>User : display report
```

**Diagram sources**
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L372-L648)
- [aiService.ts](file://src/services/ai/aiService.ts#L431-L552)

**Section sources**
- [extension.ts](file://src/extension.ts#L102-L139)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L289-L370)
- [reviewManager.ts](file://src/services/review/reviewManager.ts#L372-L648)

## Integration Patterns

### Command Pattern Implementation

The extension uses VS Code's command pattern extensively for user interactions:

```mermaid
classDiagram
class CommandHandler {
<<interface>>
+execute(...args) : Promise~any~
}
class StartReviewCommand {
+execute() : Promise~void~
}
class SelectCommitCommand {
+execute(commitHash : string) : Promise~void~
}
class ReviewCodeCommand {
+execute(params : object) : Promise~any~
}
class GenerateReportCommand {
+execute() : Promise~string~
}
CommandHandler <|.. StartReviewCommand
CommandHandler <|.. SelectCommitCommand
CommandHandler <|.. ReviewCodeCommand
CommandHandler <|.. GenerateReportCommand
```

**Diagram sources**
- [extension.ts](file://src/extension.ts#L82-L244)

### Observer Pattern for UI Updates

The UI components utilize the observer pattern for reactive updates:

```mermaid
classDiagram
class TreeDataProvider {
<<interface>>
+onDidChangeTreeData : Event
+getChildren(element?) : Promise~T[]~
+getTreeItem(element : T) : TreeItem
}
class CommitExplorerProvider {
-_onDidChangeTreeData : EventEmitter
+refresh() : void
+setLoading(isLoading : boolean) : void
+setError(message : string) : void
}
class FileExplorerProvider {
-_onDidChangeTreeData : EventEmitter
+refresh() : void
}
TreeDataProvider <|.. CommitExplorerProvider
TreeDataProvider <|.. FileExplorerProvider
```

**Diagram sources**
- [commitExplorer.ts](file://src/ui/components/commitExplorer.ts#L5-L15)
- [fileExplorer.ts](file://src/ui/components/fileExplorer.ts#L6-L15)

**Section sources**
- [extension.ts](file://src/extension.ts#L82-L244)
- [commitExplorer.ts](file://src/ui/components/commitExplorer.ts#L5-L15)
- [fileExplorer.ts](file://src/ui/components/fileExplorer.ts#L6-L15)

## Common Issues and Solutions

### API Key Configuration Issues

**Problem**: AI service fails due to missing or invalid API key.

**Solution**: The system implements a cascading validation approach:

```mermaid
flowchart TD
Start([API Call]) --> CheckKey{"API Key Available?"}
CheckKey --> |No| PromptUser["Prompt User for API Key"]
PromptKey --> Validate["Validate API Key"]
Validate --> Valid{"Valid?"}
Valid --> |Yes| Retry["Retry Original Request"]
Valid --> |No| ShowError["Show Error Message"]
CheckKey --> |Yes| DirectCall["Direct API Call"]
Retry --> Success["Success"]
DirectCall --> Success
```

**Diagram sources**
- [extension.ts](file://src/extension.ts#L141-L184)
- [aiService.ts](file://src/services/ai/aiService.ts#L712-L732)

### Git Repository Detection Issues

**Problem**: Extension fails to detect Git repositories or encounters permission issues.

**Solution**: Multi-layered detection with fallback mechanisms:

| Detection Method | Priority | Fallback |
|------------------|----------|----------|
| VS Code Git Extension API | Highest | Direct Git commands |
| SimpleGit library | Medium | Manual git CLI |
| File system checks | Lowest | Error reporting |

### Memory Management for Large Files

**Problem**: Processing large files causes memory exhaustion.

**Solution**: Intelligent compression and streaming:

```mermaid
flowchart TD
FileInput([Large File]) --> SizeCheck{"Size > Threshold?"}
SizeCheck --> |No| ProcessNormal["Process Normally"]
SizeCheck --> |Yes| Compress["Compress Content"]
Compress --> ExtractCode["Extract Code Blocks"]
ExtractCode --> Stream["Stream to AI"]
ProcessNormal --> Output([Result])
Stream --> Output
```

**Diagram sources**
- [baseModel.ts](file://src/models/baseModel.ts#L20-L60)
- [aiService.ts](file://src/services/ai/aiService.ts#L413-L424)

**Section sources**
- [extension.ts](file://src/extension.ts#L141-L184)
- [gitService.ts](file://src/services/git/gitService.ts#L67-L108)
- [baseModel.ts](file://src/models/baseModel.ts#L20-L60)

## Performance Considerations

### Concurrent Processing Strategy

The system implements several strategies for optimal performance:

1. **Batch Processing**: Files are processed in configurable batches (default: 5 files)
2. **Parallel Execution**: Multiple AI requests can be processed concurrently
3. **Caching**: Git diffs and AI responses are cached to avoid redundant operations
4. **Lazy Loading**: UI components load data on-demand rather than pre-loading

### Resource Management

| Resource Type | Management Strategy | Benefits |
|---------------|-------------------|----------|
| **Memory** | Content compression, streaming | Reduces memory footprint |
| **Network** | Request batching, caching | Minimizes API calls |
| **CPU** | Parallel processing, lazy loading | Optimizes computation |
| **Storage** | Report generation, temporary files | Efficient data persistence |

### Scalability Patterns

The architecture supports horizontal scaling through:
- Stateless service design
- Configurable batch sizes
- Pluggable model providers
- Asynchronous processing patterns

## Conclusion

The CodeKarmic architecture demonstrates a well-designed system that successfully integrates multiple specialized services while maintaining clear separation of concerns. The extension entry point effectively coordinates between ReviewManager, AIService, GitService, and UI components through well-defined interfaces and domain models.

Key architectural strengths include:
- **Modular Design**: Clear separation between UI, business logic, and external services
- **Flexible Workflow**: Support for multiple review modes and integration patterns
- **Robust Error Handling**: Comprehensive error handling and recovery mechanisms
- **Performance Optimization**: Intelligent caching, batching, and resource management
- **Extensibility**: Plugin architecture for AI models and review modes

The system's design enables easy maintenance, testing, and future enhancements while providing a seamless user experience for code review workflows. The use of domain models ensures type safety and clear data contracts between components, while the command and observer patterns provide responsive UI interactions.

This architecture serves as an excellent example of how to build complex integrated systems within the VS Code extension ecosystem, balancing functionality, performance, and maintainability.