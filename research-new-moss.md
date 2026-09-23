# OpenMOSS 新近语音路线补查

核查日期：2026-09-23。范围是影响 ASR 路线判断的近期发布；官方功能声明不等于独立复现实测。

## 1. 必须进入主报告：MOSS-Transcribe-Diarize 0.9B

这比笼统补一个“MOSS-Audio”条目更重要：它直接面向长录音中的“谁在何时说了什么”。**0.9B 权重于 2026-07-09 发布**；官方模型卡列明 Apache-2.0、50 多种语言、最长 90 分钟单次输入、热词提示，并联合生成文本、时间戳和匿名说话人编号，也可输出声学事件。它适合会议、播客、访谈和字幕，是近期专用 ASR 系统候选。[官方模型卡](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize)

应区分论文和权重日期：技术报告初稿是 **2026-01-04**，截至核查日最新版为 **2026-07-17 的 v7**，不能将七月开源误写为论文首次提出。论文把任务称为 Speaker-Attributed, Time-Stamped Transcription，报告 128k 上下文及 90 分钟输入支持。[论文版本记录](https://arxiv.org/abs/2601.01554)

公开配置可交叉核实 Qwen3 文本模块、Whisper 类型音频模块、4 倍音频帧合并和 131072 最大位置数；这些配置支持其音频—语言联合结构的描述。具体训练初始化仍应以报告为准，不能仅凭配置断言全部权重来源。[模型配置](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize/blob/main/config.json)

**工程边界：**官方仓库已提供 SGLang Omni / vLLM 服务接入及字幕工具，长录音需足够的输出 token 预算，否则可能截断。Pro 仅在仓库中指向在线体验，不能将其效果和开源 0.9B 混写；未查得公开权重证据。现有文件上传和长音频能力也不足以证明低延迟、可持续会话的原生流式能力。[官方仓库与服务说明](https://github.com/OpenMOSS/MOSS-Transcribe-Diarize)

**路线意义（研究判断）：**把 ASR、说话人归属和时间戳联合建模，使轻量开源模型可以竞争原来需要多模块拼接的会议转写系统。应同时测文本误差、归属错误、重叠讲话、长录音后段漂移、时间边界和截断率。匿名编号只在单条输入内成立，不是实名声纹识别。官方自报 CER/cpCER 表可作筛选线索，不宜转述成跨场景冠军。[输出标签与评测定义](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize#output-format)

## 2. MOSS-Audio：通用音频理解，保留独立条目

**2026-04-13** 发布，**06-01** 公布技术报告。提供 4B/8B 的 Instruct、Thinking 四款；模型名对应语言骨干量级，包含音频模块的总参数约 **4.6B / 8.6B**。采用自研音频编码器、适配器和 Qwen3，连续音频表示为 12.5 Hz，并以跨层特征注入及时间标记保留声学、时间信息。它覆盖转写、词/句时间戳、环境声音、音乐、音频问答与推理；8B-Instruct 模型卡标注 Apache-2.0。[发布与架构](https://github.com/OpenMOSS/MOSS-Audio)、[参数和许可](https://huggingface.co/OpenMOSS-Team/MOSS-Audio-8B-Instruct)

路线意义是：当任务需要环境事件、音色线索或时间定位，纯文字转写可能丢失必要信息。但更大的通用音频模型并不自然优于专用 ASR；Thinking 也不应被默认用于逐字字幕。应把“听懂和回答”的准确率与“忠实转写”的错误率分开评估。

## 3. 相邻前沿：Speech 和 Tokenizer 不能当成 ASR 产品

**MOSS-Speech** 的论文首发于 **2025-10-01**，官方仓库标注 ICLR 2026，基于 Qwen3-8B，通过模态分层和冻结预训练实现不依赖文本引导的 speech-to-speech。仓库代码许可为 Apache-2.0，但其 TODO 仍列出 Base 模型开放与 Gradio 流式输出，不能据此宣称完整开放、成熟实时转写服务。它说明原生语音交互可以绕过显式文本中间层；需要审计文本的业务仍须额外设计可核验转写通路。[论文](https://arxiv.org/abs/2510.00499)、[仓库现状](https://github.com/OpenMOSS/MOSS-Speech)

**MOSS-Audio-Tokenizer** 是音频离散表示基础设施：初版 **2026-02-09** 发布，1.6B、24 kHz 单声道；**04-13** 增加约 20M 的 Nano；**06-07** 发布 **v2，2B、48 kHz 双声道**，支持 12.5 Hz 表示、32 层 RVQ 和分块编解码，v2 模型卡标注 Apache-2.0。它输出音频 codes / 重建音频，不直接输出转写；codec 的流式能力不等于上层 ASR 的流式能力。[版本时间线](https://github.com/OpenMOSS/MOSS-Audio-Tokenizer)、[v2 规格](https://huggingface.co/OpenMOSS-Team/MOSS-Audio-Tokenizer-v2)

因此主报告应按“联合转写与分离／通用音频理解／原生语音交互／音频表示基础设施”四层纳入 MOSS，而非把所有名字并成一个模型家族排名。
