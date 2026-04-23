# Framework Export Schema

适用 skill：`novel-framework`

## 目标

`novel-framework export` 只导出控制面，不导出正文。

## 默认导出物

### 1. `07_exports/context_packs/chNNN_context.yaml`

用于单章级控制面交付，至少包含：

- `chapter`
- `title`
- `must_keep`
- `must_avoid`
- `emotion_target`
- `hook_goal`
- `continuity_notes`

推荐附加：

- `voice_notes`
- `required_beats`
- `forbidden_moves`
- `active_hooks`
- `state_focus`

### 2. `07_exports/framework_bundle.md`

用于整本框架摘要导出，至少汇总：

- project 基本信息
- brief 核心承诺
- story bible 核心规则
- 核心角色表
- 卷弧章结构快照
- 当前状态
- 伏笔池

## 原则

1. export 只导出控制面。
2. 不输出任何 prose。
3. 用户未指定时，优先导出章节上下文包。
4. 用户明确要求“整本框架包”时，再导出 `framework_bundle.md`。
