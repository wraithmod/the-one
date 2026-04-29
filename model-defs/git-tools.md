# Git Tools

Last updated: 2026-04-30

Purpose: provide a concise, LLM-friendly Git workflow for this repository and for future agents working on transfer, sync, and publishing tasks.

## Repository Identity

Current project:

```text
Name: the-one
Local path: /home/wraith/the-one
Default branch: main
Remote: git@github.com:wraithmod/the-one.git
HTTPS URL: https://github.com/wraithmod/the-one.git
```

Use this as a separate project. Do not merge it into adjacent workspaces.

## First Checks

Always start Git work with:

```bash
git status --short --branch
git remote -v
git log --oneline --decorate -5
```

If the repo may have remote changes:

```bash
git fetch --all --prune
git status --short --branch
git log --oneline --decorate --graph --all -10
```

Interpretation:

- `## main...origin/main` with no ahead/behind count means local and remote agree.
- `ahead N` means local commits need pushing.
- `behind N` means remote commits need pulling.
- `ahead N, behind M` means histories diverged and need review before pushing.

## Sync Commands

Normal update before work:

```bash
git pull --rebase
```

Normal publish after a focused commit:

```bash
git push
```

Clone on another machine:

```bash
git clone https://github.com/wraithmod/the-one.git
```

Clone with SSH when the host has GitHub SSH auth:

```bash
git clone git@github.com:wraithmod/the-one.git
```

## Remote Setup

If a clone or initialized repo is missing `origin`:

```bash
git remote add origin git@github.com:wraithmod/the-one.git
git branch -M main
git push -u origin main
```

If HTTPS was added but push fails due to missing credentials, and SSH auth works:

```bash
ssh -T git@github.com
git remote set-url origin git@github.com:wraithmod/the-one.git
git push
```

Expected successful SSH auth message:

```text
Hi wraithmod! You've successfully authenticated, but GitHub does not provide shell access.
```

## Commit Workflow

Use focused commits:

```bash
git status --short
git diff
git add <paths>
git diff --staged
git commit -m "type: concise summary"
```

Preferred commit style is Conventional Commits:

- `docs: add windows 11 network install runbook`
- `chore: update git tooling notes`
- `fix: correct iVentoy setup path`
- `feat: add post-install privacy script`

Keep large binaries out of commits. This repo should not track ISOs, WIMs, VHDs, secrets, or generated logs.

## Safety Rules

- Do not run `git reset --hard`, `git checkout -- <path>`, or destructive cleanup unless the user explicitly asks.
- Do not rewrite remote history unless the user explicitly asks and the branch impact is understood.
- Do not commit secrets, license keys, account passwords, Wi-Fi credentials, or unattended XML files containing real passwords.
- Read `git status` before and after edits.
- When unrelated untracked files exist, leave them alone unless they are part of the requested task.
- Review staged diffs before committing.

## Useful Inspection Commands

Changed files:

```bash
git diff --name-status
git diff --staged --name-status
```

Recent history:

```bash
git log --oneline --decorate --graph -20
```

Show a commit:

```bash
git show --stat --oneline <commit>
git show --name-only <commit>
```

Find when a line changed:

```bash
git blame <path>
```

Check ignored files:

```bash
git status --ignored --short
git check-ignore -v <path>
```

## Transfer Pattern For The Windows Image Host

On the computer hosting the Windows 11 ISO:

```bash
git clone https://github.com/wraithmod/the-one.git
cd the-one
git pull --rebase
```

Before changing deployment files on that host:

```bash
git status --short --branch
git pull --rebase
```

After changing deployment files:

```bash
git add plans scripts .gitignore AGENTS.md model-defs
git diff --staged
git commit -m "docs: update windows install deployment notes"
git push
```

If the host cannot push, it can still pull updates and use the runbook. Commit and push from the primary workstation later.

## LLM Automation Pattern

For agent-assisted Git work, ask the model to return:

- current branch and remote state
- files changed
- intended commit scope
- risks or excluded files
- exact commands run
- final status

Good prompt shape:

```text
Goal: review the staged diff and propose a commit message.
Context: repo is /home/wraith/the-one, branch main, remote origin.
Output: bullets with summary, risks, and one Conventional Commit message.
Validation: mention if any staged file appears to contain secrets or large binaries.
```

For push/publish work, require the model to verify:

- local branch tracks the intended remote branch
- no unrelated files are staged
- large install media is ignored
- final `git status --short --branch` is reported

