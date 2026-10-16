# AGENTS.md

## What this is

A single-file, zero-dependency browser app. All HTML, CSS, and JS are inline in `jump_height_calculator.html` (~1445 lines). There is no `package.json`, build step, bundler, test runner, linter, or CI config. Keep it that way: no external JS libraries, no npm. Splitting into ES modules would break README Option A, because a `file://` page cannot load modules.

## Running and verifying

- Open directly: `open jump_height_calculator.html` (works over `file://`).
- Or serve the folder: `python3 -m http.server` from the repo root.
- No automated test suite is checked in. `window.__appReady` is set at the end of the script purely so a harness can tell that the app booted; leave it in place.
- Load a video by dispatching `change` on `#fileInput` after assigning `input.files` from a `DataTransfer` — that is how browser-driven tests exercise `loadVideo`.

### Grading the detection

The auto-detection can be graded against synthetic clips with known ground truth. The harness used to build it lives outside the repo in a temp directory and was not committed. It consisted of:

1. a Swift `AVAssetWriter` script rendering a synthetic jump (a dark figure on a static background, an optional soft ground shadow) at a chosen fps, with a parabolic flight arc;
2. a frame extractor using `AVAssetImageGenerator` when you need to *look* at a clip rather than infer from it;
3. a static Python server that serves a directory and writes what the page `fetch`es to `/report` into a file, so results survive headless;
4. a page that loads the app in an iframe, pushes a synthetic clip through `#fileInput`, waits for `#autoStatus` to settle, and reports the marked frames.

Run headless with:
`"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --no-first-run --user-data-dir=$(mktemp -d) --autoplay-policy=no-user-gesture-required http://127.0.0.1:PORT/test.html`

Two things that will waste your time if you don't know them:

- **Give the iframe a real size** (e.g. 420x340). In a 2x2px iframe Chrome never decodes a frame, so `videoWidth` stays 0 and `analyzeMotion` bails out at its first guard.
- **Serve over HTTP, not `file://`.** Chrome blocks `file://` fetch from a `file://` page.

Ground truth measured on 2025-era iPhone clips in `~/Desktop/hspu/day2 ` (note the trailing space in the folder name): all are exactly **30 fps**, `mdhd` timescale 600 with a dominant `stts` delta of 20, matched against AVFoundation's `minFrameDuration`. `mdls`/`ffprobe` are not available on this machine, but `swiftc` and Chrome are.

Synthetic results at the last run (expect takeoff/landing as listed): clean arc 44/75 landed 45/75; same clip with a shadow 44/75 exactly; 60 fps variant 90/121 landed 92/120; a deep-countermovement shallow-jump clip **refused**; a no-jump clip **refused**; both real iPhone clips **refused** (correctly — they contain no jump).

## How it actually works

### Frame rate

`fps` is **detected, not declared**. It starts as `null`. `resolveFrameRate` tries, in order:

1. `probeContainer` — parses the file, no playback. MP4/MOV via `moov` box walking; WebM via the EBML `DefaultDuration` element.
2. `measureFps` — plays the clip muted and times `requestVideoFrameCallback` metadata.
3. Nothing — `detectionDone` is set, the note asks the user to choose, and every frame-indexed control stays disabled. There is no silent 120 default.

Details that cost real debugging time:

- **`moov` is usually at the end of the file** for these recordings. `findTopLevelBox` walks top-level boxes by header only and skips `mdat` by offset, so it costs a handful of small reads rather than a full scan.
- **Select the video track via `hdlr` == `vide`**, not the first `trak`; these files carry an audio track first.
- **Aggregate `stts` counts by delta value, not per entry.** These files split a uniform 30 fps stream across ~25 entries (delta histogram `{20: 413, 21: 7, 60: 4, 81: 1}`). Summing the whole `stts` folds in those gaps and yields 29.22 fps instead of 30. Use the dominant delta when it covers ≥90% of samples; otherwise fall back to a duration-weighted average and set `vfr`.
- **`totalFrames` is `floor(duration * fps)`, deliberately not the container sample count.** These files carry PTS offsets and discontinuities, so the sample count does not map onto the `0..duration` timeline the slider indexes. `parseMp4` still returns `frameCount`, but don't wire it into `totalFrames`.
- Detection over HTTP fails sometimes; on `file://` in Chrome, Firefox, and Safari it was not verified. If reported frame rates ever look wrong, check that first.

### Markers

Markers are stored as **media time** (`takeoffTime`, `landingTime`), and `renderMarkers` derives the frame indices from the current `fps`. This is why changing the frame rate re-quantises the markers instead of destroying them — the old behaviour wiped them via `resetMarkers`, which is now only used when loading a new video. Analysis results are also timed, so an fps change does not invalidate them.

### Auto-detection of takeoff and landing

The measured quantity is **ground contact**, not height. For each sampled frame the code measures how much of a band just above the foot level differs from a reference plate: standing, the band matches and reads ~0; once the feet leave, it shows floor where the reference had legs and reads high.

**Do not "simplify" this to the lowest row that differs from the plate.** That was the first implementation and it is wrong. The reference is a per-pixel median over the clip, so it normally *contains the standing athlete* — they stand for most of the clip. During flight the vacated silhouette below the airborne feet also differs from that reference, so the lowest differing row stays pinned to the ground for the whole flight and the signal never rises. The earlier version only appeared to work because a "row needs 10% of the width" filter happened to exclude the narrow legs, which made it track the hips by accident.

Other decisions that are load-bearing:

- **The foot level is the lowest row the scene ever differs at**, because the vacated silhouette reaches down to it. That anchors the measuring band without needing to know the floor line.
- **Thresholds are anchored to the measured noise floor, not to a share of the peak.** Using a proportion of the excursion reported takeoff several frames late and landing several frames early, because contact only registers once the feet have already travelled a noticeable distance. Noise is estimated from the grounded half of the samples only.
- **`refineContact` steps on the frame grid** (`f / fps`). Sampling at off-grid times lets the browser round to whichever frame it likes, which showed up as a systematic one-frame error.
- **Each sample costs a seek** (~20ms Chrome, ~75ms Firefox), so cost is linear in samples and `COARSE_MAX` is the knob. When the iframe/decode notes above are respected, a 5s clip analyses in ~5-6s and a 14s clip in ~16s. Lowering `COARSE_MAX` trades long-clip coarse density for time; the refine window scales with the coarse interval automatically.
- The confidence gate must stay conservative. Refusing to mark is the designed failure mode: the app never applies markers it isn't sure about, and a wrong guess is worse than asking. A deep counter-movement followed by a shallow jump is genuinely ambiguous from pixels alone and is refused on purpose.

## Values duplicated across files

These drift easily because they exist in more than one place:

- **Frame-rate presets** — the `.fps-row` buttons in the HTML, the `PRESETS` array used for snapping (which also carries off-list rates such as 23.976, 25, 59.94), and `README.md`.
- **Gravity (9.81 m/s²)** — the `calcBtn` handler, the README "Physics reference" table, and the README "How it works" formula.
- **The accuracy table in `README.md`** is derived from the sensitivity of `d = ½·g·(t/2)²` to one frame of error; if the flight-time formula ever changes, recompute it.

## Deployment

- GitHub Pages is live (`has_pages: true`) and republishes on every push to `main`. Only `main` exists — the history is direct-to-main web-UI edits, no PR or release flow to follow.
- Pages serves the repo root, so the site root renders `README.md` through Jekyll, while the app itself must be linked by filename: `https://jclaude0987.github.io/vertical-jump-calculator/jump_height_calculator.html`.
- There is no `index.html`. Renaming or moving the HTML file breaks the documented live URL — update `README.md` in the same change.
- Everything on `main` is public via both the repo and that Pages site, including this file. Nothing secret belongs here.

## UI constraints

- Barlow / Barlow Condensed fonts and the Tabler icon webfont load from CDN, so the README's "works entirely offline" only holds once those assets are cached. New icons should use Tabler `ti ti-*` classes.
- Colors belong in the `:root` custom properties (with the `prefers-color-scheme: dark` override), not hardcoded — that is the only thing keeping light and dark mode in sync. `drawChart` reads those variables via `getComputedStyle`, so use them there too.
- The `.fps-row` is `flex-wrap: wrap` because a detected off-list rate appends one extra button (e.g. `25 fps`). Keep the `min-width` so six buttons still wrap cleanly.
