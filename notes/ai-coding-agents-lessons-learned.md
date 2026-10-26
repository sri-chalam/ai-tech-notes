# AI Coding Agents: Lessons Learned

**🚧 Work in progress**: more points will be added and the content will be cleaned up.

## 📌 AGENTS.md: Every Line Must Earn Its Place

**AGENTS.md (or CLAUDE.md) is sent to the model with every prompt.** Each line uses context, even when the task has nothing to do with it.

- Keep the file **short**. Put only rules that help in **most tasks**.
- Models follow only about **150–200 instructions** reliably. The agent's own system prompt already uses some of that budget.
- **Do not add rules for rare cases** Extra rules add noise and can weaken the important ones.
- Add only what the agent **cannot learn from the code**, such as build and test commands, unusual conventions, and hard boundaries (for example, "never commit secrets").
- Ask for each line: *"If I remove this, will the agent make a mistake?"* If not, delete it.
- Move task-specific details to **separate files, skills, or path-scoped rules**. The agent then loads them only when needed.

## 🧠 Smart Zone vs Dumb Zone

- The model's **ability to reason goes down as the number of tokens in a session goes up**.
  - A short session is the *smart zone*. A long, crowded session is the *dumb zone*.
  - When a session uses **around 40–60% of the total allowed tokens**, it starts to enter the dumb zone, as a rule of thumb.
  - Dex Horthy calls this the **"dumb zone"**: the more context you add, the worse the model gets. Clear the context before it fills up.
- When a session **starts to enter the dumb zone**, use a **hand-off skill**. It works like the model's *compress* (compaction) feature.
  - It **summarizes the key points** of the current session. Then you can continue in a **new session** with a clean context.
- Do **not implement a big requirement in one session**. It uses **too many tokens** and pushes the session into the dumb zone.
  - Instead, **split the spec into smaller tickets**. **Implement each ticket separately**, in its own session.
- Start a **new session for each logically different question**.
- Do research in a **separate session or model**. Do not fill the working session with tangential conversation and pollute the session.

## ✅ Give the Agent a Way to Verify Its Work

- After a task is implemented, **give the model a way to check its own work**.
- Verification is **essential for software factories**.
- Many developers verify wrongly. They see that **the build passes** and decide the change is good. A passing build only shows that the code compiles. It does **not** show that the code works.
- Good verification is **not easy to add**. It usually needs **a significant number of tests**.
  - Add the tests first, so the model can run them after every change.
- Fix the failing tests, then **check the coverage**. Do the tests cover all the known and unexpected results.

## 🔀 Use the Right Model for Each Job

- Use a **stronger model (Opus) for planning**.
- Use a **faster, cheaper model (Sonnet or Haiku) for implementation**.

## 🤖 Use Sub-Agents

- Use a **sub-agent to review source code**.
- Use a **sub-agent for research**. The main session stays clean.

## 🔍 Code Review Is the New Bottleneck

AI writes code fast, so **reviewing it now takes the most time**.

- Find a balance:
  - Do **not review too deeply**. It slows everything down.
  - Do **not skip reviews completely**. Quality will suffer.
- Use **code review skills** to check the architecture, design, best practices, and security vulnerabilities.
- The code review skills should also **automatically check that the generated code matches the business requirements**. Code can be clean and well designed, and still **not do what was asked**.


## 🏗️ The Developer's Role Has Changed

- Developers now need **systems thinking**.
- Developers must **guide the AI**, not just accept what it produces. They need to guide:
  - The **architecture** and **design choices**
  - The **tests** and **test coverage**
  - The **quality of the code**
  - **Observability**: logs, metrics, and traces
  - **Reliability** under **unexpected conditions**

## Discuss Architecture/Design with the AI

- Share **what you plan to design**.
- Ask for the **pros and cons**.
- **Bounce ideas** back and forth.
- Ask for **alternatives**.
