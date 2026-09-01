# Huzi Image2Prompt

`huzi-image2prompt` 是一个开源 Agent Skill，用于解析图片，并输出可直接粘贴到通用模型、GPT Image、Grok Imagine 和 Midjourney 的中英文图片生成 Prompt。

## 特性

- 覆盖写实照片、插画、海报和含文字图片
- 提取主体、构图、环境、风格、光线、色彩、材质、镜头观感与宽高比
- 默认输出自然语言、GPT、Grok、Midjourney 四类中英文版本，共八个独立代码块
- 每个代码块包含完整正向描述和反义提示，一次复制、一次粘贴即可出图
- Midjourney 版本自动附加 `--ar`、`--style raw` 和 `--no`
- 只在用户能够判断且会明显改变画面的关键歧义出现时先行确认
- 支持附件、本地图片路径和可访问的图片 URL
- 图片不可访问时停止推断，不编造内容

## 安装

将 `huzi-image2prompt/` 目录复制到 Agent 的技能目录。例如 Claude Code：

```text
~/.claude/skills/huzi-image2prompt/
```

也可以在支持 `.skill` 包导入的运行时中安装 `dist/huzi-image2prompt.skill`。

## 触发示例

- “解析图片”
- “获取这个图片的 Prompt”
- “反推这张图的提示词”
- “输出 GPT、Grok、Midjourney 都能直接使用的 Prompt”
- “image to prompt”

## 默认输出

成功解析图片后，默认按以下顺序输出八个可独立复制的代码块：

1. 自然语言 Prompt（中文）
2. Natural-language Prompt (English)
3. GPT Prompt（中文）
4. GPT Prompt (English)
5. Grok Prompt（中文）
6. Grok Prompt (English)
7. Midjourney Prompt（中文）
8. Midjourney Prompt (English)

GPT 和 Grok 使用完整自然语言生成指令，并把“不要出现……”直接写在同一代码块中。Midjourney 使用描述性 Prompt，并在末尾附加：

```text
--ar [宽:高] --style raw --no [反义提示]
```

不会单独输出 Negative Prompt 或“不确定项”。若图片存在会显著改变结果、且用户能够判断的关键歧义，技能会先询问一个确认问题。

## 项目结构

```text
huzi-image2prompt/       可安装、可打包的技能目录
  SKILL.md               技能定义与工作流
  LICENSE                随技能分发的 MIT 许可证
evals/evals.json         两个真实图片输出场景、一个不可访问路径场景和一个关键歧义场景
evals/fixtures/          固定评估图片
docs/plans/              实施计划
dist/                    打包产物
```

## 局限

图片反推只能重建视觉特征，无法仅凭成图可靠恢复原始模型、LoRA、种子、采样器、步数、CFG 或后期处理参数。生成模型对精确文字的支持也不稳定，复杂排版通常需要后期处理。

## 隐私

技能本身不包含网络请求、遥测或密钥。实际图片是否发送到外部视觉服务，取决于宿主 Agent 使用的图片分析工具；处理敏感图片前应确认宿主工具的隐私策略。

## 许可证

MIT，详见 `LICENSE`。
