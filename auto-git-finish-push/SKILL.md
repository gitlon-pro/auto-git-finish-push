---
name: auto-git-finish-push
description: Automatically finish a completed coding or documentation task by checking Git safety gates, staging only task-owned changes, generating a commit message from the staged diff, committing, and pushing. Use when the user asks Codex to auto git add, commit, and push at task completion, or when project instructions require automatic finish-and-push behavior. Must fail closed on unsafe branches, dirty baselines, sensitive files, missing remotes, merge conflicts, or unclear change ownership.
---

# Auto Git Finish Push

## Goal

Finish a completed task by safely staging, summarizing, committing, and pushing the task-owned Git changes.

Fail closed. If any safety gate fails, do not commit and do not push.

## Start-of-task baseline

When this skill is active, capture the baseline before editing:

```sh
git status --short
git branch --show-current
git rev-parse --show-toplevel
```

Use this baseline to avoid staging user changes that existed before the task.

## Safety gates

Before staging or committing, verify:

1. Current directory is inside a Git repository.
2. No merge, rebase, cherry-pick, or bisect is in progress.
3. Current branch is not protected by default:
   - `main`
   - `master`
   - `production`
   - `release/*`
   - `hotfix/*`
4. Changed files do not include common secrets or local-only files:
   - `.env`
   - `.env.*`
   - `*.pem`
   - `*.key`
   - `*.p12`
   - `id_rsa`
   - `id_ed25519`
   - `.DS_Store`
5. If the worktree was dirty at task start, stage only files clearly changed by this task.
6. If change ownership is unclear, stop and report the exact files.
7. Never use `git push --force` or `git push --force-with-lease`.

## Workflow

1. Inspect current state:

```sh
git status --short
git diff --stat
git diff --name-status
```

2. If there are no changes, skip Git actions and report that no changes were detected.

3. Stage only task-owned changes:

```sh
git add -A
```

Use path-specific staging instead when the baseline was dirty:

```sh
git add -- <paths>
```

4. Inspect staged diff:

```sh
git diff --cached --stat
git diff --cached --name-status
git diff --cached
```

5. Generate the commit message from the staged diff, not from memory alone.

Use this format:

```text
type(scope): short imperative summary

- concrete change
- concrete change

Verification:
- command run, or "not run"
```

Choose `type` by actual change:

- `feat`: user-facing feature
- `fix`: bug fix
- `docs`: documentation
- `refactor`: behavior-preserving code change
- `test`: tests
- `chore`: tooling, config, maintenance

6. Commit with a message file:

```sh
git commit -F /tmp/auto-git-finish-push-message.txt
```

7. Push only with a normal push:

```sh
git push
```

If no upstream exists, push only when the user or project config allows it:

```sh
git push -u origin HEAD
```

Otherwise stop and report the manual command.

## Final response

After a successful push, report:

- commit hash;
- branch;
- remote target;
- commit message;
- verification commands run or not run;
- any files intentionally excluded.

If blocked, report:

- blocking reason;
- files involved;
- no commit or push was performed.
