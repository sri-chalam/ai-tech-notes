# AI-Assisted Java Upgrades: A Hybrid Approach with AI Instructions and OpenRewrite

## Introduction

AI coding agents have become remarkably powerful tools, capable of implementing solutions in dozens of different ways. However, this flexibility presents a challenge: organizations have specific conventions, standards, and requirements that guide how solutions should be implemented in their particular context.

To effectively leverage AI coding agents, detailed instructions are needed to communicate these organization-specific conventions and guide the agent toward the desired implementation approach. **Even as AI agents continue to mature and become more sophisticated, the need for clear and unambiguous instructions that follow team and organization-specific conventions will remain essential**.

What might seem like a straightforward task—such as upgrading a Java version—often involves numerous organization-specific decisions that AI agents need guidance to navigate effectively.

This article demonstrates this guidance approach in practice, presenting a method for automating Java version upgrades using custom AI instruction files. **Importantly, these instructions automate the complete workflow end-to-end—from environment setup (package manager installation, Java installation, certificate imports) through code migration—not just the code changes themselves**. 

While this article uses the Java 17 to 21 upgrade as the primary example, instruction files for both Java 17 to 21 and Java 21 to 25 upgrades are included. Beyond the specific examples, this article also covers the theoretical foundation of this approach, which readers can adapt to their own automation use cases.

**The Approach:** AI agents like GitHub Copilot App Modernization and Amazon Q Developer can perform Java upgrades, they lack awareness of organization-specific conventions out of the box. The hybrid approach presented here uses **custom AI instruction files** that combine:

1. **Organization-specific requirements**—encoding team conventions such as Java distribution, installation method, and trusted certificate import
2. **OpenRewrite**—a popular and actively maintained, community-driven tool that upgrades dependencies, fixes deprecated APIs, and modernizes code with safe new Java language features through predefined recipes
3. **AI-assisted problem-solving** for issues that OpenRewrite can't handle automatically
4. **Pre-emptive guidance for known patterns**—explicitly providing instructions for the library and plugin upgrades commonly needed across most Java version upgrades that would otherwise require the AI to discover through time-consuming trial-and-error

This approach is LLM-agnostic (works with Claude, ChatGPT, Gemini, or any AI coding agent) and requires no additional subscriptions beyond your chosen LLM.

## Understanding AI Instructions vs. AI Agents

To understand the approach described in this article, it's helpful to distinguish between two related but different concepts:

### AI Instructions File
An AI Instruction Markdown file is a structured document (typically with a .md extension) that contains prompts, guidelines, and instructions for how an AI model should behave or respond.

An instructions file contains things like:
- Task-specific instructions
- Behavioral guidelines and constraints

### AI Agent
An AI agent is a more complex system that can:
- Perceive its environment (receive inputs)
- Reason about what actions to take
- Act autonomously to achieve goals
- Learn from feedback (in some cases)
- Use tools or call external APIs
- Maintain state across interactions
- Plan multi-step sequences of actions

This article uses instruction files to guide AI coding agents through the hybrid upgrade approach described below.

## Why Automating Java Upgrades Is Complex?

At first glance, upgrading a Java project from version 17 to 21 might seem straightforward. One could simply prompt an LLM: "upgrade this project from Java 17 to Java 21" and expect it to handle everything. While such a prompt might accomplish a few basic changes, the reality is far more nuanced.

### The Hidden Complexity

A comprehensive Java upgrade involves addressing numerous organizational, technical, and environmental considerations:

**Java Distribution and Provider Selection**
- Which Java distribution should be used? (Oracle JDK, OpenJDK, Amazon Corretto, Azul Zulu, Eclipse Temurin, GraalVM, etc.)

**Installation and Environment Management**
- What installation method should be employed? (SDKMAN, jenv, manual installation, package managers)
- Does the organization follow specific conventions for Java installation paths?
- Are there trusted certificates that need to be imported post-installation?
- How should multiple Java versions be managed on development machines?

**Build Tool Variations**
- Should upgrade instructions cover Maven, Gradle?
- For Gradle projects, are build files written in Groovy or Kotlin DSL?

**Additional Considerations**
- CI/CD pipeline updates (GitHub Actions, Jenkins, GitLab CI, AWS CodeBuild, etc.)
- Docker and container base image updates

## Why AI Custom Instructions Instead of AI Agents?

AI agents, such as GitHub Copilot App Modernization and Amazon Q Developer, offer features to upgrade Java and Spring Boot versions. However, having a custom instruction file for upgrades has multiple advantages.

### Benefits of AI Custom Instructions File

**LLM-Agnostic Flexibility**
- Works with any LLM (Claude, ChatGPT, Gemini, etc.), not locked to a specific vendor
- Lower cost—no additional subscriptions beyond the chosen LLM

**Addresses What AI Upgrade Agents Don't Handle**
- AI upgrade agents don't address environment-specific requirements:
  - JDK distribution selection (Amazon Corretto vs others)
  - Installation methods (SDKMAN vs package managers)
  - Certificate management (importing trusted certificates)

**Includes the Same Build/Fix Loop as Copilot App Modernization**
- Custom instructions can direct the LLM to perform iterative build/fix cycles
- Addresses what OpenRewrite doesn't handle

### What About Third-Party Library Migrations?

For third-party library migrations, custom instruction files and AI agents face identical constraints—neither approach has an inherent advantage:

**The Core Issue:**
- OpenRewrite can only migrate libraries that have **predefined recipes**
- For libraries without recipes (e.g., Ehcache2->Ehcache3, proprietary frameworks), OpenRewrite cannot perform automated transformations
- For libraries without recipes (e.g., Ehcache2->Ehcache3, proprietary frameworks), OpenRewrite cannot perform automated transformations

**Bottom Line:**
- For standard Java API upgrades: Recipe-based transformations work well and produce deterministic results
- For third-party libraries without recipes: Build/fix loops (whether in Copilot or custom instruction files) can address many cases through Internet search and iteration, though this requires human oversight and may not handle all edge cases

## OpenRewrite vs. AI Upgrade Agents for Code Transformations

**OpenRewrite's Strengths:**
- **Deterministic Transformations:** Provides predictable results—you know exactly what changes will be made, and the same recipe produces identical results on every run
- **Enterprise-Scale Migrations:** Well-suited for processing hundreds of repositories with consistent, auditable transformations

**AI Upgrade Agent's Strengths (e.g., Copilot App Modernization):**
- **Contextual Problem-Solving:** Analyzes specific error messages and suggests fixes tailored to the application's architecture
- **Web-Enhanced Knowledge:** Can access recent changes to languages and libraries beyond training cut-off dates
- **Interactive Iteration:** Allows conversation and feedback to refine solutions
- **Multi-Step Reasoning:** Investigates errors, proposes multiple solution options, and adapts based on build results

## Benefits and Trade-offs of a Hybrid Approach

Rather than choosing one tool exclusively, a hybrid approach uses OpenRewrite for deterministic transformations first, then leverages an LLM (guided by instruction files) to handle remaining issues. This combines the reliability of recipe-based migrations with the flexibility of AI-driven problem-solving.

**Benefits**

1. **Broader Coverage**: OpenRewrite handles known patterns with predefined recipes; the LLM, guided by the instruction file, addresses edge cases, deprecated APIs, and third-party libraries lacking recipes.

2. **Iterative Problem Resolution**: When compilation errors or test failures remain after OpenRewrite, the LLM iteratively analyzes errors, researches solutions, and applies fixes—repeating until the build succeeds or a maximum iteration limit is reached.

3. **Pre-emptive Guidance for Known Patterns**: Certain libraries—such as Lombok and Spotless—typically require version upgrades with every Java upgrade. Rather than relying on the AI to discover this through internet searches and multiple build failures, the instructions explicitly handle these common scenarios upfront. **This reduces wasted iterations, prevents incorrect decisions from AI guesswork, and encodes institutional knowledge directly into the instructions**. For this upgrade, pre-emptive guidance covers Lombok and Spotless —applied only if already used in the project.

**Trade-offs**

1. **Extended Execution Time**: The iterative build/fix cycle typically requires 20-30 minutes or more, depending on the number of issues to resolve.

2. **Skill Atrophy Risk**: Over-reliance on automation may reduce familiarity with migration details. Reviewing upgrade logs helps mitigate this.

## AI Instruction Files - Limitations and Expectations

When using AI instruction files (the approach demonstrated in this repository) for Java upgrades, it's important to understand the nature and limitations:

### Instruction Coverage Limitations

While the build/fix loop attempts to handle new scenarios through AI analysis and Internet research, not all edge cases can be resolved automatically. Every codebase is different, and some unique configurations, libraries, or code usage patterns may require manual intervention.

## OpenRewrite and Recipes Used

**OpenRewrite** is an automated refactoring tool that applies code transformations through reusable security fixes, migrations, and code quality improvement recipes.

**Recipes** are predefined transformation patterns that transform code to adopt new language features, fix deprecated APIs, patch security vulnerabilities, or improve code quality. In addition to built-in recipes, there are community-provided recipes available, and teams can create custom recipes to implement organization-specific transformations.

### Recipes Applied in This Upgrade

The upgrade instructions use the following OpenRewrite recipes:

1. **`org.openrewrite.java.migrate.UpgradeToJava21`** - Migrates Java code from 17 to Java 21
2. **`org.openrewrite.java.migrate.SwitchPatternMatching`** - Applies pattern matching in switch statements

**Purpose of Additional Recipe:** Beyond basic compatibility provided by UpgradeToJava21, the SwitchPatternMatching recipe leverages Java 21's elegant new language features to produce more expressive and maintainable code.

## Prerequisites and Constraints for Using the Upgrade Instructions

Before using the Java 17 to Java 21 upgrade instruction file, ensure your environment meets the following prerequisites and constraints:

### Environment Requirements

1. **Operating System**: macOS
2. **Current Java Version**: Must be Java 17
3. **Build Tool**: Gradle with Groovy DSL (`build.gradle`) or Kotlin DSL (`build.gradle.kts`) - Maven requires adaptation
4. **Required Tools**: `curl` and `git`
5. **Recommended Editor**: Visual Studio Code with GitHub Copilot or Claude Code plugins (this guide was tested with these configurations)

### Scope Limitations

1. **Java Upgrade Only**: The instructions focus **exclusively on upgrading Java from version 17 to version 21**. Spring Boot is not upgraded as part of this process, unless an application library upgrade requires a corresponding Spring Boot upgrade.

2. **Amazon Corretto JDK**: The instructions use **Amazon Corretto** as the Java distribution. If your organization requires a different distribution (Oracle JDK, Eclipse Temurin, Azul Zulu, etc.), the installation steps will need modification.

3. **SDKMAN for Java Management**: The instructions use **SDKMAN** to manage Java installations. Organizations using alternative tools (jenv, manual installations, package managers) will need to adapt the installation approach.

### Gradle Wrapper Version Compatibility

For Java 21 compatibility, Gradle 8.5+ is sufficient. However, to use OpenRewrite recipes, the latest Gradle 8.x version is preferred:
- **Gradle 8.14.4+**: Preferred for using the latest OpenRewrite version

The upgrade instructions will automatically upgrade Gradle wrapper to version 8.14.4 if the current version is below 8.14.

**Note:** The instructions will only upgrade Gradle wrapper if the current wrapper version is below 8.14. If the project already uses Gradle 8.14 or higher (including 9.x), the Gradle wrapper will not be modified. Projects without an existing Gradle wrapper will not have one installed.

### Compatibility Note

If the environment does not match these prerequisites, the following actions are recommended:
- Review the instruction file and adapt it to your specific environment
- Consider creating a customized version of the instruction file for your use case

## Organization Trusted Certificates

If custom trusted certificates are used by the organization (e.g., for internal Certificate Authorities, or for TLS-inspecting proxies—also known as TLS interception, SSL forward proxy), these certificates need to be imported into the newly installed Java 21 keystore. To enable this, organization's trusted certificates should be copied to a specific directory before executing the Java upgrade instructions. The AI-assisted upgrade process will then import these certificates into the newly installed Java 21 trust store.

**Note:** This step is important for OpenRewrite recipes to work correctly. The recipes download Gradle artifacts during execution, and missing organization certificates cause PKIX SSL errors that prevent the upgrade from completing.

### Certificate Preparation Requirements

1. **Certificate Directory**: All organization trusted certificates need to be copied to `~/trusted-certs/`

2. **Supported Certificate Formats**: Only certificates with `.pem`, `.cer`, or `.crt` file extensions will be imported. Files with other extensions will be ignored during the import process.

3. **Certificate Import Process**:
   - The upgrade instructions will automatically detect and import all certificate files with supported extensions from `~/trusted-certs/`
   - Each certificate will be imported into the Java cacerts truststore with a unique alias
   - The import only occurs when Java 21 is freshly installed (not if it's already present)
   - Note: The process imports files based on extension only; it does not validate certificate authenticity or format

4. **Optional Step**: If your organization does not use custom trusted certificates, this step can be skipped. The upgrade continues normally if no certificates are found.

Once the certificates are in place, proceed with the Java upgrade instructions as documented below.

## How the Upgrade Instructions Work

The upgrade process is automated through a series of steps that handle both environment setup and code migration:

### Upgrade Steps

For detailed step-by-step execution instructions, see [java-17-to-21-upgrade-instructions.md](https://github.com/sri-chalam/ai-tech-notes/blob/main/articles/ai-assisted-java-upgrade/java-17-to-21/instructions/java-17-to-21-upgrade-instructions.md)

1. **Identify and Set Project Root Directory**: Establish the working directory for all subsequent commands.

2. **Verify Current Java Version**: Identify and log the current Java version. Version detection can be challenging in some cases, but if the upgrade instructions are executed on a project already on Java 21, OpenRewrite will skip the upgrade gracefully.

3. **SDKMAN Installation**: Installs SDKMAN if not already present at `~/.sdkman/` for Java version management

4. **Java 21 Installation**: Installs Amazon Corretto JDK 21 via SDKMAN if not already present in `~/.sdkman/candidates/java/`
   - If JDK 21 exists in other locations, the instructions still install it via SDKMAN for consistency
   - Installation is skipped only if Amazon Corretto 21 is already installed through SDKMAN

5. **Trusted Certificates Import**: Automatically imports organization certificates from `~/trusted-certs/` into the Java 21 truststore during fresh installations

6. **Upgrade Gradle Wrapper (If Needed)**: Upgrades Gradle wrapper to 8.14.4 if needed (see Prerequisites for details).

7. **OpenRewrite Plugin**: Adds the OpenRewrite Gradle plugin to the project's build configuration. This is the same underlying tool used by AI-powered upgrade assistants like GitHub Copilot App Modernization and Amazon Q Developer.

8. **Common Library Upgrades**: Proactively upgrades libraries that commonly require updates during Java migrations—Lombok and Spotless—to avoid unnecessary build/fix loop iterations.

9. **Use OpenRewrite to Migrate Java Code**: Executes OpenRewrite plugin to automatically migrate the application to Java 21.

10. **Iterative Build/Fix Loop**: After OpenRewrite migration, performs an automated build/fix cycle to resolve any remaining compilation errors and test failures that OpenRewrite couldn't handle automatically. The loop executes up to 5 iterations maximum and exits early if the same error persists for 3 consecutive attempts. This loop:
   - Executes `./gradlew clean build` to compile code and run tests
   - Analyzes compilation errors and identifies root causes
   - Applies fixes using a three-tier resolution strategy:
     1. **OpenRewrite recipes** - Searches for and applies automated recipes (safest approach)
     2. **Internet search** - Researches solutions from official Java documentation, Stack Overflow, and technical resources
     3. **Automated code fixes** - If a solution is found through Internet research, automatically applies targeted code changes and documents the changes
   - Re-runs the build after each fix
   - Logs unresolved errors if maximum iterations are reached

11. **CI/CD Pipeline Updates**: Updates CI/CD configuration files (see "Updated CI/CD and Build Files" section for details)

12. **Verify the Upgrade**: Execute a Gradle Build to verify that the upgrade was successful.

### Updated CI/CD and Build Files

The upgrade instructions automatically update Java version references in the following files:

- **GitHub Actions**: `.github/workflows/*.yml`
- **AWS CodeBuild**: `buildspec*.yml`
- **Docker**: `Dockerfile`, `Dockerfile.*`
- **Gradle Build Files**: `build.gradle` or `build.gradle.kts`, `gradle.properties`, `gradle/wrapper/gradle-wrapper.properties`

**Updates applied to build.gradle:**
- sourceCompatibility and targetCompatibility (`'17'` → `'21'`)
- JavaVersion enum references (`JavaVersion.VERSION_17` → `JavaVersion.VERSION_21`)
- OpenRewrite plugin addition/update (version 7.25.0+)
- OpenRewrite dependencies and recipe configuration
- Gradle wrapper upgrade to 8.14.4 (if current version < 8.14)

### Upgrade Log Documentation

All fixes, changes, and unresolved errors are documented during the upgrade process in:
```
/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md
```

The log file contains:
- **Fixes Applied**: Details of each successful fix including file paths, error types, solutions, and sources
- **Unresolved Errors**: Complete documentation of errors that could not be resolved, with attempted solutions and references
- **Build/Test Summary**: Iteration counts, success metrics, and final status

This log provides full traceability of changes made during the upgrade and serves as a reference for any manual interventions needed.

Just as good logging is essential for any production application, comprehensive logging is equally important for AI-assisted upgrades—it provides transparency into the decisions made at each step and their outcomes.

**Example Log:** [java-17-to-21-upgrade-log-1.md](https://github.com/sri-chalam/spring-petclinic-ai-java-upgrade/blob/java-21-upgrade/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md) — generated after upgrading a sample application from Java 17 to Java 21.

## How to Execute the Java 17 to Java 21 Upgrade Instructions

### Overview

The upgrade process uses a single instruction file that should be copied into any repository that needs to be upgraded from Java 17 to Java 21. This instruction file contains the detailed step-by-step procedures for the upgrade.

Once the file is in place, the upgrade can be executed by referencing the instruction file in an LLM, which will systematically work through all the necessary steps.

### Step 1: Download the Instruction File

Download the instruction file from the following URL:
```
https://github.com/sri-chalam/ai-tech-notes/blob/main/articles/ai-assisted-java-upgrade/java-17-to-21/instructions/java-17-to-21-upgrade-instructions.md
```

Save this file in your Git repository that needs to be upgraded from Java 17 to Java 21. The suggested location is:
```
<Git repo root>/docs/ai-tasks/instructions/
```

### Step 2: Execute the Upgrade with an LLM

**Important:** Before executing the prompt, open the instruction file (`docs/ai-tasks/instructions/java-17-to-21-upgrade-instructions.md`) in your VS Code editor. This ensures the AI coding agent can access the file contents.

The following prompt is LLM-agnostic and works with any AI coding agent (Claude Code, GitHub Copilot, ChatGPT, etc.):

**LLM Prompt:**
```
Upgrade the application from Java 17 to Java 21 using the instructions in docs/ai-tasks/instructions/java-17-to-21-upgrade-instructions.md.
If you cannot access this file, stop and ask me to provide its contents.
Do not infer or invent upgrade steps.
```

**Note:** If the above prompt doesn't work with the AI coding agent being used, some agents support direct file reference syntax that may work as an alternative. For example, Claude Code uses `@docs/ai-tasks/...` and GitHub Copilot uses `#file:docs/ai-tasks/...` to reference files.

The AI agent will read the instruction file and execute each step of the upgrade process systematically.

## Post-Upgrade Validation and Configuration

### Step 1: Validate the Java 21 Upgrade

After the upgrade instructions have been executed, it's important to validate that the upgrade was successful. The instruction file includes Gradle build execution with unit tests at the end. However, additional validation steps are recommended to ensure comprehensive testing and proper IDE configuration.

#### 1.1 Execute End-to-End Tests

The application's end-to-end test suite should be executed to verify that the system works correctly as a whole. Ensure that all critical user workflows and integrations are functioning correctly with Java 21.

#### 1.2 Execute Integration Tests (if available)

If the project has integration tests, they should be executed using the project's specific test command. For example:

```zsh
# Example for Gradle projects
./gradlew integrationTest
```

#### 1.3 Execute Manual Tests

Perform manual testing to check the application works as expected.

1. Start the application locally or in Development environment
2. Test critical user journeys
3. Verify database connectivity and operations
4. Check logging and monitoring functionality
5. Validate any Java version-specific features or optimizations
6. Verify communication to external services (AWS services or microservices)

### Step 2: Configure IDE Settings

For IDEs such as IntelliJ IDEA, update the project settings to use the new Java 21 JDK:

**IntelliJ IDEA Configuration:**

1. **Update Gradle JDK Settings:**
   - Navigate to: `Settings/Preferences` → `Build, Execution, Deployment` → `Build Tools` → `Gradle`
   - Set "Gradle JVM" to Java 21 (e.g., `21.0.x-amzn` or your installed JDK)

2. **Update Project Structure:**
   - Navigate to: `File` → `Project Structure` → `Project`
   - Set "SDK" to Java 21
   - Set "Language level" to "SDK default"

After making these changes, rebuild the project in the IDE to ensure everything compiles correctly with the new Java version.

### Step 3: Upgrade Path to Java 25 (Optional)

If an upgrade to Java 25 is needed, separate instructions are available for upgrading from Java 21 to 25.

**For Java 21 to Java 25 Upgrades:**

To upgrade an application from Java 21 to Java 25, use a separate instruction file specifically designed for this upgrade path.

**For Java 17 to Java 25 Upgrades:**

If there is a need to upgrade an application from Java 17 to Java 25, use a two-step process:

1. **Step 1**: Upgrade from Java 17 to Java 21
2. **Step 2**: Upgrade from Java 21 to Java 25

**Why Separate Instruction Files?**

Using individual instruction files for each upgrade path (17→21, 21→25) is more modular and reduces complexity. If a single instruction file attempted to handle multiple upgrade paths (17→21 or 21→25 or 17→25), the instructions would become complex and could confuse LLMs or cause hallucinations. Separate, focused instruction files ensure clearer execution and more predictable results.

## Lessons Learned

The process of building and executing these AI instructions for project upgrades revealed several important lessons that can save time and prevent common pitfalls.

### AI-Generated Instructions Need Human Review

When first creating these instructions with AI assistance, it's easy to trust the output without sufficient scrutiny. A critical example: the initial instructions added the `activeRecipes` list directly to `build.gradle`.

While this worked functionally, OpenRewrite's documentation indicates this isn't best practice. Since Java upgrades are one-time migrations, these recipes don't need to be committed to the build file. Additionally, teams often want to run specific recipes (like static code analysis) separately and more frequently.

Another critical example: the initial instructions specified an outdated Gradle OpenRewrite plugin version that was incompatible with the OpenRewrite recipe bom version. This version mismatch caused cryptic build failures that were difficult to diagnose, highlighting how AI models may reference older documentation or outdated version combinations.

**Takeaway:** Always review AI-generated instructions critically and consult official documentation. Question the approach, discuss alternatives with the AI agent, and refine the instructions iteratively.

### The Build/Fix Loop Required Multiple Iterations

The initial build/fix loop instructions had significant gaps:

- **Infinite loop risk:** If the AI agent couldn't resolve certain errors, the loop would continue indefinitely without recognition
- **Missing diagnostic guidance:** There were no instructions on how to systematically identify the root cause of compilation errors
- **Lack of resolution strategies:** The instructions didn't provide a tiered approach for attempting fixes (recipes first, then research, then code changes)

These instructions required refinement through trial and error, adding:
- Maximum iteration limits (5 attempts)
- Early exit conditions (same error 3 times consecutively)
- A three-tier resolution strategy (OpenRewrite recipes → Internet research → Automated fixes)
- Comprehensive logging of all attempts and outcomes

### Provide Explicit Instructions for Common Library Upgrades

While the build/fix loop can identify and upgrade necessary libraries, relying on it for common scenarios wastes iterations. Certain libraries—such as Lombok—typically require version upgrades with every Java upgrade. Rather than depending on the AI agent to discover this through internet searches and multiple build failures, provide explicit instructions to upgrade these common libraries and plugins before entering the build/fix loop.

For this upgrade, the instructions were added to explicitly handle (only if already used in the project—skipped otherwise):
- **Lombok**: Upgrade to the latest version, which is backwards compatible, has fewer vulnerabilities, and includes more features
- **Spotless**: If already present, upgrade to the latest version. If the deprecated Google Java Format plugin is present instead, replace it with Spotless

**Takeaway:** Identify libraries and plugins that commonly require upgrades during Java version migrations and add explicit upgrade instructions for them. Reserve the build/fix loop for unexpected issues rather than predictable compatibility updates.

### TLS Interception Certificates Must Be Imported Before Upgrade

Organizations using TLS interception (SSL forward proxy) must ensure certificates are available before executing the upgrade. During one test project upgrade, the AI coding agent was unable to download the Gradle wrapper because the organization's TLS interception certificates had not been imported after installing the new JDK. Copying these certificates to `~/trusted-certs/` prior to running the upgrade resolved the issue.

### Upgrade Spotless Gradle Plugin Version, Not Formatting Styles

During an AI-assisted Java upgrade, Spotless format verification failed due to plugin version incompatibility. Rather than upgrading the Spotless version, the AI agent resolved the issue by changing the formatting style configuration—which reformatted many files unnecessarily. Explicit instructions were added to upgrade the Spotless plugin version, rather than altering the formatting style. This prevents unintended widespread code changes.

### Monitor the Upgrade Process Actively

When the AI coding assistant is executing the upgrade, keep watching the logs in the chat window. Some AI coding agents, such as GitHub Copilot, also execute shell scripts and commands in a separate terminal window—monitor both the chat window and the terminal to track progress. Other agents, such as Anthropic Claude Code, execute commands within the chat window itself. These logs indicate whether the upgrade is proceeding correctly and help catch issues early.

If the logs in the chat window or terminal indicate that the upgrade is going in a completely wrong direction, or if the AI appears stuck, stop the execution immediately. Revert all changes and start over rather than allowing the AI to continue down an incorrect path.

### Know When to Stop and Restart

In some cases, the AI coding assistant may encounter a state where it cannot proceed correctly. In one instance, the AI reported that the terminal was in a "bad state" and could not execute commands. Rather than waiting for the terminal issue to be resolved, the AI set aside the instruction file and attempted to perform the upgrade using its own knowledge—without executing the prescribed commands. This resulted in an upgrade that bypassed the carefully crafted instructions.

When the AI deviates significantly from the instruction file or encounters persistent execution issues, it's better to stop the session, fix the underlying problem (such as terminal state or environment issues), and restart the upgrade from the beginning.

### Additional Lessons

**Some Steps May Be Skipped Silently**

During testing, the LLM successfully installed the JDK but silently skipped the step to import trusted certificates. To address this, the instruction was updated to include "it is **CRITICAL** to import..." — emphasizing the word "critical" helps signal to the LLM that this step should not be skipped.

**Clear Instructions Prevent Hallucination**

Initial instructions stated "upgrade Gradle wrapper if the current version is below 8.5." However, the AI still attempted to upgrade projects that already had Gradle 8.5. The fix was to specify both conditions explicitly: "if version >= 8.5, SKIP; if version <= 8.4, proceed."

**Use Terminology Precisely and Consistently**

Use terminology precisely and consistently throughout instruction files. Define keywords with specific meanings at the beginning of the document—for example, ABORT (stop immediately), SKIP (continue to next step), and CRITICAL PATH (failure aborts the upgrade). This prevents the LLM from interpreting terms ambiguously. 

**Use Consistent Shell Scripting**

The instruction file includes shell scripts in multiple sections (SDKMAN setup, JDK installation, Gradle wrapper, etc.). Initially, these scripts mixed zsh and bash, causing the AI coding agent to hang when switching between shells. Using bash consistently across all sections resolved this.

**Log the Outcome of Each Step**

To debug issues such as determining whether a step in the upgrade process was missed, review the log markdown file at `/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md`.

**Verify Log Output After Upgrade**

After running the upgrade, review the log file to verify the execution flow, step results, and decisions made. The log includes Java version detection results, SDKMAN and JDK installation status, certificate import outcomes, Gradle wrapper updates, OpenRewrite migration details, build/fix loop iterations, and CI/CD file updates. This provides a complete audit trail for troubleshooting and validation.

**Cacerts Path Differs Between Java Versions**

During an upgrade, the AI agent identified that the application was using an outdated cacerts path in Dockerfile. In Java 8 and earlier, the truststore was located at `$JAVA_HOME/jre/lib/security/cacerts`. Starting with Java 9, the path changed to `$JAVA_HOME/lib/security/cacerts`. The upgrade process detected an existing path bug and corrected this path.

**Leverage Multiple LLMs for Better Results**

Different LLMs produced different suggestions when creating these instructions. Comparing outputs from Claude Sonnet, GPT, and Gemini helped identify the best approaches for important steps. For example, multiple models suggested that LLM-agnostic instruction files benefit from a structured process flow. Multiple models were asked to generate this flow, and the best suggestion was chosen.

**Verify Generated Instructions Thoroughly**

When asked to generate an instruction for adding an OpenRewrite dependency with the latest release version, the LLM produced invalid syntax. Always review generated instructions and code carefully before incorporating them into instruction files.

**Respond to Terminal Prompts During Execution**

Some commands executed by the AI may prompt for user input in the terminal—for example, "Do you want to make this Java the default? (y/n)". If these prompts are not answered, the upgrade will stall. Keep an eye on the terminal for any interactive prompts that require a response.

**Scroll the Chat Window to Check for Waiting Prompts**

In GitHub Copilot, the AI chat window sometimes does not auto-scroll to show new messages. If the upgrade appears stalled, manually scroll down in the chat window. Manual scrolling may reveal that the AI assistant is waiting for input or confirmation that wasn't visible. Anthropic Claude Code does not have this issue—its chat window auto-scrolls correctly.

**Timestamp Logging May Not Be Followed Correctly**

The instruction to log timestamps for each step is not always followed correctly in Github Copilot. In some cases, multiple steps—or all steps—may show the same timestamp in the log file. Be aware of this limitation when reviewing logs for timing information.

## Where to Go From Here

The key takeaway from this work is that **instruction files created with AI assistance can deliver substantial value with relatively modest effort**—but they're not magic bullets.

Creating these instruction files takes a fraction of the time a full custom AI agent would require, while still providing much of the benefit. The build/fix loop mimics what sophisticated AI upgrade agents do, but with full transparency into every decision.

However, every codebase is unique. Users will likely need to:
- Adjust the instructions for their specific Java distribution and tooling
- Add organization-specific requirements not covered here
- Handle edge cases that arise in their particular context

These instruction files should be treated as **collaborative starting points**. They work best when treated as **living documents, refined based on real-world experiences and shared improvements** with the community.

One area for future improvement: the instructions currently hardcode version numbers for Gradle dependencies, plugins, and recipe boms. A future iteration could parameterize these values—defining them in a single section—making updates easier and reducing the risk of version incompatibilities.

Consider reviewing the instruction files to understand what they're doing—**relying solely on AI without understanding the underlying steps may gradually erode debugging and troubleshooting skills**.

An instruction file is also available for Java 21 to 25 upgrades. Feedback on what works, what doesn't, and how these instructions have been adapted for different use cases is welcome and appreciated.

## Example Repositories

The instructions discussed in this article were tested on a couple of popular Java Spring Boot projects. The Java 21 to 25 example is presented first as it better demonstrates the broader capabilities of AI-assisted upgrades, including a detailed upgrade log.

### Example Repository of Java 21 to 25 Upgrade

[Java 21 to 25 Upgrade Example - Realworld Java21 Spring Boot3](https://github.com/sri-chalam/realworld-java21-ai-assisted-upgrade?tab=readme-ov-file#how-to-upgrade-this-project-using-ai-instructions)

The `main` branch contains the source code before the upgrade. The branch `java-25-upgrade` has code after the upgrade.

A detailed AI-assisted upgrade log showing all the changes made during the upgrade process can be viewed at:
[Java 21 to 25 Sample Upgrade Log](https://github.com/sri-chalam/realworld-java21-ai-assisted-upgrade/blob/java-25-upgrade/docs/ai-tasks/logs/java-21-to-25-upgrade-log.md)

### Example Repository of Java 17 to 21 Upgrade
[Java 17 to 21 Upgrade Example - Spring PetClinic](https://github.com/sri-chalam/spring-petclinic-ai-java-upgrade?tab=readme-ov-file#how-to-upgrade-this-project-using-ai-instructions) 

The `main` branch contains the source code before the upgrade. The branch `java-21-upgrade` has code after the upgrade.

A detailed AI-assisted upgrade log showing all the changes made during the upgrade process can be viewed at:
[Java 17 to 21 Sample Upgrade Log](https://github.com/sri-chalam/spring-petclinic-ai-java-upgrade/blob/java-21-upgrade/docs/ai-tasks/logs/java-17-to-21-upgrade-log.md)

## References

- [Java 17 to 21 Upgrade Instructions](https://github.com/sri-chalam/ai-tech-notes/blob/main/articles/ai-assisted-java-upgrade/java-17-to-21/instructions/java-17-to-21-upgrade-instructions.md) - AI instruction file for upgrading Java applications from version 17 to 21

- [Java 21 to 25 Upgrade Instructions](https://github.com/sri-chalam/ai-tech-notes/blob/main/articles/ai-assisted-java-upgrade/java-21-to-25/instructions/java-21-to-25-upgrade-instructions.md) - AI instruction file for upgrading Java applications from version 21 to 25

- [GitHub Awesome Copilot - Instructions](https://github.com/github/awesome-copilot/tree/main/instructions) - A community-contributed collection of instruction files for GitHub Copilot and AI coding agents

- [OpenRewrite Java Recipes Catalog](https://docs.openrewrite.org/recipes/java)
