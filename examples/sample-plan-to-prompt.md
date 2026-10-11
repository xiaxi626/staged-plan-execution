# 示例：plan 输入 → 生成的 prompt

本示例展示给定一段简短的两步 plan，生成的 prompt 长什么样。仅为形态示意，不展示完整交接骨架。

## 输入（用户提供的 plan）

```markdown
## 修订方案

### 步骤 1：foo 加锁

**文件**：`src/foo.ts`

**改动**：导入 `withManifestLock`，包裹函数体

\`\`\`ts
import { withManifestLock } from './locks.js'

async function applyFoo(name: string): Promise<void> {
  await withManifestLock(async () => {
    // ... 原函数体
  })
}
\`\`\`

**要点**：无嵌套风险。

### 步骤 2：bar 错误脱敏

**文件**：`src/bar.ts`

**改动**：判断哨兵名

\`\`\`ts
const label = name === '__manifest__' ? 'manifest' : `name "${name}"`
\`\`\`

### 修复顺序

| 步骤 | 内容 | 涉及文件 |
|------|------|----------|
| 1 | foo 加锁 | `src/foo.ts` |
| 2 | bar 错误脱敏 | `src/bar.ts` |

每步完成后：`npm run typecheck && npm run lint && npm test`
```

## 输出（生成的 prompt 片段，仅展示结构与开头）

```text
你需要为当前项目执行以下修订方案，按给定顺序逐一完成各步骤。每完成一个步骤，运行质量门禁确认通过后再进入下一个。

## 项目风格要求（所有步骤通用）
（← 逐字复制 references/project-style-guide.md，占位符已填充为 npm run typecheck / npm run lint / npm test）

## 原子提交规则（严格遵守）
（← 逐字复制 references/atomic-commit-rules.md）

## CHANGELOG 规则（严格遵守）
（← 逐字复制 references/changelog-rules.md）

## README 更新规则
（← 逐字复制 references/readme-rules.md；本例未新增 CLI flag → 不更新 README）

## 上下文管理与阶段交接
（← 逐字复制 references/handoff-format.md，含 7 项硬性格式与推荐骨架）

---

## 修订方案

### 步骤 1：foo 加锁

**文件**：`src/foo.ts`

**改动**：导入 withManifestLock，包裹函数体

\`\`\`ts
import { withManifestLock } from './locks.js'

async function applyFoo(name: string): Promise<void> {
  await withManifestLock(async () => {
    // ... 原函数体
  })
}
\`\`\`

**要点**：无嵌套风险。

### 步骤 2：bar 错误脱敏
（... 逐字保留 ...）

### 修复顺序
（... 逐字保留 ...）

每步完成后：npm run typecheck && npm run lint && npm test

---

## 全部完成后
（← 逐字复制 references/final-verification.md，占位符已填充）
```

## 关键点

- plan 正文（步骤小节、代码块、要点、修复顺序表）**逐字保留**
- 执行框架（项目风格、原子提交、CHANGELOG、README、交接）**逐字来自 references/**
- 项目名替换为"当前项目"，但代码块内标识符（如 `withManifestLock`、`__manifest__`）保持原样
- 占位符按 plan 中出现的命令填充
