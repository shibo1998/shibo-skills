# novel

长篇小说项目型 skill。

## 定位

这不是“一次性写一章”的 prompt，而是一个**文件化、可续写、可审查、可维护**的小说工作台。

它采用：

- 一书一目录
- 先立项再写
- 强闭环维护
- 按需联网考据
- 模板驱动落盘

## 运行方式

本 skill 现在是 **prompt-native / template-native**：

- **不依赖 Python**
- **不依赖额外安装脚本**
- **不要求单独 CLI runtime**

使用时，直接通过 skill 自身说明、`templates/` 模板、`references/` 参考文档，以及宿主 CLI 的文件系统能力完成：

- 初始化项目目录
- 生成 brief / bible / characters / outline
- 写草稿与正式稿
- 生成摘要 / 审查 / 诊断
- 更新状态卡 / 伏笔池 / 时间线 / 资源账本

## 安装

```powershell
$env:SKILL_BASE_URL='https://raw.githubusercontent.com/Z-Shi-Bo/novel-skill/main/'
npx skill skills/novel
```

## 关键目录

```text
skills/novel/
├── SKILL.md
├── references/
├── templates/
├── docs/
└── evals/
```

## 项目结构

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

## 推荐入口

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

## 说明

如果你在旧版本说明里看到 `python ...`、`.py`、`build_context.py` 之类内容，那是历史实现痕迹。  
当前版本以 `SKILL.md` 为准，已不再把 Python 作为运行前提。
