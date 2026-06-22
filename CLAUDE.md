# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static, no-build Korean lotto number generator (`index.html` is the current main page; `lotto.html` is an earlier, simpler version kept for reference). There is no package manager, no bundler, and no test framework — open the HTML files directly in a browser.

## Running the App

```bash
# Serve locally (any static server works)
python3 -m http.server 8080
# then open http://localhost:8080
```

There are no build, lint, or test commands.

## Architecture (index.html — v16)

All logic lives in a single `<script>` block at the bottom of `index.html`.

### Data Flow

1. **`latestNo()`** — Estimates the current draw number from the fixed epoch `2002-12-07` (draw #1), then validates against the DH Lottery API, counting down until it gets a success response.
2. **`generate()`** — Main entry point (called on `DOMContentLoaded` and on button click). Fetches the last 100 draw results via `Promise.allSettled`, builds a frequency map, then renders all sections and the bar chart.
3. **`fetchJSON(url)`** — Wraps the DH Lottery API (`https://www.dhlottery.co.kr/common.do`) through a three-proxy fallback chain to handle CORS. Proxies are tried in order:
   - `thingproxy.freeboard.io`
   - `api.allorigins.win`
   - `corsproxy.io`

### Key Utilities

| Function | Purpose |
|---|---|
| `colorClass(n)` | Maps ball number to CSS class: 1–10 → `r1` (red), 11–20 → `r2` (orange), 21–30 → `r3` (green), 31–40 → `r4` (blue), 41–45 → `r5` (purple) |
| `ball(n)` | Returns an `<span>` colored circle HTML string |
| `pick(pool)` | Fisher-Yates–style random selection of 6 numbers from a pool |
| `score(set, freqMap)` | Sums historical frequency scores for a set of 6 numbers |
| `renderSetWithScore()` | Renders number sets with their frequency score label |

### Sections Rendered

- **Last draw** — most recent official numbers with a progress bar while loading
- **Recommended (3 sets)** — picked from the top-100 most-frequent numbers across the last 100 draws
- **Random (2 sets)** — picked from the full 1–45 pool; both shown with frequency scores
- **Top-6 frequency** — the 6 highest-frequency numbers + a Chart.js bar chart of all frequencies
- **Share** — Kakao Talk (uses hardcoded app key `KAKAO_APP_KEY`), Twitter/X, clipboard copy
- **Screenshot** — html2canvas captures `document.body` and downloads as `lotto.png`

### External CDN Dependencies

- [Chart.js](https://cdn.jsdelivr.net/npm/chart.js) — bar chart
- [html2canvas 1.4.1](https://cdn.jsdelivr.net/npm/html2canvas@1.4.1) — PNG screenshot
- [Kakao JS SDK 2.5.0](https://t1.kakaocdn.net/kakao_js_sdk/2.5.0/kakao.min.js) — social sharing

### lotto.html vs index.html

`lotto.html` is the original prototype. Key differences from `index.html`:
- Uses a single CORS proxy (allorigins only, no fallback)
- Fetches **all** historical draws (not just the last 100), making it much slower
- Pool size is top-40 (not top-100)
- No Chart.js visualization, no sharing, no screenshot, no color-coded balls
- No frequency score display
