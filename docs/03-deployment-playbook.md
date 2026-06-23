# 本地部署与试跑路线

## 推荐环境

上游 README 写明测试环境：

```text
Python 3.10
CUDA 12.6
PyTorch 2.7.1
```

`pyproject.toml` 支持 Python `>=3.10,<3.13`，PyTorch `>=2.7.1,<2.12`。Flash Attention 和 Triton 路线主要面向 Linux x86_64。Windows 机器建议优先用 WSL2 + NVIDIA CUDA，或者直接用 Linux GPU 服务器。

## 最小试跑路径

1. 克隆上游仓库。

```bash
git clone https://github.com/boogu-project/Boogu-Image.git
cd Boogu-Image
```

2. 建 Python 环境。

```bash
conda create -y -n boogu python=3.10
conda activate boogu
pip install -r requirements/torch2.7-cu126.txt
pip install -e .
python utils/get_flash_attn.py
```

也可以直接运行：

```bash
bash quick_start.sh
conda activate boogu
```

3. 下载模型。

```bash
pip install -U "huggingface_hub[cli]"
huggingface-cli download Boogu/Boogu-Image-0.1-Turbo --local-dir models/Boogu-Image-0.1-Turbo
```

4. 先跑 Turbo 1K。

```bash
export CUDA_VISIBLE_DEVICES=0
export device="cuda:0"

python inference.py \
  --pretrained_pipeline_name_or_path models/Boogu-Image-0.1-Turbo \
  --instruction "一张干净的电商商品主图，白色背景，一台黑色机械键盘，柔和棚拍光线，真实产品摄影质感。" \
  --num_inference_steps 4 \
  --height 1024 --width 1024 \
  --text_guidance_scale 1.0 \
  --output_image_path outputs/turbo_keyboard.png \
  --device "$device" \
  --rewriter_device "$device"
```

注意：Turbo pipeline 的简化脚本使用 `BooguImageTurboPipeline` 和 `use_dmd_student_inference=True`。如果主 `inference.py` 对 Turbo 的参数不稳定，应改用上游 `inference_turbo_simple.py` 或对应 demo。

## 模型选择

| 目标 | 首选 | 原因 |
| --- | --- | --- |
| 快速看效果 | Turbo / Turbo-fp8 | 少步数，反馈快 |
| 高质量文生图 | Base | 上游定位为基础模型，适合复杂文本和下游开发 |
| 商品图/海报文字 | Base 2K | 上游建议密集文字渲染优先 Base 2K |
| 修图/改色/删除对象 | Edit | 面向 TI2I 图像编辑 |
| 显存紧张 | fp8 + offload | 牺牲部分速度/稳定性换可运行 |

## 显存路线

上游 README 的显存建议可以概括为：

| 显存 | 1K 建议 | 2K 建议 |
| --- | --- | --- |
| 12GB | sequential offload；量化可 model offload + fp8 | sequential offload；量化可 group offload + fp8 |
| 16GB | sequential offload；量化可 model offload + fp8 | sequential/model offload |
| 24GB | model offload；量化可直接 fp8 | model offload |
| 32GB | model offload；量化可直接 fp8 | model offload 或 fp8 |
| 40GB | 可跑基础模型 | model offload 或 fp8 |
| 80GB | 更适合完整 2K / 批量 / rewriter | 更适合完整 2K / 批量 / rewriter |

落地建议：

1. 12GB/16GB：只做试验，不要承诺交互式产品体验。
2. 24GB：可以做低并发内部工具。
3. 40GB/48GB：更适合认真评测 Base/Edit。
4. 80GB：适合做批处理、双模型 rewriter、2K 输出和多队列服务。

## Windows 机器注意事项

如果在 Windows 上直接跑，可能会遇到：

- `triton` / `flash-attn` 兼容问题。
- shell 脚本需要 Bash 环境。
- CUDA、PyTorch wheel、编译工具链组合不一致。

更稳的方式：

1. Windows 只做项目管理和文档。
2. WSL2 Ubuntu 或 Linux GPU 服务器跑模型。
3. 模型服务通过 HTTP 暴露给 Windows 前端或业务后台。

## 封装成本

上游没有内置 API 服务。产品化最少需要补：

| 组件 | 必要性 |
| --- | --- |
| FastAPI/Flask 服务层 | 必要 |
| 任务队列 | 必要，防止 GPU 被同步请求拖死 |
| 模型常驻加载 | 必要，避免每次请求重新加载 10B 模型 |
| 参数 preset | 必要，把复杂 CLI 收敛为少量可控模式 |
| 内容审核 | 必要 |
| 用户上传图安全处理 | 必要 |
| 输出图 CDN/对象存储 | 生产需要 |
| 成本与时延统计 | 生产需要 |
| 管理后台 | 团队使用需要 |

## 推荐试跑清单

1. Turbo 1K 文生图，记录单图耗时、显存峰值、失败率。
2. Base 1K 文生图，比较质量和耗时。
3. Base 2K 中文文字海报，重点看错字、漏字、布局漂移。
4. Edit 图像删除/改色/换背景，重点看主体一致性。
5. fp8 与非 fp8 对照，记录质量差异。
6. offload 与非 offload 对照，记录耗时和稳定性。
7. DashScope remote rewrite 与无 rewrite 对照，评估是否值得引入外部 API。

