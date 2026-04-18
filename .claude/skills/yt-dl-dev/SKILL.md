---
name: yt-dl-dev
description: "YT Downloader development intelligence. 14 API endpoints, 40+ UI components, 12 yt-dlp patterns, 16 pitfalls, 35 design tokens, 13 subtitle system components, 12 download flow stages. Actions: add endpoint, build feature, fix bug, add UI component, debug download, configure subtitle, optimize search, review code. Stack: Flask + vanilla JS + yt-dlp + ffmpeg on Windows. Domains: api, ui, pitfalls, ytdlp, design, subtitle, download."
---
# YT Downloader Dev — Development Intelligence

Comprehensive development guide for the YouTube downloader project. Contains 14 API endpoints, 40+ UI components, 12 yt-dlp option patterns, 16 common pitfalls, 35 design tokens, 13 subtitle system components, and 12 download flow stages. Searchable database with BM25 ranking.

## When to Apply

Reference these guidelines when:
- Adding new API endpoints or modifying existing ones
- Working with yt-dlp (search, download, metadata, subtitles)
- Debugging download failures, 429 rate limits, or subtitle issues
- Adding/modifying UI components in the dark theme
- Troubleshooting Windows-specific issues (paths, ffmpeg, encoding)
- Understanding the subtitle caching and 429 cooldown system

## Knowledge Domains by Priority

| Priority | Domain | Entries | Impact | Use For |
|----------|--------|---------|--------|---------|
| 1 | `pitfalls` | 16 | CRITICAL | Common bugs, Windows issues, 429 handling |
| 2 | `subtitle` | 13 | CRITICAL | Subtitle caching, 429 cooldown, VTT/SRT conversion |
| 3 | `api` | 14 | HIGH | All API endpoints, params, responses |
| 4 | `download` | 12 | HIGH | Download flow, progress SSE, ffmpeg burn-in |
| 5 | `ytdlp` | 12 | HIGH | yt-dlp option patterns, format selection |
| 6 | `ui` | 40+ | MEDIUM | DOM elements, CSS classes, event listeners |
| 7 | `design` | 35 | MEDIUM | Dark theme colors, spacing, typography, animations |

## Quick Reference

### 1. Critical Pitfalls (MUST KNOW)

- `subtitle-429` — NEVER retry on 429. Global cooldown 5min. Single attempt only
- `subtitle-store-miss` — Always pass video_id explicitly from frontend, not URL regex
- `subtitle-no-srt` — YouTube only provides VTT. Use `_vtt_to_srt()` converter
- `ffmpeg-not-found` — Auto-detect from WinGet path, set `ffmpeg_location` in opts
- `explorer-backslash` — Use `data-*` + `addEventListener`, never inline `onclick` with paths
- `trending-error` — Add `ignoreerrors: True`, filter `None` entries

### 2. Architecture Overview

```
Browser (app.js)                    Flask (app.py)
  |                                   |
  |-- fetch("/api/search") ---------> search_videos()
  |-- fetch("/api/info") -----------> get_info() → yt-dlp extract_info
  |                                   |  └─ populates subtitle_url_store
  |-- fetch("/api/subtitle") -------> get_subtitle() → _get_cached_subtitle(vtt)
  |-- fetch("/api/download") -------> start_download() → spawn thread
  |-- EventSource("/api/progress") -> SSE generator polls progress_store
  |-- <video src="/api/stream"> ----> proxy YouTube CDN (720p combined)
  |-- Shaka Player → /api/mpd ------> DASH manifest (1080p+ separate streams)
```

### 3. Subtitle System (Critical Path)

```
/api/info ──► subtitle_url_store[video_id] = {urls: {lang: {vtt: url}}}
                          │
/api/subtitle ──► _get_cached_subtitle(url, lang, "vtt", video_id)
                          │
                  ┌───────┴────────┐
                  │ subtitle_cache  │  ← Content cache (30min TTL)
                  │ (video_id,lang) │
                  └───────┬────────┘
                          │ miss
                  ┌───────┴────────┐
                  │ _fetch_subtitle │  ← Single HTTP GET (no retries)
                  │     _url()      │
                  └───────┬────────┘
                          │ 429?
                  ┌───────┴────────┐
                  │ _subtitle_429  │  ← Global cooldown (5min)
                  │    _until      │     All fetches skip until expired
                  └────────────────┘
```

### 4. Download Flow

| Step | Status | Backend | Frontend |
|------|--------|---------|----------|
| 1 | `starting` | Create task_id, spawn thread | Open EventSource |
| 2 | `downloading` | yt-dlp + progress_hooks | Update bar/speed/ETA |
| 3 | `processing` | FFmpeg merge/extract | Show "Processing..." |
| 4 | `burning_subtitles` | FFmpeg subtitle filter | Show "Burning..." |
| 5 | `done` | Move file to save_dir | Green bar, add history |
| X | `error` | Catch exception | Show error, enable button |

### 5. Dark Theme Tokens

| Token | Value | Usage |
|-------|-------|-------|
| Body BG | `#0f0f0f` | Page background |
| Panel BG | `#161616` | Sidebars |
| Card | `#1a1a1a` / `#1e1e1e` | Cards, inputs |
| Border | `#2a2a2a` | Dividers, borders |
| Accent Red | `#ff4444` | Buttons, focus |
| Accent Green | `#22c55e` | Download, success |
| Text | `#e8e8e8` → `#888` → `#555` | Primary → tertiary |
| Radius | `8px` (buttons) / `12px` (panels) | Border radius |

### 6. yt-dlp Common Patterns

| Pattern | Key Options | When |
|---------|-------------|------|
| Fast search | `extract_flat: "in_playlist"` | Keyword/channel search |
| Full search | `ignoreerrors: True` | Trending (needs upload_date) |
| Video DL | `format: f"bestvideo[height<={h}]+bestaudio"` | MP4 download |
| Audio DL | `postprocessors: [{key: FFmpegExtractAudio}]` | MP3 download |
| Locale | `extractor_args: {youtube: {hl, gl}}` | Language-filtered search |
| JS fix | `js_runtimes: {"node": {}}` | PhantomJS conflict on Windows |

---

## How to Use This Skill

### Step 1: Identify the Problem Domain

| You need to... | Domain | Example query |
|----------------|--------|---------------|
| Add/modify API endpoint | `api` | "subtitle download endpoint" |
| Find/add UI component | `ui` | "progress bar button" |
| Debug a known issue | `pitfalls` | "429 rate limit subtitle" |
| Configure yt-dlp options | `ytdlp` | "download format mp4 quality" |
| Look up design values | `design` | "accent color border radius" |
| Understand subtitle flow | `subtitle` | "cache cooldown vtt srt" |
| Understand download flow | `download` | "progress status burn subtitle" |

### Step 2: Search the Knowledge Base

**Single domain search (targeted):**
```bash
python .claude/skills/yt-dl-dev/scripts/search.py "429 subtitle cooldown" --domain pitfalls
```

**Auto-detect domain (convenient):**
```bash
python .claude/skills/yt-dl-dev/scripts/search.py "subtitle 429 rate limit"
```

**Cross-domain overview (broad):**
```bash
python .claude/skills/yt-dl-dev/scripts/search.py "subtitle download" --overview
```

**List all domains:**
```bash
python .claude/skills/yt-dl-dev/scripts/search.py --domains
```

### Step 3: Apply Patterns

After finding relevant results, follow the project conventions:

#### Adding a New API Endpoint

```python
# Backend (app.py)
@app.route("/api/new-endpoint", methods=["POST"])
def new_endpoint():
    data = request.get_json()
    param = data.get("param", "").strip()
    if not param:
        return jsonify({"error": "param is required"}), 400
    try:
        # ... logic ...
        return jsonify({"result": value})
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

```javascript
// Frontend (app.js)
async function newFeature() {
    try {
        const res = await fetch("/api/new-endpoint", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ param: value }),
        });
        const data = await res.json();
        if (!res.ok) {
            showError(data.error || "Failed");
            return;
        }
        // Update DOM...
    } catch (err) {
        showError("Network error");
    }
}
```

#### Adding a UI Component (Dark Theme)

```html
<!-- Use existing CSS classes, dark theme tokens -->
<div class="result-card" data-value="..." title="...">
    <img class="result-thumb" src="..." loading="lazy">
    <div class="result-info">
        <div class="result-title">...</div>
        <div class="result-meta">...</div>
    </div>
</div>
```

```css
/* Follow token system */
.new-component {
    background: #1e1e1e;     /* Card Secondary */
    border: 1px solid #2a2a2a; /* Border */
    border-radius: 8px;       /* Radius Button */
    color: #e8e8e8;           /* Text Primary */
    transition: border-color 0.15s; /* Transition Hover */
}
.new-component:hover {
    border-color: #ff4444;    /* Accent Red */
}
```

#### Working with Subtitles (CRITICAL)

```python
# ALWAYS use _get_cached_subtitle() — never call yt-dlp for subtitles
content = _get_cached_subtitle(url, lang, "vtt", video_id=vid)

# For SRT: fetches VTT then converts automatically
srt_content = _get_cached_subtitle(url, lang, "srt", video_id=vid)

# Check 429 cooldown before returning error
if not content:
    if time.time() < _subtitle_429_until:
        return jsonify({"error": "rate_limited", "retry_after": int(_subtitle_429_until - time.time())}), 429
    return jsonify({"error": f"No subtitle found"}), 404
```

```javascript
// Frontend: always pass video_id from /api/info response
body: JSON.stringify({ url: currentUrl, lang, video_id: currentVideoId })

// Handle 429 response
if (res.status === 429) {
    const data = await res.json().catch(() => ({}));
    const mins = Math.ceil((data.retry_after || 300) / 60);
    showError(`YouTube 字幕限速中，请 ${mins} 分钟后再试`);
}
```

---

## Search Reference

### Available Domains

| Domain | File | Entries | Use For | Example Keywords |
|--------|------|---------|---------|------------------|
| `api` | api-reference.csv | 14 | Endpoint details, params, responses | endpoint, subtitle, download, stream, search |
| `ui` | ui-components.csv | 40+ | DOM elements, CSS classes, events | button, select, progress, panel, player |
| `pitfalls` | pitfalls.csv | 16 | Common bugs, causes, fixes | 429, ffmpeg, backslash, thread, encoding |
| `ytdlp` | yt-dlp-patterns.csv | 12 | yt-dlp option patterns | format, search, download, flat, locale |
| `design` | design-tokens.csv | 35 | Colors, spacing, typography, animation | color, radius, font, transition, grid |
| `subtitle` | subtitle-system.csv | 13 | Subtitle caching, 429 cooldown, conversion | cache, cooldown, vtt, srt, shaka, burn |
| `download` | download-flow.csv | 12 | Download states, progress, SSE | progress, status, thread, merge, sse |

---

## Project File Map

| File | Lines | Purpose |
|------|-------|---------|
| `app.py` | ~2300 | Flask server: all routes, yt-dlp logic, subtitle cache, download worker |
| `static/app.js` | ~1100 | Frontend: search, download, progress SSE, subtitle player, i18n, history |
| `templates/index.html` | ~140 | Single page: 3-column layout, Shaka Player CDN, data-i18n attributes |
| `static/style.css` | ~940 | Dark theme: #0f0f0f base, responsive grid, animations |

## Global State (app.py)

| Variable | Purpose |
|----------|---------|
| `progress_store` | Download progress per task (thread-safe with `lock`) |
| `file_store` | Downloaded file paths per task |
| `subtitle_cache` | Content cache: `{(video_id, lang, fmt): {content, time}}` |
| `subtitle_url_store` | URL cache: `{video_id: {urls, title, time}}` |
| `_subtitle_429_until` | Global 429 cooldown timestamp (5min) |
| `_mpd_range_cache` | MP4 probe results for DASH manifest |

## Global State (app.js)

| Variable | Purpose |
|----------|---------|
| `currentUrl` / `currentVideoId` | Selected video URL and ID |
| `currentTitle` / `currentKeyword` | Video title and search keyword |
| `shakaPlayer` | Shaka Player instance (DASH/1080p+) |
| `subtitleVttCache` | Client-side VTT blob URL cache per language |
| `suggestionsData` | Cached keyword + channel suggestions |
| `LANG` / `currentLang` | i18n translations (en, zh, ja) |

---

## Pre-Delivery Checklist

### Backend
- [ ] Route returns proper JSON error with status code (400/404/429/500)
- [ ] Thread-safe access to shared stores (use `lock`)
- [ ] File paths use `os.path.normpath()` on Windows
- [ ] yt-dlp opts include `ffmpeg_location: FFMPEG_DIR`
- [ ] Subtitle access via `_get_cached_subtitle()` only — no yt-dlp calls
- [ ] 429 cooldown check before returning subtitle errors

### Frontend
- [ ] User content escaped with `escapeHtml()` / `escapeAttr()`
- [ ] Dynamic elements use `data-*` + `addEventListener` (no inline onclick)
- [ ] Async buttons disabled during operation, re-enabled in finally
- [ ] i18n keys added to `LANG` for all 3 languages (en, zh, ja)
- [ ] Errors shown via `showError()` with translated messages
- [ ] Blob URLs revoked when no longer needed

### Design
- [ ] Uses dark theme tokens (not custom colors)
- [ ] Hover states with `transition: 0.15s`
- [ ] Border radius: 8px (buttons) / 12px (panels)
- [ ] Responsive at 1024px breakpoint (single column)
- [ ] `.hidden` class for toggling visibility
