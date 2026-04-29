# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Task tool (localSuperpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**In addition to standard code quality concerns, the reviewer should check:**
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the file structure from the plan?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)

**Test decision review** — match coverage to the code, not to a coverage target:
- If the implementation has logic (branching, transformation, validation, calculation, state changes, error handling), the reviewer should expect tests covering it. Flag missing tests as Important.
- If the implementation is pure passthrough/boilerplate (e.g. a UseCase that calls a single repository method, DI wiring, plain data classes), the reviewer should NOT demand tests. Flag tests that only mirror the implementation or `verify(mock).method()` without an assertion as Important — vacuous tests are noise.
- If the plan said `Tests: skip` but the code has real branching or mapping, that's an Important issue (skipped tests on real logic).
- If the plan said `Tests: required` but the code is just a passthrough wrapper, that's a Minor issue (plan miscalled the test decision; tests aren't necessarily wrong, but ask whether they assert anything).

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment
