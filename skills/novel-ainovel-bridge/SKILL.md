---
name: novel-ainovel-bridge
description: "Bridge a generic novel control-plane project into AI-Novel. Use this whenever the user mentions AI-Novel / ainovel, wants to 导出给 AI-Novel, 喂给项目, 生成 ainovel_feed, or wants to sync accepted chapter results back from AI-Novel into a generic novel project. This skill does not create story canon from scratch; it reads the outputs of the generic novel skill and maps them into AI-Novel specific files and conventions. Trigger aggressively for requests like 喂给 AI-Novel、导出给项目、生成 handoff/feed、同步项目结果回控制面, or explicit slash usage such as /novel-ainovel-bridge."
version: 1.0.0
user-invocable: true
argument-hint: "[export|sync|resume] [项目或路径]"
---

# Novel → AI-Novel Bridge

以简体中文工作。代码、命令、路径、文件名保持原文。

这个 skill 是**项目适配层**，不是正文创作 skill。

它的职责只有两类：

1. 把通用 `novel` 或 `novel-framework` skill 产出的控制面导出成 **AI-Novel** 可消费的 feed 包
2. 把 **AI-Novel** 已 accepted 的章节结果回流到通用小说项目的控制面文件

它默认不负责：

- 新建世界观与角色设定本身
- 直接写正式正文
- 代替 AI-Novel 的 writer / editor / polisher 流水线

## 何时使用

当用户出现下面任一意图时，使用本 skill：

- “喂给 AI-Novel”
- “导出给项目”
- “生成 ainovel_feed”
- “把 chapter context / 角色 / 设定转成项目能吃的格式”
- “把 AI-Novel 的 accepted 结果同步回控制面”
- “桥接 novel skill 和 AI-Novel”

如果用户只是要做小说设定、大纲、状态维护，而没有项目适配诉求，应该使用 `novel` 或 `novel-framework`，而不是本 skill。

## 输入前提

默认要求已经存在一个通用小说项目，至少具备：

- `project_manifest.yaml`
- `01_brief/project_brief.md`
- `02_bible/story_bible.md`
- `02_bible/character_cards.md`
- `03_outline/`
- `05_state/current_state.md`
- `05_state/pending_hooks.md`

如果这些控制面文件缺失，不要硬造 feed；先指出缺口，并让用户回到 `novel` skill 补齐。

## 任务类型

### 1. `export`

把通用小说项目导出成 AI-Novel feed：

```text
07_exports/ainovel_feed/
├── manifest.yaml
├── premise.md
├── characters.json
├── world_rules.json
├── outline.json
├── foreshadows.json
├── chapter_context/
│   └── chNNN.yaml
└── overrides/
    ├── writer.override.md
    └── reviewer.override.md
```

### 2. `sync`

把 AI-Novel 已 accepted 的结果回流到通用控制面，例如：

- `06_reports/chapter_summaries/`
- `05_state/current_state.md`
- `05_state/pending_hooks.md`
- 必要时更新时间线、资源账本、关系表

### 3. `resume`

恢复上次 bridge 现场，继续 export 或 sync。

## 工作流程

### export

```text
检查通用控制面是否完整
→ 读取 brief / bible / characters / outline / state / hooks
→ 组装 AI-Novel feed
→ 增量写入 07_exports/ainovel_feed/
→ 返回导出结果与缺口提示
```

### sync

```text
定位 AI-Novel accepted 结果
→ 检查是否为 accepted / final 结果
→ 提取摘要 / 状态变化 / 伏笔推进
→ 回流到通用控制面
→ 标注同步范围与未解决风险
```

## sync 输入契约

为了让 `sync` 可重复执行，默认要求用户提供或项目中可定位到一组 **accepted result payload**。

最少应包含：

- `chapter`: 章节号
- `accepted`: `true`
- `summary`: 本章摘要
- `state_updates`: 当前状态变化列表
- `hook_updates`: 伏笔新增 / 推进 / 回收列表

推荐附加：

- `timeline_events`
- `relationship_changes`
- `source_paths`
- `accepted_at`

如果只有 draft / polish 中间态，而没有 accepted / final 结果，必须拒绝回流。

## 读取优先级

### export 时

1. `project_manifest.yaml`
2. `01_brief/project_brief.md`
3. `02_bible/story_bible.md`
4. `02_bible/character_cards.md`
5. `03_outline/chapter_outlines/*`
6. `05_state/current_state.md`
7. `05_state/pending_hooks.md`
8. 必要时最近章节摘要

### sync 时

1. AI-Novel accepted 章节结果
2. AI-Novel review / summary / progress
3. 通用项目现有 `current_state.md`
4. 通用项目现有 `pending_hooks.md`
5. 其他状态文件

## 导出原则

- `premise.md`：融合 brief / bible / 核心关系 / 风格要求
- `characters.json`：转成 AI-Novel 可直接消费的角色卡结构
- `world_rules.json`：只保留对正文写作与一致性检查有用的规则边界
- `outline.json`：导出卷弧章结构，不把散文说明塞进去
- `foreshadows.json`：只导出仍然有效的 planted / hinted / resolved 信息
- `chapter_context/chNNN.yaml`：必须围绕本章必须保留、必须回避、情绪目标、hook 目标来写
- `overrides/*`：用于把项目级文风和审稿偏好交给 AI-Novel
- `sync` 时若没有 accepted payload，不要猜测状态更新

## 回流原则

- 只回流 **accepted / final** 结果
- draft / polish 中间态不回流为 canon
- 如果 AI-Novel 输出与控制面冲突：
  - accepted 正文事实优先于旧状态卡
  - 但不自动改世界观根规则，除非冲突已经在 accepted 结果中稳定存在且用户确认保留

## 输出规则

- export 模式：先说明将导出哪些 feed 文件，再执行
- sync 模式：先说明将同步哪些控制面文件，再执行
- 如发现缺口：明确列出缺的文件，不假装能继续
- 如只有 draft / polish 中间态：明确拒绝，并指出缺少 accepted result payload

## 参考资源

- feed 结构：`docs/schema/ainovel-feed.md`
- bridge 工作流：`references/workflow.md`
- feed 模板：`templates/*`
