# Java 25 Upgrade Log

**Timestamp**: 2026-01-26T10:54:08
**Completion Time**: 2026-01-26T11:13:27
**Project**: realworld-java21-ai-assisted-upgrade
**Upgrade**: Java 21 to Java 25
**Status**: Completed Successfully
**Total Execution Time**: ~20 minutes

---

## Summary

Successfully upgraded a multi-module Spring Boot application from Java 21 to Java 25. The upgrade required updating the Java version, Gradle wrapper, library versions, and code formatter due to Java 25 compatibility requirements.

**Key Changes:**
- Java version: 21 -> 25
- Gradle wrapper: 8.5 -> 9.3.0
- Spring Boot: 3.3.0 -> 3.5.3 (required for Java 25 compatibility)
- Spring Dependency Management: 1.1.5 -> 1.1.7
- Spotless plugin: 7.0.3 -> 7.0.4
- Lombok: Added explicit version 1.18.42
- Code formatter: Changed from palantirJavaFormat to eclipse (Java 25 compatibility)

---

## Step 1: Project Root Directory

**Timestamp**: 2026-01-26T10:54:08
**Status**: Verified

### Current Configuration
- **Project Root**: /Users/sri/projects/git-sandbox/explore/sri-chalam/realworld-java21-ai-assisted-upgrade
- **Build Files Found**:
  - ./build.gradle.kts (root)
  - ./module/core/build.gradle.kts
  - ./module/persistence/build.gradle.kts
  - ./server/api/build.gradle.kts
- **Project Type**: Multi-module Gradle project with Kotlin DSL
- **Version Catalog**: gradle/libs.versions.toml

### Issues Encountered
- None

### Decision
- Project root validated successfully. Proceeded with upgrade.

---

## Step 2: Java Version Detection

**Timestamp**: 2026-01-26T10:54:08
**Status**: Verified

### Current Configuration
- **Java Version Detected**: 21
- **Detection Location**: gradle/libs.versions.toml (line 2: `java = "21"`)
- **Build Files Checked**:
  - ./build.gradle.kts - Uses `JavaLanguageVersion.of(versionCatalog.versions.java.get())`
  - ./gradle/libs.versions.toml - Contains `java = "21"`
- **Variable Resolution**: Version resolved from version catalog

### Decision
- Java 21 detected - proceeded with upgrade to Java 25

---

## Step 3: SDKMAN Installation

**Timestamp**: 2026-01-26T10:54:30
**Status**: Skipped - Already Installed

### Current Configuration
- **SDKMAN Version**: 5.20.0
- **SDKMAN Path**: ~/.sdkman/bin/sdkman-init.sh

---

## Step 4: Install Amazon Corretto Java 25

**Timestamp**: 2026-01-26T10:55:00
**Status**: Installed

### Configuration
- **Java Version Installed**: 25.0.1-amzn
- **Installation Method**: SDKMAN (`sdk install java 25.0.1-amzn`)
- **JAVA_HOME**: /Users/sri/.sdkman/candidates/java/25.0.1-amzn
- **Certificate Import**: Skipped (no ~/trusted-certs directory)

---

## Step 5: Update Project Configuration

**Timestamp**: 2026-01-26T10:56:00
**Status**: Completed

### Configuration
- **JAVA_HOME**: /Users/sri/.sdkman/candidates/java/25.0.1-amzn
- **Java Version**: openjdk version "25.0.1" 2025-10-21 LTS

---

## Step 6: Gradle Wrapper Update

**Timestamp**: 2026-01-26T10:57:00
**Status**: Updated

### Configuration
- **Previous Gradle Version**: 8.5
- **Updated Gradle Version**: 9.3.0
- **Update Method**: `./gradlew wrapper --gradle-version=9.3.0` (executed with Java 21 first, then verified with Java 25)

### Note
- Gradle 8.5 was incompatible with Java 25 for the wrapper upgrade command
- Used Java 21 to perform the wrapper upgrade, then switched to Java 25

---

## Step 6a: Library Upgrades

**Timestamp**: 2026-01-26T10:58:00
**Status**: Completed

### Libraries Checked
- **Lombok**: Present - Added explicit version 1.18.42 (was managed by Spring BOM)
- **MapStruct**: Not present - Skipped
- **google-java-format**: Not present - Spotless already configured with palantirJavaFormat

### Changes Made
- Added `lombok = "1.18.42"` to gradle/libs.versions.toml
- Updated lombok library definition to use version.ref

---

## Step 7: OpenRewrite Migration

**Timestamp**: 2026-01-26T10:59:00
**Status**: Completed

### OpenRewrite Configuration
- **Plugin Version**: 7.25.0
- **BOM Version**: 3.23.0
- **Recipes Executed**:
  - org.openrewrite.java.migrate.UpgradeToJava25
  - org.openrewrite.java.migrate.SwitchPatternMatching

### Files Modified by OpenRewrite
1. server/api/src/main/java/io/zhc1/realworld/RealWorldApplication.java - MigrateMainMethodToInstanceMain
2. module/core/src/main/java/io/zhc1/realworld/upgradetest/AnonymousClassExample.java - ReplaceSystemOutWithIOPrint
3. module/core/src/main/java/io/zhc1/realworld/upgradetest/TextBlockExample.java - UseTextBlocks
4. module/core/src/main/java/io/zhc1/realworld/upgradetest/InstanceOfExample.java - InstanceOfPatternMatch
5. module/core/src/main/java/io/zhc1/realworld/upgradetest/OptionalExample.java - ReplaceSystemOutWithIOPrint
6. module/persistence/src/main/java/io/zhc1/realworld/persistence/ArticleSpecifications.java - ReplaceUnusedVariablesWithUnderscore

---

## Build/Fix Loop Summary

### Iteration 1
- **Error**: Unsupported class file major version 69
- **Root Cause**: Spring Boot 3.3.0 plugin doesn't support Java 25 class files
- **Fix**: Upgraded Spring Boot to 3.5.3, Spring Dependency Management to 1.1.7
- **Justification**: Framework upgrade strictly required for Java 25 compilation

### Iteration 2
- **Error**: palantir-java-format NoSuchMethodError (internal compiler API change)
- **Root Cause**: Neither palantir-java-format nor google-java-format support Java 25's internal compiler API changes
- **Fix**: Switched to Eclipse formatter in Spotless configuration

### Iteration 3
- **Error**: Main class name could not be resolved
- **Root Cause**: OpenRewrite's MigrateMainMethodToInstanceMain recipe converted static main to instance main
- **Fix**: Reverted RealWorldApplication.java main method to `public static void main(String[] args)`
- **Justification**: Spring Boot requires traditional static main method for main class detection

### Iteration 4
- **Result**: BUILD SUCCESSFUL
- **Tests**: All tests passed

---

## Step 8: Dockerfile Updates

**Status**: Not Applicable - No Dockerfiles found in repository

---

## Step 9: GitHub Actions Workflow Updates

**Status**: Not Applicable - No workflow files found in .github/workflows

---

## Step 10: AWS CodeBuild buildspec Updates

**Status**: Not Applicable - No buildspec files found in repository

---

## Step 11: Final Build and Test Verification

**Timestamp**: 2026-01-26T11:13:27
**Status**: Success

### Build Results
- **Build Command**: `./gradlew clean build`
- **Build Status**: BUILD SUCCESSFUL
- **Build Time**: 14 seconds

### Test Results
- **All tests passed**
- **JVM Warnings**: Sharing warning (expected for Spring Boot tests)

### Build Artifacts
- **Location**: server/api/build/libs/
- **Artifact**: realworld.jar (58 MB)

### Verification
- `java -version` reports: openjdk version "25.0.1" 2025-10-21 LTS
- Build succeeds without Java version-related errors
- All tests pass

---

## Files Changed Summary

### Configuration Files
- build.gradle.kts - Added OpenRewrite plugin, changed formatter to Eclipse
- gradle/libs.versions.toml - Updated Java to 25, Spring Boot to 3.5.3, added Lombok version
- gradle/wrapper/gradle-wrapper.properties - Updated to Gradle 9.3.0
- gradlew, gradlew.bat - Updated wrapper scripts

### Java Source Files
- Multiple files reformatted by Eclipse formatter (style change only)
- OpenRewrite applied Java 25 migration patterns to 6 files
- RealWorldApplication.java - Reverted main method to static (Spring Boot compatibility)

---

## Warnings and Notes

1. **Deprecation Warning**: `Specification.where()` in ArticleRepositoryAdapter.java is deprecated and marked for removal
   - This is a Spring Data JPA deprecation, not a Java 25 issue
   - Should be addressed in a future maintenance task

2. **JVM Warnings**: Native access warnings from Gradle's native-platform library
   - These are informational warnings and don't affect functionality
   - Will be resolved in future Gradle updates

3. **Code Formatter Change**: Switched from palantirJavaFormat to Eclipse formatter
   - Required due to Java 25 internal compiler API changes
   - Both google-java-format and palantir-java-format don't support Java 25 yet

---

## Final Status: SUCCESS

The Java 21 to Java 25 upgrade has been completed successfully. The application builds and all tests pass with Java 25.0.1 Amazon Corretto.
