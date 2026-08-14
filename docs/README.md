Drop `demo.gif` (or `demo.mp4`, referenced separately) here.

Suggested capture, ~15 seconds, screen recorded inside Krita:
1. Open a `.kra` file, make a visible stroke, click **add checkpoint**.
2. Make another stroke, add a second checkpoint with a message this time.
3. Right-click a history row -> **Make Current** to restore it.
4. Drag the thumbnail-size slider a couple notches.

Convert to GIF with ffmpeg, e.g.:
`ffmpeg -i demo.mp4 -vf "fps=12,scale=720:-1:flags=lanczos" -loop 0 demo.gif`
