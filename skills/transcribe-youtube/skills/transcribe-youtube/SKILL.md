---
name: transcribe-youtube
description: Use this skill when the user asks to "transcribe a YouTube video", "get transcript from YouTube", "download YouTube captions", "extract video transcript", "YouTube to markdown", or wants to convert a YouTube video's spoken content into a readable markdown document. Supports any YouTube URL with auto-generated or manual captions.
---

# Transcribe YouTube — Video Transcript to Markdown

Extract captions/subtitles from a YouTube video and produce a clean, readable
markdown transcript.

## Overview

This skill uses `yt-dlp` to download YouTube auto-generated or manual captions
(VTT format), then cleans and formats them into readable markdown paragraphs.

**Why not `markitdown` directly?** The `markitdown` YouTube extractor frequently
fails due to bot detection / captcha issues on YouTube. The `yt-dlp` approach is
more reliable.

## Prerequisites

`yt-dlp` must be installed. If not present, install it:

```bash
pip install yt-dlp
```

The executable installs to `C:\Users\pawu\AppData\Roaming\Python\Python313\Scripts\yt-dlp.exe`.
If not on PATH, use the full path.

## Workflow

### Step 1: Get the YouTube URL

Accept a YouTube URL from the user. Supported formats:
- `https://www.youtube.com/watch?v=XXXXXXXXXXX`
- `https://youtu.be/XXXXXXXXXXX`
- `https://www.youtube.com/embed/XXXXXXXXXXX`

Extract the video ID from the URL for use in temp file naming.

### Step 2: Ask for Output Filename

If the user provided a filename, use it. Otherwise, ask or generate a
descriptive filename based on the video title:

```
outputs/{filename}.md
```

Default location is the `outputs/` folder.

### Step 3: Download Captions with yt-dlp

Run `yt-dlp` to download auto-generated subtitles in VTT format:

```powershell
$ytdlp = "C:\Users\pawu\AppData\Roaming\Python\Python313\Scripts\yt-dlp.exe"
& $ytdlp --write-auto-sub --sub-lang en --skip-download --sub-format vtt -o "outputs/temp-{video_id}" "{youtube_url}"
```

**Flags explained:**
- `--write-auto-sub` — Download auto-generated subtitles (fallback if no manual subs)
- `--sub-lang en` — English language captions
- `--skip-download` — Don't download the video file itself
- `--sub-format vtt` — WebVTT format (timestamped captions)

**Output:** Creates a file like `outputs/temp-{video_id}.en.vtt`

**If captions are unavailable:** yt-dlp will report no subtitles found. Inform
the user that the video does not have English captions available.

**Language override:** If the user requests a different language, replace `en`
with the appropriate language code (e.g., `es`, `fr`, `ja`).

### Step 4: Clean VTT into Readable Markdown

The raw VTT file contains duplicate lines, timestamps, HTML tags, and position
markers that make it unreadable. Run this Python cleaning script:

```python
import re

with open(vtt_path, 'r', encoding='utf-8') as f:
    raw = f.read()

lines = raw.split('\n')
text_lines = []
seen = set()

for line in lines:
    line = line.strip()
    # Skip headers, timestamps, empty lines, position tags
    if not line:
        continue
    if line.startswith('WEBVTT') or line.startswith('Kind:') or line.startswith('Language:'):
        continue
    if re.match(r'^\d{2}:\d{2}', line):
        continue
    # Remove inline VTT tags like <00:00:01.234><c> </c>
    clean = re.sub(r'<[^>]+>', '', line)
    clean = clean.strip()
    if clean and clean not in seen:
        seen.add(clean)
        text_lines.append(clean)

# Group into paragraphs — break on sentence endings every ~3-5 lines
paragraphs = []
current = []
for line in text_lines:
    current.append(line)
    if len(current) >= 5 or (line.endswith('.') and len(current) >= 2):
        paragraphs.append(' '.join(current))
        current = []
if current:
    paragraphs.append(' '.join(current))
```

**Cleaning steps:**
1. Strip WEBVTT headers and metadata lines
2. Remove all timestamp lines (`00:00:00.000 --> ...`)
3. Remove inline VTT markup (`<c>`, `<00:00:01.234>`, etc.)
4. Deduplicate lines (auto-captions repeat each line as it builds)
5. Group into readable paragraphs (break at sentence endings)

### Step 5: Write Markdown Output

Format the cleaned transcript as markdown:

```markdown
# {Title}

**Source:** [YouTube]({url})
**Type:** Auto-generated transcript

---

{paragraph 1}

{paragraph 2}

{paragraph 3}
...
```

If the video title is not known, derive it from the filename the user requested
or use "YouTube Transcript" as a default.

Save to the user-specified path (default: `outputs/{filename}.md`).

### Step 6: Clean Up

Delete the temporary VTT file:

```powershell
Remove-Item -LiteralPath "outputs/temp-{video_id}.en.vtt" -Force
```

### Step 7: Confirm

Report to the user:
- File path and size
- Number of paragraphs extracted
- Offer to summarize the transcript or add it to the knowledgebase

## Examples

**User:** "Transcribe this YouTube video: https://www.youtube.com/watch?v=abc123"
→ Download captions, clean, save as `outputs/abc123-transcript.md`

**User:** "Get the transcript from https://youtu.be/xyz789 and save it as cloud-overview.md"
→ Download, clean, save as `outputs/cloud-overview.md`

**User:** "Transcribe this video in Spanish: https://www.youtube.com/watch?v=def456"
→ Use `--sub-lang es`, download, clean, save.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `yt-dlp` not found | Run `pip install yt-dlp` |
| No subtitles available | Video has no captions; inform the user |
| VTT file is empty | Try `--write-sub` instead of `--write-auto-sub` (manual subs) |
| Non-English captions | Use `--sub-lang {code}` with the right language code |
| yt-dlp outdated / extraction errors | Run `pip install --upgrade yt-dlp` |

## Rules

- ALWAYS use `yt-dlp` for caption download (not `markitdown` for YouTube URLs)
- ALWAYS clean VTT markup before saving — never save raw VTT as the final output
- ALWAYS deduplicate caption lines (auto-captions contain many repeats)
- ALWAYS delete the temp VTT file after conversion
- ALWAYS include the YouTube source URL in the markdown header
- ALWAYS default to English (`en`) unless the user specifies another language
- NEVER download the actual video file — use `--skip-download`
- Save output to `outputs/` by default unless the user specifies another location
