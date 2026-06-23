# 源码与发布快照

采集时间：2026-06-23 16:32:35 +08:00

## 上游仓库状态

| 项目 | 内容 |
| --- | --- |
| GitHub | https://github.com/boogu-project/Boogu-Image |
| full_name | `boogu-project/Boogu-Image` |
| 可见性 | public |
| 默认分支 | `main` |
| 本次核对 HEAD | `817ae74a4cf758a220bedf6c9ea6d0215aca0974` |
| 最新提交信息 | `fix: incorrect ti2i output when enabling Cache-DiT caching` |
| 最新提交时间 | 2026-06-23 10:19:35 +0800 |
| 许可证 | Apache-2.0 |
| 主语言 | Python |
| 上游描述 | Boogu-Image-0.1 是 Apache-2.0 开源图像生成和编辑模型族 |

本地克隆核对命令：

```powershell
git clone --depth 1 https://github.com/boogu-project/Boogu-Image.git Boogu-Image
git rev-parse HEAD
git ls-remote https://github.com/boogu-project/Boogu-Image.git HEAD
```

本次 `git rev-parse HEAD` 与远程 `ls-remote HEAD` 都指向：

```text
817ae74a4cf758a220bedf6c9ea6d0215aca0974
```

## 源码结构

上游不是普通网页应用，而是一个模型推理工程。核心结构如下：

| 路径 | 作用 |
| --- | --- |
| `README.md` / `README_CN.md` | 项目介绍、模型族、安装、模型下载、显存建议、限制说明 |
| `INFERENCE_GUIDE.md` | 详细推理参数、offload、rewriter、cache、fp8、批量推理说明 |
| `pyproject.toml` | Python 包元数据与核心依赖 |
| `inference.py` | 主推理入口，覆盖 T2I、TI2I、batch、rewriter、offload、cache 等 |
| `inference_simple.py` | Base 文生图简化示例 |
| `inference_turbo_simple.py` | Turbo 少步数文生图简化示例 |
| `inference_ti2i_simple.py` | 图生图编辑简化示例 |
| `demo_scripts/` | 多种可复制的 shell 示例 |
| `boogu/pipelines/boogu/pipeline_boogu.py` | 主要 Diffusers pipeline |
| `boogu/pipelines/boogu/pipeline_boogu_turbo.py` | Turbo pipeline |
| `boogu/models/transformers/` | Boogu transformer 相关实现 |
| `boogu/schedulers/` | 调度器 |
| `boogu/cache_functions/` | TaylorSeer 等缓存相关逻辑 |
| `utils/get_flash_attn.py` | 尝试安装适配环境的 flash-attn wheel |
| `utils/quantize_transformer.py` | transformer fp8 量化工具 |
| `batch_data_samples/` | 批量推理 YAML 示例 |
| `input_image_examples/` | 图像编辑示例输入 |
| `assets/` | README 展示图片和图表 |

## 模型族

README 中列出的主要模型：

| 模型 | 参数量 | 任务 | 步数/CFG | 权重入口 |
| --- | ---: | --- | --- | --- |
| Boogu-Image-0.1-Base | 10B | 文生图 | 25-50 step，CFG 2.0-5.0 | Hugging Face / ModelScope |
| Boogu-Image-0.1-Base-fp8 | 10B | 文生图 | 25-50 step，CFG 2.0-5.0 | Hugging Face / ModelScope |
| Boogu-Image-0.1-Edit | 10B | 图生图编辑 | 25-50 step，CFG 2.0-5.0 | Hugging Face / ModelScope |
| Boogu-Image-0.1-Edit-fp8 | 10B | 图生图编辑 | 25-50 step，CFG 2.0-5.0 | Hugging Face / ModelScope |
| Boogu-Image-0.1-Turbo | 10B | 快速文生图 | 通常 3-4 step，CFG 1.0 | Hugging Face / ModelScope |
| Boogu-Image-0.1-Turbo-fp8 | 10B | 快速文生图 | 通常 3-4 step，CFG 1.0 | Hugging Face / ModelScope |

Hugging Face API 在本次采集时可访问的三个主模型：

| 模型 | Hugging Face SHA | lastModified | downloads | likes |
| --- | --- | --- | ---: | ---: |
| `Boogu/Boogu-Image-0.1-Base` | `535b83113a705768d25c2d0b59c5a3c8fe531a90` | 2026-06-21T06:38:38Z | 368 | 40 |
| `Boogu/Boogu-Image-0.1-Turbo` | `80a69a902ce9a699cb1c18e7be32131d18978947` | 2026-06-21T06:43:57Z | 409 | 46 |
| `Boogu/Boogu-Image-0.1-Edit` | `7caaf435f807cfd41e81eb1d11e6f73e081e7ad6` | 2026-06-21T06:43:34Z | 473 | 103 |

这些数字是采集时快照，后续会变化。

## 依赖与运行环境

`pyproject.toml` 中要求：

| 项目 | 版本范围 |
| --- | --- |
| Python | `>=3.10,<3.13` |
| torch | `>=2.7.1,<2.12` |
| torchvision | `>=0.22.1,<0.27` |
| torchaudio | `>=2.7.1,<2.12` |
| diffusers | `>=0.35.2,<0.39` |
| transformers | `>=4.57.3,<6` |
| accelerate | `>=1.0` |
| torchao | `>=0.15,<0.18` |
| cache-dit | `>=1.3,<2` |
| triton | Linux x86_64 条件依赖 |

README 写明测试环境为 Python 3.10、CUDA 12.6、PyTorch 2.7.1。Flash Attention 自动安装脚本主要面向 Linux x86_64 + CUDA。

## 官方声明

上游 README_CN 中有重要声明：Boogu 团队目前没有推出任何针对 Boogu-Image 的收费 API、订阅或商业化服务；Boogu-Image-0.1 仅为研究项目，并非官方模型发布。任何以相近名称售卖的服务需要自行甄别。

## 关联项目

上游 README 在 2026-06-17 更新中提到 ComfyUI-Boogu：

- GitHub: https://github.com/boogu-project/ComfyUI-Boogu
- Hugging Face: https://huggingface.co/Comfy-Org/Boogu-Image

本次 GitHub API 快照显示 `boogu-project/ComfyUI-Boogu` 是 ComfyUI custom nodes and workflows for Boogu-Image-0.1。

