# NF3D Anaglyph Engine

**Turn any photo into red-cyan 3D anaglyph — runs entirely in your browser.**

Drop an image in, put on your glasses, adjust a few sliders. No install, no server, no account. Works offline after the first run.

Looks like an odd filter without glasses. Looks like proper depth with them. You don't need anything fancy — cardboard ones from a DVD are fine.

---

## Setup

Download both files and keep them in the same folder:

- **[NF3D Anaglyph Engine.html](NF3D%20Anaglyph%20Engine.html)**
- **[nf3d_log.png](nf3d_log.png)** ← logo file, must be alongside the HTML or the logo box will be empty

Open `NF3D Anaglyph Engine.html` in a modern browser (Chrome, Edge, or Firefox). No web server needed.

---

## First use

On first image load the app downloads **Depth Anything V2 Small** (~50 MB) from Hugging Face and caches it in the browser. The depth proxy thumbnail shows download progress. All subsequent uses are instant — the model stays cached unless you clear browser storage.

If the model fails to load (no internet, corporate firewall) the app falls back to a luminance + Y-gradient depth estimate, which still produces a usable anaglyph.

---

## How it works

1. Drop a JPG, PNG, or WEBP onto the app
2. Depth Anything V2 analyses the image and produces a depth map — closer objects brighter, distant objects darker
3. A WebGL shader displaces left and right eye views horizontally based on that depth map. An occlusion guard prevents background pixels from stretching over foreground edges at depth discontinuities
4. The two views are combined through a Dubois colour matrix into a red-cyan anaglyph. The entire colour pipeline — far-field haze, rivalry dampening, red balance, and all anaglyph modes — operates in linear light (sRGB decoded once at the start, re-encoded once at the end) for physically correct colour blending
5. Adjust sliders to taste, then save as PNG

---

## Controls

### Parallax Engine
| Control | What it does |
|---|---|
| Baseline separation | How far left and right eye views are shifted apart — the primary depth strength control |
| Convergence plane | Sets which depth sits at the screen plane; objects in front pop out, behind recede |
| Near-field boost | Emphasises foreground depth |
| Mid-plane bridge | Blends luminance depth with a top-bottom gradient (active only without ML depth) |
| Max disparity | Hard pixel cap on eye separation — prevents extreme values causing eye strain |
| Floating window | Adds per-eye black borders at the frame edge to prevent objects being cut off by the screen frame |

### Dubois & Spectral
| Control | What it does |
|---|---|
| Mode | **RC DUBOIS** — optimised red-cyan matrix, best general use. **AMBER-BLUE** — better for warm or red-heavy images. **HALF-COL** — greyscale left eye, colour right eye, minimal ghosting |
| Wimmer half-col | Checkbox (HALF-COL mode only) — switches the left-eye luminance to Wimmer's optimised weighting (0.7G + 0.3B) which preserves reds that standard luma crushes. Off by default; worth comparing on images with vivid warm colours |
| Rivalry dampener | Desaturates each eye view slightly in linear light to reduce binocular colour rivalry |
| Ghost reduction | Reduces colour leakage between channels |
| Magenta–teal bias | Fine-tunes the colour balance of the composite |
| Red balance | Scales the red channel in linear space — reduce for warm/sunset images |

### Far-Field Recession
Attenuates and hazes distant areas to reinforce depth perspective.

### Edge Sculpting
Sobel edge detection with depth-weighted unsharp masking — boosts local colour contrast at object boundaries, with stronger enhancement toward the foreground. This reinforces depth separation rather than just adding a tint.

### Iridescence Noise
Rainbow shimmer noise layer — optional creative effect.

### Temporal Pulse
Animates the depth in a slow sine wave. The **⏸ pause pulse** / **▶ start pulse** button controls playback.

### Depth controls (below thumbnails)
These appear once the ML depth model has run.

| Control | What it does |
|---|---|
| Smooth | Edge-aware bilateral filter on the depth map, guided by source image luminance — smooths depth across flat regions without bleeding across colour edges. Fires on release (values above 5 are noticeably slow) |
| Sharpen | Unsharp mask applied to the depth map itself — sharpens depth edges without involving luminance, no false geometry risk |
| Luma det | High-pass luminance injected into depth — recovers fine image detail (hair, bark, clothing folds) that the ML model smooths over. Fires on release |
| Lum blend | Real-time shader slider: blends between pure ML depth (100%) and raw luminance depth (0%). Best left near 100%; lower values add luminance as a direct depth signal, which can mislead on dark foreground / bright background scenes |
| ⇅ Invert | Flips the depth map — enabled by default to match Depth Anything V2's output convention |

---

## Tips

- **Best colour reproduction** is often achieved with **HALF-COL + Wimmer ticked + Red balance fully up**. Wimmer's left-eye weighting preserves reds that standard luma crushes, and boosting red balance compensates for the right eye's red channel being attenuated through cyan glasses. Start here if RC DUBOIS looks washed out
- **Warm or red-heavy photos** (sunsets, portraits with warm skin tones) work better with the Red balance slider pulled down a little in RC DUBOIS mode, or switch to HALF-COL with Wimmer and red balance up
- **Busy scenes** with many overlapping depth layers benefit from a small amount of depth smoothing (2–4)
- **Convergence plane** around 0.4–0.6 keeps most subjects comfortable. Push it lower to make foreground objects pop more aggressively
- **Max disparity** around 30–60px is comfortable for most images. Very large values may cause eye strain on complex scenes
- The depth proxy thumbnail (bottom right) shows the active depth map — white = nearest, black = farthest

---

## Requirements

- Modern browser with WebGL support (Chrome 90+, Edge 90+, Firefox 89+)
- Internet connection on first use only (model download)
- Red-cyan anaglyph glasses
