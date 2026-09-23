# Handoff: Flores Amarillas (página romántica interactiva)

## Overview
Single-page site, full viewport, no scroll. Yellow flowers grow from the bottom (stems draw in, leaves pop, petals bloom layer by layer). Every petal is clickable: a heart burst appears at the cursor, then a fullscreen overlay shows a love / encouragement message in Spanish. Language of all UI copy: Spanish.

## About the Design Files
`Flores Amarillas.dc.html` is a **design reference built in HTML** (a prototype showing intended look and behavior), not production code. Recreate it in the target stack. If none exists, a good fit is **Vite + vanilla TS or React, with inline SVG + CSS keyframes** (no heavy libs needed). The file opens directly in a browser (needs `support.js` beside it) — use it to compare. All geometry/animation logic lives in the `class Component` script inside the file; port it nearly 1:1.

## Fidelity
**High-fidelity.** Match colors, fonts, timings, and easing exactly.

## Screen: Main (only screen)
Root: `position:relative; width:100%; height:100vh; overflow:hidden; user-select:none`
Background: `radial-gradient(120% 80% at 50% 110%, #3a2a12 0%, #1a140c 45%, #0f0c08 100%)`; body bg `#0f0c08`.

Layers (bottom → top):
1. **SVG scene** — absolute inset 0, 100%×100%, `preserveAspectRatio="xMidYMax meet"`, viewBox `0 0 W 1000` where `W = clamp(700, 1000 * innerWidth/innerHeight, 2400)`. Recompute on resize.
2. **Pollen** — 30 dots, absolute, `pointer-events:none`. Size 2–5px, `#FFE38A`, `box-shadow: 0 0 8px 2px rgba(255,220,120,.55)`, left 0–100%, top 40–105%, `drift` 9–18s linear infinite with negative random delay.
3. **Header** (absolute top, flex space-between, padding 28px 32px, z 5, fades in 2s delay .6s):
   - Title: “Para ti, {nombre}” — Cormorant Garamond italic, `clamp(28px,4vw,44px)`, line-height 1, `#FFE7A3`.
   - Counter under it (gap 6px): Jost 12px, letter-spacing .22em, uppercase, `#BFA982`. Text: `"40 mensajes escondidos"` when 0 seen, else `"{n} de 40 mensajes descubiertos"`.
   - Button right: “Florecer de nuevo” — Jost 13px, .12em, uppercase, `#FFE7A3`, transparent bg, border `1px solid rgba(255,214,110,.35)`, radius 999px, padding 10px 18px; hover bg `rgba(255,214,110,.12)`, transition .3s.
4. **Hint** (after bloom finished, hidden while overlay open): bottom 28px, centered, “Toca cada pétalo. Cada uno guarda algo que quiero que sepas.” Cormorant italic `clamp(18px,2.4vw,24px)`, `#E9D6A8`, fadeIn 1.6s.
5. **Heart bursts** — fixed layer, z 15, pointer-events none.
6. **Message overlay** (z 20) — see Interactions.

## Flower generation (seeded, deterministic)
PRNG: mulberry32, seed `11 + n` (n = number of flowers, default 7). H = 1000, `sc = clamp(W/1500, .62, 1.1)`, speed `sp` (default 1). Random stagger order = indices shuffled with the PRNG.
Per flower i:
- `fx = W*(0.08 + 0.84*(i+.5)/n) + (r-.5)*W*.5/n`; `cy = H*(0.3 + r*.34)`; `R = (80 + r*58)*sc`
- Stem: quadratic from `(x0 = fx + (r-.5)*140*sc, H+10)` via control `((x0+fx)/2 + (r-.5)*180*sc, (H+10+cy)/2)` to `(fx, cy)`. Stroke `#4B7A2E`, width `max(4, R*.075)`, round cap.
- `delay = (0.3 + order*0.45)/sp`, `stemDur = 2/sp`, `head = delay + stemDur*.85`.
- Leaves at t = .42 and .68 on the curve; rotation = tangent angle ± 62° (alternating side); length `R*(.85 + r*.3)`; path `M0,0 C L*.25,-L*.3 L*.7,-L*.32 L,0 C L*.7,L*.18 L*.25,L*.2 0,0Z`; fill linear gradient `#2E5520 → #6E9A3E`.
- Draw order: sort flowers by `cy` ascending (farther ones first).

Flower head (translated to fx,cy; random base rotation 0–360°):
- Halo circle r = 1.9R, radial `rgba(255,205,70,.34) → 0`, fades in 2.5s at head+.6s. Not clickable.
- 3 petal layers (outer → inner):
  - A: 20 petals, length 1.0R, width .20R, gradient `#D97800 → #FFD23F`
  - B: 16 petals, length .80R, width .20R, offset half step, `#EE9A00 → #FFE066`
  - C: 12 petals, length .58R, width .19R, offset one step, `#F5B000 → #FFEC8A`
  - Gradients go from the flower center (base) to the petal tip.
  - Petal path (points up from origin): `M0,0 C w,-L*.3 w*.85,-L*.85 0,-L C -w*.85,-L*.85 -w,-L*.3 0,0Z`; stroke `rgba(130,70,0,.28)` .7px. Center vein line from -.12L to -.82L, `rgba(190,110,0,.35)` .8px.
  - Petal already read → lighter gradients: A `#FFB627→#FFF3B8`, B `#FFC640→#FFF6C8`, C `#FFD45A→#FFFADB` (fill transition .6s).
- Disc r = .3R, radial `#A8621A → #4E2A08`.
- Seeds: 63 dots in a phyllotaxis spiral (k=1..63, radius `c*sqrt(k)`, `c = .3R*.9/8`, angle `k*2.39996` rad), r = .065·disc, colors `#3A1F05` / every 3rd `#6B3B0C`.

## Animations (keyframes)
- `grow` (stem): stroke-dashoffset 1→0 with pathLength=1; `stemDur`, `cubic-bezier(.45,.1,.3,1)`, starts at `delay`.
- `pop` (leaves, disc, seeds): scale 0→1.08 (70%)→1, opacity 0→1. Leaves 1.2s ease-out; disc .8s at `head`; seeds .6s at `head + (.3 + k*.012)`.
- `bloom` (petals): `0% rotate(-28deg) scale(.15,0) opacity 0 → 65% rotate(2deg) scale(1.04,1.06) opacity 1 → 100% none`. 1.5s `cubic-bezier(.2,.8,.2,1)`, delay `head + (layer*.25 + j*.035)`. Transform origin = flower center.
- `breathe` (whole head): scale 1→1.025 + rotate 1.5° and back, 6s ease-in-out infinite, starts `head + 3s`.
- All durations/delays divided by `sp`.
- Bloom complete (show hint) at `max(head) + (0.5 + 12*.035 + 1.5)/sp`.

## Interactions & Behavior
- **Petal hover:** scale 1.13 from flower center + `brightness(1.18)`, transition .35s ease; cursor pointer.
- **Petal click:** (ignored while overlay open)
  1. Mark the petal as read; increment count.
  2. Heart burst at cursor: 14 “♥” glyphs, 12–26px, colors `#FFD23F` (2 of 3) / `#FF8A7A`, text-shadow `0 0 10px rgba(255,200,80,.6)`; each flies outward 50–120px (radially, biased -30px up), random rotation ±45°, fades out, 1–1.5s `cubic-bezier(.2,.7,.3,1)`. Remove after 1.6s.
  3. After 550ms open the overlay with message `MENSAJES[(count-1) % 40]` (messages go in order and wrap after 40).
- **Overlay:** fixed inset 0, bg `rgba(12,9,5,.74)`, backdrop blur 8px, fadeIn .5s. Centered column, max-width 760px, gap 28px, `rise` 1s (translateY 24px + scale .97 → none) delay .1s.
  - Label “Mensaje {n} de 40” — Jost 12px, .28em, uppercase, `#C9AE72`.
  - 10px diamond `#FFD23F` rotated 45°, glow `0 0 24px 6px rgba(255,210,63,.45)`.
  - Message — Cormorant Garamond italic 500, `clamp(32px,5.4vw,60px)`, line-height 1.15, `#FFF4D6`, `text-wrap:balance`.
  - Button “Seguir descubriendo” — Jost 13px, .14em, uppercase, text `#1a140c`, bg `#FFD23F` (hover `#FFE27A`), radius 999px, padding 14px 26px.
  - Clicking the backdrop or button closes it.
- **Florecer de nuevo:** remount the SVG (restart all animations), clear read petals, reset count to 0, close overlay, hide hint and re-schedule it.
- **Responsive:** handled by the viewBox width formula plus `meet` scaling; flowers overlap into a bouquet on portrait screens.

## State
`read: Set<petalId>` (id = `flower-layer-index`), `count`, `open`, `msg`, `msgNum`, `bursts[]`, `bloomed`, `run` (remount key), `aspect`.

## Config (tweakable)
- `nombre` (string, default “mi amor”) — shown in the title.
- `flores` (int 3–12, default 7).
- `velocidad` (0.5–2, default 1).

## Messages (40, in order — copy verbatim)
Listed in the `MENSAJES` array at the top of the script in `Flores Amarillas.dc.html`.

## Design Tokens
- Background: `#0f0c08`, `#1a140c`, `#3a2a12`
- Text: `#FFF4D6`, `#FFE7A3`, `#E9D6A8`, `#C9AE72`, `#BFA982`
- Accent: `#FFD23F` (hover `#FFE27A`), heart pink `#FF8A7A`
- Stem `#4B7A2E`; leaf `#2E5520→#6E9A3E`
- Fonts (Google Fonts): Cormorant Garamond (500, italic 400/500); Jost (300/400/500)
- Radius: 999px (pills)

## Assets
No images. Everything is SVG generated in code, plus Google Fonts.

## Files
- `Flores Amarillas.dc.html` — the prototype (open in a browser; `support.js` must be in the same folder)
- `support.js` — runtime for the prototype only; do not ship it
