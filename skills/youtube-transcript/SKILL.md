---
name: youtube-transcript
description: Get the transcript, subtitles or captions of YouTube videos, Shorts, playlists or whole channels as plain text, timestamped segments, SRT or VTT. Use when the user shares a YouTube link and wants a summary, notes, quotes, search or subtitles, or asks for the transcripts of a channel or playlist. Works on servers and cloud machines, where YouTube blocks direct requests.
---

# YouTube Transcript

Fetches YouTube transcripts with `scripts/transcript.py`, which calls the
[YouTube Transcript Scraper](https://apify.com/andracore/youtube-transcript-scraper) on Apify.
The script uses only the Python standard library.

## Requirements

`APIFY_TOKEN` must be set in the environment. If it is missing, the script prints how to get one:
a free Apify account at https://console.apify.com/sign-up, then the token at
https://console.apify.com/settings/integrations. Tell the user that, and don't try other ways to reach YouTube.

## Usage

```bash
python3 scripts/transcript.py <links or IDs...> [options]
```

Accepted links: `youtube.com/watch?v=...`, `youtu.be/...`, `youtube.com/shorts/...`, an 11-character video ID,
`youtube.com/playlist?list=...`, and channels (`youtube.com/@handle` or a bare `@handle`). A channel gives its
newest long videos; add `/shorts` or `/streams` to the channel link for Shorts or past live streams.

| Option | Default | What it does |
|---|---|---|
| `--lang xx` | `en` | Preferred caption language. If the video doesn't have it, the original language is returned. |
| `--format` | `text` | `text`, `timestamped` (JSON segments with `startMs` and `durationMs`), `srt`, `vtt` or `all` |
| `--max-videos N` | `10` | Stop after N videos, counted across all links. Also caps the cost of the run. |
| `--no-auto` | off | Skip videos that only have auto-generated captions |
| `--out DIR` | off | Write one file per video (`<videoId>.srt`, `.vtt`, `.txt` or `.json`) instead of printing |
| `--json` | off | Print the raw results, with title, author, channelId, lengthSeconds, viewCount and availableLanguages |
| `--translate xx` | off | Ask YouTube to machine-translate. Best effort: YouTube often refuses, and then the original is returned |

Examples:

```bash
python3 scripts/transcript.py https://youtu.be/dQw4w9WgXcQ
python3 scripts/transcript.py "https://www.youtube.com/watch?v=dQw4w9WgXcQ" --lang pt
python3 scripts/transcript.py @veritasium --max-videos 5 --format srt --out subtitles/
```

Quote URLs that contain `?` so the shell doesn't expand them.

## Output

Text output has one block per video: a `# Title` line, the URL with the caption language
(and `auto-generated` when it applies), then the transcript. Videos without captions show
`No transcript: <reason>` and are not charged.

## Guidance

- For a summary or questions about one video, use the default text format and work from the transcript.
- To translate, fetch the original and translate it yourself. YouTube's own translation is often rate limited.
- For subtitles to use in a video editor, use `--format srt --out <dir>`.
- For a channel or playlist, set `--max-videos` to what the user actually needs. A run must finish in 5 minutes,
  so split very large requests into several calls.

## Cost

US$ 0.004 per transcript on the Apify free plan, less on paid plans. The free plan includes US$ 5 of credit
every month, about 1,250 transcripts. Failed videos and videos without captions are not charged.
