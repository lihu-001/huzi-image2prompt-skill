# Huzi Image2Prompt

`huzi-image2prompt` 是一个开源 Agent Skill，用于解析图片并将可见内容整理为可复用的图像生成 Prompt。

## 特性

- 覆盖写实照片、插画、海报和含文字图片
- 提取主体、构图、环境、风格、光线、色彩、材质、镜头与宽高比
- 区分可见事实、合理推断和不可恢复参数
- 支持附件、本地图片路径和可访问的图片 URL
- 可按 Midjourney、Stable Diffusion、Flux 等目标模型调整语法
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
- “把这张海报转换成 Flux Prompt”
- “image to prompt”

## 默认输出

````markdown
## Prompt
```text
[可直接复制的完整 Prompt]
```

## Negative Prompt（可选）
```text
[与复现目标直接相关的排除项]
```

## 不确定项（可选）
- [无法从成图可靠恢复的信息]
````

## 项目结构

```text
huzi-image2prompt/       可安装、可打包的技能目录
  SKILL.md               技能定义与工作流
  LICENSE                随技能分发的 MIT 许可证
evals/evals.json         两个真实图片附件场景和一个不可访问路径场景
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
