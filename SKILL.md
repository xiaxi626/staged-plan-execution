---
name: "staged-plan-execution"
description: "将多步骤修复/改进计划包裹成可直接发给执行 agent 的完整 prompt，自动补充项目风格、原子提交、CHANGELOG、阶段交接等执行框架。Invoke when user provides a multi-step fix/improvement plan and asks to generate an executable agent prompt."
---

# Staged Plan Execution

## 用途

把用户提供的多步骤修复/改进计划，包裹成可直接发给执行 agent 的完整 prompt。

生成的 prompt = **执行框架**（来自 `references/`）+ **计划正文**（来自用户，逐字保留）。

## 适用域

默认假设以下技术栈与约定，非同类项目需在调用时声明：

- TS/npm 项目，`npm run typecheck` / `npm run lint` / `npm test` / `npm run build`
- 测试使用 `node:test` + `tsx`（或当前项目实际测试栈）
- CHANGELOG 采用 Keep a Changelog 风格（`## [Unreleased]`）
- 多会话接力执行：每步完成后输出四反引号交接 prompt，新会话复制继续

## 核心生成规则

1. **计划逐字保留**：用户提供的计划正文（含代码块、"要点"、修复顺序表）必须逐字复制到生成 prompt 中，不得摘要、改写、省略或自行发挥
2. **项目名替换**：所有对具体项目名的引用替换为"当前项目"。代码块内标识符（常量名、函数名、文件路径中的项目相关名称）保持原样
3. **占位符填充**：`{{typecheck_cmd}}` 等占位符优先用计划中已出现的命令填充；未指定时用通用表述"当前项目的质量门禁命令"
4. **不写入文件**：生成的 prompt 直接在会话中输出，供用户复制使用

## 参考资源（按需 Read）

生成 prompt 前，**必须 Read 以下参考文件**并按其要求生成对应段落。这些文件包含完整的硬性约束与模板，不得凭记忆复述：

| 文件 | 用途 |
|------|------|
| [references/project-style-guide.md](file:///workspace/.trae/skills/staged-plan-execution/references/project-style-guide.md) | 项目风格要求 6 条（注释、测试、门禁、编译产物、core 层、argv） |
| [references/atomic-commit-rules.md](file:///workspace/.trae/skills/staged-plan-execution/references/atomic-commit-rules.md) | 原子提交规则（fix + docs 拆分） |
| [references/changelog-rules.md](file:///workspace/.trae/skills/staged-plan-execution/references/changelog-rules.md) | CHANGELOG 规则（位置、修订、日期） |
| [references/readme-rules.md](file:///workspace/.trae/skills/staged-plan-execution/references/readme-rules.md) | README 更新规则（条件判断） |
| [references/handoff-format.md](file:///workspace/.trae/skills/staged-plan-execution/references/handoff-format.md) | 交接 prompt 硬性格式 7 项 + 推荐骨架 |
| [references/final-verification.md](file:///workspace/.trae/skills/staged-plan-execution/references/final-verification.md) | 全部完成后的验证清单 |
| [templates/plan-prompt-template.md](file:///workspace/.trae/skills/staged-plan-execution/templates/plan-prompt-template.md) | 生成的 prompt 整体骨架（按此顺序组装） |

首次使用前建议先 Read [examples/sample-plan-to-prompt.md](file:///workspace/.trae/skills/staged-plan-execution/examples/sample-plan-to-prompt.md) 理解输入→输出形态。

## 生成流程

1. **读取计划**：理解步骤结构、涉及文件、代码改动、测试更新
2. **识别项目名**：扫描计划正文，找出所有对具体项目名的引用
3. **扫描涉及文件**：扫描计划中提到的所有源文件路径（如 `src/foo.ts`、`src/bar.ts`），为每个涉及源文件点名对应的测试文件（同名 `.test.ts`）和相邻源文件作为风格范例——这些具体文件名将在第 4 步填入"项目风格要求"段的第二条和第三条
4. **按需加载参考**：Read 上表所有 `references/` 与 `templates/` 文件
5. **填充模板**：将计划正文逐字插入模板的"具体修复方案"部分，项目名替换为"当前项目"，填充占位符；将第 3 步点名的具体文件名填入"项目风格要求"段的"先阅读以下文件"部分，替换泛化表述
6. **组装输出**：按 `templates/plan-prompt-template.md` 的顺序组装，直接在会话中输出
7. **不写入文件**

## 触发条件

- 用户提供多步骤修复/改进计划，要求生成可执行 prompt
- 用户要求将计划包裹成带质量门禁、原子提交、阶段交接的执行框架
- 用户要求生成"交接 prompt"格式以管理多会话接力执行
