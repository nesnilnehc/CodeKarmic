# Exclusion Patterns

<cite>
**Referenced Files in This Document**   
- [package.json](file://package.json#L148-L205)
- [src/utils/fileUtils.ts](file://src/utils/fileUtils.ts#L6-L19)
- [src/services/git/gitService.ts](file://src/services/git/gitService.ts#L117)
- [src/services/review/reviewManager.ts](file://src/services/review/reviewManager.ts#L236)
- [.vscodeignore](file://.vscodeignore)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Configuration Overview](#configuration-overview)
3. [Default Exclusion Patterns](#default-exclusion-patterns)
4. [Pattern Syntax and Glob Matching](#pattern-syntax-and-glob-matching)
5. [Integration with GitService and ReviewManager](#integration-with-gitservice-and-reviewmanager)
6. [Custom Configuration](#custom-configuration)
7. [Troubleshooting](#troubleshooting)
8. [Performance Benefits](#performance-benefits)

## Introduction
CodeKarmic implements a comprehensive exclusion system to prevent unnecessary AI processing of non-code files. The `codekarmic.excludeFileTypes` configuration array allows users to specify file types that should be excluded from code review analysis. This system works in conjunction with the `isReviewableFile` function to determine which files require AI processing. The exclusion patterns are primarily defined in the package.json file as default values, but can be customized by users in their settings.json file. This documentation explains the complete exclusion pattern system, including default patterns, syntax rules, integration points, and configuration options.

## Configuration Overview
The `codekarmic.excludeFileTypes` configuration is defined in the package.json file as an array of glob patterns that specify which file types should be excluded from AI code review processing. This configuration serves as a critical performance optimization by preventing the AI system from processing binary files, images, archives, and build artifacts that cannot benefit from code analysis. The configuration is consumed by various components in the CodeKarmic system, particularly the GitService and ReviewManager, when scanning commits and files for review. While the default patterns cover most common non-code file types, users can customize this array in their settings.json file to accommodate project-specific needs.

**Section sources**
- [package.json](file://package.json#L148-L205)

## Default Exclusion Patterns
The default exclusion patterns in CodeKarmic are extensive and cover a wide range of non-code file types across several categories:

**Binaries and Executables**: `*.exe`, `*.dll`, `*.obj`, `*.o`, `*.a`, `*.lib`, `*.so`, `*.dylib`, `*.bin`, `*.class`

**Images**: `*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.bmp`, `*.ico`, `*.svg`, `*.psd`, `*.tga`

**Archives and Compressed Files**: `*.zip`, `*.tar`, `*.gz`, `*.rar`, `*.7z`

**Media Files**: `*.mp3`, `*.mp4`, `*.ogg`, `*.wav`, `*.avi`, `*.mov`, `*.mpg`, `*.mpeg`, `*.mkv`, `*.webm`

**Documents**: `*.pdf`, `*.ttf`, `*.fnt`

**Build and Dependency Directories**: `node_modules/**`, `.git/**`, `dist/**`, `build/**`, `out/**`, `target/**`, `vendor/**`, `.vscode/**`

**Temporary and Lock Files**: `*.ncb`, `*.sdf`, `*.suo`, `*.pdb`, `*.idb`, `*.lock`

These default patterns are designed to cover the vast majority of non-code files encountered in software development projects, preventing them from being processed by the AI system unnecessarily.

**Section sources**
- [package.json](file://package.json#L154-L204)

## Pattern Syntax and Glob Matching
CodeKarmic uses standard glob pattern syntax for file exclusion, which is a simplified form of regular expressions commonly used in file matching. The key syntax elements include:

- `*` (asterisk): Matches any number of characters, but not path separators
- `**` (double asterisk): Matches any number of characters including path separators (recursive)
- `?` (question mark): Matches any single character
- `[]` (character class): Matches any one of the enclosed characters

For example:
- `*.png` matches all PNG files in any directory
- `node_modules/**` matches all files and directories recursively within node_modules
- `*.log` matches all files with a .log extension
- `temp?.txt` matches files like temp1.txt, tempA.txt, but not temp12.txt

The glob patterns are case-insensitive in CodeKarmic, ensuring that files are excluded regardless of their case. When a file path matches any pattern in the `codekarmic.excludeFileTypes` array, it is excluded from AI processing. The system evaluates these patterns against the relative file path within the repository, not just the filename.

**Section sources**
- [package.json](file://package.json#L154-L204)

## Integration with GitService and ReviewManager
The exclusion patterns are integrated into CodeKarmic's core services through a multi-layered approach. The GitService component uses the exclusion patterns when retrieving commit files via the `getCommitFiles` method. Before processing each file in a commit, the system checks whether the file should be excluded based on the configured patterns.

The ReviewManager component plays a crucial role in enforcing exclusions by using the `isReviewableFile` function from the fileUtils module. This function determines whether a file should be eligible for code review based on its extension and type. When the `reviewFile` method is called, it first checks if the file is reviewable:

```typescript
if (!isReviewableFile(filePath)) {
    throw new Error(OUTPUT.REVIEW.FILE_TYPE_NOT_SUPPORTED(filePath));
}
```

The `isReviewableFile` function maintains a whitelist of file extensions that can be code reviewed, which complements the blacklist approach of the exclusion patterns. This dual approach ensures that only appropriate files are processed by the AI system. The GitService also marks binary files with a status of 'binary' in the CommitFile interface, which helps the ReviewManager identify and handle these files appropriately.

**Section sources**
- [src/services/git/gitService.ts](file://src/services/git/gitService.ts#L117)
- [src/services/review/reviewManager.ts](file://src/services/review/reviewManager.ts#L236)
- [src/utils/fileUtils.ts](file://src/utils/fileUtils.ts#L6-L19)

## Custom Configuration
Users can customize the exclusion patterns by modifying the `codekarmic.excludeFileTypes` array in their settings.json file. This allows for project-specific customization based on the types of files present in a particular repository.

To customize the exclusion patterns, add the configuration to your settings.json:

```json
{
    "codekarmic.excludeFileTypes": [
        "*.png",
        "*.jpg",
        "*.jpeg",
        "*.gif",
        "*.zip",
        "node_modules/**",
        ".git/**",
        "dist/**",
        "build/**",
        "*.log",
        "*.tmp",
        "coverage/**"
    ]
}
```

When customizing the exclusion patterns, consider the following best practices:
- Start with the default patterns and add additional ones as needed
- Use specific patterns for project-specific file types
- Include patterns for any build artifacts or temporary files generated by your tools
- Consider adding patterns for large data files that don't require code review
- Test your configuration to ensure it's working as expected

The custom configuration overrides the default values, so be sure to include all desired patterns, not just the additional ones you want to add.

**Section sources**
- [package.json](file://package.json#L148-L205)

## Troubleshooting
When exclusion patterns don't match as expected, consider the following troubleshooting steps:

1. **Verify Pattern Syntax**: Ensure your glob patterns use the correct syntax. Common mistakes include using regex patterns instead of glob patterns, or incorrect use of wildcards.

2. **Check Case Sensitivity**: Although CodeKarmic's pattern matching is case-insensitive, verify that your patterns match the actual file extensions in your repository.

3. **Test Individual Patterns**: If multiple patterns are not working, test them individually to identify which ones are problematic.

4. **Verify File Paths**: Remember that patterns are matched against the relative file path within the repository, not just the filename. Use `**` for recursive matching when needed.

5. **Check for Conflicting Configurations**: Ensure there are no conflicting configurations in different settings files (user, workspace, folder).

6. **Validate with Simple Patterns**: Start with simple patterns like `*.log` before moving to more complex ones.

7. **Restart the Extension**: After changing the configuration, restart VS Code or reload the window to ensure the new settings are loaded.

8. **Check the Output Panel**: Look for any error messages or warnings in the CodeKarmic output panel that might indicate configuration issues.

If patterns still don't work as expected, temporarily simplify your configuration to the bare minimum and gradually add patterns back while testing each change.

**Section sources**
- [package.json](file://package.json#L148-L205)

## Performance Benefits
Proper configuration of exclusion patterns provides significant performance benefits for CodeKarmic:

1. **Reduced AI Processing Costs**: By excluding non-code files, the system avoids sending unnecessary content to the AI service, reducing API calls and associated costs.

2. **Faster Review Processing**: With fewer files to process, the overall code review generation is significantly faster, especially for commits with many changed files.

3. **Lower Memory Usage**: Excluding large binary files and archives reduces memory consumption during the review process.

4. **Improved User Experience**: Faster processing times lead to a more responsive extension and quicker feedback for developers.

5. **Optimized Resource Utilization**: The system can focus its computational resources on files that actually benefit from AI analysis.

6. **Reduced Network Traffic**: Less data is transferred between the extension and AI services, improving performance especially on slower connections.

The default exclusion patterns are carefully selected to maximize these performance benefits while ensuring that all relevant code files are still processed. Customizing the patterns to match your specific project needs can further optimize performance by excluding project-specific file types that don't require code review.

**Section sources**
- [package.json](file://package.json#L148-L205)
- [src/utils/fileUtils.ts](file://src/utils/fileUtils.ts#L6-L19)