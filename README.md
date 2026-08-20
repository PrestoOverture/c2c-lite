# c2c-lite

A Claude Code Skill that lets you delegate implementation to a Claude subagent through structured Goal Contracts — the same architect/implementer workflow as [claude2codex](https://github.com/PrestoOverture/c2c), but without Codex or any external dependency.

*[中文版](./README.zh-CN.md)*

## Why

[claude2codex](https://github.com/PrestoOverture/c2c) bridges Claude (architect) and Codex (implementer) via an MCP server. It works well, but requires a Codex subscription and a running MCP server.

**c2c-lite** strips that down to zero dependencies: Claude delegates to a Claude subagent (Sonnet by default) using the `Agent` tool that's already built into Claude Code. Same contract format, same review discipline, no infrastructure.

Use this when:
- Codex is unavailable (no subscription, API down, rate limited)
- The task doesn't justify an external process
- You want the contract workflow without any setup

## How It Works

```
You (user)
  → Claude Opus (architect) — drafts Goal Contract
    → Claude Sonnet (subagent/implementer) — writes code, runs tests
  ← Claude Opus — reviews output, reports findings
← You — approve or request rework
```

No MCP server. No external process. Just Claude talking to Claude through the built-in `Agent` tool.

## Installation

```sh
mkdir -p ~/.claude/skills
curl -fsSL https://raw.githubusercontent.com/PrestoOverture/c2c-lite/main/c2c-lite.skill.md -o ~/.claude/skills/c2c-lite.md
```

Or copy manually — the entire thing is one Markdown file.

## Usage

In any Claude Code session, type `/c2c-lite` and describe what you want to build. Claude will:

1. Draft a Goal Contract and show it to you
2. On your approval, spawn a subagent to implement it
3. Review the subagent's work independently
4. Report findings for your approval

If the review fails, Claude sends a Delta Contract to the same subagent (preserving context) for targeted rework.

### Model Selection

The default implementer model is Sonnet. You can specify a different model when approving:

- **Sonnet** (default) — fast, cost-effective, good for most tasks
- **Opus** — stronger reasoning, better for complex or ambiguous tasks
- **Haiku** — fastest, cheapest, good for simple/mechanical changes

## Comparison with claude2codex

| | c2c-lite | claude2codex |
|---|---|---|
| Implementer | Claude subagent | OpenAI Codex |
| Dependencies | None | Codex CLI + MCP server |
| Setup | Copy one file | `npx claude2codex` + MCP config |
| Goal loop | Self-verification loop (prompt-driven) | Codex `/goal` loop (iterates until done) |
| Persistence | None (in-session only) | Jobs persisted to disk |
| Cost estimation | Not available | `codex_estimate` tool |
| Parallel jobs | Yes (multiple Agent calls) | Yes (`C2C_MAX_CONCURRENT`) |
| Rework | `SendMessage` to same agent | `codex_rework` resumes thread |

## License

MIT
