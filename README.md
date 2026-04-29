# Auto Git Finish Push

[中文说明](README.zh-CN.md)

Codex skill for safely finishing a task with Git: inspect changes, stage only task-owned files, generate a commit message from the staged diff, commit, and push.

The skill is intentionally conservative. It stops instead of committing when branch safety, change ownership, secrets, conflict state, or remote state is unclear.

## Install

Install with [skills.sh](https://skills.sh/):

```sh
npx skills add https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push -a codex -g -y
```

Install from GitHub with the Codex skill installer:

```sh
install-skill-from-github.py --repo gitlon-pro/auto-git-finish-push --path auto-git-finish-push
```

You can also install from the GitHub tree URL:

```sh
install-skill-from-github.py --url https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push
```

If your Codex environment exposes the `skill-installer` skill, you can ask Codex:

```text
Use $skill-installer to install https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push
```

Restart Codex after installation so the skill can be discovered.

## Usage

Use it explicitly at the end of a task:

```text
Use $auto-git-finish-push to commit and push this task.
```

Or add it to your project instructions, for example in `AGENTS.md`:

```text
At the end of every coding or documentation task, before the final response, use $auto-git-finish-push to safely stage, commit, and push task-owned changes.
```

The skill will inspect Git state first. If the repository is on a protected branch, has unclear pre-existing changes, contains sensitive files, or cannot push safely, it will stop and report the blocker instead of committing.

## Manual Install

If you do not use the installer, copy the skill folder into your Codex skills directory:

```sh
git clone https://github.com/gitlon-pro/auto-git-finish-push.git
mkdir -p ~/.codex/skills
cp -R auto-git-finish-push/auto-git-finish-push ~/.codex/skills/auto-git-finish-push
```

Then restart Codex.

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