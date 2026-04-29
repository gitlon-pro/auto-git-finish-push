# Auto Git Finish Push

[中文说明](README.zh-CN.md)

Codex skill for safely finishing a task with Git: inspect changes, stage only task-owned files, generate a commit message from the staged diff, commit, and push.

The skill is intentionally conservative. It stops instead of committing when branch safety, change ownership, secrets, conflict state, or remote state is unclear.

## Install

Install from GitHub with the Codex skill installer:

```sh
install-skill-from-github.py --repo <owner>/auto-git-finish-push --path auto-git-finish-push
```

Replace `<owner>` with the GitHub account or organization that hosts this repository.

Restart Codex after installation so the skill can be discovered.

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

## License

Add your preferred license before publishing if you want others to reuse or modify the skill clearly.
