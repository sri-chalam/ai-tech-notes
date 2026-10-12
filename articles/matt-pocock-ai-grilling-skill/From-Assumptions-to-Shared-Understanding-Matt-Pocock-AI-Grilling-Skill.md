# From Assumptions to Shared Understanding: Matt Pocock’s AI Grilling Skill

> **In short:** Matt Pocock’s agent skills bring proven engineering practices, such as deep modules and test-driven development, into the way a coding agent works. Code generated with them is well structured into modules, easy to understand, and easy to change. They are also very popular: as of October 11, 2026, the GitHub repository has about 284K stars. **Among these skills, grilling is the most useful, whether a feature starts as a vague idea or a clear requirement.** Before a final requirement or code is produced, the AI grills the person working on it, in rounds, until every case is covered and nothing is guessed. **For the product team, it clarifies requirements; for developers, it also clarifies design choices and implementation options.**

## The Problem: Gaps Are Filled with Guesses

**One of the most common reason software goes wrong is not bad code. It is misalignment**. It is assumed that the agent understood the requirement. The agent generates the code, and it is not what was meant. Even a carefully written requirement has loose ends, and the model fills those gaps with its own assumptions. The code compiles, the tests pass — and the result is still wrong.

The same gap exists earlier in the chain. A requirements document reaches engineering with cases nobody thought about and words that two readers understand differently. Each gap becomes a question asked late, or worse, a guess made silently.

The fix is Matt Pocock’s `grilling` skill. It starts a **grilling session**: before any code or final requirement is produced, the agent interviews the person working on it. The questions are relentless — every loose end is followed and every case is raised, until the human and the AI share one understanding of what is being built.

The `grilling` skill file is only about twenty lines long, but its benefits are large.

## `grilling`, `/grill-me` and `/grill-with-docs`

All three run the same interview. They differ in who starts them, what they read, and what they write.

| **Skill** | **Started By** | **Needs a code repository** | **Writes files** | **Best for** |
| --- | --- | --- | --- | --- |
| `grilling` | The agent, automatically (or the person, by typing `/grilling`) - User and Model Invoked | No | No | Any “grill me” request; <http://claude.ai> |
| `/grill-me` | The person only -  User Invoked | No | No | Requirements documents, plans, small tickets |
| `/grill-with-docs` | The person only - User Invoked | Yes | `GLOSSARY.md` and ADRs | Larger features in a code repository |

`grilling`** is the interview itself; the other two are built on it.** It is *model-invoked*: the agent reaches for it whenever “grill me” is typed or another skill needs an interview. `/grill-me` and `/grill-with-docs` are *user-invoked*: they start only when typed by name.

`/grill-me` is a typed shortcut to `grilling`. The interview is identical, but it starts only when asked for, and it needs `grilling` installed to work. `/grill-with-docs` runs `grilling` together with `domain-modeling`, which records new terms in `GLOSSARY.md` and hard-to-reverse decisions as ADRs. Most sessions write no ADRs, and that is normal.

`/grill-with-docs` is used inside a code repository, because the glossary and ADRs are saved there. `grilling` and `/grill-me` need no repository at all, so they work anywhere, including <http://claude.ai> .

**The choice depends on whether the session needs to read the code and keep records.** A product owner reviewing a requirement uses `grilling` or `/grill-me`. A developer preparing to build a feature uses `/grill-with-docs`.

## Installation of Matt Pocock Skills

### Claude Code Terminal

The suggested way to install Matt’s skills, per the project’s GitHub page, is via the official Claude marketplace:

> claude plugin install mattpocock-skills@claude-plugins-official

**Changes to the skills are pulled automatically at startup**, so they do not need to be manually pulled now and then.

If this command errors out — a known issue some users are currently running into with the `claude-plugins-official` marketplace — fall back to adding Matt’s marketplace directly:

> claude plugin marketplace add mattpocock/skills
claude plugin install mattpocock-skills@mattpocock

When the skills are installed from Matt’s marketplace directly, they are not updated automatically, so the update commands below should be run now and then.

To update Matt’s skills, execute the following commands in a terminal window:

> claude plugin marketplace update mattpocock
claude plugin update mattpocock-skills@mattpocock

### Claude Code VS Code Extension 

Run these commands in a Claude Code session (within the VS Code extension):

> /plugin marketplace add mattpocock/skills  
> /plugin install mattpocock-skills@mattpocock

Note: Matt’s skills are also available in the official Claude marketplace, but installing from there currently fails for some users.

Improvements and new skills are added continuously, so periodic updates are required to obtain the latest versions.

To update Matt’s skills, execute the following commands within a session:

> /plugin marketplace update mattpocock  
> /plugin update mattpocock-skills@mattpocock

The only change from your Terminal section is swapping `claude plugin ...` for the `/plugin ...`

### Codex Terminal

Run the below commands:

> codex plugin marketplace add mattpocock/skills
codex plugin add mattpocock-skills@mattpocock

### Installing Grilling Skill Alone

The commands above install the full plugin, which includes every skill in Matt’s repository. When only one skill is needed, such as `grilling`, it can be installed on its own with the GitHub CLI (`gh`) or with `npx`.

> gh skill install mattpocock/skills grilling --agent claude-code --scope user

`--agent claude-code` installs the skill for Claude Code, and `--scope user` makes it available in every project.

### [Claude.ai](http://Claude.ai) Browser Window

Developers usually use IDEs or VS Code extensions. Only non-developers may need this option of adding skills in the <http://claude.ai>  browser window. In IDEs, it’s enough to add a plugin, which brings in all the skills. In the browser, however, each skill needs to be added one at a time.

**This section shows how to add the /grilling skill.**

Click the user initials icon at the bottom left. In the popup context menu, click on `Settings`.

Click on `Skills` link under `Customize`, click on `add` button to add a new skill. 

  
Click on Add button dropdown and choose, Create a skill:

The name, description and contents of the skill can be copy pasted from the below skill:  
<https://raw.githubusercontent.com/mattpocock/skills/refs/heads/main/skills/productivity/grilling/SKILL.md>.  
  
After copy paste the screen looks like the below:

  Clicking on `Create` button creates the skill.

After creating the skill, the skill can be used as shown below:

While typing the name /grilling, all the matching skills are shown.

### [Claude.ai](http://Claude.ai) Browser Window - Adding Atlassian Rovo Connector

Click the user initials icon at the bottom left. In the popup context menu, click on `Settings`.

Click on `Connectors` link under `Customize`, click on `add` button to add a new Connector. 

 In the window opened, click on Connect button beside `Atlassian Rovo`.


In the browser, choose `Company-Atlassian-Domain`


Choose the permissions and click on `Accept` button.

### [Claude.ai](http://Claude.ai) Browser Window - Adding GitHub or Slack Connectors

This step is similar to adding the Atlassian Rovo Connector.  
The Atlassian Rovo, GitHub Integration, and possibly Slack connectors could be added to make the setup more useful for the product team.

## How Product Teams Can Use Grilling Skill

A product requirement document can be reviewed before it reaches engineering. The interview can be used to cover missing cases and replace ambiguous phrases with specific decisions.

Expect the Grilling skill to cover questions like:

- Who can use the feature?
- What happens in the normal case?
- What happens when a step fails?
- What are the unusual or boundary cases?
- What is out of scope?
- How will success be measured?

**Note:** If a prompt in the https://claude.ai  browser window refers to a Confluence page, make sure the Atlassian Rovo connector is added first. If it refers to a GitHub URL, add the GitHub connector instead. Both are described in the earlier sections.

### Example: From a Vague Idea to Feature Requirement Document

**Grilling works even when there is no requirement yet, only an idea.** A vague idea produces many questions, and that is where the skill is most valuable. Each round turns something loose into a decision, and the idea gets sharper as the session goes on.

#### The Idea

The company already has two features:

1. **A credit report**, generated for each customer.
2. **A score**, calculated from the customer’s details, spending patterns, and income.

A product owner wants to combine the two into a new feature: a report that helps lenders make better lending decisions. That is the whole idea. Nothing is written down yet.

#### The Prompt

> */grilling on a new feature idea. We have two existing features: a credit report generated for each customer, and a score calculated from the customer’s details, spending patterns, and income. They are described in \<credit report Confluence URL\> and \<scoring Confluence URL\>. I want to combine them into a new report that helps lenders make better lending decisions. The idea is still vague. Read both pages first and do not ask me anything they already answer. When we have reached a shared understanding and I confirm, create a new Confluence page titled “Lender Decision Report – Feature Idea” with: Problem, Target users, What the report contains, Decisions agreed, Out of scope, Open questions, Terms.*

#### Sample First-Round Questions from the Grilling Skill

The first round asks only the questions that do not depend on anything else:

```
❓ Q1 - Who is the lender user: Is the report read by a loan officer reviewing one
   application, or used by a lending system that decides automatically?

➡️ Recommended: A loan officer first. Automated decisions bring stricter rules
   on explaining every outcome.

❓ Q2 - What "better decisions" means: How will success be measured — fewer loan
   defaults, faster approvals, or more approvals for thin-file customers?

➡️ Recommended: Fewer defaults at the same approval rate. It is measurable and is
   what lenders already track.
```

Further questions in the same round might be:

- **Q3:** Has the customer agreed to share spending and income data with lenders, or does the current consent only cover the company’s own score?
- **Q4:** Must every score in the report be explained in plain terms, so a lender can tell a declined customer why?
- **Q5:** Is the report delivered as a PDF, a screen in a lender portal, or through an API?

### Example: Reviewing a Requirements Document

A requirements document can look complete and still have gaps: cases unhandled, wording that can be understood in different ways, rules that contradict each other, and assumptions that were never written down. **A grilling session finds them before the document reaches engineering.**

The scope should be stated clearly. Left alone, the agent may also ask about technical choices, such as which library to use or how data is stored. Those questions cannot be answered by the product team, and they belong to a later stage, when developers make implementation decisions.

When the requirements are in a Confluence page:

> */grilling on the requirements in \<Confluence page URL\>. Find the gaps, such as cases nobody handled, wording that can be understood in different ways, rules that contradict each other, assumptions that were never written down, and anything else that is unclear. Do not ask any questions about implementation choices, such as libraries, frameworks, or technical design. Those are decided at a later stage.*

**Note:** The Atlassian Rovo Connector is required for the prompt above.

Once all the questions asked by the Grilling skill have been answered, the decisions made in the session can be saved on a new Confluence page. A sample prompt is given below:

> The common understanding reached in the above discussion should be saved on a Confluence page, under the document found at this URL:  
> [Sample Updated Requirements Document with Grilling Skill](<Confluence-URL-of-Initial-Requirement-Document>)

Note: The above prompt asks for permissions to save the updated document in a Confluence page.


When the requirements are in a Markdown file, the file is first attached to the message in <http://claude.ai> , with the **+** button or by dragging it into the message box:

> */grilling Review the requirements in the attached file. Find the gaps, such as cases nobody handled, wording that can be understood in different ways, rules that contradict each other, assumptions that were never written down, and anything else that is unclear. Do not ask any questions about implementation choices, such as libraries, frameworks, or technical design. Those are decided at a later stage.*

## **How Engineering Teams Can Use the Grilling Skill**

For engineering teams, the full plugin is the best way to install Matt’s skills, because it brings in every skill at once. **Grilling is often the first skill a developer uses**: before any code is written, the change is agreed on.

Developers usually call a coding agent from an IDE plugin or the VS Code extension. Each coding agent, such as Claude Code, Copilot, or Codex, installs plugins and skills in its own way. The steps for Claude Code and Codex are given in the Installation section.

### Calling a Skill by Its Full Name

When skills come from a plugin, a skill can be called by its full name: the plugin name, a colon, and the skill name.

> */mattpocock-skills:grilling*  
> */mattpocock-skills:grill-with-docs*

The prefix `mattpocock-skills:` says which plugin the skill comes from. This matters when two plugins have a skill with the same name. For example, both Matt Pocock’s skills and Lauren Tan’s Pstack have a skill called `tdd`:

> */mattpocock-skills:tdd* → Matt’s `tdd` skill  
> */pstack:tdd* → Lauren Tan’s `tdd` skill

### Small Changes: Grilling From the Prompt

For a quick fix, no requirements file is needed. The changes are described directly in the prompt, along with the grilling skill:

> */mattpocock-skills:grilling Add a retry to the payment status check when the API times out.*

`/mattpocock-skills:grill-with-docs` can be used instead when the change adds new terms to the project or involves a decision worth recording.

### Larger Features: Grilling a Requirements File

When the feature is not small, the requirement is first written to a Markdown file in the Git repository, for example:

```
<Git-Repo-Root>/.scratch/misc-features/aws-sdk-custom-metric.md
```

**As much detail as possible should go into this file, both functional and non-functional:** what the feature does, as well as limits, performance, security, and error handling. The more detail the file has, the fewer questions the session needs.

The file is then grilled:

> */mattpocock-skills:grilling Review the requirements in @.scratch/misc-features/aws-sdk-custom-metric.md. Find the gaps, such as edge cases, wording that can be understood in different ways, rules that contradict each other, and assumptions that were never written down. Also ask about design choices and implementation options.*

When all the questions have been answered, the skill gives a summary of the shared understanding.

### Grilling With Docs: Keeping a Record of the Session

`/mattpocock-skills:grill-with-docs` runs the same interview, but it also identifies domain terms, defines them, and keeps a record of what is agreed. It is the better choice when the requirements or the code use domain terms that are worth writing down. Once a term is defined, future requirements and grilling sessions can simply use it, without explaining it again in several sentences.

The prompt is the same; only the skill changes:

> */mattpocock-skills:grill-with-docs Review the requirements in @.scratch/misc-features/aws-sdk-custom-metric.md. Find the gaps, such as edge cases, wording that can be understood in different ways, rules that contradict each other, and assumptions that were never written down. Also ask about design choices and implementation options.*

**Domain terms are added to **`GLOSSARY.md`, each with a clear definition. For example:

**Thin-file customer**: A customer with fewer than three credit accounts on file.

**Important decisions are recorded as ADRs** (Architecture Decision Records), short notes that explain a decision and the reason for it.

### After Grilling: Choosing the Next Skill

The shared understanding is then turned into work with one of three skills:

| **Skill** | **What it does** | **Best when** |
| --- | --- | --- |
| `/mattpocock-skills:to-spec` | Writes a specification file | The session started from a vague idea |
| `/mattpocock-skills:to-tickets` | Breaks the work into tickets | The session started from a detailed requirements file |
| `/mattpocock-skills:implement` | Builds the change straight away | The feature is very small |

### `to-spec`: From Shared Understanding to a Specification

`to-spec` writes the shared understanding to a specification file in Markdown. **It is the natural next step when a session starts from a vague idea**, because the spec captures every decision made along the way.

### `to-tickets`: From Shared Understanding to Tickets

`to-tickets` breaks the work into tickets, ready to be implemented. **It is the natural next step when the session starts from a reasonably detailed requirements file.** Tickets can be created in GitHub Issues, Linear, and other trackers, but the simplest choice is to keep them as files inside the Git repository.

The proposed tickets can be discussed with the agent before they are created. Small tickets can be combined, and large ones split. Every ticket carries the shared understanding between the developer and the AI.

Each ticket is then implemented in a **new session**:

> */mattpocock-skills:implement the ticket “01: Custom metric publisher”.*

### `implement`: Building the Change

`implement` builds the change using proven practices. Code is written test-first (test-driven development), classic software design principles are applied, and at the end, a separate sub-agent checks the code against those practices and against the original requirement.

When `implement` is run in the same session as the grilling, the grilling discussion and the implementation share one session’s tokens. **This suits very small features only.** For anything larger, the work should go through `to-tickets` first, with each ticket implemented in its own session.

### Keeping Sessions Within Token Limits

Every session has a limit on how much it can hold, measured in tokens. A single ticket of moderate complexity can use above 100K of a 200K-token limit during implementation. **As a session fills up, the model becomes slower and makes more mistakes.**

For this reason, tickets should be kept small enough that each one can be implemented comfortably in one fresh session. The usual order is:

> *grilling → to-tickets → implement (one new session per ticket)*   

### Sample Grilling Questions 

The agent was asked to grill an idea: whether batching messages sent to AWS Firehose would reduce latency and increase throughput. A few of the questions asked are shown below.

The prompt used:

> /mattpocock-skills:grilling Grill me on whether batching messages sent to AWS Firehose would reduce latency and increase throughput.

Below are a few questions asked in the grilling session:

❓ **Q1** - **Which latency are we optimising?** Per-request latency as the orchestrator sees it, or wall-clock time for the 11M-message batch job? Batching can only help the second, and only if the limit is requests per second or connections, not MB/s.

➡️ Batch job wall-clock time. Per-request latency only matters as far as it limits throughput.

---

❓ **Q2** - **What's the current limit for the batch job?** I need numbers from your metrics, which I can't get from the repo:

- How many dispatcher tasks run, and what aggregate msg/s (or MB/s) do you see on Firehose?
- Do you see `ThrottlingException` or `ServiceUnavailableException` from Firehose?
- "100 concurrent requests with publishing latency of 500 ms": is 500 ms the end-to-end request time? At 60–80 ms publish, where do the other \~420 ms go?

➡️ Work out aggregate MB/s against 80 MB/s first. If you're near the quota, drop batching and request a higher quota. If you're far below it and `publishDurationTimeMs` grows with concurrency, the limit is on the client side (connection pool or CPU) and needs a closer look.

---

❓ **Q3** - **What does the 80 MB/s quota cover?** Firehose quotas apply per delivery stream. Does 80 MB/s apply to `firehose-external` (prod) only, or to each stream? In non-prod, LOW writes the same payload to both internal and external.

➡️ Confirm it's the external stream in prod, since that's the one that matters for the 11M job. 

## Getting the Most Out of a Grilling Session

### Questions Come in Rounds

By default, the skill asks a **group of related questions at a time**. Each round contains only the questions that can be answered now; questions that depend on those answers are held for the next round. A whole round can be answered in one reply, by number.

Some readers prefer a slower pace, especially when the topic is new or the questions are long. One question at a time can be requested in the prompt:

> *Ask one question at a time.*

In Claude Code, this can be made the default by adding the following line to `CLAUDE.md`:

```
When grilling, ask one question at a time.
```

### Vague Ideas Produce Many Questions

The number of questions depends on how vague the starting point is. A clear requirement may need ten questions; **a vague idea can produce more than 50, which can feel overwhelming.** This is the skill doing its job, not a fault.

Long sessions are easier when the scope is narrowed first. A large feature can be split into phases, and one phase grilled at a time:

> *Ask questions only about Phase 1. Do not ask about later phases yet.*

A session running to hundreds of questions is a sign that the scope is too large, not a sign of thoroughness.

### Steering the Session

The skill’s questions are a starting point, not a script. The session can be steered at any time, in plain language.

When an area seems under-explored, more questions can be requested:

> *Ask more questions about error cases and what happens when a payment fails.*

When the suggested choices feel too narrow, different options can be requested:

> *Give more options for Q3, with the pros and cons of each.*

When a question cannot be answered yet, it can be set aside:

> *Skip Q4 for now. Add it to the open questions and come back to it at the end.*

When an earlier answer turns out to be wrong, it can be changed:

> *Change my answer to Q2. Partial cancellations are allowed. Recheck any questions that depended on it.*

### Doing the Research Outside the Session

Some questions cannot be answered on the spot: a competitor’s approach has to be looked up, a regulation checked, or several designs compared. **That research is best done outside the grilling session.** Everything discussed in a session is carried forward with every new message, so a long side investigation uses up tokens and crowds the grilling conversation with detail it does not need.

The research can be done in a new session or, in Claude Code, by forking the current session with `/fork`, so the research starts with the full context. The answer is then brought back to the grilling session, and the session continues, focused.

## Summary

A requirement can be carefully written and still leave decisions open. Grilling makes those open decisions visible through a structured interview before implementation begins.

For a repository-based design session, `/grill-with-docs` can also preserve shared terms and important decisions in project files. For product discussions without a repository, `/grilling` provides the interview. In both cases, the key result is a confirmed understanding that can be carried into a specification, tickets, or a new Confluence page.

## References

**Matt’s AI Skills Git Repo**  
<https://github.com/mattpocock/skills>

**The Grilling Skill**  
<https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md>

**How Grilling Works**  
<https://github.com/mattpocock/skills/blob/main/docs/productivity/grilling.md>

**Proven Engineering Practices, Applied to AI: Matt Pocock’s Skills**  
<https://sri-chalam.medium.com/proven-engineering-practices-applied-to-ai-matt-pococks-skills-51e132c546aa>
