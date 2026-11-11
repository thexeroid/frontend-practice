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
