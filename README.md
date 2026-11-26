# Git Smart Commit ZQH

由你选择 Git 提交信息、整批分支和最终操作。技能名：`git-smart-commit-zqh`。

## 使用

```text
$git-smart-commit-zqh 帮我审查当前改动并准备提交。
```

1. 审查改动、运行适用检查，按独立目的拆分 commits；单一改动不硬拆，实现和对应测试放在一起。
2. 每个 commit 展示两份完整的 Conventional Commit 候选，由你二选一，也可输入修改；选好后不再逐字段追问。
3. 整批只选一次分支，提供“保留原分支：实际名称”和“创建新分支：生成名称”，允许输入自定义新名称。
4. 展示所有提交、范围、检查结果和推送目标后，由你选择“提交本地”“提交本地并 push 远程”或“不提交”。选择后直接执行，不重复确认。
5. 验证并报告 commit hashes、分支、推送结果和剩余改动。失败时停止并保留已完成的提交。
6. 每次完成提交后，汇总本次实际创建的 commits（短 hash、原始标题、改动摘要），并给出可复制的 PR 标题和正文建议。PR 标题必须遵循 Conventional Commits 格式 `<type>[optional scope][!]: <description>`，例如 `feat(skill): 增加提交总结与 PR 建议`。PR 正文只用 `1. 2. 3.` 有序列表，每点一句话，简要描述该分支相对已确认 PR 目标分支完成的事情，合并同类改动，不分章节、不重复罗列 commits；验证结果和注意事项在正文之外单独报告。仅提交本地也会生成，不自动创建 PR。若部分提交或推送失败，仅总结已完成的提交并说明未完成项；未创建任何 commit 时不生成 PR 草稿。完整 PR 范围无法确认时，明确提示草稿仅覆盖本次已验证的提交。

每个 commit 消息、整批分支和最终操作都必须由你明确选择，不自动代选。有弹窗时优先使用，否则在聊天中列出相同的编号选项并等待回复；两种方式都允许自定义输入。沉默、超时或预选默认项均不视为选择。最终选择前不切分支、不暂存、不提交。“不提交”保留现状。

## 规范与边界

提交严格遵循 [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)：`<type>[optional scope][!]: <description>`，正文和 footer 可选；破坏性变更必须标注。默认小写类型/模块、中文描述，自定义内容也需符合规范。

新分支固定为 `<type>/<short-description>`，例如 `feat/login-session`。类型按整批主要目的选择；描述使用小写英文、数字和单连字符。只含一个 `/`，不添加 `codex`、用户名等前缀；创建前通过 `git check-ref-format --branch` 和重名检查。原分支可明确选择保留，不受新命名规则限制。

只提交确认的文件或代码块，保留无关改动和已有暂存状态。不强推、不改写历史、不自动修复代码或创建 PR。推送前展示远程、目标分支及此前未推送的 commits。

## 安装

```bash
git clone https://github.com/zhang-qihao/Git.SKILL.git \
  "$HOME/.agents/skills/git-smart-commit-zqh"
```

更新已有安装副本即可，避免重复安装同名技能。完整执行规则见 [SKILL.md](SKILL.md)；`agents/openai.yaml` 提供界面信息和调用提示。
