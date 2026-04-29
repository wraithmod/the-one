# Claude Models And Claude Code Headless Guide

Last updated: 2026-04-09

This file documents what is actually visible from this machine and this Claude Code account today.

Data sources:
- Local Claude Code auth and state: `~/.claude.json`, `~/.claude/.credentials.json`, `~/.claude/settings.json`
- Local Claude Code sessions: `~/.claude/projects/**/*.jsonl`
- Local CLI help: `claude --help`, `claude auth --help`, `claude auth status`
- Installed CLI strings: `/usr/local/lib/node_modules/@anthropic-ai/claude-code/cli.js`
- Official docs:
  - https://code.claude.com/docs/en/model-config
  - https://code.claude.com/docs/en/setup
  - https://platform.claude.com/docs/en/api/rate-limits

## Quick Reference

Current account snapshot:
- Installed CLI version: `2.1.97`
- Auth state: logged in via `claude.ai`
- API provider: `firstParty`
- Email: `drrwgla@gmail.com`
- Subscription type: `pro`
- Headless entrypoint: `claude -p "prompt"`
- Machine-readable outputs: `--output-format json` and `--output-format stream-json`
- Structured output support: `--json-schema`
- Confirmed real model identifiers seen in local sessions: `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`
- Confirmed limit signal from this account: local rate-limit message with reset at `12pm (Australia/Brisbane)`

Quick commands:

```bash
claude -p "Summarize this repository"
```

```bash
claude -p "Return a deployment checklist" --output-format json
```

```bash
claude --model sonnet -p "Review the staged diff for regressions"
```

## Scope And Caveats

This document is intentionally strict about what is confirmed.

- I can confirm the local Claude Code installation, login method, subscription type, and the model aliases documented for Claude Code.
- I can confirm model identifiers that have actually appeared in local session logs.
- I can confirm one real account-local rate-limit event from past session logs.
- I cannot truthfully claim absolute Claude.ai subscription quotas for hourly, daily, weekly, or monthly usage from local state because Claude Code does not expose them here.
- Anthropic API rate-limit tables exist, but they apply to Anthropic Console/API organizations and are not the same thing as Claude Code subscription usage limits.

## Local Auth State

Observed local configuration:
- `claude auth status` reports:
  - `loggedIn: true`
  - `authMethod: claude.ai`
  - `apiProvider: firstParty`
  - `subscriptionType: pro`
- `~/.claude.json` contains an `oauthAccount` object with:
  - `emailAddress: drrwgla@gmail.com`
  - `billingType: stripe_subscription`
  - `hasExtraUsageEnabled: true`
- `~/.claude/settings.json` currently contains only `skipDangerousModePermissionPrompt`

Operational interpretation:
- Claude Code is authenticated and usable on this machine through Anthropic’s first-party Claude account flow.
- This is not the same as having an Anthropic Console API key.
- Direct Anthropic API examples should be treated as templates unless a real API key is configured separately.

## Models Available To This Account In Claude Code

There are two distinct sources of truth here:

1. Models and aliases officially supported by Claude Code
2. Model identifiers actually observed in this account’s local session logs

### CLI-Supported Aliases

Anthropic’s official Claude Code model configuration docs currently describe these aliases:

| Model Or Alias | Status | Notes |
| --- | --- | --- |
| `default` | CLI-supported | Clears explicit override and uses the recommended model for this account type. |
| `best` | CLI-supported | Uses the most capable available model, currently equivalent to `opus`. |
| `sonnet` | CLI-supported | Official docs say this maps to the latest Sonnet model, currently Sonnet 4.6. |
| `opus` | CLI-supported | Official docs say this maps to the latest Opus model, currently Opus 4.6. |
| `haiku` | CLI-supported | Official docs describe this as the fast small model. |
| `opusplan` | CLI-supported | Uses Opus in plan mode, then Sonnet for execution. |

### Full Model Names Found In The Installed CLI

These model names were found in the installed Claude Code package. Presence here means the CLI knows about them, not that every one is necessarily active for this account right now.

| Model | Status | Notes |
| --- | --- | --- |
| `claude-sonnet-4-6` | CLI-known | Present in installed CLI strings. |
| `claude-sonnet-4-5` | CLI-known | Present in installed CLI strings. |
| `claude-sonnet-4` | CLI-known | Present in installed CLI strings. |
| `claude-opus-4-6` | CLI-known | Present in installed CLI strings. |
| `claude-opus-4-5` | CLI-known | Present in installed CLI strings. |
| `claude-opus-4-1` | CLI-known | Present in installed CLI strings. |
| `claude-opus-4` | CLI-known | Present in installed CLI strings. |
| `claude-haiku-4-5` | CLI-known | Present in installed CLI strings. |
| `claude-haiku-4` | CLI-known | Present in installed CLI strings. |
| `claude-3-7-sonnet` | CLI-known | Present in installed CLI strings. |
| `claude-3-5-sonnet` | CLI-known | Present in installed CLI strings. |
| `claude-3-5-haiku` | CLI-known | Present in installed CLI strings. |
| `claude-3-opus` | CLI-known | Present in installed CLI strings. |
| `claude-3-sonnet` | CLI-known | Present in installed CLI strings. |
| `claude-3-haiku` | CLI-known | Present in installed CLI strings. |

### Models Actually Observed In Local Sessions

These are the strongest account-local signals because they were actually used in saved Claude Code sessions on this machine.

| Model | Status | Notes |
| --- | --- | --- |
| `claude-sonnet-4-6` | observed live | Seen in local project session logs. |
| `claude-haiku-4-5-20251001` | observed live | Seen in local subagent session logs. |

Interpretation:
- `CLI-supported` means the alias is documented by Anthropic for Claude Code.
- `CLI-known` means the full model name appears in the installed Claude Code package.
- `observed live` means the model identifier actually appears in local session logs for this account.
- The strongest evidence for “available to this account” is `observed live`.

## Rate Limits

### What Is Confirmed From This Account

The strongest local rate-limit signal I found is a real Claude Code session error:

| Signal | Value |
| --- | --- |
| Error type | `rate_limit` |
| Observed message | `You've hit your limit · resets 12pm (Australia/Brisbane)` |
| Observed timestamp | `2026-02-27T20:34:29.131Z` |
| Source | local Claude Code session log |

Other account-local usage signals:
- Account type is `pro`
- `hasExtraUsageEnabled` is `true` in local Claude state

### Requested Windows

Claude Code local state does not expose absolute usage caps for the windows below.

| Window | Status | Notes |
| --- | --- | --- |
| 5-minute | not exposed | No absolute 5-minute limit surfaced in local Claude Code state. |
| Hourly | not exposed | No absolute hourly limit surfaced in local Claude Code state. |
| Daily | not exposed | No absolute daily limit surfaced in local Claude Code state. |
| Weekly | partially exposed | A local rate-limit message includes a reset time, but not a numeric weekly cap. |
| Monthly | not exposed | No absolute monthly Claude Code subscription limit surfaced in local state. |

### API Rate Limits Versus Claude Code Limits

Anthropic’s official API docs describe API organization limits in terms of RPM, ITPM, and OTPM. Those are relevant if you use the Anthropic Console/API with an API key.

That is different from this machine’s current Claude Code setup:
- Current setup: `claude.ai` first-party login with `pro` subscription
- Not currently evidenced here: a configured Anthropic Console API key

Operational guidance:
- Do not assume Claude Code subscription usage limits match Anthropic API limits.
- Treat the local reset message as the only verified account-local limit signal here.
- If you need exact API quotas, check the Anthropic Console Limits page for the API organization tied to your API key.

## Headless Claude Code Usage

Claude Code supports non-interactive mode through `-p/--print`.

Key local CLI capabilities:
- `-p, --print` runs non-interactively
- `--output-format text|json|stream-json` controls output format
- `--json-schema` validates structured output
- `--model <model>` selects a model alias or full model name
- `--fallback-model <model>` enables automatic fallback in print mode
- `--permission-mode <mode>` supports `default`, `dontAsk`, `acceptEdits`, `bypassPermissions`, and `plan`
- `--allowed-tools` and `--disallowed-tools` constrain tool access
- `--max-budget-usd <amount>` caps API spend in print mode
- `--no-session-persistence` disables session saving in print mode
- `--input-format stream-json` and `--output-format stream-json` support streaming pipelines

Core patterns:

```bash
claude -p "Summarize this repository and propose the next three tasks"
```

```bash
printf '%s\n' "Review the staged diff for regressions" | claude -p "$(cat)"
```

```bash
claude --model sonnet -p "Explain the failing test and suggest a fix" --output-format json
```

```bash
claude --model opus --fallback-model sonnet -p "Draft a migration plan" --output-format text
```

```bash
claude --json-schema '{"type":"object","properties":{"items":{"type":"array","items":{"type":"string"}}},"required":["items"]}' \
  -p "Return a deployment checklist as JSON" \
  --output-format json
```

## Headless Formatting Best Practices

For reliable automation, prefer prompts with four parts:

1. Goal
2. Constraints
3. Required output format
4. Validation rule

Example:

```text
Goal: review the staged diff for user-visible regressions.
Constraints: do not modify files; focus on correctness, security, and missing tests.
Output format: JSON matching the supplied schema.
Validation rule: if there are no findings, return an empty findings array.
```

Recommended rules:
- Ask for one artifact per run.
- State whether Claude may edit files, run tools, or only analyze.
- Use explicit model selection in automation instead of relying on whatever `default` maps to later.
- Prefer `--json-schema` when another tool will consume the result.
- Use `stream-json` only when the downstream consumer is built to handle event streams.
- Add a fallback model for unattended workflows.
- Keep prompts narrow and deterministic.

## Chaining Patterns

### Pattern 1: Review To Markdown

```bash
claude --model sonnet \
  -p "Review the current repository for deployment risks and write a short markdown report." \
  > review.md
```

### Pattern 2: Structured Output For Another Tool

```bash
claude --model sonnet \
  --json-schema '{"type":"object","properties":{"items":{"type":"array","items":{"type":"string"}}},"required":["items"]}' \
  --output-format json \
  -p "Return a machine-readable release checklist." \
  > checklist.json
```

### Pattern 3: Diff Review Pipeline

```bash
git diff --staged | claude \
  --model sonnet \
  -p "Review this staged diff. Find bugs, regressions, and missing tests. Return concise markdown." \
  > diff-review.md
```

### Pattern 4: Stream Events For Automation

```bash
claude --model sonnet \
  --output-format stream-json \
  --include-partial-messages \
  -p "Generate a migration plan for this repository."
```

### Pattern 5: Fallback Model Chain

This is the safest production pattern when higher-tier models may hit usage thresholds.

```bash
claude --model opus --fallback-model sonnet \
  -p "Draft release notes for this repository." \
  > notes.md
```

Chaining guidance:
- Persist intermediate artifacts to files.
- Prefer `json` for machine checkpoints and markdown/text for human checkpoints.
- Keep each run single-purpose.
- Make later steps consume explicit artifacts from earlier steps.
- Add fallback behavior for model overload or subscription limits.

## Recommended Defaults

Use `sonnet` when:
- You want the safest general default for Claude Code automation.
- The task is normal coding, review, or repo analysis.
- You want stable cost/performance.

Use `opus` when:
- The task is high-stakes.
- The task is planning-heavy or architecture-heavy.
- Higher latency is acceptable.

Use `haiku` when:
- The task is small, repetitive, or latency-sensitive.
- You need a cheap fast pass before a stronger second pass.

Use `opusplan` when:
- You want Opus-level reasoning during planning but Sonnet during execution.
- The workflow naturally separates planning and implementation.

## CI And Cron Examples

### GitHub Actions

```yaml
name: claude-review

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      - name: Run Claude review
        run: |
          claude \
            --model sonnet \
            --output-format text \
            -p "Review this repository for correctness, regressions, and missing tests." \
            > claude-review.md
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: claude-review
          path: claude-review.md
```

### Cron-Friendly Shell Script

```bash
#!/usr/bin/env bash
set -euo pipefail

repo_dir=/srv/my-repo
out_dir=/var/log/claude
mkdir -p "$out_dir"

cd "$repo_dir"

claude \
  --model sonnet \
  -p "Summarize repository status, likely risks, and next actions." \
  > "$out_dir/daily-summary.md"
```

### CI Best Practices

- Use explicit model aliases in CI.
- Prefer `sonnet` as the default automation model.
- Add `--fallback-model sonnet` when using `opus` in unattended jobs.
- Redirect final output to files instead of scraping terminal text.
- Use `--no-session-persistence` in ephemeral runners if you do not want stored state.

## Direct API Usage

This section is intentionally limited. I do not have a confirmed Anthropic Console API key configured on this machine, so I cannot truthfully document account-specific direct API entitlements or quotas from local evidence.

Operational guidance:
- Treat the local `claude` CLI as the confirmed interface on this machine.
- If you later add an Anthropic API key, re-run this document and add direct API examples for Messages or Responses-style workflows.
- Do not assume Claude Code subscription limits match Anthropic API organization limits.

## What To Recheck Later

Re-run this document if any of the following change:
- Claude Code version
- Claude account plan or extra-usage status
- Local observed model identifiers in saved sessions
- Anthropic’s documented Claude Code model alias mappings
- Anthropic exposes more precise Claude Code usage telemetry locally
