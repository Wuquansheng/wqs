# meeting-video-docs

`meeting-video-docs` 是一个用于处理会议录屏/会议音频的 Codex Skill。它可以从 `.mp4`、`.mov`、`.m4a`、`.mp3`、`.wav` 等会议文件出发，生成简体中文 Word 文档：

- 完整版：保留时间戳，按“演讲部分 / 问答部分”整理完整会议内容
- 总结版：分别总结演讲部分和问答部分，归纳重点、共识、行动项和注意事项

## 文件结构

```text
meeting-video-docs/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── scripts/
    └── meeting_video_docs.py
```

## 依赖

建议在项目虚拟环境中安装：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install faster-whisper python-docx opencc-python-reimplemented
```

如果 Whisper 模型下载较慢或失败，可以使用 Hugging Face 镜像和本地缓存：

```powershell
$env:HF_ENDPOINT='https://hf-mirror.com'
$env:HF_HOME='<workspace>\.hf_cache'
```

如需走本机代理，可按实际情况设置：

```powershell
$env:HTTP_PROXY='http://127.0.0.1:7890'
$env:HTTPS_PROXY='http://127.0.0.1:7890'
```

## 基本用法

从会议视频生成完整转写和完整版 Word：

```powershell
.\.venv\Scripts\python.exe skills\meeting-video-docs\scripts\meeting_video_docs.py `
  --video meeting_01.mp4 `
  --out-dir meeting_01_outputs `
  --model small `
  --language zh `
  --qa-start 00:31:04 `
  --title "meeting_01 会议"
```

脚本会生成：

- `meeting_01_segments.json`
- `meeting_01_transcript_简体.md`
- `meeting_01_transcript_简体.txt`
- `meeting_01_完整版_简体.docx`

## 生成总结版 Word

先让 Codex 根据完整转写稿整理一个 UTF-8 Markdown 总结文件，例如：

```text
meeting_01_outputs/meeting_01_summary_for_docx.md
```

建议总结 Markdown 包含：

- 总体结论
- 演讲部分总结
- 问答部分总结
- 会议共识与后续行动

然后运行：

```powershell
.\.venv\Scripts\python.exe skills\meeting-video-docs\scripts\meeting_video_docs.py `
  --segments meeting_01_outputs\meeting_01_segments.json `
  --out-dir meeting_01_outputs `
  --qa-start 00:31:04 `
  --summary-md meeting_01_outputs\meeting_01_summary_for_docx.md `
  --title "meeting_01 会议"
```

脚本会生成：

- `meeting_01_总结_简体.docx`

## 参数说明

| 参数 | 说明 |
| --- | --- |
| `--video` | 会议视频或音频文件路径 |
| `--segments` | 已有转写分段 JSON，复用转写结果时使用 |
| `--out-dir` | 输出目录 |
| `--model` | Whisper 模型，默认 `small` |
| `--language` | 识别语言，中文建议 `zh` |
| `--qa-start` | 问答开始时间，例如 `00:31:04` |
| `--summary-md` | Codex 已整理好的总结 Markdown |
| `--title` | Word 文档标题 |

## 注意事项

- 完整版保留机器转写的口语表达和时间戳。
- 总结版需要 Codex 基于完整内容归纳，脚本只负责把 Markdown 转为 Word。
- 个别人名、软件名、专业术语可能存在转写误差，应以原视频为最终依据。
- 如果会议没有明确问答部分，可以不传 `--qa-start`，脚本会生成连续完整稿。
