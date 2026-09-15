# PR

PR（Pull Request，拉取请求）是代码审查与合并的载体。GitHub CLI（`gh`）提供了完整的 PR 操作命令。

## 创建

`gh pr create` 创建 PR，成功时打印 URL。

- `-t/--title`、`-b/--body`（或 `-F/--body-file`，`-` 表示读 stdin）指定标题与正文；`-f/--fill` 用 commit 信息自动填充（`--fill-first`、`--fill-verbose`），与显式 title/body 同时给出时以显式值为准。
- `-B/--base` 目标分支（默认仓库默认分支，可用 `git config branch.<当前分支>.gh-merge-base` 指定）；`-H/--head` 源分支（默认当前分支，支持 `<user>:<branch>` 用于 fork 场景）。
- `-d/--draft` 创建草稿；`-w/--web` 浏览器创建；`--dry-run` 只打印不创建。
- `-r/--reviewer`、`-a/--assignee`、`-l/--label`、`-m/--milestone`、`-p/--project` 添加审查人、指派、标签、里程碑等。
- 正文中写 `Fixes #123`/`Closes #123`，合并时会自动关闭对应 issue。
- 分支未推送时会提示推送或 fork，`--head` 可跳过此行为。别名 `gh pr new`。

## 查看

- `gh pr view [<number>|<url>|<branch>]`：查看标题、正文、状态等，缺省为当前分支的 PR；`-c/--comments` 附带评论，`-w/--web` 浏览器打开，`--json`+`-q` 输出机器可读数据。
- `gh pr list`：列出 PR，默认仅 open；`-s/--state {open|closed|merged|all}`、`-A/--author`、`-a/--assignee`、`-l/--label`、`-B/--base`、`-H/--head`、`-d/--draft`、`-S/--search`（高级搜索语法）、`-L/--limit`（默认 30）。
- `gh pr status`：汇总与当前用户相关的 PR（当前分支、我创建的、请求我审查的）；`-c/--conflict-status` 显示冲突状态。
- `gh pr diff [<number>]`：查看 PR 的 diff；`--name-only`、`--patch`、`--color`、`-e/--exclude`、`-w/--web`。
- `gh pr checks [<number>]`：CI 检查状态；`--watch` 持续观察、`--fail-fast` 首次失败即退出、`--required` 仅看必需检查；退出码 8 表示仍有检查 pending。
- `gh pr checkout [<number>]`：检出 PR 到本地分支便于调试；`-b` 指定本地分支名、`--detach`、`-f` 重置到最新、`--recurse-submodules`。

## 审查

- `gh pr review [<number>]`：三选一 `-a/--approve`（批准）、`-r/--request-changes`（要求修改）、`-c/--comment`（仅评论）；`-b/--body` 或 `-F/--body-file` 写意见。
- `gh pr comment [<number>]`：发表普通评论，不影响审查结论；`--edit-last` 编辑自己最后一条评论，`--delete-last` 删除，`--create-if-none` 配合 `--edit-last`，`-w` 网页写。
- 行级评论与讨论串回复需在网页端或通过 `gh api` 完成。

## 合并

`gh pr merge [<number>]` 合并 PR，缺省为当前分支。三种策略互斥：

- `-m/--merge` 生成 merge commit；`-s/--squash` 压缩为单个 commit；`-r/--rebase` 变基到目标分支、不生成 merge commit。
- `-d/--delete-branch` 合并后删除本地与远程分支。
- `--auto` 满足条件后自动合并，`--disable-auto` 关闭；`--admin` 绕过分支保护强制合并。
- `-t/--subject`、`-b/--body` 自定义合并提交信息；`--match-head-commit <SHA>` 校验 head commit 未被改动。
- 目标分支启用 merge queue 时可不指定策略：检查未过自动开启 auto-merge，已过则进入队列。

## 关闭

`gh pr close {<number>|<url>|<branch>}` 不合并直接关闭（状态为 closed，可重开）；`-c/--comment` 关闭时留评论；`-d/--delete-branch` 同时删除本地与远程分支。

## 重开

`gh pr reopen {<number>|<url>|<branch>}` 重开已关闭的 PR，已合并的不能重开；`-c/--comment` 重开时留评论。

## 就绪

`gh pr ready [<number>]` 将草稿 PR 标记为可审查，缺省当前分支；`--undo` 转回草稿。

不使用 `gh` 时，可用原生 Git 操作 PR：

- 拉取 PR 到本地：`git fetch origin pull/123/head:pr-123`
- 更新 PR：向远端对应分支 push 即可

## PR Conflict

PR冲突通常源于目标分支（如 main）与源分支修改了同一文件的同一区域，合并时 Git 无法自动决定取舍。

### merge 目标分支

`git fetch origin && git merge origin/main`：把目标分支的最新提交合并进当前分支，产生一个合并提交（Merge Commit），保留分叉历史；只改动自己的分支，main 不受影响。

1. `git status` 查看冲突文件（both modified），编辑并处理冲突标记 `<<<<<<<`（本分支内容）、`=======`（分隔线）、`>>>>>>> origin/main`（目标分支内容），保留正确结果并删除标记。
2. `git add <file>` 标记已解决，全部解决后 `git commit` 完成合并。
3. `git push` 更新 PR。

特点：不改写已有提交，分支被多人共享时更安全；缺点是历史有分叉、多出合并提交。

### rebase 到目标分支

`git fetch origin && git rebase origin/main`：把当前分支独有的提交逐个重放到目标分支最新提交之上，生成全新 commit、历史呈直线，不产生合并提交（区别于 merge 保留分叉）。

1. 冲突时 rebase 暂停：编辑文件后 `git add <file>`，用 `git rebase --continue` 继续重放下一个提交；`git rebase --skip` 跳过当前提交，`git rebase --abort` 放弃整个变基、恢复原状。
2. 推送需 `git push --force-with-lease`：哈希改变后与远端历史分叉，普通 push 会被拒；该命令先校验远端分支仍指向预期提交才强推，比 `--force` 安全。

特点：历史整洁线性、无合并提交噪声；但改写历史，分支被多人共享时慎用，且每个提交都可能解一次冲突。适合堆叠 PR 中同步下层改动。


## 堆叠 PR

把一个大改动拆成多个相互依赖的小 PR，分支层层叠加，每个 PR 的目标分支指向前一层分支：

```
main ← A ← B ← C
PR1: A→main   PR2: B→A   PR3: C→B
```

- 创建：`gh pr create --base <下层分支>`。
- 合并顺序自底向上：底层 PR 合并后，需把上层 PR 的 base 改指 main，并将分支 rebase 到最新 main，避免 diff 混入已合并的提交。
- 下层分支有更新时，上层分支需级联同步（rebase 或 merge）。
- 优点：每个 PR 小而聚焦、可独立审查，审查与开发并行；代价是管理成本高，常用 Graphite、ghstack、git-branchless 等工具辅助。
