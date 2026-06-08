# wqs

个人小工具集合仓库。

## 工具列表

### meeting-video-docs

从会议录屏或会议音频出发，生成简体中文 Word 文档：

- 完整版会议内容：保留时间戳，按“演讲部分 / 问答部分”整理
- 总结版会议纪要：归纳演讲重点、问答重点、会议共识和行动项

目录：[meeting-video-docs](meeting-video-docs/)

## 目录规划

```text
wqs/
├── README.md
└── meeting-video-docs/
    ├── SKILL.md
    ├── README.md
    ├── agents/
    │   └── openai.yaml
    └── scripts/
        └── meeting_video_docs.py
```

后续新增工具时，建议每个工具独立一个目录，并在本 README 的“工具列表”中补一条索引。
