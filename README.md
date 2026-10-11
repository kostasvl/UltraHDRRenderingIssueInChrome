# Tile seams on gain-map (UltraHDR) images in Chromium

Chromium draws thin dark lines across an UltraHDR photo at particular render sizes. This folder is the investigation record and a ready-to-file bug report.

Found while building a Windows HDR photo viewer that delegates image decoding to
Chromium via WebView2. Nothing here is specific to that app — it reproduces in stock Chrome with a
plain HTML page.

The aforementioned app, as well as most of the info in this repository, including the repro html, were all created with the help of Claude Code.

| | |
|---|---|
| **Affected** | Chromium 151.0.4129.72 (also Chrome 155.0.8059.40) — Microsoft Edge, Edge WebView2, Vivaldi |
| **Firefox** | no seams — but it does not support UltraHDR, so it shows the SDR base image. A weak comparison, not a control |
| **Requires** | an HDR gain map in the image; SDR files of identical dimensions never show it |
| **Trigger** | a freshly rasterized image arriving from a transition; size-dependent, rule unknown |

## Contents

| Path | What it is |
|---|---|
| `repro/index.html` | Self-contained reproduction: use in-page UI to load 2+ gain-map JPEG, then transition between the images (with arrow keys). If no no black lines appear during the transition, step the height by 1 or 4 px, and repeat.  |
| `repro/*.jpg` | Two UltraHDR photos that demonstrate the problem. Others can be used as well.  |
| `evidence/` | Screenshot and videos that capture the black lines. The video may need frame stepping. |

## What is established

- **A gain map is required.** Flat pale-gradient SDR JPEGs at matched dimensions — built specifically
  so a one-pixel line cannot hide — are clean at every size tried, up to 4400 px.
- **A fresh rasterization is required.** A static, settled image never shows it. The image has to be
  arriving, via a cross-fade or `src` swap.
- **Transient, about a second**, then it resolves by itself — except under `transform: scale()`,
  where the seams are scaled along with the already rasterized tiles and become permanent.
- **Size-dependent.** Some render sizes are consistently affected, others consistently clean, and a
  one-pixel change can flip it. Number, position and thickness all vary with size.
- **Content hides it.** On a busy photograph the lines are effectively invisible, which makes casual
  testing unreliable in both directions.

## What is not established

**No rule that predicts which sizes are affected.** An arithmetic hypothesis — that seams appear when
a source tile boundary maps to a whole output pixel, i.e. render heights divisible by 4 for a
4096 px-tall photo — fitted four measurements neatly and was then **falsified by a fifth**: a window
size that visibly reproduced while the rule predicted clean.

## How it was narrowed down

Recorded because most of it was wrong, and the wrong turns are the useful part.

**Ruled out, none of which changed anything:** `img.decode()` before display; `decoding="sync"`;
removing the CSS cross-fade; a two-frame invisible warm-up before revealing; sizing by layout box
versus `transform: scale()`; wrapping the image in a positioned div versus not; rounding the rendered
size and centring offsets to whole pixels; `--disable-checker-imaging`; `--disable-partial-raster`;
`--default-tile-width/height=8192`; `--max-untiled-layer-width/height=8192`; and
`dynamic-range-limit: standard` versus `no-limit`.

`--disable-gpu-rasterization` cannot be evaluated — it crashes WebView2 with `STATUS_BREAKPOINT`.
