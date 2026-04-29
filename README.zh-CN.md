# Auto Git Finish Push

一个用于 Codex 的 Git 收尾 skill。

它适合在一次代码或文档任务完成后，自动检查当前 Git 状态、暂存本次任务产生的变更、根据 staged diff 生成提交信息、创建 commit，并执行普通 push。

这个 skill 的核心原则是：不确定就停止。只要分支、文件、变更归属、冲突状态或远端状态不安全，它就不会提交，也不会推送。

## 安装

通过 [skills.sh](https://skills.sh/) 安装：

```sh
npx skills add https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push -a codex -g -y
```

通过 Codex skill installer 从 GitHub 安装：

```sh
install-skill-from-github.py --repo gitlon-pro/auto-git-finish-push --path auto-git-finish-push
```

也可以直接使用 GitHub tree URL：

```sh
install-skill-from-github.py --url https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push
```

如果你的 Codex 环境支持 `skill-installer`，也可以直接让 Codex 安装：

```text
Use $skill-installer to install https://github.com/gitlon-pro/auto-git-finish-push/tree/main/auto-git-finish-push
```

安装后重启 Codex，让新 skill 生效。

## 使用方式

一次性显式使用：

```text
Use $auto-git-finish-push to commit and push this task.
```

如果希望某个项目每次任务结束后都自动触发，可以把下面内容加入项目的 `AGENTS.md`：

```text
At the end of every coding or documentation task, before the final response, use $auto-git-finish-push to safely stage, commit, and push task-owned changes.
```

执行时它会先检查 Git 状态。只要发现保护分支、任务外脏文件、敏感文件、冲突状态、远端不可安全推送等问题，就会停止并报告原因，不会强行提交。

## 手动安装

如果不使用 installer，也可以手动复制 skill 目录：

```sh
git clone https://github.com/gitlon-pro/auto-git-finish-push.git
mkdir -p ~/.codex/skills
cp -R auto-git-finish-push/auto-git-finish-push ~/.codex/skills/auto-git-finish-push
```

复制完成后重启 Codex。

## 目录结构

可安装的 skill 位于：

```text
auto-git-finish-push/
├── SKILL.md
└── agents/
    └── openai.yaml
```

仓库根目录的 `README.md` 和 `README.zh-CN.md` 只给 GitHub 用户阅读，不属于 skill 运行时必须内容。

## 它会做什么

- 在任务开始前记录 Git baseline。
- 检查当前目录是否在 Git 仓库内。
- 检查是否处于 merge、rebase、cherry-pick、bisect 等中间状态。
- 拒绝在 `main`、`master`、`production`、`release/*`、`hotfix/*` 等保护分支上直接提交。
- 拒绝提交 `.env`、私钥、证书、`.DS_Store` 等敏感或本地文件。
- 如果任务开始前工作区已经有脏文件，只暂存本次任务明确拥有的文件。
- 根据 `git diff --cached` 生成 commit message。
- 只执行普通 `git push`。
- 永远不执行 `git push --force` 或 `git push --force-with-lease`。

## 它什么时候会停止

遇到下面情况，skill 会停止，并告诉你原因：

- 当前目录不是 Git 仓库；
- 当前仓库正在 merge、rebase、cherry-pick 或 bisect；
- 当前分支属于保护分支；
- 变更文件疑似包含敏感文件或本地临时文件；
- 无法确认哪些文件属于本次任务；
- 没有 upstream，且项目规则没有允许自动执行 `git push -u origin HEAD`；
- 推送需要 force。

## 提交信息格式

skill 会根据 staged diff 生成类似下面的 commit message：

```text
type(scope): short imperative summary

- concrete change
- concrete change

Verification:
- command run, or "not run"
```

常用 `type`：

- `feat`：用户可感知的新功能；
- `fix`：问题修复；
- `docs`：文档；
- `refactor`：不改变行为的重构；
- `test`：测试；
- `chore`：工具、配置、维护类变更。

## 适合谁

适合希望 Codex 在完成任务后自动做 Git 收尾的人：

- 不想每次手动整理 `git add` / `commit` / `push`；
- 但又不希望 AI 把不属于本次任务的脏文件一起提交；
- 希望遇到不安全状态时默认停止，而不是冒险提交。

## 设计取舍

这个 skill 偏保守。

它不会追求“任何情况下都自动推上去”，而是优先保护仓库状态。对于多人协作、长期脏工作区、敏感配置较多的项目，这种策略更安全。

## 发布前建议

如果你要 fork 或重新发布这个仓库，建议补充：

- `LICENSE`：推荐 MIT，方便别人明确复用；
- GitHub 仓库描述；
- 示例截图或一段真实使用示例；
- 如果 fork 到其他账号，把安装命令里的 `gitlon-pro` 改成你的 GitHub 用户名或组织名。