# Transcribe YouTube

Extract and clean YouTube video transcripts into readable markdown.

## Quick Start

1. Copy the `transcribe-youtube/` folder into your project's `skills/` directory
2. Install yt-dlp: `pip install yt-dlp`
3. Ask Copilot: *"Transcribe this YouTube video: https://youtube.com/watch?v=..."*

## Prerequisites

| Requirement | Required | Notes |
|-------------|----------|-------|
| **GitHub Copilot CLI** | Yes | Any project folder — no framework dependencies |
| **yt-dlp** | Yes | Install via `pip install yt-dlp` |
| **Python 3** | Yes | For VTT parsing and text cleanup |
| **Internet access** | Yes | To download captions from YouTube |

The video must have English captions available (auto-generated or manual).

This skill is **framework-independent** — it works with plain Copilot CLI from any project folder.

## What It Does

1. Downloads captions/subtitles from a YouTube video via `yt-dlp`
2. Parses the VTT file and strips timestamps, markup tags, and metadata
3. Deduplicates overlapping caption lines
4. Groups cleaned text into readable paragraphs
5. Saves as a clean markdown file

## Inputs

| Input | Required | Example |
|-------|----------|---------|
| YouTube URL | Yes | `https://www.youtube.com/watch?v=dQw4w9WgXcQ` |
| Output filename | No (auto-generated from video title) | "Azure AI Overview Transcript" |
| Language | No (default: English) | `es`, `fr`, `de` |

## Output

```
outputs/Azure-AI-Overview-Transcript.md
```

Clean markdown with:
- Header (title, source URL, transcript type)
- Readable paragraphs — no timestamps, no VTT markup
- Automatic sentence-boundary paragraph breaks

## Supported URL Formats

- `https://www.youtube.com/watch?v=VIDEO_ID`
- `https://youtu.be/VIDEO_ID`
- `https://youtube.com/watch?v=VIDEO_ID&t=123`

## License

MIT
