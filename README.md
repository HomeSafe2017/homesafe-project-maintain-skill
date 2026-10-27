# Project Maintain Skill（AI 项目笔记管理）

一个面向 AI Agent（Trae、Claude Code、Cursor、Codex 等）的 **Skill**：指导模型在长期开发中维护 `AI_proj_note/` 认知资产，让换人、换 Agent、换对话时项目认知不丢。

> 灵感来源：[AOCI-CODE](https://github.com/aoci-spec/aoci-code) 的“认知是可版本化资产”思想。Git 管理代码版本，本 Skill 管理项目认知。

## 解决什么问题

- 新会话或上下文压缩后，Agent 对项目“失忆”；
- 用户口头澄清、否决过的方案随对话蒸发；
- 笔记越写越长，真正有效的规范被历史日志淹没；
- Agent 不读完整文件，反复添加历史，无法可靠判断是否归档；
- 日志有时写头部、有时写尾部，文件逐渐混乱；
- 首次代码考古很清楚，后续维护却因上下文过长而注意力溃散。

本 Skill 的核心不是要求模型“更认真地记住”，而是把维护过程变成**可寻址、可去重、可验证的局部操作**。

## 核心设计

### 1. 三层寻址结构

- `SKILL.md`：通用协议，定义 READ、UPSERT、REPLACE、INSERT_NEWEST、VERIFY；
- `AI_proj_note/index.md`：项目字典，记录 Section 对应的文件、锚点、触发事件、写入方式和唯一键；
- 各笔记文件：保存真实事实和固定锚点。

字典只回答“去哪读、去哪写、怎么写”，不复制项目知识。

### 2. 分层读取，减少注意力稀释

- **L0，每次开发必读**：精简的 `rule.md`、`index.md`、`progress.md` 的 CURRENT；
- **L1，按任务读取**：UI、模块、文件、复用资产、未来意向或相关踩坑记录；
- **L2，按需读取**：完整历史、产品分析、早期日志和全项目考古。

不再要求新 Agent 每次顺序读完所有笔记。

### 3. 稳定规范与动态历史分离

`rule.md` 只保存当前有效规范、否决、修改要求和运维约束，**不保存 Git History**。

```text
AI_proj_note/
├── index.md          # 项目字典与路由表
├── rule.md           # 当前有效约束，无动态日志
├── files.md          # 文件级 FRAS 地图与可复用资产
├── blocks.md         # 模块职责、依赖、涉及文件、产物清理级别
├── vision_prefer.md  # UI、视觉 token、组件样式、用户偏好
├── preview.md        # 模型的产品分析，区分事实与推断
├── note.md           # 未来意向与预留接口
├── progress.md       # CURRENT 单例状态与任务日志
├── lessons.md        # 踩坑与可复现解法
└── GitHistory.md     # 按需创建：里程碑、决策归档、功能提交历史
```

### 4. 固定锚点与唯一键

- 单例状态使用 `APN:BEGIN:* / APN:END:*`，只替换区块内部；
- 日志使用 `APN:INSERT:*`，统一新到旧，插入锚点后；
- 规范、模块、文件、任务和历史分别以 ID、路径或 commit hash 去重；
- 写入前先搜索唯一键，存在则更新，不存在才新增；
- 锚点缺失时停止写入并局部修复，不让模型猜测位置。

没有“最近七条”计数，也不让模型搬运归档。

### 5. Git 历史幂等状态机

- **功能提交**：可以生成一条历史，commit hash 是唯一键；
- **`notesync:` 笔记同步提交**：只提交笔记，永远不生成历史，避免递归记录；
- `CURRENT.last_history_commit` 保存已处理基线；
- 接手时通过 `git log` 找出漏记提交，而不是数 Markdown 条目；
- 旧提交补记只能依据 commit message 和 diff，不编造用户提示词或评价。

### 6. 初始化考古与日常维护分离

首次接触已有项目时建立一次基线；日常只处理增量事件，不在每个新会话重新扫描全项目。

代码可以证明当前实现，但不能证明用户曾否决什么。推断必须标注“推断、待确认”，不得伪装成用户决定。

## 仓库结构

```text
homesafe-project-maintain-skill/
├── SKILL.md                      # 主协议，日常加载
├── references/
│   ├── project-setup.md          # 初始化、默认规范、模块切割与考古分类
│   └── templates.md              # 字典、九文件、GitHistory 与锚点模板
├── README.md
└── LICENSE
```

参考文件采用渐进加载：只有初始化、结构修复或模块规划时才读取，不占普通维护轮次的注意力。

## 安装

将整个 `homesafe-project-maintain-skill` 目录安装为技能目录，并保留 `SKILL.md` 与 `references/` 的相对位置。

Trae 全局安装示例：

```text
C:\Users\<用户名>\.trae-cn\skills\homesafe-project-maintain\
├── SKILL.md
└── references\
```

其他 Agent 工具可使用其 Skill/规则安装方式，但应同时提供参考文件，不能只复制 SKILL.md 后又要求初始化模板。

首次在项目中使用时，先确认启用；Agent 随后创建 `AI_proj_note/`，写入项目级记忆并开始按协议维护。

## License

[MIT](LICENSE)
