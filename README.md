# Jump Height Calculator

A browser-based tool for calculating vertical jump height from video footage using flight time analysis. No installation, no server, no data uploaded anywhere — everything runs locally in your browser.

## How it works

The app uses the physics of free-fall to calculate jump height from the time a jumper spends in the air:

1. You upload a slow-motion video of a jump
2. You mark the exact takeoff and landing frames
3. The app calculates flight time from the frame count and frame rate
4. Height is derived using: `d = ½ × g × (t/2)²` where `g = 9.81 m/s²` and `t/2` is the time to peak

## Features

- Video scrubbing with frame-by-frame step controls
- Mark takeoff and landing frames via slider or play-and-mark
- Supports 30, 60, 90, 120, and 240 fps footage
- Displays results in centimeters and inches
- Load multiple videos without refreshing
- Works entirely offline after first load
- Light and dark mode support

## Usage

### Option A — Open directly
Download `jump_height_calculator.html` and open it in any modern browser (Chrome, Firefox, Safari, Edge).

### Option B — GitHub Pages
Visit the live version at:
```
https://jclaude0987.github.io/vertical-jump-calculator/jump_height_calculator.html
```

## Tips for accurate results

- **Match your frame rate** — the single most important setting. If your video is 120 fps, select 120 fps. A mismatch causes large errors (selecting 2× the real fps underestimates height by 75%).
- **Use slow-motion footage** — higher frame rates give more precise flight time measurements.
- **Mark carefully** — takeoff is the last frame both feet are on the ground; landing is the first frame a foot contacts the ground again.
- **Step frame-by-frame** — use the ← → buttons to find the exact frame rather than scrubbing.

## Physics reference

| Variable | Value |
|---|---|
| Gravity (g) | 9.81 m/s² |
| Flight time (t) | frames in air ÷ frame rate |
| Time to peak | t ÷ 2 |
| Jump height (d) | ½ × g × (t/2)² |

## License

MIT — see [LICENSE](LICENSE)
