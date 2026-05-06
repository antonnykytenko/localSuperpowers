---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, the *behavior* and *contracts*, docs they might need to check, and — for tasks that contain real logic — how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. Test logic, not boilerplate. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well — be explicit about which tasks need tests and which don't.

**Specify behavior, not code.** Plans describe *what* needs to happen and the contracts (signatures, types, edge cases) the implementer must honor. The implementer writes the code. Inline code in plans is the exception, not the default — see "When to Include Code" below.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)
- **DO NOT commit the plan.** `docs/superpowers/` is in `.gitignore` by design — plans and specs are local working artifacts, not repository history. The folder being gitignored does NOT mean "don't write here" — read and write freely; just never `git add` or commit anything under `docs/superpowers/`.

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## When to Include Code

**Default: don't.** Describe behavior, contracts, and edge cases in prose. The implementer is a skilled developer — they can write idiomatic code from a precise spec.

**Include code only when:**
- **The user provided an example** — paste it verbatim so nothing is lost in paraphrasing.
- **The signature must be locked in** — when later tasks reference a function/class, show *just the signature* (`fun map(dto: Dto): Domain`), not the body. Locks the contract; lets the implementer write the body.
- **Genuine ambiguity** — multiple valid implementations differ in observable behavior, and you need to pin one down (e.g. "use `Flow.combine`, not `zip`, because we want latest-of-each").
- **Non-obvious idiom** — a specific framework pattern (DataStore migration, custom Coroutine scope) where the implementer would likely guess wrong without an example.
- **A test case where the input/expected pair is dense** — sometimes a 3-line code block beats two paragraphs of prose. Use judgment.

**Don't include code for:**
- Trivial mappings, getters, single-statement functions
- Standard idioms in the codebase (the implementer can grep for examples)
- "Fill in the obvious implementation" cases — describe the behavior and stop
- Test bodies for straightforward behavior — list inputs and expected outputs instead

**Behavior spec format (use instead of code):**

```
- [ ] **Step 3: Implement `Mapper.map(dto)`** in `Mapper.kt`

  Signature: `fun map(dto: Dto): Domain`

  Behavior:
  - When `dto.name` is null → `Domain.name = "Unknown"`
  - Otherwise → pass through
  - All other fields map 1:1 by name

  Tests cover the null branch (see Step 1).
```

**Test spec format (use instead of full test code):**

```
- [ ] **Step 1: Add test `maps null name to Unknown`** in `MapperTest.kt`

  Given: `Dto(name = null, id = 42)`
  Expect: `Domain(name = "Unknown", id = 42)`
```

This is shorter, easier to skim, and forces the implementer to choose idiomatic test scaffolding for the project (JUnit5, Kotest, Turbine, etc.) rather than copy-pasting whatever style the plan happened to use.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes).**

For tasks with logic worth testing:
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

For tasks that are pure boilerplate (no logic):
- "Implement the file" - step
- "Verify it compiles / builds" - step
- "Commit" - step

## Test Decision Per Task

**Before drafting steps for a task, decide whether it warrants tests.** This decision belongs in the plan — don't push it onto the implementer. Mark it explicitly with a `**Tests:**` line under each task header.

**Mark `Tests: required` when the task introduces or changes:**
- Branching / conditionals / fallbacks
- Transformation or mapping with non-trivial fields
- Validation rules
- Calculations or aggregation
- State changes / state machines / state management
- Error handling, retries, recovery
- Combining multiple inputs (Flow combine, Result merging)
- Side-effect ordering or cancellation logic

**Mark `Tests: skip` when the task is:**
- A UseCase / Repository method that just forwards to a single source (e.g. `prefs.getString(KEY, null)`)
- A trivial getter/setter or property exposure
- A plain data class, DTO, or sealed-class state object with no methods
- DI / Hilt / module wiring
- Generated code or scaffolding
- Pure Compose UI rendering static state (no decision logic)

**If unsure, draft the test signature.** If the test body would only assert "the value flows through" or `verify(repo).get()`, mark `Tests: skip`. Don't pad the plan with tests that mirror the implementation.

When `Tests: skip`, briefly state *why* (e.g. "passthrough to SharedPreferences, no mapping or fallbacks") so the implementer and reviewer can sanity-check the call.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use localSuperpowers:subagent-driven-development (recommended) or localSuperpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

Default to behavior specs. Drop into code only when "When to Include Code" applies.

### Template A — Task with logic (`Tests: required`)

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/File.kt`
- Modify: `exact/path/to/Existing.kt:123-145`
- Test: `exact/path/to/FileTest.kt`

**Tests:** required — [one-line reason: "maps DTO to domain with default fallback when name is null"]

**Contract:** `fun map(dto: Dto): Domain`

- [ ] **Step 1: Add failing test `maps null name to Unknown`** in `FileTest.kt`

  Given: `Dto(name = null, id = 42)`
  Expect: `Domain(name = "Unknown", id = 42)`

- [ ] **Step 2: Run test to verify it fails**

  Run: `./gradlew :module:test --tests "*FileTest.maps null name to Unknown*"`
  Expected: FAIL — function not defined or assertion error

- [ ] **Step 3: Implement `Mapper.map`** in `Mapper.kt`

  Behavior:
  - `dto.name == null` → `Domain.name = "Unknown"`
  - otherwise → pass through `dto.name`
  - all other fields map 1:1 by name

- [ ] **Step 4: Run test to verify it passes**

  Run: `./gradlew :module:test --tests "*FileTest*"`
  Expected: PASS

- [ ] **Step 5: Commit**

  `git commit -m "feat: map DTO to domain with default name"`
````

### Template B — Task without logic (`Tests: skip`)

Use when the task is a trivial passthrough, DI wiring, data class, or other boilerplate.

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/File.kt`

**Tests:** skip — [one-line reason: "passthrough to SharedPreferences, no mapping or fallbacks"]

**Contract:** `class GetUserNameUseCase(prefs: SharedPreferences) { operator fun invoke(): String? }`

- [ ] **Step 1: Implement `GetUserNameUseCase`** in `GetUserNameUseCase.kt`

  Returns `prefs.getString(KEY_USER_NAME, null)`. No fallback, no mapping.

- [ ] **Step 2: Verify it builds**

  Run: `./gradlew :module:assemble`
  Expected: BUILD SUCCESSFUL

- [ ] **Step 3: Commit**

  `git commit -m "feat: add GetUserNameUseCase"`
````

### When to drop into code

If the task hits one of the cases in "When to Include Code" (user-provided example, genuine ambiguity, non-obvious idiom), include the code block in the relevant step. Otherwise stick to contract + behavior spec.

## No Placeholders

Every step must contain enough specificity that the engineer can act on it without guessing. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases" — list the specific cases instead
- "Write tests for the above" — list the test cases (input → expected) instead
- "Similar to Task N" — restate the contract; the engineer may read tasks out of order
- Steps that don't pin down behavior — for code steps, either show the signature + behavior spec, or show code (per "When to Include Code")
- References to types, functions, or methods not defined in any task

**Specificity ≠ code.** A precise behavior spec ("when `dto.name` is null, return `Unknown`; otherwise pass through") is specific. "Implement the mapping function" is a placeholder. Don't conflate the two.

## Remember
- Exact file paths always
- Specify behavior + contracts, not full code — drop into code only when "When to Include Code" applies
- Exact commands with expected output
- Every task has an explicit `**Tests:** required | skip — [reason]` line
- Tasks with logic also list a `**Contract:**` line (signature / class shape) so later tasks can reference it
- DRY, YAGNI, frequent commits, test logic but not boilerplate

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Test decisions:** Every task has a `Tests: required | skip — [reason]` line. For each `skip`, ask: does the implementation as written really have no logic? If a "passthrough" actually maps fields, applies a default, or has a branch, change it to `required`. For each `required`, ask: would the test assert anything beyond "the language works"? If not, change it to `skip`.

**5. Code-block justification:** Scan every code block in the plan. For each one, can you state which "When to Include Code" rule justifies it (user example, locked signature, ambiguity, non-obvious idiom, dense input/expected pair)? If not, replace it with a behavior spec. Code blocks that just restate the obvious implementation are token waste.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use localSuperpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use localSuperpowers:executing-plans
- Batch execution with checkpoints for review
