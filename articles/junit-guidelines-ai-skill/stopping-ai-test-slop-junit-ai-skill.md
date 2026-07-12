# Stopping AI Test Slop: A JUnit Best-Practices AI Skill

AI coding agents write unit tests fast, confidently, and in bulk. None of that guarantees the tests are any good.

Ask an AI agent to "add some tests" for a Java class, and a wall of them is happily produced — dozens of `@Test` methods, all green, all compiling. It feels productive. But when they're actually read through, a lot of that code isn't testing anything meaningful.

This has come to be called "test slop": test code that compiles, passes, and adds almost no real verification value. It's a quiet problem, because it doesn't look like a problem. The build is green. Coverage numbers go up. It's only later, during a refactor, that it becomes clear which tests were real and which ones were just noise — and by then, time has already been spent chasing false alarms or, worse, a real regression has been missed because a test wasn't actually checking anything.

This is what the AI skill exists to fix — unit-testing best practices, put together as a set of rules a skill can follow: [**junit-guidelines**](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/SKILL.md).

**Using this skill, the unit tests generated are built on best practices, with the quality of an experienced developer's work — in a fraction of the time. It covers the full range of day-to-day needs:**

- Planning tests first — behaviors are enumerated and justified before any test code is written
- Generating tests for uncommitted changes or recent commits
- Filling gaps in existing tests
- Reviewing PRs for test coverage
- Surveying the codebase for a test-coverage report
- Finding tests that don't follow best practices
- A fresh-context subagent reviews the generated tests as an independent pair of eyes, and the skill then fixes any issues it flags and resolves compile errors — so the output isn't just fast, it's already validated

Ready-to-use prompts for each of these are in the *Common use cases* section below.

## Why AI-generated tests need explicit guidance

Writing unit tests is easy for an AI coding agent — too easy. **Without clear guidance, large volumes of tests get generated — many of them mocking more dependencies than necessary, verifying little of value, and ignoring best practices.**

Two of the most common failure modes sit at opposite extremes:

- **Over-mocked tests** silently fail to catch real regressions, because multiple dependencies are mocked that hardly any real code is exercised.
- **Implementation-detail tests** are tightly coupled to private internals, so they break noisily on every refactor even when observable behavior hasn't changed.

Both erode trust in the test suite, just from different sides — one through false confidence, the other through false alarms. A suite that can't be trusted is arguably worse than no suite at all. When the application is later refactored, a wall of low-value tests breaks alongside (or instead of) any real regressions, and a failing test can no longer be told apart from a brittle one.

The goal of this skill is simple: keep AI-generated tests few, meaningful, and genuinely useful during refactoring, rather than something to wade through.

There is a second benefit beyond individual test quality: **consistency at scale**. Training every developer on a team to apply the same naming convention, the same mock-vs-fake judgment, the same test structure — and to keep applying it months later, under deadline pressure, on someone else's code — is hard. A codified skill is applied the same way on every invocation, by every developer's AI agent, without relying on individual memory or discipline.

### Example: a "slop" test

```java
@ExtendWith(MockitoExtension.class)
public class OrderServieTest {
  @Mock private OrderClient orderClient;
  @Mock private CustomerService customerService;
  @Mock private SQSUtil sqsUtil;

  // Except the orderservice, every depedency of this service is mocked.
  private OrderService orderService;

  @BeforeEach
  void setup() {
    orderService = new OrderService(orderClient, customerService, sqsUtil);
  }

  @Test
  public void getOrdersOfCustomer_withValidRequest_returnsOrdersOfCustomers() {
    // Given
    Order expectedOrder = mock(Order.class);
    when(orderClient.getOrders(any())).thenReturn(List.of(expectedOrder));

    // When
    List<Order> orderList = orderService.getOrdersOfCustomre(customerId);

    // Then
    assertThat(orderList)
      .as("getOrdersOfCustomer should return exactly the orders belonging to customerId=%s", customerId)
      .containsExactly(expectedOrder); // the "expected" value is the stub itself
  }
}
```

This test passes no matter what `getOrders` actually does with `customerId` — `any()` accepts any argument, and `containsExactly(expectedOrder)` only confirms Mockito returned the exact mock the test just stubbed. The assertion is circular: the "expected" value was manufactured by the test itself. Nothing here verifies real behavior.

## What the skill actually does

`junit-guidelines` is an AI skill — think of it as a set of instructions an AI coding agent (Claude Code, in this case) loads automatically whenever it's about to plan, write, review, or modify a JUnit 5 test file, or audit production classes that can only be mocked, never faked, because they don't implement any interface. The agent isn't asked to invent its own testing philosophy; one is handed to it, with concrete rules and both good and bad examples.

A few of the ideas at the core of it:

- **FIRST principles** — tests should be Fast, Independent, Repeatable, Self-Validating, and Timely.
- **Behavior-driven naming** — a test name should read like `behavior_action_expectedResult`, so a failure communicates what broke without the file being opened.
- **Given-When-Then structure** — every test follows the same readable shape.
- **Mocks vs. fakes** — the skill is explicit about when a Mockito mock is fine and when a stateful dependency deserves an interface-based fake instead.
- **Never mock value objects, data classes, or pure in-process logic** — Money-style types, DTOs, dates, IDs, collections, mappers, and validators are constructed for real; only dependencies that cross a process boundary (databases, external services, message queues) get mocked or faked.
- **Logic-owning vs. orchestrating methods** — methods with real conditional logic are tested exhaustively; methods that just wire calls together get a representative test, not ten redundant ones.
- **"A test that cannot catch a real bug should not be written"** — if a method has no branching, no transformation, no error handling, and just forwards its arguments to a dependency, the test is skipped. It would only verify Mockito wiring, not application behavior.

In total, the skill covers 14 rules (Rule 0 through Rule 13), from planning down to descriptive failure messages, each with a rationale rather than just a directive.

A smaller rule worth calling out: any identifier, status or error code, or string reused across more than one test is expected to become a named `public static final` constant. It sounds minor, but it removes a whole category of "which magic string was this again" confusion later.

### No test code before a test plan

Planning before coding is a well-established best practice when working with AI agents, and it applies to tests just as much as to production code. That is why the skill's very first rule requires a test plan before any test code is generated. Without a test plan, generation defaults to a happy-path test per method; which tests are worth creating is never consciously decided at all.

Under this rule:

- Each public method of the class under test is classified first — logic-owning or orchestrating — and only then are tests enumerated: one scenario per branch, boundary, null/empty case, and error path for logic-owning methods; one or two representative wiring scenarios for orchestrating ones.
- Every planned test must survive the question *"what specific production bug would make this test fail?"* Any test with no plausible answer is dropped at the planning stage — before it exists, not after.
- The plan is presented as a simple table before code generation begins, so scope is decided up front rather than ad hoc mid-generation.

## A second pair of eyes: the validator subagent

The rules aren't the only valuable part — the last step matters just as much. Once tests are generated, the skill hands them off to a separate subagent, [`junit-validator`](https://github.com/sri-chalam/ai-tools/blob/main/agents/engineering/junit-guidelines/junit-validator.md), which reviews them in a completely fresh context.

That "fresh context" part matters. The same context that wrote the tests isn't well positioned to notice its own mistakes — its own naming and coverage decisions are already assumed to be correct. The validator has no memory of how the tests were written. The tests are read, the class under test is read, the guidelines are read, and a findings table is reported back. It's read-only — no code is ever edited by it; only what needs fixing is reported.

First, every rule in the guidelines is checked — including the mock-vs-fake judgment and Given-When-Then structure. On top of that, targeted checks run against each test method:

- **Missing coverage** — every branch, null check, and catch block of each logic-owning method is mapped to a test; unmapped branches are flagged.
- **Naming and name-vs-body match** — does the method name follow the convention, and does the body actually verify what the name claims? A test named `throwsIllegalArgumentException` that never asserts a thrown exception gets flagged.
- **What gets mocked** — mocked value objects or DTOs (which should be constructed for real) and mocked third-party clients like `S3Client` or `RestTemplate` are flagged as must-fix — instead of mocking the client itself, the class that calls the third-party client should implement an application-owned interface, and the mock goes on that interface.
- **Assertion strength** — circular tests are caught: a test whose only assertion checks that the result equals the exact value the test itself stubbed proves only that Mockito returns what it was told. Weak assertion-only tests (`hasSize`, `isNotNull`) are flagged too.

Two design choices keep the findings trustworthy. Each finding carries a severity (must-fix, should-fix, or nit), and every must-fix must cite and quote the exact guideline line being enforced — a finding that can't point to a specific rule is downgraded or dropped, so the report stays grounded in the guidelines rather than the validator's opinion. And if the validator can't read the test files or resolve the guidelines, it reports "not run" loudly instead of returning an empty table that would look like a clean pass.

The skill takes that report back, fixes whatever the validator flagged, and resolves any compile errors the changes introduce — so the fresh pair of eyes isn't just advisory, it's acted on before the work is considered done.

## What humans must still decide

The skill judges whether a test is *well-formed* — verifying real behavior instead of mock wiring, making meaningful assertions, correctly named, testing behavior instead of implementation, using the right kind of mock or fake. It cannot decide *what's worth testing* in the first place. That's a strategy question, and it depends on things no class file can reveal:

- **Business risk** — which paths are catastrophic if broken vs. mildly annoying. The skill can't know that a bug in one method costs far more than a bug in another.
- **Production failure history** — the most valuable edge cases often come from past incidents. A null field that caused an outage six months ago isn't visible in the code.
- **Integration contracts** — when integrating with certain partners, certain behaviors matter far more than others. What downstream consumers actually depend on isn't captured in the class file.
- **What inputs are actually possible** — production traffic patterns, client behavior, upstream data quality.
- **Acceptable risk thresholds** — a team building a payments system tests differently than one building a dashboard widget, even with identical code structure.

Strategy is about deciding what matters; the skill is about deciding whether the tests written are well-formed. **What to test — and why — should be decided by a human before the skill is invoked.** That context is then supplied in the prompt itself:

```bash
/junit-guidelines Using the JUnit guidelines, generate tests for @/path/to/PropertyService.java. confirmProperties is high-risk — a null field there caused a production incident — so cover that edge case and any adjacent null/boundary cases exhaustively. getProperties is low-risk and rarely changes; representative coverage there is enough.
```

The human's role doesn't end once tests are generated, either:

- **Generated tests must be reviewed, not trusted blindly** — anything that looks off deserves a second look before being accepted.
- **Project-specific improvements come from humans.** A project suffering from mock-heavy, brittle tests might decide to migrate toward interface-based fakes; the skill can find the candidates (see the prompts below), but the decision — and the knowledge of where the pain actually is — belongs to someone who knows the project.
- **When something incorrect slips through, it's worth asking why.** Why didn't the skill catch it, and what change to SKILL.md would prevent a repeat? Feeding that back into the guidelines is how the skill improves over time.

## Common use cases

This isn't a "run it once and forget it" tool. A few situations where it earns its keep, in day-to-day use — each with a sample Claude Code prompt that can be copied and adapted (swap the `/junit-guidelines` and `@file` syntax for plain English on other agents):

- **Generating tests for uncommitted changes**

  ```bash
  /junit-guidelines Using the JUnit guidelines, review my uncommitted changes (`git diff HEAD`) and generate or update JUnit tests covering the new/modified behavior.
  ```

- **Generating tests for the last few commits**

  ```bash
  /junit-guidelines Using the JUnit guidelines, review the changes introduced by the last two commits (`git diff HEAD~2 HEAD`) and generate or update JUnit tests covering the new/modified behavior.
  ```

- **Generating tests for a specific class**

  ```bash
  /junit-guidelines Using the JUnit guidelines, generate test cases for @/path/to/OrderService.java
  ```

- **Filling gaps in an existing test class**

  ```bash
  /junit-guidelines Using the JUnit guidelines, review @/path/to/OrderService.java and its existing tests in @/path/to/OrderServiceTest.java. Identify missing test coverage (conditional logic paths, edge cases, exceptions) and add the missing test methods.
  ```

- **Surveying the codebase for test-coverage ROI**

  ```bash
  /junit-guidelines Using the JUnit guidelines, explore the codebase and identify classes whose existing tests (or lack thereof) fall short of the guidelines. Report the top 10 classes that would benefit most from added test coverage, ranked by expected benefit, with a one-line reason for each (e.g. untested logic-owning methods, missing exception paths, no tests at all). Do not write any test code yet — just the report.
  ```

- **Helping a reviewer check a PR's test coverage** — before approving, the skill verifies that every new or changed piece of logic in the PR has appropriate unit tests, not just that some test files were touched. (Before running the skill, check out the PR branch with `gh pr checkout 123` so the agent sees complete files, not just the changed lines.)

  ```bash
  /junit-guidelines Using the JUnit guidelines, review this PR's changes (PR branch is checked out; `git diff main...HEAD`) and check whether every new or changed piece of logic has tests that actually exercise it. Report the gaps only; do not write any test code.
  ```

- **Finding test classes that should use fakes instead of mocks** (report only, no code changes)

  ```bash
  /junit-guidelines Using the JUnit guidelines, explore the codebase's test classes and identify those mocking stateful, complex external dependencies where an interface-based fake would be more appropriate. Report the top 10 test classes ranked by expected maintainability benefit, with a one-line reason for each (e.g. dependency has multi-step stateful behavior, mock setup is duplicated across many tests). Do not write any code yet — just the report.
  ```

- **Finding production classes that can only be mocked, never faked**, because they use a third-party client directly without implementing any interface (report only, no code changes)

  ```bash
  /junit-guidelines Using the JUnit guidelines, find production classes that use a third-party client (S3Client, RestTemplate, WebClient, etc.) directly and don't implement any interface — so a fake can't be swapped in for testing, and mocking is the only choice. Report the top 10, ranked by test-count impact, with a one-line reason each. Report only, no code changes.
  ```

- **Finding "slop" tests across the codebase** — tests that compile and pass but verify nothing meaningful (report only, no code changes)

  ```bash
  /junit-guidelines Using the JUnit guidelines, find "slop" tests — tests that only check the stub returned what it was told to return, weak-assertion-only tests. Report the top 10 offending test classes with a one-line reason each. Report only, no code changes.
  ```

- **Auditing an existing test class for rule violations and fixing them**

  ```bash
  /junit-guidelines Using the JUnit guidelines, review @/path/to/OrderServiceTest.java against every rule in the skill. Identify test methods that violate any rule (naming, logic-in-tests, mocking external deps vs. fakes, delegation testing, etc.), then rewrite each violating method to comply — do not just report violations.
  ```

## AI agent compatibility

This skill's guidelines are written for any AI coding agent — Claude Code, Codex, Copilot, and others. However, the final validation step relies on a subagent — a fresh-context reviewer that checks generated tests against the guidelines, fixes discrepancies, and resolves compile issues. A subagent is used so validation happens with a fresh pair of eyes rather than the same context that wrote the tests, but the subagent invocation mechanism itself is Claude Code-specific, and its instructions may not work in other AI coding agents. There does not appear to be a portable, cross-agent way to invoke subagents at this time. On other agents, test generation still follows the guidelines, and validation still runs — just in the same context that wrote the tests, rather than with a fresh pair of eyes.

## Getting set up

A naive setup — copying the skill files into `~/.claude/skills` and `~/.claude/agents` — creates a recurring chore: every time the guidelines are tweaked (and they change often, as edge cases surface and rules get refined), the copies would need to be redone by hand. Symbolic links avoid that: clone the repo once, link into place, and from then on a `git pull` plus restarting the AI coding agent is all it takes to pick up updates everywhere they're linked.

```bash
git clone https://github.com/sri-chalam/ai-tools.git
ln -s /path/to/ai-tools/skills/engineering/junit-guidelines ~/.claude/skills/junit-guidelines
ln -s /path/to/ai-tools/agents/engineering/junit-guidelines/junit-validator.md ~/.claude/agents/junit-validator.md
```

After that, the AI coding agent is restarted, and the skill is picked up automatically whenever something matching `src/test/**/*.java`, `**/*Test.java`, or `**/*Tests.java` is touched. Auto-loading can occasionally be skipped when the context doesn't clearly indicate test-related work, so the skill can also be invoked explicitly with `/junit-guidelines` — worth doing whenever it needs to be certain the skill is loaded rather than left to be inferred.

Full details, including the complete rule list and ready-to-use prompts, are in the [README](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/README.md).

## Project-specific conventions

This skill is generic and has no knowledge of a specific project's conventions — for example, whether `customerId` is a `UUID` or a `String`. Such details should live in a project-level `test-instructions.md` (or similar file dedicated to test conventions), not in this skill or in a catch-all CLAUDE.md/AGENTS.md, so the skill stays portable and project conventions stay easy to find as they grow.

## Known limitations

- For complex stateful dependencies, fakes are preferred over mocks — but a fake needs an interface to implement. When existing classes don't provide one, the skill deliberately falls back to mocks rather than refactoring production code just to enable a fake. In new code, where the interface can be designed in from the start, fakes are written instead.
- Fresh-context validation is Claude Code-only for now — no portable way to invoke a subagent on other agents exists yet, so validation there runs in the same context that wrote the tests.

## References

- [SKILL.md](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/SKILL.md) — the skill's full rule definitions, with good and bad examples for each rule.
- [README](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/README.md) — setup details and ready-to-use prompts.
- [junit-validator](https://github.com/sri-chalam/ai-tools/blob/main/agents/engineering/junit-guidelines/junit-validator.md) — the fresh-context validator subagent.
- Some of the testing best practices baked into the guidelines are inspired by [Software Engineering at Google](https://abseil.io/resources/swe-book), particularly its chapters on unit testing and test doubles.
- This skill was reviewed against [Writing Great Skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-great-skills), which argues a good skill stays small, avoids redundant instructions, and gives each real use case a single, unambiguous trigger.

## Closing thought

None of this is groundbreaking — it's mostly unit-testing advice that's existed for years, just written down explicitly enough that an AI agent can actually follow it. AI-generated tests aren't necessarily worse than human-written ones. They're worse *by default*, because nothing stops the agent from taking the path of least resistance: mock everything, assert something, move on. Writing the guidance down, and adding a second pass that checks the work with fresh eyes, has made the resulting tests noticeably more trustworthy — and that's really all this was aimed at.

This AI skill is a living document, not a finished one, and using it well means human involvement at every stage — before the tests are generated, while reviewing them, and when improving the skill itself:

- **What's worth testing should be decided before the skill runs** — business risk, past incidents, and partner contracts are strategy calls only a human can make.
- **Generated tests should be checked, not trusted blindly** — anything that looks off deserves a second look before being accepted.
- **When something incorrect slips through, it's worth asking why.** Why didn't the skill catch it, and what change to SKILL.md would prevent a repeat? That suggestion can then be fed back into the guidelines so the same mistake doesn't recur.
