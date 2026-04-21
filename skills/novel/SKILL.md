---
name: novel
description: Create, plan, write, review, maintain, and resume long-form fiction projects with a file-based one-book-one-folder workflow. Use this whenever the user wants to start a novel, build story bible and outlines, write or revise chapters, maintain current story state, track hooks and continuity, diagnose long-form consistency, or resume a fiction project across sessions. Also use it for reality-grounded fiction that needs on-demand research and fact verification. Trigger aggressively for requests like “开一本新书”, “写第12章”, “续写”, “审稿”, “同步状态”, “做卷纲/章纲”, “恢复上次创作现场”, or explicit slash usage such as /novel.
version: 3.0.0
user-invocable: true
argument-hint: "[init|brief|bible|characters|outline|write|revise|review|state|diagnose|research|export|resume] [项目或需求]"
---

# Universal Novel Studio

以简体中文工作。代码、命令、路径、文件名保持原文。

这是一个面向长篇小说项目的完整工作台型 skill，不是一次性写文 prompt。  
它默认采用：

- 一书一目录
- 先立项再写
- 强闭环维护
- 按需联网考据
- 文件系统作为持久记忆
- 模板驱动落盘

## 何时使用

只要用户要做下面任一件事，就应使用本 skill：

- 新建小说项目
- 明确题材、卖点、目标读者、风格和禁忌
- 建立故事圣经、角色卡、关系表、卷纲、章纲
- 写章节草稿、续写、补写、扩写、改写、重写
- 检查 OOC、设定冲突、时间线问题、节奏问题、AI 味
- 更新当前状态卡、伏笔池、章节摘要、时间线、资源账本
- 恢复上次创作现场并继续推进
- 为现实题材或现实细节做联网核验

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

默认文件命名：

- 章纲：`03_outline/chapter_outlines/chNNN.md`
- 草稿：`04_manuscript/drafts/chNNN_draft.md`
- 正式稿：`04_manuscript/chapters/chNNN.md`
- 摘要：`06_reports/chapter_summaries/chNNN_summary.md`
- 审查：`06_reports/reviews/chNNN_review.md`
- 诊断：`06_reports/diagnostics/chNNN_diag.md`

## 先读哪些 references

按任务需要读取：

- `references/workflow.md`：新建项目、做卷纲、走完整主链时读取
- `references/context-loading.md`：写作、续写、审查、恢复时读取
- `references/research-policy.md`：需要事实核验时读取
- `references/quality-rubric.md`：做 review / diagnose 时读取
- `references/revision-rules.md`：改写、重写、扩写时读取
- `references/genre-notes.md`：题材差异显著时读取

不要一次性把所有 reference 全塞进上下文。按任务最小读取。

## 支持的任务类型

先判断当前任务属于哪一类：

1. `init`：创建项目
2. `brief`：立项定盘
3. `bible`：设定圣经
4. `characters`：角色与关系
5. `outline`：卷纲 / 章纲
6. `write`：正文草稿
7. `revise`：润色 / 改写 / 重写 / 扩写
8. `review`：质量审查
9. `state`：状态维护
10. `diagnose`：章节级或项目级诊断
11. `research`：联网考据
12. `export`：导出成果
13. `resume`：恢复续写

如果用户只是闲聊、比题材、问方向是否成立，进入讨论模式，不默认落盘。

## 命令面

显式命令优先：

- `/novel init`
- `/novel brief`
- `/novel bible`
- `/novel characters`
- `/novel outline`
- `/novel write`
- `/novel revise`
- `/novel review`
- `/novel state`
- `/novel diagnose`
- `/novel research`
- `/novel export`
- `/novel resume`

自然语言兜底路由：

- “帮我开一本新书” → `init + brief`
- “先帮我把这本小说的卖点和读者定位定下来” → `brief`
- “补一下世界观和设定铁律” → `bible`
- “整理主角和反派的关系变化” → `characters`
- “做第一卷卷纲” → `outline`
- “列一下第12章章纲” → `outline`
- “继续写第12章” → `write`
- “把这一章改顺一点，但别动结构” → `revise` with `polish`
- “重写这一章” → `revise` with `rewrite`
- “检查第12章有没有 OOC” → `review`
- “同步一下现在的状态” → `state`
- “看看有哪些伏笔还没收” → `state` or `diagnose`
- “查一下 1993 年香港巡警通讯设备” → `research`
- “接着上次继续” → `resume`

## 默认工作流

```text
项目初始化
→ 立项定盘
→ 设定圣经
→ 角色与关系
→ 卷纲
→ 章纲
→ 写正文
→ 审稿/修稿/润色
→ 章节闭环维护
→ 诊断
→ 进入下一章
```

默认规则：

- 未初始化项目时，不直接写正文
- 未有章纲时，不默认直接写正式章节
- review 有 P0 时，不进入下一章
- 一章只有 `synced` 后才算真正完成

## 上下文装配规则

在写作、续写、审查、恢复时，默认读取顺序：

1. `project_manifest.yaml`
2. `05_state/current_state.md`
3. `03_outline/chapter_index.md`
4. 目标章章纲
5. 最近 2-3 章摘要
6. `05_state/pending_hooks.md`
7. 必要角色卡片段
8. 必要设定片段
9. 只有在需要精确衔接时，才补读最近 1-3 章正式正文

不要机械追读全书。扩窗必须有理由，例如：

- 连续战斗 / 连续场景 / 长对话接续
- 长线伏笔回收
- 跨卷因果
- 审查时发现状态卡不足以解释冲突
- 用户明确要求按原文精确衔接

如果需要结构化上下文包，不依赖外部 runtime，直接在回复里组装并落盘一个最小结构块，至少包含：

- `task_type`
- `target_chapter`
- `current_story_state`
- `required_constraints`
- `recent_story_memory`
- `active_hooks`
- `character_focus`
- `world_rules`
- `research_notes`
- `unresolved_risks`

## 联网考据规则

默认不因为“真实感”三个字就联网。只有在不确定的现实事实会影响可信度时才触发。

必触发场景：

- 历史年代
- 地理与机构细节
- 行业 / 职业流程
- 法律 / 政策
- 金融 / 经济 / 物价
- 武器 / 装备 / 交通工具
- 医疗 / 刑侦 / 执法流程
- 社会风俗与时代物件

工作流：

```text
发现不确定点
→ 拆成具体问题
→ 搜索与交叉验证
→ 生成 research note
→ 提炼为可写事实
→ 再进入写作或审稿
```

默认把研究结论落盘为 research note，不直接改正文。

## 修订边界

严格区分：

- `polish`：润色，不改结构主干
- `revise`：改写，允许局部重组
- `rewrite`：重写，允许大改
- `expand`：扩写，保持主干不变

改稿前读取 `references/revision-rules.md`，不要混用词义。

## 审查与强闭环

每章定稿后，默认必须完成：

1. 更新 `chapter_index.md`
2. 生成章节摘要
3. 更新当前状态卡
4. 更新伏笔池
5. 按需更新关系、时间线、资源账本
6. 运行轻量诊断

实现方式：

- 直接用 `templates/` 下对应模板生成或更新文件
- 优先增量修改现有文件，不整页重写已有内容
- 表格型文件优先只改对应行
- 若缺模板字段，先补齐字段再写入内容

审查问题分级：

- `P0`：阻断级，未修不能进入下一章
- `P1`：高优先级，原则上本章定稿前修复
- `P2`：优化级，可并入润色阶段

## 什么时候必须落盘

优先把可重复、可结构化、可校验的动作下沉到**模板化落盘**，而不是外部运行时脚本。

必落盘场景：

- 初始化一书一目录项目
- 构建章节或项目级上下文包
- 章节摘要
- 审查报告
- 章节级 / 项目级诊断
- 状态卡 / 伏笔池 / 时间线 / 资源账本更新
- research note
- 导出章节或全书

如果自动化模板不足，就按现有模板手工补齐，不依赖 Python 或额外 CLI。

## 输出规则

默认输出遵守：

- 讨论模式：只给结论和建议，不落文件
- 项目模式：说明将更新哪些文件，然后执行
- 审查模式：先给审查报告，再决定是否修改正文
- 研究模式：先给结论摘要和落盘路径，不自动覆盖正文

所有文件更新优先写到项目目录，不在根目录乱放文档。

## 失败与冲突处理

冲突优先级：

```text
用户本轮硬要求
> 最新正式章节
> current_state.md
> 当前章章纲
> 最近章节摘要
> story_bible.md
> 角色卡/关系表
> 卷纲
> 更旧资料
```

处理规则：

- 状态卡和最新正式章节冲突：以最新正式章节为准，先修状态卡
- 章纲和设定圣经冲突：以设定圣经为准，除非用户明确要求重构
- 摘要和原文冲突：以原文为准，重写摘要
- research 来源冲突：记录冲突点，给保守写法，不装作已确定
- review 存在 P0：阻断下一章

## 交付标准

一章只有在以下条件都满足时，才算完成：

- 正式稿存在
- review 完成
- 无 P0
- 摘要完成
- 状态卡同步
- 伏笔池同步
- 诊断通过

如果用户请求“接着写下一章”，而上一章未完成闭环，先补闭环，再继续。
