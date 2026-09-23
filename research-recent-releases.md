# 近期 ASR 发布与重大更新：优先覆盖新进展

核查截止：2026-09-23。选取原则：历史模型保留解释技术路线所必需的代表；2026年发布或发生重大能力变化的模型优先核查，尤其关注最近三个月。以下不是全部新模型清单，覆盖的是本次检索到、有一手证据且影响路线判断的新进展。日期明确区分发布公告、论文提交与文档查询，不把三者混同。

## 1. 最近进展速览

| 模型/更新 | 时间依据 | 对技术路线的意义 | 证据与边界 |
|---|---|---|---|
| ElevenLabs Scribe v2 Medical | 2026-09-22 官方GA公告 | 通用转写之上做医疗领域适配 | 批量接口，不能当作Medical实时模型；见下节 |
| Xiaomi-CocktailASR-1 | 技术报告2026-09-10提交 | 用参考声音识别目标说话人，处理多人重叠 | 不是默认转写所有人的普通ASR。[论文](https://arxiv.org/abs/2609.11274) |
| MAI-Transcribe-2 | 2026-09-03 官方公告 | 微软自研转写升级；把时间、分人、文本风格整合到接口 | Azure接入仍有预览及长音频分人限制，见下节 |
| VibeVoice-ASR-Streaming | 2026-09-03 官方仓库公告 | 长音频联合建模之外发展专门流式版本 | 与离线ASR、TTS分开测。[仓库](https://github.com/microsoft/VibeVoice) |
| NVIDIA Nemotron-Labs-Audex-30B-A3B | 2026年7月技术报告 | ASR融入统一音频—文本理解与生成模型 | 30B MoE/3B激活不等于3B内存；模型卡为非商业许可。[论文](https://arxiv.org/abs/2607.05196)、[模型卡](https://huggingface.co/nvidia/Nemotron-Labs-Audex-30B-A3B) |
| MOSS-Transcribe-Diarize | 2026-07-09权重发布；论文初稿更早 | 一次产生转写、时间与说话人 | 单独展开，不以MOSS-TTS代替ASR介绍 |
| AssemblyAI Universal-3.5 Pro | Realtime 2026-06-23；文件版07-07，官方发布索引 | 上下文、混说和说话人结合的产品化 | 文件与Realtime分别验证；09-15 Dictation API属于工作流更新。[发布索引](https://www.assemblyai.com/collection/releases) |
| Soniox v5 | Realtime 2026-06-16公告；另有v5 Async | 输出向带说话人和语言信息的结构化流发展 | 使用v5精确ID，不能停留在旧v4介绍。[实时公告](https://soniox.com/blog/soniox-v5-real-time)、[Async公告](https://soniox.com/blog/soniox-v5-async) |
| Deepgram Flux Multilingual | 2026年4月：changelog 04-15，新闻稿04-29 | 把轮次与识别一起优化并扩到多语种 | 首发10语言不含中文；两份官方日期不同，保留来源口径。[变更记录](https://developers.deepgram.com/changelog/2026/4/15)、[新闻稿](https://deepgram.com/learn/deepgram-launches-flux-multilingual-press-release) |
| Granite-4.0-1B-Speech | 2026-03-06 模型卡 | 小型Speech-LLM、关键词偏置 | 语音输入语言与翻译输出语言不同。[IBM模型卡](https://huggingface.co/ibm-granite/granite-4.0-1b-speech) |
| Voxtral Mini 4B Realtime 2602 | 2026年2月发布系列 | 开放权重的专门实时识别路线 | 独立于文件版Voxtral Mini Transcribe V2。[Mistral公告](https://mistral.ai/news/voxtral-transcribe-2/) |
| Qwen3-ASR / FireRedASR2 | 2026年新系列，详见后文模型专题 | 中文、多语言与专门ASR持续演进 | 保留精确权重、流式和语言能力差异。[Qwen论文](https://arxiv.org/abs/2601.21337)、[FireRed仓库](https://github.com/FireRedTeam/FireRedASR2S) |
| GPT-Transcribe；Qwen-Audio-3.1-ASR-Flash | 2026-09-23当前官方文档已列出；此处不推断首发日 | 上下文转写、实时/文件商品分化 | 已进入商业生态；未核实发布日期不写成“本月发布”。[OpenAI模型页](https://developers.openai.com/api/docs/models/gpt-transcribe)、[阿里模型概述](https://help.aliyun.com/zh/model-studio/asr-model) |

## 2. MAI：应研究 Transcribe-2，而非只提第一代

微软2026-09-03发布 MAI-Transcribe-2，公告以多语言准确率与效率作为主要改进方向。公告中的“最快、最准确、最便宜”是厂商比较结论，本报告不将其转换为无条件排名。[Microsoft AI发布](https://microsoft.ai/news/mai-transcribe-2-is-the-fastest-most-accurate-and-cheapest-speech-recognition-model-in-the-world/)

当前Azure官方页列出60语言、词级时间戳、说话人分段、关键词偏置和逐字/清洁文本风格，并列出2与1.5，标注1在2026-08-20弃用。该接入页仍为public preview；特别说明约15分钟及更长录音开启分人可能失败，建议长录音关闭分人后另做处理。这是实际接口边界，不能只凭模型发布公告判断生产就绪。[Azure MAI文档](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/mai-transcribe?pivots=ai-foundry)

**路线判断：** 应把MAI-Transcribe-2列入托管文件转写候选，并把“有无分人、长短录音、逐字或清洁模式”作为不同配置验收。不能因为Azure Speech支持实时识别，就自动认定这里的MAI文件接口具有同样的增量流式能力。其研究价值在于检查“语义与结构能力增强”能否减少整个流水线和人工工作量。

## 3. ElevenLabs：Scribe家族应拆成三条路线

Scribe v2 的官方发布于2026-01-09，定位长、复杂录音的批量转写；Realtime面向低延迟交互，两者不是同一延迟设置。最新API概述还列出Medical，并分别规定能力与参数。[Scribe v2发布](https://elevenlabs.io/blog/introducing-scribe-v2)、[API能力概述](https://elevenlabs.io/docs/overview/capabilities/speech-to-text)

2026-09-22的公告将 Scribe v2 Medical 宣布为GA，模型ID为 `scribe_v2_medical`，运行于batch端点。厂商在Eka英文临床集上报告WER由8.6%降到7.0%：下降1.6个百分点，约18.6%相对下降；不能理解为提高18个百分点，也不能外推为中文医疗准确率。公告引用的Omi新榜单计划09-25更新，晚于本报告截止日，因此这里只视为厂商提前披露，未称已独立核对该榜单。[Medical GA公告](https://elevenlabs.io/blog/scribe-v2-medical-generally-available)

**路线判断：** 医疗版说明垂直领域适配仍有价值。可用同一医疗测试集比较通用版、通用版加术语、医疗版三种配置，观察收益来自模型适配还是词表；同时测药名、剂量与单位，并保留一般语音回归。对于实时产品则独立评估Scribe v2 Realtime，不将batch版的分人和长文能力直接移植过去。实际术语参数以API文档为准，当前batch与realtime限制不同，旧帮助页数字可能滞后。

## 4. 新路线不只发生在通用转写

**目标说话人识别。** Xiaomi-CocktailASR-1以参考说话人声音确定要识别的目标，包含非目标声音拒识。官方表还把剔除空输出后的WER与误拒率分开报告，因此不能直接拿其“非空WER”与其他系统完整WER比较。应联合评估目标文字、错转他人声音、错误拒识和参考样本质量。[小米官方仓库](https://github.com/xiaomi-research/xiaomi-cocktailasr-1)

**统一音频模型。** Audex同时涉及ASR、翻译、声音理解与生成，适合研究语音任务和LLM能力统一；本报告不把它当作低成本转写的默认替代。公开权重标注非商业许可，部署候选资格须先核对；思考模式、指令模式与各任务解码设置也不能混用。[NVIDIA模型卡](https://huggingface.co/nvidia/Nemotron-Labs-Audex-30B-A3B)

**轻量端侧。** ABR的Niagara与SDK代表小型本地流式路线，官方SDK文档包含 `niagara-38m-live.en` 示例。这里把它列为端侧候选线索，尚未对所有目标芯片、语言和发布包做实测或许可审查，不从“端侧”推导所有手机均可用。[ABR包说明](https://docs.appliedbrainresearch.com/sdk/concepts/application-packages/)、[ASR说明](https://docs.appliedbrainresearch.com/sdk/asr/overview/)

**相邻音频研究。** Step-Audio-R1.5涉及音频理解、推理与对话，应作为ASR外围路线跟踪；2026年9月的StepAudio 3 Gen和Music主要是生成任务，不能为了追新就列作新的ASR引擎。[R1.5论文](https://arxiv.org/abs/2604.25719)、[3 Gen论文](https://arxiv.org/abs/2609.12945)、[3 Music论文](https://arxiv.org/abs/2609.16034)

**综合判断：** 最近变化至少沿五个方向展开：原生流式与轮次控制，转写/时间/说话人联合输出，领域适配，目标说话人选择，音频理解与语言模型统一。选型应为每条相关路线放一个近期候选，再用稳定基线测收益；“模型新”代表值得验证，不代表可以跳过同条件评测。
