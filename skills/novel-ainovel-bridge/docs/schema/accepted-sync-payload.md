# Accepted Sync Payload Schema

适用 skill：`novel-ainovel-bridge`

## 目标

`sync` 只接收 AI-Novel 的 accepted / final 结果，拒绝 draft / polish 中间态。

## 最小字段

- `chapter`
- `accepted`（必须为 `true`）
- `summary`
- `state_updates`
- `hook_updates`

## 推荐字段

- `timeline_events`
- `relationship_changes`
- `source_paths`
- `accepted_at`

## 回流目标

最常见的回流位置：

- `06_reports/chapter_summaries/chNNN.md`
- `05_state/current_state.md`
- `05_state/pending_hooks.md`
- `05_state/timeline.md`（如果存在）
- `02_bible/relationship_matrix.md`（或后续关系账本）

## 原则

1. 没有 `accepted: true` 就拒绝回流。
2. draft / polish payload 不得进入 canon。
3. state / hook / timeline / relationship 的变更应可追溯到 `source_paths`。
