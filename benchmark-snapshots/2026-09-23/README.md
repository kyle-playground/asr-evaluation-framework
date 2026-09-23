# ASR 排名快照（2026-09-23）

- `retrieval.json`：抓取时间及官方来源。
- `hf-displayed-tables.json`：HF公开Gradio配置中的当前展示表，移除了模型名周围的HTML；排名报告采用此值。
- `hf-space-config.json`：原始公开配置。
- `hf-version.json`、`hf-*.csv`：2026-09-19登记的公开CSV及revision；某些平均与前端展示口径不同，不能直接替代当前榜。
- `hf-*.py.txt`：公开聚合逻辑快照，未执行。
- `aa-batch-models.json`：离线页面59条API明细；并非71模型全榜。
- `aa-streaming-models.json`：流式页面37配置，报告使用defaultSelectedStreaming的31项。
- `aa-*-tables.json`：静态HTML表格提取，streaming为空；流式数值来自页面公开结构化数据。
- `gigaspeechbench-README.md`：固定ca782bf版本的官方结果。

本目录没有私有音频或标注，也不包含本机运行模型结果。
