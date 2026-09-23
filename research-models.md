# ASR 深度研究：模型、学习范式与选型边界

研究日期：2026-09-23。范围为自动语音识别（Automatic Speech Recognition）。本文依据论文、作者官方仓库及模型卡；动态页面代表本次检索所见状态。未自行复现榜单，所有厂商成绩均应视为其指定设置下的报告结果。下文将事实与选型推断分开，避免把模型家族、具体权重和外接流水线混为一谈。

<a id="recent-models"></a>

## 近期 ASR 发布与重大更新：优先覆盖新进展

核查截止：2026-09-23。选取原则：历史模型保留解释技术路线所必需的代表；2026年发布或发生重大能力变化的模型优先核查，尤其关注最近三个月。以下不是全部新模型清单，覆盖的是本次检索到、有一手证据且影响路线判断的新进展。日期明确区分发布公告、论文提交与文档查询，不把三者混同。

### 1. 最近进展速览

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

### 2. MAI：应研究 Transcribe-2，而非只提第一代

微软2026-09-03发布 MAI-Transcribe-2，公告以多语言准确率与效率作为主要改进方向。公告中的“最快、最准确、最便宜”是厂商比较结论，本报告不将其转换为无条件排名。[Microsoft AI发布](https://microsoft.ai/news/mai-transcribe-2-is-the-fastest-most-accurate-and-cheapest-speech-recognition-model-in-the-world/)

当前Azure官方页列出60语言、词级时间戳、说话人分段、关键词偏置和逐字/清洁文本风格，并列出2与1.5，标注1在2026-08-20弃用。该接入页仍为public preview；特别说明约15分钟及更长录音开启分人可能失败，建议长录音关闭分人后另做处理。这是实际接口边界，不能只凭模型发布公告判断生产就绪。[Azure MAI文档](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/mai-transcribe?pivots=ai-foundry)

**路线判断：** 应把MAI-Transcribe-2列入托管文件转写候选，并把“有无分人、长短录音、逐字或清洁模式”作为不同配置验收。不能因为Azure Speech支持实时识别，就自动认定这里的MAI文件接口具有同样的增量流式能力。其研究价值在于检查“语义与结构能力增强”能否减少整个流水线和人工工作量。

### 3. ElevenLabs：Scribe家族应拆成三条路线

Scribe v2 的官方发布于2026-01-09，定位长、复杂录音的批量转写；Realtime面向低延迟交互，两者不是同一延迟设置。最新API概述还列出Medical，并分别规定能力与参数。[Scribe v2发布](https://elevenlabs.io/blog/introducing-scribe-v2)、[API能力概述](https://elevenlabs.io/docs/overview/capabilities/speech-to-text)

2026-09-22的公告将 Scribe v2 Medical 宣布为GA，模型ID为 `scribe_v2_medical`，运行于batch端点。厂商在Eka英文临床集上报告WER由8.6%降到7.0%：下降1.6个百分点，约18.6%相对下降；不能理解为提高18个百分点，也不能外推为中文医疗准确率。公告引用的Omi新榜单计划09-25更新，晚于本报告截止日，因此这里只视为厂商提前披露，未称已独立核对该榜单。[Medical GA公告](https://elevenlabs.io/blog/scribe-v2-medical-generally-available)

**路线判断：** 医疗版说明垂直领域适配仍有价值。可用同一医疗测试集比较通用版、通用版加术语、医疗版三种配置，观察收益来自模型适配还是词表；同时测药名、剂量与单位，并保留一般语音回归。对于实时产品则独立评估Scribe v2 Realtime，不将batch版的分人和长文能力直接移植过去。实际术语参数以API文档为准，当前batch与realtime限制不同，旧帮助页数字可能滞后。

### 4. 新路线不只发生在通用转写

**目标说话人识别。** Xiaomi-CocktailASR-1以参考说话人声音确定要识别的目标，包含非目标声音拒识。官方表还把剔除空输出后的WER与误拒率分开报告，因此不能直接拿其“非空WER”与其他系统完整WER比较。应联合评估目标文字、错转他人声音、错误拒识和参考样本质量。[小米官方仓库](https://github.com/xiaomi-research/xiaomi-cocktailasr-1)

**统一音频模型。** Audex同时涉及ASR、翻译、声音理解与生成，适合研究语音任务和LLM能力统一；本报告不把它当作低成本转写的默认替代。公开权重标注非商业许可，部署候选资格须先核对；思考模式、指令模式与各任务解码设置也不能混用。[NVIDIA模型卡](https://huggingface.co/nvidia/Nemotron-Labs-Audex-30B-A3B)

**轻量端侧。** ABR的Niagara与SDK代表小型本地流式路线，官方SDK文档包含 `niagara-38m-live.en` 示例。这里把它列为端侧候选线索，尚未对所有目标芯片、语言和发布包做实测或许可审查，不从“端侧”推导所有手机均可用。[ABR包说明](https://docs.appliedbrainresearch.com/sdk/concepts/application-packages/)、[ASR说明](https://docs.appliedbrainresearch.com/sdk/asr/overview/)

**相邻音频研究。** Step-Audio-R1.5涉及音频理解、推理与对话，应作为ASR外围路线跟踪；2026年9月的StepAudio 3 Gen和Music主要是生成任务，不能为了追新就列作新的ASR引擎。[R1.5论文](https://arxiv.org/abs/2604.25719)、[3 Gen论文](https://arxiv.org/abs/2609.12945)、[3 Music论文](https://arxiv.org/abs/2609.16034)

**综合判断：** 最近变化至少沿五个方向展开：原生流式与轮次控制，转写/时间/说话人联合输出，领域适配，目标说话人选择，音频理解与语言模型统一。选型应为每条相关路线放一个近期候选，再用稳定基线测收益；“模型新”代表值得验证，不代表可以跳过同条件评测。

## OpenMOSS 新近语音路线补查

核查日期：2026-09-23。范围是影响 ASR 路线判断的近期发布；官方功能声明不等于独立复现实测。

### 1. 必须进入主报告：MOSS-Transcribe-Diarize 0.9B

这比笼统补一个“MOSS-Audio”条目更重要：它直接面向长录音中的“谁在何时说了什么”。**0.9B 权重于 2026-07-09 发布**；官方模型卡列明 Apache-2.0、50 多种语言、最长 90 分钟单次输入、热词提示，并联合生成文本、时间戳和匿名说话人编号，也可输出声学事件。它适合会议、播客、访谈和字幕，是近期专用 ASR 系统候选。[官方模型卡](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize)

应区分论文和权重日期：技术报告初稿是 **2026-01-04**，截至核查日最新版为 **2026-07-17 的 v7**，不能将七月开源误写为论文首次提出。论文把任务称为 Speaker-Attributed, Time-Stamped Transcription，报告 128k 上下文及 90 分钟输入支持。[论文版本记录](https://arxiv.org/abs/2601.01554)

公开配置可交叉核实 Qwen3 文本模块、Whisper 类型音频模块、4 倍音频帧合并和 131072 最大位置数；这些配置支持其音频—语言联合结构的描述。具体训练初始化仍应以报告为准，不能仅凭配置断言全部权重来源。[模型配置](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize/blob/main/config.json)

**工程边界：**官方仓库已提供 SGLang Omni / vLLM 服务接入及字幕工具，长录音需足够的输出 token 预算，否则可能截断。Pro 仅在仓库中指向在线体验，不能将其效果和开源 0.9B 混写；未查得公开权重证据。现有文件上传和长音频能力也不足以证明低延迟、可持续会话的原生流式能力。[官方仓库与服务说明](https://github.com/OpenMOSS/MOSS-Transcribe-Diarize)

**路线意义（研究判断）：**把 ASR、说话人归属和时间戳联合建模，使轻量开源模型可以竞争原来需要多模块拼接的会议转写系统。应同时测文本误差、归属错误、重叠讲话、长录音后段漂移、时间边界和截断率。匿名编号只在单条输入内成立，不是实名声纹识别。官方自报 CER/cpCER 表可作筛选线索，不宜转述成跨场景冠军。[输出标签与评测定义](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize#output-format)

### 2. MOSS-Audio：通用音频理解，保留独立条目

**2026-04-13** 发布，**06-01** 公布技术报告。提供 4B/8B 的 Instruct、Thinking 四款；模型名对应语言骨干量级，包含音频模块的总参数约 **4.6B / 8.6B**。采用自研音频编码器、适配器和 Qwen3，连续音频表示为 12.5 Hz，并以跨层特征注入及时间标记保留声学、时间信息。它覆盖转写、词/句时间戳、环境声音、音乐、音频问答与推理；8B-Instruct 模型卡标注 Apache-2.0。[发布与架构](https://github.com/OpenMOSS/MOSS-Audio)、[参数和许可](https://huggingface.co/OpenMOSS-Team/MOSS-Audio-8B-Instruct)

路线意义是：当任务需要环境事件、音色线索或时间定位，纯文字转写可能丢失必要信息。但更大的通用音频模型并不自然优于专用 ASR；Thinking 也不应被默认用于逐字字幕。应把“听懂和回答”的准确率与“忠实转写”的错误率分开评估。

### 3. 相邻前沿：Speech 和 Tokenizer 不能当成 ASR 产品

**MOSS-Speech** 的论文首发于 **2025-10-01**，官方仓库标注 ICLR 2026，基于 Qwen3-8B，通过模态分层和冻结预训练实现不依赖文本引导的 speech-to-speech。仓库代码许可为 Apache-2.0，但其 TODO 仍列出 Base 模型开放与 Gradio 流式输出，不能据此宣称完整开放、成熟实时转写服务。它说明原生语音交互可以绕过显式文本中间层；需要审计文本的业务仍须额外设计可核验转写通路。[论文](https://arxiv.org/abs/2510.00499)、[仓库现状](https://github.com/OpenMOSS/MOSS-Speech)

**MOSS-Audio-Tokenizer** 是音频离散表示基础设施：初版 **2026-02-09** 发布，1.6B、24 kHz 单声道；**04-13** 增加约 20M 的 Nano；**06-07** 发布 **v2，2B、48 kHz 双声道**，支持 12.5 Hz 表示、32 层 RVQ 和分块编解码，v2 模型卡标注 Apache-2.0。它输出音频 codes / 重建音频，不直接输出转写；codec 的流式能力不等于上层 ASR 的流式能力。[版本时间线](https://github.com/OpenMOSS/MOSS-Audio-Tokenizer)、[v2 规格](https://huggingface.co/OpenMOSS-Team/MOSS-Audio-Tokenizer-v2)

因此主报告应按“联合转写与分离／通用音频理解／原生语音交互／音频表示基础设施”四层纳入 MOSS，而非把所有名字并成一个模型家族排名。

## 1. 不要把架构发展理解成一条淘汰链

ASR 至少有四个正交设计维度：**编码器如何表示声音、训练目标如何学习对齐、解码器如何利用语言知识、运行时能看多少未来音频**。Conformer 是编码器结构，CTC/RNN-T 是对齐与序列建模方式，Whisper 是完整模型家族，wav2vec 2.0 是预训练范式；把它们排成同层“模型排行榜”会误导技术决策。相同编码器可以接不同输出头；同一模型家族也可能同时有流式和离线权重。[Conformer 论文](https://arxiv.org/abs/2005.08100)、[NeMo 模型文档](https://docs.nvidia.com/nemo-framework/user-guide/26.02/nemotoolkit/asr/models.html)

### 从显式模块到端到端对齐

传统声学模型、发音词典与语言模型分工明确，便于注入领域词汇，但训练、搜索和维护链条复杂。端到端方法把更多模块联合训练，不等于词典、语言模型与外部知识从此没有价值；外部重打分仍可以提高识别结果。LAS 的早期工作就同时报告有无语言模型重打分的结果，说明“端到端”描述的是学习方式，而不是禁止系统组件。[LAS 原始论文](https://arxiv.org/abs/1508.01211)

| 方法 | 关键机制 | 工程含义与代价 |
|---|---|---|
| CTC | 对含 blank 的帧级路径求和，再折叠重复标签；无需逐帧人工对齐 | 输出路径有条件独立假设，解码相对简单；适合与外部语言模型或词表约束结合 |
| RNN-T / Transducer | 编码器处理声音，预测网络处理已输出符号，联合网络决定符号或 blank | 同时考虑声学与输出历史；使用因果/受限上下文编码器时适合增量识别 |
| Attention Encoder-Decoder | 解码器每一步关注音频表示，并条件于历史输出 | 擅长全句上下文；普通全局注意力配置需要额外设计才能在线运行 |
| 非自回归模型 | 预测长度或对齐后并行输出多个符号 | 降低逐 token 解码串行开销，但必须解决长度、对齐和词间依赖 |

表中机制分别来自 [CTC](https://www.cs.toronto.edu/~graves/icml_2006.pdf)、[RNN-T](https://arxiv.org/abs/1211.3711)、[在线 LAS 分析](https://arxiv.org/abs/2008.05514)、[Paraformer](https://arxiv.org/abs/2206.08317)。工程含义为机制推导，不表示任意 CTC 都天然流式，也不表示任意 attention 模型只能离线。

CTC 的“条件独立”不意味着编码器不看上下文：双向编码器可以先综合全句声音，然后在输出分布分解时采用独立假设。相反，RNN-T 名称中的 RNN 也不限制现代实现只能采用循环网络。选型时应检查**实际编码器的右侧上下文、缓存和解码策略**，不能仅看损失函数名字。[CTC 论文](https://www.cs.toronto.edu/~graves/icml_2006.pdf)、[NeMo 实现说明](https://docs.nvidia.com/nemo-framework/user-guide/26.02/nemotoolkit/asr/models.html)

### 编码器和解码效率仍是核心研究方向

Conformer 将局部卷积与全局自注意力结合，让语音的局部时频规律和长距离依赖同时得到建模。Zipformer 则通过不同时间分辨率等设计降低编码成本。两者的意义是更高效地提取声学证据；增加一个大型文本解码器并不能替代这个问题。[Conformer](https://arxiv.org/abs/2005.08100)、[Zipformer](https://arxiv.org/abs/2310.11230)

FastConformer 进一步优化下采样和注意力计算；Token-and-Duration Transducer（TDT）不仅预测符号，还预测推进时长，从而减少逐帧转移开销。推断：当业务是海量录音归档，改进编码与跳帧解码可能比增加参数更能改善单位小时成本，但实际收益取决于输入长度和批处理设置。[FastConformer](https://arxiv.org/abs/2305.05084)、[TDT](https://arxiv.org/abs/2304.06795)

## 2. 三种学习路线：它们解决不同的数据问题

**自监督预训练**利用无转写音频学习表示，再用标注语音微调。wav2vec 2.0 在潜在表示上掩码，通过量化目标的对比任务学习；HuBERT 先用聚类产生离散目标，再预测被遮挡的目标。二者都降低对人工转写量的依赖，但“无标注预训练”不等于无需下游标注，也不保证对任意业务领域泛化。[wav2vec 2.0](https://arxiv.org/abs/2006.11477)、[HuBERT](https://arxiv.org/abs/2106.07447)

**大规模弱监督**直接学习互联网音频与相应文字。Whisper 论文以 68 万小时多语言、多任务数据训练，重点展示跨数据集零样本泛化。其贡献不只是 Transformer 结构，而是训练数据规模与多样性改变了“每个领域先做大量微调”的使用模式。原论文数据量不能套用到后续所有版本。[Whisper 论文](https://arxiv.org/abs/2212.04356)

**多语言基础模型**把语言覆盖当成数据与迁移问题。Google USM 用大规模无标注多语言编码器预训练后微调；Meta MMS 将自监督与宗教文本朗读数据结合，分别报告了 1,107 语言 ASR 和更多语言的语言识别能力。关键边界：预训练语言数、可转写语言数和可识别语种数不同；低资源语言“有一个权重”也不等于自然会话、医疗或方言准确率已达商业要求。[USM](https://arxiv.org/abs/2303.01037)、[MMS](https://arxiv.org/abs/2305.13516)

数据策略的一个直接启示是：手中有大量目标场景录音但标注预算有限时，应考虑领域自监督适配或伪标签，而非只扩大通用模型。Robust wav2vec 2.0 的实验发现目标领域无标注数据能显著缩小域偏移差距；这是有设置边界的实验结论，并非承诺每项业务都能获得同等提升。[领域偏移研究](https://arxiv.org/abs/2104.01027)

SpecAugment 通过特征时间/频率遮挡等方式增强训练；这类方法说明，模型参数之外的数据增强仍能影响鲁棒性。工程推断：应分别验证远场混响、设备频响、压缩、口音与重叠语音，不能用单一“加噪训练”覆盖所有真实失配。[SpecAugment](https://arxiv.org/abs/1904.08779)

## 3. Speech-LLM：语言能力怎样帮助，又怎样越界

常见 Speech-LLM 由声音编码器、投影/适配器和文本 LLM 组成，先把音频表示映射到语言模型空间，再生成文字。Canary-Qwen 是可检查的例子：它组合 FastConformer、投影层和 Qwen3-1.7B，并采用 LoRA。其官方模型卡同时注明该权重主要用于英语 ASR，不能因为基座 LLM 会多语言，就假设语音识别也可靠支持这些语言。[Canary-Qwen 模型卡](https://huggingface.co/nvidia/canary-qwen-2.5b)

FireRedASR 同时提供 AED 和 Encoder-Adapter-LLM 两条路线，体现出准确率与效率的不同选择；大语言模型不是 ASR 的唯一终点。第一代仓库还明确列出输入长度边界和长输入重复/幻觉风险，因此“可以将一小时文件传给 API”与“模型一次处理一小时声音”是两回事。[FireRedASR 官方仓库](https://github.com/FireRedTeam/FireRedASR)

Qwen3-ASR 的技术报告把语音基础模型能力迁移到专门 ASR，并提供独立的非自回归强制对齐器。强制对齐的任务是给已知音频—文字对定位时间，不能验证文字确实正确。系统中应保留“转写正确性”与“时间定位正确性”两个独立指标。[Qwen3-ASR 技术报告](https://arxiv.org/abs/2601.21337)

研究判断：领域术语、人名和长程语境使语言先验有价值，但它也可能把含混声音补成语法通顺的错误。产品应明确选择逐字稿、去口吃稿或编辑稿；同一个 LLM 既做 ASR 又做润色时，不能仅凭文本更好读判断 ASR 更准确。验证应特别保留否定词、数字、罕见实体和不合常理但实际说出的句子，而不是只评估平均流畅度。此为从生成式解码与任务差异得出的设计建议。

## 4. 截至研究日值得纳入候选池的开放模型

下表是覆盖不同需求的候选清单，不是统一排名。参数量、语种数和能力应绑定具体 checkpoint；“支持”也不意味着每种语言同等可靠。

| 家族 / 具体版本 | 核实到的能力与适用方向 | 必须注意的边界 |
|---|---|---|
| Whisper large-v3 / turbo | 成熟多语言基线；turbo 为 809M、偏向更快转写 | 官方 `transcribe` 使用 30 秒滑窗；turbo 未针对翻译训练，不应把 `translate` 参数视作有效翻译能力。[仓库](https://github.com/openai/whisper) |
| Paraformer | 非自回归中文路线，FunASR 中有离线与流式相关模型 | 必须选对应在线权重与运行时，不能把“推理很快”叫做真正流式。[论文](https://arxiv.org/abs/2206.08317)、[工具库](https://github.com/modelscope/FunASR) |
| SenseVoiceSmall | 中文、粤语、英语、日语、韩语 ASR，以及情绪、事件标签 | 公开 Small 的五语种范围不同于研究中更大的语言覆盖；说话人分离需要外接流水线。[仓库](https://github.com/QwenAudio/SenseVoice) |
| Qwen3-ASR 0.6B / 1.7B | 30 种语言加 22 种中文方言；提供流式用法与语言识别 | 不应简称“52 种语言”；强制对齐另有权重。离线与流式成绩有差别，应独立测量。[仓库](https://github.com/QwenLM/Qwen3-ASR) |
| Fun-ASR-Nano / MLT-Nano | 两者均约 800M；前者中英日及中文方言，后者单独覆盖 31 种语言 | FunASR 是工具库，Fun-ASR 是模型家族；基础权重不直接输出说话人标签，时间戳还需检查分发渠道与修订版本。[仓库](https://github.com/QwenAudio/Fun-ASR) |
| GLM-ASR-Nano-2512 | 1.5B，普通话、英语、粤语及低音量场景是官方强调方向 | “17 种高可用语言”按作者 WER 阈值定义；不能据名称或平均分推出每一方言表现。[仓库](https://github.com/zai-org/GLM-ASR) |
| FireRedASR2-AED / LLM | 中文、英语、混说、方言与歌声；配套 VAD/LID/Punc | 整个 ASR2S 系统的 100+ 语种 LID/VAD 能力不等于 ASR 支持 100+ 语种；AED 有时间戳和置信分数。[仓库](https://github.com/FireRedTeam/FireRedASR2S) |
| Parakeet TDT 0.6B v3 | FastConformer-TDT，25 种欧洲语言，时间戳、标点，高吞吐方向 | 不覆盖中文；长音频上限与注意力模式及硬件绑定，不能将其吞吐成绩移植到笔记本。[模型卡](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3) |
| Canary-Qwen 2.5B | 语音编码器接 LLM 的英语识别路线 | 模型卡说明训练最长音频为 40 秒；通用语音理解能力并未因使用 Qwen 基座而自动保留。[模型卡](https://huggingface.co/nvidia/canary-qwen-2.5b) |
| VibeVoice-ASR / ASR-Streaming | 长音频的说话人、时间与内容联合输出；官方 2026-09-03 公布 10 语种 streaming 变体 | 长音频版本与流式版本分别评估；不可将家族里的 TTS 延迟、语言数或生成结构直接归给 ASR。[仓库](https://github.com/microsoft/VibeVoice) |

补充两条值得纳入比较的路线：Voxtral Mini 4B Realtime 2602 是专门流式权重，官方提供可调延迟并以 Apache 2.0 开放；不要将其与批量 Voxtral Mini Transcribe V2 混为一个模型，也不能把可配置的低延迟视为所有硬件上的端到端保证。[Mistral 官方发布](https://mistral.ai/news/voxtral-transcribe-2/)。IBM Granite-4.0-1B-Speech 支持英、法、德、西、葡、日语音输入及关键词偏置，采用 Apache 2.0；其英语到中文的翻译输出能力不意味着支持中文语音输入。[IBM 模型卡](https://huggingface.co/ibm-granite/granite-4.0-1b-speech)

许可会改变候选资格：Qwen3-ASR 技术报告说明模型以 Apache 2.0 发布，而 MMS-1B-all 模型卡标为 CC-BY-NC-4.0，含非商业限制。应分别检查代码、权重、适配器、数据和推理依赖，并保存实际使用版本的许可证；“可下载”与“可商用”不能互换。[Qwen3-ASR 报告](https://arxiv.org/abs/2601.21337)、[MMS 模型卡](https://huggingface.co/facebook/mms-1b-all)

仓库工具也影响路线选择：WeNet 的 U2 以统一流式/非流式训练和运行时为目标，适合把模型训练与生产部署一起研究；icefall 提供 CTC、Transducer、Zipformer 等训练配方。它们不是一个固定模型名，而是建立可重复训练和推理流程的入口。[WeNet 论文](https://arxiv.org/abs/2102.01547)、[icefall 官方仓库](https://github.com/k2-fsa/icefall)

## 5. 流式不是一个布尔字段

严格流式需要区分：音频分块能否增量编码、计算缓存能否复用、输出何时可暂时展示、何时不再回改、何时认定话轮结束。RNN-T 也可能延迟发射符号；FastEmit 专门通过训练目标促进更早发射，说明“可在线运行”和“用户看见字的延迟足够低”不是同一指标。[FastEmit](https://arxiv.org/abs/2010.11148)

建议分别记录音频右侧上下文、首个字延迟、稳定字延迟、最终结果延迟、回改率及声学证据到显示的 P95 延迟。分块跑离线模型可以形成准实时产品，但每个块重算、块边界漏词与错误的标点终止需要验证。模型宣称毫秒级 TTFT 时，必须检查是预先给全量音频后的生成首 token，还是声音实时到达时的真实延迟；高并发吞吐也不能代替单用户交互延迟。此为系统评估建议。

## 6. 如何读成绩，如何决定是否微调

厂商榜单最有价值的是暴露失效场景和候选差异，而非提供普适排名。中文 CER、英文 WER、标点归一化、繁简转换、数字 ITN、短句/长句比例、语音切分方式均会影响成绩。两张表即使出现同名数据集，也可能不同版本、不同归一化或不同推理设置。训练数据不完全公开的模型，还难以独立排除测试污染；应在主报告的数据与评测章节中单独讨论。

建议先做三个层次的对照实验：

1. **冻结模型、修复输入与系统**：采样率、声道、VAD 切分、解码长度、语言提示和热词；看错误究竟来自哪里。
2. **固定输出规范、加入业务词汇与少量领域适配**：用独立留出集检验热词误触发、通用能力退化和罕见实体召回。
3. **确有稳定领域差距再微调**：按说话人、设备、组织或时间划分数据，防止相似录音同时进入训练与测试；保留通用回归集。

以上是研究综合建议。是否从零训练取决于数据资产、目标语言、延迟/部署约束和长期维护能力；对于多数新业务，更有信息价值的第一步是建立真实场景盲测集，用成熟开放模型和 API 做统一比较。模型越强，剩余错误越集中于业务最在意的长尾，因而平均错误率下降不能直接替代业务验收。

## 7. 面向长期投入的路线判断

以下是基于机制与能力边界的研究判断，适合组织候选实验，不是未经实测的采购排名。

| 主要约束 | 应优先验证的路线 | 最容易忽略的代价 |
|---|---|---|
| 语音交互、车载、字幕首字延迟 | 受限上下文编码器配 Transducer，或明确支持增量缓存的专门流式模型 | 话轮结束、回改率和噪声误启动可能比离线 WER 更影响体验 |
| 数千万小时存量录音转写 | 高效 CTC/NAR/TDT，配批量调度 | 模型吞吐之外还有音频解码、VAD、小文件调度和存储成本 |
| 中文会议与复杂领域实体 | 中文专门模型与多语言基础模型并行盲测，比较热词及领域适配 | 会议重叠和说话人归属不是普通 CER 能反映的能力 |
| 大量陌生语言、标注稀缺 | 多语言基础模型、自监督迁移与当地人工评测 | 语言覆盖数容易掩盖方言、脚本、文化语境和语料来源偏差 |
| 一小时访谈的结构化记录 | 联合长音频模型与模块化 ASR＋对齐＋说话人流水线对照 | 联合模型省接口，但失败定位与局部重跑可能更困难 |
| 端侧隐私与离线运行 | 小编码器、量化、可维护运行时；在目标设备上验证 | 模型文件大小不是峰值内存；持续录音还受耗电、散热和系统抢占影响 |

长期来看，最有复用价值的资产通常是能回放的错误案例、严格定义的输出规范、覆盖真实设备与人群的评测集，以及可替换模型的接口。架构变化可以带来一次准确率或效率跃升，这些资产则能帮助团队判断下一次跃升是否对业务有效。建议把模型调用封装为音频输入、可修订文本事件、最终文本、时间戳及可选说话人字段，而不要把某家 SDK 的单一响应结构扩散到全部业务代码。这是工程设计建议。

## 8. 当前证据的局限与值得跟踪的研究问题

- 本报告核实能力声明和方法来源，没有下载权重运行，也没有证明任何候选在特定业务里最优。动态 README 可能变化，正式实验应锁定权重 revision、软件版本、解码配置与测试集散列。
- 长音频联合转写和说话人归属开始成为模型级任务，但应继续分别检查说话人漂移、重叠说话、时间戳误差和漏段，不能被漂亮的结构化输出掩盖。
- 继续研究的高价值方向包括：真实低延迟与低回改率的共同优化、低资源方言与混说、忠实转写与语言推断的边界、无需大规模人工标签的领域适配，以及能够拒绝无声/不可辨声音的校准机制。
- 模型开放应进一步拆成代码、权重、训练配方、训练数据和许可；仅能下载权重不足以意味着训练全过程可复现。许可证及商业使用条件应在部署决策时针对准确 artifact 单独核查。
