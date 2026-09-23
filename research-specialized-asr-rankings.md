# 专项 ASR benchmark 排名核查

核查时间：2026-09-23。以下不同测试集、历史日期与任务条件不可横向比较分数。错误率均越低越好；RTFx 越高吞吐越快。

## FFASR：当前远场榜

从[官方 Space](https://huggingface.co/spaces/treble-technologies/ffasr)的公开 Gradio `_on_startup` 接口直接获取；Latest 下拉菜单最新版本为 **2026-09-05 13:38 UTC**，共 38 行。主榜是近场、High/Mid/Low SNR 四列 WER 的平均；不是只算远场，也不包含 beta moving-source 列。固定 L4 测试，统一 held-out 音频与规范化。[官方说明](https://huggingface.co/blog/ffasr-leaderboard)

| 排名 | 模型 | 平均 WER % | 近场 | High SNR | Mid SNR | Low SNR | RTFx |
|---|---|---:|---:|---:|---:|---:|---:|
| 1 | zhifeixie/Mega-ASR | 13.38 | 3.88 | 7.17 | 14.05 | 28.42 | 19.7833 |
| 2 | Qwen/Qwen3-ASR-1.7B | 13.41 | 3.76 | 6.74 | 13.89 | 29.26 | 25.7132 |
| 3 | CohereLabs/cohere-transcribe-03-2026 | 15.03 | 4.28 | 7.79 | 15.25 | 32.79 | 63.2846 |
| 4 | ibm-granite/granite-speech-4.1-2b-nar | 15.16 | 4.07 | 8.22 | 16.41 | 31.95 | 211.1197 |
| 5 | rohansheth/tiro-qwen3-asr-1.7b-ffasr-v1 | 15.29 | 4.11 | 8.36 | 15.79 | 32.89 | 28.0293 |

Whisper-large-v3 在同榜第 12，18.02%；Parakeet-tdt-0.6b-v3 第 13，19.04%。完整数据见 [CSV 快照](ffasr-ranking-snapshot-2026-09-23.csv)。这里只能说明该远场测试协议下的结果，不能推出日韩识别排名。Mega-ASR 与 Qwen3 只差 0.03 个百分点，榜单未提供置信区间，不能把它解读为稳定的显著优势。

## SpeechColab / SpeechIO：历史分测试集结果，没有已核实的当前总榜

[仓库首页](https://github.com/SpeechColab/Leaderboard)当前引用的英文成绩图是 **2022 年 10 月**，中文是 **2024 年 8 月**。不是持续更新至 2026 年的统一模型总榜。官方图按测试集展示错误率，没有定义跨全部数据的总分，因此不应自行补一个“官方综合第一”。下表仅将英文历史图的对应 test 列从低到高排序：

| 排名 | GigaSpeech v1 test：WER % | Common Voice v11 test：WER % |
|---|---|---|
| 1 | microsoft_sdk_en：9.02 | nemo_conformer_transducer_xlarge_en：5.72 |
| 2 | whisper_large_v2：9.19 | nemo_conformer_ctc_large_en：9.14 |
| 3 | whisper_large：9.68 | whisper_large_v2：10.01 |
| 4 | k2_gigaspeech：9.71 | microsoft_sdk_en：10.36 |
| 5 | tencent_api_en：10.37 | whisper_large：10.79 |

来源：[官方英文原图](https://github.com/SpeechColab/Leaderboard/blob/master/misc/SpeechColab_ASR_EN_2022_10.png)。中文逐场景 CER 见[2024-08 官方原图](https://github.com/SpeechColab/Leaderboard/blob/master/misc/SpeechIO_TIOBE_2024_08.png)。这些 API 名称是当时的接口配置，不代表今天的产品版本；该仓库首页数据清单主要为中英，也不能拿来代替日韩榜。

## CHiME-8 DASR：2024 挑战赛最终结果

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

## ASR-EC：2025 论文内的方法排序，不是持续榜单

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
