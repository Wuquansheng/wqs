---
name: meeting-video-docs
description: Convert meeting recordings such as mp4, mov, m4a, wav, or other audio/video files into simplified-Chinese meeting documents. Use when the user asks to start from a meeting recording, meeting screen recording, video, audio, or transcript and produce Word documents containing a complete meeting transcript, separated presentation/speech and Q&A sections, meeting summary, notes, decisions, action items, or follow-up items.
---

# Meeting Video Docs

## Purpose

Turn a meeting recording into two Word files:

1. **完整版**: simplified-Chinese complete transcript, grouped into presentation/speech and Q&A sections, preserving timestamps.
2. **总结版**: simplified-Chinese structured summary of the presentation/speech and Q&A sections, emphasizing key points, decisions, risks, action items, and follow-ups.

Use `scripts/meeting_video_docs.py` for deterministic transcription post-processing and DOCX generation.

## Workflow

1. **Locate the recording**
   - Accept common audio/video files: `.mp4`, `.mov`, `.m4a`, `.mp3`, `.wav`, `.webm`.
   - If multiple meeting recordings exist and the user did not name one, ask which file to use.

2. **Prepare dependencies**
   - Prefer the project virtual environment if present.
   - Required Python packages: `faster-whisper`, `python-docx`, `opencc-python-reimplemented`.
   - Install missing packages only after normal sandbox/network rules are satisfied.
   - If Hugging Face model download fails in China/network-restricted environments, try:
     - `HF_ENDPOINT=https://hf-mirror.com`
     - `HF_HOME=<workspace>/.hf_cache`
     - User proxy if available, commonly `HTTP_PROXY=http://127.0.0.1:7890` and `HTTPS_PROXY=http://127.0.0.1:7890`.

3. **Transcribe and create the complete Word file**
   - First run the bundled script without `--qa-start` unless the user already provided a split time. Example:

```powershell
.\.venv\Scripts\python.exe skills\meeting-video-docs\scripts\meeting_video_docs.py `
  --video meeting_01.mp4 `
  --out-dir meeting_01_outputs `
  --model small `
  --language zh `
  --title "meeting_01 会议"
```

   - Use `small` as the default Whisper model for a balanced speed/quality tradeoff.
   - Use `medium` only when accuracy is more important and runtime is acceptable.
   - The script creates:
     - `<stem>_segments.json`
     - `<stem>_transcript_简体.md`
     - `<stem>_transcript_简体.txt`
     - `<stem>_完整版_简体.docx`

4. **Identify the Q&A start time**
   - If the user provides a split point, use it.
   - Otherwise inspect the transcript after transcription and choose a meeting-specific split point around likely transition phrases:
     - “大家有什么问题”
     - “有什么问题吗”
     - “我先讲到这里”
     - “接下来大家交流”
     - “问答”
   - Pass the split time with `--qa-start HH:MM:SS` only after identifying the actual transition.
   - If no Q&A exists, omit `--qa-start` and produce one continuous complete transcript.
   - Do not treat any example timestamp as a default; each meeting needs its own split judgment.

5. **Create the summary Markdown**
   - Read the full simplified transcript and produce a concise but complete Markdown summary.
   - Always include separate sections:
     - `# 总体结论`
     - `# 演讲部分总结`
     - `# 问答部分总结`
     - `# 会议共识与后续行动`
   - For each section, preserve important details; do not collapse distinct topics into one vague bullet.
   - Capture:
     - meeting theme
     - background and purpose
     - presentation logic
     - key concepts and examples
     - questions raised
     - answers and recommendations
     - decisions/consensus
     - action items
     - risks, blockers, and caveats
   - Save the summary as UTF-8 Markdown, for example `meeting_01_outputs/meeting_01_summary_for_docx.md`.

6. **Generate the summary Word file**
   - Re-run the script with `--segments` and `--summary-md`:

```powershell
.\.venv\Scripts\python.exe skills\meeting-video-docs\scripts\meeting_video_docs.py `
  --segments meeting_01_outputs\meeting_01_segments.json `
  --out-dir meeting_01_outputs `
  --qa-start <actual-q-and-a-start-time> `
  --summary-md meeting_01_outputs\meeting_01_summary_for_docx.md `
  --title "meeting_01 会议"
```

   - The script creates `<stem>_总结_简体.docx`.

7. **Verify outputs**
   - Check both DOCX files exist and are non-empty.
   - Open them with `python-docx` and verify paragraph counts are reasonable.
   - Report paths to the user and state any limitations, especially machine-transcription uncertainty for names, software names, and specialized terms.

## Summary Quality Rules

- Use simplified Chinese in final outputs.
- Separate presentation/speech content from Q&A content.
- Keep timestamps in the complete version.
- Do not invent speaker names if the transcript does not identify speakers.
- Correct obvious transcription variants only when context is clear, e.g. “深信/身性” likely means “生信”.
- Keep uncertain proper nouns conservative; mention possible transcription error rather than over-correcting.
- For long meetings, sample multiple time ranges before summarizing and ensure late Q&A conclusions are included.

## Useful Commands

Install dependencies in a project venv:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install faster-whisper python-docx opencc-python-reimplemented
```

Run with Hugging Face mirror and local cache:

```powershell
$env:HF_ENDPOINT='https://hf-mirror.com'
$env:HF_HOME='<workspace>\.hf_cache'
.\.venv\Scripts\python.exe skills\meeting-video-docs\scripts\meeting_video_docs.py --video meeting_01.mp4 --out-dir meeting_01_outputs
```
