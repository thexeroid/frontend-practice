# Frontend answers (in my own words)

One section per question. Short.

## `<script>` vs `async` vs `defer`

- Plain `<script>`: parsing **stops**, script downloads, runs, then parsing continues.
  - In `<head>` → user sees a **blank page** (body not parsed yet) + `querySelector` returns `null` → crash on use.
- `async`: downloads in parallel, runs **as soon as it arrives** (pauses parsing only while running).
  - **No order guarantee** → a small `app.js` can run before the big lib it needs. DOM may not be ready.
  - Use for independent scripts: analytics, ads, chat widgets.
- `defer`: downloads in parallel, runs **after parsing ends** (before `DOMContentLoaded`), **in written order**.
  - DOM ready + order kept → the safe default for app code.
- `type="module"` scripts are deferred by default.
- `async`/`defer` on an inline script (no `src`) are **silently ignored** → it runs right away → DOM access crashes, no warning.
  - Fix: inline `type="module"` (inline modules *are* deferred), or `DOMContentLoaded`, or put it at the end of `<body>`.

| | Blocks parsing? | Runs when | Order kept? |
|---|---|---|---|
| `<script>` | yes | right away | yes |
| `async` | only while running | when downloaded | no |
| `defer` | no | after parsing | yes |

## CSS `<link>` in `<head>`, JS before `</body>` (and why `defer` beats both)

- Browser pipeline: HTML → **DOM**, CSS → **CSSOM** → render tree → layout → **paint** (= drawing pixels).
- The DOM needs no CSS (it's just structure). Only **painting** needs both.
- 🎨 **First paint = partial DOM + complete CSSOM.** CSS can override anything anywhere, so a half CSSOM = page "disco dancing".
- CSS download does **not** block HTML parsing. When it arrives, the main thread pauses briefly to parse it, then resumes.
- CSS **blocks painting** → bigger CSS = longer blank screen. Chrome fetches CSS at Highest priority.
- CSS at the bottom → page painted unstyled, then jumps = **FOUC** (flash of unstyled content). Fonts swapping = FOUT.
- Fixes: critical CSS inline + rest later · `media="print"` doesn't block screen paint · remove unused CSS.
- ⚠️ A plain `<script>` waits for the CSS above it (it might read styles) → slow CSS can **indirectly** block parsing.
- Plain `<script>` before `</body>`: page parsed + painted first (perceived speed) + DOM elements exist (no `null` crash).
- **Even better: `<script defer src>` in `<head>`**: download overlaps parsing, runs right when parsing ends. Bottom script = download only starts at the end.
  - Rule: **find resources early, download in parallel, run late.**
  - The preload scanner (reads ahead while parsing is blocked) narrows the gap, but `defer` in `<head>` is the guaranteed way.
- One main thread per tab runs JS + builds DOM/CSSOM + style + layout. Block it → the page freezes. Network, raster, compositing run elsewhere.
- Check: DevTools → Performance (the "Main" row) · Network → Priority column · Lighthouse → "render-blocking resources".
