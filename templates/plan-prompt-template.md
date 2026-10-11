# 生成的 Prompt 整体骨架

> agent 按此顺序组装生成的 prompt。`{{...}}` 为占位符，根据用户计划填充。

生成 prompt 的整体结构如下（按顺序）：

---

## 1. 开头执行指令（一句话）

```text
你需要为当前项目执行以下修订方案，按给定顺序逐一完成各步骤。每完成一个步骤，运行质量门禁确认通过后再进入下一个。
```

## 2. 执行框架段（来自 references/）

按以下顺序逐字输出各参考文件的内容：

1. `## 项目风格要求（所有步骤通用）` ← references/project-style-guide.md
2. `## 原子提交规则（严格遵守）` ← references/atomic-commit-rules.md
3. `## CHANGELOG 规则（严格遵守）` ← references/changelog-rules.md
4. `## README 更新规则` ← references/readme-rules.md
5. `## 上下文管理与阶段交接` ← references/handoff-format.md

## 3. 用户提供的具体修复方案（逐字插入）

```text
---

## {{用户提供的方案标题}}

{{此处逐字插入用户提供的计划正文，包括：}}
- 所有分步骤小节（含文件、改动内容、代码块、"要点"说明）
- 修复顺序表
- 每步完成后的质量门禁要求
- 方案中指定的专项验证测试（如有）

注意：计划正文中所有对具体项目名的引用替换为"当前项目"，但代码块内的标识符保持原样。
```

## 4. 全部完成后的验证

```text
---

## 全部完成后

← references/final-verification.md 内容，占位符已填充
```

## 占位符填充规则

| 占位符 | 填充规则 |
|--------|---------|
| `{{typecheck_cmd}}` | 优先用计划中出现的命令（如 `npm run typecheck`）；未出现用 `typecheck` |
| `{{lint_cmd}}` | 优先用计划中出现的命令（如 `npm run lint`）；未出现用 `lint` |
| `{{test_cmd}}` | 优先用计划中出现的命令（如 `npm test`）；未出现用 `test` |
| `{{build_cmd}}` | 优先用计划中出现的命令（如 `npm run build`）；未出现用 `build` |

## README 更新规则的调整

若用户计划新增了对用户可见的 CLI flag 或新命令，则在 README 更新规则中说明需要更新 `README.md`；否则保持"不更新 README"。
