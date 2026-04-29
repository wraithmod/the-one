# Model Definitions

Last updated: 2026-04-30

This folder is the local operating guide for choosing AI tools and model CLIs in this repository. Treat these files as practical machine-local references, not provider billing dashboards or global model catalogs.

## Files

- [OPENAI_MODELS.md](OPENAI_MODELS.md): Codex/OpenAI model availability, local auth notes, headless `codex exec` usage, and rate-limit evidence.
- [GEMINI_MODELS.md](GEMINI_MODELS.md): Gemini CLI model names, local auth notes, headless `gemini -p` usage, and observed capacity behavior.
- [CLAUDE-MODELS.md](CLAUDE-MODELS.md): Claude Code aliases, local auth notes, headless `claude -p` usage, and observed limit behavior.
- [git-tools.md](git-tools.md): Repository setup, remotes, sync commands, commit workflow, GitHub SSH/HTTPS notes, and safe automation patterns.

## Quick Chooser

Use `codex` when:

- The task is repository-heavy, coding-heavy, or agentic.
- Strong local tool integration matters.
- You want a headless interface built around `codex exec`.

Use `gemini` when:

- You want a fast prompt-driven CLI with text, JSON, or streaming JSON output.
- The work is bounded and can tolerate model-capacity variability.
- You can treat preview models cautiously and verify claims.

Use `claude` when:

- You want high-signal review, planning, or structured non-interactive output.
- You want schema validation with `--json-schema`.
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

## Recommended Defaults

- Default coding workflow: start with `codex`.
- Fast headless automation: start with `gemini -m gemini-2.5-flash -p ...`.
- Review and structured planning: start with `claude --model sonnet -p ...`.
- High-stakes planning: prefer `claude --model opus` or `codex exec -m gpt-5.4`.
- High-volume iterative coding: prefer `codex exec -m gpt-5.4-mini` or `gemini -m gemini-2.5-flash`.

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

## Rate-Limit Reality Check

The local tools do not expose limits in the same way.

- OpenAI Codex: local telemetry has exposed rolling `300` minute and `10080` minute windows.
- Gemini CLI: local evidence includes a `429 RESOURCE_EXHAUSTED` capacity response on `gemini-3-flash-preview`.
- Claude Code: local evidence includes a saved rate-limit message with a reset time.

Do not build automation that assumes absolute 5-minute, hourly, daily, weekly, or monthly caps unless a provider-specific source exposes those values for the exact account and auth mode in use.

## Prompting Standards

For reliable headless work, prompts should usually include:

1. Goal: the concrete task to perform.
2. Context: files, commands, current branch, known constraints.
3. Output format: bullets, JSON schema, patch plan, test list, or exact artifact.
4. Acceptance criteria: what must be true for the answer to be usable.
5. Hard limits: max length, no markdown tables, no speculation, or cite-only claims when needed.

For code tasks, ask for:

- root cause
- minimal patch plan
- risks and regressions
- tests to add or update

For factual or operational tasks, require explicit assumptions and source confidence.

## When To Recheck

Re-run or refresh the provider-specific docs if any of the following change:

- CLI version
- login or subscription state
- visible model catalog
- local rate-limit behavior
- provider docs for headless mode or model aliases
- repository remote or hosting provider

