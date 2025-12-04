# Translation Process

<cite>
**Referenced Files in This Document**   
- [index.ts](file://src/i18n/index.ts)
- [types.ts](file://src/i18n/types.ts)
- [ui.ts](file://src/i18n/en/ui.ts)
- [prompts.ts](file://src/i18n/en/prompts.ts)
- [output.ts](file://src/i18n/en/output.ts)
- [ui.ts](file://src/i18n/zh/ui.ts)
- [prompts.ts](file://src/i18n/zh/prompts.ts)
- [output.ts](file://src/i18n/zh/output.ts)
- [notificationManager.ts](file://src/services/notification/notificationManager.ts)
</cite>

## Table of Contents
1. [Translation File Structure](#translation-file-structure)
2. [Language Configuration and Module Import](#language-configuration-and-module-import)
3. [Message Resolution Flow](#message-resolution-flow)
4. [Type Safety and Interface Consistency](#type-safety-and-interface-consistency)
5. [AI Prompt Localization](#ai-prompt-localization)
6. [Testing Considerations](#testing-considerations)

## Translation File Structure

The CodeKarmic translation system organizes translation files by both category and language, creating a structured and maintainable internationalization framework. The i18n directory contains separate subdirectories for each supported language (en/ for English and zh/ for Chinese), with each language directory containing three distinct translation modules: ui.ts, output.ts, and prompts.ts.

The ui.ts files contain user interface text such as button labels, tab names, placeholders, and messages displayed directly to users. These translations are organized into logical categories like BUTTONS, TABS, PLACEHOLDERS, and MESSAGES, making it easy to locate and modify specific UI elements. For example, the UI.BUTTONS.REVIEW constant provides the text for the "Review Code" button in English, while its Chinese counterpart provides "代码审查".

The output.ts files contain system output messages used in logs, notifications, and status indicators. These are organized by functional areas such as REVIEW, REPOSITORY, GIT, and REPORT, allowing for contextual grouping of related messages. This structure ensures that all output messages follow a consistent format and can be easily maintained.

The prompts.ts files contain AI prompt templates used when generating model inputs for code review. These include comprehensive templates for different types of analysis, such as full file reviews, diff analysis, and final scoring. The prompts are structured to maintain consistency in the AI's response format across languages.

All translation modules export typed constants that maintain a parallel structure between languages, ensuring that each language implementation contains the same set of keys and nested objects. This structural consistency is critical for the type safety and reliability of the internationalization system.

**Section sources**
- [ui.ts](file://src/i18n/en/ui.ts#L1-L70)
- [output.ts](file://src/i18n/en/output.ts#L1-L201)
- [prompts.ts](file://src/i18n/en/prompts.ts#L1-L108)
- [ui.ts](file://src/i18n/zh/ui.ts#L1-L70)
- [output.ts](file://src/i18n/zh/output.ts#L1-L201)
- [prompts.ts](file://src/i18n/zh/prompts.ts#L1-L108)

## Language Configuration and Module Import

The translation system in CodeKarmic is centered around the LANGUAGE_CONFIG object defined in index.ts, which serves as the central registry for all translation resources. This configuration object maps each translation category (UI, OUTPUT, PROMPTS) to its corresponding language implementations using the Language enum as keys.

The import process begins with importing all translation modules from both language directories, using aliasing to create distinct references for each language version. For example, the English UI translations are imported as EN_UI, while the Chinese UI translations are imported as ZH_UI. This approach prevents naming conflicts and makes the language mapping explicit.

The LANGUAGE_CONFIG object then organizes these imported modules into a structured hierarchy where each category contains a mapping of Language enum values to their corresponding translation objects. This centralized configuration enables the I18nManager to efficiently access and manage translations based on the current language setting.

The system supports only two languages (English and Chinese) as defined in the Language enum in types.ts, with ENGLISH mapped to 'en' and CHINESE mapped to 'zh'. This enum provides type safety throughout the application, ensuring that language values are consistent and validated at compile time.

**Section sources**
- [index.ts](file://src/i18n/index.ts#L5-L38)
- [types.ts](file://src/i18n/types.ts#L4-L7)

## Message Resolution Flow

The message resolution flow in CodeKarmic follows a well-defined process when a component accesses translation constants such as i18n.UI.buttonLabel or i18n.PROMPTS.reviewPrompt. This process begins with the singleton I18nManager, which initializes with the current language setting from the application configuration.

When a component accesses a translation property, the resolution flow proceeds as follows: First, the I18nManager checks the current language setting, which is synchronized with the application's configuration. If the current language is English, the system directly returns the English translation object. However, if the current language is Chinese (or any non-English language), the system creates a proxy object that wraps the Chinese translation with fallback capabilities.

The createTranslationProxy function is central to this resolution process, implementing a Proxy pattern that intercepts property access. When a property is accessed on the proxied translation object, the proxy first checks if the property exists in the current language's translation. If the property is not found, the proxy automatically falls back to the English translation, ensuring that all messages are available even if some translations are missing or incomplete.

This proxy mechanism also handles nested objects recursively, creating proxied versions of any sub-objects to maintain the fallback behavior throughout the entire translation hierarchy. For example, when accessing i18n.UI.MESSAGES.NO_COMMENTS, if the specific message is missing in the current language, the system will fall back to the English version while maintaining the same access pattern.

The resolution flow is optimized for performance by initializing the proxied translation objects only when the language changes, rather than on every property access. The I18nManager listens for configuration changes and updates the translation objects accordingly, ensuring that the UI reflects language changes without requiring a restart.

**Section sources**
- [index.ts](file://src/i18n/index.ts#L52-L69)
- [index.ts](file://src/i18n/index.ts#L118-L131)
- [index.ts](file://src/i18n/index.ts#L169-L172)

## Type Safety and Interface Consistency

CodeKarmic maintains type safety and consistency across locales through a combination of TypeScript interfaces, enum-based language definitions, and structural parallelism between translation files. The Language enum in types.ts provides compile-time safety for language values, preventing invalid language specifications throughout the codebase.

The BilingualMessage interface defines a standardized structure for messages that need to be available in both languages simultaneously, with explicit zh and en properties. This interface is used in components like the notification manager to ensure that messages are properly formatted for bilingual display.

The translation system enforces structural consistency by maintaining identical object hierarchies across all language implementations. For example, the UI.BUTTONS object exists in both the English and Chinese ui.ts files with the same set of button constants, ensuring that any code accessing these properties will work regardless of the active language.

The proxy-based fallback mechanism further enhances type safety by ensuring that property access always returns a valid string, even when a specific translation is missing. This prevents runtime errors due to undefined values while maintaining the expected return type.

The system also includes utility functions like getLanguageDisplayName and getLanguageFromDisplayName that provide safe conversion between language enum values and their display names, with appropriate fallback behavior for unrecognized values.

**Section sources**
- [types.ts](file://src/i18n/types.ts#L12-L15)
- [types.ts](file://src/i18n/types.ts#L18-L36)
- [notificationManager.ts](file://src/services/notification/notificationManager.ts#L79-L91)

## AI Prompt Localization

AI prompt localization in CodeKarmic is a critical aspect of the translation system, as it directly impacts the quality and effectiveness of model input generation. The prompts.ts files contain comprehensive templates that are fully localized for each supported language, ensuring that AI models receive instructions in the appropriate language.

For English prompts, the system uses clear, direct language optimized for AI comprehension, with specific formatting instructions and expected response structures. The Chinese prompts maintain the same logical structure and intent but are expressed in natural Chinese that is optimized for Chinese-speaking AI models.

The localization process for AI prompts goes beyond simple translation, adapting cultural context, technical terminology, and response expectations to the target language. For example, the Chinese prompts use Chinese-specific technical terms and formatting conventions that are more familiar to Chinese-speaking developers and AI models.

The system maintains parallel prompt structures across languages, with identical template variables and formatting markers. This ensures that the dynamic content substitution process works identically regardless of the active language. For instance, template variables like ${filePath} and ${diffContent} are preserved in both language versions, allowing for consistent content injection.

The prompt localization also considers the AI model's language capabilities, with the DIFF_SYSTEM_PROMPT explicitly specifying that suggestions should be provided in the appropriate language (English or Chinese) based on the active locale. This ensures that the AI's output matches the user's language preference.

**Section sources**
- [prompts.ts](file://src/i18n/en/prompts.ts#L1-L108)
- [prompts.ts](file://src/i18n/zh/prompts.ts#L1-L108)

## Testing Considerations

Testing the translation system in CodeKarmic requires a comprehensive approach to verify both translation completeness and context accuracy. The primary testing consideration is ensuring that all translation keys exist in both language implementations, as missing keys will fall back to English but may represent incomplete localization.

A key testing strategy involves validating the structural consistency between language files by comparing the object hierarchies of corresponding translation modules. Automated tests can verify that each category (UI, OUTPUT, PROMPTS) contains the same set of properties and nested objects across all languages.

Context accuracy testing is particularly important for AI prompts, where the translation must preserve the technical meaning and instructional intent of the original text. This requires manual review by bilingual technical experts who can verify that the localized prompts will produce equivalent AI behavior.

The system's fallback mechanism should be tested to ensure it works correctly when translations are missing, verifying that the English fallback is applied appropriately without causing runtime errors. This includes testing edge cases such as deeply nested missing properties and function templates with missing translations.

For dynamic content, testing should verify that template variables are properly preserved and substituted in both languages, ensuring that interpolated values appear correctly in the final output. This is particularly important for messages that include file paths, commit hashes, or other dynamic data.

Finally, integration testing should verify that language changes are properly propagated throughout the application, with all UI elements, output messages, and AI prompts updating to the new language without requiring a restart. This includes testing the event-driven update mechanism to ensure all components respond to language change events.

**Section sources**
- [index.ts](file://src/i18n/index.ts#L97-L102)
- [types.ts](file://src/i18n/types.ts#L24-L36)
- [notificationManager.ts](file://src/services/notification/notificationManager.ts#L73-L99)