# Auto Git Finish Push

[中文说明](README.zh-CN.md)

Skill for AI coding assistants and AI editors to safely finish a task with Git: inspect changes, stage only task-owned files, generate a commit message from the staged diff, commit, and push.

Codex can install it as a native skill. Other AI editors can reuse the same instructions if they support skills, rules, instructions, or project-level agent guidance.

The skill is intentionally conservative. It stops instead of committing when branch safety, change ownership, secrets, conflict state, or remote state is unclear.

## Install

Install with [skills.sh](https://skills.sh/). Use the repository root URL only:

```sh
npx skills add https://github.com/5046312/auto-git-finish-push -a codex -g -y
```

The command above uses Codex as the example agent. For other AI editors, replace `-a codex` with that tool's agent name in `skills.sh`; the repository URL stays the same.

If your Codex environment exposes the `skill-installer` skill, you can ask Codex:

```text
Use $skill-installer to install https://github.com/5046312/auto-git-finish-push
```

Restart the relevant AI editor or session after installation so the skill or rule can be discovered.

## Usage

Use it explicitly at the end of a task in Codex or any environment that supports `$skill-name` triggers:

```text
Use $auto-git-finish-push to commit and push this task.
```

For AI editors that do not support `$auto-git-finish-push` triggers, ask the assistant to read and follow `auto-git-finish-push/SKILL.md` directly:

```text
Before the final response, follow auto-git-finish-push/SKILL.md to safely stage, commit, and push only the current task changes.
```

Or add it to your project instructions. Codex can use `AGENTS.md`; other AI editors can use their equivalent rules, instructions, or project rules file:

```text
At the end of every coding or documentation task, before the final response, use $auto-git-finish-push to safely stage, commit, and push task-owned changes.
```

The skill will inspect Git state first. If the repository is on a protected branch, has unclear pre-existing changes, contains sensitive files, or cannot push safely, it will stop and report the blocker instead of committing.

## Manual Install

If you do not use the installer, you can copy the skill folder manually. Codex example:

```sh
git clone https://github.com/5046312/auto-git-finish-push.git
mkdir -p ~/.codex/skills
cp -R auto-git-finish-push/auto-git-finish-push ~/.codex/skills/auto-git-finish-push
```

For other AI editors, place `auto-git-finish-push/SKILL.md` in that tool's global rules, project rules, or custom skill directory. If the tool cannot install skills directly, reference this file from its project instructions.

Then restart the relevant AI editor or session.

## Skill Path

The installable skill lives in:

```text
auto-git-finish-push/
├── SKILL.md
└── agents/
    └── openai.yaml
```

## What It Does

- Captures the Git baseline before work starts.
- Refuses protected branches such as `main`, `master`, `production`, `release/*`, and `hotfix/*`.
- Refuses common secret or local-only files such as `.env`, private keys, and `.DS_Store`.
- Stages only task-owned changes when the worktree was already dirty.
- Builds the commit message from `git diff --cached`.
- Uses normal `git push` only.
- Never uses force push.

## When It Stops

The skill does not commit or push when:

- no Git repository is detected;
- merge, rebase, cherry-pick, or bisect is active;
- current branch is protected;
- changed files look sensitive or local-only;
- task-owned files cannot be separated from pre-existing user changes;
- no upstream exists and project rules do not allow `git push -u origin HEAD`;
- push would require force.
