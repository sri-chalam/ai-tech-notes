# Claude Code Setup: Giving Access to Common Programs and Sites

# Claude Code Sandbox

When Claude Code runs commands like Gradle build, the commands are executed inside a sandbox. This is for security. For example, it prevents the agent from accidentally accessing or changing files that it should not.  

But sometimes the sandbox also blocks things that are needed for normal development.  

For Java/Gradle projects, I have seen issues like:

1. Gradle cannot download libraries from Maven repositories.
2. Gradle cannot write downloaded libraries to `~/.gradle`.
    1. By default, Gradle cannot write to a a directory outside of the project directory.
3. Tests or applications cannot open a local port or access a required
socket.
4. Commands like `gh` or `acli` may not work because they need network
or credential access.

When this happens, **Claude does not recognize the sandbox restriction as the root cause. It tries various workarounds instead — for example, replacing a socket call with a different Gradle feature or adding JVM arguments**. This can take a lot of time and tokens without fixing the real problem.

So, **if a build or command works in a Macbook terminal but fails from Claude Code, check the sandbox restrictions first**.

**Note**: GitHub Copilot agent does not impose these restrictions. It can run Gradle builds, download libraries from Maven, and open network connections without any additional configuration.

# Give Access to Commonly Used Commands and Sites

Enterprise developers typically do not have authorization to disable the sandbox. Even when it is possible, it is not recommended. Think of the sandbox as a firewall — **everything is blocked by default. Explicitly allow only what is needed: the commands the sandbox blocks, the directories Claude needs to write to, and the websites it is allowed to communicate with**.

For example, in `~/.claude/settings.json`:

```json
{
  "sandbox": {
    "enabled": true,
    "network": {
      "allowLocalBinding": true
    },
    "excludedCommands": [
      "gradle *",
      "./gradlew *",
      "gh *",
      "acli *"
    ],
    "allowedDomains": [
        "github.com",
        "*.atlassian.net",
        "repo.maven.apache.org",
        "search.maven.org",
        "repo1.maven.org",
        "central.maven.org"
      ]
  }
}
```


`excludedCommands` means those commands run outside the sandbox. 

Gradle downloads libraries from Maven repositories and stores them under \~/.gradle. By default, the sandbox does not allow writes outside the project directory, so \~/.gradle must be explicitly added to the allowed write paths.

**Note**: In the example above, gradle \* and ./gradlew \* are listed under excludedCommands, which means Gradle runs entirely outside the sandbox. When a command is excluded, the filesystem.allowWrite boundary does not apply to it — the command was never sandboxed to begin with.

# Claude Permissions and Sandbox Access Are Different

The sandbox configuration alone is not always sufficient. The following `permissions` settings are also required. For example, without `~/.gradle` under `additionalDirectories`, Gradle builds in Claude Code will fail with a sandbox write error.

```json
{
  "permissions": {
    "allow": [
      "Bash(./gradlew *)",
      "Bash(gradle *)",
      "Bash(grep *)",
      "Bash(find *)",
      "mcp__atlassian__getConfluencePage",
      "mcp__atlassian__getAccessibleAtlassianResources",
      "mcp__atlassian__searchJiraIssuesUsingJql",
      "mcp__atlassian__getJiraIssue",
      "mcp__atlassian__atlassianUserInfo",
      "mcp__atlassian__addCommentToJiraIssue",
      "mcp__atlassian__editJiraIssue",
      "mcp__atlassian__createJiraIssue",
      "mcp__atlassian__transitionJiraIssue",
      "mcp__atlassian__getTransitionsForJiraIssue",
      "mcp__atlassian__lookupJiraAccountId",
      "mcp__atlassian__search",
      "mcp__atlassian__getJiraIssueTypeMetaWithFields",
      "mcp__atlassian__getVisibleJiraProjects",
      "mcp__atlassian__getJiraProjectIssueTypesMetadata",
      "mcp__atlassian__addWorklogToJiraIssue",
      "mcp__atlassian__getIssueLinkTypes",
      "mcp__atlassian__createIssueLink",
      "mcp__atlassian__getJiraIssueRemoteIssueLinks",
      "WebFetch(domain:search.maven.org)",
      "WebFetch(domain:repo1.maven.org)",
      "WebFetch(domain:central.maven.org)"
    ],
    "additionalDirectories": [
      "~/.gradle",
      "/var/folders",
      "/Users/username/projects/git-sandbox"
    ]
  }
}
```


There is a small difference between these permissions and sandbox
settings.

For example, `Bash(./gradlew *)` tells Claude that it can run Gradle without asking permission every time. But Gradle may still be blocked by the sandbox when it tries to write to `~/.gradle`.

The following permissions tell Claude not to prompt for approval when retrieving Jira issue details from Claude Code.

```bash
      "mcp__atlassian__getConfluencePage",
      "mcp__atlassian__getAccessibleAtlassianResources",
      "mcp__atlassian__searchJiraIssuesUsingJql",
```


There are times when it is necessary to look at code from other Git repos the team works on. The following permission instructs Claude not to request permission when reading code from any Git repo cloned under the directory `/Users/schalamalasetti/projects/git-sandbox`.

```json
    "additionalDirectories": [
      "/Users/schalamalasetti/projects/git-sandbox"
    ]
```


Spring Boot Controller REST API tests use Tomcat, which writes to a temp directory under `/var/folders`. Claude must be granted write permission to this directory.

```json
    "additionalDirectories": [
      "/var/folders"
    ]
```


Claude sometimes attempts to identify underlying problems such as version compatibility issues between libraries. To do this, Claude needs to access the Maven Central repository to retrieve libraries and check compatibility. Claude must be given permission to access the Maven Central host URLs listed below.

```
      "WebFetch(domain:search.maven.org)",
      "WebFetch(domain:repo1.maven.org)",
      "WebFetch(domain:central.maven.org)"
```


# Opening Access Is an Iterative Process


When working on a use case, if Claude keeps asking permission for a normal operation or something is blocked by the sandbox:

1. Check what exactly is blocked.
2. If the access is reasonable, add the required command, directory or
domain.
3. Run the use case again.
4. Keep the access as small as possible.

When working on a use case, if Claude keeps asking permission for a
normal operation or something is blocked by the sandbox:

1. Check what exactly is blocked.
2. If the access is reasonable, add the required command, directory or
domain.
3. Run the use case again.
4. Keep the access as small as possible.

For example, I was working on a Spring Boot application that uses Apache Arrow and many other libraries. After Java and Spring Boot versions were upgraded, the application had some strange compatibility issues.

Claude was able to check many library versions from Maven repositories and identify the compatibility problem. For this kind of investigation, Claude needs access to the required Maven repositories.

# Do Not Change JVM Arguments for Sandbox Problems

When tests fail due to sandbox restrictions, Claude may attempt to resolve them by adding JVM arguments. This is undesirable — we do not want workarounds applied to bypass sandbox limitations. If the agent begins attempting such fixes, ask the model what instruction should be added to AGENTS.md to prevent this behavior.

Add the following instruction to `AGENTS.md`:

```text
- **Do not add JVM args to `tasks.withType<Test>`** to work around
  local or sandbox environment restrictions, for example
  `-Djava.io.tmpdir`, `-XX:+EnableDynamicAgentLoading`, or
  `-Djdk.attach.allowAttachSelf`. These are environment-specific and
  should not go into the shared build configuration.
```

# \`acli\` May Still Not Work

Even if `*.atlassian.net` is in the allowed domains and `acli` is listed in excluded commands, `acli` is still unable to communicate with Atlassian Jira.

This happens because the org's **managed settings** have `allowManagedDomainsOnly: true`, which means the IT-controlled allowlist takes precedence and any user-defined `allowedDomains` are ignored.

**Only the IT team can resolve this** by adding `*.atlassian.net` to the managed allowlist.

# Claude Code Vs Github Copilot - Default Tool Call Restrictions

In the default Claude Code setup, network and file access are isolated at the OS level. Gradle builds fail until the required domains and programs are allowlisted.

In the default GitHub Copilot setup, commands are controlled with approval prompts instead of OS-level isolation. After a command is approved, it is run in the real integrated terminal with normal user permissions and full network access.

Copilot is less restrictive by default — no sandbox permissions, domains, or programs need to be identified and allowlisted before tasks can run. A sandboxed environment is available but is not enabled by default.
