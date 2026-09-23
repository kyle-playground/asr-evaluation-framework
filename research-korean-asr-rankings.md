# 韩语 ASR benchmark 排名核查

检索快照：2026-09-23。下列是各来源自身受测集合内的名次，不能合成为全行业排名；所有错误率均为百分数，越低越好。未经本机复跑。

## 1. Korean OSS ASR Benchmark：8 个韩语数据集同口径比较

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

## 2. HiKE：韩英混说，正式 EACL 2026 版

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

## 3. KoALa-Bench：只能摘 ASR 分榜，不能用 QA 总分代替

来源：[2026年4月论文 Table 2](https://arxiv.org/html/2604.19782v1)。它有真正的转写子任务，但其余QA、指令遵循、忠实度不属于ASR。以下是作者5个音频语言模型实验中的top3，CER↓；不是专用ASR模型全集。括号里是额外加噪结果。

| ASR 子集 | 第一 | 第二 | 第三 |
|---|---|---|---|
| Zeroth，457条 | Qwen3-Omni 3.33 (3.91) | GPT-audio-mini 6.87 (9.00) | Gemini-flash-lite 13.60 (14.56) |
| Common Voice，523条 | Qwen3-Omni 4.96 (6.78) | Gemini-flash-lite 13.74 (26.74) | GPT-audio-mini 33.05 (36.21) |
| KsponSpeech-Clean，3,000条 | Qwen3-Omni 8.46 | Voxtral 62.62 | Gemini-flash-lite 83.19 |
| KsponSpeech-Other，3,000条 | Qwen3-Omni 7.91 | Gemini-flash-lite 45.14 | Voxtral 56.04 |

完整模型：Qwen3-Omni-30B-A3B-Instruct、Gemma-3n-E4B-it、Voxtral-Mini-3B-2507、GPT-audio-mini、Gemini-flash-lite（API精确dated snapshot未明确）。Common Voice版本未知；不与上面CV15混排。开源仓库提供backend注册和评测脚本：[KoALa-Bench](https://github.com/scai-research/KoALa-Bench)。

## 4. SKT A.X K2 ALM 的 Kspon 比较：独立厂商表

[SKT 模型卡](https://huggingface.co/skt/A.X-K2-ALM)，2026-09-23访问，实验具体日期未知。厂商自测4个音频语言模型，CER↓。

| 名次（两子集一致） | 模型 | eval-clean | eval-other |
|---|---|---:|---:|
| 1 | Qwen3-Omni 30B-A3B | 8.46 | 7.91 |
| 2 | A.X K2 ALM 22B-A4B | 9.00 | 9.12 |
| 3 | HyperCLOVA X 8B Omni | 10.22 | 10.15 |
| 4 | Qwen2.5-Omni 7B | 18.96 | 22.72 |

该表是模型卡STT分项。KVoiceBench后面的Speech-QA/IF平均分和语音生成CER不可替代ASR排名；本表也不可与第一节CER直接拼榜。

## 5. 最新模型卡补充，但不足以拼出新总榜

[BuzzASR Korean 模型卡](https://huggingface.co/BuzzASR/korean)，2026年9月论文，作者自测：FLEURS CER 5.16、WER 13.22；CV25 CER 4.24、WER 14.11；合并CER 4.61、WER 13.74。Whisper-v3对应CER为5.31、6.01、5.72。只明确给两模型逐集分数，故不能在这里给可靠top3；其“最好”措辞限定于自身比较集合与合并测试集。训练使用FLEURS及CV，因此不能当作zero-shot成绩。

[Sori-1B](https://huggingface.co/snkii/Sori-1B) 虽名称是韩语“声音”，公开页面主要呈现音频理解/MMAU及ASR调用，没有核实到同协议韩语转写CER榜；不列作韩语ASR冠军。

## 实用结论

韩语选择基准应该覆盖：朗读（FLEURS/CV）、自然对话（Kspon）、实际领域（电话/会议/讲课）、韩英混说（HiKE）。每类都有可核实结果，但受测集合、reference和normalize不同，因此没有可诚实宣称的统一“韩语第一名”。建议先复用第1节脚本和HiKE做统一复测；若要评测音频LLM，则添加KoALa的ASR子任务。
