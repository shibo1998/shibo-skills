# novel-skill

这是 `novel` skill 的仓库包装层。

真正的 skill 内容在：

```text
skills/novel/
```

## 安装

### 用 `npx skill`

```powershell
$env:SKILL_BASE_URL='https://raw.githubusercontent.com/Z-Shi-Bo/novel-skill/main/'
npx skill skills/novel
```

这会把 `skills/novel/` 作为远程 skill 包下载。  
这里用 raw GitHub 地址，是为了避开 GitHub tree API 的匿名 rate limit。

### 本地 CLI 使用

本仓库根目录保留了一个轻量 `SKILL.md` 包装层，方便 Claude / Codex / Qoder / Gemini 在本地继续把这个仓库识别为 `novel` skill。

## 开发

- 仓库根目录：分发包装层
- `skills/novel/`：canonical skill source
- `.omc/`、`projects/`：本地运行态，不进 git
