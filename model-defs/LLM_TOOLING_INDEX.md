# LLM Tooling Index

Last updated: 2026-04-09

This index links the provider-specific model guides in this repository and gives a fast way to choose the right local CLI for a task.

Provider guides:
- [OPENAI_MODELS.md](/home/wraith/pm-assist/OPENAI_MODELS.md)
- [GEMINI_MODELS.md](/home/wraith/pm-assist/GEMINI_MODELS.md)
- [CLAUDE-MODELS.md](/home/wraith/pm-assist/CLAUDE-MODELS.md)

## Quick Chooser

Use `codex` when:
- The task is repository-heavy, coding-heavy, or agentic.
- You want strong local tool integration and coding workflows.
- You want a headless interface built around `codex exec`.

Use `gemini` when:
- You want a simple prompt-driven CLI with `json` or `stream-json` output.
- You want a fast headless pipeline and can tolerate model-capacity variability.
- You are comfortable treating preview models cautiously.

Use `claude` when:
- You want a strong general coding assistant with clean headless output and schema validation.
- You want first-party subscription auth through `claude.ai`.
- You want explicit fallback-model support in non-interactive mode.

## Local Snapshot

| Tool | Installed Command | Installed Version | Local Auth Mode | Strongest Confirmed Models | Headless Entry |
| --- | --- | --- | --- | --- | --- |
| OpenAI Codex | `codex` | `0.125.0` | ChatGPT login in Codex | `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, `gpt-5.2` | `codex exec` |
| Gemini CLI | `gemini` | `0.37.0` | personal OAuth | CLI-known Gemini 2.5/3.x family; live `429` observed on `gemini-3-flash-preview` | `gemini -p` |
| Claude Code | `claude` | `2.1.97` | `claude.ai` first-party login, `pro` | observed live `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` | `claude -p` |

## Headless Interface Summary

| Tool | Text Output | JSON Output | Streaming Output | Structured Schema Support | Fallback Model Support |
| --- | --- | --- | --- | --- | --- |
| `codex` | yes | yes via `--json` event stream | yes via JSONL events | yes via `--output-schema` | no first-class fallback flag |
| `gemini` | yes | yes | yes | no first-class schema flag visible in local help | no first-class fallback flag visible in local help |
| `claude` | yes | yes | yes | yes via `--json-schema` | yes via `--fallback-model` |

## Rate-Limit Reality Check

The three local tools do not expose limits in the same way.

OpenAI Codex:
- Strongest local evidence is rolling usage telemetry
- Confirmed windows: `300` minutes and `10080` minutes
- No absolute 5-minute, hourly, daily, or monthly caps exposed locally

Gemini CLI:
- Strongest local evidence is a live `429 RESOURCE_EXHAUSTED` capacity error
- No absolute hourly, daily, or monthly caps exposed locally

Claude Code:
- Strongest local evidence is a saved local rate-limit event with a reset time
- No absolute hourly, daily, weekly, or monthly caps exposed locally

Practical rule:
- Treat all three docs as local operational references, not billing dashboards.
- Where a limit is not exposed, do not build automation that assumes a numeric cap.

## Recommended Defaults

Default coding workflow:
- Start with `codex`

Default fast headless automation:
- Start with `gemini -m gemini-2.5-flash -p ...`

Default high-signal review and structured non-interactive work:
- Start with `claude --model sonnet -p ...`

High-stakes planning:
- Prefer `claude --model opus` or `codex exec -m gpt-5.4`

High-volume iterative coding:
- Prefer `codex exec -m gpt-5.4-mini` or `gemini -m gemini-2.5-flash`

## Common Patterns

Repository summary:

```bash
codex exec -m gpt-5.4-mini --ephemeral --json "Summarize this repository"
```

Fast review:

```bash
claude --model sonnet -p "Review this repository for regressions and missing tests."
```

Simple JSON-producing step:

```bash
gemini -m gemini-2.5-flash -p "Return a release checklist as JSON" --output-format json
```

Fallback-sensitive unattended run:

```bash
claude --model opus --fallback-model sonnet -p "Draft release notes for this repository."
```

## When To Recheck

Re-run the provider-specific docs if any of the following change:
- CLI version
- login or subscription state
- visible model catalog
- local rate-limit behavior
- provider docs for headless mode or model aliases
