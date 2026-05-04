# UPScore — CLAUDE.md

Single-file SPA prototype. All HTML, CSS, and JS live in `index.html`. No build step, no framework. Deploy via git push to `main` → Vercel auto-deploys.

Live URL: https://upscore-pi.vercel.app/
Repo: https://github.com/abhijaytondak/upscore

---

## Screen map

| Index | Name | Notes |
|-------|------|-------|
| 0 | S1 — The Hook | Story auto-progress (6 s) |
| 1 | S2 — The Metric | Story auto-progress |
| 2 | S3 — Aha (peers) | Story auto-progress |
| 3 | S4 — Solutions | Story auto-progress |
| 4 | S5 — Rewards | Explicit CTA only (no auto-advance) |
| 5 | Trust Layer | "I'm ready — let's go" |
| 6 | Q1 Income | Slider |
| 7 | Q2 Fixed | Slider |
| 8 | Q3 Lifestyle | Chip multi-select |
| 9 | Q4 Identity | Chip multi-select → Loading |
| 10 | Loading | 8 s RAF animation → auto-advances to 11 |
| 11 | Score Reveal | Count-up animation |
| 12 | Dashboard | Main LP with locked cards + bank-connect sheet |
| 13 | Insights carousel | Swipe/drag |
| 14 | Missions | Final screen |

`TOTAL = screens.length` = **15**. Always keep this accurate.

---

## Navigation architecture

### Data attributes (HTML)
- `data-next` — advance one screen (`show(current + 1)`)
- `data-prev` — go back one screen (`show(current - 1)`)
- `data-skip` — always jump to Trust Layer (`show(5)`)
- `data-goto="N"` — jump to a specific screen index; use for CTAs whose destination must be deterministic regardless of `current` (e.g. Q4 "calculate my score →" uses `data-goto="10"`)
- `data-chip` — toggles `.on` class, does NOT navigate
- `data-bank` / `data-bank-close` / `data-bank-go` — controls the bank-connect bottom sheet on the Dashboard screen only

### JS handlers (end of `<script>`)
Two-layer approach: delegated listener on `document` + direct belt-and-suspenders listeners on every `[data-next/prev/skip/goto]` element at load. All direct handlers call `e.stopPropagation()` to prevent double-firing.

### `show(i)` lifecycle
```
show(i)
  → screens toggle .active
  → current = i
  → onLeave(prev)  // kills timers
  → onEnter(i)     // starts timers / animations
```

---

## Key CSS rules

- `.screen` — `position: absolute; inset: 0; pointer-events: none; opacity: 0`
- `.screen.active` — `pointer-events: auto; opacity: 1`
- `.pad` — `height: 100%; display: flex; flex-direction: column` — **no overflow-y** (removing it fixed scroll-vs-click cancellation on iOS)
- `.btn` — has `touch-action: manipulation` (prevents iOS scroll-cancel on tap)
- `.stage` — `390×844 px` desktop; `100vw×100vh` on mobile (≤430 px)
- `.zones` — `position: absolute; inset: 0; z-index: 3` — only on story screens (0–3), not on question screens
- `.home` — iOS home indicator bar, hidden on mobile via media query

---

## Common pitfalls / past bugs

| Bug | Root cause | Fix |
|-----|-----------|-----|
| X/close buttons dead | Missing `data-next` attribute | Added `data-next` to all 9 header X buttons |
| Q1–Q4 CTAs not tapping on iOS | `.pad { overflow-y: auto }` → browser cancelled touch-as-scroll | Removed overflow-y from `.pad` |
| Q4 "calculate my score →" unreliable | `data-next` depends on `current` being correct; touch drift could cancel | Changed to `data-goto="10"` + `touch-action: manipulation` on `.btn` |
| Story screens X buttons jumping to Trust Layer | Used `data-skip` instead of `data-next` | Fixed to `data-next` (sequential advance) |

---

## Animations

- **Story progress** (screens 0–3): RAF-based 6 s bars. Screen 4 (S5) has explicit CTA only.
- **Loading ring** (screen 10): 8 s RAF animates `stroke-dashoffset` 754→0; auto-calls `show(11)` on complete. IDs: `lringStroke`, `lringPct`.
- **Score count-up** (screen 11): 1.5 s ease-out from 0 to 62. ID: `rnum`.
- All durations collapse to 0 when `prefers-reduced-motion: reduce`.

---

## Deploy

```bash
git add index.html
git commit -m "..."
git push origin main   # Vercel picks up automatically
```

No `.env`, no secrets, no build. `vercel.json` is minimal routing only.
