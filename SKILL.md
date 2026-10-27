---
name: homesafe-project-maintain
description: 为项目建立并持续维护 AI_proj_note 认知笔记体系，通过项目字典、固定锚点、分层读取和幂等写入，使换人、换 Agent、换对话时项目认知不丢。触发场景为用户要求初始化或维护项目笔记、首次接触新项目或新接手已有项目需要召回项目认知时。
---

# Skill：AI 项目笔记管理

> 目标：让模型依靠可验证的地址、状态和局部操作维护项目认知，而不是依靠阅读全文、全局计数或长期记忆。
> 核心结构：SKILL 定义协议，`index.md` 定义项目路由，笔记文件保存事实与锚点，`rule.md` 只保存当前有效约束。

## 1. 不可违反的约定

1. **稳定与动态分离**：`rule.md` 不写 Git 历史、任务流水账或已失效规范；动态内容进入 `progress.md` 或 `GitHistory.md`。
2. **索引不是知识库**：`index.md` 只保存 Section ID、文件地址、锚点、触发事件、写入方式和唯一键，不复制项目事实。
3. **按需阅读**：默认只读 L0；不得为了维护一条记录而通读全部笔记或整个项目。
4. **先查再写**：任何写入都先定位 Section、搜索唯一键，再执行局部 Upsert/Replace/Insert；禁止凭记忆直接追加。
5. **日志顺序唯一**：所有日志均为新到旧，只能插入对应 `APN:INSERT:*` 锚点后；不重排旧条目，不使用含义模糊的“追加”。
6. **不按条数归档**：不数“最近几条”，不让模型搬运历史。旧历史保留在冷文件中，需要时局部读取。
7. **事实与推断分离**：代码只能证明当前实现，不能证明用户曾否决什么。用户意图必须来自对话或已有明确记录；推断必须标注“推断、待确认”。
8. **最小 diff**：无实质变化不重写文件；禁止为了更新一个条目而重写整份笔记。

## 2. 启动与分层读取

### 2.1 启动路由

- 项目根目录不存在 `AI_proj_note/rule.md`：先与用户确认是否启用；确认后阅读 [project-setup.md](references/project-setup.md) 和 [templates.md](references/templates.md)，执行初始化。
- 已存在且 `index.md`、L0 锚点齐全：进入 L0，不重新初始化，不做全项目考古。
- 已有旧版 `rule.md`，但缺少 `index.md` 或锚点：进入结构升级模式，阅读 [project-setup.md](references/project-setup.md) 的旧版升级节和 [templates.md](references/templates.md)；保留并迁移原内容，不重新初始化。
- 已声明的文件存在，但字典目标路径或锚点局部缺失：按模板局部修复后操作；不得自行猜测写入位置。

确认启用后，立即向项目级记忆写入：

> 本项目已启用 AI 项目笔记管理。任何开发动作前先阅读 `AI_proj_note/rule.md`、`AI_proj_note/index.md` 和 `progress.md` 的 CURRENT 区块，再按任务读取相关 Section。

### 2.2 L0：每次开发必读

1. 当前用户要求；
2. `rule.md`：只含当前有效约束，应保持短小；
3. `index.md`：本轮需要的路由；
4. `progress.md` 的 `CURRENT`：当前任务、下一步和 Git 记录基线。

### 2.3 L1：按任务读取

| 当前任务/事件 | 通过 index.md 读取 |
|---|---|
| 修改已有文件 | `BLOCKS` 中相关模块、`FILES` 中相关路径 |
| 新模块、接口或数据结构设计 | `BLOCKS`、`FUTURE`、`REUSE` |
| UI 开发或改版 | `UI`、`TOKENS`、`UI_REUSE`、`USER_PREF` |
| 恢复中断任务 | `CURRENT` 及对应 `PROGRESS` 条目 |
| 排查相似故障 | 搜索 `LESSONS` |
| 服务器部署或运维 | `OPS` 及相关模块条目 |
| 新功能设计前 | `REUSE`、`UI_REUSE`、`FUTURE` |

### 2.4 L2：仅明确需要时读取

`PREVIEW`、完整 `HISTORY`、早期 `PROGRESS`、决策归档和全项目代码考古。只有初始化、历史回溯或用户明确要求审计时才加载。

## 3. 项目字典与锚点协议

### 3.1 字典职责

`AI_proj_note/index.md` 是唯一路由表。每个 Section 必须定义：

- 文件相对路径；
- 唯一锚点；
- 读取/写入触发；
- 写入方式：`UPSERT`、`REPLACE` 或 `INSERT_NEWEST`；
- 唯一键：如规则 ID、模块路径、文件路径、commit hash。

其他笔记通过 Section ID 引用，不复制字典。文件移动时更新字典并检查引用。

### 3.2 两类真实锚点

真实锚点只存在于目标笔记文件，不存在于 SKILL 或 rule：

- 固定单例区块：`APN:BEGIN:<SECTION>` 与 `APN:END:<SECTION>`，用于 `REPLACE`；
- 条目插入点：`APN:INSERT:<SECTION>`，用于 `UPSERT` 的新条目或 `INSERT_NEWEST`。

锚点在单文件内必须唯一。操作时搜索完整锚点，例如 `<!-- APN:INSERT:HISTORY -->`；锚点缺失时停止本次写入并局部修复，禁止选择“看起来合适”的位置。

### 3.3 通用操作

- **READ(section)**：查字典 → 搜索锚点/唯一键 → 只读目标区块。
- **UPSERT(section, key)**：搜索 key；存在则原地更新，不存在才在指定锚点后插入。
- **REPLACE(section)**：只替换 BEGIN/END 之间的内容。
- **INSERT_NEWEST(section, key)**：先搜索 key 去重，再插入锚点后；旧条目保持原顺序。
- **VERIFY**：复查目标区块和 diff，确认地址、唯一键、顺序及修改范围正确。

## 4. 事件路由与写入事务

### 4.1 用户话语和代码事件的默认路由

| 事件 | Section | 唯一键 |
|---|---|---|
| “以后默认这样”、新规范、技术约束 | `RULES` | `RULE-xxx` |
| “不要这个方案”、否决、短路、缓兵之计 | `DECISIONS` | `DEC-xxx` |
| 新修改要求 | `REQUESTS`；完成后转 `PROGRESS` | `REQ-xxx` |
| “以后要做”、暂缓、预留接口 | `FUTURE` | `FUT-xxx` |
| 文件职责、接口、隐藏约束变化 | `FILES` | 文件路径 |
| 模块职责、依赖、产物变化 | `BLOCKS` | 模块路径 |
| UI、视觉 token、组件、样式或偏好变化 | `UI` / `TOKENS` / `UI_REUSE` / `USER_PREF` | 页面、token、路径或主题 |
| 产生可复用代码或模块 | `REUSE` | 路径 + 名称 |
| 用户澄清产品定位或模型分析 | `PREVIEW` | 单例区块 |
| 运维配套文件或分析结论变化 | `OPS` | `OPS-xxx` |
| 任务开始、完成或中断 | `CURRENT` / `PROGRESS` | 任务 ID |
| 难排查问题得到解决 | `LESSONS` | `LESSON-xxx` |

### 4.2 每次写入的固定事务

1. 识别一个明确事件，选择唯一归属 Section；
2. 查 `index.md`，获取文件、锚点、写入方式和唯一键；
3. 搜索唯一键并局部读取，不默认加载全文；
4. 执行一种写操作；存在同键条目时不得新建；
5. 需要跨文件引用时只写 Section ID 或路径，不复制事实；
6. VERIFY 后才算完成。

用户口头澄清、否决或纠正必须当场回写。代码漂移以代码修正“当前实现”记录；用户意图漂移只能依据用户新表述修正。

## 5. Git 历史：hash 幂等状态机

### 5.1 提交类型与唯一事实源

- **功能提交**：包含实质代码/文档变更，可以生成一条 `HISTORY`。
- **笔记同步提交**：提交信息以 `notesync:` 开头，只同步笔记，永远不生成历史。
- commit hash 是历史唯一键；`git log` 是提交事实源，模型不得通过数 Markdown 条目判断状态。
- 无实质变更时不创建空提交，也不写历史。

### 5.2 正常一轮开发

1. 完成功能、测试和应有的笔记更新，执行提交前密钥与 `.gitignore` 检查；
2. 创建功能提交，取得 hash `C = git rev-parse HEAD`；
3. 在 `HISTORY` 搜索 `C`：已存在则跳过，不存在才插入一条；
4. 更新 `CURRENT.last_change_commit = C`、`CURRENT.last_history_commit = C` 和当前任务状态；
5. 如需提交上述笔记变化，创建一个 `notesync:` 提交；该提交不再写历史。

历史条目至少包含：commit、日期、标题、本轮提示词摘要、上一版本用户评价（没有则写“无”）、已实现功能、涉及文件。不得为了获得 hash 而把“包含自身 hash 的文件”塞进同一个提交。

### 5.3 接手或漏记恢复

1. 从 `CURRENT` 读取 `last_history_commit = H`；
2. 用 `git log` 找出 H 之后的功能提交，排除 `notesync:`；
3. 按从旧到新处理，每条先按 hash 去重，再插入 `HISTORY` 锚点后，最终文件仍为新到旧；
4. 全部处理后，将 `last_history_commit` 更新为最新已处理功能提交；
5. H 为空时只记录用户要求或能够可靠摘要的范围，不擅自补写全部历史；H 因 rebase 等原因不存在时，暂停并请用户确认基线。

补记旧提交时只能依据 commit message 和 diff 写“来源：Git 补记”；不知道的用户评价或提示词不得编造。

## 6. 初始化考古与日常维护分离

- **初始化考古是一次性动作**：按 [project-setup.md](references/project-setup.md) 建立基线，详细背景放冷文件，进入 rule 的只能是当前有效短约束。
- **日常维护只处理增量事件**：不因新会话重新扫描全项目，不把考古细节反复带回注意力。
- **rule 只保留现役内容**：失效规范、否决和修改要求移入 `DECISION_ARCHIVE` 或 `PROGRESS`，保留原因与日期。
- **S 优先**：`FILES` 优先记录代码表面看不出的隐藏约束，不复述整段代码。
- **单一事实源**：一个事实只有一个归属 Section，其他地方引用。

## 7. 按需参考文件

- 初始化、默认开发规范、模块切割与考古分类：[project-setup.md](references/project-setup.md)
- 创建或修复字典、九个常驻文件、GitHistory 与全部锚点：[templates.md](references/templates.md)

不要在普通维护轮次预先读取上述参考文件。
