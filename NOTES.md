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

## Resource hints: preload vs prefetch

- **Preload scanner** (built-in, automatic): while a plain script blocks parsing, a light reader skims ahead in the HTML and starts downloading later `<link>`/`<script src>`/`<img>`. Downloads only.
- `<link rel="preload" href="font.woff2" as="font" crossorigin>` (written by me): starts downloading a file **hidden inside CSS/JS** (fonts, JS-loaded images) from the HTML. Downloaded **once**, waits in cache until the CSS/JS asks for it.
  - Preload doesn't make things non-blocking. It makes them **start sooner**.
- `rel` = **why** you link. Fixed vocabulary from the HTML spec (check MDN). Wrong or unknown value → **silently ignored**. `<link>` uses `href`, not `src`.
  - `stylesheet` on a font → read as CSS, fails, wasted. `preconnect` → only opens a connection.
- **preload** = sure + soon, **this** page, high priority. **prefetch** = probably, later (the **next** page, e.g. `/cart` → `/checkout`), low priority, only when idle.
- Background = doesn't block. **Priority = who gets the network pipe first.** A high-priority prefetch would steal bandwidth from the current page.
- Prefetched files land in the cache (usable by the current page), but low priority → may arrive too late. Wrong tool for now-needs.
- HTML fails silently → verify in DevTools → Network (did it load, when, what priority).
