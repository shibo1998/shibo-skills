# Context Loading

## 总原则

1. 先读控制面，再读正文面。
2. 默认最小必要上下文。
3. 扩窗必须有理由，不为心理安慰追读全书。

## 默认读取顺序

1. `project_manifest.yaml`
2. `05_state/current_state.md`
3. `03_outline/chapter_index.md`
4. 目标章章纲
5. 最近 2-3 章摘要
6. `05_state/pending_hooks.md`
7. 必要角色卡
8. 必要设定片段
9. 只有在需要精确衔接时，才补读最近 1-3 章正式正文

## 按任务类型加载

### 立项 / 设定类
读 brief、bible、characters、relationship、outline。一般不读大量正文。

### 写作 / 续写类
读状态卡、目标章纲、最近摘要、伏笔池。需要时再补正文。

### 审查类
读当前章正式稿、章纲、最近摘要、状态卡、伏笔池、必要设定。

### 状态维护类
读当前正式稿、状态卡、伏笔池、时间线、资源账本、章节索引。

## 扩窗条件

满足任一项才扩窗：

- 连续战斗或连续场景接续
- 长线伏笔回收
- 跨卷因果
- 审查时发现控制面不足以解释冲突
- 用户明确要求按原文精确衔接

## Context Pack

如果需要结构化上下文，直接在当前回复里整理并落盘一个最小结构块：

- task_type
- target_chapter
- current_story_state
- required_constraints
- recent_story_memory
- active_hooks
- character_focus
- world_rules
- research_notes
- unresolved_risks
