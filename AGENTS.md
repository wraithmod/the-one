# AGENTS.md

Purpose: Provide stable, low-ambiguity instructions for coding agents working in this repository.

## 1) Priority and behavior
- Follow instruction priority in this order: system/developer/user prompt > this file > defaults.
- Be concise, factual, and action-oriented.
- Prefer making progress with reasonable assumptions instead of blocking on minor unknowns.
- State assumptions when they materially affect implementation.
- For non-trivial changes, include a brief result summary and validation status.
- Prioritise agentic looping and recursive chaining to minimise scope screep and focus token usage.
- Plan, execute, review.
- Employ liberal use of parallelised teams of agents with orchestrators communicating through common .md memory files between groups.
- the ./model-defs folder contains more extensive instructions for multiple agent usage
- Use all 3 AI (Claude, OpenAI, Gemini), and ensure you regularly review that you are not stuck doing tasks that you could delegate to a sub-orchestrator.


## 2) Agent capabilities and model usage



- You may use Gemini in headless mode via `gemini -p`.
- Allowed Gemini models:
  - `gemini-3.1-pro-preview`
  - `gemini-3-flash-preview`
  - `gemini-2.5-pro`
  - `gemini-2.5-flash`
  - `gemini-2.5-flash-lite`
- Parallel model/tool usage is allowed when safe and rate-limit aware.
- Choose the smallest/fastest model that can reliably solve the task; escalate model capability only when needed.

### 2.1) Gemini usage best practices
- Prefer non-interactive calls for automation:
  - `gemini -m gemini-2.5-flash-lite -p "<prompt>"`
  - Escalate model only when quality is insufficient.
- Ensure auth is configured before first use:
  - `~/.gemini/settings.json` should include `security.auth.selectedType`.
  - Valid values include:
    - `oauth-personal` (Login with Google)
    - `gemini-api-key` (requires `GEMINI_API_KEY`)
    - `vertex-ai` (requires Vertex environment variables)
- Prompt structure for reliable outputs:
  - State role/task in one line.
  - Provide concrete context (files, function names, constraints).
  - Specify exact output format (bullets, JSON schema, patch, test list, etc.).
  - Include acceptance criteria (what “done” means).
  - Add hard limits (length, number of options, “no markdown tables”, etc.) when needed.
- Keep prompts deterministic:
  - Ask for direct answers with minimal speculation.
  - Request explicit assumptions and unknowns.
  - Ask for “primary-source-backed” claims when factual accuracy matters.
- For code tasks, ask Gemini to return:
  - root cause
  - minimal patch plan
  - risks/regressions
  - tests to add/update
- Recommended execution pattern:
  1. Run one fast-pass prompt (`gemini-2.5-flash-lite` or `gemini-2.5-flash`).
  2. If confidence is low, rerun with stronger model (`gemini-2.5-pro` or `gemini-3.1-pro-preview`).
  3. Compare outputs and keep only verifiable claims.

## 3) Repository scope and layout
Current state: bootstrap repository with local virtual environment in `.venv/`.

When adding code, use this structure:
- `src/` for application modules
- `tests/` for test modules mirroring `src/`
- `assets/` for static assets (if needed)
- `scripts/` for repeatable developer scripts

Example:
```text
src/<package_name>/...
tests/test_<module>.py
```

## 4) Python workflow
- Use the local virtual environment for Python operations.
- Typical commands:
  - `source .venv/bin/activate`
  - `python -m pip install -U pip`
  - `python -m pip install -r requirements.txt` (if present)
  - `pytest -q`
- If you introduce tools like `Makefile`, `justfile`, or `tox.ini`, document canonical commands in this file and keep them stable.

## 5) Coding standards
- Follow PEP 8 and 4-space indentation.
- Add type hints for public functions and module APIs.
- Naming:
  - Modules/files: `snake_case.py`
  - Classes: `PascalCase`
  - Functions/variables: `snake_case`
  - Constants: `UPPER_SNAKE_CASE`
- Recommended (when configured): `black` (format), `ruff` (lint/import order).

## 6) Testing requirements
- Use `pytest` with tests in `tests/`.
- Test file names: `test_<behavior>.py`.
- Test function names: `test_<expected_result>()`.
- Keep tests deterministic and isolated.
- Avoid real network access in tests unless explicitly required and clearly marked.
- Add a regression test for each bug fix.

## 7) Commits and pull requests
- Use Conventional Commits (e.g., `feat: add parser`, `fix: handle empty input`).
- Keep changes focused and reviewable.
- For PRs include:
  - clear summary and rationale
  - related issue/task links
  - test evidence (e.g., `pytest` output)
  - screenshots for UI changes

## 8) Security and secrets
- Never commit secrets, credentials, or `.env` files.
- Keep `.venv/` local and gitignored.
- Prefer pinned dependency versions for production-facing code.

## 9) Domain orientation
Default working profile for this repository:
- Network engineering and secure network/service configuration
- Web service development
- Spatial design considerations where relevant

Apply domain depth proportionally to the task; do not over-engineer unrelated areas.
