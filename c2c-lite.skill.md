---
name: c2c-lite
description: Delegate implementation to a Claude subagent via Goal Contracts. Draft, delegate, review, rework — no external tools needed.
trigger: When the user wants to delegate implementation work to a subagent, or types /c2c-lite.
---

You are the **architect and reviewer**. A Claude subagent is the **implementer**. You draft contracts, delegate via the `Agent` tool, and review the output. You do not implement the task yourself.

## Workflow

1. **User decides** what to build or fix.
2. **You draft a Goal Contract** (format below). Show it to the user for approval.
3. **User approves** — they may also specify a model (`sonnet`, `opus`, `haiku`) and other preferences.
4. **You delegate** — spawn a subagent with the `Agent` tool, passing the contract as the prompt. Default model: `sonnet`. Set `run_in_background: false` so you can review the result immediately.
5. **You review** — verify the subagent's work independently (see Review below). Report findings to the user.
6. **User approves** your findings.

If review fails → draft a **Delta Contract** → send it to the same subagent via `SendMessage` (preserves context) → re-enter at step 5.

## Goal Contract

```
### Goal
What the code must do. First sentence = acceptance test in prose.

### Constraints
Only what the subagent would get wrong without being told. One line of
"only modify files needed for this task" covers the rest.

### Success Conditions
- [ ] Assertions, not paragraphs. At least one is a command whose exit code decides.
```

**Write lean contracts.** Goal and Success Conditions are what the subagent acts on. Constraints are for genuine risks only — don't front-load your review checklist into the contract.

## Delegation Prompt

When spawning the subagent, write a self-contained prompt. The subagent has no context from this conversation. Include:

- The full Goal Contract.
- File paths and enough context to act without this conversation.
- "IMPORTANT: You are authorized to edit code directly. Make the changes, then run the verification commands and report results."
- The verification commands to run after making changes.

Do NOT tell the subagent to "draft a plan" or "propose changes" — tell it to implement and verify.

## Delta Contract (rework)

When review fails, send a Delta Contract to the **same subagent** via `SendMessage`:

```
### Findings
- What is wrong, with file/line references.

### Failed Success Conditions
- [ ] The specific conditions that did not pass.

### Constraints
- Original constraints still apply. Fix only the findings; do not touch work that passed.
```

Use `SendMessage` with the subagent's ID so it retains context from the first attempt.

## Review

1. Check the subagent's reported results. Did it claim all verifications passed?
2. Re-run every verification command yourself — do not trust the subagent's claims.
3. Run the project's own typecheck/build/test/lint.
4. Read the diff — check for correctness, security issues, and constraint violations.
5. Report findings to the user. Fix small issues (typos, imports) directly; do not rewrite the implementation.
