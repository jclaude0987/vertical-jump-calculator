# Jump Height Calculator

A browser-based tool for calculating vertical jump height from video footage using flight time analysis. No installation, no server, no data uploaded anywhere — everything runs locally in your browser.

## How it works

The app uses the physics of free-fall to calculate jump height from the time a jumper spends in the air:

1. You upload a video of a jump
2. The app works out the frame rate — it reads it straight out of the video file, falls back to measuring playback, and asks you to set it if neither works
3. It finds the takeoff and landing frames by watching the athlete's feet leave the ground and touch down again
4. You confirm those two frames, nudging them a frame either way if needed
5. Height is derived using: `d = ½ × g × (t/2)²` where `g = 9.81 m/s²` and `t/2` is the time to peak

## Features

- **Automatic frame-rate detection** — read from the MP4/MOV/WebM metadata, or measured by playing the video
- **Automatic takeoff and landing detection**, with a foot-contact chart so you can see why it chose those frames
- Frame-by-frame stepping, scrubbing, and manual marking as a fallback
- Any frame rate, including off-list rates such as 25, 50, 100 or 200 fps
- Results in centimeters and inches
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

- **Trim the clip to the jump first.** Detection works by seeking through the video frame by frame, so a 3-second clip is analysed in a couple of seconds while a 30-second clip can take 15 seconds or more.
- **Keep the camera still.** A tripod or a propped-up phone is ideal. Handheld footage usually defeats the detection, and the app will tell you so rather than guess.
- **Keep one athlete in frame**, with their feet visible and the ground in shot.
- **Verify the detected frames.** Automatic detection is usually within a frame or two, and one frame is a real amount of height at low frame rates (see below). Use the ← → buttons to nudge, then Calculate.
- **Match your frame rate** — now automatic, but worth checking. A mismatch causes large errors: selecting 2× the real fps underestimates height by 75%.
- **Use slow-motion footage** — higher frame rates give more precise flight time measurements.
- **Mark carefully if doing it by hand** — takeoff is the last frame both feet are on the ground; landing is the first frame a foot contacts the ground again.

## Accuracy notes

One frame of error matters more than it sounds, because height depends on flight time squared:

| fps | 1-frame error on a 0.5 s flight | Resulting height error |
|---|---|---|
| 240 | 0.4% | ~0.8% |
| 120 | 1.7% | ~3.3% |
| 60 | 3.3% | ~6.7% |
| 30 | 6.7% | ~13% |

Automatic detection was measured against synthetic clips with a known jump and landed within one or two frames, so at 30 fps budget roughly 10% uncertainty in the height unless you check and correct the marked frames. Verifying the two frames is always worth the few seconds.

## Limitations of automatic detection

The detection is a pixel-based heuristic, not machine learning, and it will refuse to act rather than guess. It reports that it can't find the jump when:

- the camera moves, pans, or zooms during the jump
- more than one person is moving in frame
- the background is unstable (shutters, changing light, passing traffic)
- the athlete's feet are out of frame, or their clothing is the same tone as the floor
- a deep counter-movement is followed by a shallow jump, which pixels alone can't reliably tell apart

When that happens, mark takeoff and landing by hand — the app never quietly applies markers it isn't confident about.

## Physics reference

| Variable | Value |
|---|---|
| Gravity (g) | 9.81 m/s² |
| Flight time (t) | frames in air ÷ frame rate |
| Time to peak | t ÷ 2 |
| Jump height (d) | ½ × g × (t/2)² |

## License

MIT — see [LICENSE](LICENSE)
