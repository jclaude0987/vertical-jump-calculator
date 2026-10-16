# Jump Height Calculator

A browser-based tool for calculating vertical jump height from video footage using flight time analysis, and tracking those jumps over time. No installation, no server, no data uploaded anywhere — everything runs locally in your browser.

Two dashboards: **Measure** turns a video into a height, and **Career** plots your history.

## How it works

The app uses the physics of free-fall to calculate jump height from the time a jumper spends in the air:

1. You upload a video of a jump
2. The app works out the frame rate — it reads it straight out of the video file, falls back to measuring playback, and asks you to set it if neither works
3. You step to the takeoff and landing frames and mark each one
4. Height is derived using: `d = ½ × g × (t/2)²` where `g = 9.81 m/s²` and `t/2` is the time to peak

## Features

- **Automatic frame-rate detection** — read from the MP4/MOV/WebM metadata, or measured by playing the video
- Frame-by-frame stepping and scrubbing to mark takeoff and landing precisely
- Any frame rate, including off-list rates such as 25, 50, 100 or 200 fps
- A flight-path diagram that scales with the measured height
- Results in centimeters and inches
- **Career dashboard** — every jump is saved to the day's session, and each day's mean is plotted on a line graph so you can see your height variation over weeks and months
- Sessions are labelled by date and can be renamed, expanded to see every jump, and exported or imported as JSON
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

- **Match your frame rate** — detected automatically, but worth checking. A mismatch causes large errors: selecting 2× the real fps underestimates height by 75%.
- **Use slow-motion footage** — higher frame rates give more precise flight time measurements.
- **Mark carefully.** Takeoff is the last frame both feet are on the ground; landing is the first frame a foot contacts the ground again. Use the ← → buttons to step one frame at a time rather than scrubbing.
- **Frame one whole jump**, with the feet visible and the ground in shot.
- **Save every jump, not just the best one.** The career graph plots the daily mean, so a session with five jumps is a better estimate of that day than a single good rep.

## Accuracy notes

One frame of error matters more than it sounds, because height depends on flight time squared:

| fps | 1-frame error on a 0.5 s flight | Resulting height error |
|---|---|---|
| 240 | 0.4% | ~0.8% |
| 120 | 1.7% | ~3.3% |
| 60 | 3.3% | ~6.7% |
| 30 | 6.7% | ~13% |

## Tracking over time

Jumps are saved into **sessions**. One session is one day, and it holds every jump you measured that day.

**The graph plots one point per session: the mean of that day's jumps.** Jump five times on a Tuesday and Tuesday is a single point, the average of all five. The Sessions list underneath shows every individual jump, so a good or bad rep is not hidden by the average.

Statistics shown above the graph:

| Stat | What it is |
|---|---|
| Personal best | Your highest single jump, with the date |
| Sessions | How many days you have recorded, and total jumps |
| Last 5 sessions | Mean of the last five daily means |
| Trend | Change from your first session in the range to your latest |

Use **All / 90d / 30d** to change the window.

### Where your data lives

In your browser's `localStorage`. Nothing is uploaded. Two consequences worth knowing:

- **It is per-browser and per-origin.** Your local copy and the published GitHub Pages copy keep *separate* histories, and clearing site data wipes it.
- **Use Export** to write a JSON file, and **Import** to load one back. Import merges by date rather than replacing, so it is safe to combine a local history with the published one.

### Session dates

A session is dated when you save the jump, not when the video was filmed. Video files carry unreliable timestamps for anything copied, synced, or re-encoded, so the app uses the day you did the analysis. Use **Change** next to the session name to rename a session or set a different date.

## Physics reference

| Variable | Value |
|---|---|
| Gravity (g) | 9.81 m/s² |
| Flight time (t) | frames in air ÷ frame rate |
| Time to peak | t ÷ 2 |
| Jump height (d) | ½ × g × (t/2)² |

## License

MIT — see [LICENSE](LICENSE)
