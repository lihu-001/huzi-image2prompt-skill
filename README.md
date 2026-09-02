# Huzi Image2Prompt

`huzi-image2prompt` 是一个开源 Agent Skill。它读取参考图，提取可见的主体、结构、构图、光色、材质和文字信息，再生成可直接用于 GPT Image、Grok Imagine、Midjourney 的中英文平台专用 Prompt，并将结果保存为 Markdown 文件。

## 快速开始

### 运行条件

- 支持 `SKILL.md` 的 Agent 运行时
- 能读取用户附件、本地图片或图片 URL 的视觉能力
- 能在当前工作目录创建 Markdown 文件并复制原图

技能本身没有第三方运行时依赖，也不包含网络请求、遥测或密钥。

### 安装

任选一种方式：

1. 将 [`huzi-image2prompt/`](huzi-image2prompt/) 复制到宿主的技能目录。
2. 在支持 `.skill` 包导入的运行时中安装 [`dist/huzi-image2prompt.skill`](dist/huzi-image2prompt.skill)。

常见目录示例：

```text
Codex:       $CODEX_HOME/skills/huzi-image2prompt/
Claude Code: ~/.claude/skills/huzi-image2prompt/
```

### 触发

上传图片或提供可访问路径，然后发送类似指令：

- `解析图片`
- `获取这个图片的 Prompt`
- `反推这张图的提示词`
- `只要 Grok 中文 Prompt`

若图片不存在、损坏、无权限或无法解码，技能会说明阻塞原因，不会编造图片内容或创建占位文件。

## 默认输出

未指定保存位置时，每张图片生成一份独立 Markdown，并在同级 `assets/` 中保存原图副本：

```text
.huzi-image-prompts/
  <图片名>-prompts-<YYYYMMDD-HHmmss-fff>.md
  assets/
    <图片名>-source-<YYYYMMDD-HHmmss-fff>.<扩展名>
```

每份 Markdown 包含：

- 来源元数据（原图名称、精确原始尺寸、规范化生成比例）
- 原图预览（相对路径引用 `assets/` 副本，移动整个目录仍可预览）
- “图片解析结果”章节（画面事实与必须保持要点）
- 三个平台的中英文共六段可独立复制的 Prompt，每段自带正向描述和反义约束，无需另拼 Negative Prompt

| 平台 | 语言 |
|---|---|
| GPT Image | 中文、English |
| Grok Imagine | 中文、English |
| Midjourney | 中文、English |

用户指定目录或文件名时优先使用；明确限制平台或语言时，只生成所需范围。写入成功后，控制台只返回实际 Markdown 路径，不重复打印 Prompt。

## 项目结构

```text
huzi-image2prompt/
  SKILL.md                       技能契约和完整工作流
  references/
    reconstruction-method.md     结构测量、门禁、评分和迭代规则
    model-adapters.md            视觉事实规范与三平台 adapter
evals/                           行为回归用例与结果
demo/assets/                     公开演示图片
dist/huzi-image2prompt.skill     可安装包
```

行为细节（重建顺序、色彩保真、比例规范化、平台差异等）以 `SKILL.md` 和 `references/` 为准，此处不再展开。

## 局限

- 只能重建成图中可见的视觉特征，无法恢复原始模型、种子、采样器等生成参数。
- 复现质量受宿主视觉分析能力和目标生成模型能力影响；精确输出像素由平台自身控制。

## 隐私

成功输出时会在 `assets/` 中额外保存一份原图副本，请按原图相同的敏感级别管理该目录。

## 许可证

MIT，详见 [`LICENSE`](LICENSE)。
