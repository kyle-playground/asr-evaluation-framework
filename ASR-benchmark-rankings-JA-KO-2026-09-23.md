# ASR Benchmark 排名整理：日语、韩语与各主要榜单

核查日期：2026-09-23。交流语言不代表目标识别语言；本文按多语言范围整理，特别补齐日语与韩语。它补充此前以评测框架为主的研究，重点是**已有结果中的模型名次与分数**。

所有 WER、CER、MER、tcpWER 表均越低越好。排名只在同一来源、同一数据／评分协议、同一受测集合内成立。未重跑模型，不将厂商自报、社区实测和官方挑战榜混成“世界总排名”。完整集、子样本、历史论文表、单模型结果分别注明。

阅读顺序：先看日韩共同测试，再看日语／韩语专门结果，最后看 HF、Artificial Analysis、远场、会议和纠错榜。前三并不能证明显著差异；若原文没有显著性证据，本文只称数值领先。

## 1. 最直接的日韩同协议比较：GigaSpeechBench

来源：[官方结果表](https://github.com/SpeechColab/GigaSpeechBench/blob/ca782bff09a424233cd3aa1aff11c346cd0f2ed7/README.md)。固定 commit `ca782bf`（2026-07-21），2026-09-23读取。指标 CER%，使用时长 >0.5秒过滤。下面分别按 JPN、KOR 列重新排序，而非照搬两语言 AVG 排序。

| 名次 | 日语模型 | 日语 CER% | 韩语模型 | 韩语 CER% |
| --- | --- | --- | --- | --- |
| 1 | FUNASR-REALTIME | 25.44 | FUNASR-REALTIME | 9.92 |
| 2 | QWEN3.5-OMNI-PLUS | 27.36 | ELEVENLABS-SCRIBE-V2 | 11.81 |
| 3 | AZURE | 27.51 | QWEN3-ASR-1.7B | 12.90 |
| 4 | QWEN3-ASR-FLASH | 28.40 | QWEN3.5-OMNI-PLUS | 13.10 |
| 5 | FUNASR-MLT-NANO | 29.03 | AZURE | 13.13 |

两语言平均榜前三为 FUNASR-REALTIME 17.68%、QWEN3.5-OMNI-PLUS 20.23%、AZURE 20.32%。但是韩语单项的 Scribe v2 和 Qwen3-ASR-1.7B 排在 Qwen3.5-Omni-Plus 前面，所以只看 AVG 会错过语言差异。

这些名称保留发布者写法，AZURE／FUNASR-REALTIME 等未在此表给完整可冻结的 API snapshot；不能擅自映射成另一个榜的同名服务版本。NVIDIA-NEMO 缺韩语值，不作为完整日韩系统排名。该集的困难音频与 FLEURS 朗读不同，绝对错误率不能横比。

## 2. 日语专门基准：有哪些、各自谁领先


核查日期：2026-09-23。以下是公开结果的整理，未重新运行模型。排名只适用于各表参与模型和评分协议；未发现一个囊括所有日语模型、所有测试集的统一权威总榜。CER 均为百分比，越低越好。公开数据集不等于持续维护的排行榜。

### 2.1. FLEURS 日语全量 test：可复现社区比较

`kuro-shiba/ASR_ja_comparison` 的 650 条 `ja_jp/test`，2026-04-18 冻结，snapshot `fleurs_ja-test-first_n-650-add52417544c`。这是实验作者发布的比较，8 个完成模型，不是 FLEURS 官方榜；缺 Whisper large-v3、Parakeet RNNT 1.1B 等。原榜按统计 tier 后以速度打破平局，故并非单纯按 CER 排序。[原始结果、置信区间和规则](https://github.com/kuro-shiba/ASR_ja_comparison/blob/main/asr/fleurs_ja/results/fleurs-ja-650-summary-pass1-no-whisper-large-v3.md)

| 原榜名次 | 模型 | CER % | tier |
|---:|---|---:|---:|
| 1 | Cohere Transcribe 03-2026 | 4.47 | 1 |
| 2 | NVIDIA Parakeet TDT CTC 0.6B JA | 5.92 | 2 |
| 3 | Whisper large-v3-turbo | 5.73 | 2 |
| 4 | ReazonSpeech NeMo v2 | 5.59 | 2 |
| 5 | Qwen3-ASR-1.7B | 6.06 | 2 |

若只按 CER 数值，前五为 Cohere、Reazon NeMo、Whisper turbo、Parakeet JA、Qwen3-ASR。不要把同 tier 中的小差距称为显著优势。[复现实验仓库](https://github.com/kuro-shiba/ASR_ja_comparison)

### 2.2. FLEURS 日语全量 test：2026-09 的另一组完整测评

2026-09-01，Classmethod 实验作者测试 6 个模型、7 个运行配置；650 条、2.36 小时，数据 revision `70bb2e84b976b7e960aa89f1c648e09c59f894dd`。指标为 micro-average normalized CER，NFKC、大小写统一、去空白/标点/符号；不转换汉字读音或数字。以下按 CER 排序。[原始实验](https://dev.classmethod.jp/articles/japanese-asr-l40s-fleurs-benchmark/)

| 名次 | 模型 / 配置 | CER % |
|---:|---|---:|
| 1 | Whisper large-v3 / faster-whisper | 4.49 |
| 2 | Whisper large-v3-turbo / faster-whisper | 4.81 |
| 3 | Parakeet TDT-CTC 0.6B Japanese / NeMo TDT | 5.94 |
| 4 | Kotoba-Whisper v2.2 / Transformers ASR weights | 6.92 |
| 5 | ReazonSpeech K2 v2 / sherpa-onnx | 9.40 |

没有评测 Cohere 或 Qwen，因此不能据此称 Whisper 为“日语第一”。代码、固定配置、manifest 和结果见[实验仓库](https://github.com/cm-nakamura-shogo/japanese-asr-l40s-benchmark)。与上表差异说明推理实现、规范化和参与模型范围必须连同排名保存。

### 2.3. Common Voice 8、JSUT、ReazonSpeech：模型卡比较表

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

### 2.4. 新模型覆盖更广但样本小的补充比较

HEROZ 2026-08-18 作者实验，11 个模型，分别抽约 10 分钟：FLEURS 46 条、Common Voice **26.0** 127 条、JSUT 132 条；macro-average CER。以下前三仅供筛选候选，不能视作全量 benchmark 排名或与 CV8 比较。[实验方法及结果](https://techblog.heroz.jp/entry/2026/08/18/120000)

| 数据集子样本 | 第 1 | 第 2 | 第 3 |
|---|---|---|---|
| FLEURS ja | gpt-4o-transcribe 2.26% | Cohere Transcribe 2.89% | Whisper large-v3-turbo 3.88% |
| Common Voice 26.0 ja | Cohere Transcribe 20.22% | Parakeet JA 21.53% | gpt-transcribe 23.26% |
| JSUT Basic5000 | gpt-4o-transcribe 5.14% | gpt-transcribe 6.54% | gpt-4o-mini-transcribe 6.57% |

此处 `gpt-transcribe` 名字忠实保留作者表，未独立确认其版本/别名；API 结果缺少固定快照信息，复现风险高于固定开放权重。

### 2.5. 只有单模型结果的基准：不能伪装成排名

NVIDIA 官方 Parakeet TDT-CTC 0.6B JA 表为同一模型两种解码头的结果，greedy，无外部 LM。TDT 的 CER：JSUT 6.4%、CV8 test 7.1%、CV16.1 dev 10.1%、CV16.1 test 13.2%、TEDxJP-10k 9.0%。CTC 对应 6.5%、7.2%、10.2%、13.3%、9.1%。规范化包含 `num2words` 数字转文字，因此不直接并入第 3 节。CV16.1/TEDxJP 在这个来源没有跨模型前三名。[NVIDIA 固定版本模型卡](https://huggingface.co/nvidia/parakeet-tdt_ctc-0.6b-ja/blob/44edb27eea9317daf89333e75eb830db4b1cc298/README.md)

### 2.6. 可补充的开源日语评测框架

[ADLIB](https://github.com/holotherapper/adlib)面向日语表记差异和专业词，已有 DevTerm v1.0.0、247 条、3 位说话人。输出 CER、术语准确率和复合分数，支持预测 JSONL，因此适合比较不同模型和热词/纠错方法。代码 Apache-2.0，数据 CC BY-NC-SA 4.0。核查 README、DevTerm README 和目录未见已发布多模型结果表，因此没有可给出的前三名，不把它称作现成排行榜。[DevTerm 原说明](https://github.com/holotherapper/adlib/blob/main/domains/devterm/README.md)

传统数据集还应考虑 CSJ、TEDxJP；本次有可核验数字的排名主要集中上述 FLEURS、JSUT、CV 和 Reazon。不能用对 YODAS 或其他训练语料的提及代替一个实际有评分协议和结果的榜单。

## 3. 韩语专门基准：有哪些、各自谁领先


检索快照：2026-09-23。下列是各来源自身受测集合内的名次，不能合成为全行业排名；所有错误率均为百分数，越低越好。未经本机复跑。

### 3.1. Korean OSS ASR Benchmark：8 个韩语数据集同口径比较

来源：[作者仓库及完整数表](https://github.com/seastar105/oss-asr-korean-benchmark)。固定快照 commit `522e0bfb99cdfa9fb029a81ba3912f28d39a4493`，最后更新 2026-04-11（加入 Raon、SenseVoice 并更改归一化）。10 个开放模型，由仓库作者统一测评，包含作者自己的微调模型，并非官方比赛。指标 CER，短音频≤30秒；AI Hub 使用 orthographic reference。CV 为15版；FLEURS Korean test；4个AI Hub场景各取validation 3,000条，Kspon取eval-clean/other；其余版本未明确。

为精简，R=Raon-Speech-9B；K=whisper-medium-komixv2；W=whisper-large-v3；T=whisper-large-v3-turbo；Q=Qwen3-ASR-1.7B。

| 数据集 / 聚合 | 第一 | 第二 | 第三 |
|---|---:|---:|---:|
| 8项 Average | R 6.44 | K 7.05 | W 7.83 |
| Common Voice 15 ko | R 4.43 | T 5.48 | Q 5.73 |
| FLEURS ko | R 2.89 | Q 2.96 | W 3.01 |
| KsponSpeech eval-clean | R 8.66 | K 8.91 | W 14.16 |
| KsponSpeech eval-other | K 8.31 | R 9.10 | Q 10.82 |
| Callcenter 低质量电话 | W 5.34 | R 5.72 | K 5.78 |
| Conference 广播/讨论 | R 8.98 | K 9.33 | W 9.41 |
| Callcenter2 咨询电话 | R 3.48 | W 3.74 | T 4.17 |
| Lecture 教学广播 | W 8.07 | R 8.27 | K 8.37 |

其余5模型：Whisper medium、Qwen3-ASR-0.6B、Cohere Transcribe 03-2026、SenseVoiceSmall、Voxtral Mini 4B Realtime 2602。作者说明 Raon 的 FLEURS/Kspon 数值不同于其技术报告，可能因reference不同。README文字仍称W平均最好，与更新后的表矛盾；这里按数表排序。RTF硬件和批处理条件不全相同，不能据此直接排速度榜。

### 3.2. HiKE：韩英混说，正式 EACL 2026 版

[正式论文 Table 2、3](https://aclanthology.org/2026.findings-eacl.33.pdf)，会议时间2026-03-24—29。作者对10模型的统一基线实验；另列作者微调方法，非持续提交榜。1,121条、约2.2小时、13位双语者；MER混合韩文字符与英文词，PIER只看切换位置。

| Overall MER 名次 | 模型 | MER↓ | PIER↓ |
|---|---|---:|---:|
| 1 | GPT-4o-transcribe | 21.8 | 28.8 |
| 2 | Whisper-Large | 26.1 | 36.0 |
| 3 | Whisper-Medium | 31.3 | 41.3 |
| 4 | SenseVoice-Small | 32.5 | 54.7 |
| 5 | Whisper-Small | 43.5 | 50.1 |

PIER前3相同；第4 Whisper-Small 50.1，第5 SenseVoice-Small 54.7。受测其余模型：Whisper tiny/base、Seamless-M4T-v2-large、Gemma-3n、Audio-Flamingo-3。Whisper-Large这里保留论文名称，不擅自写large-v3。

方法实验：Whisper-Medium经真实句内混说微调 MER=9.0；合并真实与合成数据=20.4；仅合成句间混说=22.1；原模型31.3。这很好地展示了「不同手段」的评测。

**版本提醒：2025 arXiv v2 数表与正式版不同，不能混用；以上全部使用2026正式论文。** [开源评测框架](https://github.com/ThetaOne-AI/HiKE) 支持扩展 BaseASR、借词双写法归一化、分词/短语/句子切换分析。

### 3.3. KoALa-Bench：只能摘 ASR 分榜，不能用 QA 总分代替

来源：[2026年4月论文 Table 2](https://arxiv.org/html/2604.19782v1)。它有真正的转写子任务，但其余QA、指令遵循、忠实度不属于ASR。以下是作者5个音频语言模型实验中的top3，CER↓；不是专用ASR模型全集。括号里是额外加噪结果。

| ASR 子集 | 第一 | 第二 | 第三 |
|---|---|---|---|
| Zeroth，457条 | Qwen3-Omni 3.33 (3.91) | GPT-audio-mini 6.87 (9.00) | Gemini-flash-lite 13.60 (14.56) |
| Common Voice，523条 | Qwen3-Omni 4.96 (6.78) | Gemini-flash-lite 13.74 (26.74) | GPT-audio-mini 33.05 (36.21) |
| KsponSpeech-Clean，3,000条 | Qwen3-Omni 8.46 | Voxtral 62.62 | Gemini-flash-lite 83.19 |
| KsponSpeech-Other，3,000条 | Qwen3-Omni 7.91 | Gemini-flash-lite 45.14 | Voxtral 56.04 |

完整模型：Qwen3-Omni-30B-A3B-Instruct、Gemma-3n-E4B-it、Voxtral-Mini-3B-2507、GPT-audio-mini、Gemini-flash-lite（API精确dated snapshot未明确）。Common Voice版本未知；不与上面CV15混排。开源仓库提供backend注册和评测脚本：[KoALa-Bench](https://github.com/scai-research/KoALa-Bench)。

### 3.4. SKT A.X K2 ALM 的 Kspon 比较：独立厂商表

[SKT 模型卡](https://huggingface.co/skt/A.X-K2-ALM)，2026-09-23访问，实验具体日期未知。厂商自测4个音频语言模型，CER↓。

| 名次（两子集一致） | 模型 | eval-clean | eval-other |
|---|---|---:|---:|
| 1 | Qwen3-Omni 30B-A3B | 8.46 | 7.91 |
| 2 | A.X K2 ALM 22B-A4B | 9.00 | 9.12 |
| 3 | HyperCLOVA X 8B Omni | 10.22 | 10.15 |
| 4 | Qwen2.5-Omni 7B | 18.96 | 22.72 |

该表是模型卡STT分项。KVoiceBench后面的Speech-QA/IF平均分和语音生成CER不可替代ASR排名；本表也不可与第一节CER直接拼榜。

### 3.5. 最新模型卡补充，但不足以拼出新总榜

[BuzzASR Korean 模型卡](https://huggingface.co/BuzzASR/korean)，2026年9月论文，作者自测：FLEURS CER 5.16、WER 13.22；CV25 CER 4.24、WER 14.11；合并CER 4.61、WER 13.74。Whisper-v3对应CER为5.31、6.01、5.72。只明确给两模型逐集分数，故不能在这里给可靠top3；其“最好”措辞限定于自身比较集合与合并测试集。训练使用FLEURS及CV，因此不能当作zero-shot成绩。

[Sori-1B](https://huggingface.co/snkii/Sori-1B) 虽名称是韩语“声音”，公开页面主要呈现音频理解/MMAU及ASR调用，没有核实到同协议韩语转写CER榜；不列作韩语ASR冠军。

### 实用结论

韩语选择基准应该覆盖：朗读（FLEURS/CV）、自然对话（Kspon）、实际领域（电话/会议/讲课）、韩英混说（HiKE）。每类都有可核实结果，但受测集合、reference和normalize不同，因此没有可诚实宣称的统一“韩语第一名”。建议先复用第1节脚本和HiKE做统一复测；若要评测音频LLM，则添加KoALa的ASR子任务。

## 4. HF Open ASR Leaderboard 当前各赛道

来源：[实时榜单](https://huggingface.co/spaces/hf-audio/open_asr_leaderboard)、[前端公开数据](https://hf-audio-open-asr-leaderboard.hf.space/config)、[版本与数据源配置](https://huggingface.co/spaces/hf-audio/open_asr_leaderboard/blob/main/init.py)。以下取2026-09-23读取的 `latest` 展示值；版本登记中最新日期为2026-09-19。默认勾选数据会改变名次。

**目前多语言默认列是德、法、意、西、葡、印地语、荷兰语，没有日语、韩语。因此不能把下列多语言榜名次转述为日韩成绩。**

### 4.1 英语短音频默认总榜 Top 10

默认平均含8个公开数据列＋Private scripted／conversational两列；采用展示计算的 Average WER，不用旧 CSV 的 `avg` 列替代。RTFx是本地评测吞吐，不是流式首字延迟。

| 原榜名次 | 模型／服务版本 | Average WER% | RTFx |
| --- | --- | --- | --- |
| 1 | zoom/scribe_v2_pro | 4.38 | — |
| 2 | microsoft/azure-speech-07-2026 | 4.53 | — |
| 3 | modulate/multilingual | 4.56 | — |
| 4 | reson8/resonant-1 | 4.77 | — |
| 5 | reson8/resonant-1-flash | 4.79 | — |
| 6 | elevenlabs/scribe_v2 | 4.84 | — |
| 7 | zoom/scribe_v1 | 4.9 | — |
| 8 | microsoft/azure-speech-06-2026 | 4.91 | — |
| 9 | Qwen/Qwen3-ASR-1.7B-hf | 4.95 | 819.96 |
| 10 | assemblyai/universal-3-5-pro | 5.02 | — |

本表第一为 Zoom Scribe v2 Pro；Qwen3-ASR-1.7B-hf 在默认混合榜为第9。这里不额外按“是否有RTFx”推断许可证或创建开源榜，开放程度需回到模型许可证核查。

### 4.2 默认短音频中每个数据集的前三

仅按该列有分数的条目排序；并列保留相同分数。以下是该 HF 快照内的分项比较，不代表这些数据集跨所有论文的历史 SOTA。

| 数据集／子集 | 前三位置（同分可能多于3项） |
| --- | --- |
| AMI-Cleaned | zoom/scribe_v2_pro 6.08%; modulate/multilingual 6.31%; zoom/scribe_v1 6.94% |
| Earnings22-Cleaned-AA-chunked | zoom/scribe_v2_pro 4.12%; smallestai/pulse 4.74%; elevenlabs/scribe_v2 4.80% |
| Gigaspeech-Cleaned | HojoAI/Hojo-ASR-V1 6.14%; OpenMOSS-Team/MOSS-Transcribe-preview-2B 6.69%; modulate/multilingual 6.91% |
| LS Clean | modulate/multilingual 0.87%; reson8/resonant-1 0.92%; reson8/resonant-1-flash 0.95% |
| LS Other | modulate/multilingual 1.77%; CohereLabs/cohere-transcribe-03-2026 2.05%; bosonai/higgs-audio-v3-8b-stt-v2 2.06% |
| SPGISpeech | zoom/scribe_v2_pro 1.36%; zoom/scribe_v1 1.43%; OpenMOSS-Team/MOSS-Transcribe-preview-2B 1.61% |
| Voice Arena Monsoon | zoom/scribe_v2_pro 3.00%; microsoft/azure-speech-07-2026 3.00%; reson8/resonant-1 3.04%; reson8/resonant-1-flash 3.04% |
| Voxpopuli-AA-Cleaned | elevenlabs/scribe_v2 1.41%; microsoft/azure-speech-07-2026 1.65%; assemblyai/universal-3-5-pro 1.92% |
| Private (scripted) | microsoft/azure-speech-07-2026 2.12%; microsoft/azure-speech-06-2026 2.27%; modulate/multilingual 2.56% |
| Private (conversational) | zoom/scribe_v2_pro 12.10%; Qwen/Qwen3-ASR-1.7B-hf 12.14%; modulate/multilingual 12.30% |

### 4.3 多语言默认7语言总榜 Top 5

| 原榜名次 | 模型／服务 | 宏平均 WER% | 覆盖 |
| --- | --- | --- | --- |
| 1 | elevenlabs/scribe_v2 | 3.56 | 7/7 |
| 2 | microsoft/azure-speech-06-2026 | 3.73 | 7/7 |
| 3 | microsoft/azure-speech-07-2026 | 4.14 | 7/7 |
| 4 | assemblyai/universal-3-5-pro | 4.48 | 7/7 |
| 5 | soniox/stt-async-v5 | 5.09 | 7/7 |

按语言均分再宏平均，需完整覆盖选中的语言才参与该组合排名。部分语言有私有数据补充，不能仅用公开 CSV 重算后冒充当前展示值。与旧版按所有数据集直接平均的结果不可混用。

### 4.4 英语长音频 Top 5

当前显示的三列为 Earnings21、Earnings22、CORAAL；不是旧 CSV `Average` 中可能含 TED-LIUM 的口径。

| 原榜名次 | 模型／服务 | Average WER% | Earnings21 | Earnings22 | CORAAL |
| --- | --- | --- | --- | --- | --- |
| 1 | elevenlabs/scribe_v2 | 9.05 | 6.48 | 9.99 | 10.67 |
| 2 | assemblyai/universal-3-pro | 10.35 | 7.62 | 10.59 | 12.83 |
| 3 | reson8/resonant-1-flash | 10.46 | 7.61 | 11.03 | 12.74 |
| 4 | reson8/resonant-1 | 10.56 | 7.66 | 11.15 | 12.87333333 |
| 5 | speechmatics/enhanced | 10.98 | 7.9 | 10.75 | 14.29 |

### 4.5 私有数据赛道 Top 5

平均由 scripted 与 conversational 分组平均构成。显示到两位小数的相同成绩不强行解释为实际差异；下面保留榜单行序。

| 原榜行序 | 模型／服务 | 平均 WER% | Scripted | Conversational |
| --- | --- | --- | --- | --- |
| 1 | microsoft/azure-speech-07-2026 | 7.42 | 2.12 | 12.71 |
| 2 | modulate/multilingual | 7.43 | 2.56 | 12.3 |
| 3 | Qwen/Qwen3-ASR-1.7B-hf | 7.51 | 2.87 | 12.14 |
| 4 | microsoft/azure-speech-06-2026 | 7.51 | 2.27 | 12.75 |
| 5 | zoom/scribe_v2_pro | 7.51 | 2.92 | 12.1 |

## 5. Artificial Analysis：离线与流式分开

来源：[离线页面](https://artificialanalysis.ai/speech-to-text/non-streaming)、[流式页面](https://artificialanalysis.ai/speech-to-text/streaming)、[评分方法](https://artificialanalysis.ai/methodology/speech-to-text)。2026-09-23页面快照；AA-WER由AgentTalk/VoxPopuli/Earnings22按50/25/25加权。这是英语 API／模型服务比较，不能作为日韩榜。

### 5.1 离线 API 明细集合 Top 10

按页面可取的59条服务明细的精确AA-WER重排；不是声称覆盖页面全部71个模型，也不把显示用的四舍五入数字当成精确并列。以每个模型／服务配置为条目，旧版也可能在内。

| 名次 | 模型／服务 | AA-WER% |
| --- | --- | --- |
| 1 | StepAudio 3 ASR | 1.7252 |
| 2 | Fun-Realtime-ASR-preview, Alibaba | 1.7279 |
| 3 | MAI-Transcribe-2 | 2.0400 |
| 4 | Scribe v2 | 2.1800 |
| 5 | Grok Voice Transcribe 2.0 | 2.2900 |
| 6 | MAI-Transcribe-1.5 | 2.3800 |
| 7 | Smallest AI Pulse Pro | 2.4300 |
| 8 | Gemini 3.5 Transcribe | 2.5996 |
| 9 | MAI-Transcribe-1 | 2.6100 |
| 10 | Voxtral Small | 2.7700 |

前两项显示到两位小数都是1.73%，不能据极小数值差距宣称显著领先。此处保留四位仅为解释排序，不表示四位精度具有统计意义。首页 Highlights 是精选模型，不是全榜；本文使用服务明细而非精选卡片排序。

### 5.2 流式最终转写准确率 Top 10

使用页面默认勾选的31条配置，按 final AA-WER排序。延迟是语音结束之后的 final transcript 时间，包含该榜强制端点／fallback规则，并非从开口到首字。

| 名次 | 模型／配置 | Final AA-WER% | 句末后 final延迟ms |
| --- | --- | --- | --- |
| 1 | Grok Voice Transcribe 2.0 (Streaming) | 2.73 | 490 |
| 2 | Muse Voice Transcribe | 3.06 | 163 |
| 3 | Cartesia Ink Preview (external endpoints) | 3.11 | 105 |
| 4 | Cartesia Ink Preview (semantic endpoints) | 3.21 | 427 |
| 5 | Cartesia Ink-2 (semantic endpoints) | 3.36 | 431 |
| 6 | ElevenLabs Scribe v2 Realtime | 3.59 | 141 |
| 7 | Qwen3 ASR Flash Realtime | 3.73 | 476 |
| 8 | GPT Live Transcribe | 3.92 | 812 |
| 9 | Grok Voice Transcribe 1.0 (Streaming) | 3.93 | 373 |
| 10 | Alebex STT | 3.96 | 742 |

Cartesia 的 external／semantic endpoints 是不同策略配置，必须分行。Preview 名称按页面原样保留，不映射成正式版。同一个模型“准确率榜第一”不表示“延迟榜第一”。

### 5.3 流式最终提交延迟 Top 5

| 名次 | 模型／配置 | 句末后 final延迟ms | Final AA-WER% |
| --- | --- | --- | --- |
| 1 | Deepgram Flux | 21 | 7.39 |
| 2 | Soniox v5 Real-Time | 54 | 4.50 |
| 3 | Deepgram Nova-3 Realtime | 66 | 6.59 |
| 4 | Cartesia Ink-2 (external endpoints) | 67 | 4.02 |
| 5 | Nemotron 3 ASR 80ms, Together AI | 70 | 7.03 |

## 6. GigaSpeechBench 其余分榜

来源仍为[同一固定版本官方表](https://github.com/SpeechColab/GigaSpeechBench/blob/ca782bff09a424233cd3aa1aff11c346cd0f2ed7/README.md)。各表保留官方AVG%的排序；语言覆盖、行业权重和实体指标不同，不计算跨表总冠军。下表是前3个名次位置，同分追加列出。

| 分榜与指标 | 前三位置 |
| --- | --- |
| 🌏 Low-Resource — Southeast Asian (WER %) | FUNASR-REALTIME 16.85%; QWEN3.5-OMNI-PLUS 19.61%; CHIRP-3 20.87% |
| 🌍 Low-Resource — Arabic (WER %) | QWEN3.5-OMNI-PLUS 32.80%; GEMINI-3.0-FLASH 36.22%; CHIRP-3 38.23% |
| 🗣️ CH-EN Dialects — Chinese Dialects (CER %) | FUNASR-REALTIME 22.79%; QWEN3.5-OMNI-PLUS 27.22%; SEEDASR 29.41% |
| 🗣️ CH-EN Dialects — English Accents (WER %) | QWEN3.5-OMNI-PLUS 14.05%; FUNASR-REALTIME 14.40%; QWEN3-ASR-1.7B 15.12% |
| 🏢 Vertical Domain — Chinese B-CER (%) [Hotword] | FUNASR-REALTIME 10.91%; QWEN3.5-OMNI-PLUS 11.89%; SEEDASR 14.79% |
| 🏢 Vertical Domain — Chinese CER (%) | FUNASR-REALTIME 3.12%; QWEN3.5-OMNI-PLUS 3.36%; SEEDASR 3.84%; BIGASR 3.84% |
| 🏢 Vertical Domain — English B-WER (%) [Hotword] | QWEN3.5-OMNI-PLUS 12.80%; FUNASR-REALTIME 12.94%; GEMINI-3.0-FLASH 13.67% |
| 🏢 Vertical Domain — English WER (%) | QWEN3-ASR-1.7B 6.64%; FUNASR-REALTIME 6.95%; QWEN3.5-OMNI-PLUS 7.26% |

B-CER／B-WER 是实体相关错误率，不能替代整体CER/WER。论文介绍的任务范围比当前README有数表的范围更广；未公开足够结果的老人／儿童等部分，不据介绍文字制造排名。

## 7. 其他已介绍榜单：远场、会议、中文历史与纠错

以下单独标明最新可取结果、历史挑战结果与论文内部比较。


核查时间：2026-09-23。以下不同测试集、历史日期与任务条件不可横向比较分数。错误率均越低越好；RTFx 越高吞吐越快。

### FFASR：当前远场榜

从[官方 Space](https://huggingface.co/spaces/treble-technologies/ffasr)的公开 Gradio `_on_startup` 接口直接获取；Latest 下拉菜单最新版本为 **2026-09-05 13:38 UTC**，共 38 行。主榜是近场、High/Mid/Low SNR 四列 WER 的平均；不是只算远场，也不包含 beta moving-source 列。固定 L4 测试，统一 held-out 音频与规范化。[官方说明](https://huggingface.co/blog/ffasr-leaderboard)

| 排名 | 模型 | 平均 WER % | 近场 | High SNR | Mid SNR | Low SNR | RTFx |
|---|---|---:|---:|---:|---:|---:|---:|
| 1 | zhifeixie/Mega-ASR | 13.38 | 3.88 | 7.17 | 14.05 | 28.42 | 19.7833 |
| 2 | Qwen/Qwen3-ASR-1.7B | 13.41 | 3.76 | 6.74 | 13.89 | 29.26 | 25.7132 |
| 3 | CohereLabs/cohere-transcribe-03-2026 | 15.03 | 4.28 | 7.79 | 15.25 | 32.79 | 63.2846 |
| 4 | ibm-granite/granite-speech-4.1-2b-nar | 15.16 | 4.07 | 8.22 | 16.41 | 31.95 | 211.1197 |
| 5 | rohansheth/tiro-qwen3-asr-1.7b-ffasr-v1 | 15.29 | 4.11 | 8.36 | 15.79 | 32.89 | 28.0293 |

Whisper-large-v3 在同榜第 12，18.02%；Parakeet-tdt-0.6b-v3 第 13，19.04%。完整数据见 [CSV 快照](ffasr-ranking-snapshot-2026-09-23.csv)。这里只能说明该远场测试协议下的结果，不能推出日韩识别排名。Mega-ASR 与 Qwen3 只差 0.03 个百分点，榜单未提供置信区间，不能把它解读为稳定的显著优势。

### SpeechColab / SpeechIO：历史分测试集结果，没有已核实的当前总榜

[仓库首页](https://github.com/SpeechColab/Leaderboard)当前引用的英文成绩图是 **2022 年 10 月**，中文是 **2024 年 8 月**。不是持续更新至 2026 年的统一模型总榜。官方图按测试集展示错误率，没有定义跨全部数据的总分，因此不应自行补一个“官方综合第一”。下表仅将英文历史图的对应 test 列从低到高排序：

| 排名 | GigaSpeech v1 test：WER % | Common Voice v11 test：WER % |
|---|---|---|
| 1 | microsoft_sdk_en：9.02 | nemo_conformer_transducer_xlarge_en：5.72 |
| 2 | whisper_large_v2：9.19 | nemo_conformer_ctc_large_en：9.14 |
| 3 | whisper_large：9.68 | whisper_large_v2：10.01 |
| 4 | k2_gigaspeech：9.71 | microsoft_sdk_en：10.36 |
| 5 | tencent_api_en：10.37 | whisper_large：10.79 |

来源：[官方英文原图](https://github.com/SpeechColab/Leaderboard/blob/master/misc/SpeechColab_ASR_EN_2022_10.png)。中文逐场景 CER 见[2024-08 官方原图](https://github.com/SpeechColab/Leaderboard/blob/master/misc/SpeechIO_TIOBE_2024_08.png)。这些 API 名称是当时的接口配置，不代表今天的产品版本；该仓库首页数据清单主要为中英，也不能拿来代替日韩榜。

### CHiME-8 DASR：2024 挑战赛最终结果

按 constrained LM 主赛道 **macro tcpWER(eval)** 排序，宏平均覆盖 CHiME-6、DiPCo、Mixer6、NOTSOFAR-1 四场景。这里测的是 diarization + ASR 的整套多人会议系统；不能当成基础 ASR 模型的普通 WER 排名。

| 排名（含官方基线） | 团队 / 系统 | macro tcpWER % | macro cpWER % | macro DER % |
|---|---|---:|---:|---:|
| 1 | STCON / sys1 | 19.90 | 19.16 | 20.77 |
| 2 | NTT / ntt3 | 22.02 | 20.82 | 21.45 |
| 3 | NeMo_baseline / sys1 | 56.52 | 52.94 | 43.12 |
| 4 | ESPnet_baseline / sys1 | 62.57 | 58.78 | 30.49 |
| 5 | Anonymous Team / sys1 | 80.88 | 72.90 | 46.03 |

注意：实际参赛团队是 STCON、NTT、Anonymous；两条 baseline 不应说成参赛团队的第 3 / 4 名。unconstrained LM 只有 STCON sys2 一项，macro tcpWER **19.62%**，应分开列；同一个 NTT 的不同系统也不能拿来充当不同团队。

来源：[官方结果页](https://www.chimechallenge.org/challenges/chime8/task1/results)、[主榜 JSON](https://www.chimechallenge.org/challenges/chime8/task1/resources/main_ranking.json)、[全部系统与分场景 JSON](https://www.chimechallenge.org/challenges/chime8/task1/resources/main_ranking_scatter.json)。官方另有 CHiME-7+8 混合比较表，只比较三种共同场景；其中 STCON 的 21.3% 与这里四场景 19.62% 是不同分母，不能混用。页面的 2026 构建时间也不应当作比赛发生时间。

### ASR-EC：2025 论文内的方法排序，不是持续榜单

这是中文后处理测试，原始错误稿来自固定 Kaldi-K1 / Kaldi-K2 系统，不涉及日韩。采用 EMNLP Industry 2025 正式论文 Table 4 的 Mixed Utterances CER，不跨 A* / B* 混算。下面包括“不纠错”的 baseline，展示纠错方法是否有实际收益；排名是本报告对论文行排序。

| 排名 | ASR-EC A*：方法 / CER % | ASR-EC B*：方法 / CER % |
|---|---|---|
| 1 | Baichuan2 多模态：5.96 | Baichuan2 多模态：5.12 |
| 2 | Qwen 多模态：6.07 | Qwen 多模态：6.16 |
| 3 | ChatGLM3 多模态：11.51 | ChatGLM3 多模态：6.64 |
| 4 | Baichuan2 LoRA：12.36 | Baichuan2 LoRA：7.88 |
| 5 | 原始 ASR baseline：12.42 | 原始 ASR baseline：8.11 |

如果只列纠错系统，排除 baseline 后第 5 名分别是 ChatGLM3 LoRA 13.46%（A*）和 Qwen LoRA 8.48%（B*），均劣于相应 baseline。多模态方法使用音频加错误稿，文本方法只用错误稿，必须保留这个输入差异。这里的 Qwen 是该论文所测 7B 模型，不能替换称为 Qwen3-ASR。零样本、three-shot、multi-step 结果普遍比原始稿差，不能因此断言所有现代 LLM 纠错都无效。

来源：[正式论文 Table 4，第 7 页](https://aclanthology.org/2025.emnlp-industry.110.pdf)、[作者代码与数据](https://github.com/WANGWC1996/2025EMNLP-ASR-EC-Benchmark)。正式会议日期为 2025-11-04 至 2025-11-09，早期 arXiv 版本为 2024-12；此处明确使用正式论文版本。

## 8. 哪些只有数据或工具，没有可诚实给出的统一排名

| 名称 | 可以提供什么 | 不能提供什么 |
|---|---|---|
| FLEURS／Common Voice | 按语言、版本和同一研究给模型排序，日韩见上文 | 将不同CV版本或normalize拼成全局冠军 |
| JSUT／ReazonSpeech／KsponSpeech | 指定论文、模型卡或社区同测表内的排名 | 忽略不同reference、受测模型范围的“官方总榜” |
| AISHELL-1/4、WenetSpeech、TALCS | 任务数据与各研究中的可复现比较 | 本次未核实跨当前全部模型的统一更新榜，不追加虚构总排名 |
| ADLIB | 日语表记与术语评测代码／数据 | 本次没找到可核实多模型榜单 |
| JiWER／MeetEval／asr_eval／ASR.lab／UltraEval-Audio | 用于产生你自己的分数和实验结果 | 框架本身不是参赛模型，没有“模型名次”可填 |
| KoALa／KVoiceBench 等音频理解体系 | 只取明确的ASR子任务来比较转写 | 不能用QA或整体语音理解分数充当ASR排名 |

这里“每个 benchmark 的排名”整理的是上文具体列出的赛道与实际发表结果，不能穷尽所有数据集上的所有论文。缺结果的条目明确缺口，数据集与排行榜分开。

## 9. 对开源框架研究的补充结论

日语可增加 `kuro-shiba/ASR_ja_comparison` 的可复现FLEURS流程、`cm-nakamura-shogo/japanese-asr-l40s-benchmark` 的固定配置，以及 ADLIB 的术语／表记测试。韩语可复用 `seastar105/oss-asr-korean-benchmark` 的跨场景评测和 HiKE 的韩英混说评测。它们均不意味着任意新模型无需适配。

统一平台依然可以使用前篇研究的跨模型 harness，但normalizer、tokenizer、reference约定应按语言和任务版本化：日语重点是汉字／假名／数字的表记，韩语重点是空格、正字转写与发音转写，混说单独看语言切换。不要因为聊天使用中文而预设测试语言。

## 10. 快照与复核

本次保存了 HF 展示表、版本登记、公开CSV及AA可见页面数据提取，位于 [benchmark-snapshots/2026-09-23](benchmark-snapshots/2026-09-23)。后续更新应同时保存日期、默认过滤条件和指标口径，再比较排名变化。

HF私有源CSV直接下载返回401，但榜单公开展示其聚合分数；本文引用公开展示，不宣称取得私有音频／标注。AA静态HTML与动态图表数据不是完全相同的模型集合，离线表已限定为59条API明细，流式表限定为默认31条配置。未把搜索摘要当成最终分数。
