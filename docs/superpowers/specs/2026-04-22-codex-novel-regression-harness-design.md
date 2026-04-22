# Codex `$novel` 自动验收与回归 Harness 设计

日期：2026-04-22

## 1. 背景

`skills/novel` 的本体是一个 **prompt-native / template-native** 的 skill 规范包。

它的核心价值不在独立 runtime，而在于：

- 驱动真实宿主 `Codex $novel`
- 约束项目目录、工作流、闭环规则与落盘格式
- 让 LLM 生成的大纲、正文、审查、状态文件持续符合合同

因此，工程化目标不是把 `novel` 改造成程序，而是为它补一层 **真实宿主级的自动验收与回归 Harness**。

## 2. 目标

本设计要解决以下问题：

1. 验证 `Codex $novel` 在真实调用链上是否仍可用。
2. 验证 skill 产物是否符合 `SKILL.md`、`templates/`、`docs/schema/` 中定义的合同。
3. 防止修改 `SKILL.md`、`templates/`、`references/` 后出现静默退化。
4. 让本地与 CI 都能重复运行同一批固定场景。

## 3. 非目标

以下内容不作为首版强约束：

- 不做全文逐字比对。
- 不把“文笔优美”“情绪到位”作为首版阻断条件。
- 不要求 skill 本体引入 Python 或额外 runtime。
- 不依赖人工临场判断才能给出通过/失败结论。

## 4. 设计原则

1. **真实宿主优先**：必须通过真实 `codex exec` 驱动 `$novel`，不做假的模拟器替代主验证链。
2. **黑盒优先**：把 `novel` 当作 skill 合同来验，不把宿主内部实现耦合进断言。
3. **不变量断言**：只校验结构、状态、流程、阻断规则与关键标记，不校验正文逐字内容。
4. **本地优先**：本地一条命令可跑；CI 只是复用同一套入口。
5. **skill 本体零侵入**：`skills/novel/` 保持 prompt-native；工程化只落在 `tests/` 与 `tools/`。
6. **失败可定位**：失败报告必须能明确指出是宿主调用失败、产物缺失、schema 不合规，还是流程规则破坏。

## 5. 方案比较

### 方案 A：纯产物校验器

只对已有输出目录做 schema 与文件检查。

优点：
- 实现最简单
- 稳定且速度快

缺点：
- 不验证真实 `Codex $novel` 调用链
- 无法捕捉宿主层静默回退

### 方案 B：真实宿主黑盒 E2E Harness（首版推荐）

用真实 `codex exec` 非交互执行固定用例，再对生成产物做确定性校验。

优点：
- 直接覆盖真实使用面
- 能同时验证宿主调用、skill 路由、产物合同

缺点：
- 速度慢于纯校验器
- 断言设计必须避免对 LLM 文本波动过敏

### 方案 C：B + 语义裁判层（二期）

在 B 的基础上增加语义质量裁判，例如 OOC、闭环遗漏、research 违规等。

优点：
- 覆盖最完整
- 可补足结构断言无法发现的质量性退化

缺点：
- 成本更高
- 首版复杂度偏大

### 结论

采用：

- **首版：方案 B**
- **二期：在 B 之上叠加 C**

## 6. 总体架构

```text
case yaml
→ workspace manager
→ fixture copy / seed data
→ codex adapter (`codex exec`)
→ artifact collection
→ deterministic validators
→ regression report
```

核心组件：

1. **Case Loader**：读取 `tests/cases/*.yaml`
2. **Workspace Manager**：为每个 case 准备隔离工作目录
3. **Codex Adapter**：真实调用 `codex exec` 运行 `$novel`
4. **Artifact Collector**：收集 stdout、最后消息、生成文件、目录快照
5. **Validators**：执行 schema、流程、阻断、索引同步等校验
6. **Reporter**：输出人类可读报告与机器可读 JSON 报告

## 7. 目录设计

```text
docs/
└── superpowers/
    └── specs/
        └── 2026-04-22-codex-novel-regression-harness-design.md

tests/
├── cases/
│   ├── smoke-init.yaml
│   ├── smoke-review.yaml
│   ├── workflow-mainline.yaml
│   ├── guardrail-block-write-without-outline.yaml
│   ├── guardrail-block-next-chapter-on-p0.yaml
│   └── research-note.yaml
├── fixtures/
│   ├── empty/
│   ├── initialized-project/
│   ├── outlined-project/
│   ├── project-with-p0-review/
│   └── project-with-missing-state/
├── validators/
│   ├── manifest.py
│   ├── markdown_contracts.py
│   ├── workflow_rules.py
│   └── artifact_rules.py
└── reports/
    └── .gitkeep

tools/
└── novel_harness/
    ├── run.py
    ├── case_loader.py
    ├── codex_adapter.py
    ├── workspace.py
    ├── reporter.py
    └── validators/
```

## 8. 运行时边界

### 8.1 skill 本体

`skills/novel/` 继续保持：

- 无 Python 运行前提
- 无额外 CLI runtime
- 无嵌入式测试逻辑

### 8.2 Harness

Harness 允许使用 **Python 3.11+** 作为测试与验证 runtime。

理由：
- skill 本体仍保持零 runtime
- YAML / JSON / Markdown 校验在 Python 中实现成本最低
- 本地与 CI 都容易运行

首版依赖约束：
- 优先标准库
- 允许最小外部依赖：`PyYAML`
- 不引入重量级测试框架作为首版前置条件

## 9. `codex exec` 调用策略

统一通过真实宿主执行：

```powershell
codex exec \
  --skip-git-repo-check \
  --ephemeral \
  -C <case-workspace> \
  -o <last-message-file> \
  "<case-prompt>"
```

约束：

1. 每个 case 在独立工作目录内运行。
2. 所有生成文件必须落在 case workspace，不污染仓库根目录。
3. `--ephemeral` 用于避免会话状态污染；持久记忆只看文件系统产物。
4. 所有 stdout/stderr/last message 必须保留到报告目录。

已验证事实：
- 本地存在 `codex exec` 非交互入口。
- 本地可通过该入口识别 `$novel` skill。

## 10. Case Schema

每个测试用例使用一个 YAML 文件描述，推荐结构如下：

```yaml
id: smoke_init
suite: smoke
purpose: verify init creates the contracted project skeleton
fixture: empty
workspace_mode: copy-fixture
steps:
  - name: init_project
    prompt: |
      $novel init 创建一本赛博修仙小说《霓虹飞升》，slug 为 neon-ascension。
    expect:
      files_exist:
        - projects/neon-ascension/project_manifest.yaml
      dirs_exist:
        - projects/neon-ascension/01_brief
        - projects/neon-ascension/02_bible
        - projects/neon-ascension/03_outline
        - projects/neon-ascension/04_manuscript
        - projects/neon-ascension/05_state
        - projects/neon-ascension/06_reports
        - projects/neon-ascension/07_exports
      validators:
        - manifest_contract
```

必填字段：

- `id`
- `suite`
- `purpose`
- `fixture`
- `steps`

每个 `step` 必填：

- `name`
- `prompt`
- `expect`

`expect` 支持的首版断言类型：

- `files_exist`
- `files_absent`
- `dirs_exist`
- `stdout_contains`
- `last_message_contains`
- `validators`
- `forbid_phase_advance`
- `forbid_new_chapter`

## 11. Validator 设计

### 11.1 结构类

1. **manifest_contract**
   - 校验 `project_manifest.yaml` 必填字段
   - 校验枚举值、日期格式、整数语义

2. **current_state_contract**
   - 校验 `05_state/current_state.md` 的快照表与块标记
   - 校验 `active_constraints` 区块存在性

3. **review_contract**
   - 校验 review 文件存在且包含 `P0 / P1 / P2` 分级结构

4. **summary_contract**
   - 校验章节摘要存在且命名正确

5. **diagnostic_contract**
   - 校验诊断报告存在且包含结论区块

### 11.2 流程类

1. **block_write_without_outline**
   - 无章纲时，不能直接产出正式章节

2. **block_next_chapter_on_p0**
   - 存在 `P0` 时，不得生成下一章正式稿

3. **state_sync_required**
   - 章节完成后必须补齐 `current_state.md` 与 `pending_hooks.md`

4. **research_note_before_body_mutation**
   - `research` 默认产出 research note，不直接偷偷改正文

5. **index_row_updated**
   - 章节定稿后，`chapter_index.md` 必须同步

### 11.3 宿主类

1. **codex_exit_ok**
   - `codex exec` 成功返回

2. **skill_invoked**
   - last message 或产物行为表明 `$novel` 路由已生效

3. **workspace_scoped**
   - 所有产物位于 case workspace 内

## 12. 回归策略

### 12.1 套件分层

1. **smoke**
   - 最小链路冒烟
   - 目标：快速发现 skill 失效或路径大面积断裂

2. **workflow**
   - 完整主链：`init → brief → bible → characters → outline → write → review → state`

3. **guardrail**
   - 专测阻断逻辑和合同边界

4. **research**
   - 专测联网考据落盘与保守写法

### 12.2 执行频率

- 本地开发：优先跑 `smoke`
- 修改 `SKILL.md` / `templates/` / `references/`：跑 `smoke + guardrail`
- 发布前：全量跑 `smoke + workflow + guardrail + research`
- CI：首版先接入 `smoke + guardrail`，二期再放全量

### 12.3 基线策略

不保存正文全文 golden snapshot。

只保存：
- 文件树快照
- 关键字段快照
- 结构化报告 JSON
- 失败时的 stdout / stderr / last-message

## 13. 首批用例矩阵

### smoke

1. `smoke_init`
2. `smoke_brief_after_init`
3. `smoke_outline_then_write`
4. `smoke_review_generates_report`
5. `smoke_state_sync`

### workflow

6. `workflow_mainline_ch1`
7. `workflow_resume_after_synced`

### guardrail

8. `guardrail_block_write_without_outline`
9. `guardrail_block_next_chapter_on_p0`
10. `guardrail_repair_missing_state_before_resume`

### research

11. `research_generates_note_not_body_mutation`
12. `research_conflict_yields_conservative_output`

## 14. 报告格式

Harness 每次运行产出两类报告：

### 14.1 人类可读 Markdown 报告

内容包含：
- 运行时间
- 使用的 suite / case
- 每一步 prompt 摘要
- 通过 / 失败情况
- 失败断言详情
- 关键产物路径

### 14.2 机器可读 JSON 报告

字段建议：

```json
{
  "run_id": "20260422-114500",
  "suite": "smoke",
  "cases_total": 5,
  "cases_passed": 4,
  "cases_failed": 1,
  "cases": []
}
```

## 15. 失败分类

失败必须落到以下类型之一：

- `host_failure`：`codex exec` 未成功执行
- `routing_failure`：未体现 `$novel` 生效
- `artifact_missing`：应生成文件缺失
- `schema_failure`：字段或格式不合规
- `workflow_violation`：阻断逻辑失效或流程越权
- `workspace_leak`：产物落到错误目录
- `assertion_error`：测试断言自身配置错误

## 16. CI 接入策略

首版保留本地优先，但设计必须兼容 CI。

建议：

1. 本地入口：

```powershell
python tools/novel_harness/run.py --suite smoke
```

2. CI 入口：

```powershell
python tools/novel_harness/run.py --suite smoke --report-dir tests/reports/ci
```

3. CI 只在以下路径变更时触发：
- `skills/novel/**`
- `tools/novel_harness/**`
- `tests/**`

## 17. 实施顺序

### Phase 1：最小可用 Harness

交付：
- `codex exec` 适配器
- `smoke_init`
- `smoke_review_generates_report`
- `manifest_contract`
- `review_contract`
- Markdown / JSON 报告

### Phase 2：流程与阻断规则

交付：
- `guardrail_block_write_without_outline`
- `guardrail_block_next_chapter_on_p0`
- `current_state_contract`
- `state_sync_required`

### Phase 3：主链与 research

交付：
- `workflow_mainline_ch1`
- `research_generates_note_not_body_mutation`
- `research_conflict_yields_conservative_output`

### Phase 4：CI 与二期语义裁判

交付：
- CI workflow
- 失败工件上传
- 可选语义裁判层

## 18. 风险与应对

### 风险 1：LLM 输出波动导致误报

应对：
- 只断言结构与不变量
- 不做全文快照比较

### 风险 2：真实宿主运行耗时较长

应对：
- suite 分层
- 本地默认只跑 smoke

### 风险 3：宿主版本升级导致行为变化

应对：
- 报告中记录 `codex --version`
- 区分 `host_failure` 与 `skill_failure`

### 风险 4：测试用例污染工作目录

应对：
- 每个 case 独立 workspace
- 统一在 `tests/reports/runs/<run-id>/` 下保留工件

## 19. 验收标准

本设计落地后，首版 Harness 视为可交付的标准：

1. 能通过真实 `codex exec` 调用 `$novel`
2. 至少覆盖 5 个 smoke 用例
3. 至少覆盖 2 个 guardrail 用例
4. 能输出 Markdown + JSON 双报告
5. 失败时能定位到具体 case、step、validator 与产物路径
6. skill 本体无需引入任何额外 runtime

## 20. 决策摘要

最终决策：

- **真实宿主**：`Codex $novel`
- **首版架构**：黑盒 E2E Harness + 确定性产物校验
- **实现语言**：Python 3.11+（仅用于 tests/tooling）
- **断言策略**：结构与不变量优先，不做正文全文 diff
- **执行策略**：本地优先，CI 兼容，分层运行
