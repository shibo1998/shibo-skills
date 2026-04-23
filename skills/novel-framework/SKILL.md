---
name: novel-framework
description: "Create, plan, maintain, and export the control-plane of a long-form fiction project without generating正文内容. Use this whenever the user wants only the novel framework: overall background, worldbuilding, setting rules, character roles, personality traits, factions, volume or chapter outlines, hooks, plot progression, current state, pending hooks, or chapter contracts — and explicitly does NOT want prose output. ALWAYS treat explicit $novel-framework or /novel-framework commands as direct entrypoints to this skill instead of generic brainstorming or planning skills. If the user asks for any sample prose, draft chapters, rewrites, or full narrative content, switch to the full novel skill instead."
version: 1.0.0
user-invocable: true
argument-hint: "[init|brief|bible|characters|outline|state|diagnose|research|export|resume] [项目或需求]"
---

# Novel Framework Studio

以简体中文工作。代码、命令、路径、文件名保持原文。

这是一个面向长篇小说项目的**纯框架** skill。  
显式 `$novel-framework ...` / `/novel-framework ...` 命令属于**直接入口**。  
它的职责是：只生成小说的控制面，不生成正文内容。

## 何时使用

只要用户要做下面任一件事，就应使用本 skill：

- 只生成小说整体背景
- 只做世界观、设定铁律、力量体系
- 只做人物角色、性格、关系、势力
- 只做卷纲、章纲、剧情骨架、钩子、故事推进设计
- 只维护当前状态卡、伏笔池、时间线、资源账本
- 只导出 chapter contract / chapter context / 通用上下文包
- 检查控制面有没有设定冲突、节奏问题、伏笔断裂
- 为现实题材框架做联网核验

如果用户要：

- 样章
- 对话试写
- 章节草稿
- 改写 / 重写正文
- 连续续写

则不要继续停留在本 skill，改用全功能 `novel`。

## 默认职责边界

本 skill 只承担**控制面**。

默认负责：

- 立项、定盘、卖点、题材、读者定位
- 世界观、设定铁律、角色卡、关系表
- 卷纲 / 弧纲 / 章纲
- 故事情节骨架、钩子、chapter contract、chapter context
- 当前状态卡、伏笔池、时间线、资源账本
- 控制面诊断与一致性检查

默认不负责：

- 样稿
- tone reference
- 章节草稿
- 正式正文
- 正文改写 / 重写 / 扩写

## 项目合同

默认项目结构：

```text
projects/{novel-slug}/
├── project_manifest.yaml
├── 01_brief/
├── 02_bible/
├── 03_outline/
├── 04_manuscript/
├── 05_state/
├── 06_reports/
└── 07_exports/
```

即使是纯框架 skill，也沿用与 `novel` 相同的项目结构，方便后续无缝切换到全功能创作或 bridge 对接。

## 支持的任务类型

1. `init`：创建项目壳子
2. `brief`：立项定盘
3. `bible`：设定圣经
4. `characters`：角色与关系
5. `outline`：卷纲 / 章纲 / chapter contract
6. `state`：状态维护
7. `diagnose`：框架级或项目级诊断
8. `research`：联网考据
9. `export`：导出通用上下文包 / chapter context / framework bundle
10. `resume`：恢复框架规划现场

## init 快速通道

`init` 是**确定性脚手架动作**，不是创意设计访谈。

当用户明确要求：

- `/novel-framework init`
- “先初始化框架项目”
- “只建骨架，不继续正文”

则必须：

- 立即在 **`projects/{slug}/`** 下创建项目目录与基础文件
- 不追问题材细节
- 不自动继续 `brief`、`bible`、`outline`
- **只写约定内的骨架文件，不自创额外文件名**

`init` 时至少应在 **`projects/{slug}/`** 下创建并优先使用这些固定文件名：

- `project_manifest.yaml`
- `01_brief/project_brief.md`
- `02_bible/story_bible.md`
- `02_bible/character_cards.md`
- `02_bible/relationship_matrix.md`
- `03_outline/chapter_index.md`
- `05_state/current_state.md`
- `05_state/pending_hooks.md`

`project_manifest.yaml` 必须满足与 `novel` 相同的 manifest 合同字段：

- `title`
- `slug`
- `genre`
- `style`
- `audience`
- `language`
- `research_mode`
- `strong_sync`
- `current_phase`
- `current_volume`
- `current_chapter`
- `last_updated`

字段要求：

- `research_mode` 只能是 `off` / `on-demand` / `strict`，默认 `on-demand`
- `strong_sync` 用字符串 `true`
- `current_phase` 默认 `initialized`
- `current_volume` / `current_chapter` 默认 `0`
- `last_updated` 用当天日期 `YYYY-MM-DD`

不要写成嵌套结构（如 `project: { ... }`），不要自创 manifest schema。
## 命令面

显式命令优先：

- `/novel-framework init`
- `/novel-framework brief`
- `/novel-framework bible`
- `/novel-framework characters`
- `/novel-framework outline`
- `/novel-framework state`
- `/novel-framework diagnose`
- `/novel-framework research`
- `/novel-framework export`
- `/novel-framework resume`

自然语言兜底路由：

- “只做小说框架” → `brief + bible + characters + outline`
- “只做背景、大纲、人物、钩子” → `brief + bible + characters + outline`
- “不要正文，只做故事骨架” → `outline`
- “同步一下控制面状态” → `state`
- “看看设定和伏笔有没有打架” → `diagnose`

## 默认工作流

```text
项目初始化
→ 立项定盘
→ 设定圣经
→ 角色与关系
→ 卷纲
→ 章纲
→ chapter contract / chapter context
→ 状态维护
→ 诊断
→ 导出通用上下文包
```

## export 产物约定

`export` 默认只导出控制面，不导出正文。

推荐导出两类产物：

### 1. 章节上下文包

路径：

- `07_exports/context_packs/chNNN_context.yaml`

最少包含：

- `chapter`
- `title`
- `must_keep`
- `must_avoid`
- `emotion_target`
- `hook_goal`
- `continuity_notes`

### 2. 框架总包

路径：

- `07_exports/framework_bundle.md`

至少汇总：

- brief 核心承诺
- story bible 核心规则
- 角色主表
- 卷弧章结构
- 当前状态卡
- 伏笔池

如果用户没指定导出目标，优先导出章节上下文包；如果用户明确说“导出整本框架”，再补框架总包。

默认规则：

- 未初始化项目时，不直接落控制面文件之外的内容
- 不写任何正文 prose
- 未有章纲时，不生成 chapter context
- `export` 只导出控制面，不导出 prose
- 框架未诊断通过时，不建议切到正文创作

## 上下文装配规则

在规划、诊断、恢复时，默认读取顺序：

1. `project_manifest.yaml`
2. `01_brief/project_brief.md`
3. `02_bible/story_bible.md`
4. `02_bible/character_cards.md`
5. `03_outline/chapter_index.md`
6. 目标章章纲
7. `05_state/current_state.md`
8. `05_state/pending_hooks.md`
9. 必要时最近章节摘要（如果项目已经进入写作期）

## 联网考据规则

默认不因为“真实感”三个字就联网。只有在不确定的现实事实会影响框架可信度时才触发。

必触发场景：

- 历史年代
- 地理与机构细节
- 行业 / 职业流程
- 法律 / 政策
- 金融 / 经济 / 物价
- 武器 / 装备 / 交通工具
- 医疗 / 刑侦 / 执法流程
- 社会风俗与时代物件

## 输出规则

默认输出遵守：

- 讨论模式：只给框架建议，不落文件
- 项目模式：说明将更新哪些控制面文件，然后执行
- 研究模式：先给研究结论摘要和落盘路径，不自动生成正文

## 与其他 skill 的关系

- 要写正文 → 用 `novel`
- 要对接 AI-Novel → 用 `novel-ainovel-bridge`
- 只做框架 → 用 `novel-framework`
