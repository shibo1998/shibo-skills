# novel-ainovel-bridge

`novel` 通用控制面 skill 的 **AI-Novel 适配层**。

## 定位

这个 skill 不是拿来直接做小说创作的，而是：

- 把 `skills/novel` 产出的控制面导出成 AI-Novel 可消费的 feed
- 把 AI-Novel 已 accepted 的结果回流到通用小说项目控制面

也就是说：

- `novel` 负责：设定 / 角色 / 大纲 / 状态 / 伏笔 / 样稿参考 / 正文创作
- `novel-framework` 负责：纯框架（背景 / 大纲 / 人设 / 钩子 / 情节骨架）
- `novel-ainovel-bridge` 负责：AI-Novel 专属 packaging / sync
- `AI-Novel` 项目负责：正文 / 细节 / 审稿 / 润色流水线

## 安装

```powershell
npx skills add shibo1998/shibo-skills --skill novel-ainovel-bridge
```

或：

```powershell
npx skills add https://github.com/shibo1998/shibo-skills --skill novel-ainovel-bridge
```

## 典型使用场景

- “把这本书喂给 AI-Novel”
- “导出 ainovel_feed”
- “把 accepted 章节结果同步回控制面”
- “桥接 novel skill 和 AI-Novel 项目”

## 最小使用流程

### export

1. 先用 `novel` 或 `novel-framework` 产出控制面
2. 再执行 bridge 导出 feed

### sync

1. 确认 AI-Novel 侧已经得到 accepted / final 结果
2. 准备 accepted payload（至少包含 `chapter`、`accepted=true`、`summary`、`state_updates`、`hook_updates`）
3. 再执行 bridge 回流控制面

桥接层不负责写正文，它只负责：
- 把控制面转成 AI-Novel 可消费格式
- 把 AI-Novel accepted 结果回流到控制面
- 拒绝把 draft / polish 中间态误当成 canon

## 主要产物

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

## 文件说明

- `SKILL.md`：bridge 行为规则
- `docs/schema/ainovel-feed.md`：feed 结构说明
- `references/workflow.md`：export / sync 流程
- `templates/*`：feed 模板
