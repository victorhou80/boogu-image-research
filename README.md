# Boogu-Image 中文研究笔记

这是对 [boogu-project/Boogu-Image](https://github.com/boogu-project/Boogu-Image) 的中文研究整理仓库。它不是上游源码镜像，也不包含模型权重，只保存面向选型、部署和产品化判断的研究文档。

采集时间：2026-06-23 16:32:35 +08:00  
上游仓库：`boogu-project/Boogu-Image`  
本次核对的上游提交：`817ae74a4cf758a220bedf6c9ea6d0215aca0974`  
许可证：Apache-2.0  

## 一句话结论

Boogu-Image-0.1 是一个偏研究发布形态的开源图像生成与编辑模型族，核心价值在于“文生图 + 图像编辑 + 中英文文字渲染 + prompt 重写/理解”的统一能力。它适合继续研究、本地推理验证、做设计/电商/海报类工具原型，也适合作为 ComfyUI 或自建推理服务的底层模型候选；但它不是一个开箱即用的商业 SaaS，也不建议不加审核地直接上生产。

## 主要判断

| 维度 | 结论 |
| --- | --- |
| 项目类型 | Python 推理工程 + 模型权重入口，不是网页应用 |
| 模型族 | Base、Turbo、Edit，以及对应 fp8 版本 |
| 核心能力 | 文生图、图生图编辑、中文/英文文字渲染、海报/产品图、prompt rewriting |
| 推理入口 | `inference.py`、`inference_simple.py`、`inference_turbo_simple.py`、`demo_scripts/` |
| 硬件门槛 | 10B 模型，本地 GPU 友好度取决于分辨率、offload、fp8 和是否启用 rewriter |
| 推荐试用路径 | 先用 Turbo 或 fp8 跑 1K，再验证 Base/Edit 的 2K 和文字能力 |
| 产品化方向 | 电商商品图、海报设计、图像局部编辑、内容生产后台、ComfyUI 工作流 |
| 关键风险 | 生成内容安全、品牌/人物/IP 合规、文字稳定性、身份一致性、显存成本、上游仍处研究阶段 |

## 文档索引

- [源码与发布快照](docs/01-source-snapshot.md)
- [架构与运行机制](docs/02-architecture-and-runtime.md)
- [本地部署与试跑路线](docs/03-deployment-playbook.md)
- [产品化机会与集成建议](docs/04-productization-opportunities.md)
- [风险、限制与合规清单](docs/05-risk-and-limitations.md)
- [评测记录模板](templates/evaluation-sheet.md)

## 这项目能做什么

1. 文生图：按中文或英文 prompt 生成图片，适合摄影感、风格化、海报和产品图。
2. 图像编辑：用输入图 + 指令做删除、替换、改色、背景/材质修改等编辑。
3. 文字渲染：上游强调中英文文字渲染能力，尤其是海报、文档、界面、品牌规范等文字场景。
4. 快速生成：Turbo 版本是少步数推理变体，上游示例里使用 4 step。
5. Prompt 重写：可用本地 rewriter 或 DashScope 远程重写，把用户指令变成更适合生成模型的视觉 prompt。
6. 低显存尝试：支持 sequential CPU offload、model CPU offload、group offload、fp8、TeaCache、TaylorSeer、Cache-DiT 等策略。

## 不适合直接做什么

- 不适合当成“官方商业 API”采购。上游 README 明确提醒，Boogu 团队没有推出针对 Boogu-Image 的收费 API、订阅或商业服务。
- 不适合未经审核直接给终端用户开放生成。模型可能产生不准确、有偏见、不合规或不适当内容。
- 不适合承诺严格一致的商用修图。上游也说明图生图一致性、主体身份保持、细节稳定性仍有限。
- 不适合低配电脑轻量部署。即使有 offload/fp8，2K、高质量、多图批处理仍会带来显存和延迟压力。

## 推荐下一步

1. 如果只是看效果：优先用官方 Demo、ComfyUI-Boogu 或 Turbo 模型做小样。
2. 如果要本地验证：先跑 `Boogu-Image-0.1-Turbo-fp8` 或 `Boogu-Image-0.1-Turbo` 的 1K 文生图。
3. 如果要做产品：把 Boogu 包在独立推理服务后面，不把用户请求直接暴露给底层脚本；增加队列、审核、日志、失败重试、成本统计。
4. 如果要做电商/海报工具：单独建立中文文字渲染、品牌词、商品主体一致性和人工复核评测集。
5. 如果要商业使用：保留 Apache-2.0 许可证和上游 attribution，额外检查模型权重页、依赖库、素材、用户上传图像和生成结果的合规边界。

## 主要来源

- 上游 GitHub 仓库：[boogu-project/Boogu-Image](https://github.com/boogu-project/Boogu-Image)
- 上游 ComfyUI 适配：[boogu-project/ComfyUI-Boogu](https://github.com/boogu-project/ComfyUI-Boogu)
- Hugging Face 组织：[Boogu](https://huggingface.co/Boogu)
- ModelScope 组织：[Boogu](https://modelscope.cn/organization/Boogu)

