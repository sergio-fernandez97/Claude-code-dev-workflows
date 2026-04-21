# Session 3: From GitHub Issue to Tested Feature with Claude Code

This live session shows how to use Claude Code as part of a realistic software development workflow on top of an existing production-style repository. The goal is not to generate code from scratch, but to demonstrate how Claude Code can help read a requirement, analyze an unfamiliar codebase, work safely in isolation, delegate tasks, validate implementation decisions, run tests, and produce a PR-ready outcome.

During the session, the instructor will work against the repository `fastapi/full-stack-fastapi-template` and simulate the full path from issue intake to tested implementation. The main feature example is a small backend enhancement such as adding filtering, pagination, validation, and tests to an existing endpoint. The exact acceptance criteria can be adjusted before the session, but the flow should remain the same.

## Session Goal

By the end of the live session, participants should understand how to:

1. Start from a GitHub issue or feature request.
2. Explore and summarize an existing codebase safely.
3. Use Claude Code worktrees to isolate implementation work.
4. Delegate specialized tasks to subagents.
5. Use Context7 only when documentation lookup is actually needed.
6. Enforce quality boundaries with hooks.
7. Validate the result with tests and GitHub Actions.
8. Produce a PR-ready summary of the work.

## What Will Be Done

The instructor will:

1. Present a realistic feature request as if it came from GitHub.
2. Open the target repository and explain the relevant architecture.
3. Create an isolated Claude Code worktree for the feature.
4. Delegate parts of the work to subagents with clearly separated roles.
5. Use Context7 for targeted framework and testing lookups.
6. Configure or demonstrate hooks that constrain scope and enforce validation.
7. Implement the feature in the backend.
8. Add or update tests.
9. Run local validation commands.
10. Show how GitHub Actions audits the key checkpoints.
11. End with a PR-style summary and next steps.

## Prerequisites

### Instructor prerequisites

- Access to Claude Code with:
  - worktree support
  - subagents
  - hooks
  - MCP access
- Access to the GitHub MCP server
- Access to the Context7 MCP server
- Git installed locally
- Docker and Docker Compose available if the repository setup requires containers
- Python, Node.js, and any repository-specific dependencies needed by `fastapi/full-stack-fastapi-template`
- A local clone of this workshop repository
- A local clone or prepared copy of `fastapi/full-stack-fastapi-template`
- A GitHub repository where GitHub Actions can run

### Student prerequisites

- Basic familiarity with Git, Python, and terminal usage
- Basic understanding of backend APIs and tests
- Claude Code installed or available in the environment used for the workshop
- GitHub account with access to create a branch or fork a repository

### Instructor confirmation checklist

Before the live session starts, verify:

- Claude Code can open in the target repository.
- `claude --worktree` works in the current environment.
- The GitHub MCP server can read repository and issue context.
- Context7 can resolve and query documentation.
- The local test command for the target feature is known and working.
- GitHub Actions is enabled in the demo repository.

If any of these items are not confirmed ahead of time, mark them in the live session as instructor fill-ins instead of improvising.

## Claude Code Features and Tools Used

The session should explicitly call out the following features and tools:

### Claude Code features

- Main coding assistant session
- Worktrees via `claude --worktree`
- Subagents for delegation
- Hooks for scope and quality enforcement
- Diff review and PR-style summarization

### MCP tools

- GitHub MCP server
  - read issue context
  - inspect repository state
  - support PR-ready output
- Context7 MCP server
  - retrieve authoritative framework references
  - confirm FastAPI, Pydantic, or pytest usage patterns

### External development tools

- `git`
- `pytest`
- optional lint and format commands
- GitHub Actions

## Session Structure

Total duration: approximately 60 to 90 minutes.

Suggested pacing:

1. Introduction and problem framing: 10 minutes
2. Repository exploration: 10 to 15 minutes
3. Isolated setup with worktrees: 10 minutes
4. Delegation and implementation: 15 to 20 minutes
5. Testing and GitHub Actions audit: 10 to 15 minutes
6. Wrap-up and assignment: 10 minutes

## Step-by-Step Live Session Guide

### 1. Introduce the Goal and the Repository

State clearly that this is a real workflow demonstration, not a toy example.

Suggested explanation:

> In this session we will take a realistic issue, inspect a production-style codebase, isolate the work in a Claude Code worktree, delegate tasks to subagents, validate decisions with documentation only when needed, enforce scope with hooks, run tests, and produce a PR-ready result.

Then present the target repository:

```bash
cd full-stack-fastapi-template
```

Explain why this repo is useful for the demo:

- backend API
- frontend application
- authentication and users
- tests
- realistic project structure

Action:

- Show the repository tree at a high level.
- Highlight the backend paths where the demo will focus.

Example commands:

```bash
pwd
git status
find backend -maxdepth 3 -type f | head -n 30
find frontend -maxdepth 3 -type f | head -n 20
```

Instructor note:

- If you want a more controlled demo, prepare a specific issue in advance and display it here.

### 2. Present the GitHub Issue or Feature Request

The session should start from a requirement framed as a GitHub issue.

The issue should describe a small but realistic backend enhancement. A good example is:

- add filtering to an endpoint
- add pagination parameters
- validate invalid query values
- add or update tests

Action:

- Read the issue aloud.
- Extract explicit acceptance criteria.
- Write down non-goals.

Suggested acceptance criteria template:

1. Endpoint supports query filtering.
2. Endpoint supports pagination parameters.
3. Invalid input returns the expected validation behavior.
4. Existing patterns in the repository are preserved.
5. Automated tests cover the new behavior.

Instructor fill-in:

- `[Instructor: insert the exact issue URL, issue number, or copied issue text used in the session.]`

If GitHub MCP is available, explain that the issue context can be read through MCP rather than manually copying everything.

### 3. Explore the Codebase Before Changing Anything

This step is important because participants should see that Claude Code is used to understand a codebase before editing it.

Action:

1. Ask Claude Code to summarize the relevant architecture.
2. Identify the API route file.
3. Identify related models, schemas, or validation logic.
4. Identify the test files that should be updated.

Suggested exploration targets in this repository:

- `backend/app/api/routes/`
- `backend/app/models.py`
- `backend/app/crud.py`
- `backend/tests/api/routes/`

Suggested prompts for the live session:

- "Summarize the backend architecture relevant to this endpoint."
- "Identify the files involved in adding filtering and pagination to this route."
- "List the tests that should change if we implement this feature."

Expected teaching point:

- Claude Code should help narrow scope before implementation starts.

### 4. Create an Isolated Claude Code Worktree

This is one of the main teaching moments of the session.

Run:

```bash
claude --worktree feature-search-pagination
```

Explain what this does:

- creates a dedicated git branch
- creates an isolated working directory
- starts Claude Code in that isolated environment

Explain why this matters:

- avoids accidental edits in the main working tree
- makes the feature self-contained
- supports parallel work

Also show the no-name version:

```bash
claude --worktree
```

Instructor fill-in:

- `[Instructor: confirm the exact local path where Claude stores worktrees in the installed version used for the workshop. The draft assumes .claude/worktrees/<name>/, but this should be verified live or updated here beforehand.]`

Action:

1. Show the current branch before the command.
2. Run the worktree command.
3. Show the new branch and directory after the command.

Suggested verification commands:

```bash
git branch --show-current
pwd
git worktree list
git status
```

### 5. Define Subagent Roles

The purpose of this step is to show that Claude Code can divide work into specialized roles instead of using one long monolithic prompt.

Recommended roles:

1. Planner
2. Implementer
3. Tester
4. Reviewer

Suggested responsibilities:

- Planner
  - interpret the issue
  - define scope
  - identify files to edit
- Implementer
  - modify the code
  - preserve repository conventions
  - avoid unrelated changes
- Tester
  - add or update automated tests
  - identify edge cases
- Reviewer
  - inspect the result
  - check for regressions or missing coverage

Action:

- Explain each role before delegation.
- Show at least one concrete delegation example in the live session.

Instructor fill-in:

- `[Instructor: replace this section with the exact subagent invocation syntax supported by the Claude Code version used during the workshop if you want to show commands instead of describing the interaction conceptually.]`

### 6. Use Context7 Selectively

This step demonstrates disciplined documentation lookup.

Use Context7 only when a question cannot be answered confidently from the local codebase.

Good examples:

- FastAPI query parameter patterns
- pagination parameter conventions
- Pydantic validation behavior
- pytest fixture or test style examples

Bad example:

- querying documentation for patterns that are already obvious from the repository itself

Action:

1. Identify a concrete uncertainty.
2. Query Context7 for that exact point.
3. Apply the result conservatively.

Teaching point:

- Documentation lookup should support implementation, not replace reasoning.

Instructor fill-in:

- `[Instructor: insert one concrete Context7 lookup you want to demo, for example a FastAPI query parameter example or a pytest validation pattern.]`

### 7. Configure Hooks as Guardrails

Hooks should be presented as workflow constraints, not as decoration.

Recommended demo hooks:

### Hook 1: Scope guard

Allow changes only in:

- `backend/app/**`
- `backend/tests/**`

This makes it obvious when the assistant drifts outside the intended scope.

### Hook 2: Post-change validation

Run targeted validation after edits, such as:

- backend tests
- lint or format checks if appropriate

Action:

1. Explain the purpose of the hooks.
2. Show the configuration or pseudo-configuration.
3. Trigger behavior at least once if possible.

Instructor fill-in:

- `[Instructor: insert the exact hook configuration syntax and command used by the Claude Code version in your environment.]`
- `[Instructor: if hook support is not stable in your environment, present this section as a conceptual design and show the equivalent validation commands manually.]`

### 8. Implement the Feature

At this point the instructor should already have:

- a defined issue
- a scoped plan
- a worktree
- a set of subagent roles
- any necessary documentation lookup
- hook constraints or an equivalent validation plan

Now implement the feature in the backend.

Typical files likely involved in this repository:

- `backend/app/api/routes/items.py`
- `backend/app/crud.py`
- `backend/tests/api/routes/test_items.py`

Action:

1. Update the route or query logic.
2. Add filtering and pagination parameters.
3. Validate invalid inputs using the repository's existing conventions.
4. Preserve existing naming, response, and test patterns.

Suggested live narration:

> We are not trying to invent a new architecture. We are extending the existing design in the smallest coherent way.

Instructor fill-in:

- `[Instructor: replace the example file list above if you choose a different endpoint for the live session.]`

### 9. Run Local Validation

This step should be explicit and visible.

The audience should see that changes are validated, not just assumed to work.

At minimum, run targeted tests for the affected backend area.

Possible commands in this repository:

```bash
cd backend
pytest tests/api/routes/test_items.py
```

If the repository requires containerized commands, use the appropriate local equivalent instead.

Possible alternative commands:

```bash
./scripts/test.sh
./scripts/test-local.sh
```

Optional additional validation:

```bash
./scripts/lint.sh
./scripts/format.sh
```

Instructor fill-in:

- `[Instructor: confirm the exact test command that is known to pass in the prepared environment and replace the examples above if necessary.]`
- `[Instructor: if Docker Compose is required for the live demo, document the startup command here.]`

### 10. Audit the Main Steps with GitHub Actions

The session should make it clear that the workflow is not complete until CI checks the important parts.

The GitHub Actions workflow for this session should audit these checkpoints:

1. The change is restricted to the intended backend paths.
2. The relevant tests run successfully.
3. Optional lint or format checks pass.
4. A summary artifact or step log makes the session checkpoints visible.

Recommended audit points:

- Step 1: repository checkout
- Step 2: dependency setup
- Step 3: backend test execution
- Step 4: path or diff guard
- Step 5: optional linting
- Step 6: final workflow summary

Suggested workflow responsibilities:

- fail if files outside allowed paths were modified
- fail if backend tests do not pass
- record which session checkpoint failed

Instructor fill-in:

- `[Instructor: add the exact workflow filename, trigger, and status badge used for this workshop repository.]`
- `[Instructor: if you want the workflow to validate path scope, add a job or script that inspects git diff and fails on unauthorized paths.]`

Suggested explanation to students:

> Claude Code helps us move faster, but GitHub Actions remains the external audit layer that verifies the result independently of the local session.

### 11. Inspect the Diff and Produce a PR-Ready Summary

Before closing the session:

1. Show the changed files.
2. Review the diff briefly.
3. Summarize the implementation in PR language.

Suggested commands:

```bash
git status
git diff --stat
git diff
```

Suggested PR summary structure:

1. What changed
2. Why it changed
3. Tests added or updated
4. Known limitations

Instructor fill-in:

- `[Instructor: if you want to end with an actual draft PR, add the exact GitHub or MCP-based action here.]`

### 12. Wrap Up the Main Takeaway

Close with the core message of the session:

> Claude Code is not only useful for generating code. It is useful for orchestrating a safe, auditable, real development workflow.

Reinforce the four main ideas:

1. isolate with worktrees
2. delegate with subagents
3. validate with tests and hooks
4. audit with GitHub Actions

## Commands Summary

These are the main commands that should appear during the session. Replace any command that differs in your prepared environment.

```bash
cd full-stack-fastapi-template
git status
git branch --show-current
claude --worktree feature-search-pagination
git worktree list
pwd
find backend -maxdepth 3 -type f | head -n 30
cd backend
pytest tests/api/routes/test_items.py
git diff --stat
git diff
```

Optional commands depending on environment:

```bash
./scripts/test.sh
./scripts/test-local.sh
./scripts/lint.sh
./scripts/format.sh
docker compose up -d
```

## Live Session Checklist

Use this checklist during the session:

- The issue is visible and acceptance criteria are explicit.
- The repository is explored before editing.
- The feature is implemented inside a Claude worktree.
- At least one delegation example is shown.
- At least one Context7 lookup is shown.
- Hooks or equivalent guardrails are discussed and demonstrated.
- Tests are run locally.
- GitHub Actions is shown as the audit layer.
- A PR-style summary is produced at the end.

## Student Practical Assignment

Each student should reproduce the same workflow on a small variation of the live task.

### Assignment goal

Students must take a small backend requirement, implement it in isolation with Claude Code, validate it with tests, and summarize it as if preparing a pull request.

### Assignment instructions

1. Choose an existing backend endpoint in `fastapi/full-stack-fastapi-template` or in another approved repository.
2. Define a small feature request with 3 to 5 acceptance criteria.
3. Create a dedicated Claude Code worktree for the task.
4. Ask Claude Code to identify the files relevant to the change before editing anything.
5. Use at least one subagent or delegated role.
6. Use Context7 only if you hit a real uncertainty.
7. Keep the change scoped to the minimum necessary files.
8. Add or update automated tests.
9. Run the relevant test command locally.
10. Produce a short PR summary with:
   - what changed
   - why it changed
   - how it was tested
   - what remains uncertain

### Suggested student feature ideas

- add a new query filter to an existing endpoint
- add pagination parameters to a list endpoint
- add a validation rule for invalid input
- add one missing negative test case to an existing route
- improve response handling for an edge case already covered by the codebase patterns

### Required student deliverables

Each student should submit:

1. The issue statement or feature request they worked from.
2. The Claude worktree name they used.
3. The list of files they changed.
4. The test command they ran.
5. A short PR-style summary.

### Evaluation rubric

Assess the work using these questions:

1. Was the scope defined before implementation began?
2. Was the work isolated in a worktree?
3. Was the change limited to relevant files?
4. Were repository conventions preserved?
5. Were tests added or updated?
6. Was the final summary clear and reviewable?

## Instructor Fill-Ins to Complete Before the Session

Replace the placeholders in this document before teaching:

- exact issue URL or issue text
- exact worktree storage path if it differs from the draft assumption
- exact subagent invocation syntax if it will be demonstrated
- exact hook configuration syntax if hooks will be demonstrated live
- exact Context7 lookup example
- exact local test command
- exact GitHub Actions workflow filename and trigger
- exact PR creation step if the session includes opening a draft PR
