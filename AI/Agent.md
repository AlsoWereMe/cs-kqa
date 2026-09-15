# Agent

智能体（Agent）可加载技能（Skill）来扩展自身能力。

## 技能

技能是智能体的可复用能力单元：以目录组织，入口为固定命名的 Markdown 文件（`SKILL.md`），正文写指令、示例与参考资料；目录名即技能名。加载器扫描到该文件后，根据其 YAML frontmatter（元数据区）决定是否向模型暴露该技能。该约定已被多款智能体工具采用。

### frontmatter 字段

| 字段 | 必要性 | 作用 |
| --- | --- | --- |
| `name` | 必要 | 技能标识符；通常小写、连字符分隔，与所在目录同名，有长度上限（如 64 字符） |
| `description` | 必要 | 模型据此判断何时加载此技能；缺失的技能会被过滤、不会呈现给模型。应写清"做什么"与"何时用"，并前置用户可能说出的触发关键词或文件名 |
| `license` | 可选 | 许可证声明（如 MIT） |
| `compatibility` | 可选 | 兼容性说明，标注适用的运行环境 |
| `metadata` | 可选 | 自定义字符串键值对（如 audience、workflow），仅作附加元信息，不参与加载与触发逻辑 |

要点：`name` 与 `description` 是仅有的必要字段；触发完全依赖 `description` 的措辞质量。

### 示例

```markdown
---
name: conventional-commit
description: 按约定式提交（Conventional Commits）规范撰写 commit message；当用户要求提交代码或编写提交信息时使用。
metadata:
  workflow: github
---

# Conventional Commit

格式：`<type>(<scope>): <subject>`
- type 取值：feat、fix、docs、refactor、test、chore
- subject 用祈使句，不超过 50 字符

示例：`feat(auth): add login rate limiting`
```

