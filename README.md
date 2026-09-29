# VirtualTachometer

[![Version](https://img.shields.io/badge/version-3.9-blue.svg)](https://github.com/FireRayo/VirtualTachometer)
[![HTML5](https://img.shields.io/badge/HTML5-single--file-orange.svg)](https://github.com/FireRayo/VirtualTachometer)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)

**VirtualTachometer** is a browser-based industrial tachometer for measuring **linear speed** and **rotational speed (RPM)** from recorded video.

Designed for field work, commissioning, troubleshooting, maintenance, and machine validation when a physical tachometer is not available but a video can be recorded with a phone, camera, or other device.

**Live application:**  
https://firerayo.github.io/VirtualTachometer/

**Author:** Raymundo Ortiz  
**Current version:** V3.9

---

## Overview

VirtualTachometer calculates speed from the elapsed time between two selected video frames.

Typical applications include:

- Conveyor speed measurement
- Package or bottle transport verification
- Roller peripheral speed measurement
- Shaft, pulley, wheel, or roller RPM measurement
- Industrial commissioning and troubleshooting
- Comparing commanded speed with observed mechanical speed

The application runs directly in a modern browser using HTML5, CSS, and JavaScript.

---

## What's new in V3.9

### Automatic unit conversion

Changing the Distance or Diameter unit now converts the numeric value automatically while preserving the same physical length.

Supported units:

```text
mm / cm / m / km / in / ft / yd / mi
```

Examples:

```text
1000 mm  →  1 m
1 m      →  100 cm
1 m      →  1000 mm
1 m      →  39.3700787402 in
12 in    →  1 ft
1 mi     →  1.609344 km
```

This behavior is available in both:

- **Distance**
- **Diameter**

The conversion preserves the physical measurement instead of changing only the unit label.

### Recalculation workflow

When any measurement variable changes, the previous result is invalidated.

This includes:

- Distance
- Diameter
- Observed revolutions
- Distance unit
- Diameter unit
- Time unit
- Measurement mode
- Start frame
- End frame

Pressing **Compute** always recalculates the result using the current values.

This prevents an old result from remaining visible after measurement parameters are changed.

---

## Main features

- Distance, Diameter, and Revolutions modes
- Linear speed calculation
- RPM calculation
- Multiple observed revolutions
- Automatic metric/imperial unit conversion
- Explicit recalculation with **Compute**
- Frame-by-frame BWD/FWD navigation
- Keyboard `←` / `→` frame stepping
- CFR and VFR timing support
- MP4/MOV timing analysis
- Real sample timestamps when available
- `stts`, `ctts`, and common `elst` handling
- `requestVideoFrameCallback()` frame confirmation
- Seek locking and timeout recovery
- Approximate timestamp fallback when required
- Manual FPS input
- FPS **Autotune**
- Timing-resolution estimate
- English, Spanish, and Italian interface
- Google Material Symbols playback controls
- Persistent preferences with `localStorage`
- Single-file HTML5 application

---

# Measurement modes

## Distance

Use this mode when an object travels a known linear distance.

### Procedure

1. Open the video.
2. Select **Distance**.
3. Enter the known distance.
4. Select the desired unit.
5. Navigate to the first reference frame and press **Start**.
6. Navigate to the final reference frame and press **End**.
7. Press **Compute**.

Calculation:

```text
Elapsed Time = End Time - Start Time

Linear Speed = Distance / Elapsed Time
```

Example:

```text
Distance = 1000 mm
Elapsed Time = 2.000 s

Speed = 500 mm/s
```

---

## Diameter

Use this mode to calculate the peripheral speed of a roller, wheel, pulley, or similar rotating component.

Enter:

- Diameter
- Diameter unit
- Observed revolutions

Calculation:

```text
Linear Speed =
π × Diameter × Observed Revolutions
-----------------------------------
            Elapsed Time
```

Example:

```text
Diameter = 100 mm
Observed revolutions = 5
Elapsed Time = 2.500 s

Speed ≈ 628.319 mm/s
```

Using several complete revolutions generally improves measurement resolution.

---

## Revolutions

Use this mode to calculate rotational speed directly in RPM.

```text
RPM = 60 × Observed Revolutions / Elapsed Time
```

Example:

```text
Observed revolutions = 10
Elapsed Time = 4.000 s

RPM = 150 RPM
```

---

# Unit conversion

Distance and Diameter support:

| Metric | Imperial / US customary |
|---|---|
| mm | in |
| cm | ft |
| m | yd |
| km | mi |

When the selected unit changes, VirtualTachometer automatically converts the current numeric value.

Example:

```text
1000 mm
```

changed to meters becomes:

```text
1 m
```

The physical distance remains the same.

---

# Time units

Speed results can be displayed using:

- `/s`
- `/min`
- `/h`

For example:

```text
mm/s
m/min
ft/min
km/h
```

Changing the time unit invalidates the previous result. Press **Compute** to calculate again using the selected time basis.

---

# FPS and frame navigation

VirtualTachometer does not silently assume 30 FPS.

FPS can come from:

- MP4/MOV container timing
- Manual entry
- **Autotune**

For CFR video:

```text
Frame duration ≈ 1 / FPS
```

| FPS | Approx. frame duration |
|---:|---:|
| 24 | 41.67 ms |
| 25 | 40.00 ms |
| 30 | 33.33 ms |
| 50 | 20.00 ms |
| 60 | 16.67 ms |
| 120 | 8.33 ms |
| 240 | 4.17 ms |

---

## MP4/MOV timing analysis

For compatible ISO Base Media / QuickTime-style files, VirtualTachometer reads timing information from the video track.

Supported timing structures include:

- `mdhd` — media timescale
- `stts` — sample timing
- `ctts` — composition offsets
- common `elst` edit lists

When a usable timing table can be constructed, frame stepping uses real sample timestamps instead of relying only on:

```text
1 / FPS
```

This is especially important for VFR video.

---

## CFR and VFR

### CFR

Constant Frame Rate video is handled normally.

### VFR

Variable Frame Rate is detected from the timing table.

When exact sample timestamps are available:

- BWD/FWD navigates using actual sample times.
- Current frame estimation uses the sample-time table.

When exact VFR timing is unavailable, VirtualTachometer avoids presenting average-FPS stepping as exact.

---

# Presented-frame confirmation

When supported, the application uses:

```javascript
HTMLVideoElement.requestVideoFrameCallback()
```

and its:

```text
mediaTime
```

to identify the frame actually presented by the browser.

After seeking, the application waits for frame confirmation whenever possible.

If the browser does not confirm the frame in time, VirtualTachometer falls back to:

```text
video.currentTime
```

as an approximate timestamp so **Start** and **End** remain usable.

---

# Accuracy and timing resolution

Video measurement is limited by the temporal resolution of the recording.

Higher FPS generally improves temporal resolution.

V3.9 estimates timing resolution using local frame timing when available.

The displayed estimate does not include every physical source of uncertainty, including:

- Browser seek behavior
- Motion blur
- Rolling shutter
- Camera exposure
- Reference-mark selection error
- Incorrect physical distance or diameter
- Video compression artifacts

For better measurements:

1. Prefer high-frame-rate video.
2. Prefer CFR when possible.
3. Measure over longer intervals.
4. Use several complete revolutions.
5. Keep the camera stable.
6. Use high-contrast reference marks.
7. Avoid excessive motion blur.

---

# Controls

## Video

The playback controls use **Google Material Symbols Rounded**:

- `play_circle`
- `pause_circle`
- `fast_rewind`
- `fast_forward`

Functions:

- **Open Video** — select a local video
- Play icon — start playback
- Pause icon — pause playback
- Fast Rewind icon — one frame backward
- Fast Forward icon — one frame forward
- `← / →` — keyboard frame navigation
- Progress bar — seek through the video

## Measurement

- **Start** — store the starting timestamp
- **End** — store the ending timestamp
- **Compute** — calculate using all current measurement values

## FPS

- **Frame rate (FPS)** — detected or manually entered frame rate
- **Autotune** — estimate FPS from displayed-frame timing when supported

---

# Video compatibility

Automatic container timing analysis is implemented for compatible:

```text
.mp4
.m4v
.mov
.qt
```

Other formats may still play if supported by the browser, but FPS or timing information may need to be entered manually or estimated with **Autotune**.

Common MP4/H.264 video generally provides the widest browser compatibility.

---

# Privacy

Selected videos are processed locally in the browser.

VirtualTachometer does not require a backend or account and does not intentionally upload the selected video to a server.

---

# Use

## Online

https://firerayo.github.io/VirtualTachometer/

## Local

Download:

```text
index.html
```

and open it in a modern browser.

The application itself is a single HTML file.

Google Material Symbols are loaded as a web font, so the playback/navigation icons require internet access unless the font is already cached by the browser.

## Clone

```bash
git clone https://github.com/FireRayo/VirtualTachometer.git
cd VirtualTachometer
```

Then open:

```text
index.html
```

---

# Repository

https://github.com/FireRayo/VirtualTachometer

Typical structure:

```text
VirtualTachometer/
├── index.html
├── README.md
├── LICENSE
└── assets/
    └── IMG01.JPG
```

---

# Version history

## V3.9

- Added automatic numeric conversion when changing Distance or Diameter units.
- Supports conversion between `mm`, `cm`, `m`, `km`, `in`, `ft`, `yd`, and `mi`.
- Preserves the same physical measurement when changing units.
- Keeps the explicit Compute workflow introduced in V3.8.

## V3.8

- Any measurement-variable change invalidates the previous result.
- **Compute** always re-reads the current values and recalculates from scratch.
- Updated handling for Distance, Diameter, revolutions, units, time basis, Start, and End changes.

## V3.7

- Added Google Material Symbols Rounded playback/navigation controls.
- Play/Pause dynamically switches between `play_circle` and `pause_circle`.
- BWD/FWD use `fast_rewind` and `fast_forward`.

## V3.6

- Fixed Start/End becoming unavailable after some FWD/BWD operations.
- Added approximate `currentTime` fallback when displayed-frame confirmation times out.

## V3.1–V3.5

- Added MP4/MOV sample timestamp tables.
- Added `ctts` composition offsets.
- Added common `elst` edit-list handling.
- Improved VFR navigation.
- Added post-seek frame confirmation.
- Added seek timeout recovery.
- Added multiple revolutions to Diameter mode.
- Improved timing-resolution reporting.
- Removed silent nominal FPS snapping from calibrated FPS.
- Standardized main control labels.

## V3.0

- Redesigned FPS handling.
- Added MP4/MOV timing detection.
- Added VFR detection.
- Removed hidden 30 FPS fallback.
- Added manual FPS entry and recalibration.
- Added seek locking.
- Added `requestVideoFrameCallback()` timestamp tracking.
- Added multiple observed revolutions for RPM measurement.
- Added persistent preferences.
- Added English, Spanish, and Italian interfaces.

---

# License

Licensed under the **GNU General Public License v3.0 (GPL-3.0)**.

See [LICENSE](LICENSE).

---

# Author

**Raymundo Ortiz**

GitHub:  
https://github.com/FireRayo

Project repository:  
https://github.com/FireRayo/VirtualTachometer

Live application:  
https://firerayo.github.io/VirtualTachometer/

---

## Disclaimer

VirtualTachometer is a measurement aid based on recorded video timing.

Results may be affected by frame rate, variable frame timing, compression, dropped frames, motion blur, camera exposure, browser decoding behavior, seek accuracy, and Start/End frame selection.

For safety-critical calibration, certification, regulatory verification, or metrology applications, use properly calibrated measurement equipment and a validated measurement procedure.
