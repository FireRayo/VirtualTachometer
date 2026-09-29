# VirtualTachometer

[![Version](https://img.shields.io/badge/version-4.0-blue.svg)](https://github.com/FireRayo/VirtualTachometer)
[![HTML5](https://img.shields.io/badge/HTML5-single--file-orange.svg)](https://github.com/FireRayo/VirtualTachometer)
[![Offline](https://img.shields.io/badge/offline-ready-success.svg)](https://github.com/FireRayo/VirtualTachometer)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)

**VirtualTachometer** is a browser-based industrial tachometer for measuring **linear speed** and **rotational speed (RPM)** from recorded video.

It is designed for field service, commissioning, troubleshooting, maintenance, machine validation, conveyor checks, roller measurements, and other industrial applications where speed must be estimated from video.

**Live application:**  
https://firerayo.github.io/VirtualTachometer/

**Author:** Raymundo Ortiz  
**Current version:** V4.0

---

## Overview

VirtualTachometer measures the elapsed time between two selected video frames and calculates speed from a known distance, diameter, or number of revolutions.

Typical applications include:

- Conveyor speed measurement
- Package or bottle transport verification
- Roller peripheral speed measurement
- Shaft, pulley, wheel, or roller RPM measurement
- Machine commissioning and troubleshooting
- Comparing commanded and observed mechanical speed
- Quick field measurements when a physical tachometer is not available

The application runs directly in a modern browser using HTML5, CSS, and JavaScript.

---

# What's new in V4.0

V4.0 is a major architectural and interface revision focused on **offline operation, maintainability, visual consistency, and robustness**.

## Fully self-contained HTML5 application

V4.0 is distributed as a **single `.html` file** containing everything required by the application:

- HTML
- CSS
- JavaScript
- SVG icons
- UI graphics
- Light/Dark themes
- Measurement logic
- MP4/MOV timing parser

There are:

- No CDN dependencies
- No remote JavaScript libraries
- No external fonts
- No external icon libraries
- No online resources required for normal operation

The application can therefore be opened directly from the local filesystem and used without an Internet connection.

---

## New Neomorphism / Soft UI interface

The complete interface was redesigned using **Neomorphism / Soft UI**.

Two complete themes are included:

- **Light**
- **Dark**

The selected theme can be changed from the interface and is stored locally.

### Light theme

```text
Background:      #DFE4EA
Surface:         #E5E9EE
Shadow light:    #FFFFFF
Shadow dark:     #C2C6CA
Text:            #30363B
Muted text:      #687078
Accent light:    #6C9EFF
Accent:          #557BD9
Accent dark:     #526FC0
```

### Dark theme

```text
Background:      #20252B
Surface:         #272C33
Shadow light:    #323841
Shadow dark:     #181C21
Text:            #E6EAF0
Muted text:      #9CA6B2
Accent light:    #739DF2
Accent:          #5C7FD6
Accent dark:     #4868B8
```

Cards and buttons use raised dual-shadow surfaces, while inputs, selectors, sliders, and editable controls use inset Neomorphic surfaces.

---

## Offline SVG controls

The playback controls are now embedded directly as SVG graphics.

No icon font is required.

Included controls:

- Play
- Pause
- Fast Rewind
- Fast Forward

This replaces the external Material Symbols dependency used by V3.7–V3.9.

---

## Maintainable internal architecture

The V4.0 source code was reorganized to make diagnosis and future modifications easier.

The application is divided logically into responsibilities such as:

- Configuration
- Application state
- Preferences
- Internationalization
- Theme management
- Unit conversion
- Measurement calculations
- Video loading
- Frame tracking
- Seeking
- FPS handling
- MP4/MOV timing parsing
- UI rendering
- Error handling

Technical identifiers remain in English, while important internal documentation and comments are written in **Spanish** to explain:

- What each section does
- Why the implementation exists
- How the state flows through the application
- What should be modified when behavior needs to change

---

# Main features

- Single-file HTML5 application
- Completely self-contained
- Offline operation
- Light and Dark Neomorphism themes
- Responsive desktop/mobile layout
- Distance mode
- Diameter mode
- Revolutions mode
- Linear speed calculation
- RPM calculation
- Multiple observed revolutions
- Automatic metric/imperial unit conversion
- Explicit recalculation with **Compute**
- Frame-by-frame BWD/FWD navigation
- Keyboard `←` / `→` frame stepping
- Controlled asynchronous seeking
- MP4/MOV timing-table inspection
- CFR/VFR detection
- Real sample timestamps when available
- `stts`, `ctts`, and common `elst` handling
- `requestVideoFrameCallback()` support
- Post-seek frame confirmation
- Approximate timestamp fallback
- Manual FPS entry
- FPS **Autotune**
- Timing-resolution estimate
- Video decoding/error messages
- English, Spanish, and Italian UI
- Persistent preferences with `localStorage`

---

# Measurement modes

## 1. Distance

Use this mode when an object travels a known linear distance.

### Procedure

1. Open a video.
2. Select **Distance**.
3. Enter the known distance.
4. Select the desired unit.
5. Navigate to the first reference frame.
6. Press **Start**.
7. Navigate to the final reference frame.
8. Press **End**.
9. Press **Compute**.

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

## 2. Diameter

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

Using several complete revolutions usually improves the effective timing resolution.

---

## 3. Revolutions

Use this mode to calculate rotational speed directly in RPM.

Calculation:

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

# Automatic unit conversion

Distance and Diameter support:

```text
mm / cm / m / km / in / ft / yd / mi
```

Changing the selected unit automatically converts the numeric value while preserving the same physical length.

Examples:

```text
1000 mm  →  1 m
1 m      →  100 cm
1 m      →  1000 mm
1 m      →  39.3700787402 in
12 in    →  1 ft
1 mi     →  1.609344 km
```

This applies to both:

- Distance
- Diameter

An empty field remains empty when its unit is changed.

---

# Compute workflow

Changing any measurement variable invalidates the previous result.

This includes:

- Measurement mode
- Distance
- Diameter
- Observed revolutions
- Distance unit
- Diameter unit
- Time unit
- Start frame
- End frame

Pressing **Compute** always reads the current values and performs a new calculation.

This prevents an old result from being displayed after the measurement parameters have changed.

---

# Time units

Linear-speed results can be displayed using:

- `/s`
- `/min`
- `/h`

Examples:

```text
mm/s
m/min
ft/min
km/h
```

Rotational speed is displayed in:

```text
RPM
```

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

# MP4/MOV timing analysis

For compatible ISO Base Media / QuickTime-style files, VirtualTachometer reads video-track timing information directly from the container.

Supported timing structures include:

- `mdhd` — media timescale
- `stts` — sample timing
- `ctts` — composition offsets
- common `elst` edit lists

When a usable sample-time table can be generated, frame navigation uses actual sample timestamps instead of relying only on:

```text
1 / FPS
```

This is especially useful for Variable Frame Rate video.

---

# CFR and VFR

## CFR

Constant Frame Rate video is handled normally.

## VFR

Variable Frame Rate is detected from the timing table.

When exact sample timestamps are available:

- BWD/FWD uses real sample timing
- Frame indexing uses the sample-time table
- Measurement marks use displayed-frame timestamps whenever possible

If exact VFR timing is unavailable, VirtualTachometer avoids presenting average-FPS stepping as exact.

---

# Presented-frame confirmation

When supported by the browser, VirtualTachometer uses:

```javascript
HTMLVideoElement.requestVideoFrameCallback()
```

and its:

```text
mediaTime
```

to identify the frame actually presented.

After a seek, the application waits for frame confirmation whenever possible.

If the browser does not confirm the frame in time, VirtualTachometer falls back to:

```text
video.currentTime
```

as an approximate timestamp so **Start** and **End** remain usable.

---

# Accuracy and timing resolution

Video-based measurement is inherently limited by the temporal resolution of the recording.

Higher FPS generally improves timing resolution.

VirtualTachometer estimates timing resolution using local frame timing when available.

The displayed estimate does not include every possible physical source of uncertainty, including:

- Browser seek behavior
- Motion blur
- Rolling shutter
- Camera exposure
- Reference-mark selection error
- Incorrect physical distance or diameter
- Video compression artifacts

For better results:

1. Prefer high-frame-rate video.
2. Prefer CFR when possible.
3. Measure over longer intervals.
4. Use several complete revolutions.
5. Keep the camera stable.
6. Use clear, high-contrast reference marks.
7. Avoid excessive motion blur.

---

# Controls

## Video

- **Open Video** — select a local video file
- Play icon — start playback
- Pause icon — pause playback
- Fast Rewind icon — one frame backward
- Fast Forward icon — one frame forward
- `← / →` — keyboard frame navigation
- Progress slider — seek through the video

## Measurement

- **Start** — store the starting timestamp
- **End** — store the ending timestamp
- **Compute** — calculate using all current measurement parameters

## FPS

- **Frame rate (FPS)** — detected or manually entered frame rate
- **Autotune** — estimate FPS from displayed-frame timing when supported

## Interface

- Language selector — English / Español / Italiano
- Theme selector — Light / Dark

---

# Test videos

Test videos for checking FPS detection, frame navigation, timing, and measurement behavior are available in the repository:

https://github.com/FireRayo/VirtualTachometer/tree/main/assets

These files can be used to verify the application after changes to timing, seeking, FPS detection, or measurement logic.

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

Common MP4/H.264 files generally provide the widest browser compatibility.

---

# Offline use

V4.0 is designed to operate independently from the Internet.

Download:

```text
index.html
```

and open it directly in a modern browser.

All required application resources are contained in the HTML file itself.

No web font, CDN, remote library, or external UI dependency is required.

---

# Online use

https://firerayo.github.io/VirtualTachometer/

---

# Privacy

The selected video is processed locally in the browser.

VirtualTachometer does not require:

- A backend
- An account
- A cloud service
- Video upload

Users should still follow their organization's security and data-handling policies when working with industrial or confidential video.

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
    ├── test videos
    └── other project assets
```

---

# Installation

No installation is required.

## Option 1 — Online

https://firerayo.github.io/VirtualTachometer/

## Option 2 — Offline

Download `index.html` and open it directly in a modern browser.

## Option 3 — Clone the repository

```bash
git clone https://github.com/FireRayo/VirtualTachometer.git
cd VirtualTachometer
```

Then open:

```text
index.html
```

---

# Version history

## V4.0

- Complete Neomorphism / Soft UI redesign.
- Added complete Light and Dark themes.
- Added theme selector with persistent preference.
- Converted the application to a fully self-contained single HTML file.
- Removed external Material Symbols dependency.
- Added inline SVG playback/navigation icons.
- Removed all remote fonts, CDN resources, and external libraries.
- Reorganized the source code into clearer logical responsibilities.
- Added extensive internal documentation and comments in Spanish.
- Improved maintainability and diagnostic readability.
- Preserved all V3.9 measurement, timing, unit-conversion, CFR/VFR, and FPS features.
- Improved empty-field handling during unit conversion.
- Verified operation in both Light and Dark themes.

## V3.9

- Added automatic numeric conversion when changing Distance or Diameter units.
- Added conversion between `mm`, `cm`, `m`, `km`, `in`, `ft`, `yd`, and `mi`.
- Preserved physical measurement when changing units.

## V3.8

- Any measurement-variable change invalidates the previous result.
- **Compute** re-reads the current values and calculates from scratch.
- Updated handling for Distance, Diameter, revolutions, units, time basis, Start, and End.

## V3.7

- Added icon-based playback/navigation controls.

## V3.6

- Fixed Start/End becoming unavailable after some FWD/BWD operations.
- Added approximate `currentTime` fallback when displayed-frame confirmation times out.

## V3.1–V3.5

- Added MP4/MOV sample timestamp tables.
- Added `ctts` composition-offset handling.
- Added common `elst` edit-list handling.
- Improved VFR navigation.
- Added post-seek frame confirmation.
- Added seek timeout recovery.
- Added multiple revolutions to Diameter mode.
- Improved timing-resolution reporting.
- Removed silent nominal FPS snapping from calibrated FPS.

## V3.0

- Redesigned FPS handling.
- Added MP4/MOV timing detection.
- Added VFR detection.
- Removed hidden 30 FPS fallback.
- Added manual FPS input and recalibration.
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

Test videos:  
https://github.com/FireRayo/VirtualTachometer/tree/main/assets

---

## Disclaimer

VirtualTachometer is a measurement aid based on recorded video timing.

Results may be affected by frame rate, variable frame timing, compression, dropped frames, motion blur, camera exposure, browser decoding behavior, seek accuracy, and Start/End frame selection.

For safety-critical calibration, certification, regulatory verification, or metrology applications, use properly calibrated measurement equipment and an appropriate validated measurement procedure.
