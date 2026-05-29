# Spec Document Reviewer Prompt Template

Use this template when reviewing a spec document in Codex CLI.

**Purpose:** Verify the spec is complete, consistent, and ready for implementation planning.

**Run after:** Spec document is written to docs/superpowers/specs/ and the agent has completed its local spec self-review.

**Required review mechanism:** Use Codex CLI for this external review gate, not a subagent or another inline self-review. Start one dedicated Codex CLI session for the spec and keep using that same session for every re-review until the spec is approved.

Check availability first:

```bash
command -v codex
```

If `codex` is not installed or not on `PATH`, tell the user Codex CLI is required for the spec review gate and ask whether to proceed directly to user review or wait for Codex installation.

Preferred same-session command:

```bash
codex --no-alt-screen -C "$PWD" "You are a spec document reviewer. Verify SPEC_FILE_PATH is complete and ready for planning. Use skills/brainstorming/spec-document-reviewer-prompt.md as your review rubric. Return Approved only if no blocking issues remain; otherwise return Issues Found."
```

Keep that Codex session open. If Codex shows a session id, record it immediately. For every updated draft, send the follow-up in the same session:

```text
I updated SPEC_FILE_PATH to address your findings. Re-review the current file in this same review thread. Return Approved only if no blocking issues remain; otherwise return Issues Found.
```

If an interactive session cannot stay open, use `codex exec` for the first review and `codex exec resume` for each re-review. Do not start a fresh `codex exec` session for follow-up reviews.

Prefer resuming by session id:

```bash
codex exec resume SESSION_ID "I updated SPEC_FILE_PATH to address your findings. Re-review the current file in this same review thread. Return Approved only if no blocking issues remain; otherwise return Issues Found."
```

Never use `codex exec resume --last` for spec review. Parallel agents or other Codex work may have created a newer session. If the interactive session is closed and you did not record the session id, start a new Codex review round from the beginning: review the current spec as a fresh first review, record the new session id, and use that new same session for all follow-up re-reviews.

```
Codex CLI review prompt:
    You are a spec document reviewer. Verify this spec is complete and ready for planning.

    **Spec to review:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", incomplete sections |
    | Consistency | Internal contradictions, conflicting requirements |
    | Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
    | Scope | Focused enough for a single plan — not covering multiple independent subsystems |
    | YAGNI | Unrequested features, over-engineering |

    ## Calibration

    **Only flag issues that would cause real problems during implementation planning.**
    A missing section, a contradiction, or a requirement so ambiguous it could be
    interpreted two different ways — those are issues. Minor wording improvements,
    stylistic preferences, and "sections less detailed than others" are not.

    Approve unless there are serious gaps that would lead to a flawed plan.

    ## Output Format

    ## Spec Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters for planning]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
