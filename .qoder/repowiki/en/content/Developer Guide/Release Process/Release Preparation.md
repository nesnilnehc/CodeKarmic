# Release Preparation

<cite>
**Referenced Files in This Document**   
- [package.json](file://package.json)
- [webpack.config.js](file://webpack.config.js)
- [CHANGELOG.md](file://CHANGELOG.md)
- [docs/release-guide.md](file://docs/release-guide.md)
- [tsconfig.json](file://tsconfig.json)
</cite>

## Table of Contents
1. [Versioning Strategy](#versioning-strategy)
2. [Version Update Process](#version-update-process)
3. [CHANGELOG Maintenance](#changelog-maintenance)
4. [Pre-Release Validation](#pre-release-validation)
5. [Build Process Integration](#build-process-integration)
6. [Common Pitfalls and Troubleshooting](#common-pitfalls-and-troubleshooting)
7. [Internal Consistency Requirements](#internal-consistency-requirements)

## Versioning Strategy

CodeKarmic follows Semantic Versioning 2.0.0 as specified in the project's package.json file. The versioning scheme uses a three-part version number in the format MAJOR.MINOR.PATCH, where each component has a specific meaning:

- **MAJOR**: Incremented when making incompatible API changes or significant architectural changes that could break existing functionality
- **MINOR**: Incremented when adding new functionality in a backward-compatible manner
- **PATCH**: Incremented when making backward-compatible bug fixes

The current version in package.json is defined as "0.2.0", indicating the project is in active development with feature additions and improvements being made while maintaining backward compatibility where possible.

Semantic versioning is critical for managing dependencies and ensuring users understand the impact of updates. Breaking changes require a MAJOR version increment, while new features without breaking changes require a MINOR increment. Bug fixes and minor improvements that don't add new features use PATCH increments.

**Section sources**
- [package.json](file://package.json#L5)

## Version Update Process

To update the version number for CodeKarmic, use the npm version command without creating Git tags by including the --no-git-tag-version flag. This approach allows for version updates while maintaining separate control over Git tagging and release processes.

The recommended command sequence is:
```bash
npm version <new-version> --no-git-tag-version
```

Where `<new-version>` can be specified as:
- A specific version number (e.g., 0.3.0)
- A semantic versioning keyword (e.g., patch, minor, major)

This command automatically updates the version field in package.json and creates a new entry in the CHANGELOG.md file. The --no-git-tag-version flag prevents npm from automatically creating a Git tag, allowing for additional pre-release validation before finalizing the release.

After updating the version, verify that package.json has been modified correctly and that the new version reflects the appropriate semantic versioning level based on the changes included in the release.

**Section sources**
- [package.json](file://package.json#L5)
- [docs/release-guide.md](file://docs/release-guide.md#L50-L52)

## CHANGELOG Maintenance

The CHANGELOG.md file must be maintained according to the Keep a Changelog format and organized into specific categories that reflect the nature of changes. Each release entry should include the version number, release date, and categorized changes.

The required categories are:
- **Added**: New features, functionality, or capabilities
- **Changed**: Modifications to existing features, improvements, or refactoring
- **Fixed**: Bug fixes and issue resolutions
- **Documentation**: Changes to documentation, guides, or help content

When preparing a release, move changes from the "Unreleased" section to a new version entry with the appropriate version number and release date. Ensure all changes are accurately categorized and described in clear, concise language that communicates the impact to users.

The CHANGELOG serves as a critical communication tool for users and developers, providing transparency about what has changed between releases and helping users understand the value and potential impact of upgrading.

**Section sources**
- [CHANGELOG.md](file://CHANGELOG.md#L8-L92)

## Pre-Release Validation

Before proceeding with a release, complete a comprehensive validation process to ensure the extension functions correctly and meets quality standards. This process includes dependency management, compilation, and testing.

### Dependency Installation
Ensure all dependencies are up to date by running:
```bash
npm install
```

This command installs both production and development dependencies specified in package.json, ensuring the build environment has all required packages.

### TypeScript Compilation
Compile the TypeScript code using webpack with the compile script defined in package.json:
```bash
npm run compile
```

This command executes webpack to transpile TypeScript files into JavaScript, following the configuration in webpack.config.js. The build process targets ESNext with node as the target environment and outputs to the dist directory.

### Testing
Run the test suite to validate functionality:
```bash
npm test
```

The pretest script automatically runs compilation, linting, and test preparation before executing tests. This ensures code quality and functionality validation before release.

**Section sources**
- [package.json](file://package.json#L284-L291)
- [webpack.config.js](file://webpack.config.js#L5-L47)
- [tsconfig.json](file://tsconfig.json#L1-L19)

## Build Process Integration

The build process for CodeKarmic is integrated through npm scripts and webpack configuration, creating a streamlined workflow for compilation and packaging. The key integration points are:

### npm Scripts
The package.json defines several scripts that orchestrate the build process:
- **compile**: Executes webpack to compile TypeScript code
- **watch**: Runs webpack in watch mode for development
- **package**: Builds for production with optimization
- **pretest**: Runs compilation, tests, and linting before testing

### Webpack Configuration
The webpack.config.js file configures the build process with the following key settings:
- Entry point: src/extension.ts
- Output: dist/extension.js with commonjs2 library target
- Target environment: node
- Mode: production
- Source maps: hidden-source-map
- Module resolution: TypeScript loader with node module resolution

### Compilation Command
The npm run compile command is central to the build process, triggering webpack to process the extension's TypeScript code. This command must succeed without errors before proceeding with release preparation.

The build process transpiles TypeScript to JavaScript, bundles dependencies, and outputs a single file that can be packaged as a VS Code extension. The output is optimized for production use with minimized code and hidden source maps.

**Section sources**
- [package.json](file://package.json#L284-L286)
- [webpack.config.js](file://webpack.config.js#L5-L47)

## Common Pitfalls and Troubleshooting

Several common issues can occur during release preparation. Understanding these pitfalls and their solutions ensures a smooth release process.

### Missing Dependency Updates
**Issue**: Forgetting to run npm install after changes to package.json
**Solution**: Always run npm install after updating dependencies or cloning the repository
**Verification**: Check that node_modules contains all required packages

### Build Failures
**Issue**: TypeScript compilation errors or webpack build failures
**Troubleshooting steps**:
1. Verify TypeScript syntax in all source files
2. Check tsconfig.json configuration
3. Ensure all imports are valid and files exist
4. Validate webpack.config.js settings
5. Check for version compatibility between dependencies

### Test Execution Issues
**Issue**: Tests failing or not running
**Solution**:
- Ensure the out directory exists for test output
- Verify test files are properly configured
- Check that all test dependencies are installed
- Validate that the test runner can access required resources

### Versioning Mistakes
**Issue**: Incorrect semantic versioning application
**Prevention**:
- Review all changes to determine appropriate version increment
- Consult team members for significant changes
- Document the rationale for version decisions

### CHANGELOG Errors
**Issue**: Improperly formatted or incomplete changelog entries
**Solution**:
- Follow the Keep a Changelog format strictly
- Categorize all changes appropriately
- Include all significant changes
- Verify dates and version numbers

**Section sources**
- [package.json](file://package.json#L283-L291)
- [webpack.config.js](file://webpack.config.js#L5-L47)
- [tsconfig.json](file://tsconfig.json#L1-L19)
- [CHANGELOG.md](file://CHANGELOG.md#L8-L92)

## Internal Consistency Requirements

Before proceeding to packaging, ensure internal consistency across all project components. This verification process confirms that all elements align with the release version and function cohesively.

### Version Synchronization
Verify that the version number is consistent across:
- package.json
- CHANGELOG.md
- Documentation references
- Any embedded version strings in code

### Documentation Alignment
Ensure all documentation reflects the current release:
- Update release notes with accurate information
- Verify installation instructions
- Confirm feature descriptions match current functionality
- Check configuration options and defaults

### Functionality Verification
Test key workflows to ensure they function as expected:
- Extension activation and startup
- Core feature execution
- Configuration management
- Error handling scenarios

### Quality Gates
Confirm all quality requirements are met:
- All tests pass successfully
- Code passes linting rules
- Build completes without warnings or errors
- CHANGELOG is complete and properly formatted

Internal consistency ensures a professional, reliable release that provides a positive user experience and maintains the project's reputation for quality.

**Section sources**
- [package.json](file://package.json#L5)
- [CHANGELOG.md](file://CHANGELOG.md#L8-L92)
- [docs/release-guide.md](file://docs/release-guide.md#L70-L80)