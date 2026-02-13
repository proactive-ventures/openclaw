---
name: video-frames
description: Extract frames or short clips from videos using ffmpeg.
homepage: https://ffmpeg.org
metadata:
  {
    "openclaw":
      {
        "emoji": "🎞️",
        "requires": { "bins": ["ffmpeg"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "ffmpeg",
              "bins": ["ffmpeg"],
              "label": "Install ffmpeg (brew)",
            },
            {
              "id": "apt",
              "kind": "apt",
              "package": "ffmpeg",
              "bins": ["ffmpeg"],
              "label": "Install ffmpeg (apt)",
            },
          ],
      },
  }
---

# Video Frames (ffmpeg)

Extract frames, thumbnails, or clips from video files.

## Quick Start

First frame:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
```

At a timestamp:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

## Direct ffmpeg Commands

Extract a single frame at 10 seconds:

```bash
ffmpeg -ss 00:00:10 -i video.mp4 -frames:v 1 -q:v 2 frame.jpg
```

Extract one frame every N seconds (thumbnail grid):

```bash
ffmpeg -i video.mp4 -vf "fps=1/5" thumb_%04d.jpg
```

Extract a 3-second clip starting at 30s:

```bash
ffmpeg -ss 00:00:30 -i video.mp4 -t 3 -c copy clip.mp4
```

Create a GIF from a clip:

```bash
ffmpeg -ss 00:00:05 -i video.mp4 -t 3 -vf "fps=10,scale=320:-1" output.gif
```

Get video info (duration, resolution, codec):

```bash
ffmpeg -i video.mp4 2>&1 | grep -E "Duration|Stream"
```

## Notes

- Prefer `--time` or `-ss` for "what is happening around here?".
- Use `.jpg` for quick share; use `.png` for crisp UI frames.
- `-q:v 2` sets JPEG quality (1=best, 31=worst; 2 is good).
- `-ss` before `-i` is faster (seeks without decoding).
