# Huzi Image2Prompt

`huzi-image2prompt` 是一个开源 Agent Skill。它读取参考图，提取可见的主体、结构、构图、光色、材质和文字信息，再生成可直接用于 GPT Image、Grok Imagine、Midjourney 的中英文平台专用 Prompt。

项目使用同一套通用规则处理可读取的照片、插画、海报和含文字图片，不依赖某张特定图片，也不会把单个案例的主体、场景或调色板固化为全局规则。

## 快速开始

### 运行条件

- 支持 `SKILL.md` 的 Agent 运行时
- 能读取用户附件、本地图片或图片 URL 的视觉能力
- 能在当前工作目录创建 Markdown 文件并复制原图

技能本身没有第三方运行时依赖，也不包含网络请求、遥测或密钥。实际图片是否会发送给外部服务，取决于宿主 Agent 使用的视觉工具。

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
- `输出 GPT、Grok、Midjourney 都能直接使用的 Prompt`
- `只要 Grok 中文 Prompt`

若图片不存在、损坏、无权限或无法解码，技能会说明阻塞原因，不会编造图片内容或创建占位文件。

## 默认输出

未指定保存位置时，每张图片会生成一份独立 Markdown，并在同级 `assets/` 中保存本次读取的原图副本：

```text
.huzi-image-prompts/
  <图片名>-prompts-<YYYYMMDD-HHmmss-fff>.md
  assets/
    <图片名>-source-<YYYYMMDD-HHmmss-fff>.<扩展名>
```

Markdown 和图片副本共用同一个运行标识。若文件名冲突，会追加 `-001`、`-002` 等序号；已有文件和原图不会被覆盖或修改。用户指定目录或文件名时优先使用该位置，缺少 `.md` 扩展名会自动补齐，运行标识仍会插入扩展名前。Markdown 使用相对路径嵌入副本，因此移动整个输出目录后仍可预览原图。

每份 Markdown 包含图片来源、精确原始尺寸、规范化生成比例和原图预览，并默认生成六个可独立复制的平台 Prompt：

| 顺序 | 平台 | 语言 |
|---:|---|---|
| 1 | GPT Image | 中文 |
| 2 | GPT Image | English |
| 3 | Grok Imagine | 中文 |
| 4 | Grok Imagine | English |
| 5 | Midjourney | 中文 |
| 6 | Midjourney | English |

每个代码块都包含完整正向描述和与当前画面直接相关的反义约束，不需要再拼接单独的 Negative Prompt。默认不输出“通用 Prompt”“原版 Prompt”“自然语言 Prompt”或“不确定项”章节；用户明确限制平台或语言时，只生成所需范围。

写入成功后，聊天或控制台只返回实际 Markdown 路径，不重复打印六段 Prompt。用户明确要求“同时显示”时才会额外展示。

## 工作原理

```text
参考图
  → Canonical Visual Spec：统一记录可见画面事实
  → Reconstruction Spec：按结构优先顺序组织复现约束
  → Platform Adapters：分别编译 GPT / Grok / Midjourney
  → Markdown + assets/ 原图副本
```

### 1. 单一视觉事实源

Canonical Visual Spec 记录画布、主体、几何分区、对象关系、前后景、遮挡、光色、材质、文字和直接冲突项，并将事实分成：

- `P0 identity`：改变后会破坏主体身份或画面骨架
- `P1 similarity`：明显影响相似度
- `P2 flexible`：允许合理变化

平台 adapter 可以改变语法、顺序、冗余和有证据的偏差补偿，但不能改变这些共享事实。项目没有面向特定图片的固定 Prompt 或专属纠偏规则。

### 2. 结构优先重建

重建顺序固定为：

1. 骨架与轴线
2. 比例与关键锚点
3. 负形与遮挡
4. 三值明暗体积
5. 空间与尺度层级
6. 色彩标定、材质与纹理

外轮廓与骨架、比例与关键锚点合计占结构诊断权重的 55%。任何身份关键 P0 冲突都会触发硬门禁，不能靠“高细节”“真实材质”或装饰纹理抵消。

### 3. 色彩与材质保真

- 按主要区域分别记录色相、相对明度、相对饱和度和色度排序。
- 高饱和区域必须保持鲜艳；抑制全局偏色时不能把正确的局部颜色一起压灰。
- 低饱和或中性区域不能因为局部增艳而被统一染色。
- 只描述图片中确实可见的颗粒、织纹、粗糙度、光泽、厚度、褶皱和磨损。
- 平面图形或无纹理图片不会被自动添加纸张颗粒、织物纤维或照片级微结构。

用户提供生成结果进行对比时，每轮只修正一到两个最高权重误差，并锁定已经正确的 P0。只有用户明确要求生成或连续迭代时才调用生图能力；自动纠偏最多三轮。

## 三个平台如何分工

GPT 与 Grok 共享同一份 Canonical Visual Spec，但不直接混用同一段成品 Prompt。严格参考图复现时，应保留各自 adapter：

| 平台 | 编译方式 | 画布表达 | 反义约束 |
|---|---|---|---|
| GPT Image | 完整自然语言和明确关系句；限制靠近对应对象 | 开头写方向、生成比例和可靠的原始尺寸参考 | 使用内嵌自然语言限制 |
| Grok Imagine | 更紧凑、直接；复杂主体列出不可合并的结构分区 | 开头写紧凑画布子句和原始尺寸参考 | 内嵌在同一代码块 |
| Midjourney | 先写取景、完整轮廓、主体占比和 P0 几何，再写光色与材质 | 末尾使用保真且紧凑的 `--ar` | 使用相关 `--no` |

Midjourney 默认附加：

```text
--ar [生成比例] --style raw --no [与原图直接相关的反义提示]
```

不会猜测或固定 `--v`、`--stylize`、种子、采样器或其他无法从成图恢复的参数。

## 原始尺寸与生成比例

项目明确区分两个概念：

| 字段 | 示例 | 用途 |
|---|---|---|
| 原始尺寸 | `405 × 793` | 精确元数据；作为 GPT/Grok 的尺寸参考 |
| 生成比例 | `24:47` | 表达原图画布形状；供三平台 Prompt 使用 |

比例规范化规则：

1. 只有当原图与常用比例的相对误差不超过 `0.25%` 时，才采用 `1:1`、`4:5`、`16:9`、`21:9` 等常用表达。
2. 未命中常用比例时，若精确约分后的两项都不大于 `99`，直接使用精确比例。
3. 若精确比例含三位数，优先寻找两项均不大于 `99`、相对误差不超过 `0.25%` 的紧凑近似。
4. 两位数近似无法满足精度时，回退到精确约分结果。画布保真优先于数字简短。

| 原始尺寸 | 生成比例 | 说明 |
|---|---|---|
| `1920 × 1080` | `16:9` | 精确常用比例 |
| `1080 × 1920` | `9:16` | 精确常用比例 |
| `2560 × 1080` | `64:27` | 精确比例；行业中常笼统称为 `21:9` |
| `405 × 793` | `24:47` | 紧凑近似，误差约 `0.016%` |
| `406 × 900` | `9:20` | 命中常用竖屏比例 |

Prompt 中的原始尺寸是构图参考，不代表仅凭文字就能强制平台输出精确像素。实际输出尺寸仍由对应平台的界面或 API 控制。无法可靠取得尺寸时，技能不会虚构像素；比例也无法判断时会省略生成比例和 Midjourney `--ar`。

## 关键行为边界

- 只记录图片可见事实和有助于复现的保守推断，不声称恢复创作者意图或隐藏生成参数。
- 可辨认文字逐字保留；中英文 Prompt 都不翻译图片中的原文。不可辨认内容不会被补写。
- 只有两个以上解释都受图片支持、用户能够判断且答案会明显改变结果时，才先询问一个关键问题。
- 多张图片默认分别生成 Markdown 和原图副本，不混合视觉事实。
- 不会自动加入“电影感”“杰作”“8K”“获奖摄影”等原图没有的风格增强词。
- 不会仅因某个平台一次失败，就把该案例升级成所有图片的永久规则。

## 项目结构

```text
huzi-image2prompt/
  SKILL.md                       当前技能契约和完整工作流
  LICENSE
  references/
    reconstruction-method.md     结构测量、门禁、评分和迭代规则
    model-adapters.md            Canonical Visual Spec、比例和三平台 adapter
evals/
  evals.json                     主行为用例
  fixtures/                      公共固定评估图片
  *-baseline.json                修改前基线，允许记录 FAIL
  *-regression.json              通用回归契约
  *-results.json                 当前验证结果
docs/                            历史设计与实施记录
dist/
  huzi-image2prompt.skill        可安装包
```

当前行为以以下文件为准，优先级从高到低：

1. [`huzi-image2prompt/SKILL.md`](huzi-image2prompt/SKILL.md)
2. [`reconstruction-method.md`](huzi-image2prompt/references/reconstruction-method.md) 与 [`model-adapters.md`](huzi-image2prompt/references/model-adapters.md)
3. [`evals/`](evals/) 中的 regression 和 results

`docs/` 保留项目演进过程中的设计与实施记录，部分文档可能描述旧阶段行为，不作为当前运行契约。

## 验证与维护

仓库中的回归覆盖六代码块输出、模型 adapter、结构层级、色彩饱和度、画布比例、原图嵌入、时间戳、重复生成和失败路径。

校验所有评测 JSON：

```powershell
Get-ChildItem evals -Filter *.json |
  ForEach-Object { Get-Content -Raw -Encoding UTF8 $_.FullName | ConvertFrom-Json | Out-Null }
```

查看 `.skill` 包内容：

```text
python -m zipfile -l dist/huzi-image2prompt.skill
```

修改工作流时，应同步更新 `SKILL.md`、直接相关的 reference、回归契约和结果文件，然后重新构建 `dist/huzi-image2prompt.skill`。不要把某张测试图的主体、服装、场景或调色板写成全局规则。

## 局限

- 只能重建成图中可见的视觉特征，无法可靠恢复原始模型、LoRA、种子、采样器、步数、CFG 或后期参数。
- GPT/Grok Prompt 中的原始尺寸只是参考；精确输出像素取决于平台提供的尺寸控制。
- 生成模型对精确文字和复杂排版的支持仍不稳定，必要时需要后期处理。
- 复现质量受宿主视觉分析能力和目标生成模型能力影响。

## 隐私

成功输出时，技能会在 Markdown 同级 `assets/` 中额外保存一份原图副本。请按原图相同的敏感级别管理该目录。处理敏感图片前，应确认宿主 Agent 及其视觉工具的隐私策略。

## 许可证

MIT，详见 [`LICENSE`](LICENSE)。
