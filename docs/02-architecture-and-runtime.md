# 架构与运行机制

## 总体形态

Boogu-Image 是一个 Diffusers 风格的本地推理工程，主体由三部分组成：

1. 推理入口：`inference.py` 和简化脚本负责解析参数、加载 pipeline、处理输入输出。
2. Pipeline：`boogu/pipelines/boogu/` 里实现文生图、图生图、prompt 重写、guidance、offload、输出拼图等流程。
3. 模型组件：transformer、VAE、MLLM instruction encoder、scheduler、cache/加速模块按 Hugging Face/Diffusers 方式加载。

它没有内置 REST API、Web UI、用户管理、任务队列或内容审核。做产品时需要另包一层服务。

## `inference.py` 的运行流程

`INFERENCE_GUIDE.md` 把主流程拆成：

1. 解析 CLI 参数。
2. 从 `--pretrained_pipeline_name_or_path` 加载 `BooguImagePipeline` 或 prompt tuning pipeline。
3. 可选替换 instruction encoder。
4. 可选加载 prompt tuning embedding 和 LoRA。
5. 可选挂载本地 instruction rewriter。
6. 可选替换 diffusion transformer。
7. 可选加载 transformer LoRA。
8. 处理 TeaCache / TaylorSeer / Cache-DiT 参数。
9. 校验 device/offload 兼容性。
10. 启用选定 offload。
11. 准备单样本或 batch 输入。
12. 执行 pipeline，保存输出图或拼图。

这说明上游已经把大多数研究推理能力塞进一个入口，方便试验，但产品化时也意味着参数面很宽，必须收敛成少量稳定 preset。

## 三类典型任务

| 任务 | 模型 | 输入 | 入口 |
| --- | --- | --- | --- |
| 文生图 T2I | Base / Turbo | 文本 prompt | `inference.py` 或 `inference_simple.py` |
| 图像编辑 TI2I | Edit | 文本 prompt + 输入图 | `inference.py` 或 `inference_ti2i_simple.py` |
| 批量生成/编辑 | Base / Edit | YAML 数据 | `--use_batch_inference True` |

批量 YAML 结构大致是：

```yaml
data:
  instructions:
    - "Prompt for sample 1"
    - "Prompt for sample 2"
  input_image_paths:
    - []
    - ["./examples/ref.png"]
```

## Prompt 重写机制

Boogu 的一个重点是 instruction reasoner / rewriter。它可以把用户的普通请求重写成更具体的视觉生成 prompt。

两种模式：

| 模式 | 说明 | 风险/成本 |
| --- | --- | --- |
| 本地 rewriter | 使用本地 VLM/LLM 或复用 instruction encoder | 吃显存，和 offload 组合有兼容限制 |
| DashScope 远程重写 | 调用 `qwen-vl-max-latest` 等远程模型 | 需要 API key，请求会出网，涉及隐私和成本 |

可选 prompt preset：

| preset | 用途 |
| --- | --- |
| `default` | 通用文生图/图生图 prompt polishing |
| `ppt` | 幻灯片、报告、信息图、设计布局 |
| `custom` | 自定义多步 prompt rewrite |

产品化建议：不要直接把 rewriter 暴露为无限自由文本能力。应预设少量“商品图、海报、证件风、详情页”等模板，把用户输入映射到稳定 prompt 结构。

## 显存与 offload

上游提供三种 offload 标志，互斥：

| 策略 | 大致含义 | 适用 |
| --- | --- | --- |
| `enable_sequential_cpu_offload_flag` | 最省显存，最慢 | 12GB/16GB 试跑 |
| `enable_model_cpu_offload_flag` | pipeline 级 CPU offload | 24GB/32GB 比较实用 |
| `enable_group_offload_flag` | transformer/MLLM/VAE 分组 offload | 低显存 + fp8 场景 |

代码层面会校验：

- 三个 offload 至多启用一个。
- 启用任意 offload 时 `device` 不能是 CPU。
- 共享 MLLM/rewriter 的本地 prompt rewriting 与 offload 不兼容，除非使用自定义 rewriter 或远程 rewriting。

## FP8、Torch Compile 和缓存加速

| 能力 | 用途 | 注意事项 |
| --- | --- | --- |
| FP8 weights | 降低 transformer 权重显存 | 需要预量化 checkpoint |
| Torch Compile | 降低部分重复 block 延迟 | 上游警告某些 GPU/模型可能黑图 |
| TeaCache | transformer cache 加速 | 先在基线稳定后再开 |
| TaylorSeer | 另一类缓存加速 | 与 TeaCache 互斥，优先级高于 TeaCache |
| Cache-DiT | 高优先级 cache 策略 | 开启后覆盖 TaylorSeer/TeaCache |

上游的缓存优先级：

```text
Cache-DiT > TaylorSeer > TeaCache
```

产品化建议：不要一开始就混开 compile、fp8、cache 和 offload。先保存一套能稳定出图的 baseline，再逐一打开优化项。

## 输出逻辑

`inference.py` 会创建输出目录，保存单图或多图。当 `num_images_per_instruction > 1` 时，会保存编号图片，并生成横向 collage 到 `--output_image_path`。

产品化建议：

- 输出目录要和用户、任务 ID、模型版本、seed、prompt hash 绑定。
- 保存原始 prompt、重写 prompt、模型版本、参数、输入图 hash，用于复现和质量追责。
- 对外只返回审核通过的图，不直接暴露内部 outputs 目录。

