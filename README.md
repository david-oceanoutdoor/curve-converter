# The Curve Converter

Single-file, client-side tool (`TheCurveConverter.html`) for a 12×3 tile grid of 324×720 tiles
(LED/DOOH panel layout). Everything runs in the browser; no uploads, no backend, no network
access needed (the MP4 muxer is bundled into the page).

## Modes (Direction dropdown)
- **Squeeze:** 3888×2160 → squeezed to 3840 wide + 48px black pad on the right, output 3888×2160. Selected by default.
- **Remove gaps:** gapped → clean (4064×2192 → 3888×2160 with the default 16px gaps)
- **Add gaps:** clean → gapped (reverse). For stills the gaps are transparent in the PNG.
- **Compress only:** video re-encode at the chosen bitrate, no layout change (video only)

## Content (dropdown under Direction)
Sets the video bitrate: **Full-motion video** → 15 Mbps (default), **Mostly stills** → 5 Mbps.
The bitrate can still be fine-tuned by hand under Settings.

## Settings (collapsed under "Settings")
Columns, rows, tile size, horizontal gap, "last horizontal gap" (for the 4010px layout:
11px gaps with the final one 12px, vertical 16px), fps, bitrate (Mbps, default 15),
squeeze width.

## How it works
- **Stills:** canvas `drawImage` per tile (source rect → dest rect), PNG out.
- **Video:** seek per frame, draw to canvas, WebCodecs `VideoEncoder`
  (H.264 High), [mp4-muxer](https://github.com/Vanilagy/mp4-muxer) 5.2.2 (MIT, inlined at the
  end of the HTML). The H.264 level is picked from output size × fps (5.1 for 4064×2192 at 25 fps,
  5.2 at 30 fps, up to 6.2), and support is checked before encoding starts. The fps setting must
  match the source.
- **Silent video only.** Sources are always silent, so there is no audio path; output MP4s have
  no audio track. (An earlier real-time `MediaRecorder` mode existed only to keep audio and was
  removed.)
- Output filenames append the mode: `-remove-gaps`, `-add-gaps`, `-squeezed`, `-compressed`.

## Test results (headless Chromium 141, Linux, no GPU)
| Path | Result |
| --- | --- |
| Stills: remove / add / squeeze | Pixel-exact against a synthetic gapped pattern (16px and 11/12px layouts); remove → add round trip is exact; squeeze pad is pure black |
| Frame-accurate seeking | Canvas content matched the source on all 50 frames of a test clip whose brightness encodes the frame number: no off-by-one |
| Frame-accurate frame count / duration | 50 frames, 2.000 s out for 50 frames, 2.000 s in (remove and compress) |
| Frame-accurate muxing | Valid MP4 from the bundled muxer |
| **H.264 encoding** | **Not tested**: open-source Chromium has no H.264 encoder. The pipeline was tested with VP9 swapped in; needs a real run in Chrome or Edge |

## Known risks / next steps
- Run a real H.264 export in Chrome/Edge (Windows and macOS) and check it plays on the target player.
- fps is a manual input and a mismatch silently retimes the video; consider reading it from the
  file (e.g. via mp4box.js, or `requestVideoFrameCallback` media times).
- Bitrate default (15 Mbps) is a guess; compare file size and quality at several values.

## Related (not in repo)
After Effects scripts (AddGridGaps / RemoveGridGaps / RecomposeGrid) and a Photoshop script
(SplitToCleanMaster) do the same tile maths.
