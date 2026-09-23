# 接入伪代码：ElevenLabs、MOSS 与 Whisper

2026-09-23。本文件将[总体设计](ASR-framework-design.md)和[数据抽象](ASR-framework-data-design.md)落到调用层，检查不同方法是否真的能在同一 framework 下评测。

**这里只写设计伪代码，不是可运行 SDK 示例。** `InputView`、`MethodDescriptor`、`run`、`artifacts`、`registry`、`wire` 等都是拟议接口，当前仓库没有其实现。标注为“已核查”的 HTTP 路径、参数或库行为来自文末官方来源；部署版本、响应 fixtures、异常/关闭行为还需实现时验证。没有调用付费推理、启动服务器或下载权重。

## 1. 先给出适配结论

三个模型的离线转写可以共用一个 whole-case Interface。API、自部署 HTTP、本地库是执行位置/传输方式，不能代替模型输出语义。实时音频输入采用另一份 incremental Interface；不要强迫所有模型实现假的流式。

| 方法配置 | 执行路径 | 可映射的结果 | 接入工作主要在哪里 |
|---|---|---|---|
| ElevenLabs Scribe 文件转写 | HTTPS multipart | 转写、可选词时间/说话人归属 | 参数与输入权限、HTTP 错误、原响应投影 |
| MOSS + SGLang Omni | 自部署 HTTP 转写 | 联合文字、段时间、匿名 speaker | 固定后端版本/格式、SATS 解析与时间粒度 |
| MOSS + vLLM 或本地 Transformers | HTTP 或进程内推理 | 同一类联合结果，响应表示可能不同 | 替换 transport/driver 和对应 parser，不改 scorer |
| Whisper + faster-whisper | 本地模型库 | 文字、段时间、可选词时间 | 模型生命周期、生成器完整执行、配置固定 |
| ElevenLabs Scribe Realtime | WebSocket 双向流 | partial、段提交、可选迟到的时间信息 | session、修订映射、并发收发、有界收尾 |
| Whisper + 滚动窗口与确认策略 | 组合方法 | 通过窗口方法生成的在线修订 | 单独 recipe、真实供音、去重/提交、重复计算成本 |

下面按“同一个调用者，换不同 adapter”展示。Whisper 本身没有在本例中提供 speaker 输出；它可以参加普通文字任务，不能自动参加要求多人归属输出的任务。

## 2. 调用者只描述数据、任务和方法

```python
# 全部是伪代码；rev_* / checkpoint_* 必须替换为核实后冻结的身份。
dataset = datasets.freeze(
    source=LocalManifest("data/manifest.csv"),
    mapping=Columns(audio="audio_path", reference="reference_text"),
    snapshot="dataset_rev_1",
)

cases = dataset.view(
    split="test",
    unit="recording",                 # 不让各模型各自筛样本
    channel_policy="fixed_track_0",   # 本例所有方法拿同一轨
).freeze()

task = ProtocolSpec(
    mode="whole_case",
    goal="literal_transcription",
    allowed_inputs={"audio"},          # 默认不给金标语言/人数/边界
    output_view="provider_rendered",   # 不声称所有模型返回 spoken 双形式
    required_outputs={"transcript"},
    deadline="frozen_per_case_policy",
    failure_policy="score_valid_output_at_cutoff_else_empty",
)

methods = [
    MethodSpec("elevenlabs.file", model="scribe_v2",
               options={"diarize": False, "no_verbatim": False}),
    MethodSpec("moss.file", checkpoint="checkpoint_moss",
               backend="sglang_omni_rev_pinned",
               options={"max_new_tokens": 32768}),
    MethodSpec("whisper.file", checkpoint="checkpoint_whisper",
               backend="faster_whisper_rev_pinned",
               options={"beam_size": 5, "vad_filter": False,
                        "word_timestamps": False}),
]

plan = compile_plan(cases, task, methods, execution="quality_baseline")
run_index = execute(plan)

# 独立评分进程：这里才允许读取 reference。
scores = score(
    run_index,
    references=dataset.references_for(cases),
    spec=ScoreSpec("cer_zh_v1", normalizer="frozen_zh_policy",
                   aggregate="sum_errors_over_sum_reference_units"),
)
report = compare(scores, on="same_protocol_and_case_ids")
```

只有 `MethodSpec` 和 adapter 注册项不同。评分器不需要 `if method == "moss"`。模型的数字/标点与事件输出先由确定的解析规则投影成可评分文字，评分 normalizer 再按共同协议处理；两者有各自版本，均不允许读 gold 帮忙清洗模型回答。

配置解析时补齐并保存所有 adapter 使用的默认值，例如时间戳粒度、Whisper 的历史文字条件。`scribe_v2` 是服务模型标识，不保证是永不变化的权重快照；记录调用日期、可得服务版本、请求配置和响应原件，服务无法固定的行为标 unknown，不宣称将来重调用逐字相同。

质量比较可以采用相同输入；性能则需另冻结硬件、负载、网络和计时范围。不能把 API 客户端耗时与本地 GPU kernel 耗时放进同一列。

## 3. 最小公共 Interface 与预检

### 3.1 方法、结果与执行上下文

```python
record MethodDescriptor:
    id, adapter_revision, parameter_schema
    execution_profile                  # whole_case 或 incremental
    requires_inputs                    # 资产类型/授权/格式要求
    outputs_for(resolved_config)        # 配置相关保证，含时间粒度
    state_scope                        # none / case / episode
    limits                             # 已核实的格式/长度/并发条件

record ResultBundle:
    transcript                         # provider_rendered，原始词序保留
    alignment = Missing("not_provided")
    attribution = Missing("not_provided")
    extensions = []                    # 显式 type/version 的产物
    raw_evidence_refs = []
    completion = Unknown               # 不把 HTTP 200 当全文覆盖证明

interface WholeCaseMethod:
    transcribe(input: InputView, run: AttemptContext) -> ResultBundle

# registry factory 在 worker 初始化时注入 driver / client / 配置。
# load/unload 属于 worker 生命周期，transcribe 不是每条重新加载权重。
interface IncrementalMethod:
    open(session_input, run) -> Session

interface Session:
    send(input_event)                   # 媒体块、允许的上下文、InputEnd
    receive() -> stream[ResultEvent]    # 同时运行，不能等 send 完才开始收
    close()                            # 有界 drain 后释放连接/状态
```

`AttemptContext` 只提供截止/取消、受控媒体读取、原始证据记录、用量与受观察调用，不提供 reference、评分结果、任意 dataset 行或其他会话状态。凭证在 worker 初始化时按引用注入，输出的请求记录不含密钥/header。

`run` 不是万能容器：它不替方法实现 beam、VAD、拼接或 speaker 聚类。runner 的外层计时覆盖完整调用与解析；内层 span 只帮助归因，不替代端到端耗时。结果产物由 runner 校验后统一登记依赖。

### 3.2 运行之前拒绝不成立的组合

```python
def compile_plan(cases, task, methods, execution):
    for spec in methods:
        descriptor = registry.describe(spec.adapter_id)
        resolved = resolve_and_validate(spec, descriptor.parameter_schema)
        offered = descriptor.outputs_for(resolved)

        require(task.mode == descriptor.execution_profile)
        require(task.required_outputs <= offered.guarantees)
        for case in cases:
            require_inputs(case, descriptor.requires_inputs, task.allowed_inputs)
            require_method_limits(case, resolved, descriptor.limits)
            require_reference_coverage(case, task.score_requirements)

    return freeze_plan(cases, task, resolved_methods, execution)

# 非运行结果，只是预期：
compile(WhisperBase, MeetingTask(required={"transcript", "attribution"}))
# -> UnsupportedBeforeRun("whisper.file 未承诺 attribution")

compile(MossFile, RealtimeTask())
# -> UnsupportedBeforeRun("whole_case 不能履行 incremental 生命周期")

compile(ElevenLabsFile, Recipe(requires="candidate_scoring"))
# -> UnsupportedBeforeRun("该 adapter 没有指定候选评分能力")
```

未知能力不能默认为 false 然后偷偷降级；需要接入者核实或明确拒绝。可选能力声明缺失与已承诺能力在运行中没返回不同，后者是 invalid_output/字段覆盖问题。

## 4. 同一个 whole-case runner

```python
def execute_whole_case(plan):
    for method in plan.methods:
        with workers.prepare(method) as adapter:
            # 初始化时间单独保存；冷启动任务按协议计入总时长。
            for case in plan.case_manifest:
                chosen = None
                for attempt_no in plan.retry_policy.attempts:
                    attempt = ledger.begin(case.id, method.id, attempt_no)
                    run = make_limited_context(attempt, deadline=plan.deadline(case))
                    input_view = project_inputs(case, plan.allowed_inputs)
                    try:
                        with observer.end_to_end(attempt):
                            # enforce_deadline 需要进程/传输级取消；不只 try/except。
                            bundle = enforce_deadline(
                                adapter.transcribe(input_view, run), run.deadline)
                            validate_bundle(bundle, method.outputs, input_view)
                            refs = artifacts.commit_atomically(
                                bundle, parents=input_view.artifact_ids,
                                producer=attempt.id, method_identity=method.identity)
                        ledger.finish(attempt, status="ok", outputs=refs)
                        chosen = attempt.id       # 第一个合法结果，不看测试分数
                        break
                    except ExecutionFailure as error:
                        # run 已实时保存原始证据；已知部分输出也按规则保留。
                        ledger.finish(attempt, status=classify(error),
                                      evidence=run.evidence_refs)
                        if not plan.retry_policy.may_retry(error, run.remaining):
                            break
                ledger.select_for_scoring(case.id, method.id, chosen,
                                          cutoff_policy=plan.failure_policy)
    return ledger.freeze_index()                  # 保留全部名单和尝试
```

为使伪代码简短，上面只展示无状态/Case 独立的离线路径。有 episode 状态的方法必须按顺序调度，并从 checkpoint/起点恢复；不能直接套用任意样本重试。云 SDK 自动重试默认关闭或全部接入 ledger，避免重试藏在 adapter 中。

## 5. ElevenLabs 文件 API adapter

已核查的接口事实：文件入口为 `POST /v1/speech-to-text`，本例选 `scribe_v2`；请求可指定 `diarize` 与时间戳粒度。具体限制应从冻结版本的配置校验规则读取。[官方文件接口](https://elevenlabs.io/docs/api-reference/speech-to-text/convert)

```python
class ElevenLabsFile(WholeCaseMethod):
    def __init__(self, http_client, credentials, resolved_config):
        self.http = http_client           # 超时、取消、禁用隐藏重试
        self.auth = credentials           # 不序列化到 MethodSpec/证据
        self.cfg = resolved_config

    def transcribe(self, input, run):
        audio = run.media.open_authorized(input.audio)
        form = {
            "model_id": self.cfg.model,
            "diarize": self.cfg.diarize,
            "timestamps_granularity": self.cfg.timestamp_granularity,
            "no_verbatim": False,        # 本例任务要求保留口语内容
            "tag_audio_events": False,   # 本例只评语音文字
        }
        if input.has_allowed("language_hint"):
            form["language_code"] = map_language(input.language_hint)
        # 不从 ReferenceBundle 填 num_speakers / language / keyterms。

        raw = run.observe_call(
            self.http.post_multipart(
                url="https://api.elevenlabs.io/v1/speech-to-text",
                auth=self.auth,
                file=audio,
                fields=form,
                deadline=run.deadline))
        raw_ref = run.record_raw_response(raw, redact_transport_secrets=True)
        require_success_or_raise_typed_failure(raw)   # 429 等交给 runner 策略

        body = parse_expected_response_shape(raw)     # 本例明确是单通道
        projection = ElevenTextProjection(self.cfg.parser_revision)
        text, word_spans, acoustic_events = projection.from_response(body)

        return ResultBundle(
            transcript=Transcript(view="provider_rendered", text=text),
            alignment=word_alignment_if_requested_and_valid(
                word_spans, body.words, source_time_map=input.audio.time_map),
            attribution=word_speakers_if_requested_and_valid(
                word_spans, body.words, scope=input.case_id),
            extensions=acoustic_events,
            raw_evidence_refs=[raw_ref],
            completion=CompletedRequest(content_coverage="not_independently_proven"),
        )
```

官方响应区分 word、spacing 和 audio_event。[官方响应说明](https://elevenlabs.io/docs/overview/capabilities/speech-to-text) 对应的 parser 必须保留空格与文字跨度映射，并将非语音事件分开；不能简单把 `words` 全部用空格 join，尤其对中文不成立。若顶层 text 与结构化项无法一致映射，保存原文，标记不能对齐的字段；必需字段失配使该任务失败，不靠 gold 修正。

这里仅处理单通道响应。要使用服务的多通道模式，需为其响应形状新增 adapter 内的已验证分支和声明；channel 不自动变为 speaker。框架 runner 无需添加 ElevenLabs 分支。

**文件异步任务的扩展**：若采用另一个已核实的异步端点，adapter 内完成 submit→保存 job ID→轮询/接收结果；每次远程动作可追踪。恢复时优先查询原 job，只有确认策略允许才重提交。此处不编造一个 ElevenLabs polling URL，也不将文件上传自动解释为异步。

## 6. MOSS 自部署 adapter：传输可换，SATS 语义不变

本例选择 MOSS-Transcribe-Diarize 的 SGLang Omni 路径：官方模型卡给出 `/v1/audio/transcriptions` 与 `verbose_json`，后者提供解析后的 speaker 段；`json` 只返回原始转写字符串，长输出还涉及 token 预算。[官方模型卡](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize)

```python
class MossFile(WholeCaseMethod):
    def __init__(self, driver, parser, resolved_config):
        self.driver = driver             # HTTP 或 local Transformers
        self.parser = parser             # 与 backend revision/format 配对
        self.cfg = resolved_config

    def transcribe(self, input, run):
        raw = self.driver.generate(
            audio=run.media.open_authorized(input.audio),
            prompt=self.cfg.frozen_prompt,
            max_output_tokens=self.cfg.max_output_tokens,
            run=run,
        )
        raw_ref = run.record_raw_response(raw)
        parsed = self.parser.parse(raw)   # 原始串/JSON -> text, SATS segments
        validate_sats_segments(parsed, duration=input.audio.duration)
        # 检查 start<=end、speaker scope、解析残留；不要求各人的区间不重叠。

        transcript = transcript_from_sats(parsed, view="provider_rendered")
        return ResultBundle(
            transcript=transcript,
            alignment=Alignment(
                transcript_revision=transcript.revision,
                spans=parsed.segment_text_spans,
                intervals=map_to_source(parsed.segment_times, input.audio.time_map),
                granularity="segment", source="model_generated"),
            attribution=Attribution(
                transcript_revision=transcript.revision,
                spans=parsed.segment_text_spans,
                speakers=scope_ids(parsed.speakers, input.case_id)),
            raw_evidence_refs=[raw_ref],
            completion=completion_from_actual_metadata_or_unknown(raw),
        )

class MossSglangDriver:
    def generate(self, audio, prompt, max_output_tokens, run):
        return run.observe_call(http.post_multipart(
            self.base_url + "/v1/audio/transcriptions",
            file=audio,
            fields={"model": self.served_model_name,
                    "response_format": "verbose_json",
                    "prompt": prompt,
                    "max_new_tokens": max_output_tokens},
            deadline=run.deadline))

class MossLocalDriver:
    def generate(self, audio, prompt, max_output_tokens, run):
        # worker 已加载固定 model/processor；以下 helper 都是拟议封装。
        inputs = self.processor.prepare_authorized_audio_and_prompt(audio, prompt)
        generated = run.observe_call(self.model.generate(
            inputs, max_new_tokens=max_output_tokens))
        return self.processor.decode_with_available_stop_metadata(generated)

# 注册的是两个完整配置；同一 parser 只有在中间格式确实相同才复用。
registry.bind("moss.sglang", MossFile(MossSglangDriver(...), SglangSatsParser(...), ...))
registry.bind("moss.local", MossFile(MossLocalDriver(...), TaggedSatsParser(...), ...))
```

本地 Transformers 的预处理/生成链，以及 vLLM 部署入口可在[官方仓库](https://github.com/OpenMOSS/MOSS-Transcribe-Diarize)核查。`MossLocalDriver` 的 helper 不是仓库现有函数名；HTTP `prompt` 等参数也需与锁定的服务实现核对，不应仅凭兼容路径推断支持。

若换 vLLM，使用独立的 `MossVllmDriver + VerifiedVllmResponseParser`，按实际版本确定响应格式和输出预算参数。不能将 SGLang 的 `verbose_json/max_new_tokens` 原样假定为所有 vLLM 版本的等价参数。driver/parser 是此 adapter 的内部 Seam，不需要再扩散成全框架统一继承层。

**不会伪造的能力**：段时间不变成词时间；speaker 标号不变成实名；末段结束早于音频末尾可能只是静音，不能据此断言截断。明确长度终止/解析残缺按失败规则处理；无停止信息则 completeness=unknown，并在完整参考范围照常计算漏词，不把未知当确认完整。

## 7. Whisper 本地 adapter：模型常驻，迭代结束才算推理完成

本例选 faster-whisper。官方提供词时间戳/VAD 配置；特别提醒 `transcribe` 返回的 segments 是惰性生成器，迭代时才实际执行。[faster-whisper 官方说明](https://github.com/SYSTRAN/faster-whisper)

```python
class WhisperFile(WholeCaseMethod):
    def __init__(self, loaded_model, resolved_config):
        self.model = loaded_model        # 每 worker 加载一次固定 checkpoint
        self.cfg = resolved_config       # runtime/device/precision 全部冻结

    def transcribe(self, input, run):
        # 此例固定 VAD=False，避免隐藏前处理；打开 VAD 产生另一个 MethodSpec。
        with run.span("decode_and_transcribe"):
            audio = run.media.decode_authorized(input.audio)
            iterator, info = self.model.transcribe(
                audio.samples,
                beam_size=self.cfg.beam_size,
                language=allowed_language_or_none(input),
                task="transcribe",
                vad_filter=self.cfg.vad_filter,
                word_timestamps=self.cfg.word_timestamps,
                condition_on_previous_text=self.cfg.condition_on_previous_text,
            )
            segments = list(iterator)    # 必须在计时/异常捕获/取消范围内消费

        raw_ref = run.record_native_output(segments, info)
        text, segment_spans, word_spans = project_whisper_text_without_normalizing(
            segments)                   # 保留模型文字，分词/数字处理留给 scorer
        return ResultBundle(
            transcript=Transcript(view="provider_rendered", text=text),
            alignment=select_requested_alignment(
                segments, segment_spans, word_spans,
                input.audio.time_map, self.cfg.word_timestamps),
            attribution=Missing("base_whisper_has_no_speaker_output"),
            raw_evidence_refs=[raw_ref],
            completion=CompletedRequest(content_coverage="not_independently_proven"),
        )
```

数组输入的采样率必须与锁定运行库的数组输入契约匹配；若需要重采样，`decode_authorized` 使用已冻结 MediaView 变换，保留源时间映射并计入规定的计时范围。不能只传数组后忘记它是 8 kHz 还是 16 kHz。

Whisper 没有提供 speaker 时，普通文字指标仍正常工作。接入会议归属可以另建组合方法：

```python
class WhisperWithDiarization(WholeCaseMethod):
    def transcribe(self, input, run):
        words = run.child_method(self.whisper_with_word_times, input)
        activity = run.child_method(self.diarizer, input)
        attribution = associate(words.alignment, activity, self.frozen_policy)
        return words.with_new_attribution(attribution, parents=[words, activity])
```

`child_method` 是组合方法的受计量调用封装，不创建新的主测试样本；完整方法墙钟与所有子调用用量均保留。公开报告使用新 MethodIdentity“Whisper + diarizer”。同样的 recipe 原理可以复用，但不强制 MOSS 也走一个外部分人步骤。

## 8. 流式：ElevenLabs Realtime 需要 session，不复用文件假接口

官方 realtime 接口使用 WebSocket，音频与 partial/commit 事件双向传递。这里用独立的 `scribe_v2_realtime` 配置。[官方实时协议](https://elevenlabs.io/docs/api-reference/speech-to-text/v-1-speech-to-text-realtime)

### 8.1 框架负责真实供音及观察时钟

```python
async def execute_incremental(case, adapter, plan):
    run = new_attempt_context(case, plan)
    # 只传 session 元数据，不交付可读取整文件的路径。
    session = await adapter.open(project_session_metadata(case, plan), run)
    try:
        async def feed():
            async for chunk in controller.replay(case, plan.delivery_schedule):
                run.record_input_delivery(chunk.media_range, observer.now())
                await session.send(chunk)          # 同时监控背压、实际供音偏差
            await session.send(InputEnd())         # 不等于收到最终结果

        async def observe():
            async for event in session.receive():
                # observed_at 在 transport/native 收到事件时由可信 observer 打点；
                # published_at 是规范事件被消费的时刻。二者不混为同一个延迟。
                validate_event_revision(event)
                run.append_event(event, published_at=observer.now())

        await supervise_concurrently(
            feed(), observe(),
            total_deadline=plan.total_deadline,
            drain_after_input_end=plan.drain_deadline,
            on_disconnect="record_failure_no_implicit_reconnect",
        )
    finally:
        await bounded_close(session)
        run.seal_trace_and_cutoff_snapshot()
```

控制进程可以提前读音频文件，但方法进程只接收已释放样本。推理慢时不能把供音计划自动放慢后仍称正常实时负载。线程/进程中的阻塞本地推理不能阻塞接收与时钟。

### 8.2 adapter 负责线上协议映射

```python
class ElevenRealtime(IncrementalMethod):
    async def open(self, input, run):
        ws = await self.transport.connect(
            "wss://api.elevenlabs.io/v1/speech-to-text/realtime",
            params={"model_id": "scribe_v2_realtime",
                    "audio_format": "pcm_16000",
                    "commit_strategy": self.cfg.commit_strategy,
                    "include_timestamps": self.cfg.include_timestamps},
            credentials=self.credentials,
            deadline=run.deadline)
        return ElevenSession(ws, wire=self.verified_wire_contract, run=run)

class ElevenSession(Session):
    async def send(self, event):
        if event is AudioChunk:
            await self.ws.send(self.wire.encode_chunk(event))
        elif event is InputEnd:
            # 由固定 SDK/协议的已验证收尾规则处理剩余音频/commit。
            # 不编造 finish 字段；不从参考话轮生成人工端点。
            await self.wire.finish_input(self.ws)

    async def receive(self):
        async for raw, observed_at in self.ws.observed_messages():
            ref = self.run.record_wire_message(raw, observed_at)
            if raw.kind == "partial_transcript":
                yield self.state.replace_tentative_tail(raw.text, ref, observed_at)
            elif raw.kind == "committed_transcript":
                yield self.state.commit_text_once(raw.text, ref, observed_at)
            elif raw.kind == "committed_transcript_with_timestamps":
                match = self.correlator.identify_existing_commit(raw)
                if match.is_unambiguous:
                    yield self.state.attach_alignment(match, raw, ref, observed_at)
                else:
                    yield UnmappedEvidence(ref, reason="ambiguous_commit_association")
            elif self.wire.is_error(raw):
                raise ProviderFailure(self.wire.classify(raw), evidence=ref)
            else:
                self.state.record_control_event(raw, ref)
```

关键点是文本提交后可能再有附加时间信息；不能把两条消息当成两段转写。`correlator`、`finish_input` 是必须用真实响应 fixture 验证的 adapter 内部逻辑，不是已知现成 SDK 方法。[官方时间戳事件说明](https://elevenlabs.io/docs/api-reference/speech-to-text/v-1-speech-to-text-realtime)说明时间戳可延迟到达。

若没有可靠的段关联键，需要锁定并验证顺序契约；不能只靠文本相同关联，两次真实重复“好的”可能属于不同段。无法确定就保留原事件，将相应 alignment 标为缺失/歧义。基本文字修订仍可保存；以参考词时间配对文本 trace 的稳定延迟，并不要求 provider 原生词时间戳。

收尾不能把 socket close 无条件视为转写成功。必须按已核实协议判断提交是否完成；否则记录 finalization=unknown/timeout，并评分截止时可见结果。`observed_at` 来源与 `published_at` 来源固定：若评价 adapter 后的用户可见文本，用后者；若评价到客户端的网络事件，用前者，报告不能混用。

### 8.3 两种看起来类似但不应混淆的“stream”

```python
# 文件输出逐段 yield / HTTP token streaming：整段音频已经可见。
for segment in whisper.transcribe(full_audio):
    ...                                  # 仍然是 whole_case

# 真正渐进供音的组合方法：只有当时已经到达的音频。
class BufferedWhisperSession:
    on_audio(chunk):
        buffer.append(chunk)
        if frozen_schedule.decode_due():
            schedule_nonblocking_worker_job(buffer.allowed_window())

    on_decode_complete(window_result):
        globally_timed = map_window_result_to_source(window_result)
        revision = policy.merge_and_confirm(globally_timed, previous_hypotheses)
        publish(revision)                 # 明确去重/确认策略版本
        trim_only_confirmed_audio()
```

后一种是新方法“Whisper + window/confirmation policy”，要记重复计算、缓存和端点开销。MOSS 的文件响应即使逐 token 返回，也不能因此变成 online ASR。若未来提供真实 incremental 后端，再增加明确能力的 adapter。

## 9. 长录音与后处理：组合层是否够用

### 9.1 超长输入不由 runner 偷偷切开

```python
class ChunkedMethod(WholeCaseMethod):
    def transcribe(self, input, run):
        parts = self.segmenter.split(input.audio, frozen_policy=self.cfg.segmentation)
        predictions = []
        for part in parts:
            # 保存 source-time mapping、上下文 padding 和负责输出的区域。
            predictions.append(run.child_method(self.base, input.with_audio(part)))

        return self.merger.merge(
            predictions, source_maps=parts.maps,
            text_policy=self.cfg.text_merge,
            speaker_policy=self.cfg.speaker_linking,
            parents=[input.audio] + predictions)
```

对 Whisper，文字合并是重点；对分块 MOSS，跨块 speaker 关联还需要真实方法。每块的 S01 不能直接连为同一个人。若没有 speaker linking 方法，则该组合只能保证局部归属，不能声明全会议 speaker 一致性。已有长输入模型能直接运行时，优先保留其自然 whole-case 方法作对照。

### 9.2 冻结首遍后比较纠错

```python
correction_cases = prediction_dataset(
    transcripts=artifacts.from_run(baseline_run),
    audio_access="allowed_if_protocol_declares",  # 文本-only 可不含音频
    preserve={"case_id", "recording_id", "producer_identity"},
)

class CorrectionMethod(WholeCaseMethod):
    def transcribe(self, input, run):
        old = input.require("baseline_transcript")
        new = run.child_method(self.corrector, allowed_text_and_context(input))
        return ResultBundle(
            transcript=new,
            alignment=Missing("text_changed_requires_new_alignment"),
            attribution=Missing("spans_changed_requires_remapping"),
            raw_evidence_refs=[old.id, new.id])
```

若修改仅是标点，且方法提供可验证跨度映射，可保留映射后的 alignment；否则不能继承旧位置。此实验评分仍引用同一冻结 reference，另外报告净错误减少和误改。上游结果身份保留，成本区分“新增纠错成本”和“完整链成本”。

## 10. 三种输出实际怎样交给同一 scorer

```python
def score_recording(result_refs, reference, score_spec):
    # 三个模型已投影为标准产物，scorer 不读取厂商响应字段。
    hypothesis = artifacts.read(result_refs.transcript)
    ref_units = score_spec.normalize_and_tokenize(reference.transcript)
    hyp_units = score_spec.normalize_and_tokenize(hypothesis.text)
    return edit_statistics(ref_units, hyp_units)  # S/D/I/N，聚合时再相除

def score_speaker_transcription(result_refs, reference, score_spec):
    require(result_refs.attribution.is_valid_for(result_refs.transcript))
    if score_spec.requires_time:
        require(result_refs.alignment.satisfies(score_spec.time_policy))
    return score_spec.metric.evaluate(
        to_metric_view(result_refs, score_spec), reference)
```

| 任务 | ElevenLabs 文件 | MOSS 文件 | Whisper 本地 |
|---|---|---|---|
| 相同输入的普通文字 CER/WER | 可以，固定输出投影 | 可以，剥离结构标签的 parser 固定 | 可以，完整消费 iterator |
| 多人归属文字 | 配置启用并返回合法归属时 | 返回合法 speaker 段时 | 基础版缺能力；组合分人后可另测 |
| 严格原生词时间误差 | 配置/数据满足并校验覆盖时 | 本例仅段时间，拒绝 | 配置/数据满足并校验覆盖时 |
| 实时稳定/提交延迟 | 文件版不可以；Realtime 单列方法 | 文件版不可以 | 基础文件版不可以；窗口方法单列 |
| DER | 不能仅因词带 speaker 就默认有标准活动轨 | 不能默认 SATS 段等于完整活动轨 | 需专门活动输出或明确派生协议 |
| 冻结首遍的纠错 | 可以，经产物数据视图 | 可以，经产物数据视图 | 可以，经产物数据视图 |
| N-best/任意候选评分 | 本例 adapter 未提供，拒绝 | 本例 adapter 未提供，拒绝 | 本例 adapter 未提供；beam 参数不等于导出能力 |

“可以”仍需方法版本和数据资格检查，不能作为已运行认证。多人重叠内容不要简单按 speaker ID 排序后算普通 WER 来代替对应多说话人指标。

### 手工结果推演

假设某条人工参考为“你好”，三个方法都输出“你号”。这是人为构造的契约样例，不是模型调用结果。投影后的 Transcript 均是两个字符，普通 CER 的 S=1、D=0、I=0、N=2，结果均为 50%。ElevenLabs 可能还附词时间与 speaker，MOSS 附段时间与 speaker，Whisper 只有文字/所选时间；这些额外字段不会改变普通文字分数。

若改成 speaker 归属任务，前两者仅在开启/提供合法归属时可入选，基础 Whisper 在运行前被拒绝。若改成严格词时间任务，本例 MOSS 的段时间不足，被拒绝。用同一个 bundle 保存多种信息，不意味着强迫同一个指标适用所有方法。

## 11. 这次伪代码暴露的接口细节

1. **部署方式与方法身份分开，但一起冻结**。自部署 HTTP 不比 SaaS 更像“模型”；Whisper 暴露成 HTTP 也不应改变其输出语义。backend/runtime 的变化仍形成新配置，不能承诺分数完全一致。
2. **outputs 必须由配置决定**。同一 ElevenLabs adapter 的 diarize 开关、Whisper 的 word_timestamps 开关改变输出承诺；不能用注册时一个永远不变的 capability 集合。
3. **原始接收与规范事件发布分开计时**。解析/队列延迟真实存在；从网络接收到 scorer 可见不能用一个不明归属的 timestamp 代替。
4. **文字提交与其他字段补充独立**。实时文本提交后补时间信息，不能重复计字，也不能推迟文字首现时间。
5. **worker 准备与每次请求分开**。GPU 权重常驻、API 凭证注入与 connection pool 不属于每个 Case 的答案状态；冷/热计时仍由协议固定。
6. **generation budget 与完成性独立**。MOSS 长输出、HTTP 成功、局部解析成功都不能单独证明完整覆盖；需要保留停止证据和未知状态。

这些属于既有 Module 内 Interface 的具体化，不需要让核心 runner 知道模型名。但 `outputs_for(config)`、观察/发布时刻和字段补充语义应进入总体文档，不能只留在示例中。

## 12. 如何判断接入是否真的“好接”

建议后续实现按下列最小 fixture 验收；目前都是预期，不是已执行测试：

| 输入/事件 fixture | 必须得到的结果 |
|---|---|
| 三个 adapter 返回同一规范文字“你好” | 同一文字 scorer 产生相同编辑统计，无模型名分支 |
| Whisper 返回到第 3 段才抛异常 | 不能在调用 transcribe 返回时宣称推理成功；保留部分证据 |
| ElevenLabs 429 后重试成功 | 两个 attempt，失败时间/调用账本保留，最终结果只选一次 |
| MOSS 一个 speaker 段越过输入时长 | 结构校验发现错误；不能 clamp 后隐藏问题 |
| MOSS 只有段时间 | 请求严格词时间任务在预检失败，不做均分造词时间 |
| Realtime partial A→AB→commit AB→timestamps AB | 文字最终只有 AB，迟到时间附加到同一提交 |
| Realtime 两段都说“好的” | 不按文本内容去重；关联歧义显式标出 |
| 分块 MOSS 每块都有 S01 | 无关联证据时保持局部 scope，不合成全局同人 |
| 只改评分 normalizer | 从相同 artifact 重评分，三个 adapter 均不再调用 |
| MOSS 切换 SGLang→vLLM | 只改其 driver/parser/配置身份；参考、Case、文字 scorer 不改 |

第一版最小接入量：三个 whole-case adapters、一个共享离线 runner、一个文字 scorer、一个标准数据 view 即可证明主路径。再用 ElevenLabs Realtime 验证第二种生命周期。先完成这些真实契约试验，再判断是否需要提取更多共享 transport/worker 工具；不先写一个全能 ASRClient。

## 一手来源

- [ElevenLabs 文件转写接口](https://elevenlabs.io/docs/api-reference/speech-to-text/convert)
- [ElevenLabs 输出类型与功能说明](https://elevenlabs.io/docs/overview/capabilities/speech-to-text)
- [ElevenLabs 实时 WebSocket 协议](https://elevenlabs.io/docs/api-reference/speech-to-text/v-1-speech-to-text-realtime)
- [ElevenLabs 服务端流式指南](https://elevenlabs.io/docs/eleven-api/guides/how-to/speech-to-text/realtime/server-side-streaming)
- [MOSS 模型卡](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize)
- [MOSS 官方仓库](https://github.com/OpenMOSS/MOSS-Transcribe-Diarize)
- [faster-whisper 官方仓库](https://github.com/SYSTRAN/faster-whisper)
