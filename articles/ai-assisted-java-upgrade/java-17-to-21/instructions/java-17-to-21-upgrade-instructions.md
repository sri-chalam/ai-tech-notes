# Java 21 Upgrade Instructions

## AI Execution Contract — Mandatory

This document is designed to be executed directly by an LLM.
The following rules are mandatory and must be followed strictly.

## Role
You are an AI agent tasked with upgrading a Java application from Java 17 to Java 21.

## Strict Execution Rules
- Follow **only** the steps defined in this document
- Do not add, remove, reorder, or optimize steps
- Do not introduce tools, commands, or approaches not explicitly listed
- Abort or warn when conditions specified in this document are not met

## Step Execution Requirements
For every step:
- Detect current state before acting
- Output status: Installed | Updated | Skipped | Verified | Failed
- Verify success before proceeding
- Use detected values (versions, paths), never hardcoded values
- Stop execution if a blocking condition is encountered

## Sequencing
Steps must be executed in the exact order defined.
No step may be skipped unless explicitly allowed.

## Output Expectations
- Clear progress after each step
- Explicit warnings when assumptions are violated
- Final summary of changes

---

## Executive Summary
This document guides AI coding agents through upgrading Java applications from version 17 to 21 using Amazon Corretto JDK on macOS. The process includes:
- JDK installation and configuration
- Gradle wrapper upgrades
- Automated code migration using OpenRewrite
- Iterative build/test/fix cycles with structured error resolution
- CI/CD configuration updates (Docker, GitHub Actions, AWS CodeBuild)

**Expected Duration:** Variable (depends on codebase size and errors)
**Prerequisites:** Java 17 application, Gradle build system
**Success Criteria:** Clean build with all tests passing on Java 21

---

## Core Execution Rules

If required information is missing or ambiguous:
- **DO NOT** guess
- **DO NOT** invent defaults
- **DO NOT** assume standard configurations
- Log the uncertainty and stop at the nearest safe boundary

---

## How to Use This Instruction File

- Execute steps strictly in order unless explicitly redirected.
- Do not infer missing steps or optimize the process.
- Abort means stop the entire workflow immediately.
- Skipped steps must be explicitly logged as skipped.
- 🔴 **CRITICAL:** All shell scripts must be executed from the project root directory (the directory containing `gradlew`, `build.gradle`, or `pom.xml`). Step 1 establishes this directory at the start of execution.

### Keywords and Meanings

- **ABORT**: Stop execution immediately; do not proceed further.
- **SKIP**: Do not perform this step; continue to the next step.
- **DO NOT MODIFY**: No file changes are allowed under any circumstances.
- **CRITICAL PATH**: Failure here aborts the upgrade.
- **SAME ERROR**: An error is considered the same if:
  - The exception type is identical AND
  - The primary error message is identical
  
---

## Non-Goals

This upgrade does NOT attempt to:
- Improve code quality
- Refactor legacy APIs
- Introduce Java 21 language features
- Optimize build performance

---

## Upgrade Process Flow

START

EXECUTION RULES:
- Follow steps strictly in order.
- Do not invent steps or tools.
- "ABORT" means stop the entire upgrade process immediately.
- "DO NOT MODIFY" means no file changes of any kind.
- Loops must not exceed stated limits.

Step 1: Set Project Root Directory
- Prompt user to enter or confirm the project root directory path
- Validate the directory exists and contains build files (`gradlew`, `build.gradle`)
- Change to the project root directory
- All subsequent scripts will execute from this directory

Step 2: Detect Java Version
- If Java 17 → CONTINUE
- If Java 21 → ABORT (Upgrade already completed; no migration required)
- If other or not detected → CONTINUE with caution

Step 3: SDKMAN Setup
- If OS is macOS or Linux AND SDKMAN is not installed → Install SDKMAN
- Otherwise → SKIP

Step 4: JDK Installation
- If Java 21 JDK is not installed → Install JDK 21 using SDKMAN
- Import required certificates if applicable
- If JDK 21 is already installed → SKIP

Step 5: Configure Environment
- Set JAVA_HOME to JDK 21
- Update PATH if required

Step 6: Build Tool Detection
- If Gradle project:
    - Detect Gradle Wrapper version
    - If wrapper < 8.5 → Upgrade wrapper
    - Else → SKIP
- If Maven project → SKIP (DO NOT MODIFY Maven files)

Step 6a: Upgrade Known Java 21 Incompatible Libraries and Plugins
- If Lombok present AND version < 1.18.30 → Upgrade to latest version
- If MapStruct present AND version < 1.5.5 → Upgrade to latest version
- If google-java-format plugin present → Comment out and add Spotless plugin
- Verify upgraded dependencies resolve successfully

Step 7: OpenRewrite Java Upgrade
[CRITICAL PATH]

Build/Fix Loop:
- Run build
- If build fails:
    - Fix issues
    - Repeat
- Maximum 5 iterations
- If still failing after 5 → ABORT

Test/Fix Loop:
- Run tests
- If tests fail:
    - Fix issues
    - Track failure signature (exception type + primary message)
- If same failure signature occurs in 3 iterations → ABORT
- Maximum 5 iterations
- If still failing after 5 → ABORT

Step 8–10: CI/CD Updates
- If Dockerfile exists → Update base image to Java 21
- If GitHub Actions exists → Update Java version
- If AWS CodeBuild exists → Update runtime
- If none exist → SKIP section

Step 11: Final Verification

Verification requires:
- `java -version` reports 21
- Build succeeds without warnings related to Java version
- Test suite passes

END


## Common Pitfalls to Avoid

### 🚨 **CRITICAL — AI Agents MUST NOT violate any rule in this section**

Violating any rule below is considered a **critical failure**.  
If any instruction elsewhere conflicts with this section, **this section takes precedence**.

---

### 1. **Modify Maven-Based Projects**
- If `pom.xml` exists, the project is Maven-based.
- **DO NOT**:
  - Add Gradle files or Gradle wrapper
  - Add OpenRewrite Gradle plugins
  - Convert Maven projects to Gradle
- Maven projects must be **explicitly skipped** unless a separate Maven-specific instruction file is provided.

---

### 2. **Upgrade Spring Boot or Framework Versions**
- **DO NOT** upgrade:
  - Spring Boot
  - Spring Framework
  - Jakarta EE or other framework versions
- Java upgrade and framework upgrade are **separate tasks** with **separate instructions**.
- Framework-related changes are allowed **only if strictly required** to make the Java 21 build compile, and must be clearly documented.

---

### 3. **Assume Single-Module Gradle Structure**
- **DO NOT** assume:
  - Plugins live in the root `build.gradle(.kts)`
  - Dependencies are declared only once
- Always:
  - Inspect root and submodule build files
  - Respect aggregator-only root projects
  - Modify only the files that actually own the configuration

---

### 4. **Add OpenRewrite Configuration to the Wrong Build File**
- **DO NOT** blindly add OpenRewrite plugins or dependencies to:
  - Root build files with no plugins
  - Aggregator-only builds
- Add OpenRewrite only where other plugins already exist.
- In multi-module projects, this may require **multiple targeted updates**.

---

### 5. **Re-run Previously Executed OpenRewrite Recipes**
- Once a migration recipe has completed:
  - **DO NOT** re-run it during error resolution
- During build or test failures:
  - Only run **new, targeted recipes** that directly address the current error
  - Never re-run the full migration set

---

### 6. **Batch Multiple Fixes Before Verification**
- **DO NOT** apply multiple fixes without verification in between.
- The correct sequence is:
  1. Apply **one fix**
  2. Re-run build or tests
  3. Verify the result
- This prevents error masking and simplifies rollback.

---

### 7. **Ignore Iteration Limits or Stalled Progress Detection**
- **DO NOT**:
  - Exceed maximum iteration counts
  - Continue looping when the same error appears repeatedly
- If:
  - The same error appears in **three consecutive iterations**, or
  - Maximum iteration limits are reached
- Then:
  - **STOP**
  - Document the issue
  - Request human intervention

---

### 8. **Skip or Delay Logging**
- Logging is **mandatory**.
- Every major action must be logged **immediately**, including:
  - What was changed
  - Why it was changed
  - Result (success or failure)
- Logs must be written to: `/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`


---

### 9. **Mark Tasks or TODOs as Complete Before Verification**
- **DO NOT** mark:
- Steps
- Fixes
- TODOs
as complete until:
- Build succeeds
- Tests pass
- Verification steps are executed
- Completion without verification is considered invalid.

---

### 10. **Introduce Unrequested Optimizations or Refactoring**
- **DO NOT**:
- Refactor code for style or readability
- Introduce new Java 21 features unless strictly required
- Modify unrelated code
- All changes must be:
- Minimal
- Directly required for Java 21 compatibility

---

### 11. **Assume Tool or Platform Availability**
- **DO NOT** assume:
- CI/CD tools exist unless detected
- Docker is available locally
- SDKMAN is available on unsupported platforms
- Always detect tools and platforms before acting.
- Skip steps cleanly when tools are not present.

---

### 12. **Proceed After an Explicit Abort Condition**
- **DO NOT** continue execution after:
- Java 21 is already configured (upgrade must be aborted)
- Iteration limits are exceeded
- Critical failures are encountered
- Abort means **stop immediately and document the reason**.

---

### Final Note to AI Agents
This upgrade is a **controlled, auditable migration**, not an optimization exercise.  
Correctness, traceability, and restraint are more important than speed.

## Overview
These instructions guide an AI coding agent to upgrade a Java application from Java 17 to Java 21 using Amazon Corretto JDK.

## Environment Assumptions
- Platform: macOS (Macbook)
- Default shell: bash
- curl is already installed
- JDK Provider: Amazon Corretto
- Build tool: Gradle with Groovy DSL (build.gradle) or Kotlin DSL (build.gradle.kts)
  - **Note:** Instructions are specific to Gradle. For Maven-based projects, this file requires adaptation.

## Important: Build Tool Scope

**ONLY upgrade Gradle-based projects.** This upgrade applies exclusively to:
- Gradle wrapper files (`gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.properties`, `gradle/wrapper/gradle-wrapper.jar`)
- `build.gradle` or `build.gradle.kts`
- `gradle.properties`
- `settings.gradle` or `settings.gradle.kts`

**DO NOT modify Maven-based projects.** Specifically, do not modify:
- Maven wrapper files (`mvnw`, `mvnw.cmd`, `.mvn/`)
- `pom.xml`
- Maven-specific configuration files

If the project uses Maven instead of Gradle, skip all Gradle-specific steps (Steps 5 and 6).

**IMPORTANT: DO NOT upgrade Spring Boot version during the Java upgrade.** Spring Boot will be upgraded in a separate task using a separate instruction file. This instruction file is focused solely on upgrading Java from version 17 to version 21.

## Gradle Project Structure Patterns

**CRITICAL:** This section describes how Gradle projects can be structured. AI agents must understand these patterns to correctly locate and modify build files throughout the upgrade process.

### Single-Module Projects

In single-module projects:
- Root directory contains: `build.gradle` or `build.gradle.kts`
- All plugins and dependencies are declared in the root build file
- Simple, straightforward structure

### Multi-Module Projects

In multi-module projects:
- **Root directory** may contain a minimal `build.gradle` or `build.gradle.kts` that only has project-wide configuration (no plugins or dependencies)
- **Submodules** in subdirectories (e.g., `app/`, `core/`, `service/`, `api/`) each have their own `build.gradle` or `build.gradle.kts` files
- **Plugins and dependencies** may be declared in:
  - Root build file only (applied to all modules)
  - Individual submodule build files only
  - Both root and submodule build files (with different configurations per module)
  - The root build file may be empty or minimal, with all actual build logic in submodules

### Dependency Declaration Locations

Dependencies can be declared in multiple locations:
- **Traditional approach:** `build.gradle` or `build.gradle.kts` (at root level or in any submodule)
- **Version catalogs:** `gradle/libs.versions.toml` or other `gradle/*.toml` files

### Variable Declaration Locations

Version numbers and other values may be referenced as variables defined in:
- **gradle.properties:** Simple key-value pairs (e.g., `javaVersion=17`)
- **ext {} block:** Extra properties in `build.gradle` or `build.gradle.kts` (commonly used in multi-module projects to define shared values in the root build file)
  ```groovy
  ext {
      javaVersion = '17'
      springBootVersion = '3.1.0'
  }
  ```

### Important Guidelines for AI Agents

When these instructions reference `build.gradle`, `build.gradle.kts`, or dependency files, the AI agent must:

1. **Search comprehensively:**
   - First check the root directory
   - If not found OR if the root file is minimal (no plugins/dependencies), search all subdirectories
   - Use `find` command to locate all build files: `find . -type f \( -name "build.gradle" -o -name "build.gradle.kts" \)`

2. **Add to the correct location:**
   - Add plugins and dependencies to build files that already contain them
   - Do NOT blindly add to the root build file if it's empty or minimal
   - For multi-module projects, you may need to modify multiple build files

3. **Check for version catalogs:**
   - Look for `gradle/libs.versions.toml` or other `.toml` files
   - Update dependency versions in the appropriate location (build file vs. version catalog)

4. **Verify variable references:**
   - Dependencies may reference variables from `gradle.properties` or other build files
   - Resolve variable values before making changes

**Throughout this document, whenever build files are mentioned, refer back to this section to ensure correct file location and modification.**

## Guidelines for AI Agent Execution

When executing these instructions, the AI agent should:

1. **Provide Clear Feedback**: After each step, provide clear feedback about what action was taken (e.g., "Installed", "Skipped - already present", "Updated", "Verified")

2. **Echo Status Messages**: Include echo statements in scripts to inform about the current state and progress

3. **Handle Errors Gracefully**: Check for errors and provide meaningful error messages when operations fail

4. **Verify Before Proceeding**: Verify that each step completed successfully before moving to the next step

5. **Avoid Redundant Operations**: Check if a component is already installed/configured before attempting to install/configure it again

6. **Use Dynamic Values**: Prefer dynamically detecting versions and paths over hardcoded values to ensure the instructions work at the time of execution

7. **Document All Changes**: Maintain an upgrade log file at `/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md` to track all actions, decisions, and outcomes throughout the entire upgrade process (see "Logging Requirements" section below for detailed instructions)

8. **Multi-Module Project Awareness**: Always refer to the "Gradle Project Structure Patterns" section when locating and modifying build files. Do not assume all projects have a single root `build.gradle` file.

---

## Logging Requirements

**CRITICAL:** Throughout the entire upgrade process, maintain a detailed log file to track all actions, decisions, and outcomes. This log provides a complete audit trail and helps with troubleshooting if issues arise.

### Log File Location

Create and maintain the upgrade log at:

```
/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md
```

**Before starting Step 1**, create the directory structure and initialize the log file:

```bash
mkdir -p docs/ai-tasks/logs
touch docs/ai-tasks/logs/java-17-to-21-upgrade-log.md
```

### Log File Structure

Initialize the log file with the following structure:

```markdown
# Java 21 Upgrade Log

**Timestamp**: YYYY-MM-DDTHH:MM:SS
**Execution Time**: X seconds
**Project**: [Project Name]
**Upgrade**: Java 17 to Java 21
**Status**: In Progress | Completed | Blocked
**Total Execution Time**: X minutes Y seconds

---

## Summary

Brief overview of the upgrade process and overall status. Update this section as you progress.

---

## Step 2: Java Version Detection

**Timestamp**: YYYY-MM-DDTHH:MM:SS
**Execution Time**: X seconds

### Current Configuration
- **Java Version Detected**: [version or "Not detected"]
- **Detection Location**: [e.g., build.gradle, gradle.properties, ext{} block]
- **Build Files Checked**:
  - ./build.gradle
  - ./app/build.gradle
  - [list all files checked]
- **Variable Resolution**: [Any variables resolved and their values]

### Issues Encountered
- [List any issues or "None"]

### Decision
- [Proceeding with upgrade | Skipping - already Java 21 | Continuing despite detection issues]

---

## Build/Test Summary

- **Total Build Iterations**: X
- **Total Test Iterations**: X
- **Total Fixes Applied**: X
- **Unresolved Errors**: X
- **Tests Passed**: X / Y
- **Final Status**: [In Progress | Success | Needs Manual Intervention]
```

### Logging Best Practices

1. **Log immediately** after completing each major action or step
2. **Be specific** about file paths, line numbers, and versions
3. **Include timestamps** in the Date fields (use YYYY-MM-DDTHH:MM:SS format)
4. **Capture timestamps at execution time** - Do NOT pre-generate or estimate timestamps. At the START of each step (after user approval), run `date -Iseconds` to get the real current time. This ensures accurate timing even when there are pauses between steps.
5. **Document both successes and failures**
6. **Keep entries concise but informative** - focus on what was done and why
7. **Update the Summary section** as you progress through the upgrade
8. **Update Status field** at the top of the log file as the upgrade progresses

### Integration with Steps

At the beginning of each major step, add an entry to the log file. At the end of each step, update the entry with results. This creates a complete audit trail of the entire upgrade process.

---

## Upgrade Steps

### Step 1: Set Project Root Directory

**Purpose:** Establish the working directory for all subsequent commands. All shell scripts in this document must be executed from the project root directory.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 1:**
Update the "Step 1: Project Root Directory" section in the log file with:
- Project root directory path
- Validation result (which build files were found)
- Any issues encountered

#### 1.1 Prompt for Project Root Directory

<!-- In most cases, step 1.1 is not needed.
  By following the instructions in step 1.2, the LLM can identify the project root directory on its own, without prompting the user to provide it.
 -->

Ask the user to enter the project root directory path:

```bash
echo "=========================================="
echo "Step 0: Set Project Root Directory"
echo "=========================================="
echo ""
echo "All scripts in this upgrade process must be executed from the project root directory."
echo "The project root is the directory containing gradlew, build.gradle, or pom.xml."
echo ""
read -p "Enter the project root directory path (or press Enter for current directory): " PROJECT_ROOT

# Use current directory if no input provided
if [ -z "$PROJECT_ROOT" ]; then
    PROJECT_ROOT="$(pwd)"
    echo "Using current directory: $PROJECT_ROOT"
fi

# Expand ~ to home directory if used
PROJECT_ROOT="${PROJECT_ROOT/#\~/$HOME}"

# Verify directory exists
if [ ! -d "$PROJECT_ROOT" ]; then
    echo ""
    echo "ERROR: Directory does not exist: $PROJECT_ROOT"
    echo "ABORT: Cannot proceed without a valid project root directory."
    exit 1
fi

echo ""
echo "Project root set to: $PROJECT_ROOT"
```

#### 1.2 Validate Project Root Directory

Verify the directory contains expected build files:

```bash
cd "$PROJECT_ROOT" || { echo "ERROR: Cannot change to directory: $PROJECT_ROOT"; exit 1; }

echo ""
echo "Validating project root directory..."
echo ""

VALID_PROJECT=false

if [ -f "gradlew" ]; then
    echo "✓ Found: gradlew"
    VALID_PROJECT=true
fi

if [ -f "build.gradle" ]; then
    echo "✓ Found: build.gradle"
    VALID_PROJECT=true
fi

if [ -f "build.gradle.kts" ]; then
    echo "✓ Found: build.gradle.kts"
    VALID_PROJECT=true
fi

if [ -f "pom.xml" ]; then
    echo "✓ Found: pom.xml"
    VALID_PROJECT=true
fi

if [ "$VALID_PROJECT" = false ]; then
    echo ""
    echo "WARNING: No build files (gradlew, build.gradle, build.gradle.kts, pom.xml) found in: $PROJECT_ROOT"
    echo ""
    read -p "Continue anyway? (y/n): " CONTINUE_ANYWAY
    if [[ ! "$CONTINUE_ANYWAY" =~ ^[Yy]$ ]]; then
        echo "ABORT: User chose not to continue without build files."
        exit 1
    fi
    echo "Continuing as requested by user..."
else
    echo ""
    echo "✓ Project root directory validated successfully"
fi

echo ""
echo "Current working directory: $(pwd)"
echo ""
echo "All subsequent commands will execute from this directory."
echo ""
```

---

### Step 2: Verify Current Java Version (Prerequisite Check)

**IMPORTANT: These instructions are designed to upgrade Java 17 applications to Java 21 only.** Before proceeding, verify that the application is currently using Java 17.

**Before starting Step 2:** Initialize the upgrade log file as described in the "Logging Requirements" section.

**Logging for this step:** Update the "Step 2: Java Version Detection" section in the log file with:
- Current Java version detected (or "Not detected")
- Location where version was found (build.gradle, gradle.properties, ext{} block, version catalog)
- All build files checked (list each file path)
- Any variables resolved and their values
- Any issues with version detection (e.g., variable resolution failed, edge cases, complex patterns)
- Decision made (Proceeding with upgrade | Skipping - already Java 21 | Continuing despite detection issues)

#### 2.1 Check Java Version in Project Configuration

<!--
Important Note: Java version can be configured in Gradle using many different patterns. Variables may be resolved through functions, project properties (e.g., "project.javaVersion"), or external property references. The script below handles common patterns but may not cover all edge cases.

If the script cannot detect the Java version due to edge cases, there are two options:
1. Manually set a hard-coded Java version in the Gradle configuration files
2. Skip the Java version check step entirely

Skipping the version check is safe because the OpenRewrite plugin will handle the upgrade intelligently:
- If the Java version is already 21, the Rewrite recipe will not make any changes
- If the Java version cannot be identified, the remaining upgrade instructions will still execute successfully
-->

Check the Java version specified in the project's Gradle configuration files:

```bash
echo "Checking current Java version in project configuration..."
echo ""

# Function to resolve variable value from gradle.properties or build files
resolve_variable() {
    local var_name=$1
    local value=""

    # First check gradle.properties
    if [ -f "gradle.properties" ]; then
        value=$(grep -E "^${var_name}\s*=" gradle.properties | sed -E "s/^${var_name}\s*=\s*//" | tr -d ' "' | head -1)
    fi

    # If not found in gradle.properties, check build.gradle files
    if [ -z "$value" ] && [ -n "$BUILD_FILES" ]; then
        while IFS= read -r build_file; do
            value=$(grep -E "^\s*${var_name}\s*=" "$build_file" | sed -E "s/.*=\s*//" | tr -d ' "' | head -1)
            if [ -n "$value" ]; then
                break
            fi
        done <<< "$BUILD_FILES"
    fi

    echo "$value"
}

# Find all build.gradle and build.gradle.kts files
BUILD_FILES=$(find . -type f \( -name "build.gradle" -o -name "build.gradle.kts" \) 2>/dev/null)

JAVA_VERSION_BUILD_GRADLE=""

if [ -n "$BUILD_FILES" ]; then
    echo "Found build files:"
    echo "$BUILD_FILES"
    echo ""

    # Check each build file for Java version
    while IFS= read -r build_file; do
        echo "Checking $build_file..."

        # Check for sourceCompatibility/targetCompatibility with direct value
        VERSION_COMPAT=$(grep -E "sourceCompatibility|targetCompatibility" "$build_file" | grep -o "1[0-9]\+\|[0-9]\+" | head -1)

        # Check if sourceCompatibility/targetCompatibility references a variable
        if [ -z "$VERSION_COMPAT" ]; then
            VAR_REF=$(grep -E "sourceCompatibility|targetCompatibility" "$build_file" | sed -E 's/.*=\s*//;s/["\047]//g' | tr -d ' ' | head -1)
            if [ -n "$VAR_REF" ] && [[ ! "$VAR_REF" =~ ^[0-9]+$ ]]; then
                # It's a variable reference, try to resolve it
                RESOLVED=$(resolve_variable "$VAR_REF")
                if [[ "$RESOLVED" =~ ^[0-9]+$ ]]; then
                    VERSION_COMPAT="$RESOLVED"
                    echo "Resolved variable $VAR_REF to: $VERSION_COMPAT"
                fi
            fi
        fi

        # Check for JavaVersion.VERSION_XX enum syntax
        VERSION_ENUM=$(grep -E "JavaVersion\.VERSION_" "$build_file" | grep -oE "VERSION_[0-9]+" | grep -o "[0-9]\+" | head -1)

        # Check for Java toolchain syntax: languageVersion = JavaLanguageVersion.of(17)
        VERSION_TOOLCHAIN=$(grep -E "languageVersion\s*=\s*JavaLanguageVersion\.of\(" "$build_file" | grep -o "[0-9]\+" | head -1)

        # Check if toolchain uses a variable reference
        if [ -z "$VERSION_TOOLCHAIN" ]; then
            VAR_REF=$(grep -E "languageVersion\s*=\s*JavaLanguageVersion\.of\(" "$build_file" | sed -E 's/.*JavaLanguageVersion\.of\(//;s/\).*//;s/["\047]//g' | tr -d ' ' | head -1)
            if [ -n "$VAR_REF" ] && [[ ! "$VAR_REF" =~ ^[0-9]+$ ]] && [[ ! "$VAR_REF" =~ JavaVersion\.VERSION ]]; then
                # It's a variable reference, try to resolve it
                RESOLVED=$(resolve_variable "$VAR_REF")
                if [[ "$RESOLVED" =~ ^[0-9]+$ ]]; then
                    VERSION_TOOLCHAIN="$RESOLVED"
                    echo "Resolved variable $VAR_REF to: $VERSION_TOOLCHAIN"
                fi
            fi
        fi

        # Check for Java toolchain with enum syntax: languageVersion = JavaLanguageVersion.of(JavaVersion.VERSION_17.majorVersion)
        VERSION_TOOLCHAIN_ENUM=$(grep -E "languageVersion\s*=\s*JavaLanguageVersion\.of\(JavaVersion\.VERSION_" "$build_file" | grep -oE "VERSION_[0-9]+" | grep -o "[0-9]\+" | head -1)

        # Use whichever version was found (prefer toolchain > toolchain enum > enum > compat)
        FILE_VERSION=${VERSION_TOOLCHAIN:-${VERSION_TOOLCHAIN_ENUM:-${VERSION_ENUM:-$VERSION_COMPAT}}}

        if [ -n "$FILE_VERSION" ]; then
            echo "Found Java version in $build_file: $FILE_VERSION"

            # Store the first version found if we don't have one yet
            if [ -z "$JAVA_VERSION_BUILD_GRADLE" ]; then
                JAVA_VERSION_BUILD_GRADLE="$FILE_VERSION"
            fi
        fi
    done <<< "$BUILD_FILES"
fi

# Check gradle.properties for Java version
if [ -f "gradle.properties" ]; then
    echo ""
    echo "Checking gradle.properties..."
    JAVA_VERSION_GRADLE_PROPS=$(grep -E "javaVersion|java\.version|sourceCompatibility|targetCompatibility" gradle.properties | grep -o "1[0-9]\+\|[0-9]\+" | head -1)

    if [ -n "$JAVA_VERSION_GRADLE_PROPS" ]; then
        echo "Found Java version in gradle.properties: $JAVA_VERSION_GRADLE_PROPS"
    fi
fi

# Determine the current Java version
CURRENT_JAVA_VERSION=${JAVA_VERSION_BUILD_GRADLE:-$JAVA_VERSION_GRADLE_PROPS}

echo ""
echo "Current Java version detected: ${CURRENT_JAVA_VERSION:-Not found}"
echo ""
```

#### 2.2 Validate Java 17 Requirement

Verify that the detected Java version is 17:

```bash
if [ -z "$CURRENT_JAVA_VERSION" ]; then
    echo "=========================================="
    echo "WARNING: Java Version Not Detected"
    echo "=========================================="
    echo ""
    echo "The Java version could not be identified in the Gradle configuration."
    echo "This may be due to:"
    echo "  - Complex variable resolution (functions, project properties)"
    echo "  - Non-standard Gradle configuration patterns"
    echo "  - Custom build script logic"
    echo ""
    echo "CONTINUING: The OpenRewrite plugin will handle the upgrade intelligently."
    echo "  - If Java is already at version 21, no changes will be made"
    echo "  - If Java is at version 17, it will be upgraded to version 21"
    echo ""
    echo "Proceeding with remaining upgrade steps..."
    echo ""
elif [ "$CURRENT_JAVA_VERSION" != "17" ]; then
    echo "=========================================="
    echo "WARNING: Java Version Mismatch"
    echo "=========================================="
    echo ""
    echo "These instructions are designed to upgrade Java 17 applications to Java 21."
    echo "Current Java version detected: $CURRENT_JAVA_VERSION"
    echo ""
    if [ "$CURRENT_JAVA_VERSION" = "21" ]; then
        echo "NOTE: Java 21 is already configured. The upgrade may not be necessary."
        echo "However, the OpenRewrite plugin will verify and apply any missing updates."
        echo ""
        echo "Proceeding with remaining steps..."
        echo ""
    else
        echo "CAUTION: The detected Java version is neither 17 nor 21."
        echo "  - If this version is incorrect, the upgrade will still proceed"
        echo "  - The OpenRewrite plugin will upgrade to Java 21 regardless"
        echo ""
        echo "Proceeding with remaining upgrade steps..."
        echo ""
    fi
else
    echo "✓ Java 17 detected - proceeding with upgrade to Java 21"
    echo ""
fi
```

This script:
- Searches all `build.gradle` and `build.gradle.kts` files for Java version patterns
- Handles multiple declaration styles: sourceCompatibility, targetCompatibility, JavaVersion enums, and toolchain configurations
- Resolves variable references from `gradle.properties` and build files
- Searches `gradle.properties` for Java version properties
- Extracts the Java version number
- Validates the detected version and provides appropriate warnings

**The script will CONTINUE execution in all cases:**
- If Java 17 is detected: Proceeds with the upgrade as expected
- If Java 21 is detected: Continues (OpenRewrite will skip unnecessary changes)
- If the version cannot be identified: Continues with a warning (OpenRewrite will handle the upgrade intelligently)
- If a different version is detected: Continues with a caution message (OpenRewrite will upgrade to Java 21)

---

### Step 3: Check and Install SDKMAN

SDKMAN (Software Development Kit Manager) is used to install and manage Java versions.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 3:**
Update the "Step 3: SDKMAN Installation" section in the log file with:
- Note if SDKMAN was already installed or newly installed
- If newly installed, log the SDKMAN version (output of `sdk version` command)
- Installation path
- Any installation issues or warnings

#### 3.1 Check if SDKMAN is Already Installed

Check if the SDKMAN initialization script exists:

```bash
[ -f ~/.sdkman/bin/sdkman-init.sh ] && echo "SDKMAN is installed" || echo "SDKMAN is not installed"
```

#### 3.2 Install SDKMAN (if not present)

If the file `~/.sdkman/bin/sdkman-init.sh` does not exist, install SDKMAN:

```bash
curl -s "https://get.sdkman.io" | bash
echo "SDKMAN installation complete"
```

This command downloads and executes the SDKMAN installation script, then confirms completion.

#### 3.3 Initialize SDKMAN in Current Shell

After confirming SDKMAN is installed (either already present or newly installed), initialize it in the current shell session:

```bash
source ~/.sdkman/bin/sdkman-init.sh
```

**Important:** This command adds the `sdk` function to your shell environment, which is the core command for managing Java installations.

#### 3.4 Verify SDKMAN Installation

Confirm SDKMAN is working correctly:

```bash
sdk version
```

This should display the SDKMAN version information.

---

### Step 4: Install Amazon Corretto Java 21

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 4:**
Update the "Step 4: Install Amazon Corretto Java 21" section in the log file with:
- Note if Amazon Corretto Java 21 was already installed or newly installed
- Log the Java version (output of `java -version` command)
- Installation path
- Any installation issues or warnings


#### 4.1 Check if Amazon Corretto Java 21 is Already Installed

First, check if Amazon Corretto Java 21 is already installed locally:

```bash
if sdk list java | grep -v "local only" | grep -q "21\..*amzn"; then
    echo "Amazon Corretto Java 21 is already installed"
    JAVA21_ALREADY_INSTALLED=true
else
    echo "Amazon Corretto Java 21 is not installed"
    JAVA21_ALREADY_INSTALLED=false
fi
```

This checks if any Amazon Corretto Java 21 version is installed locally by filtering out the "local only" header and searching for installed versions.

#### 4.2 Prepare Organization Trusted Certificates (if applicable)

**Before installing Java**, if your organization uses custom trusted certificates (e.g., for internal Certificate Authorities, SSL inspection, or corporate proxies), prepare them for import.

First, create the directory for trusted certificates:

```bash
mkdir -p ~/trusted-certs
```

Then, prompt the developer to prepare certificates:

```bash
echo ""
echo "=========================================="
echo "Organization Trusted Certificates Setup"
echo "=========================================="
echo ""
echo "If your organization uses custom trusted certificates, please copy them to:"
echo "  ~/trusted-certs/"
echo ""
echo "Supported certificate formats: .pem, .cer, .crt"
echo "Certificates with other extensions will be ignored during import."
echo ""
echo "If you don't have organization-specific certificates, you can skip this step."
echo ""
read -p "Have you copied all organization trusted certificates to ~/trusted-certs? (y/n): " cert_response
echo ""

if [[ "$cert_response" =~ ^[Yy]$ ]]; then
    echo "✓ Proceeding with certificate preparation acknowledged"
else
    echo "ℹ No certificates prepared - will check directory anyway"
fi

echo "Certificate preparation step complete"
echo ""
```

**Note:** The import process in section 2.4 will automatically detect and import any certificates present in `~/trusted-certs`, regardless of the response to this prompt.

#### 4.3 Find and Install Latest Amazon Corretto Java 21 (if not present)

If Amazon Corretto Java 21 is not installed, find and install the latest version:

```bash
if [ "$JAVA21_ALREADY_INSTALLED" = false ]; then
    # Find the latest Amazon Corretto Java 21 version
    LATEST_JAVA21=$(sdk list java | grep "21\..*amzn" | grep -v ">>>" | head -1 | awk '{print $NF}')

    if [ -z "$LATEST_JAVA21" ]; then
        echo "Error: No Amazon Corretto Java 21 versions found"
        exit 1
    fi

    echo "Installing Amazon Corretto Java 21: $LATEST_JAVA21"
    sdk install java "$LATEST_JAVA21"
    echo "Amazon Corretto Java 21 installation complete"
else
    echo "Skipping installation - Amazon Corretto Java 21 is already present"
fi
```

This script:
- Dynamically finds the latest Amazon Corretto Java 21 version available at the time of execution
- Installs it only if not already present
- Provides clear feedback about what action was taken

#### 4.4 Import Organization Trusted Certificates into Java 21 (if applicable)

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 4.4:**
Update the "Step 4.4: Import Organization Trusted Certificates" section in the log file with:
- Whether certificates were imported (yes/no)
- If skipped, reason for skipping (Java 21 already installed, no certificates found, etc.)
- Total number of certificates imported
- Names of each certificate imported (the alias names used)
- Success/failure status for each certificate import
- Any import errors or warnings encountered

**Only if Amazon Corretto Java 21 was freshly installed in section 3.3**, it is **CRITICAL** to import any organization trusted certificates into the Java keystore. Without these certificates, the application may fail to connect to internal services or repositories that use organization-specific SSL certificates:

```bash
# Only proceed if Java 21 was installed (not if installation was skipped)
if [ "$JAVA21_ALREADY_INSTALLED" = false ]; then
    # Get the installed Java 21 version and set JAVA_HOME
    JAVA21_VERSION=$(sdk list java | grep "21\..*amzn" | head -1 | awk '{print $NF}')

    if [ -n "$JAVA21_VERSION" ]; then
        export JAVA_HOME=$(sdk home java "$JAVA21_VERSION")
        echo "JAVA_HOME set to: $JAVA_HOME"

        # Check if trusted certs directory exists and contains certificates
        CERT_DIR="$HOME/trusted-certs"

        if [ -d "$CERT_DIR" ]; then
            # Count certificates with supported extensions
            CERT_COUNT=$(find "$CERT_DIR" -type f \( -name "*.pem" -o -name "*.cer" -o -name "*.crt" \) 2>/dev/null | wc -l | tr -d ' ')

            if [ "$CERT_COUNT" -gt 0 ]; then
                echo "Found $CERT_COUNT certificate(s) to import from ~/trusted-certs"

                # Get current date for alias suffix (YYYYMMDD format)
                CURRENT_DATE=$(date +%Y%m%d)

                # Import each certificate
                find "$CERT_DIR" -type f \( -name "*.pem" -o -name "*.cer" -o -name "*.crt" \) | while read -r cert_file; do
                    # Extract certificate filename without extension
                    cert_basename=$(basename "$cert_file")
                    cert_name="${cert_basename%.*}"

                    # Create alias with cert name and current date
                    alias_name="${cert_name}-${CURRENT_DATE}"

                    echo "Importing certificate: $cert_basename as alias: $alias_name"

                    # Import certificate into Java cacerts truststore
                    "$JAVA_HOME/bin/keytool" -import -alias "$alias_name" \
                        -cacerts -trustcacerts \
                        -file "$cert_file" \
                        -storepass changeit \
                        -noprompt

                    if [ $? -eq 0 ]; then
                        echo "✓ Successfully imported: $cert_basename"
                    else
                        echo "✗ Failed to import: $cert_basename"
                    fi
                done

                echo "Certificate import process complete"

                # List imported certificates for verification
                echo ""
                echo "Verifying imported certificates in Java truststore:"
                "$JAVA_HOME/bin/keytool" -list -cacerts -storepass changeit | grep -i "$CURRENT_DATE"

            else
                echo "No certificates found in ~/trusted-certs (looking for .pem, .cer, or .crt files)"
                echo "Skipping certificate import"
            fi
        else
            echo "Directory ~/trusted-certs does not exist"
            echo "Skipping certificate import"
        fi
    else
        echo "Error: Could not determine Java 21 version for certificate import"
    fi
else
    echo "Skipping certificate import - Java 21 was already installed (certificates may have been imported previously)"
fi
```

This script:
- Only attempts to import certificates if Java 21 was freshly installed (not if it was already present)
- Checks if the `~/trusted-certs` directory exists
- Only processes files with `.pem`, `.cer`, or `.crt` extensions
- Creates a unique alias for each certificate using the certificate name and current date
- Imports each certificate into the Java cacerts truststore with the default password `changeit`
- Uses `-noprompt` to avoid interactive confirmation
- Displays import success/failure for each certificate
- Lists the imported certificates for verification using the current date in the alias

**Important Notes:**
- If Java 21 was already installed before running these instructions, certificate import is skipped (assuming certificates were imported during the initial installation)
- The default cacerts password is `changeit` (this is the standard Java default)
- If you need to re-import certificates into an existing Java installation, you can manually run the import commands or temporarily set `JAVA21_ALREADY_INSTALLED=false`

#### 4.5 Set Java 21 as Default (Optional)

To make Java 21 the default version for all new shell sessions:

```bash
# Get the installed Java 21 version (either newly installed or already present)
JAVA21_VERSION=$(sdk list java | grep "21\..*amzn" | head -1 | awk '{print $NF}')

if [ -n "$JAVA21_VERSION" ]; then
    SDKMAN_AUTO_ANSWER=true sdk default java "$JAVA21_VERSION"
    echo "Set $JAVA21_VERSION as default Java version"
else
    echo "Error: Could not find Amazon Corretto Java 21 version"
fi
```

This dynamically identifies the installed Java 21 version and sets it as default.

#### 4.6 Verify Java 21 Installation

Confirm Java 21 is active:

```bash
java -version
```

Expected output should show:
```
openjdk version "21.0.x" 2024-xx-xx LTS
OpenJDK Runtime Environment Corretto-21.0.x.x.x
OpenJDK 64-Bit Server VM Corretto-21.0.x.x.x
```

---

### Step 5: Update Project Configuration

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 5:**
Update the "Step 5: Update Project Configuration" section in the log file with:
- Log the JAVA_HOME value after setting it (output of `echo $JAVA_HOME` command)

#### 5.1 Set JAVA_HOME Environment Variable

Ensure JAVA_HOME points to the correct Java 21 installation:

```bash
export JAVA_HOME=$(sdk home java 21.0.9-amzn)
echo $JAVA_HOME
```

---

### Step 6: Upgrade Gradle Wrapper (If Needed)

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 6:**
Update the "Step 6: Gradle Wrapper Update" section in the log file with:
- Note if Gradle Wrapper exists in the project or not
- If Gradle Wrapper exists, log the current Gradle version (from `gradle-wrapper.properties`)
- If upgraded, log the target Gradle version after the update
- Update method used (e.g., `./gradlew wrapper --gradle-version=X.X`)
- Verification results (Success/Failed)
- Any compatibility issues or warnings encountered

#### 6.1 Check if Gradle Wrapper Exists

First, verify that the project uses Gradle wrapper:

```bash
if [[ ! -f "gradlew" ]] || [[ ! -f "gradle/wrapper/gradle-wrapper.properties" ]]; then
    echo "=========================================="
    echo "Gradle wrapper not found"
    echo "=========================================="
    echo ""
    echo "This project does not use Gradle wrapper."
    echo "Skipping Gradle wrapper upgrade (Step 4)."
    echo ""
    GRADLE_WRAPPER_EXISTS=false
else
    echo "✓ Gradle wrapper found - proceeding with version check"
    echo ""
    GRADLE_WRAPPER_EXISTS=true
fi
```

**If Gradle wrapper is not found, skip the remaining subsections of Step 6 and proceed to Step 7.**

#### 6.2 Check Current Gradle Wrapper Version

**Only proceed if `GRADLE_WRAPPER_EXISTS=true`.**

Check the Gradle version specified in `gradle/wrapper/gradle-wrapper.properties`:

```bash
grep "distributionUrl" gradle/wrapper/gradle-wrapper.properties
```

This will display the current Gradle version being used by the wrapper.

#### 6.3 Verify Java 21 Compatibility

**Only proceed if `GRADLE_WRAPPER_EXISTS=true`.**

Gradle versions have specific Java compatibility requirements:
- **Gradle 8.14.4+**: Preferred for executing OpenRewrite recipes with the latest Gradle 8.x version.

To check your current Gradle version:

```bash
./gradlew --version
```

#### 6.4 Upgrade Gradle Wrapper (If Necessary)

**Only proceed if `GRADLE_WRAPPER_EXISTS=true`.**

**Check the Gradle version before upgrading:**
- If `GRADLE_VERSION` >= 8.14: **SKIP this upgrade step** 
- If `GRADLE_VERSION` <= 8.13: Proceed with upgrading to Gradle 8.14.4.

```bash
./gradlew wrapper --gradle-version=8.14.4
```

This command will update the Gradle wrapper files to use version 8.14.4.

#### 6.5 Verify Gradle Wrapper Upgrade

**Only proceed if `GRADLE_WRAPPER_EXISTS=true`.**

After upgrading, verify the new Gradle version:

```bash
./gradlew --version
```

Expected output should show Gradle 8.14.4 and Java 21:
```
------------------------------------------------------------
Gradle 8.14.4
------------------------------------------------------------

Build time:   2024-xx-xx xx:xx:xx UTC
Revision:     <revision-hash>

Kotlin:       1.9.x
Groovy:       3.0.x
Ant:          Apache Ant(TM) version 1.10.x compiled on <date>
JVM:          21.0.x (Amazon.com Inc. 21.0.x+xx-LTS)
OS:           Mac OS X 14.x.x aarch64
```

---

### Step 6a: Upgrade Known Java 21 Incompatible Libraries and Plugins

Certain libraries and plugins are known to require version upgrades or replacement for Java 21 compatibility. Proactively addressing these **before** running OpenRewrite and the build avoids unnecessary iteration cycles where the AI agent discovers and fixes these issues through trial and error.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 6a:**
Update the "Step 6a: Library Upgrades" section in the log file with:
- Libraries/plugins checked and their current versions
- Libraries upgraded and their new versions
- Plugins commented out and replacements added
- Libraries/plugins skipped (not present in project or already compatible)
- Any issues encountered during upgrade

> **Multi-Module Project Note:** Dependencies and plugins may be declared in root or submodule build files, or in version catalog files (`gradle/libs.versions.toml`). Refer to "Gradle Project Structure Patterns" to locate the correct file.

#### 6a.1 Check and Upgrade Lombok to Latest Version (If Present)

Lombok is a common library that generates boilerplate code at compile time. If present, upgrade to the latest version for fewer vulnerabilities and better stability (Lombok library latest versions are backwards compatible).

**🔴 Logging for this step:**
- If Lombok is not found: Log "Lombok not present - skipped"
- If Lombok is found and upgraded: Log "Lombok upgraded from [previous_version] to latest version [new_version]"

**Check if Lombok is present:**

```bash
# Search for Lombok in build files
grep -r "lombok" --include="build.gradle*" --include="*.toml" .

# Check gradle.properties for Lombok version variables
grep -i "lombok" gradle.properties 2>/dev/null
```

**If Lombok is found, check the version:**

```bash
# Extract Lombok version from build.gradle
grep -E "lombok.*[0-9]+\.[0-9]+\.[0-9]+" build.gradle

# Or from gradle.properties (any variable containing "lombok")
grep -i "lombok" gradle.properties 2>/dev/null

# Or from version catalog
grep -A1 "lombok" gradle/libs.versions.toml 2>/dev/null
```

<!-- for Java 21 the lombok version 1.18.30 is sufficient. However, there is
no harm in using the latest lombok release version.
As of writing the article, the latest version is 1.18.42 -->
**Upgrade Lombok to the latest version (minimum 1.18.42 required):**

First, find the latest Lombok version from Maven Central:
```bash
# Search for latest Lombok release version on Maven Central
curl -s "https://search.maven.org/solrsearch/select?q=g:org.projectlombok+AND+a:lombok&rows=1&wt=json" | grep -o '"latestVersion":"[^"]*"' | cut -d'"' -f4
```

Alternatively, check the Lombok releases page: https://projectlombok.org/changelog

Use the latest version retrieved above. If the Maven Central query fails, use the fallback version specified in the section heading:

For `build.gradle` (Groovy DSL):
```groovy
// Before
compileOnly 'org.projectlombok:lombok:1.18.30'
annotationProcessor 'org.projectlombok:lombok:1.18.30'

// After - use latest version (1.18.42 or higher)
compileOnly 'org.projectlombok:lombok:<LATEST_VERSION>'
annotationProcessor 'org.projectlombok:lombok:<LATEST_VERSION>'
```

For `build.gradle.kts` (Kotlin DSL):
```kotlin
// Before
compileOnly("org.projectlombok:lombok:1.18.30")
annotationProcessor("org.projectlombok:lombok:1.18.30")

// After - use latest version (1.18.42 or higher)
compileOnly("org.projectlombok:lombok:<LATEST_VERSION>")
annotationProcessor("org.projectlombok:lombok:<LATEST_VERSION>")
```

For `gradle/libs.versions.toml` (Version Catalog):
```toml
[versions]
# Before
lombok = "1.18.30"

# After - use latest version (1.18.42 or higher)
lombok = "<LATEST_VERSION>"
```

#### 6a.2 Check and Upgrade MapStruct to Latest Version of 1.x (If Present)

MapStruct is an annotation processor for generating type-safe bean mappers. If present, upgrade to the latest version for fewer vulnerabilities and better stability (MapStruct library latest versions are backwards compatible).

**🔴 Logging for this step:**
- If MapStruct is not found: Log "MapStruct not present - skipped"
- If MapStruct is found and upgraded: Log "MapStruct upgraded from [previous_version] to latest version of 1.x [new_version]"

**Check if MapStruct is present:**

```bash
# Search for MapStruct in build files
grep -r "mapstruct" --include="build.gradle*" --include="*.toml" .

# Check gradle.properties for MapStruct version variables
grep -i "mapstruct" gradle.properties 2>/dev/null
```

**If MapStruct is found, check the version:**

```bash
# Extract MapStruct version from build.gradle
grep -E "mapstruct.*[0-9]+\.[0-9]+\.[0-9]+" build.gradle

# Or from gradle.properties (any variable containing "mapstruct")
grep -i "mapstruct" gradle.properties 2>/dev/null

# Or from version catalog
grep -A1 "mapstruct" gradle/libs.versions.toml 2>/dev/null
```

**Upgrade MapStruct to the latest version of 1.x (minimum 1.6.3 required):**

First, find the latest MapStruct 1.x version from Maven Central:
```bash
# Search for latest MapStruct version on Maven Central
curl -s "https://search.maven.org/solrsearch/select?q=g:org.mapstruct+AND+a:mapstruct&rows=1&wt=json" | grep -o '"latestVersion":"[^"]*"' | cut -d'"' -f4
```

Alternatively, check the MapStruct releases page: https://mapstruct.org/news/

**Important:** If the returned version is 2.x or higher, do NOT use it. Use the latest 1.x version instead (fallback: 1.6.3). Major version upgrades may introduce breaking changes.

Use the latest 1.x version retrieved above. If the Maven Central query fails or returns a 2.x version, use the fallback version 1.6.3:

For `build.gradle` (Groovy DSL):
```groovy
// Before
implementation 'org.mapstruct:mapstruct:1.5.3.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'

// After - use latest version (1.5.5 or higher)
implementation 'org.mapstruct:mapstruct:<LATEST_VERSION>'
annotationProcessor 'org.mapstruct:mapstruct-processor:<LATEST_VERSION>'
```

For `build.gradle.kts` (Kotlin DSL):
```kotlin
// Before
implementation("org.mapstruct:mapstruct:1.5.3.Final")
kapt("org.mapstruct:mapstruct-processor:1.5.3.Final")

// After - use latest version (1.6.3 or higher)
implementation("org.mapstruct:mapstruct:<LATEST_VERSION>")
kapt("org.mapstruct:mapstruct-processor:<LATEST_VERSION>")
```

For `gradle/libs.versions.toml` (Version Catalog):
```toml
[versions]
# Before
mapstruct = "1.5.3.Final"

# After - use latest version (1.6.3 or higher)
mapstruct = "<LATEST_VERSION>"
```

#### 6a.3 Replace google-java-format Plugin with Spotless (If Present)

The `google-java-format` plugin does not work with Java 21. If this plugin is present, comment it out and add the Spotless plugin as a replacement.

**Logging for this step:**
- If google-java-format is not found: Log "google-java-format not present - skipped"
- If google-java-format is found and replaced: Log "google-java-format plugin commented out, Spotless plugin added with version [spotless_version]"

**Check if google-java-format plugin is present:**

```bash
# Search for google-java-format in build files
grep -r "google-java-format\|googleJavaFormat" --include="build.gradle*" .
```

**If google-java-format is found, perform the following steps:**

##### Action A: Comment out the google-java-format plugin

For `build.gradle` (Groovy DSL):
```groovy
plugins {
    // ... other plugins ...
    // Commented out due to Java 21 incompatibility - replaced with Spotless
    // id 'com.github.sherter.google-java-format' version 'x.x.x'
}
```

Also comment out any related configuration blocks:
```groovy
// Commented out due to Java 21 incompatibility - replaced with Spotless
// googleJavaFormat {
//     toolVersion = 'x.x.x'
//     options style: 'GOOGLE'
// }
```

For `build.gradle.kts` (Kotlin DSL):
```kotlin
plugins {
    // ... other plugins ...
    // Commented out due to Java 21 incompatibility - replaced with Spotless
    // id("com.github.sherter.google-java-format") version "x.x.x"
}
```

##### Action B: Add the Spotless plugin

Add the Spotless plugin to the `plugins` block.

First, find the latest Spotless plugin version from the Gradle Plugin Portal: https://plugins.gradle.org/plugin/com.diffplug.spotless

**Note:** The Gradle Plugin Portal is the authoritative source for Gradle plugin versions. Maven Central may show older versions as Gradle plugins are published separately to the plugin portal.

Use the latest version from the Gradle Plugin Portal. If unable to check, use version 8.2.0 or higher:

For `build.gradle` (Groovy DSL):
```groovy
plugins {
    // ... other plugins ...
    id 'com.diffplug.spotless' version '<LATEST_VERSION>'
}
```

For `build.gradle.kts` (Kotlin DSL):
```kotlin
plugins {
    // ... other plugins ...
    id("com.diffplug.spotless") version "<LATEST_VERSION>"
}
```

##### Action C: Configure Spotless to use Google Java Format

Add the Spotless configuration block.

For `build.gradle` (Groovy DSL):
```groovy
spotless {
    java {
        target 'src/*/java/**/*.java'
        googleJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
    }
}
```

For `build.gradle.kts` (Kotlin DSL):
```kotlin
spotless {
    java {
        target("src/*/java/**/*.java")
        googleJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
    }
}
```

**Note:** Spotless uses its bundled google-java-format version by default, which is compatible with Java 21. No explicit version is needed.

##### Action D: Update any task dependencies (if applicable)

If the build file has tasks that depended on google-java-format tasks, update them to use Spotless equivalents:

| Old Task (google-java-format) | New Task (Spotless) |
|------------------------------|---------------------|
| `googleJavaFormat` | `spotlessApply` |
| `verifyGoogleJavaFormat` | `spotlessCheck` |

#### 6a.4 Verify Library and Plugin Upgrades

After upgrading libraries and plugins, verify the changes compile successfully:

```bash
./gradlew compileJava --dry-run
```

This validates that the upgraded dependencies can be resolved without running the full build.

If Spotless was added, also verify the plugin is configured correctly:

```bash
./gradlew spotlessCheck --dry-run
```

---

### Step 7: Use OpenRewrite to Migrate Java Code

OpenRewrite is an automated refactoring tool that can help migrate Java code from Java 17 to Java 21.

**Before proceeding with Step 7:** Review the "Gradle Project Structure Patterns" section to understand where build files may be located in single-module vs. multi-module projects.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 7:**
Update the "Step 7: OpenRewrite Migration" section and build/fix sections in the log file with:
- OpenRewrite recipes executed (list each recipe name)
- Number of files modified by OpenRewrite
- Summary of key automated changes made
- Compilation errors encountered (document each in "Fixes Applied" or "Unresolved Errors" sections)
- Test failures encountered (document each appropriately)
- Fixes applied using Error Resolution Methodology (detailed documentation in "Fixes Applied" section)
- Unresolved issues (detailed documentation in "Unresolved Errors" section)
- Build/test iteration counts
- Final status (Success, Completed with warnings, or Needs Manual Intervention)

#### 7.1 Check if OpenRewrite Plugin is Present

First, check if the OpenRewrite plugin is already configured in your build file(s).

> **Multi-Module Project Note:** The plugin may be in the root `build.gradle`/`build.gradle.kts` or in submodule build files. Check all build files if needed. See "Gradle Project Structure Patterns" for details.

Check for the plugin:
```bash
# Check root build file
grep -q "org.openrewrite.rewrite" build.gradle && echo "OpenRewrite plugin found in root" || echo "OpenRewrite plugin not found in root"

# For multi-module projects, check all build files
find . -type f \( -name "build.gradle" -o -name "build.gradle.kts" \) -exec grep -l "org.openrewrite.rewrite" {} \;
```

#### 7.2 Add OpenRewrite Plugin (if not present or upgrade needed)

> **Multi-Module Project Note:** Add the plugin to the build file that contains other plugins. This may be the root build file or a submodule build file. See "Gradle Project Structure Patterns" for guidance.

If the OpenRewrite plugin is not present, or if a newer version is required for Java 17 to 21 migration, add or update it in the `plugins` section of your build file:

```groovy
plugins {
  // ... existing plugins ...
  id("org.openrewrite.rewrite") version "7.25.0"
}
```

**Note:** Use the latest version of the OpenRewrite Gradle plugin. Version 7.25.0 has known compatibility issues with Java 21 migration recipes.

#### 7.3 Add Rewrite Dependencies (if not present)

> **Multi-Module Project Note:** Add dependencies to the build file that already contains other dependencies. Refer to "Gradle Project Structure Patterns" for guidance on locating the correct build file.

Check if the OpenRewrite dependencies are already present in your build file. If they are not present, add them to the `dependencies` section:

```groovy
dependencies {
    // ... existing dependencies ...

    // Import the BOM to manage versions automatically
    rewrite(platform("org.openrewrite.recipe:rewrite-recipe-bom:3.23.0"))

    // Add the specific migration artifact without a version
    rewrite("org.openrewrite.recipe:rewrite-migrate-java")
}
```

The `rewrite-migrate-java` recipe provides automated refactoring rules for Java version migrations.

**Note:** If these dependencies already exist in the build file, skip this step and proceed to the next section.

#### 7.4 Run Rewrite Migration

Execute the OpenRewrite migration to automatically refactor code for Java 21 compatibility. Pass the recipes directly as command line parameters:

```bash
./gradlew rewriteRun \
  -Drewrite.activeRecipes=org.openrewrite.java.migrate.UpgradeToJava21,\
org.openrewrite.java.migrate.SwitchPatternMatching
```

**Recipe Descriptions:**
- `UpgradeToJava21` - Core Java 21 upgrade recipe that updates deprecated APIs and patterns
- `SwitchPatternMatching` - Applies pattern matching in switch statements (Java 21 feature)

**Why pass recipes via command line?**
- Upgrade recipes are one-time operations and shouldn't be permanently added to [build.gradle](build.gradle)
- Prevents accidental re-execution of upgrade recipes in future builds
- Keeps the build configuration clean and focused on ongoing build tasks

This command will:
- Analyze the codebase for Java 17 to Java 21 migration opportunities
- Apply automated refactoring rules for the specified recipes
- Update deprecated APIs and patterns
- Use a few simple and safe Java 21 features
- Modify source files

#### 7.5 Review Changes

After running the migration, review the changes made by OpenRewrite:

```bash
git diff
```

OpenRewrite makes changes directly to source files, so use git to review what was modified. Common changes include:
- Updated API usage for Java 21
- Removed deprecated method calls
- Updated language constructs to use Java 21 features
- Fixed compatibility issues

#### 7.6 Verify Migration Success

After reviewing the changes, compile the project to ensure the migration was successful:

```bash
./gradlew clean build
```

If there are any compilation errors, address them before proceeding.

#### 7.7 Build/Fix Loop for Compilation Errors

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 7.7:**
Update the "Step 7.7: Build/Fix Loop" section in the log file with:

**Fixes Applied in Build Fix Loop:**

For each fix applied, log the following:

**Fix #1**: [Brief description]
- **Iteration**: X (of max 10)
- **File**: path/to/file.java:line_number
- **Error Type**: [Compilation Error | Test Failure | Deprecated API | etc.]
- **Root Cause**: Description of the issue
- **Solution Applied**: Description of the fix
- **Source**: [URL or reference to documentation] (if applicable)

**Fix #2**: [Brief description]
- ...

**Unresolved Errors (if any):**

For each unresolved error, log the following:

**Error #1**: [Brief description]
- **File**: path/to/file.java:line_number
- **Error Message**: Complete error message
- **Error Category**: [API Deprecation | Package Migration | etc.]
- **Attempted Solutions**:
  1. OpenRewrite recipe attempted: recipe-name (Result: Failed)
  2. Internet search queries: "query text"
  3. Manual fix attempted: Description (Result: Failed because...)
- **References**: Links to documentation, Stack Overflow, etc. (if applicable)
- **Recommended Next Steps**: Suggestions for manual resolution

**Error #2**: [Brief description]
- ...

**Build Fix Loop Summary:**
- Total iterations completed: X
- Total fixes applied: Y
- Unresolved errors: Z
- Final build status: Success/Failed

---

After running the OpenRewrite migration and reviewing changes, iteratively fix any remaining compilation errors:

1. Execute the build command (skip tests for now):
   ```bash
   ./gradlew clean build -x test
   ```
2. If there are compilation errors:
  - Analyze each error message carefully
  - Fix the errors using the **Error Resolution Methodology** (see section 7.7.1 below)
  - Common issues to address:
    - Deprecated APIs not handled by OpenRewrite
    - Changed method signatures in Java 21
    - New restrictions or requirements in Java 21
    - Third-party library compatibility issues
3. Re-run the build after making fixes
4. **Repeat steps 2-3 with the following exit conditions:**
   - **Success condition:** Build succeeds without compilation errors
   - **Maximum iterations:** Up to 5 build/fix cycles
   - **Stalled progress detection:** If the same error persists for 3 consecutive iterations, stop and document the issue
   - **Failure exit:** If unable to resolve after maximum iterations, document all remaining errors and request human intervention
5. Once compilation succeeds, run the test suite:
   ```bash
   ./gradlew test
   ```
6. If tests fail:
  - Analyze test failure messages carefully
  - Fix test failures using the **Error Resolution Methodology** (see section 7.7.1 below)
  - Common test issues:
    - Behavior changes in Java 21 APIs
    - Timing or ordering differences
    - Mock/stub compatibility with new APIs
    - Test assertions that rely on implementation details that changed in Java 21
7. **Repeat steps 5-6 with the following exit conditions:**
   - **Success condition:** All tests pass
   - **Maximum iterations:** Up to 5 test/fix cycles
   - **Stalled progress detection:** If the same test failure persists for 3 consecutive iterations, stop and document the issue
   - **Failure exit:** If unable to resolve after maximum iterations, document all remaining test failures and request human intervention

**Important:** If either loop exits due to reaching maximum iterations or stalled progress:
- Create a detailed report listing all unresolved errors/failures
- Document any attempted fixes that didn't work
- Include relevant error messages and stack traces
- Stop execution and notify the user for manual review and intervention

##### 7.7.1 Error Resolution Methodology

When compilation errors or test failures occur, follow this systematic approach to resolve them:

###### Error Resolution Step 1: Extract and Categorize the Error

1. **Capture the complete error message** including:
   - File path and line number
   - Error type (compilation error, deprecation warning, etc.)
   - Full error description and stack trace
   - Any suggested fixes from the compiler

2. **Categorize the error type:**
   - **API Deprecation/Removal**: Method or class no longer available in Java 21
   - **API Signature Change**: Method parameters or return types changed
   - **Package Migration**: Classes moved to different packages (e.g., javax.* → jakarta.*)
   - **Access Modifier Restriction**: New access restrictions in Java 21
   - **Third-party Library Incompatibility**: External dependencies not compatible with Java 21
   - **Behavioral Change**: API behavior changed in Java 21
   - **Other**: Uncategorized errors

**Log the error information:** Keep a record in the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) of the extracted error details (file path, line number, error type, category, and full error message). This information will be used to document the error in either the "Fixes Applied" section (if resolved) or the "Unresolved Errors" section (if it cannot be fixed).

###### Error Resolution Step 2: Search for OpenRewrite Recipes

**Before making manual code changes**, search for existing OpenRewrite recipes that can automatically fix the error:

1. **Search the OpenRewrite recipe catalog:**
   ```bash
   # List all available recipes in the rewrite-migrate-java dependency
   ./gradlew rewriteDiscover
   ```

2. **Search for recipes by keyword** related to the error:
   - Visit the OpenRewrite recipe catalog: https://docs.openrewrite.org/recipes
   - Search for recipes related to the error (e.g., "deprecated", "remove", "migrate", specific API names)
   - Look specifically in the "Java version migration" section: https://docs.openrewrite.org/recipes/java/migrate

3. **Search for additional OpenRewrite recipes:**

   **Note:** The `UpgradeToJava21` recipe executed in section 6.4 is a comprehensive recipe that includes many common migration sub-recipes. However, if you encounter compilation errors, there may be additional specialized recipes that weren't included or that need to be run independently.

   **How to search for recipes:**
   - Visit the OpenRewrite recipe catalog: https://docs.openrewrite.org/recipes/java/migrate/upgradetojava21
   - Look for recipes that specifically address the error you're seeing
   - Check if the recipe is already included in `UpgradeToJava21` by reviewing the recipe composition page

   **Examples of specialized recipes that might help:**
   - `org.openrewrite.java.migrate.RemovedModifierRestrictions` - Fix removed modifier restrictions
   - `org.openrewrite.java.migrate.DeprecatedAPIs` - Replace deprecated APIs
   - `org.openrewrite.java.migrate.RemovedJavaSecurityManagerAPIs` - Remove SecurityManager usage
   - `org.openrewrite.java.migrate.RemovedThreadMethods` - Update removed Thread methods
   - `org.openrewrite.java.migrate.jakarta.*` - Migrate javax to jakarta packages (often needed for Spring/Jakarta EE projects)

   **When to run additional recipes:**
   - Only if the error is clearly related to a specific migration pattern
   - If the recipe is NOT already part of `UpgradeToJava21` (check the recipe documentation)
   - For specialized migrations like javax → jakarta that may not be in the core upgrade recipe

4. **If a relevant recipe is found:**
   - Run ONLY the new recipe to fix the specific error (do not re-run the recipes that were already executed in section 6.4):
     ```bash
     ./gradlew rewriteRun \
       -Drewrite.activeRecipes=org.openrewrite.java.migrate.<NewRecipeForTheError>
     ```
     Replace `<NewRecipeForTheError>` with the actual recipe name that addresses the error.

   **Why run only the new recipe?**
   - The initial migration recipes (`UpgradeToJava21`, `PatternMatchingInstanceof`, `SwitchExpressions`, `SwitchPatternMatching`) have already been executed in section 6.4
   - Re-running them is unnecessary and wasteful - they are idempotent but will re-scan the entire codebase
   - Running only the new recipe is faster and makes it easier to see what changed in git diff
   - OpenRewrite recipes are designed to be independent and can be run individually

   - Re-run the build to verify the fix:
     ```bash
     ./gradlew clean build
     ```
   - If the error is resolved, continue to the next error (if any)
   - If the error persists, proceed to Step 3

**Track recipe attempts:** Keep note of which OpenRewrite recipes you searched for and tried in the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) - you'll need this information for either:
- The "Fixes Applied" section if the recipe succeeds (document which recipe resolved the error)
- The "Unresolved Errors" section under "Attempted Solutions" if the recipe fails

###### Error Resolution Step 3: Search the Internet for Solutions

**If no OpenRewrite recipe is found or the recipe didn't resolve the error**, search the Internet for solutions:

1. **Construct an effective search query** using:
   - The exact error message (in quotes)
   - "Java 21" or "Java 17 to 21 migration"
   - The affected API, class, or method name
   - Example: `"java.lang.SecurityManager deprecated" Java 21 migration`

2. **Search authoritative sources in this order:**
   - **Official Java documentation:**
     - Java 21 Release Notes: https://www.oracle.com/java/technologies/javase/21-relnotes.html
     - Java 21 API Documentation: https://docs.oracle.com/en/java/javase/21/docs/api/
     - JEP (JDK Enhancement Proposals) related to Java 21
   - **Stack Overflow** with filters:
     - Search with tags: [java-21], [java], [migration]
     - Look for answers with high votes and recent dates (2023+)
   - **GitHub Issues and Discussions:**
     - Search in repositories of affected third-party libraries
     - Look for migration guides and compatibility issues
   - **Technical blogs and migration guides:**
     - Baeldung, DZone, InfoQ articles on Java 21 migration
     - Spring Blog (if using Spring Framework)

3. **Evaluate search results:**
   - Prioritize official documentation and well-established sources
   - Look for solutions that explain the root cause, not just workarounds
   - Verify the solution applies to your specific Java version (17 → 21)
   - Check if the solution has been tested and validated by others

4. **Document the source** of the solution for future reference in the upgrade log file (see Documentation section below)

**Track your research:** Keep a record in the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) of:
- Search queries you used (you'll need these for "Attempted Solutions" if the fix fails)
- Sources found and URLs (you'll need these for the "Source" field in "Fixes Applied" if the fix succeeds, or "References" in "Unresolved Errors" if it fails)
- Potential solutions identified (helps document what was attempted)

###### Error Resolution Step 4: Apply Automated Code Fixes

**If Internet research provides a solution**, automatically apply the fix and document the changes in the upgrade log file:

1. **Common fix patterns for Java 21 migration:**

   **A. Deprecated API Replacement:**
   ```java
   // Before (Java 17)
   Integer.parseInt("123", 10);

   // After (Java 21) - if the API was removed or changed
   // Use the recommended replacement from documentation
   ```

   **B. Package Migration (javax → jakarta):**
   ```java
   // Before
   import javax.servlet.http.HttpServlet;

   // After
   import jakarta.servlet.http.HttpServlet;
   ```

   **C. SecurityManager Removal:**
   ```java
   // Before
   SecurityManager sm = System.getSecurityManager();

   // After
   // Remove SecurityManager usage or use alternative security mechanisms
   ```

   **D. Thread Method Changes:**
   ```java
   // Before
   Thread.stop();  // Removed in Java 21

   // After
   // Use interrupt() and proper thread coordination
   thread.interrupt();
   ```

   **E. Third-party Library Updates:**

   > **Multi-Module Project Note:** Dependencies may be in root or submodule build files, or in version catalog files. Refer to "Gradle Project Structure Patterns" to locate the correct file.

   - Check if a newer version of the library supports Java 21
   - Update the dependency version in your dependency configuration file:
     - [build.gradle](build.gradle) or [build.gradle.kts](build.gradle.kts) for traditional dependency declarations (check root and submodule directories)
     - [gradle/libs.versions.toml](gradle/libs.versions.toml) if using Gradle version catalogs

     Example for build.gradle:
     ```groovy
     dependencies {
       implementation 'group:artifact:new-version'  // Updated version
     }
     ```

     Example for gradle/libs.versions.toml:
     ```toml
     [versions]
     library = "new-version"

     [libraries]
     library-name = { group = "group", name = "artifact", version.ref = "library" }
     ```

2. **Make targeted, minimal changes:**
   - Only modify the code necessary to fix the error
   - Avoid refactoring or restructuring beyond what's needed
   - Preserve existing functionality and behavior

3. **Test the fix:**
   - Re-run the build after each fix
   - Verify the specific error is resolved
   - Ensure no new errors are introduced

**If the fix succeeds:** Immediately document it in the "Fixes Applied" section of the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) following the format specified in section 6.7.1. Include:
- File path and line number
- Error type and root cause
- Solution applied (the specific code changes made)
- Source (URL or reference from Step 3)
- Date

This documentation should be done immediately while the details are fresh.

###### Error Resolution Step 5: Document Unresolvable Errors

**If the error cannot be resolved** after trying all strategies:

1. **Create a detailed error report** in the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) including:
   - Complete error message and stack trace
   - File path and line number
   - Error category (from Step 1)
   - OpenRewrite recipes attempted (if any)
   - Internet search queries used
   - Solutions attempted and why they didn't work
   - Links to relevant documentation or Stack Overflow posts
   - Recommended next steps or alternatives

2. **Add the error to the unresolved errors section** in the upgrade log file

3. **Continue to the next error** (if within iteration limits) or exit the loop

###### Error Resolution Step 6: Detect Stalled Progress and Prevent Infinite Loops

**To implement stalled progress detection:**

1. **Maintain a record** in the upgrade log file (`/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`) of errors encountered in each iteration:
   - Error signature (file path + line number + error message)
   - Iteration number when the error was first seen
   - Resolution attempts made

2. **Compare errors across iterations:**
   - If the exact same error appears in 3 consecutive iterations, it's considered "stalled"
   - Stop attempting to fix that error and add it to the unresolvable list

3. **Count successful fixes:**
   - If at least one error is fixed in an iteration, continue to the next iteration
   - If no progress is made (same number of errors or same errors), increment the stall counter

###### Example Workflow

**Example: Fixing a deprecated API error**

1. **Error encountered:**
   ```
   error: cannot find symbol
   symbol:   method getSubject()
   location: class javax.security.auth.Subject
   ```

2. **Categorize:** API Deprecation/Removal

3. **Search OpenRewrite recipes:**
   - Find recipe: `org.openrewrite.java.migrate.RemovedJavaSecurityManagerAPIs`
   - Add recipe to build.gradle
   - Run `./gradlew rewriteRun`
   - Result: Error persists (recipe doesn't cover this specific case)

4. **Search Internet:**
   - Query: `"Subject.getSubject() deprecated" Java 21 migration`
   - Find: Official Java 21 release notes indicating `Subject.getSubject()` was removed
   - Solution: Use `Subject.current()` instead

5. **Apply fix:**
   ```java
   // Before
   Subject subject = Subject.getSubject(AccessController.getContext());

   // After
   Subject subject = Subject.current();
   ```

6. **Verify:**
   - Run `./gradlew clean build`
   - Error resolved ✓

---

### Step 8: Update Dockerfile (If Present)

If the repository contains a Dockerfile with a Java 17 base image, it needs to be updated to use Java 21.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 8:**
Create a new "Step 8: Dockerfile Updates" section in the log file (or note "Not Applicable" if no Dockerfiles exist) with:
- Whether Dockerfile(s) exist in the repository (Yes/No)
- Number of Dockerfiles found and their paths
- Java version detected in each Dockerfile (e.g., Java 17, Java 11, or "No Java version found")
- Changes made to each Dockerfile (list specific updates like base image changes, VARIANT updates)
- Verification results for each Dockerfile
- Docker build test results (if performed - Success/Failed/Skipped)

#### 8.1 Check for Dockerfile

Search for Dockerfile(s) in the repository:

```bash
find . -name "Dockerfile*" -type f
```

#### 8.2 Identify Java Version in Dockerfile

For each Dockerfile found, check if it references Java 17:

```bash
grep -i "java.*17\|17.*java\|VARIANT=17\|JAVA_VERSION=17" <path-to-dockerfile>
```

#### 8.3 Update Dockerfile to Java 21

If Java 17 is found in the Dockerfile, update it to Java 21. Common patterns to look for and update:

##### Pattern 1: VARIANT argument
```dockerfile
# Before
ARG VARIANT=17-bullseye

# After
ARG VARIANT=21-bullseye
```

##### Pattern 2: JAVA_VERSION argument
```dockerfile
# Before
ARG JAVA_VERSION=17.0.7-ms

# After
ARG JAVA_VERSION=21.0.9-amzn
```

**Note:** Update the specific Java version to the latest available Java 21 version. For Amazon Corretto, use a version like `21.0.9-amzn` or later.

##### Pattern 3: Base image with explicit Java version
```dockerfile
# Before
FROM [<registry>/]amazoncorretto:17-alpine

# After
FROM [<registry>/]amazoncorretto:21-alpine
```

**Note:** The `[<registry>/]` prefix is optional and represents container registry paths like `ghcr.io/`, `docker.io/`, etc. If present, preserve the registry prefix and only update the Java version number from 17 to 21.

##### Pattern 4: SDKMAN installation in Dockerfile
```dockerfile
# Before
RUN bash -lc '. /usr/local/sdkman/bin/sdkman-init.sh && sdk install java 17.0.7-ms && sdk use java 17.0.7-ms'

# After
RUN bash -lc '. /usr/local/sdkman/bin/sdkman-init.sh && sdk install java 21.0.9-amzn && sdk use java 21.0.9-amzn'
```

#### 8.4 Verify Dockerfile Changes

After updating the Dockerfile, verify the changes:

```bash
grep -i "java.*21\|21.*java\|VARIANT=21\|JAVA_VERSION=21" <path-to-dockerfile>
```

This should confirm that all Java 17 references have been updated to Java 21.

#### 8.5 Test Docker Build (Optional)

If Docker is available, test building the image to ensure the Dockerfile changes are valid:

```bash
docker build -f <path-to-dockerfile> -t test-java21-upgrade .
```

This verifies that the Dockerfile syntax is correct and the Java 21 base image is accessible.

---

### Step 9: Update GitHub Actions Workflow Files (If Present)

GitHub Actions workflow files may specify Java versions for CI/CD builds. If any workflow files reference Java 17, they need to be updated to Java 21.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 8:**
Create a new "Step 8: GitHub Actions Workflow Updates" section in the log file (or note "Not Applicable" if no workflows exist) with:
- Whether GitHub Actions workflows exist in the repository (Yes/No)
- Number of workflow files found and their paths
- Java version references found in each workflow file
- Changes made to each workflow file (list specific updates like java-version, distribution changes)
- Verification results for each workflow file

#### 9.1 Check for GitHub Actions Workflow Files

Search for workflow files in the repository:

```bash
find .github/workflows -name "*.yml" -o -name "*.yaml" 2>/dev/null
```

#### 9.2 Identify Java 17 References in Workflow Files

Check all workflow files for Java 17 references:

```bash
grep -r -i "java.*17\|17.*java\|java-version.*17" .github/workflows/ 2>/dev/null
```

#### 9.3 Update Workflow Files to Java 21

If Java 17 is found in workflow files, update it to Java 21. Common patterns to look for and update:

##### Pattern 1: Matrix strategy with Java version
```yaml
# Before
strategy:
  matrix:
    java: [ '17' ]

# After
strategy:
  matrix:
    java: [ '21' ]
```

##### Pattern 2: Setup Java action with java-version
```yaml
# Before
- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'corretto'

# After
- name: Set up JDK 21
  uses: actions/setup-java@v3
  with:
    java-version: '21'
    distribution: 'corretto'
```

##### Pattern 3: Multiple Java versions in matrix (keeping Java 17 for compatibility testing)
```yaml
# If testing against multiple Java versions and you want to keep Java 17 for compatibility:
strategy:
  matrix:
    java: [ '17', '21' ]

# If upgrading completely to Java 21 only:
strategy:
  matrix:
    java: [ '21' ]
```

**Note:** Ensure the `distribution` field is set to `'corretto'` to use Amazon Corretto JDK, consistent with the local development environment.

#### 9.4 Verify Workflow File Changes

After updating the workflow files, verify the changes:

```bash
grep -r -i "java.*21\|21.*java\|java-version.*21" .github/workflows/ 2>/dev/null
```

This should confirm that Java version references have been updated to 21.

#### 9.5 Validate Workflow Syntax (Optional)

If the GitHub CLI (`gh`) is installed, you can validate the workflow syntax:

```bash
gh workflow list
```

Or commit the changes and check the Actions tab on GitHub to ensure workflows run successfully with Java 21.

---

#### Step 10: Update AWS CodeBuild buildspec.yml Files (If Present)

AWS CodeBuild buildspec.yml files may specify Java runtime versions. If any buildspec files reference Java 17, they need to be updated to Java 21.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 10:**
Create a new "Step 10: AWS CodeBuild buildspec Updates" section in the log file (or note "Not Applicable" if no buildspec files exist) with:
- Whether buildspec files exist in the repository (Yes/No)
- Number of buildspec files found and their paths
- Java version references found in each buildspec file
- Changes made to each buildspec file (list specific runtime updates)
- Verification results for each buildspec file

#### 10.1 Check for AWS CodeBuild buildspec Files

Search for buildspec files in the repository:

```bash
find . -name "buildspec*.yml" -o -name "buildspec*.yaml" 2>/dev/null
```

#### 10.2 Identify Java 17 References in buildspec Files

Check all buildspec files for Java 17 references:

```bash
grep -r -i "java.*17\|corretto17\|runtime.*17" buildspec*.yml buildspec*.yaml 2>/dev/null
```

#### 10.3 Update buildspec Files to Java 21

If Java 17 is found in buildspec files, update it to Java 21. Common patterns to look for and update:

##### Pattern 1: Runtime version in install phase
```yaml
# Before
phases:
  install:
    runtime-versions:
      java: corretto17

# After
phases:
  install:
    runtime-versions:
      java: corretto21
```

##### Pattern 2: Explicit Java version specification
```yaml
# Before
phases:
  install:
    runtime-versions:
      java: 17

# After
phases:
  install:
    runtime-versions:
      java: 21
```

##### Pattern 3: Environment variables for Java version
```yaml
# Before
env:
  variables:
    JAVA_VERSION: "17"
    JAVA_HOME: "/usr/lib/jvm/java-17-amazon-corretto"

# After
env:
  variables:
    JAVA_VERSION: "21"
    JAVA_HOME: "/usr/lib/jvm/java-21-amazon-corretto"
```

##### Pattern 4: CodeBuild image with Java version
```yaml
# Before
version: 0.2
# Using standard image with Java 17
# If comments reference Java 17, update them

# After
version: 0.2
# Using standard image with Java 21
# Update any comments referencing Java version
```

**Note:** Ensure you're using Amazon Corretto (`corretto21`) to maintain consistency with the local development environment.

#### 10.4 Verify buildspec File Changes

After updating the buildspec files, verify the changes:

```bash
grep -r -i "java.*21\|corretto21\|runtime.*21" buildspec*.yml buildspec*.yaml 2>/dev/null
```

This should confirm that Java version references have been updated to 21.

#### 10.5 Test CodeBuild Execution (Optional)

If you have access to AWS CodeBuild, trigger a build to ensure the buildspec changes work correctly with Java 21:

```bash
aws codebuild start-build --project-name <your-project-name>
```

Monitor the build logs to verify that Java 21 is being used during the build process.

---

### Step 11: Execute Gradle Build with Test Cases

After completing all the configuration updates and migrations, it's essential to verify that the application builds successfully and all tests pass with Java 21.

**🔴 CRITICAL - REQUIRED LOGGING FOR STEP 11:**
Create a new "Step 11: Final Build and Test Verification" section in the log file with:
- Build execution command used
- Build result (Success/Failed)
- Compilation warnings or errors (if any)
- Number of tests executed
- Number of tests passed/failed
- Test execution time
- Build artifacts generated and their locations
- JAR/WAR artifact verification results
- Final upgrade status (Success/Completed with warnings/Failed)

#### 11.1 Clean Build with Tests

Execute a clean build with all test cases to ensure the Java 21 upgrade is successful:

```bash
./gradlew clean build -x test
```

This command will:
- Clean the build directory to remove any cached artifacts from previous builds
- Compile the source code with Java 21
- Compile the test code
- Run all unit tests
- Run all integration tests (if configured)
- Generate build artifacts

#### 11.2 Verify Build Success

After the build completes, verify that:
1. The build completed successfully without errors
2. All tests passed
3. No deprecation warnings related to Java version compatibility

#### 11.3 Run Tests Separately (Optional)

If you want to run tests separately without building artifacts:

```bash
./gradlew test
```

For integration tests (if configured separately):

```bash
./gradlew integrationTest
```

#### 11.4 Review Test Results

Check the test results in the console output. For detailed test reports, review:

```bash
# Open test report in browser (macOS)
open build/reports/tests/test/index.html
```

#### 11.5 Verify JAR/WAR Artifacts

Ensure that the build artifacts are created with Java 21:

```bash
# Check the build output directory
ls -lh build/libs/

# Verify the Java version in the JAR manifest
unzip -p build/libs/*.jar META-INF/MANIFEST.MF | grep -i "build-jdk"
```

---

## Post-Upgrade Manual Steps

After the automated upgrade completes, perform these manual steps to ensure a fully functional Java 21 environment:
1. Running end-to-end tests and any other tests (e.g., performance tests, smoke tests, manual tests) that are not executed as part of the Gradle build to identify compatibility issues
2. Manually updating any remaining deprecated APIs not handled by OpenRewrite
3. Building and deploying the application in Dev and Integration environments

---

## References
- [Github Copilot AI Instruction Files Repository](https://github.com/github/awesome-copilot/tree/main/instructions)
- [SDKMAN Installation Guide](https://sdkman.io/install/)
- [Amazon Corretto via SDKMAN](https://sdkman.io/jdks/amzn/)
