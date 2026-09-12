# Proven Engineering Practices, Applied to AI: Matt Pocock's Skills

# Introduction

Matt Pocock's agent skills have struck a nerve: **the repository has passed 250,000 GitHub stars and 21,000 forks** (September 2026) — a scale normally reached only by major frameworks and language runtimes, and extraordinary for a project that ships no code at all, only Markdown. 

There is no shortage of AI development workflows: Amazon's AI-DLC and Kiro, GitHub's Spec Kit, BMAD, GSD and others. Most concern themselves with process — the stages a change passes through, the roles involved, the documents produced at each step.  

**Matt’s skills start somewhere else. Each is a short set of instructions that brings a practice engineering already knows works — a design interview, a shared glossary, red-green TDD, deep modules, tracer bullets, two-axis code review** — into the way a coding agent works. The claim is not a better process. It is that the fundamentals matter more now, not less, because code arrives faster than anyone can review it.

This article covers what a good AI workflow needs, how these skills provide it, and how they fit together in practice.

# What features should an AI Workflow have?

## Context Window - Dumb Zone - Smart Zone

Dex Horthy of HumanLayer describes a coding session as having two zones. Early on, while the context window is still lightly loaded, the model is in the **smart zone**: attention is focused, recall is reliable, and output is usable. As the window fills, the session crosses into the **dumb zone**, **where quality degrades sharply: instructions given earlier are quietly ignored, work already done is repeated, the agent loops instead of converging, its tool calls start coming out malformed, and the code it produces gets worse**. Cost moves the other way — every turn re-sends the whole conversation, so the tokens spent per step keep rising as the usefulness of each step falls.

The boundary arrives earlier than most developers expect. Horthy puts diminishing returns at roughly **60% of the context window** — around 120K tokens on a 200K-token model such as Sonnet — while stressing that this is a heuristic, not a hard limit: it moves with the model and with the complexity of the task.

A workflow's first job, then, is to keep sessions inside the smart zone. That gives a short list of things it must provide:

1. **A unit of work smaller than a feature.** A single ticket of moderate complexity can consume 100K tokens on its own, so the workflow must break work into pieces each sized to fit one fresh context window.
2. **A way to keep the working context small.** Expensive investigation belongs in sub-agents that read widely and return only their findings.
3. **A defined point at which the session ends.** One ticket, one session — with the context cleared between tickets rather than carried forward.
4. **A way to hand off rather than continue.** When work spans more than one window, what matters must be compressed into a document that a fresh session starts from.

None of these are properties of the model. They are properties of the process — which is why the workflow, not the model, is what needs designing.

Matt skills answer this list directly: `/to-tickets` sizes the work, sub-agents keep the context small, one ticket per session bounds it, and `/handoff` carries what matters into the next one.

## Keep the Main Context Clean with Sub-Agents

Some work does not belong in the main session at all. **Research** must not bring back everything it read, and a **review** must not inherit the reasoning that produced the code. **A workflow should push such tasks to sub-agents, each working in its own context and returning only its findings**.

## Review the Design and the Plan, Not Just the Pull Request 

An agent will follow a plan faithfully, including a wrong one. Left to itself it commits to a direction early and then spends an entire session building in it.

A workflow must therefore **provide an explicit checkpoint the developer has to pass: the requirement questioned, the architecture and the modules affected agreed, the plan approved** — all before any code is written. Without such a point, the developer's judgement arrives too late to change anything, at pull-request review, when the only remaining options are accept or rework.

# Core Features of Matt Pocock’s Skills

The underlying point is simple. **Software engineering fundamentals have not become less important now that AI writes the code. They have become more important, because code now arrives faster than the design can drift**.

## The AI and the Developer Share the Same Understanding

**The most common reason software goes wrong is not bad code. It is misalignment.** The developer is assumed to have understood the requirement. Then the result is seen, and it is not what was meant.

Working with AI is no different. **A communication gap exists between the developer and the model. **Even a carefully written spec has loose ends, and those gaps are quietly filled by the model with its own assumptions. The code compiles, the tests pass — and the result is still wrong.

The fix is a **grilling session**: before any code is written, the AI interviews the developer.

The `/grill-with-docs` skill treats the design as a tree of decisions. Each round asks every question that can be answered *now*, numbered, each with a recommended answer; the answers unlock the next round. The session ends only when no branch is left unvisited and nothing is silently assumed.

Two details make this work well in practice:

- **Facts are found by the AI; decisions are left to the developer.** If a question can be answered by reading the codebase, the codebase is read instead of the developer being asked.
- **No work is started until the developer confirms** that a shared understanding has been reached.

A vague idea produces many rounds of questions. A well-considered plan produces few. In either case, the change is better understood at the end of the session than at the start.

**Example Prompt**

> /grill-with-docs Review the requirements in @docs/requirements/feature-requirements.md.
> 
> I want to implement them iteratively. For now, ask questions only about Phase 1 under "Phased Implementation." Do not ask about later phases yet.
> 
> Ask one question at a time.

## The AI and the Developer Use the Same Terminology

Every team has its own business vocabulary. Domain-Driven Design calls this the ***ubiquitous language***: one set of terms shared by the business, the developers, and the code.

An AI agent has none of it. It is dropped into a repository and left to guess the jargon. So twenty words are used where one would do, and every conversation becomes long and imprecise.

The fix is a shared glossary, kept in a `CONTEXT.md` file at the root of the repository.

The `/domain-modeling` skill (invoked by `/grill-with-docs`) builds and maintains `CONTEXT.md` file. The codebase is read, the terms in use are put to the developer, and each definition is recorded as it is settled. A pointer to `CONTEXT.md` is then added to `AGENTS.md`, so every later session speaks and understands the same business language that is usually present in the Jira issues.

**Example Prompt**

The skill /grill-with-docs invokes /grilling and /domain-modeling skills. So the same prompt in previous example is used to create `CONTEXT.md`.

> /grill-with-docs Review the requirements in @docs/requirements/feature-requirements.md.
> 
> I want to implement them iteratively. For now, ask questions only about Phase 1 under "Phased Implementation." Do not ask about later phases yet.
> 
> Ask one question at a time.

## Feedback Loops: Test-Driven Development

> "Always take small, deliberate steps. The rate of feedback is your speed limit. Never take on a task that's too big." — David Thomas & Andrew Hunt, *The Pragmatic Programmer*

Without feedback on how its code actually behaves, the agent is flying blind. Good feedback comes from automated tests.

Tests offer the agent the most feedback, because a failing test is unambiguous — it states exactly what is wrong, in a form that can be acted on.

The `/tdd` skill drives a strict red → green loop:

1. A failing test is written.
2. Only enough code to make it pass is written.
3. The cycle repeats.

One term is needed first. A **seam** is the place where a module's public interface sits — the point at which its behavior can be observed and swapped without reaching inside it. Tests belong at seams, never against internals.

The discipline enforced around that loop is what makes it real, not just an idea:

- **Small steps.** One seam, one test, one minimal implementation per cycle. No code is written for tests that do not yet exist.
- **Vertical slices, not horizontal ones.** An agent's instinct is to build each layer in isolation: asked for a database service, it produces every endpoint, the request models, error middleware, auth, rate limiting and logging — and only then tries to connect to the database, where the connection string turns out to be wrong. A **tracer bullet** is the opposite: one thin slice cut through every layer at once, tested immediately, so the critical path is proven before anything is widened.
- **Behavior is tested, not implementation.** Tests are placed at a **seam**. A good test reads like a specification and survives refactoring.
- **Seams are settled before any test exists.** The agent proposes them and the developer confirms — never discovered along the way.

**Example Prompt**

The skill /implement invokes /tdd and /codebase-design skills. 

> /implement the ticket "01: Phase 1 — Insurance policy creation" under Phase 3. Do not commit the changes. I want to review the changes before commit. 

## Design the Interface, Delegate the Implementation

> "The best modules are deep. They allow a lot of functionality to be accessed through a simple interface."
— John Ousterhout, *A Philosophy of Software Design*

One point is easy to miss: **codebases that are easy to test are also easy for an AI to work in.** Humans and agents need the same thing — a small surface to learn, and a clear place to stand and observe behavior.

Module design turns on a single measure: **depth** — how much behaviour sits behind an interface.

- A **deep** module places a lot of behaviour behind a small interface. Complexity is hidden. Little must be learned by a caller — or by a test — to gain a lot.
- A **shallow** module has an interface almost as complicated as its implementation.

Deep is the goal; shallow is what to avoid.

`/codebase-design` gives the agent these principles while code is being designed; `/to-spec` applies them before any code is written, settling the modules touched and the seams tested.

## Avoid Building Too Much in One Go

> "Always take small, deliberate steps. The rate of feedback is your speed limit."
— David Thomas & Andrew Hunt, *The Pragmatic Programmer*

An agent will happily accept a task far too large for it. Thousands of lines can be produced, and which part is wrong will be difficult to tell. The rule from *The Pragmatic Programmer* applies directly: a task that is too big should never be taken on.

Two skills keep the work small at two scales: `/tdd` inside a slice, where a single failing test bounds each cycle, and `/to-tickets` across slices.

`/to-tickets` runs before any code is written. A plan, spec or conversation is broken into **tracer-bullet tickets**:

- Each ticket cuts a narrow but **complete** path through every layer — schema, API, UI, tests. It is a vertical slice, not one horizontal layer.
- Each ticket is demoable or verifiable on its own.
- Each ticket is sized to fit in a single fresh context window.
- Each ticket declares what **blocks** it, so the order of work is explicit.

**Example Prompt**

> /to-tickets Generate tickets for the changes discussed above. 

## Build Code According to Good Design Practices

> "Invest in the design of the system *every day*."
— Kent Beck, *Extreme Programming Explained*

Agents make coding dramatically faster. As a result, **software entropy** is also accelerated. A codebase can turn into a ball of mud in weeks rather than years, simply because more code arrives per day.

Design therefore cannot be something done at the start of a project and then forgotten. It must be continuous, and it must be part of the workflow rather than a separate clean-up phase.

The skills provided by Matt such as /tdd, /implement, /codebase-design, /to-tickets, /to-spec make sure that the good design principles are followed throughout.   

## A Code Review That Checks the Spec, Not Just the Code

**Most code-review prompts available online do general programming language code review: they inspect the code as code. Naming, duplication, error handling, null pointer exceptions**. What they never ask is whether the code does what was actually requested. An agent can produce a clean, idiomatic, well-tested implementation of the wrong feature, and a review of that kind will pass it.

`/code-review` reviews the change along **two axes**, and keeps them apart:

- **Spec** — does the diff faithfully implement the originating issue? Missing requirements, unrequested behaviour, and wrongly implemented ones are reported, each quoting the spec line it came from.
- **Standards** — does the diff follow the repository's documented coding standards, or, where none exist, a baseline of Fowler code smells? Documented standards override the baseline.

**Both axes run as separate sub-agents, in parallel. Each starts on a clean context** — carrying neither the reasoning that produced the code nor the other axis's findings — so the** review is a fresh pair of eyes** rather than the author marking its own work.

**Example Prompt**

> `/code-review` is invoked automatically at the end of `/implement`, and can also be run directly:

## Adapt the Skills, Don't Be Tied to a Process

Most AI development frameworks own the process: a fixed lifecycle, prescribed roles, and mandatory stages. That helps until something goes wrong inside the process itself, at which point there is little to reach for. 

These skills are the opposite. Each is a short Markdown file that can be read in a minute, edited in place, or ignored entirely — and they compose without chaining, so no step is a precondition for the next. A workflow that misbehaves can be fixed rather than worked around.

## More Than a Coding Workflow

The skills are also worth reading as examples of the form. Each is a short Markdown file that states its discipline and stops — no preamble, no restating the obvious to the model. Teams writing their own skills learn more from these than from any guide, and \`writing-for-agents\` sets out the principles behind them explicitly. 

The set also reaches past code. `/teach` runs a multi-session lesson using the working directory as a stateful workspace; `/handoff` compacts a conversation so another agent can continue it; 

`/writing-for-agents` covers how to write documents an agent will read — skills, `AGENTS.md`, `CLAUDE.md`, and anything reached by a pointer — so a draft skill can be written against its principles, or handed to the agent to be checked against them.

# Skills

## User Invoked Vs Model Invoked Skills 

The skills split on one axis: who can invoke them. **User-invoked** skills — `/grill-with-docs`, `/to-spec`, `/to-tickets`, `/implement` — are typed by the developer and orchestrate a stage of work. **Model-invoked** skills — `/tdd`, `/codebase-design`, `/domain-modeling`, `/code-review` — hold reusable discipline and are reached for automatically when the task fits, which is why they rarely need naming.

## Important Workflow Skills

### **/setup-matt-pocock-skills** 

Run once per repository, before anything else. Asks which issue tracker to use, which labels triage applies, and where generated docs should live. 

### /grill-with-docs

A relentless design interview that also builds the project's shared language. Combines \`/grilling\` and \`/domain-modeling\`: questions are asked in rounds until no branch of the design is left unresolved, while \`CONTEXT.md\` and ADRs are updated as decisions settle.  

### /domain-modeling

Builds and sharpens the project's glossary. Terms that clash are challenged, fuzzy words are made precise, and each definition is written to \`CONTEXT.md\` as it is agreed. Normally reached through \`/grill-with-docs\`.  

### /to-spec

Turns the current conversation into a spec and publishes it to the issue tracker. No new interview: it synthesises what has already been discussed, and confirms which modules and seams the change will touch.  

### /to-tickets

Breaks a plan, spec or conversation into tracer-bullet tickets — vertical slices, each verifiable on its own and sized to fit one fresh context window — with the blocking order between them made explicit.  

### /implement

Builds the work described by a spec or ticket, driving \`/tdd\` at the agreed seams, running type checks and tests as it goes, and closing with \`/code-review\` before committing.  

### /tdd

Red-green-refactor, one vertical slice at a time. Defines what a good test is, where tests belong, and which anti-patterns to reject.  

### /codebase-design

The shared vocabulary for designing deep modules: a lot of behaviour behind a small interface, placed at a clean seam and testable through it.  

### /code-review

Reviews the diff since a fixed point along two independent axes — Spec and Standards — each in its own sub-agent, and reports them side by side.  

### /diagnosing-bugs

A disciplined loop for hard bugs and performance regressions, gated phase by phase: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → add a regression test. 

## Other Useful Skills

### /grill-me

The same relentless design interview as `/grill-with-docs`, without the documentation step. The design is treated as a tree of decisions and worked in rounds: every question that can be answered now is asked together, each with a recommended answer, and the session ends only when no branch is left unvisited.

Four things make it worth knowing separately: 

- **It writes nothing**. No \`CONTEXT.md\` entries, no ADRs. The output is a shared understanding, not a document — so it can be used where there is no repository to write to. 
- **It is tiny.** The skill file does one thing: invoke the underlying \`grilling\` interview. That same primitive sits behind \`/grill-with-docs\`, \`/triage\`, \`/wayfinder\` and \`/improve-codebase-architecture\`. 
- **It is not for developers only.** Any plan, decision or specification can be grilled. A product owner can use it to find the gaps and uncovered cases in a requirements document before it reaches engineering — the questions surface what the document left implicit. 
- **It stands alone**. No repository, no issue tracker, no prior setup, and no other skill. It can be the only skill a person ever uses and still be worth having.

### /handoff

Compacts the current conversation into a handoff document so a fresh session, or another agent, can continue the work without repeating the investigation. 

### /teach

Teaches a concept or skill across multiple sessions, using the working directory as a stateful teaching workspace. 

### /writing-for-agents

How to write documents an agent will read: skills, \`AGENTS.md\`, \`CLAUDE.md\`, and anything reached by a pointer. Useful both for drafting a skill and for having a draft checked against its principles.

# Typical Workflow

Most work follows the same three steps.

> /grill-with-docs → /to-tickets → /implement  
> align slice build

`/grill-with-docs` settles what is being built and in whose vocabulary. `/to-tickets` breaks the agreed change into vertical slices, each sized for one fresh context window. `/implement` then builds them one at a time, driving `/tdd` at the agreed seams and closing with `/code-review` before anything is committed. For larger work, `/to-spec` can be inserted after the grilling to publish a spec to the issue tracker first.

The sequence is a habit, not a rule. What matters is the order of concerns — align, then slice, then build — not the exact chain of commands.

**Most AI workflows share this broad shape but leave out its most valuable parts: no grilling session before code is written, no review in an isolated context, and no check that the finished code matches what was originally asked for.**

## Worked Example: One Feature, Three Prompts Workflow Prompts

Step 1: /grill-with-docs — reach a shared understanding: 

> /grill-with-docs See the requirements in @docs/requirements/feature-requirements.md.
These will be implemented iteratively. Ask questions only about the Phase 2a requirement
under "Reporting Pipeline Endpoint". Do not ask about the later phases yet, including 2b and 2c.
Ask one question at a time.

Step 2: /to-tickets — break it into a verifiable slice

> /to-tickets Generate a ticket for the changes discussed above.

Step 3: /implement — build it, review before committing

> /implement the ticket "01: Phase 2a — Reporting Pipeline Endpoint".
Do not commit the changes. They will be reviewed first.

# Installation of Matt’s Skills

## Claude Code

Terminal Command:

> claude plugins install mattpocock-skills

# What the Agent Missed, and Where to Watch

On a recent greenfield application, the results were strong. The modularisation and design were excellent, the generated code was clean, and the tests were meaningful rather than decorative. Most of the work needed no intervention at all.

In a minority of cases, though, the developer had to look closely.

> **Note:** The misses below are not attributable to Matt Pocock's skills. They reflect general model behaviour, or missing instructions in `AGENTS.md` and the team's own skills — and most were resolved by adding the instruction that was absent.

## Observed Gaps

- **An outdated version of a new dependency.** A library was added correctly, but pinned to an old version. Nothing in the code looked wrong; the developer had to check the version explicitly.
- **Field and type mismatches in generated classes.** A few classes declared fields whose types did not match the values actually assigned to them.
- **A silently lossy type mapping.** The Arrow type `dictionary<uint8,utf8>` was mapped to `VarCharVector`, which would have produced a `ClassCastException` at runtime.
- **Missing tests exactly where the risk was.** No tests covered the Arrow type mapping until this was pointed out, after which they were added.
- **An incomplete request payload.** Building the request payload object produced repeated errors: some fields were omitted, others were given the wrong type.
- **Logging absent at critical points.** Read operations and similar key steps were left without logging. The logging skill was then amended so that these points would not be missed again.
- **Detail buried in the log message.** The `eventData` was written into the `message` string rather than as structured fields. The code was corrected and the skill updated to prevent a recurrence.
- **Placeholder variable names.** Names such as `grouped` described the mechanics rather than the meaning. An instruction was added requiring names to reflect the business purpose of the value.
- **The same constant declared in several classes.** The agent was asked to extract them into a single constants class.
- **An implementation fitted to the first ticket only.** All field and property names followed the first data source. A second source, whose naming differed considerably, had not been allowed for — the instructions had never asked for a design that would accommodate it.

# References


**Design, Code Best Practices**
<https://www.youtube.com/watch?v=v4F1gFy-hqg>


**Matt’s AI Skill’s Git Repo**

<https://github.com/mattpocock/skills> 


**Mats AI Skills Documentation**

<https://github.com/mattpocock/skills/tree/main/docs/engineering> 
