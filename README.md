# Huzi Image2Prompt

`huzi-image2prompt` 是一个开源 Agent Skill，用于解析图片，生成可直接粘贴到 GPT Image、Grok Imagine 和 Midjourney 的中英文平台专用 Prompt，并将结果保存为 Markdown 文件。

## 使用场景

- 看到喜欢的照片、插画或海报，希望反推出可复现其主体、构图、光色与风格的提示词
- 需要将同一张参考图转换为 GPT Image、Grok Imagine 和 Midjourney 可直接使用的 Prompt
- 复刻人物自拍、服装、产品或复杂物体时，希望准确保留版型、材质、遮挡关系和裁切位置
- 分析含文字图片的视觉层级、排版和可辨认文字，为重新设计或生成相似画面提供描述
- 批量解析多张参考图，并将中英文 Prompt、图片尺寸和原图预览分别归档为 Markdown 文件
- 为提示词研究、模型对比或生成效果回归测试建立结构一致的图片描述基线

## 特性

- 覆盖写实照片、插画、海报和含文字图片
- 提取主体、构图、环境、风格、光线、色彩、材质微结构、遮挡关系、服装/物体结构、镜头观感与宽高比
- 先建立厂商无关的 Canonical Visual Spec，显式记录对象关系、前后层级和 `P0 identity / P1 similarity / P2 flexible` 复现优先级
- 按骨架与轴线、比例与锚点、负形与遮挡、三值明暗、空间尺度、材质纹理的顺序建立 Reconstruction Spec；结构未通过时触发纹理门禁
- 对生成结果采用结构优先评分：骨架和比例占 55%，材质纹理仅占 5%；任何身份关键 P0 冲突都会直接判定结构失败
- 按“平台 / 可选模型 / 可选版本 / 任务类型”选择 Prompt adapter；未指定模型或版本时使用保守平台档案，不从图片猜测生成参数
- 默认将 GPT、Grok、Midjourney 三个平台的中英文版本写入 Markdown 文件，共六个独立代码块；不再输出容易被误用于 Midjourney 的通用/自然语言 Prompt
- 每个代码块包含完整正向描述和反义提示，一次复制、一次粘贴即可出图
- 对织物颗粒、粗糙度、光泽、厚度、褶皱和拉伸进行显式描述，避免只写笼统的“真实材质”
- 人物自拍会锁定脸部可见性、头部角度、手机/头发遮挡关系、主体占比和裁切
- 服装或物体的领口、交叠、开口、腰线、附件等结构作为跨平台必须保持的事实
- GPT 使用完整关系句并把限制放在对应对象附近；Grok 对复杂主体列出不可合并的结构分区；Midjourney 先锁定完整构图和 P0 几何，再写高显著局部细节
- 模型偏差只作为有证据、带场景范围的 adapter 补偿，不会污染共享视觉事实或被误写成对厂商训练方式的推断
- Midjourney 版本自动附加 `--ar`、`--style raw` 和相关 `--no`，不猜测或固定模型版本
- 只在用户能够判断且会明显改变画面的关键歧义出现时先行确认
- 支持附件、本地图片路径和可访问的图片 URL
- 每次生成都保存为 `image-prompts/<图片名>-prompts-YYYYMMDD-HHmmss-fff.md`，控制台只返回文件路径
- 将原图复制到同级 `assets/` 并用相对路径嵌入 Markdown，打开结果即可预览原图
- 同一图片可重复生成；时间戳冲突时追加递增序号，Markdown 和图片副本都不覆盖历史文件
- 多张图片默认分别落盘，每张图片对应独立 Markdown 和原图副本
- 图片不可访问或文件写入失败时明确报错，不编造内容或伪造保存结果

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

成功解析图片后，默认把结果写入当前工作目录：

```text
image-prompts/
  <图片名>-prompts-20260901-150000-123.md
  assets/
    <图片名>-source-20260901-150000-123.png
```

每次生成都会创建毫秒级运行标识 `YYYYMMDD-HHmmss-fff`。Markdown 与本次复制的原图副本使用同一运行标识；若发生名称冲突，则追加 `-001`、`-002` 等递增序号。用户明确指定保存路径或文件名时，仍会在 `.md` 扩展名前插入运行标识，并在同级 `assets/` 目录保存图片副本。任何已有文件都不会被覆盖。写入成功后，聊天或控制台只返回实际 Markdown 路径，不再重复打印完整 Prompt。

Markdown 文件包含图片来源、尺寸、宽高比、通过 `![原图](<assets/...>)` 相对路径嵌入的原图预览，以及以下六个可独立复制的平台专用代码块：

1. GPT Prompt（中文）
2. GPT Prompt (English)
3. Grok Prompt（中文）
4. Grok Prompt (English)
5. Midjourney Prompt（中文）
6. Midjourney Prompt (English)

内部先建立厂商无关的 Canonical Visual Spec，记录画布、主体、几何分区、空间和遮挡关系、前后景、光色、材质、文字，以及 P0/P1/P2 复现优先级。随后以 Reconstruction Spec 锁定骨架、比例、负形、三值明暗和空间层级，最后才放行纹理；这些内部表示不会作为“原版”“通用”或“自然语言”Prompt 输出。最后由三个平台 adapter 分别编译：GPT 使用完整关系句，Grok 使用紧凑的结构分区指令，Midjourney 把完整取景和 P0 几何放在高显著局部细节之前，并在末尾附加：

```text
--ar [宽:高] --style raw --no [反义提示]
```

不会单独输出 Negative Prompt 或“不确定项”。生成前会建立骨架比例、负形遮挡、材质、结构、可见性、构图空间、明暗色彩七类高漂移画像，并要求所有平台版本保持一致；模型偏差只在 adapter 中基于用户对比或回归证据进行场景相关补偿。用户提供生成结果时，每轮只修正一到两个最高权重误差并锁定已正确 P0；除非用户明确要求生成，否则不会自动出图，自动纠偏最多三轮。若图片存在会显著改变结果、且用户能够判断的关键歧义，技能会先询问一个确认问题。多张图片默认分别生成 Markdown 文件。

## 项目结构

```text
huzi-image2prompt/       可安装、可打包的技能目录
  SKILL.md               技能定义与工作流
  LICENSE                随技能分发的 MIT 许可证
  references/
    reconstruction-method.md  结构测量、负形、明暗、空间、纹理门禁与评分规则
    model-adapters.md     Canonical Visual Spec、目标档案和三平台 adapter 规则
evals/evals.json         成功输出、不可访问路径、关键歧义、续答和重复生成行为用例
evals/cross-platform-fidelity-regression.json  跨平台材质、结构、遮挡和构图回归规范
evals/embedded-image-timestamp-regression.json  原图嵌入、时间戳和重复生成回归规范
evals/model-adapter-regression.json  模型感知中间表示、优先级和平台偏差补偿回归规范
evals/reconstruction-hierarchy-regression.json  骨架优先重建、纹理门禁与纠偏回归规范
evals/fixtures/          固定评估图片
docs/plans/              实施计划
dist/                    打包产物
```

## 局限

图片反推只能重建视觉特征，无法仅凭成图可靠恢复原始模型、LoRA、种子、采样器、步数、CFG 或后期处理参数。生成模型对精确文字的支持也不稳定，复杂排版通常需要后期处理。

## 隐私

技能本身不包含网络请求、遥测或密钥。实际图片是否发送到外部视觉服务，取决于宿主 Agent 使用的图片分析工具；处理敏感图片前应确认宿主工具的隐私策略。成功输出时会在 Markdown 同级 `assets/` 中额外保存一份原图副本，用户应按原图相同的敏感级别管理该目录。

## 许可证

MIT，详见 `LICENSE`。
