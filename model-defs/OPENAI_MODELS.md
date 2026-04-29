# OPENAI Models And Codex Headless Guide

Last updated: 2026-04-09

This file documents what is actually visible from this machine and this account today.

Data sources:
- Local Codex auth state: `~/.codex/auth.json`
- Local Codex model cache: `~/.codex/models_cache.json`
- Latest Codex session telemetry: `~/.codex/sessions/.../rollout-2026-04-09T23-07-11-019d725a-905c-7861-82c6-9c191a085b40.jsonl`
- Local CLI help: `codex --help`, `codex exec --help`
- Official docs:
  - https://developers.openai.com/api/docs/models/all
  - https://developers.openai.com/api/docs/guides/rate-limits
  - https://developers.openai.com/codex/learn/best-practices

## Quick Reference

Current account snapshot:
- Auth mode in Codex: ChatGPT login
- Observed plan in Codex telemetry: `plus`
- Models visible in normal Codex picker: `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, `gpt-5.2`
- Confirmed rolling usage windows exposed locally: `300` minutes and `10080` minutes
- Best default for headless automation: `gpt-5.4-mini`
- Best default for high-stakes coding: `gpt-5.4`
- Best default for code-heavy agentic work: `gpt-5.3-codex`

Quick commands:

```bash
codex exec -m gpt-5.4-mini --ephemeral --json "Summarize this repo"
```

```bash
codex exec -m gpt-5.4 --output-schema schema.json -o result.json "Return structured output"
```

## Scope And Caveats

This account is authenticated in Codex via ChatGPT, not a valid Platform API key. That matters:

- I can confirm the models Codex exposes to this account.
- I can confirm the rolling usage windows Codex telemetry exposes for this account.
- I cannot truthfully claim absolute Platform caps for `5-minute`, `hourly`, `daily`, or `monthly` windows because those values are not exposed by the local auth mode or local telemetry.
- Where a window is not exposed, this document marks it as `not exposed` instead of guessing.

Plan observed in telemetry: `plus`

## Local Auth State

Observed local configuration:
- `~/.codex/auth.json` is present
- `auth_mode` is `chatgpt`
- Stored token set includes `access_token`, `refresh_token`, `id_token`, and `account_id`
- `codex login status` reports `Logged in using ChatGPT`

Operational interpretation:
- Codex is authenticated and usable on this machine.
- This is not the same as having a valid OpenAI Platform API key.
- Platform API examples in this document should be treated as templates unless a real `OPENAI_API_KEY` is configured.

## Models Available To This Account In Codex

These are the models currently present in the local Codex model catalog for this account.

| Model | Visibility In Picker | Context Window | API Supported | Parallel Tool Calls | Search Tool | Default Reasoning | Default Verbosity | Notes |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- |
| `gpt-5.4` | list | 272000 | yes | yes | yes | medium | low | Latest frontier agentic coding model. |
| `gpt-5.4-mini` | list | 272000 | yes | yes | yes | medium | medium | Smaller frontier agentic coding model. |
| `gpt-5.3-codex` | list | 272000 | yes | yes | yes | medium | low | Frontier Codex-optimized agentic coding model. |
| `gpt-5.2` | list | 272000 | yes | yes | yes | medium | low | Optimized for professional work and long-running agents. |
| `gpt-5.2-codex` | hide | 272000 | yes | yes | yes | medium | unset | Frontier agentic coding model. |
| `gpt-5.1` | hide | 272000 | yes | yes | no | medium | low | Broad world knowledge with strong general reasoning. |
| `gpt-5.1-codex-max` | hide | 272000 | yes | no | no | medium | unset | Codex-optimized model for deep and fast reasoning. |
| `gpt-5.1-codex` | hide | 272000 | yes | no | no | medium | unset | Optimized for Codex. |
| `gpt-5.1-codex-mini` | hide | 272000 | yes | no | no | medium | unset | Cheaper, faster, less capable Codex model. |
| `gpt-5` | hide | 272000 | yes | no | no | medium | unset | Broad world knowledge with strong general reasoning. |
| `gpt-5-codex` | hide | 272000 | yes | no | no | medium | unset | Optimized for Codex. |
| `gpt-5-codex-mini` | hide | 272000 | yes | no | no | medium | unset | Cheaper, faster, less capable Codex model. |

Interpretation:
- `list` means the model is visible in the current Codex picker/catalog.
- `hide` means the model is still present in the account-local catalog, but not surfaced as a normal top-level picker choice.
- This is a Codex-visible model set, not a complete OpenAI platform-wide entitlement dump.

## Rate Limits

### What Is Confirmed From This Account

The latest Codex session telemetry exposes two rolling windows:

| Window | Minutes | Meaning | Current Usage | Reset Time UTC | Reset Time Australia/Brisbane |
| --- | ---: | --- | ---: | --- | --- |
| Primary | 300 | 5-hour rolling window | 2% used | 2026-04-09 18:07:50 UTC | 2026-04-10 04:07:50 AEST |
| Secondary | 10080 | 7-day rolling window | 13% used | 2026-04-14 23:37:26 UTC | 2026-04-15 09:37:26 AEST |

Other observed telemetry:
- `plan_type`: `plus`
- Latest session context window reported by Codex telemetry: `258400`

### Requested Windows

The account-local data does not expose all of the windows you asked for as absolute caps. The honest table is:

| Window | Status | Notes |
| --- | --- | --- |
| 5-minute | not exposed | No absolute 5-minute cap surfaced by local Codex auth or telemetry. |
| Hourly | not exposed | No absolute hourly cap surfaced by local Codex auth or telemetry. |
| Daily | not exposed | No absolute daily cap surfaced by local Codex auth or telemetry. |
| Monthly | not exposed | No absolute monthly cap surfaced by local Codex auth or telemetry. |
| 5-hour | confirmed | Exposed as Codex `primary` window: 300 minutes. |
| 7-day | confirmed | Exposed as Codex `secondary` window: 10080 minutes. |

### Operational Guidance

Use the telemetry as a burn-rate signal, not as a billing-grade quota table.

Practical implications:
- Watch the 5-hour window if you are running many `codex exec` jobs, long sessions, or multi-agent workflows.
- Watch the 7-day window if you are doing sustained automation through the week.
- Do not build automation that assumes fixed 5-minute, hourly, daily, or monthly hard limits unless you can query a Platform project with a real API key.

## Headless Codex Usage

`codex exec` is the supported non-interactive entrypoint.

Core patterns:

```bash
codex exec "Summarize this repository and propose the next three tasks"
```

```bash
printf '%s\n' "Review the staged diff for regressions" | codex exec -
```

```bash
codex exec -m gpt-5.4-mini --json "Explain the failing test and suggest a fix"
```

```bash
codex exec --output-last-message final.md "Write release notes from the git history"
```

```bash
codex exec --output-schema schema.json "Return a machine-readable deployment plan"
```

Useful flags from local CLI help:
- `-m, --model <MODEL>` selects the model.
- `--json` emits JSONL events for automation.
- `--output-schema <FILE>` constrains the final response shape.
- `-o, --output-last-message <FILE>` writes only the final assistant message to a file.
- `--ephemeral` avoids persisting the session to disk.
- `-C, --cd <DIR>` sets the working root.
- `--skip-git-repo-check` allows running outside a Git repository.
- `--full-auto` is a lower-friction automation mode.
- `--dangerously-bypass-approvals-and-sandbox` should be reserved for externally sandboxed environments.

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

Good headless rules:
- Ask for one artifact per run.
- Specify whether Codex may edit files, run tests, or only analyze.
- Name the exact files, directories, or diff scope when possible.
- Prefer `--output-schema` for downstream machine processing.
- Use `--json` when you need event streams, progress, or tool traces.
- Use `--ephemeral` for CI steps that should not leave state behind.
- Keep prompts specific and bounded. Ambiguous prompts create unstable pipelines.

## Chaining Patterns

### Pattern 1: Prompt In, Final Markdown Out

Use when the next step is human review.

```bash
codex exec \
  -m gpt-5.4 \
  -o review.md \
  "Review the current repository for deployment risks and write a short markdown report."
```

### Pattern 2: Prompt In, Structured JSON Out

Use when another tool will consume the result.

```bash
codex exec \
  -m gpt-5.4-mini \
  --output-schema ./schema.json \
  -o plan.json \
  "Return a release checklist for the current repo."
```

### Pattern 3: Git Diff To Codex

Use when the model should reason over current changes rather than the whole repository.

```bash
git diff --staged | codex exec --skip-git-repo-check - <<'EOF'
Review this staged diff.
Find bugs, regressions, and missing tests.
Return concise markdown.
EOF
```

### Pattern 4: JSONL Event Stream To Another Tool

Use when you want progress or machine-visible events.

```bash
codex exec --json "Generate a migration plan for this repo" | jq -c .
```

### Pattern 5: Multi-Step Shell Pipeline

Use when each step narrows ambiguity.

```bash
codex exec -o findings.md "Review the repo for API contract risks" &&
codex exec -o changelog.md "Draft release notes using findings.md and the current git history"
```

Chaining guidance:
- Persist intermediate artifacts to files, not just stdout.
- Prefer markdown for human checkpoints and JSON for machine checkpoints.
- Keep each run single-purpose.
- Make later steps consume explicit artifacts from earlier steps.
- Avoid long conversational state in automation; restate the task and inputs each time.

## Recommended Defaults

For most headless work:

```bash
codex exec \
  -m gpt-5.4-mini \
  --ephemeral \
  --json \
  "..."
```

Use `gpt-5.4` when:
- The task is high-stakes.
- The task is long-horizon.
- The cost of a wrong answer is higher than the cost of extra latency.

Use `gpt-5.4-mini` when:
- You want fast iteration.
- You are building a multi-step pipeline.
- You can validate or retry cheaply.

Use `gpt-5.3-codex` when:
- The task is specifically coding-heavy and agentic.
- You want a model tuned toward codebase work rather than general prose.

## CI And Cron Examples

### GitHub Actions

This pattern is suitable when the runner already has Codex authenticated.

```yaml
name: codex-review

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Codex
        run: npm install -g @openai/codex
      - name: Run Codex review
        run: |
          codex exec \
            -m gpt-5.4-mini \
            --ephemeral \
            --json \
            -o codex-review.md \
            "Review this repository for correctness, regressions, and missing tests."
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: codex-review
          path: codex-review.md
```

### Cron-Friendly Shell Script

```bash
#!/usr/bin/env bash
set -euo pipefail

repo_dir=/srv/my-repo
out_dir=/var/log/codex
mkdir -p "$out_dir"

cd "$repo_dir"

codex exec \
  -m gpt-5.4-mini \
  --ephemeral \
  -o "$out_dir/daily-summary.md" \
  "Summarize repository status, likely risks, and next actions."
```

### CI Best Practices

- Prefer `--ephemeral` in CI to avoid carrying state between jobs.
- Write final artifacts to files with `-o` instead of scraping terminal output.
- Use `--output-schema` for machine-consumed results.
- Use smaller models for fan-out jobs and larger models only on the narrow high-value steps.
- Treat `--dangerously-bypass-approvals-and-sandbox` as last resort only in already-isolated runners.

## Direct API Usage

This section is for direct OpenAI Platform API usage. It requires a real `OPENAI_API_KEY`. The current local Codex login is ChatGPT-based, so these examples are best-practice templates, not currently validated on this machine.

### Minimal Example

```bash
curl https://api.openai.com/v1/responses \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.4-mini",
    "input": "Summarize the repository at a high level."
  }'
```

### Structured Output Example

```bash
curl https://api.openai.com/v1/responses \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.4",
    "input": "Return a deployment checklist.",
    "text": {
      "format": {
        "type": "json_schema",
        "name": "deployment_checklist",
        "schema": {
          "type": "object",
          "properties": {
            "items": {
              "type": "array",
              "items": { "type": "string" }
            }
          },
          "required": ["items"],
          "additionalProperties": false
        }
      }
    }
  }'
```

### Direct API Best Practices

- Use structured output for machine-to-machine workflows.
- Keep prompts self-contained rather than relying on previous conversational state.
- Separate analysis and action into different calls when correctness matters.
- Make the model produce a single artifact per request.
- Add your own retry, timeout, and idempotency handling around the API.

## What To Recheck Later

Re-run this document if any of the following change:
- Codex version
- OpenAI account plan
- Login mode changes from ChatGPT auth to Platform API key
- OpenAI updates the model catalog
- OpenAI exposes additional rate-limit windows in Codex telemetry
