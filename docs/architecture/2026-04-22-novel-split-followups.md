# novel skill 三层拆分后续修改清单

日期：2026-04-22

## 当前已完成

- `novel` 恢复为通用全功能小说 skill
- `novel-framework` 新建为纯框架 skill
- `novel-ainovel-bridge` 保持为 AI-Novel 专属桥接层
- 根 README 与各 skill README 已更新
- `novel` 的 smoke / guardrail 回归已通过
- `novel-framework` 的 smoke init / block write 回归已通过
- `novel-framework` 的 export context pack 已完成本地验证
- `novel-ainovel-bridge` 的 export / missing-control-plane 回归已通过
- `novel-ainovel-bridge` 的 sync accepted payload 已完成本地验证
- bridge 的 `ainovel_feed` 文档已细化到字段级

## 后续建议修改

### P1：继续补 `novel-framework` 回归

当前已完成：

1. `framework smoke init`
2. `framework` 遇到“写正文”请求时明确拒绝并提示用 `novel`

仍建议补：

3. `framework outline only`

### P1：bridge 的 export / sync 细化

当前已完成：

1. `bridge export` 成功导出 feed
2. `bridge export` 在控制面缺口时正确拒绝
3. `ainovel_feed` 文档已细化为字段级规范

当前已明确 `sync` 时建议回流：

- summary
- pending hooks
- current state
- timeline
- relationship changes

仍建议补：

4. 把 `bridge sync` 的本地验证进一步固化成稳定自动回归
5. 增加多章节 / 多次 sync 的幂等性验证

### P1：统一三层之间的 schema

重点统一：

- chapter context
- current state
- pending hooks
- outline entry
- character card

避免 `novel` / `novel-framework` / `bridge` 三边未来漂移。

### P2：README 增加组合用法示例

建议补一个最小示例：

1. `$novel-framework init`
2. `$novel-framework brief`
3. `$novel-framework bible`
4. `$novel-framework outline`
5. `$novel-ainovel-bridge export`
6. AI-Novel 跑正文
7. `$novel-ainovel-bridge sync`

再补一个全功能示例：

1. `$novel init`
2. `$novel brief`
3. `$novel outline`
4. `$novel write`
5. `$novel review`

### P2：桥接层测试 fixture

当前已完成：

- 一个完整 `control-plane-ready` fixture
- 一个 `control-plane-missing-state` fixture

仍建议补：

- 一个完整 `novel-framework` 专属 fixture
- 一个 AI-Novel accepted fixture
- 一个 AI-Novel draft-only fixture

### P3：命名再评估

当前名称：

- `novel`
- `novel-framework`
- `novel-ainovel-bridge`

可选再评估：

- `novel-framework` 是否要叫 `novel-planner`
- `novel-ainovel-bridge` 是否要缩短成 `ainovel-bridge`

## 建议顺序

推荐后续按这个顺序做：

1. 补 `novel-framework` 的 outline only 测试
2. 把 `bridge sync` 的本地验证固化成更稳定自动回归
3. 统一三层 schema
4. 再考虑命名微调

## 判断是否需要继续改

如果当前目标只是：

- `novel` 保持全功能
- 同时提供一个纯框架入口
- 把 AI-Novel 耦合固定在 bridge

那现在已经够用。

如果目标是：

- 长期稳定联动 `novel-framework` + AI-Novel

那优先做：

- framework 额外测试
- bridge sync 幂等性与多章节测试
- 字段级 schema 统一
