# Portfolio — Pixelation Effect Site

## Overview
Personal portfolio website for mityasudakov.com featuring a full-screen animated pixelation backdrop effect. The effect is adapted from Cargo CMS's pixelation backdrop module, running on PIXI.js v4.8.9 (WebGL).

**Live site:** https://mityasudakov.com
**Repository:** https://github.com/sailorstale/portfolio
**Hosting:** GitHub Pages (deploy from `main` branch, auto-deploys on push)
**Domain:** mityasudakov.com (DNS on GoDaddy → GitHub Pages IPs)

## Architecture

### Files

| File | Purpose |
|------|---------|
| `index.html` | Main site. Loads PIXI.js, photo, settings.json, runs pixelation effect |
| `settings.json` | Desktop & mobile settings (grid size, turbulence, mouse params, etc.) |
| `admin.html` | Admin panel to edit settings per device, saves to GitHub via API |
| `pixi.min.js` | PIXI.js v4.8.9 (local copy, NOT a CDN) |
| `photo1.png` | Hero image (2800×4200px, ~4.9MB) |
| `photo1_data.js` | Base64-encoded photo for file:// development (6.3MB, NOT deployed to GitHub) |

### How the effect works

The pixelation effect divides a single image into an NxN grid of PIXI.Sprite objects. Each sprite shows a cropped portion of the source texture. The effect animates by:

1. **Stir turbulence** (`stirPixels`): Sine/cosine waves inject small speed impulses into each sprite based on its grid position and a global counter. Controlled by `stir_freq`, `stir_strength`, `stir_disorder`.

2. **Neighbor averaging** (`animateFrame`): Each sprite's movement is a weighted average of its neighbors' movements + its own speed. The `elasticity` parameter controls damping and neighbor influence. This creates fluid-like wave propagation.

3. **Texture frame shifting**: As sprites move, their texture crop region shifts — moving right reveals pixels to the left, creating a parallax/displacement effect. Frame boundaries are defined by `min_frame`, `max_frame`, `base_frame` per sprite.

4. **Mouse interaction** (`spriteMouseMove`): Mouse position is mapped to grid coordinates. Nearby sprites receive speed impulses proportional to mouse velocity. `tolerance` = radius, `mouse_sensitivity` = force, `mouse_zoom` = zoom on hover.

5. **Flip alternate** (`flip_it`): When enabled, odd-indexed sprites are horizontally/vertically flipped via negative scale, creating a kaleidoscope look.

6. **Cover-fit** (`resizeImage`): The sprite container is scaled to cover the viewport while maintaining image aspect ratio.

### Key functions (all in index.html)

- `Init()` — Creates PIXI renderer, stage, container. Sets up event listeners. Calls `loadImage()`.
- `loadImage()` — Loads photo via `<img>`, creates PIXI.BaseTexture. On file:// uses base64 from `photo1_data.js`.
- `loadSettingsJSON(callback)` — Fetches `settings.json`, detects mobile/desktop, merges into `settings` object, then calls callback.
- `makeSprites()` — Creates/adjusts NxN PIXI.Sprite grid based on `grid_size`.
- `setSpritePosition()` — Calculates texture frame regions for each sprite based on `zoom`, `flip_it`, grid position.
- `resizeImage()` — Scales the composition container to cover-fit the viewport.
- `stirPixels()` — Applies turbulence forces each frame.
- `animateFrame()` — Updates sprite movements via neighbor averaging, updates texture frames.
- `spriteMouseMove()` — Applies mouse-based forces to nearby sprites.
- `draw()` — Main animation loop via `requestAnimationFrame`.
- `initGyroscope()` / `startGyroListener()` — On mobile, converts device tilt to virtual cursor position.

### Settings (in settings.json)

```json
{
  "desktop": { ... },
  "mobile": { ... }
}
```

Each profile contains:

| Setting | Type | Range | Description |
|---------|------|-------|-------------|
| `grid_size` | int | 3–32 | Grid subdivisions (NxN) |
| `zoom` | int | -50–100 | Cell image zoom (negative = slight zoom out) |
| `flip_it` | bool | | Mirror alternate cells |
| `stir_grid` | bool | | Enable turbulence |
| `stir_freq` | int | 0–100 | Turbulence wave frequency |
| `stir_strength` | int | 0–100 | Turbulence force magnitude |
| `stir_disorder` | int | 0–100 | Per-cell turbulence variation |
| `elasticity` | int | 0–100 | Motion damping / neighbor influence |
| `mouse_interaction` | bool | | Enable mouse/touch/gyro response |
| `mouse_sensitivity` | int | 0–100 | Mouse movement force |
| `tolerance` | int | 0–100 | Mouse effect radius |
| `mouse_zoom` | int | -100–100 | Zoom on mouse hover |

### Admin panel (admin.html)

Located at mityasudakov.com/admin.html. Features:
- Two tabs: Desktop / Mobile — separate settings for each device
- Sliders and checkboxes for all settings
- "Save & Deploy" button commits `settings.json` to GitHub via REST API
- Requires GitHub Personal Access Token (stored in localStorage, never sent elsewhere)
- After save, GitHub Pages auto-deploys in ~1 minute

### Development notes

- **file:// protocol**: WebGL cannot use local images (tainted canvas error). The workaround is `photo1_data.js` which embeds the photo as a base64 data URL. When developing locally, include this script. It's NOT needed on the live server (https).
- **PIXI version**: Must use v4.8.9. The rendering logic uses v4-specific APIs (`PIXI.autoDetectRenderer(w, h, options)`, `new PIXI.Texture(baseTexture, rectangle)`, etc.). Do NOT upgrade to v5+.
- **All rendering functions** are adapted from Cargo's original pixelation.js. The formulas for stir, neighbor averaging, frame calculation, and mouse interaction are taken directly from the decompiled source.

### Deployment

Push to `main` branch on GitHub → GitHub Pages auto-deploys → live at mityasudakov.com in ~1 minute.

Files to deploy: `index.html`, `settings.json`, `admin.html`, `pixi.min.js`, `photo1.png`
Do NOT deploy: `photo1_data.js`, `pixelation_source.txt`, `cargo_pixelation_source.js`
