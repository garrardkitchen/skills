---
name: GIF Creator
description: Convert video recordings (mp4, mov, avi, etc.) to optimized GIFs with automatic sensitive area blurring/masking
---

# GIF Creator Skill

When invoked, this skill converts a video recording into an optimized GIF and optionally blurs or masks sensitive regions (passwords, PII, API keys, faces, etc.) using ffmpeg and Python/Pillow.

## General list of text to blur out:

- http(s) URLs
- Email addresses
- File paths
- usernames
- Passwords (often in terminal output)
- API keys, tokens, credentials
- IDs
- Human names
- PII (Personally Identifiable Information)
- Telephone numbers
- Addresses
- Machine names (e.g. server hostnames)
- Company/organization names
- GUIDs and UUIDs
- Anything hypenated or snake_case that looks like a variable name

---

## Step 1: Gather User Input

Ask the user for the following (if not already provided):

1. **Input video path** — full or relative path to the source video (mp4, mov, avi, mkv, webm, etc.)
2. **Output GIF path** — where to save the resulting GIF (default: same directory as input, `.gif` extension)
3. **Quality preset** — one of:
   - `small` — 10 fps, 480px wide (fast, compact file)
   - `medium` *(default)* — 15 fps, 800px wide (balanced)
   - `high` — 24 fps, 1280px wide (crisp, larger file)
   - `custom` — user specifies fps and width manually
4. **Sensitive area masking** — yes or no; if yes, proceed to Step 4 before finalizing

---

## Step 2: Install Dependencies

Detect the OS and run the appropriate block. Inform the user what will be installed before running.

### macOS (Homebrew)
```bash
# Install ffmpeg and gifsicle
brew install ffmpeg gifsicle

# Install Python dependencies (uses system python3)
pip3 install --quiet Pillow
```

### Linux — Debian/Ubuntu
```bash
sudo apt-get update -qq
sudo apt-get install -y ffmpeg gifsicle python3 python3-pip
pip3 install --quiet Pillow
```

### Linux — Fedora/RHEL/CentOS
```bash
# Enable RPM Fusion for ffmpeg if not already enabled
sudo dnf install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm 2>/dev/null || true
sudo dnf install -y ffmpeg gifsicle python3 python3-pip
pip3 install --quiet Pillow
```

### Linux — Arch
```bash
sudo pacman -S --noconfirm ffmpeg gifsicle python python-pip
pip3 install --quiet Pillow
```

### Windows (winget — preferred)
```powershell
winget install --id Gyan.FFmpeg -e
winget install --id GIFSICLE.gifsicle -e
pip install Pillow
# Reload PATH so ffmpeg is available in this session
$env:PATH = [System.Environment]::GetEnvironmentVariable("PATH","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("PATH","User")
```

### Windows (Chocolatey — alternative)
```powershell
choco install ffmpeg gifsicle -y
pip install Pillow
```

**Verify installation:**
```bash
ffmpeg -version | head -1
gifsicle --version
python3 -c "import PIL; print('Pillow OK')"
```

---

## Step 3: Convert Video to GIF

Use ffmpeg with a two-pass palette approach for the best color quality.

### Quality Presets

| Preset | FPS | Width | Palette Colors |
|--------|-----|-------|----------------|
| small  | 10  | 480   | 128            |
| medium | 15  | 800   | 192            |
| high   | 24  | 1280  | 256            |

### Conversion Command

```bash
# Set variables (substitute actual values)
INPUT="path/to/recording.mp4"
OUTPUT="path/to/output.gif"
FPS=15
WIDTH=800
COLORS=192

# Two-pass conversion: generate optimal palette then render
ffmpeg -i "$INPUT" \
  -vf "fps=${FPS},scale=${WIDTH}:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=${COLORS}:stats_mode=diff[p];[s1][p]paletteuse=dither=bayer:bayer_scale=5" \
  -loop 0 \
  "$OUTPUT"
```

**Trimming a clip** (optional — ask user if they want only a portion):
```bash
# -ss = start time, -to = end time (HH:MM:SS or seconds)
ffmpeg -ss 00:00:05 -to 00:00:30 -i "$INPUT" \
  -vf "fps=${FPS},scale=${WIDTH}:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=${COLORS}:stats_mode=diff[p];[s1][p]paletteuse=dither=bayer:bayer_scale=5" \
  -loop 0 \
  "$OUTPUT"
```

---

## Step 4: Mask Sensitive Areas (Optional)

If the user wants to blur or mask regions containing sensitive content (passwords, credentials, personal info, faces, etc.), follow this process.

### 4a: Extract Reference Frame

Extract the first frame so the user can identify coordinates:

```bash
ffmpeg -i "$INPUT" -vframes 1 -q:v 2 reference_frame.png
```

Tell the user: *"Reference frame saved as `reference_frame.png`. Open it and note the pixel coordinates (x, y, width, height) of each region to mask."*

### 4b: Gather Region Coordinates

Ask the user for each region in format: `x,y,width,height` (top-left origin).

Example regions:
- Password field at top-right: `900,45,320,35`
- Name in sidebar: `0,200,250,30`
- Terminal command output: `10,400,600,80`

Multiple regions can be specified as a comma-separated list of `x:y:w:h` groups.

### 4c: Option A — Blur During Conversion (ffmpeg only, fastest)

For each region `x,y,w,h`, build a filter chain. Example with two regions:

```bash
# Template — extend [fg2][fg3]... pattern for more regions
INPUT="path/to/recording.mp4"
OUTPUT="path/to/output.gif"
FPS=15
WIDTH=800

ffmpeg -i "$INPUT" -filter_complex \
  "[0:v]crop=320:35:900:45,boxblur=20:5[r1]; \
   [0:v]crop=250:30:0:200,boxblur=20:5[r2]; \
   [0:v][r1]overlay=900:45[tmp1]; \
   [tmp1][r2]overlay=0:200[blurred]; \
   [blurred]fps=${FPS},scale=${WIDTH}:-1:flags=lanczos,split[s0][s1]; \
   [s0]palettegen=max_colors=192:stats_mode=diff[p]; \
   [s1][p]paletteuse=dither=bayer:bayer_scale=5[out]" \
  -map "[out]" -loop 0 "$OUTPUT"
```

> **Note:** Adjust `crop=W:H:X:Y` and `overlay=X:Y` to match each region's coordinates.

### 4c: Option B — Blur GIF Frames with Python (more flexible, supports solid fill)

Use this when coordinates come from the *output GIF* (after potential scaling), or when you want a solid color block instead of blur.

**Generate and run this Python script:**

```python
#!/usr/bin/env python3
"""
blur_gif.py — Apply blur or solid mask to regions of an animated GIF.
Usage: python3 blur_gif.py input.gif output.gif x1,y1,x2,y2 [x1,y1,x2,y2 ...]
       Coordinates are x1,y1 (top-left) to x2,y2 (bottom-right) in output GIF pixels.
       To use solid fill instead of blur, set SOLID_FILL=True and FILL_COLOR below.
"""

import sys
from PIL import Image, ImageFilter, ImageDraw

# ── Configuration ──────────────────────────────────────────────────────────
BLUR_RADIUS = 6      # Gaussian blur radius (higher = more obscured) - was 18, changed to 4
SOLID_FILL  = False   # Set True to use an opaque color block instead of blur
FILL_COLOR  = (30, 30, 30)  # RGB color for solid fill (dark grey default)
# ───────────────────────────────────────────────────────────────────────────

def parse_regions(args):
    regions = []
    for arg in args:
        parts = list(map(int, arg.split(",")))
        assert len(parts) == 4, f"Expected x1,y1,x2,y2 but got: {arg}"
        regions.append(tuple(parts))
    return regions

def mask_gif(input_path, output_path, regions):
    src = Image.open(input_path)
    frames, durations = [], []

    try:
        while True:
            frame = src.copy().convert("RGBA")
            draw = ImageDraw.Draw(frame)

            for (x1, y1, x2, y2) in regions:
                # Clamp to image bounds
                x1 = max(0, x1); y1 = max(0, y1)
                x2 = min(frame.width, x2); y2 = min(frame.height, y2)
                if x2 <= x1 or y2 <= y1:
                    continue

                if SOLID_FILL:
                    draw.rectangle([x1, y1, x2, y2], fill=FILL_COLOR + (255,))
                else:
                    region = frame.crop((x1, y1, x2, y2))
                    blurred = region.filter(ImageFilter.GaussianBlur(radius=BLUR_RADIUS))
                    frame.paste(blurred, (x1, y1))

            frames.append(frame.convert("P", palette=Image.ADAPTIVE, colors=192))
            durations.append(src.info.get("duration", 80))
            src.seek(src.tell() + 1)
    except EOFError:
        pass

    if not frames:
        print("ERROR: No frames found in GIF.")
        sys.exit(1)

    frames[0].save(
        output_path,
        save_all=True,
        append_images=frames[1:],
        loop=0,
        duration=durations,
        optimize=False,
    )
    print(f"✅ Saved masked GIF → {output_path}  ({len(frames)} frames)")

if __name__ == "__main__":
    if len(sys.argv) < 4:
        print("Usage: python3 blur_gif.py input.gif output.gif x1,y1,x2,y2 [...]")
        sys.exit(1)

    input_path  = sys.argv[1]
    output_path = sys.argv[2]
    regions     = parse_regions(sys.argv[3:])

    print(f"Masking {len(regions)} region(s) across all frames...")
    mask_gif(input_path, output_path, regions)
```

**Run the script:**
```bash
# Save the script
cat > blur_gif.py << 'EOF'
# (paste the script above)
EOF

# Apply masking — add as many regions as needed
python3 blur_gif.py output.gif output_masked.gif 900,45,1220,80 0,200,250,230
```

---

## Step 5: Optimize GIF Size (Optional but Recommended)

Use `gifsicle` to reduce file size without visible quality loss:

```bash
# Lossy compression (recommended — often 40-60% smaller)
gifsicle --lossy=80 --optimize=3 --colors 256 -o output_final.gif output.gif

# Lossless optimization only
gifsicle --optimize=3 -o output_final.gif output.gif

# Check file sizes
ls -lh output*.gif
```

---

## Step 6: Report Results

After completion, report:
- ✅ Output file path and size
- 📐 Dimensions (width × height) and frame count
- ⏱️ Duration
- 🎭 Number of masked regions (if any)

```bash
# Get GIF metadata
python3 - <<'EOF'
from PIL import Image
img = Image.open("output_final.gif")
frames = 0
try:
    while True:
        frames += 1
        img.seek(img.tell() + 1)
except EOFError:
    pass
duration_ms = img.info.get("duration", 80) * frames
print(f"Size: {img.width}x{img.height}px")
print(f"Frames: {frames}")
print(f"Duration: {duration_ms/1000:.1f}s")
EOF

ls -lh output_final.gif
```

---

## Quick Reference — Common Scenarios

### Screen recording with terminal output to blur
```bash
# Blur a terminal panel region — adjust coordinates to match your recording
ffmpeg -i screen_recording.mp4 -filter_complex \
  "[0:v]crop=600:200:0:600,boxblur=25:5[r1];[0:v][r1]overlay=0:600[v]; \
   [v]fps=15,scale=800:-1:flags=lanczos,split[s0][s1]; \
   [s0]palettegen=max_colors=192[p];[s1][p]paletteuse=dither=bayer[out]" \
  -map "[out]" -loop 0 demo.gif
```

### Convert a specific screen region only (crop + GIF)
```bash
# Crop to region before converting — X:Y:W:H
ffmpeg -i input.mp4 \
  -vf "crop=1200:800:100:50,fps=15,scale=800:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" \
  -loop 0 cropped.gif
```

### Slow down / speed up GIF
```bash
# Slow to 50% speed (setpts=2.0), speed up to 200% (setpts=0.5)
ffmpeg -i input.mp4 \
  -vf "setpts=2.0*PTS,fps=15,scale=800:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" \
  -loop 0 slow.gif
```

---

## Troubleshooting

| Problem | Solution |
|--------|---------|
| `ffmpeg: command not found` | Re-run install step; on Windows restart terminal after install |
| GIF is too large | Lower FPS, reduce width, or use `gifsicle --lossy=80` |
| Colors look washed out | Add `palettegen=stats_mode=diff` and `paletteuse=dither=sierra2_4a` |
| Blur coordinates are off after scaling | Extract reference frame *from the GIF*, not the source video |
| Python script exits with `KeyError` | Some GIF encoders omit `duration` metadata — script defaults to 80ms/frame |
| `PIL` not found | Run `pip3 install Pillow` (capital P) |
| `gifsicle: not found` on Linux | Try `sudo apt install gifsicle` or `sudo dnf install gifsicle` |
