# pyledstrip-detector

Automatically detects the physical 2D position of each LED on an LED strip using a camera and Gray code structured light.

## How it works

1. **Detection** (`detector.py`): The LED strip is illuminated with Gray code patterns using [pyledstrip](https://github.com/ledstrip/pyledstrip). A camera captures each pattern, and per-pixel Gray code decoding maps every camera pixel to the LED index it sees. The median pixel position for each LED is exported as a JSON heightmap.

2. **Post-processing** (`process_led_map.py`): The raw heightmap is cleaned up — missing LEDs are filled in by linear interpolation, the path is smoothed with a Savitzky-Golay filter, and coordinates are optionally converted to metric units (metres, assuming 5 mm LED pitch by default).

## Setup

Install dependencies (OpenCV, NumPy, matplotlib, pyledstrip, pyperclip):

```bash
pip install opencv-python numpy matplotlib pyledstrip pyperclip
```

Connect your LED strip and point a camera at it.

## Usage

### Step 1 — Detect LED positions

```bash
python detector.py --host <pyledstrip-host> [pyledstrip options]
```

This flashes Gray code patterns on the strip, captures frames from the default camera (`/dev/video0`), and writes the raw LED map to `data/heightmap.<timestamp>.json`.

### Step 2 — Post-process the map

```bash
python process_led_map.py --infile data/heightmap.<timestamp>.json --outfile led_map.json
```

If `--infile` is omitted, the filename is read from the clipboard. If `--outfile` is omitted, it defaults to `<infile>.processed.json`.

## Output format

Both scripts produce a JSON array of LED positions:

```json
[
  { "id": 0, "x": 0.042, "y": 0.015 },
  { "id": 1, "x": 0.047, "y": 0.016 }
]
```

## Files

| File | Description |
|------|-------------|
| `detector.py` | Captures Gray code frames and decodes LED positions |
| `ledworld.py` | `LedWorld` / `Led` data model with interpolation, smoothing, and metric conversion |
| `process_led_map.py` | CLI wrapper for post-processing a raw heightmap |
| `test_ledworld.py` | Unit tests for `LedWorld` serialisation |
| `data/` | Output directory for generated heightmaps (JSON files excluded from git) |

## Running tests

```bash
python -m unittest test_ledworld.py
```
