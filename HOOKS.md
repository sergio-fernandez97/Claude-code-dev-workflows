# Hooks

[Hooks reference](https://code.claude.com/docs/en/hooks)

# Prompt

> Help me create the claude hooks necessary for the repo `./full-stack-fastapi-template` described in @README.md section 7. Then fill a file named HOOKS.md explaining the structure of the configuration and each item, be explicit. Also include at the beging the this prompt.

---

# Claude Hooks Configuration for `full-stack-fastapi-template`

This document explains the Claude Code hooks configured for the `full-stack-fastapi-template` repository, as described in **@README.md section 7: Configure Hooks as Guardrails**.

---

## File Location

The hooks configuration lives at:

```
full-stack-fastapi-template/.claude/settings.json
```

The actual hook shell scripts are stored in:

```
full-stack-fastapi-template/.claude/hooks/
```

Claude Code loads project-level settings from `.claude/settings.json` inside the repository root. These settings apply whenever Claude Code is opened in this directory.

---

## Configuration Structure Overview

The `settings.json` follows the Claude Code settings schema. The top-level key used for hooks is `"hooks"`, which contains event keys. Each event key maps to an array of hook objects. Every hook object must define:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | A short, unique identifier for the hook. Used in logs and output. |
| `command` | string | Yes | A shell command string executed when the hook fires. |
| `timeout` | integer (ms) | No | Maximum time allowed for the hook command in milliseconds. Defaults vary by event. |

### Supported Hook Events

| Event | When it fires | Typical use |
|-------|---------------|-------------|
| `beforeUserPromptSubmit` | Before the user’s message is sent to Claude. | Show reminders, print scope rules, confirm context. |
| `beforeAssistantPromptSubmit` | Before Claude’s response is sent to the user. | Rarely used; can intercept or annotate assistant output. |
| `afterToolUse` | After any tool (Read, Edit, Write, Bash, etc.) is executed. | Validate side effects, run checks, enforce guards. |
| `afterAssistantPromptSubmit` | After the assistant finishes its turn. | Summary hooks, cleanup, or post-conversation logging. |

In this configuration we use two events:

1. **`beforeUserPromptSubmit`** — to remind the user and Claude about the allowed scope before any work begins.
2. **`afterToolUse`** — to inspect the actual changes made by tools and run validation.

---

## Hook Scripts Directory

Each hook is implemented as an **executable shell script** inside `.claude/hooks/`. This separation has several benefits:

- **Readability:** Shell logic is easier to read, review, and maintain in standalone files than inside JSON-escaped strings.
- **Version control:** The scripts themselves are tracked by Git, so changes to hook logic produce normal diffs.
- **Testability:** You can run each script manually from the terminal to verify behavior before Claude Code executes it.
- **IDE support:** Editors provide syntax highlighting and linting for `.sh` files but not for inline JSON strings.

### Directory layout

```
full-stack-fastapi-template/.claude/
├── settings.json
└── hooks/
    ├── scope-reminder.sh
    ├── scope-guard.sh
    └── backend-validation.sh
```

All scripts in `hooks/` are executable (`chmod +x`).

---

## Hook 1: `scope-reminder` (`beforeUserPromptSubmit`)

### Purpose
Print a visible reminder at the start of every user turn so that the allowed file paths are always top-of-mind.

### settings.json entry
```json
{
  "name": "scope-reminder",
  "command": "bash .claude/hooks/scope-reminder.sh",
  "timeout": 5000
}
```

### Script: `.claude/hooks/scope-reminder.sh`
```bash
#!/usr/bin/env bash
set -e

echo "[HOOK: scope-reminder] Allowed paths: backend/app/** and backend/tests/**"
```

### How it works
- Every time the user submits a prompt, this hook runs **before** Claude processes it.
- The `settings.json` entry invokes `bash .claude/hooks/scope-reminder.sh`.
- The script executes a simple `echo` command that prints:
  ```
  [HOOK: scope-reminder] Allowed paths: backend/app/** and backend/tests/**
  ```
- This is a lightweight, read-only reminder. It does not block or fail.
- The `timeout` of `5000` ms (5 seconds) is generous for a simple echo; it prevents the hook from hanging the session if something unexpected happens.

### Why this matters
In a workshop or team setting, it is easy to drift into unrelated files (e.g., editing `frontend/`, `compose.yml`, or `README.md`). This hook makes the scope boundary explicit **before** work starts, reducing the chance of accidental scope creep.

---

## Hook 2: `scope-guard` (`afterToolUse`)

### Purpose
Detect when files outside the allowed scope (`backend/app/**` and `backend/tests/**`) have been modified and warn the user immediately.

### settings.json entry
```json
{
  "name": "scope-guard",
  "command": "bash .claude/hooks/scope-guard.sh",
  "timeout": 10000
}
```

### Script: `.claude/hooks/scope-guard.sh`
```bash
#!/usr/bin/env bash
set -e

cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"

CHANGED=$(git diff --name-only 2>/dev/null \
    | grep -v "^backend/app/" \
    | grep -v "^backend/tests/" \
    | grep -v "^\.claude/" \
    | grep -v "^\.git" \
    | grep -v "^HOOKS\.md$" \
    || true)

if [ -n "$CHANGED" ]; then
    echo "[HOOK: scope-guard] WARNING: Changes detected outside allowed scope:"
    echo "$CHANGED"
    echo "Allowed: backend/app/**, backend/tests/**"
fi
```

### How it works
1. **`cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"`**
   Moves to the repository root so that `git diff` paths are consistent. Falls back to the current directory if `git rev-parse` fails.

2. **`git diff --name-only`**
   Lists every file that has uncommitted changes.

3. **`| grep -v "^backend/app/" | grep -v "^backend/tests/"`**
   Removes changes that are inside the two allowed directories.

4. **`| grep -v "^\.claude/" | grep -v "^\.git" | grep -v "^HOOKS\.md$"`**
   Removes expected side-effect paths:
   - `.claude/` — Claude Code’s own metadata directory.
   - `.git/` — Git internals (should never appear in `diff --name-only`, but included for safety).
   - `HOOKS.md` — This documentation file itself.

5. **`|| true`**
   Ensures the pipeline returns success even when no out-of-scope files are found. This prevents the hook from appearing to fail.

6. **`if [ -n "$CHANGED" ]; then ... fi`**
   Only prints the warning when there is actual out-of-scope content.

### Output example
If someone edits `frontend/src/main.tsx`, the hook prints:
```
[HOOK: scope-guard] WARNING: Changes detected outside allowed scope:
frontend/src/main.tsx
Allowed: backend/app/**, backend/tests/**
```

### Why this matters
This is a runtime guardrail. Unlike the reminder (which is preventive), the guard is **detective**. It audits the actual filesystem state after every tool use and makes scope violations visible immediately, while they are still easy to revert.

---

## Hook 3: `backend-validation` (`afterToolUse`)

### Purpose
Run fast, targeted validation on backend Python files whenever they are modified.

### settings.json entry
```json
{
  "name": "backend-validation",
  "command": "bash .claude/hooks/backend-validation.sh",
  "timeout": 30000
}
```

### Script: `.claude/hooks/backend-validation.sh`
```bash
#!/usr/bin/env bash
set -e

cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"

if git diff --name-only 2>/dev/null | grep -q "^backend/.*\.py$"; then
    echo "[HOOK: backend-validation] Running ruff check on backend..."
    cd backend && ruff check app tests \
        || echo "[HOOK: backend-validation] ruff check found issues."
fi
```

### How it works
1. **Repository root detection** — same `git rev-parse` pattern as Hook 2.

2. **`git diff --name-only | grep -q "^backend/.*\.py$"`**
   Checks whether any file under `backend/` ending in `.py` has uncommitted changes. The `-q` flag makes the check silent; we only care about the exit status.

3. **`if ...; then ... fi`**
   The actual lint command runs **only** when a Python file in the backend was touched. If the user edits `frontend/` files or documentation, nothing happens.

4. **`cd backend && ruff check app tests`**
   Runs the `ruff` linter on:
   - `app/` — the application source code.
   - `tests/` — the test suite.
   This mirrors the linting commands already defined in the repository (`backend/scripts/lint.sh`).

5. **`|| echo "[HOOK: backend-validation] ruff check found issues."`**
   If `ruff` reports errors, the hook prints a message but **does not fail the session**. In a teaching/demo context, a warning is often more useful than a hard block. If you want the hook to be blocking, remove the `|| ...` fallback.

### Output example
When `backend/app/api/routes/items.py` is edited:
```
[HOOK: backend-validation] Running ruff check on backend...
```
If `ruff` finds no issues, no further output appears. If it finds issues, you see:
```
[HOOK: backend-validation] ruff check found issues.
```

### Why this matters
Validation should happen as close to the change as possible. By attaching this to `afterToolUse`, the user gets immediate feedback after every edit instead of discovering lint failures later in GitHub Actions. The conditional check (`grep -q`) ensures we do not waste time running `ruff` when the change had nothing to do with backend Python code.

---

## Relationship to README.md Section 7

The README specifies two recommended demo hooks:

| README Requirement | Implementation |
|--------------------|----------------|
| **Scope guard** — Allow changes only in `backend/app/**` and `backend/tests/**` | `scope-reminder` (preventive) + `scope-guard` (detective) |
| **Post-change validation** — Run targeted validation after edits | `backend-validation` (runs `ruff check` conditionally) |

The configuration is intentionally layered:

- **Reminder** → prevents mistakes by keeping scope visible.
- **Guard** → detects mistakes by inspecting actual diffs.
- **Validation** → confirms quality by running fast lint checks.

---

## Customization Notes

### Making the scope-guard blocking
If you want the `scope-guard` hook to **stop** the session when out-of-scope files are changed, edit `.claude/hooks/scope-guard.sh` and replace the `echo` lines with `exit 1`:

```bash
...; then echo "..."; exit 1; fi
```

**Caution:** A blocking guard can be disruptive if you genuinely need to update config files (e.g., `pyproject.toml`). Use blocking mode only when the scope rule is absolute.

### Expanding validation
To also run tests automatically, edit `.claude/hooks/backend-validation.sh` and append the test command. For example:

```bash
cd backend && pytest tests/api/routes/test_items.py || echo "Tests failed."
```

Keep in mind that tests in this repository may require a running database or Docker Compose, so local `pytest` might not succeed in every environment.

### Adjusting timeouts
Timeouts are declared in `settings.json` per hook entry:

- `scope-reminder`: `5000` ms is more than enough for an `echo`.
- `scope-guard`: `10000` ms allows for large diffs in big repositories.
- `backend-validation`: `30000` ms gives `ruff` room to scan many files.

If the repository grows, increase the `timeout` value in `settings.json` accordingly. You do not need to modify the shell scripts themselves.

---

## Quick Reference: Files in This Setup

| File | Purpose |
|------|---------|
| `full-stack-fastapi-template/.claude/settings.json` | Active hooks configuration loaded by Claude Code |
| `full-stack-fastapi-template/.claude/hooks/scope-reminder.sh` | Preventive scope reminder (runs before every prompt) |
| `full-stack-fastapi-template/.claude/hooks/scope-guard.sh` | Detective scope guard (runs after every tool use) |
| `full-stack-fastapi-template/.claude/hooks/backend-validation.sh` | Post-change lint validation (runs after every tool use) |
| `full-stack-fastapi-template/backend/scripts/lint.sh` | Reference script: `mypy`, `ty`, `ruff check`, `ruff format --check` |
| `full-stack-fastapi-template/backend/scripts/test.sh` | Reference script: `coverage run -m pytest tests/` |
| `full-stack-fastapi-template/.pre-commit-config.yaml` | Repository pre-commit hooks (separate from Claude Code hooks) |
