# Stopping AI Test Slop: A JUnit Best-Practices AI Skill

AI coding agents write unit tests fast, confidently, and in bulk. None of that guarantees the tests are any good.

Ask an AI agent to "add some tests" for a Java class, and a wall of them is happily produced — dozens of `@Test` methods, all green, all compiling. It feels productive. But when they're actually read through, a lot of that code isn't testing anything meaningful.

This has come to be called "test slop": code that compiles, passes, and adds almost no real verification value. It's a quiet problem, because it doesn't look like a problem. The build is green. Coverage numbers go up. It's only later, during a refactor, that it becomes clear which tests were real and which ones were just noise — and by then, time has already been spent chasing false alarms or, worse, a real regression has been missed because a test wasn't actually checking anything.

Those rules were written down and turned into an AI skill: [**junit-guidelines**](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/SKILL.md).

**Using this skill, the unit tests generated are built on best practices, with the quality of an experienced developer's work — in a fraction of the time:**

- Applies whether tests are written from scratch, reviewed for quality, or checked for missing coverage
- A fresh-context subagent reviews the generated tests as an independent pair of eyes, and the skill then fixes any issues it flags and resolves compile errors — so the output isn't just fast, it's already validated

## Why AI-generated tests need explicit guidance

Writing unit tests is easy for an AI coding agent — too easy. **Without clear guidance, large volumes of tests get generated that mock everything and verify nothing meaningful.**

This tends to go wrong in two opposite directions at once:

- **Over-mocked tests** silently fail to catch real regressions, because everything the test touches has been faked out.
- **Implementation-detail tests** are tightly coupled to private internals, so they break noisily on every refactor even when observable behavior hasn't changed.

Both erode trust in the test suite, just from different sides — one through false confidence, the other through false alarms. A suite that can't be trusted is arguably worse than no suite at all. When the application is later refactored, a wall of low-value tests breaks alongside (or instead of) any real regressions, and it can no longer be told whether a failing test means "something broke" or "this test was brittle."

The goal of this skill is simple: keep AI-generated tests few, meaningful, and genuinely useful during refactoring, rather than something to wade through.

### Example: A "Slop" Test

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
    when(orderClient.getOrders(any())).thenReturn(List.of(mock(Order.class), mock(Order.class)));

    // When
    List<Order> orderList = orderService.getOrdersOfCustomre(customerId);

    // Then
    assertThat(orderList)
      .as("getOrdersOfCustomer should return exactly the orders belonging to customerId=%s", customerId)
      .hasSize(2); // verifies the mock's arity, not real behavior
  }
}
```

This test passes no matter what `getOrders` actually does with `customerId` — `any()` accepts any argument, and `hasSize(2)` just confirms the mock returned two mock objects. Nothing here verifies real behavior.

## Common Use Cases

This isn't a "run it once and forget it" tool. A few situations where it earns its keep, in day-to-day use — each with a sample Claude Code prompt that can be copied and adapted (swap the `/junit-guidelines` and `@file` syntax for plain English on other agents):

- **Generating tests for uncommitted changes**

  ```bash
  /junit-guidelines Using the JUnit guidelines, review my uncommitted changes (`git diff HEAD`) and generate or update JUnit tests covering the new/modified behavior.
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

- **Reviewing a PR's tests, to catch gaps a quick glance at the diff wouldn't** — checking whether new or changed logic-owning methods actually have tests that exercise them, not just tests that exist

  ```bash
  /junit-guidelines Using the JUnit guidelines, review this PR's diff (`gh pr diff 123`). For each new or changed logic-owning method, check whether the PR's tests actually exercise that logic (conditional logic paths, edge cases, exceptions) — not just whether a test method exists. Report any business logic added or changed without corresponding test coverage. Do not write any test code yet — just the report.
  ```

## What the skill actually does

`junit-guidelines` is an AI skill — think of it as a set of instructions an AI coding agent (Claude Code, in this case) loads automatically whenever it's about to write, review, or modify a JUnit 5 test file. The agent isn't asked to invent its own testing philosophy; one is handed to it, with concrete rules and both good and bad examples for each.

A few of the ideas at the core of it:

- **FIRST principles** — tests should be Fast, Independent, Repeatable, Self-Validating, and Timely.
- **Behavior-driven naming** — a test name should read like `behavior_action_expectedResult`, so a failure communicates what broke without the file being opened.
- **Given-When-Then structure** — every test follows the same readable shape.
- **Mocks vs. fakes** — the skill is explicit about when a Mockito mock is fine and when a stateful dependency deserves an interface-based fake instead.
- **Logic-owning vs. orchestrating methods** — methods with real conditional logic are tested exhaustively; methods that just wire calls together get a representative test, not ten redundant ones.
- **"A test that cannot catch a real bug should not be written"** — if a method has no branching, no transformation, no error handling, and just forwards its arguments to a dependency, the test is skipped. It would only verify Mockito wiring, not application behavior.

A smaller rule worth calling out: any identifier, code, or string reused across more than one test is expected to become a named `public static final` constant. It sounds minor, but it removes a whole category of "which magic string was this again" confusion later.

In total, the skill covers 13 rules, from general test guidelines down to descriptive failure messages, each with a rationale rather than just a directive.

## A second pair of eyes: the validator subagent

The most useful part isn't the rules themselves — it's the last step. Once tests are generated, the skill hands them off to a separate subagent, [`junit-validator`](https://github.com/sri-chalam/ai-tools/blob/main/agents/engineering/junit-guidelines/junit-validator.md), which reviews them in a completely fresh context.

That "fresh context" part matters. The same context that wrote the tests isn't well positioned to notice its own mistakes — its own naming and coverage decisions are already assumed to be correct. The validator has no memory of how the tests were written. The tests are read, the class under test is read, the guidelines are read, and a findings table is reported back: naming problems, cases where a test's name doesn't match what its body actually asserts, and missing coverage for real branches or error paths. It's read-only — no code is ever edited by it; only what needs fixing is reported.

The skill takes that report back, fixes whatever the validator flagged, and resolves any compile errors the changes introduce — so the fresh pair of eyes isn't just advisory, it's acted on before the work is considered done.

## AI Agent Compatibility

This skill's guidelines are written for any AI coding agent — Claude Code, Codex, Copilot, and others. However, the final validation step relies on a subagent — a fresh-context reviewer that checks generated tests against the guidelines, fixes discrepancies, and resolves compile issues. A subagent is used so validation happens with a fresh pair of eyes rather than the same context that wrote the tests, but the subagent invocation mechanism itself is Claude Code-specific, and its instructions may not work in other AI coding agents. There does not appear to be a portable, cross-agent way to invoke subagents at this time. On other agents, test generation still follows the guidelines directly; only the separate validation pass is unavailable.

## Getting set up

A naive setup — copying the skill files into `~/.claude/skills` and `~/.claude/agents` — creates a recurring chore: every time the guidelines are tweaked (and they change often, as edge cases surface and rules get refined), the copies would need to be redone by hand. Symbolic links avoid that: clone the repo once, link into place, and from then on a `git pull` plus restarting the AI coding agent is all it takes to pick up updates everywhere they're linked.

```bash
git clone https://github.com/sri-chalam/ai-tools.git
ln -s /path/to/ai-tools/skills/engineering/junit-guidelines ~/.claude/skills/junit-guidelines
ln -s /path/to/ai-tools/agents/engineering/junit-guidelines/junit-validator.md ~/.claude/agents/junit-validator.md
```

After that, the AI coding agent is restarted, and the skill is picked up automatically whenever something matching `src/test/**/*.java`, `**/*Test.java`, or `**/*Tests.java` is touched. It can also be invoked explicitly with `/junit-guidelines`, which is worth doing when it needs to be certain the skill is loaded rather than left to be inferred from context.

Full details, including the complete rule list and ready-to-use prompts, are in the [README](https://github.com/sri-chalam/ai-tools/blob/main/skills/engineering/junit-guidelines/README.md).

## Reference

This skill was reviewed against [Writing Great Skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-great-skills), which argues a SKILL.md should stay small and free of redundant ("noop") instructions — that its description should front-load the skill's leading word so the description does its invocation work first, and treat each real use case as one branch, giving each branch exactly one trigger.

## Closing thought

None of this is groundbreaking — it's mostly unit-testing advice that's existed for years, just written down explicitly enough that an AI agent can actually follow it. AI-generated tests aren't necessarily worse than human-written ones. They're worse *by default*, because nothing stops the agent from taking the path of least resistance: mock everything, assert something, move on. Writing the guidance down, and adding a second pass that checks the work with fresh eyes, has made the resulting tests noticeably more trustworthy — and that's really all this was aimed at.

This is a living document, not a finished one, and using it well means staying involved rather than trusting it blindly:

- **Verify the generated tests.** The AI isn't perfect — always check the output for anything that looks off before trusting it.
- **When something incorrect slips through, ask why.** Prompt the model: why didn't the AI skill catch this, and what change to the SKILL.md would prevent it from recurring? Feed that suggestion back into the guidelines so the same mistake doesn't repeat.
