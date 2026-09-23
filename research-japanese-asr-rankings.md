# 日语 ASR benchmark 与实际排名核查

核查日期：2026-09-23。以下是公开结果的整理，未重新运行模型。排名只适用于各表参与模型和评分协议；未发现一个囊括所有日语模型、所有测试集的统一权威总榜。CER 均为百分比，越低越好。公开数据集不等于持续维护的排行榜。

## 1. FLEURS 日语全量 test：可复现社区比较

`kuro-shiba/ASR_ja_comparison` 的 650 条 `ja_jp/test`，2026-04-18 冻结，snapshot `fleurs_ja-test-first_n-650-add52417544c`。这是实验作者发布的比较，8 个完成模型，不是 FLEURS 官方榜；缺 Whisper large-v3、Parakeet RNNT 1.1B 等。原榜按统计 tier 后以速度打破平局，故并非单纯按 CER 排序。[原始结果、置信区间和规则](https://github.com/kuro-shiba/ASR_ja_comparison/blob/main/asr/fleurs_ja/results/fleurs-ja-650-summary-pass1-no-whisper-large-v3.md)

| 原榜名次 | 模型 | CER % | tier |
|---:|---|---:|---:|
| 1 | Cohere Transcribe 03-2026 | 4.47 | 1 |
| 2 | NVIDIA Parakeet TDT CTC 0.6B JA | 5.92 | 2 |
| 3 | Whisper large-v3-turbo | 5.73 | 2 |
| 4 | ReazonSpeech NeMo v2 | 5.59 | 2 |
| 5 | Qwen3-ASR-1.7B | 6.06 | 2 |

若只按 CER 数值，前五为 Cohere、Reazon NeMo、Whisper turbo、Parakeet JA、Qwen3-ASR。不要把同 tier 中的小差距称为显著优势。[复现实验仓库](https://github.com/kuro-shiba/ASR_ja_comparison)

## 2. FLEURS 日语全量 test：2026-09 的另一组完整测评

2026-09-01，Classmethod 实验作者测试 6 个模型、7 个运行配置；650 条、2.36 小时，数据 revision `70bb2e84b976b7e960aa89f1c648e09c59f894dd`。指标为 micro-average normalized CER，NFKC、大小写统一、去空白/标点/符号；不转换汉字读音或数字。以下按 CER 排序。[原始实验](https://dev.classmethod.jp/articles/japanese-asr-l40s-fleurs-benchmark/)

| 名次 | 模型 / 配置 | CER % |
|---:|---|---:|
| 1 | Whisper large-v3 / faster-whisper | 4.49 |
| 2 | Whisper large-v3-turbo / faster-whisper | 4.81 |
| 3 | Parakeet TDT-CTC 0.6B Japanese / NeMo TDT | 5.94 |
| 4 | Kotoba-Whisper v2.2 / Transformers ASR weights | 6.92 |
| 5 | ReazonSpeech K2 v2 / sherpa-onnx | 9.40 |

没有评测 Cohere 或 Qwen，因此不能据此称 Whisper 为“日语第一”。代码、固定配置、manifest 和结果见[实验仓库](https://github.com/cm-nakamura-shogo/japanese-asr-l40s-benchmark)。与上表差异说明推理实现、规范化和参与模型范围必须连同排名保存。

## 3. Common Voice 8、JSUT、ReazonSpeech：模型卡比较表

下表是在 Liquid AI 官方模型卡所列 14 个模型中分别排序的前五；属于厂商发布比较，不是独立统一重测榜。其非 Liquid 基线注明来自 japanese-asr，包含 Kotoba 蒸馏系列。不能把后续其他论文分数硬加入同一排名。[Liquid AI 原表](https://huggingface.co/LiquidAI/LFM2.5-Audio-1.5B-JP/blob/6c34b4d590f80563f8cb2939c2ebd7686d952394/README.md)

| 名次 | Common Voice 8 日语 test | CER % | JSUT Basic5000 | CER % | ReazonSpeech held-out test | CER % |
|---:|---|---:|---|---:|---|---:|
| 1 | LFM2.5-Audio-1.5B-JP | 4.42 | Whisper large-v3 | 7.1 | ReazonSpeech NeMo v2 | 11.2 |
| 2 | Whisper large-v3 | 8.5 | ReazonSpeech NeMo v2 | 7.4 | Kotoba-Whisper v2.0 / distil-all | 11.6 |
| 3 | ReazonSpeech NeMo v2 | 9.1 | LFM2.5-Audio-1.5B-JP | 8.07 | Kotoba-Whisper v1.0 / distil-large | 12.2 |
| 4 | Kotoba-Whisper v2.0 / distil-all | 9.2 | Whisper large-v2 | 8.2 | distil-whisper-ja-reazonspeech-medium | 14.8 |
| 5 | Kotoba-Whisper v1.0 / distil-large | 9.4 | Kotoba-Whisper v2.0 / distil-all | 8.4 | Whisper large-v3 | 14.9 |

模型别名由 [japanese-asr 发布方页面](https://huggingface.co/japanese-asr)确认。对应测试集：[CV8](https://huggingface.co/datasets/japanese-asr/ja_asr.common_voice_8_0)、[JSUT](https://huggingface.co/datasets/japanese-asr/ja_asr.jsut_basic5000)、[Reazon held-out](https://huggingface.co/datasets/japanese-asr/ja_asr.reazonspeech_test)。Reazon 是电视音频领域；JSUT 以单人朗读为主，不能当成真实会议能力。

Liquid AI 仓库元数据：创建于 2026-05-26、最后修改 2026-06-11，以上固定 commit；这是仓库时间，不宣称全部基线都于该日测得。[元数据 API](https://huggingface.co/api/models/LiquidAI/LFM2.5-Audio-1.5B-JP)

历史 Kotoba 官方表（2024 年系列，2026-09-23 核查）自身仅包含 9 个 Kotoba/Whisper 模型，CV8 前三为 Whisper large-v3 8.5、Kotoba v2.0 9.2、Kotoba v1.0 9.4；JSUT 前三是 Whisper large-v3 7.1、large-v2 8.2、Kotoba v2.0 8.4；Reazon 前三为 Kotoba v2.0 11.6、v1.0 12.2、Whisper large-v3 14.9。这说明“第一”依赖参与集合。[Kotoba 当前模型卡](https://huggingface.co/kotoba-tech/kotoba-whisper-v2.0)

## 4. 新模型覆盖更广但样本小的补充比较

HEROZ 2026-08-18 作者实验，11 个模型，分别抽约 10 分钟：FLEURS 46 条、Common Voice **26.0** 127 条、JSUT 132 条；macro-average CER。以下前三仅供筛选候选，不能视作全量 benchmark 排名或与 CV8 比较。[实验方法及结果](https://techblog.heroz.jp/entry/2026/08/18/120000)

| 数据集子样本 | 第 1 | 第 2 | 第 3 |
|---|---|---|---|
| FLEURS ja | gpt-4o-transcribe 2.26% | Cohere Transcribe 2.89% | Whisper large-v3-turbo 3.88% |
| Common Voice 26.0 ja | Cohere Transcribe 20.22% | Parakeet JA 21.53% | gpt-transcribe 23.26% |
| JSUT Basic5000 | gpt-4o-transcribe 5.14% | gpt-transcribe 6.54% | gpt-4o-mini-transcribe 6.57% |

此处 `gpt-transcribe` 名字忠实保留作者表，未独立确认其版本/别名；API 结果缺少固定快照信息，复现风险高于固定开放权重。

## 5. 只有单模型结果的基准：不能伪装成排名

NVIDIA 官方 Parakeet TDT-CTC 0.6B JA 表为同一模型两种解码头的结果，greedy，无外部 LM。TDT 的 CER：JSUT 6.4%、CV8 test 7.1%、CV16.1 dev 10.1%、CV16.1 test 13.2%、TEDxJP-10k 9.0%。CTC 对应 6.5%、7.2%、10.2%、13.3%、9.1%。规范化包含 `num2words` 数字转文字，因此不直接并入第 3 节。CV16.1/TEDxJP 在这个来源没有跨模型前三名。[NVIDIA 固定版本模型卡](https://huggingface.co/nvidia/parakeet-tdt_ctc-0.6b-ja/blob/44edb27eea9317daf89333e75eb830db4b1cc298/README.md)

## 6. 可补充的开源日语评测框架

[ADLIB](https://github.com/holotherapper/adlib)面向日语表记差异和专业词，已有 DevTerm v1.0.0、247 条、3 位说话人。输出 CER、术语准确率和复合分数，支持预测 JSONL，因此适合比较不同模型和热词/纠错方法。代码 Apache-2.0，数据 CC BY-NC-SA 4.0。核查 README、DevTerm README 和目录未见已发布多模型结果表，因此没有可给出的前三名，不把它称作现成排行榜。[DevTerm 原说明](https://github.com/holotherapper/adlib/blob/main/domains/devterm/README.md)

传统数据集还应考虑 CSJ、TEDxJP；本次有可核验数字的排名主要集中上述 FLEURS、JSUT、CV 和 Reazon。不能用对 YODAS 或其他训练语料的提及代替一个实际有评分协议和结果的榜单。
