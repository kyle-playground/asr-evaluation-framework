# ASR Evaluation Framework

面向不同 ASR 模型与方法的评测框架设计，以及模型、数据、后处理、说话人和公开基准调研。

当前交付为研究与架构设计，尚未实现评测框架或运行模型验证。资料以各文档标明的核查日期为准；榜单结果只适用于对应的数据、版本与评测协议。

## 框架设计

设计原则：统一评测契约，让方法保留自己的内部结构；数据来源、实验视图、方法运行和评分独立扩展。

建议阅读顺序：

1. [总体设计](ASR-framework-design.md)：模块、方法接口、结果表示、运行、评分和缓存。
2. [数据抽象](ASR-framework-data-design.md)：数据来源、媒体资产、标注层、实验视图与供给方式。
3. [方法适配检查](ASR-framework-method-fit.md)：24 类方法/机制的纸面适配与拒绝规则。
4. [架构审查](ASR-framework-architecture-review.md)：设计取舍、反例审查和 33 项后续验收规格。
5. [方法证据](ASR-framework-method-evidence.md)：8 类方法的一手资料核查与未知项。
6. [具体接入伪代码](ASR-framework-adapter-pseudocode.md)：ElevenLabs 文件/实时 API、MOSS 自部署、Whisper 本地，以及长录音和纠错组合。

## 综合调研

- [ASR 深度调研](ASR-deep-research-2026-09-23.md)
- [后处理与说话人质量](ASR-postprocessing-speaker-quality-2026-09-23.md)
- [评测基准深度调研](ASR-benchmark-deep-research-2026-09-23.md)
- [日语、韩语及专项基准排名](ASR-benchmark-rankings-JA-KO-2026-09-23.md)

## 专题材料

| 主题 | 文档 |
|---|---|
| 模型与生态 | [模型](research-models.md)、[近期发布](research-recent-releases.md)、[MOSS](research-new-moss.md)、[市场概览](research-overview-market.md) |
| 数据与系统 | [数据和评测](research-data-evaluation.md)、[生产系统](research-systems.md)、[现有评测框架](research-asr-evaluation-frameworks.md) |
| 方法改进 | [纠错方法](research-correction-methods.md)、[后处理设计](research-postprocessing-design.md)、[说话人质量](research-speaker-quality.md) |
| 排名核查 | [日语](research-japanese-asr-rankings.md)、[韩语](research-korean-asr-rankings.md)、[专项](research-specialized-asr-rankings.md) |

## 数据快照

- [2026-09-23 榜单快照说明](benchmark-snapshots/2026-09-23/README.md)
- [FFASR 排名 CSV](ffasr-ranking-snapshot-2026-09-23.csv)

快照包含公开表格、检索记录以及用于核查的第三方源码文本。其出处和使用条件应以原始来源为准；不将第三方材料视为本项目原创代码。
