# novel-framework

长篇小说**纯框架** skill。

## 定位

这是 `novel` 的受限门面版：

- 只生成背景、世界观、人物、性格、势力、钩子、剧情骨架、卷纲章纲、状态与伏笔
- 不生成任何正文 prose

适合这样的场景：

- “只做小说整体框架”
- “不要正文，只要设定和大纲”
- “只做人物、钩子、故事情节骨架”
- “只导出 chapter context / framework bundle 给后续流程”

## 与 `novel` 的区别

- `novel`：框架 + 内容双栈
- `novel-framework`：只做框架

## 与 `novel-ainovel-bridge` 的区别

- `novel-framework`：做框架
- `novel-ainovel-bridge`：把框架喂给 AI-Novel，并回流 accepted 结果

## 安装

```powershell
npx skills add shibo1998/shibo-skills --skill novel-framework
```

或：

```powershell
npx skills add https://github.com/shibo1998/shibo-skills --skill novel-framework
```

## 导出产物

默认可导出：

- `07_exports/context_packs/chNNN_context.yaml`
- `07_exports/framework_bundle.md`

它们都属于控制面交付物，不包含正文。

## 内容

为了尽量复用 `novel` 已有形式，`novel-framework` 继续沿用同类模板、项目结构和控制面 schema，只是在行为上禁用正文生成。
