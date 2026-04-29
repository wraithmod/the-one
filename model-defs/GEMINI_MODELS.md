# Gemini Models And Headless CLI Guide

Last updated: 2026-04-09

This file documents what is actually visible from this machine and this Gemini CLI installation today.

Data sources:
- Local Gemini state: `~/.gemini/settings.json`, `~/.gemini/google_accounts.json`, `~/.gemini/oauth_creds.json`
- Local Gemini CLI help: `gemini --help`
- Installed Gemini CLI bundle strings: `/usr/local/lib/node_modules/@google/gemini-cli`
- Live CLI behavior observed on this machine: `429 RESOURCE_EXHAUSTED` for `gemini-3-flash-preview`

## Quick Reference

Current account snapshot:
- Installed CLI version: `0.37.0`
- Auth state: personal OAuth is configured locally
- Active Google account in local state: `drrwgla@gmail.com`
- Headless entrypoint: `gemini -p "prompt"`
- Machine-readable outputs: `--output-format json` and `--output-format stream-json`
- Confirmed live capacity symptom: `gemini-3-flash-preview` returned `429 RESOURCE_EXHAUSTED`

Quick commands:

```bash
gemini -p "Summarize this repository"
```

```bash
gemini -m gemini-2.5-flash -p "Return a release summary" --output-format json
```

```bash
git diff --staged | gemini -p "Review this staged diff for regressions" --output-format text
```

## Scope And Caveats

This document is intentionally strict about what is confirmed.

- I can confirm the local Gemini CLI installation, auth mode, and the model names embedded in the installed CLI.
- I can confirm at least one real server-side limit condition from this account: a `429 RESOURCE_EXHAUSTED` response for `gemini-3-flash-preview`.
- I cannot truthfully claim account-specific absolute hourly, daily, or monthly quotas from the current local state because the Gemini CLI does not expose them here.
- Model names found in the installed bundle should be treated as CLI-known targets, not guaranteed entitlements for every request.

## Local Auth State

Observed local configuration:
- `~/.gemini/settings.json` shows `oauth-personal` under security auth
- `~/.gemini/oauth_creds.json` contains populated OAuth credentials
- `~/.gemini/google_accounts.json` shows active account `drrwgla@gmail.com`

One inconsistency exists in local settings:
- `selectedAuthType` is recorded as `apiKey`
- `security.auth.selectedType` is recorded as `oauth-personal`

Operational interpretation:
- The CLI is effectively using personal OAuth credentials.
- The top-level `selectedAuthType` field should not be treated as authoritative by itself.

## Models Available To This Account In Gemini CLI

These model names were extracted from the installed Gemini CLI bundle. They are the safest local inventory of what the CLI knows how to reference.

Primary models surfaced repeatedly in the installed bundle:

| Model | Status | Notes |
| --- | --- | --- |
| `gemini-2.0-flash` | CLI-known | Present in bundle. |
| `gemini-2.5` | CLI-known | Present in bundle. |
| `gemini-2.5-flash` | CLI-known | Present in bundle and help-adjacent code paths. |
| `gemini-2.5-flash-base` | CLI-known | Present in bundle. |
| `gemini-2.5-flash-lite` | CLI-known | Present in bundle. |
| `gemini-2.5-flash-image` | CLI-known | Present in bundle. |
| `gemini-2.5-pro` | CLI-known | Present in bundle and interactive model lists. |
| `gemini-2.5-computer-use-preview-10-2025` | CLI-known | Preview/specialized model string present in bundle. |
| `gemini-3` | CLI-known | Present in bundle. |
| `gemini-3-flash` | CLI-known | Present in bundle and interactive model lists. |
| `gemini-3-flash-base` | CLI-known | Present in bundle. |
| `gemini-3-flash-preview` | CLI-known and observed live | Returned `429 RESOURCE_EXHAUSTED` when queried on this account. |
| `gemini-3-pro` | CLI-known | Present in bundle and interactive model lists. |
| `gemini-3-pro-preview` | CLI-known | Present in bundle. |
| `gemini-3.1-flash-lite-preview` | CLI-known | Present in bundle. |
| `gemini-3.1-pro` | CLI-known | Present in bundle and interactive model lists. |
| `gemini-3.1-pro-preview` | CLI-known | Present in bundle. |
| `gemini-3.1-pro-preview-customtools` | CLI-known | Present in bundle. |

Non-production-looking string found in the bundle:

| Model | Status | Notes |
| --- | --- | --- |
| `gemini-9001-super-duper` | ignore | Almost certainly internal or test-only. Do not rely on it. |

Interpretation:
- `CLI-known` means the installed Gemini CLI contains explicit references to the name.
- It does not guarantee the account can use the model successfully at runtime.
- Preview models are especially likely to hit availability, entitlement, or capacity constraints.

## Rate Limits

### What Is Confirmed From This Account

The only rate-limit behavior I could verify directly from this machine and account was a live server response:

| Signal | Value |
| --- | --- |
| HTTP status | `429` |
| Error status | `RESOURCE_EXHAUSTED` |
| Error reason | `MODEL_CAPACITY_EXHAUSTED` |
| Observed model | `gemini-3-flash-preview` |
| Provider endpoint family | `cloudcode-pa.googleapis.com` |

### Requested Windows

The account-local Gemini CLI state does not expose absolute quotas for the windows below.

| Window | Status | Notes |
| --- | --- | --- |
| 5-minute | not exposed | No absolute 5-minute cap surfaced in local CLI state. |
| Hourly | not exposed | No absolute hourly cap surfaced in local CLI state. |
| Daily | not exposed | No absolute daily cap surfaced in local CLI state. |
| Monthly | not exposed | No absolute monthly cap surfaced in local CLI state. |

### Operational Guidance

Practical interpretation:
- Treat preview Gemini models as less reliable for unattended automation.
- Prefer stable flash or pro variants for batch/headless tasks.
- Build retries and fallback models into automation.
- Separate quota errors from capacity errors. A `429` here does not prove your account hit a hard monthly quota; it may only indicate server-side model exhaustion.

## Headless Gemini Usage

The local CLI explicitly supports headless mode:

- `-p, --prompt` runs in non-interactive mode
- `-m, --model` selects the model
- `--output-format text|json|stream-json` controls output format
- `-s, --sandbox` toggles sandbox mode
- `-y, --yolo` or `--approval-mode yolo` auto-approve actions
- `--approval-mode plan` provides a read-only planning mode
- `--include-directories` widens the workspace
- `-w, --worktree` starts in a new git worktree

Core patterns:

```bash
gemini -p "Summarize this repository"
```

```bash
gemini -m gemini-2.5-flash -p "Generate release notes from the current repo" --output-format text
```

```bash
printf '%s\n' "Review the staged diff for regressions" | gemini -m gemini-2.5-pro -p "$(cat)" --output-format json
```

```bash
gemini -m gemini-3.1-pro -p "Return a deployment checklist as JSON" --output-format json
```

## Headless Formatting Best Practices

For reliable Gemini CLI automation, use prompts with four parts:

1. Goal
2. Constraints
3. Output format
4. Validation rule

Example:

```text
Goal: review the repository for user-visible regressions.
Constraints: do not modify files; focus on correctness, security, and missing tests.
Output format: JSON with keys findings, risks, and test_gaps.
Validation rule: if there are no findings, return an empty findings array.
```

Recommended rules:
- Make one request produce one artifact.
- State whether file edits are allowed.
- Specify whether commands may run or the task is analysis-only.
- Prefer `json` or `stream-json` output when another tool will consume the result.
- Use stable model names in automation and reserve preview models for interactive use.
- Keep prompts explicit about scope, repository root, and output contract.

## Chaining Patterns

### Pattern 1: Review To Markdown

```bash
gemini -m gemini-2.5-flash \
  -p "Review the repository for deployment risks and write a short markdown report." \
  --output-format text > review.md
```

### Pattern 2: Structured Output For Another Tool

```bash
gemini -m gemini-2.5-pro \
  -p "Return a machine-readable release checklist as JSON." \
  --output-format json > checklist.json
```

### Pattern 3: Diff Review Pipeline

```bash
git diff --staged | gemini \
  -m gemini-2.5-flash \
  -p "Review this staged diff. Find bugs, regressions, and missing tests. Return concise markdown." \
  --output-format text > diff-review.md
```

### Pattern 4: Stream Events For Automation

```bash
gemini -m gemini-2.5-flash \
  -p "Generate a migration plan for this repository." \
  --output-format stream-json
```

### Pattern 5: Fallback Model Chain

This is the safest production pattern when preview models can exhaust capacity.

```bash
gemini -m gemini-3.1-pro -p "Draft release notes" --output-format text > notes.md ||
gemini -m gemini-2.5-pro -p "Draft release notes" --output-format text > notes.md
```

Chaining guidance:
- Save intermediate artifacts to files.
- Use `json` for machine checkpoints and `text` for human checkpoints.
- Keep each run single-purpose.
- Add a fallback model for non-interactive pipelines.
- Do not assume preview capacity will be available.

## Recommended Defaults

Use `gemini-2.5-flash` when:
- You want the best general default for headless automation.
- Latency matters.
- You can validate outputs cheaply.

Use `gemini-2.5-pro` when:
- The task is high-stakes.
- You want a stronger reasoning model.
- Latency is less important than accuracy.

Use `gemini-3.1-pro` when:
- You are testing newer model behavior interactively or in guarded automation.
- You have a fallback path.

Avoid preview-only models in unattended workflows when:
- The pipeline must be reliable.
- Retries are expensive.
- You do not have a graceful fallback.

## CI And Cron Examples

### GitHub Actions

```yaml
name: gemini-review

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Gemini CLI
        run: npm install -g @google/gemini-cli
      - name: Run Gemini review
        run: |
          gemini \
            -m gemini-2.5-flash \
            -p "Review this repository for regressions and missing tests." \
            --output-format text > gemini-review.md
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: gemini-review
          path: gemini-review.md
```

### Cron-Friendly Shell Script

```bash
#!/usr/bin/env bash
set -euo pipefail

repo_dir=/srv/my-repo
out_dir=/var/log/gemini
mkdir -p "$out_dir"

cd "$repo_dir"

gemini \
  -m gemini-2.5-flash \
  -p "Summarize repository status, likely risks, and next actions." \
  --output-format text > "$out_dir/daily-summary.md"
```

### CI Best Practices

- Prefer stable non-preview models for unattended jobs.
- Always redirect outputs to files.
- Use `json` or `stream-json` when another tool consumes the result.
- Add a fallback model path for `429` or capacity failures.
- Keep automation prompts deterministic and scoped.

## Direct API Usage

This section is intentionally limited. I do not have a confirmed Gemini Developer API key or Vertex AI project configuration on this machine, so I cannot truthfully document account-specific direct API entitlements or quota values from local evidence.

Operational guidance:
- Treat the local CLI as the confirmed interface on this machine.
- If you later add a Gemini API key or Vertex AI project, re-run this document and add provider-specific API examples.
- Do not assume CLI-visible model names map one-to-one to your direct API entitlements.

## What To Recheck Later

Re-run this document if any of the following change:
- Gemini CLI version
- Google account or auth mode
- The installed CLI bundle model catalog
- Observed server-side capacity behavior
- Google exposes account-specific quota data through the CLI or API
