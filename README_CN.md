# staged-plan-execution

一个 Trae SKILL：将任意多步骤修复/改进计划包裹成可直接交给 agent 执行的完整 prompt——内置质量门禁、原子提交、CHANGELOG 规则与多会话交接。

[English](README.md)

## 它做什么

给它一段多步骤计划，比如：

```markdown
## 步骤 1：foo 加锁
**文件**：`src/foo.ts`
**改动**：导入 withManifestLock，包裹函数体
...
## 步骤 2：bar 错误脱敏
...
| 步骤 | 内容 | 涉及文件 |
| 1 | foo 加锁 | src/foo.ts |
| 2 | bar 错误脱敏 | src/bar.ts |
每步完成后：npm run typecheck && npm run lint && npm test
```

它会生成一个完整 prompt，长这样：

```text
你需要为当前项目执行以下修订方案，按给定顺序逐一完成各步骤...
每完成一个步骤，运行质量门禁确认通过后再进入下一个。

## 项目风格要求（所有步骤通用）
（点名 plan 涉及的源文件 + 同名测试文件作为风格范例）

## 原子提交规则（严格遵守）
（fix + docs 拆分规则）

## CHANGELOG 规则（严格遵守）
（位置、修订、日期规则）

## README 更新规则
（条件判断）

## 上下文管理与阶段交接
（四反引号交接 prompt 7 项硬性格式 + 推荐骨架）

---

## <plan 标题>
（plan 正文逐字保留，项目名替换为"当前项目"，代码块内标识符不动）

---

## 全部完成后
（最终验证清单）
```

生成的 prompt 面向**多会话接力执行**：每完成一个步骤，执行 agent 输出一份交接 prompt（用四反引号包裹），用户复制到新会话继续。

## 为什么需要

跨多个 agent 会话执行多步骤计划时，三个问题反复出现：

1. **上下文漂移**——后一个会话忘了前一个会话建立的约定
2. **交接格式不一致**——每个会话各自发明"我做到哪了"的格式
3. **风格退化**——上下文紧张时，agent 跳过质量门禁，或把代码和文档混在一个提交里

本技能把约定固化进模板驱动的生成器，让每次交接都把完整执行框架逐字带下去。

## 结构

```
.trae/skills/staged-plan-execution/
├── SKILL.md                       # 入口：用途、适用域、生成规则、参考资源表
├── references/                    # 硬性约束，逐字加载进生成的 prompt
│   ├── project-style-guide.md     # 6 条风格规则 + 生态识别 + 文件点名
│   ├── atomic-commit-rules.md     # fix + docs 提交拆分 + 编译产物适用范围
│   ├── changelog-rules.md         # Keep a Changelog 约定
│   ├── readme-rules.md            # 条件性 README 更新逻辑
│   ├── handoff-format.md          # 交接 prompt 7 项必含内容 + 推荐骨架
│   └── final-verification.md      # 完成后的验证清单
├── templates/
│   └── plan-prompt-template.md    # 生成 prompt 的整体骨架 + 占位符规则
└── examples/
    └── sample-plan-to-prompt.md   # 完整示例：plan 输入 → 生成的 prompt
```

这样拆分是有意的：

- **SKILL.md 精炼**——加载快，让 agent 快速判断是否调用本技能
- **references/ 承载重约束**——按需加载，然后逐字复制进生成的 prompt。这保证可复现性：同一个 plan 每次生成的 prompt 都一样，不依赖 agent 记忆
- **templates/ 定义组装顺序**——agent 不即兴发挥结构
- **examples/ 锚定理解**——具体展示输入→输出形态

## 适用域

默认假设以下技术栈与约定：

- **语言**：TypeScript / npm（`npm run typecheck` / `lint` / `test` / `build`）
- **测试**：`node:test` + `tsx`，测试文件命名 `xxx.test.ts`，临时目录隔离
- **CHANGELOG**：Keep a Changelog 风格（`## [Unreleased]`）
- **执行模型**：多会话接力——每步结束输出四反引号交接 prompt

本技能针对 **DeepSeek harness 插件生态**调优（中英双语 CONTRIBUTING、`src/` + `test/` + `lib/` 三段结构、`dsh-` CLI 前缀）。其他 TS/npm 项目也能用；非 TS 项目需要适配 `references/`。

## 关键设计选择

| 选择 | 理由 |
|------|------|
| 计划正文逐字保留 | 计划作者的意图不能被改写或摘要 |
| 项目名 → "当前项目" | 解耦技能与具体仓库 |
| 代码块内标识符不动 | 它们是代码本身，不是散文 |
| agent 生成时必须 Read `references/` | 防止记忆漂移；同一个 plan → 同一个 prompt |
| agent 扫描 plan 点名具体文件 | 恢复 plan↔prompt 的紧密联系——风格要求段点名 plan 涉及的具体文件和同名测试文件 |
| 四反引号交接包裹 | 让内部三反引号代码块在用户复制交接 prompt 时保持完整 |

## 如何使用

### 作为 Trae SKILL

1. 把 `staged-plan-execution/` 目录放到工作区的 `.trae/skills/` 下
2. 向 agent 提供一段多步骤计划
3. agent 识别到本技能，Read `references/` 和 `templates/`，然后在会话中输出完整 prompt
4. 把生成的 prompt 复制到新的 agent 会话开始执行

### 作为其他工具的参考

`references/` 和 `templates/` 都是纯 Markdown。可以适配到其他 agent 框架、CI 流水线或文档生成器中——这些约定本身与工具无关。

## 局限

- **以 TS/npm 为中心**：`references/` 使用 JS 专属术语（JSDoc、`after()` 钩子、`lib/` 编译产物）。Python/Go/Rust 需要适配风格指南
- **单次调用单计划**：不能把多个计划组合成一个 prompt
- **不做计划校验**：不检查计划本身是否合理——用户给什么就包裹什么

## 许可

MIT
