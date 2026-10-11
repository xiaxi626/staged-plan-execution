# 上下文管理与阶段交接

> 此段内容在生成的 prompt 中作为"## 上下文管理与阶段交接"小节输出。**注意**：交接 prompt 的"逐字复制段落"只包含前四段（项目风格要求、原子提交规则、CHANGELOG 规则、README 更新规则），不递归复制本节自身。

每完成一个步骤并 git 提交后，无论上下文是否充裕，都必须输出一份交接 prompt，然后立即停止本回合。不要开始下一个步骤，不要做任何"顺手"的改动。交接 prompt 直接在会话中输出，用户会复制到新会话中使用；不要写入文件。

## 交接 prompt 的硬性格式

交接 prompt 必须整体用**四反引号**围栏包裹，内部原有三反引号代码块保持原样。末尾 `HANDOFF AFTER STEP <n>` 或 `HANDOFF AFTER ALL STEPS` 必须位于四反引号代码块内。不要在四反引号外输出交接正文；可在外层只说一句"以下为交接 prompt"。

交接 prompt 必须包含以下内容，不得摘要、不得省略、不得自行发挥：

### 1. 新会话启动确认

- 要求新会话先执行并报告：
  - `git status --short --branch`
  - `git rev-parse HEAD`
  - `git remote -v`
  - `git log -1 --oneline`
- 明确当前分支、当前 HEAD、工作区是否干净、是否与远程同步
- 明确本步骤应基于哪个 commit 继续

### 2. 已完成步骤

- 已完成步骤编号
- 每一步的提交 hash；若同一步有多个提交，全部列出

### 3. 质量门禁状态

- 最后一次质量门禁命令的通过/失败状态
- 若已到最终步骤，还必须列出全部编译和构建相关命令的结果
- 若有失败，必须原样附上失败输出摘要；不得只写"失败"

### 4. 下一步完整要求

- 逐字复制本方案中该步骤对应的全部小节正文，含代码块与"要点"，不得只复制修复顺序表里的那一行
- 若一个步骤对应多个小节，全部复制
- 若某步骤没有独立小节正文，必须展开为可执行清单，至少包含：涉及文件、改动内容、测试更新、需要运行的命令、提交范围
- 必须注明该步骤完成后要跑的质量门禁命令

### 5. 剩余未做步骤清单

- 按原修复顺序表列出剩余步骤编号和内容
- 不要省略任何剩余步骤

### 6. 逐字复制以下段落

> 这四段内容来自本技能的 `references/` 文件，agent 在生成交接 prompt 时应从已加载的参考文件中逐字复制：

- `## 项目风格要求（所有步骤通用）` ← references/project-style-guide.md
- `## 原子提交规则（严格遵守）` ← references/atomic-commit-rules.md（含编译产物适用范围补充说明）
- `## CHANGELOG 规则（严格遵守）` ← references/changelog-rules.md
- `## README 更新规则` ← references/readme-rules.md

必须逐字复制，不得摘要、不得改写、不得只复制标题。

不要把本节"上下文管理与阶段交接"自身递归复制进交接 prompt；只复制上述四段和本节的格式要求。

逐字复制上述段落之后，追加"原子提交规则"文件中的"编译产物适用范围补充说明"（不改变原文）。

### 7. 停止指令

- 交接 prompt 末尾必须写一行：`HANDOFF AFTER STEP <n>`；最后一步完成后写 `HANDOFF AFTER ALL STEPS`
- 输出交接 prompt 后立即停止本回合

## 交接 prompt 推荐骨架

交接时按以下骨架生成，外层用四反引号包裹：

````markdown
请在新会话中继续执行当前项目修订方案。

## 新会话启动确认
请先执行并报告：
- `git status --short --branch`
- `git rev-parse HEAD`
- `git remote -v`
- `git log -1 --oneline`

## 当前状态
- 当前分支：
- 当前 HEAD：
- 工作区状态：
- 远程同步状态：
- 已完成步骤：
- 提交 hash：
- 质量门禁结果：

## 下一步完整要求
（逐字复制本方案中该步骤对应的全部小节正文，含代码块与"要点"）

## 剩余未做步骤
（按修复顺序表列出）

## 项目风格要求（逐字复制）
（逐字复制 references/project-style-guide.md 整段）

## 原子提交规则（逐字复制）
（逐字复制 references/atomic-commit-rules.md 整段，含编译产物适用范围补充说明）

## CHANGELOG 规则（逐字复制）
（逐字复制 references/changelog-rules.md 整段）

## README 更新规则（逐字复制）
（逐字复制 references/readme-rules.md 整段）

HANDOFF AFTER STEP <n>
````

如果这是最后一个步骤，交接 prompt 只需包含完成状态汇总：步骤编号、提交 hash、门禁结果，并写 `HANDOFF AFTER ALL STEPS`。无需后续步骤要求。
