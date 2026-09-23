# ASR 开源评测框架调查

核查日期：2026-09-23。范围：统一接入多个模型、实验不同推理/音频处理/文本处理方法；仅引用官方仓库、代码与文档。本次为源码与文档调查，未安装运行模型。下文“缺口”表示本次核查没有看到现成实现，不等于无法自行扩展。

## 核心判断

确实有可复用框架，但不存在一个已核实能原生覆盖所有本地模型、所有云 API、各种 beam search/VAD/降噪/LLM 纠错/流式指标的共同标准。应把“实验调度与模型适配”“方法本身”“评分”分层。

按用户需求的契合程度，优先试 **SibNN/asr_eval**（pipeline 方法实验与流式分析）、**UltraEval-Audio**（中文/英文专用 ASR 与 Audio LLM、本地/API）、**Open ASR Leaderboard 的评测代码**（对齐公共榜单）。若重点是音频增强与退化实验，加看 **ASR.lab**；若重点是云厂商与流式延迟，加看 **Picovoice speech-to-text-benchmark**。以下推荐是基于所列功能的研究判断，不是实际跑测结论。

## 真正的评测运行框架

### 1. SibNN/asr_eval：最贴近“模型 × 不同处理手段”的研究工作台

- `asr_eval.bench` 把一个固定模型配置命名为 pipeline，逐样本保存预测；支持自定义音频操作（混响、背景/麦克风噪声）、文本 normalization/tokenization，并把推理与后续评分/可视化分离。保留原始预测及输入输出 chunks，适合反复改评分规则而不重复跑模型。[官方评测指南](https://sibnn.github.io/asr_eval/guide_evaluation_dashboard.html)
- 已注册本地 Whisper、Wav2Vec2、NeMo、SpeechBrain、Vosk、GigaAM 等；实际 API 注册代码是 Yandex SpeechKit、Salute，不能据此声称已覆盖 OpenAI/Azure/国内所有厂商。接口位置：`asr_eval/bench/pipelines/_registry.py` 与各 `_registered/*.py`。[注册目录](https://github.com/SibNN/asr_eval/tree/main/asr_eval/bench/pipelines/_registered)、[API 注册代码](https://github.com/SibNN/asr_eval/blob/main/asr_eval/bench/pipelines/_registered/api.py)
- 有真实“方法组合”例子：GigaAM CTC + LM 解码 + Whisper + 俄语词频 comparator；这比单纯换模型更接近用户想要的消融实验，但该例有明确俄语特性。[composite.py](https://github.com/SibNN/asr_eval/blob/main/asr_eval/bench/pipelines/_registered/composite.py)
- 包含多参考转写与流式评价工具；dashboard 可单独读取 predictions/annotations CSV。文档将 bench 称为 experimental framework，应先做小样本集成验证；中文 normalization 和未内置 API 需补适配。[官方仓库](https://github.com/SibNN/asr_eval)
- 许可证不能简单写“标准 MIT”：LICENSE 标为 MIT，但增加保留公司 URL、专利授权/终止条款，GitHub SPDX 识别为 `NOASSERTION`。[LICENSE](https://github.com/SibNN/asr_eval/blob/main/LICENSE)

### 2. UltraEval-Audio：较完整的多模型、多任务、本地/API harness

- 统一命令 `python audio_evals/main.py --dataset <name> --model <name> --prompt <name>`；包括专用 ASR 与 Audio LLM，中文/英文数据，支持断点续跑、重试、仅从已有 inference 文件评分，以及模型隔离运行环境。仓库已有 Qwen3-ASR 与 Fun-ASR-Nano 复现说明。[README](https://github.com/OpenBMB/UltraEval-Audio)、[replication](https://github.com/OpenBMB/UltraEval-Audio/tree/main/replication)
- 自定义云服务继承 `APIModel`、本地模型继承 `OfflineModel`，实现 `_inference(PromptStruct, **kwargs)`，在 `registry/model/` YAML 注册；可以把完整“预处理→ASR→后处理”封进模型适配器。[模型接入指南](https://github.com/OpenBMB/UltraEval-Audio/blob/main/docs/how%20eval%20your%20model.md)
- prompt、模型/数据集注册和输出分离适合测试 prompting、推理参数和纠错。不能把它已支持多任务推断成原生支持通用 ASR beam/VAD 网格搜索或端点延迟测量；这些仍需明确写 adapter/任务。Apache-2.0。[代码/许可证](https://github.com/OpenBMB/UltraEval-Audio/blob/main/LICENSE)

### 3. Hugging Face Open ASR Leaderboard：最适合复现公共榜单协议

- 不只有网页：按模型家族提供 `run_eval.py`、运行脚本和依赖环境，输出 predictions JSONL、WER、RTFx；2026-07-24 版说明将英文/多语言短音频评测搬到 HF Jobs，通过各家族 Docker 镜像统一硬件条件。长音频当时尚待迁移。[运行说明](https://github.com/huggingface/open_asr_leaderboard)
- 已有云 API 路径 `api/run_api.sh`、多语种 `api/run_api_ml.sh`，可按 model/dataset/language 选择；无需 GPU，通过本地 Docker 调 API。[API README](https://github.com/huggingface/open_asr_leaderboard/blob/main/api/README.md)
- 最强的是数据、normalizer、报告协议与公开结果的对应关系；不是一个任意节点组成 pipeline 的插件系统。新方法需要修改/新增家族脚本，完整实验追踪及流式端点指标需另补。代码 Apache-2.0；模型、数据和 API 使用条件另计。[LICENSE](https://github.com/huggingface/open_asr_leaderboard/blob/main/LICENSE)

### 4. ASRBench：轻量 YAML 多配置比较，有清晰扩展接口

- `ConfigLoader(...).set_up_benchmark().run()`；CLI 是独立的 `asrbench-cli run config.yml`。同一 YAML 定义数据集、多个命名 transcriber 及输出；示例已经能设 faster-whisper model/device/compute_type/beam_size/language。[框架](https://github.com/ASRBench/asrbench)、[CLI](https://github.com/ASRBench/asrbench-cli)、[文档](https://asrbench.github.io/asrbench/)
- CLI 内置文件为 faster-whisper、Whisper、Vosk、Wav2Vec、HF provider；注意名为 `hf_provider.py` 的实现实际加载本地 `AutoModelForCTC`，不是 Hugging Face 云推理 API。新 provider 继承 `Transcriber` 并注册，实现 load/unload/transcribe 等。[适配器目录](https://github.com/ASRBench/asrbench-cli/tree/main/asrbench_cli/transcribers)、[HF 实现](https://github.com/ASRBench/asrbench-cli/blob/main/asrbench_cli/transcribers/hf_provider.py)
- 适合规模不大的离线参数对比；本次未发现现成流式评价、广泛云 API adapters、任意音频处理图。Python 3.12+，MIT。最后默认分支提交为 2025-06-16，近期维护证据弱于上述项目；不能因文档可访问就判断活跃。

### 5. Picovoice speech-to-text-benchmark：多云厂商 + 本地 + 流式延迟

- 本地 Whisper、whisper.cpp、Vosk、Moonshine 与 Amazon/Azure/Google/IBM 等云服务共用入口；`benchmark.py` 评估 WER/PER、CPU core-hour 等；`benchmark_latency.py` 评估 streaming word emission latency。此延迟定义是单词说完到发出转写的平均时间，需准备词级时间对齐数据。[README](https://github.com/Picovoice/speech-to-text-benchmark)、[延迟实现](https://github.com/Picovoice/speech-to-text-benchmark/blob/master/benchmark_latency.py)
- 扩展入口是 `engine.py`、`dataset.py`、`normalizer.py`，代码简单易审查。当前文档语言枚举主要为英语及欧洲语言，没有中文；core-hour 不适合当 GPU RTF 或 API 成本。降噪、VAD、热词、LLM 纠错实验需要外层 runner 或 adapter。[engine.py](https://github.com/Picovoice/speech-to-text-benchmark/blob/master/engine.py)
- Apache-2.0。由同时销售 ASR 产品的厂商维护；可以复用公开代码，选型结论最好用自己的冻结数据和配置重跑。

### 6. ASR.lab：声学处理消融最直接，但目前主要本地模型

- 声明并给出配置的执行链为原音频→VST3 degradation→enhancement→LUFS normalization→ASR→metrics，自动组合处理维度；包括 Whisper、Wav2Vec2、NeMo、Vosk、SeamlessM4T、Moonshine、SenseVoice；提供 raw/normalized 文本两套分数与 HTML 对比报告。[README/config 示例](https://github.com/berangerthomas/ASR.lab)
- 入口 `python main.py run -c configs/default.yaml`；数据 manifest 为 `audio_filepath/text/lang`。适合噪声/混响、Demucs/DeepFilterNet、响度实验，不应宣称覆盖全部 API。2026-03-21 最近提交明确为“Remove legacy cloud engines, reports & metrics”。代码 MIT。[提交证据](https://github.com/berangerthomas/ASR.lab/commit/f2686d76340af996a8e8a2b267ceaa9a7216b301)
- 本次仅核查 README/配置与仓库状态，未验证它宣称的各模型运行成功与各平台兼容性。

### 7. SpeechColab / SpeechIO Leaderboard：中文数据与容器化接口值得复用

- 提供 dataset zoo、local/API model zoo、准备数据→识别→后处理→错误率的统一 pipeline。新系统打成带 Dockerfile、assets、model.yaml、SBI 的 model-image；`./SBI <audio_list> <result_dir>` 写 `raw_rec.txt`，`ops/benchmark -m <model> -d MINI_ZH` 验证。[仓库](https://github.com/SpeechColab/Leaderboard)、[接入规范](https://github.com/SpeechColab/Leaderboard/blob/master/HOW_TO_SUBMIT.md)
- 云 API、本地引擎和不同解码方案都能装进 SBI，但这并不自动提供实验参数搜索。规范输入为 16k/16bit/mono、单条小于 30 秒，不能直接当 long-form/streaming 基准。locked 中文集不能自行完整复现。
- GitHub 仓库元数据未识别许可证，根目录本次未核实到统一 LICENSE；公开可见不等于有明确再分发授权，不能在严格开源清单里直接标 MIT/Apache。[仓库元数据](https://api.github.com/repos/SpeechColab/Leaderboard)

### 8. AU-Harness / lmms-eval：Audio LLM 评测扩展项

- **AU-Harness**：`config.yaml` 定义 dataset-metric、多个模型 endpoints、prompt/temperature/request 参数，`bash evaluate.sh` 执行；有 preprocessor/postprocessor、长音频与 code-switching ASR 任务。适合把语音理解、QA、音频大模型转写一起测。其多模型能力不代表任意传统 ASR 引擎已经接好；低层 beam/VAD 和流式性能仍需验证/扩展。Apache-2.0。[README](https://github.com/ServiceNow/AU-Harness)、[ASR tasks](https://github.com/ServiceNow/AU-Harness/tree/main/tasks/speech_recognition)
- **lmms-eval**：更广泛的多模态模型/任务框架，适合已有多模态评测工程时加入 audio/ASR；不是优先面向传统 ASR serving latency 或降噪网格实验的专门工具。许可证分层：main pipeline MIT，新增 `tasks`/`models` Apache-2.0，不能简化成全仓单一许可证。[官方仓库](https://github.com/EvolvingLMMs-Lab/lmms-eval)、[LICENSE](https://github.com/EvolvingLMMs-Lab/lmms-eval/blob/main/LICENSE)

## 评分层：可共用，但不要叫它们完整模型评测框架

| 工具 | 已核实可复用部分 | 边界 |
|---|---|---|
| [JiWER](https://github.com/jitsi/jiwer) | WER/CER/MER/WIL/WIP、文本 transformation、逐句/对齐分析；4.x 定义空参考场景，适合静音 hallucination 计分 | 接收 reference/hypothesis，不执行模型、不测流式端点。Apache-2.0 |
| [MeetEval](https://github.com/fgnt/meeteval) | 会议转写 cpWER、ORC-WER、MIMO-WER 及 time-constrained 指标；有时间/说话人格式工具和可视化 | 专门评分多说话人/长会议；不是模型适配或实验调度器。MIT |
| [belambert/asr-evaluation](https://github.com/belambert/asr-evaluation) | `wer ref hyp`、WER/识别率/SER、confusion/错句分析、Kaldi/Sphinx id 格式 | 轻量历史评分 CLI；无模型运行层，最后默认分支提交 2021-04-14。Apache-2.0 |
| [HF Evaluate](https://huggingface.co/docs/evaluate/v0.4.6/en/package_reference/evaluator_classes) | `evaluator('automatic-speech-recognition').compute(...)` 接 model/pipeline/callable、data、metric、generation_kwargs，支持 bootstrap | 比单一 scorer 多一层执行；假设 Transformers ASR Pipeline 兼容输入输出，API/多步骤方法仍需包装；不是云 API/流式一站式平台。Apache-2.0 |

## 训练工具包的正确定位

- **NeMo** 是需要深入测试 CTC/RNNT/TDT decoding、streaming look-ahead 的模型生态工具。`examples/asr/speech_to_text_eval.py` 接 JSONL `audio_filepath/text`，继承 transcribe 配置；支持 `use_cer`、标点错误率、逐样本分数与 `only_score_manifest=True`；代码中有 `decoder_type`、`att_context_size`。这很适合 NeMo 内部消融，但统一比较别家的本地/API 引擎应再放一层 harness。[实际入口](https://github.com/NVIDIA/NeMo/blob/main/examples/asr/speech_to_text_eval.py)
- **ESPnet** 提供 recipe、`asr_inference.py`、模型/LM/解码参数，以及增强→ASR 评分链；可复用研究解码/增强 recipe，但不能等同 provider-neutral API benchmark。其文档也有 Whisper 推理评分脚本。[ASR CLI](https://espnet.github.io/espnet/tools/espnet2_bin/asr_inference.html)、[enhancement/ASR scoring](https://espnet.github.io/espnet/recipe/enh1.html)、[教程](https://github.com/espnet/espnet/blob/master/doc/espnet2_tutorial.md)
- **SpeechBrain** 的模型 recipes 和 `ErrorRateStats` 可保存 substitutions/deletions/insertions、对齐统计、weighted error rate。适合已有 SpeechBrain 项目与模型训练后评价，不是现成多云 API/方法矩阵平台。[官方 metrics API](https://speechbrain.readthedocs.io/en/stable/API/speechbrain.utils.metric_stats.html)
- 三者代码许可证均 Apache-2.0；不必为了算 WER 而安装整套训练依赖。

## 维护证据快照

以下为本次直接查询 GitHub REST API `repos/<owner>/<repo>/commits?per_page=1` 返回的默认分支最新提交日期，不是发布日期，也不证明所有 adapters 仍有效。均未标 archived。默认分支会变化，下面永久 commit 链接锁定本次证据。

| 项目 | 最近提交 UTC | 固定证据 |
|---|---|---|
| Open ASR Leaderboard | 2026-09-21 | [6fec425](https://github.com/huggingface/open_asr_leaderboard/commit/6fec425b243962c0e7fd0b36b9b5841aef676c0a) |
| UltraEval-Audio | 2026-09-16 | [bead726](https://github.com/OpenBMB/UltraEval-Audio/commit/bead726925d43f526bed48a4a6a827595169429d) |
| SibNN/asr_eval | 2026-03-09 | [b2f15a5](https://github.com/SibNN/asr_eval/commit/b2f15a5f2e04242d8650c02b9ee0f504bbe10ada) |
| ASRBench | 2025-06-16 | [74dcd2f](https://github.com/ASRBench/asrbench/commit/74dcd2fa1e4e93c05edf6a16977ef81226b34dd0) |
| ASRBench CLI | 2025-06-16 | [51fb806](https://github.com/ASRBench/asrbench-cli/commit/51fb8063276b72b60d8680efd42c2c228ac8e818) |
| Picovoice benchmark | 2026-03-19 | [43e7689](https://github.com/Picovoice/speech-to-text-benchmark/commit/43e7689f013694e9ba9910695643af9c579ca8d2) |
| ASR.lab | 2026-03-21 | [f2686d7](https://github.com/berangerthomas/ASR.lab/commit/f2686d76340af996a8e8a2b267ceaa9a7216b301) |
| SpeechColab | 2025-03-29 | [678f55a](https://github.com/SpeechColab/Leaderboard/commit/678f55a708bb759ec3d0dacb5eefe4a57b9d05d4) |
| AU-Harness | 2026-06-09 | [e07548b](https://github.com/ServiceNow/AU-Harness/commit/e07548bafd94ef816c9108c3983fc2f3c2356d55) |
| lmms-eval | 2026-09-22 | [14b9c07](https://github.com/EvolvingLMMs-Lab/lmms-eval/commit/14b9c079873445a039a3eb822d5b30fa595d0f9b) |
| JiWER | 2026-04-16 | [2227002](https://github.com/jitsi/jiwer/commit/2227002889f20f3fb19d523eeb322732cb67b431) |
| MeetEval | 2026-08-11 | [6e3dc81](https://github.com/fgnt/meeteval/commit/6e3dc81284f2d6928f7ef9e620fd3b6906daa429) |
| belambert/asr-evaluation | 2021-04-14 | [1f55986](https://github.com/belambert/asr-evaluation/commit/1f5598601ed4064f3518a4b1f1b9f9bcfd9a307b) |
| SpeechBrain | 2026-08-27 | [89ead74](https://github.com/speechbrain/speechbrain/commit/89ead74d163463d30c62329a09cfdb4c54f5abc1) |
| ESPnet | 2026-09-22 | [1a753b4](https://github.com/espnet/espnet/commit/1a753b4905932f50e7ea38012e1a289f2c3ed5d8) |
| NeMo | 2026-09-22 | [5dbdde6](https://github.com/NVIDIA/NeMo/commit/5dbdde68d3897c03faeda1b0d9655cd90b1c5e08) |

## 适合下一步验证的小实验

建议选择一个主要 harness、一个固定 scorer，不同时维护数套评分规则。冻结 30–100 条自有中文/中英混说样本，接一个本地模型和一个 API，分别验证 baseline、VAD/chunking、一种降噪、热词/提示、LLM 纠错；每个配置独立 pipeline ID。保留 raw hypothesis、processed hypothesis、reference、时间、失败/重试和配置 hash。音频/解码/后处理变化才需重跑对应阶段，纯 normalization 变化应从缓存重评分。这个方案为本研究的工程建议，不是任何框架已经一键实现的保证。
