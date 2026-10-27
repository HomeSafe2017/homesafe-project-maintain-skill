# AI_proj_note 标准模板

> 仅用于初始化或局部修复。真实锚点写入项目笔记，不写入 rule 的说明文字。不要给普通内容逐条增加机器锚点。

## 1. index.md

```markdown
# AI_proj_note 项目字典

> 只保存地址与操作，不保存项目事实。路径相对于 AI_proj_note/。
> 所有写入先搜索唯一键；UPSERT 存在即更新，INSERT_NEWEST 只能插入锚点后。

| Section ID | 文件 | 锚点 | 主要触发 | 写入方式 | 唯一键 |
|---|---|---|---|---|---|
| RULES | rule.md | APN:INSERT:RULES | 开发前读取；新规范 | UPSERT | RULE-ID |
| DECISIONS | rule.md | APN:INSERT:DECISIONS | 开发前读取；否决/短路 | UPSERT | DEC-ID |
| REQUESTS | rule.md | APN:INSERT:REQUESTS | 新修改要求 | UPSERT | REQ-ID |
| OPS | rule.md | APN:INSERT:OPS | 运维前读取；配套变化 | UPSERT | OPS-ID |
| FILES | files.md | APN:INSERT:FILES | 修改文件前后 | UPSERT | 文件路径 |
| REUSE | files.md | APN:INSERT:REUSE | 设计前读取；新复用资产 | UPSERT | 路径/包名+名称 |
| BLOCKS | blocks.md | APN:INSERT:BLOCKS | 模块设计/变更 | UPSERT | 模块路径 |
| UI | vision_prefer.md | APN:INSERT:UI | UI 开发/改版 | UPSERT | 页面/路由 |
| TOKENS | vision_prefer.md | APN:INSERT:TOKENS | UI 开发；token 变化 | UPSERT | token 名 |
| UI_REUSE | vision_prefer.md | APN:INSERT:UI_REUSE | UI 设计；组件/样式变化 | UPSERT | 路径/包名+名称 |
| USER_PREF | vision_prefer.md | APN:INSERT:USER_PREF | 用户表达偏好 | UPSERT | 偏好主题 |
| PREVIEW | preview.md | APN:BEGIN:PREVIEW | 需要产品分析时 | REPLACE | 单例区块 |
| FUTURE | note.md | APN:INSERT:FUTURE | 设计前读取；未来意向 | UPSERT | FUT-ID |
| CURRENT | progress.md | APN:BEGIN:CURRENT | 每次接手；状态变化 | REPLACE | 单例区块 |
| PROGRESS | progress.md | APN:INSERT:PROGRESS | 任务完成/中断 | INSERT_NEWEST | 任务 ID |
| LESSONS | lessons.md | APN:INSERT:LESSONS | 相似故障；新解法 | INSERT_NEWEST | LESSON-ID |
| MILESTONES | GitHistory.md（按需） | APN:INSERT:MILESTONES | 重大版本/决策 | UPSERT | MILESTONE-ID |
| DECISION_ARCHIVE | GitHistory.md（按需） | APN:INSERT:DECISION_ARCHIVE | 现役决策失效 | INSERT_NEWEST | DEC-ID |
| HISTORY | GitHistory.md（按需） | APN:INSERT:HISTORY | 回溯；功能提交后 | INSERT_NEWEST | commit hash |
```

## 2. rule.md

```markdown
# 开发规范与用户要求

> 本文件只保存当前有效约束，是项目最高优先级笔记；不保存 Git 历史或任务流水账。
> 笔记读写地址见 index.md。失效规范、否决和要求移入 DECISION_ARCHIVE 或 PROGRESS。

## 硬性规范
> 格式：RULE-xxx | 日期 | 来源：默认规范/用户原话 | 规范内容
<!-- APN:INSERT:RULES -->

## 当前有效否决与决策
> 格式：DEC-xxx | 日期 | 来源 | 方案/文件 | 原因 | 当前替代做法
<!-- APN:INSERT:DECISIONS -->

## 进行中的修改要求
> 格式：REQ-xxx | 日期 | 要求 | 涉及文件 | 状态；完成后移入 PROGRESS
<!-- APN:INSERT:REQUESTS -->

## 服务器运维配套记录
> 格式：OPS-xxx | 日期 | 文件/会话及位置 | 用途 | 变化与原因 | 分析结论
<!-- APN:INSERT:OPS -->
```

## 3. files.md

```markdown
# 文件认知地图

> 只覆盖关键文件。F=职责，R=修改时必须一起看的文件，A=对外接口，S=代码表面看不出的隐藏约束。
> 格式：路径 | F:... | R:... | A:... | S:...
<!-- APN:INSERT:FILES -->

## 可复用资产
> 格式：名称 | 来源：内部路径/外部包名@版本 | 能力关键词 | 接口/契约 | 适用与不适用场景 | 扩展点/包装层路径 | 使用方
> 外部资产另附官方 URL 与许可证；内部资产无对应字段时写“无”，不要编造。
<!-- APN:INSERT:REUSE -->
```

## 4. blocks.md

```markdown
# 模块认知地图

> 每个模块以路径为唯一键，新增模块使用以下结构。
<!-- APN:INSERT:BLOCKS -->

<!-- 条目结构示例，不作为真实模块记录
## 模块：[模块路径]
- 职责：
- 输入 / 输出：
- 复用决策：A/B/C/D/E、选择理由；E 需说明 A–D 不适用原因
- 候选资产/外部库：内部路径或官方 URL、包名与版本
- 稳定核心 / 变化点 / 公共契约：
- 依赖：外部依赖注明版本与许可证
- 约束：
- 测试方式：
- 对外入口：
- 涉及文件：
- 使用/产生的文件：[类型与地址]（自动清理/允许清理/重点保存）
-->
```

## 5. vision_prefer.md

```markdown
# 产品 UI 与用户偏好

## 产品 UI
> 格式：页面/路由 | 界面结构 | 视觉风格 | 交互约定 | 明确不做
<!-- APN:INSERT:UI -->

## 视觉 Token
> 格式：token 名 | 取值 | 定义文件 | 用途
<!-- APN:INSERT:TOKENS -->

## 可复用组件与样式
> 格式：组件/样式名 | 来源：内部路径/外部包名@版本 | 能力关键词 | 属性/事件/插槽或样式契约 | 适用与不适用场景 | 主题与扩展点/包装层路径 | 使用方
> 外部资产另附官方 URL 与许可证；内部资产无对应字段时写“无”，不要编造。
<!-- APN:INSERT:UI_REUSE -->

## 用户偏好
> 格式：偏好主题 | 偏好内容 | 来源 | 日期
<!-- APN:INSERT:USER_PREF -->
```

## 6. preview.md

```markdown
# 产品分析

> 本文件记录模型理解，不是用户事实。推断必须标注“推断、待确认”；引用事实只写来源 Section ID。
<!-- APN:BEGIN:PREVIEW -->
## 产品定位
## 核心功能需求（我的理解与拆解）
## 使用场景
## 使用人群
## 演进方向
## 疑问与待确认
<!-- APN:END:PREVIEW -->
```

## 7. note.md

```markdown
# 未来开发意向与预留接口

> 格式如下；来源必须区分“用户原话/推断”。被引用路径失效时立即修正。
<!-- APN:INSERT:FUTURE -->

<!-- 条目结构示例，不作为真实意向记录
## FUT-xxx | 日期 | 意向一句话（来源：用户原话/推断）
- 需求描述：
- 涉及模块：[Section ID / 模块路径]
- 建议预留接口或格式：
- 状态：暂缓 / 已纳入开发
-->
```

## 8. progress.md

```markdown
# 任务进展

## CURRENT：当前唯一状态
<!-- APN:BEGIN:CURRENT -->
task_id: none
status: idle
current_task: 无
next: 等待任务
last_change_commit:
last_history_commit:
updated:
<!-- APN:END:CURRENT -->

## 任务日志（新到旧）
> 任务开始先更新 CURRENT；完成或中断时在锚点后插入一条，再更新 CURRENT。
<!-- APN:INSERT:PROGRESS -->

<!-- 条目结构示例，不作为真实任务记录
### 日期 | 任务 ID | 任务名
- 完成：
- 进行中/阻塞：
- 下一步：
- 涉及文件：
-->
```

## 9. lessons.md

```markdown
# 踩坑记录

> 只记难排查问题、环境坑、失败方案的结论与可复现解法，不记过程流水账。
<!-- APN:INSERT:LESSONS -->

<!-- 条目结构示例，不作为真实踩坑记录
## LESSON-xxx | 日期 | 问题一句话
- 现象：
- 根因：
- 解法：
- 验证：
- 相关文件：
-->
```

## 10. GitHistory.md（首次使用时创建）

```markdown
# 项目演进与历史

> 本文件是冷存储，不需要在每次开发前通读。所有日志新到旧，按唯一键去重，不按条数归档。

## 里程碑
> 只记重大版本或用户确认的重大决策。
<!-- APN:INSERT:MILESTONES -->

## 决策归档（新到旧）
> 保存从 rule.md 移出的失效规范、否决和决策，保留原 DEC/RULE ID、失效日期与原因。
<!-- APN:INSERT:DECISION_ARCHIVE -->

## 功能提交历史（新到旧）
> 只记录功能提交；notesync: 提交永不写入。插入前必须搜索 commit hash。
<!-- APN:INSERT:HISTORY -->

<!-- 条目结构示例，不作为真实提交记录
### commit | 日期 | 短标题
- 本轮提示词摘要：
- 上版本用户评价：无/具体内容
- 已实现功能：
- 涉及文件：
- 来源：当轮对话 / Git 补记
-->
```
