# YT Downloader Dev Skill

A Claude Code Skill that gives AI agents structured, searchable knowledge about a YouTube downloader project (Flask + vanilla JS + yt-dlp + ffmpeg).

**152 entries** across **7 knowledge domains**, powered by a **BM25 search engine** — zero external dependencies.

---

## What's Inside

| Domain | Entries | Coverage |
|--------|---------|----------|
| `pitfalls` | 16 | Common bugs, Windows issues, YouTube 429 rate limiting |
| `subtitle` | 13 | Subtitle caching, 429 cooldown, VTT/SRT conversion |
| `api` | 14 | All 14 API endpoints with params and responses |
| `download` | 12 | Download flow stages, progress SSE, ffmpeg burn-in |
| `ytdlp` | 12 | yt-dlp option patterns, format selection, locale |
| `ui` | 46 | DOM elements, CSS classes, event listeners |
| `design` | 39 | Dark theme tokens: colors, spacing, typography, animation |

## How It Works

Instead of loading all project knowledge into CLAUDE.md (which gets too long to read), this Skill structures knowledge into **CSV databases** that Claude can **search on demand** via a BM25 search engine.

```
User: "The subtitle download is broken"

Claude: (runs search.py "subtitle download" → finds relevant entries)
Claude: "Use _get_cached_subtitle() with explicit video_id from frontend.
         YouTube only provides VTT — convert with _vtt_to_srt().
         On 429, global cooldown (5min) activates — no retries."
```

Claude searches first, then answers — instead of re-reading 2300 lines of app.py every time.

---

## Installation

### For Claude Code

```bash
# Clone into your project's .claude/skills/ directory
cd your-project
git clone https://github.com/shushuitie2017/YTDownloader-SKILL.git .claude/skills/yt-dl-dev
```

That's it. Claude Code will automatically detect the SKILL.md and use the search engine.

### Manual Installation

```bash
# 1. Clone the repo
git clone https://github.com/shushuitie2017/YTDownloader-SKILL.git

# 2. Copy to your project's skill directory
cp -r YTDownloader-SKILL /path/to/your-project/.claude/skills/yt-dl-dev
```

### Verify Installation

```bash
cd .claude/skills/yt-dl-dev
python scripts/search.py --domains
```

Expected output:

```
Available domains:
  api          — api-reference.csv              (14 entries)
  ui           — ui-components.csv              (46 entries)
  pitfalls     — pitfalls.csv                   (16 entries)
  ytdlp        — yt-dlp-patterns.csv            (12 entries)
  design       — design-tokens.csv              (39 entries)
  subtitle     — subtitle-system.csv            (13 entries)
  download     — download-flow.csv              (12 entries)
```

---

## Usage

### Search a Specific Domain

```bash
python scripts/search.py "429 subtitle cooldown" --domain pitfalls
```

Output:

```
## YT-DL Dev Search Results
**Domain:** pitfalls | **Query:** 429 subtitle cooldown
**Source:** pitfalls.csv | **Found:** 4 results

### Result 1
- **Issue:** Subtitle 429 rate limiting
- **Cause:** YouTube timedtext API IP-level rate limit from too many requests
- **Fix:** Global cooldown _subtitle_429_until (5min). NO retries
- **Severity:** CRITICAL
- **Code Example:** _subtitle_429_until = time.time() + 300
```

### Auto-Detect Domain

```bash
# Automatically routes to "design" domain
python scripts/search.py "dark theme color accent"

# Automatically routes to "subtitle" domain
python scripts/search.py "vtt srt cache 429"
```

### Cross-Domain Overview

```bash
python scripts/search.py "subtitle download" --overview
```

Searches all 7 domains and shows top matches from each.

---

## Project Structure

```
YTDownloader-SKILL/
├── README.md                          # This file
├── skill.json                         # Skill metadata
├── LICENSE                            # MIT
├── .claude/skills/yt-dl-dev/
│   └── SKILL.md                       # Main skill guide (Claude reads this first)
├── data/
│   ├── api-reference.csv              # 14 API endpoints
│   ├── ui-components.csv              # 46 UI components
│   ├── pitfalls.csv                   # 16 common pitfalls
│   ├── yt-dlp-patterns.csv            # 12 yt-dlp patterns
│   ├── design-tokens.csv              # 39 design tokens
│   ├── subtitle-system.csv            # 13 subtitle components
│   └── download-flow.csv              # 12 download flow stages
└── scripts/
    └── search.py                      # BM25 search engine (150 lines, 0 deps)
```

## The Target Project

This Skill was built for **YT Downloader** — a web-based YouTube downloader with:

| Layer | Tech |
|-------|------|
| Backend | Python Flask, yt-dlp, requests |
| Frontend | Vanilla JavaScript, HTML5, CSS3 |
| Player | HTML5 Video (720p) + Shaka Player (1080p+ DASH) |
| Dependencies | ffmpeg (WinGet auto-detect) |
| Platform | Windows (Explorer integration, tkinter folder picker) |

Key subsystems: multi-layer subtitle caching with YouTube 429 cooldown, threaded download worker with SSE progress, DASH MPD manifest generation, CJK language detection engine, dark theme UI with 3-column responsive layout.

---

## Adapting for Your Own Project

This Skill is designed as a **reference implementation**. To create a similar Skill for your project:

1. **Identify domains** — What do you repeatedly look up? (APIs, bugs, config, patterns)
2. **Create CSVs** — Each domain = one CSV with a `Keywords` column for search
3. **Copy search.py** — Change `CSV_CONFIG` and `DOMAIN_KEYWORDS` to match your domains
4. **Write SKILL.md** — Quick reference + architecture diagrams + search instructions

The BM25 engine is project-agnostic. Only the data and domain config need to change.

---

## Requirements

- Python 3.6+ (standard library only — no pip install needed)
- Claude Code (or any AI coding assistant that supports Skills)

## License

MIT
