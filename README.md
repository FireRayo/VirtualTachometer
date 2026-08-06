# VirtualTachometer

[![Version](https://img.shields.io/badge/version-3.0-blue.svg)](https://github.com/FireRayo/VirtualTachometer)
[![HTML5](https://img.shields.io/badge/HTML5-single--file-orange.svg)](https://github.com/FireRayo/VirtualTachometer)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)

**VirtualTachometer** is a browser-based industrial tachometer for measuring **linear speed** and **rotational speed (RPM)** from recorded video.

It is designed for field work, commissioning, troubleshooting, maintenance, and machine validation when a physical tachometer is not available but a video can be recorded with a phone, camera, or other device.

**Live application:**  
https://firerayo.github.io/VirtualTachometer/

**Author:** Raymundo Ortiz  
**Current version:** V3.0

---

## Overview

VirtualTachometer converts a measured time interval in a video into a speed value.

Typical industrial applications include:

- Conveyor speed measurement.
- Package or bottle transport speed verification.
- Roller peripheral speed measurement.
- Shaft, pulley, wheel, or roller RPM measurement.
- Commissioning and troubleshooting of industrial machinery.
- Comparing commanded machine speed with observed mechanical speed.
- Performing quick field measurements when no physical tachometer is available.

The application runs entirely in the browser using HTML5, CSS, and JavaScript. No installation, backend, account, or external JavaScript library is required.

---

## What's new in V3.0

Version 3.0 introduces a major revision of video timing, frame navigation, measurement reliability, and user interface behavior.

### Improved video loading

- Loading a video no longer starts playback automatically.
- The video is loaded and left paused so the user can immediately select the desired Initial and Final frames.
- Previous video object URLs are released when another file is loaded to avoid unnecessary memory usage.
- Video decoding errors are detected and displayed inside the application instead of leaving the user with a black player and no explanation.

### Improved FPS handling

The previous approach depended on briefly playing the video to estimate its frame rate. V3.0 removes that behavior from normal video loading.

For compatible **MP4 / M4V / MOV** files, VirtualTachometer now attempts to read timing information directly from the container:

- `mdhd` is used to obtain the media timescale.
- `stts` is used to inspect sample timing.
- Total video samples/frames can be derived from the timing table.
- Average FPS is calculated from the actual container timing data.
- Variable Frame Rate (**VFR**) content is detected when multiple frame durations are present.

This means opening a supported MP4/MOV file does **not** require automatically playing a section of the video just to determine FPS.

### No hidden 30 FPS fallback

V3.0 no longer assumes 30 FPS when the real frame rate is unknown.

If FPS has not yet been detected or entered:

- Frame stepping is disabled.
- The application clearly reports that FPS is unavailable.
- The user can enter FPS manually.
- The user can explicitly run **Recalibrate FPS**.

This prevents incorrect ±1-frame navigation on 25, 50, 60, 120, 240 FPS, or other video sources.

### Manual FPS and recalibration

V3.0 provides:

- Automatic MP4/MOV timing inspection when possible.
- Manual FPS entry.
- A **Recalibrate FPS** button for files where direct container detection is unavailable or unsuccessful.
- Clear indication of the current FPS source:
  - Container metadata
  - Manual value
  - Calibrated value

Recalibration is explicit and is never started automatically when a video is opened.

### More stable frame stepping

Frame navigation has been redesigned to reduce the visible jumping or "vibration" that can occur when many asynchronous seeks overlap.

V3.0 includes:

- A seek lock using `isSeeking`.
- BWD/FWD requests are blocked while the previous seek is still being resolved.
- The UI is updated after the browser confirms the seek.
- Progress is no longer immediately refreshed with an unconfirmed requested timestamp.
- Slider scrubbing uses a **latest-wins** pending target instead of accumulating every intermediate `input` event.

This is particularly useful with long-GOP H.264/H.265 videos, where backward seeking can require decoding forward again from a previous keyframe.

### Presented-frame timestamp tracking

When supported by the browser, VirtualTachometer uses:

```text
requestVideoFrameCallback()
```

and stores the reported:

```text
mediaTime
```

This timestamp represents the media time associated with the video frame submitted for composition and is used as the preferred reference when the user marks **Initial** or **Final**.

If the API is unavailable, the application falls back to the HTML video element timing.

### Measurement uncertainty

V3.0 no longer presents a result without context about video timing resolution.

When a valid FPS and measurement interval are available, the application displays:

- Approximate temporal uncertainty of **±1 frame**.
- The equivalent uncertainty in seconds.
- Approximate propagated uncertainty in the calculated speed.
- Percentage uncertainty.

For a video with frame rate \(FPS\):

```text
Frame duration ≈ 1 / FPS
```

The shorter the measured event, the greater the relative influence of one frame.

### Multiple-revolution RPM measurement

The **Revolutions** mode now supports an **Observed revolutions** value.

Instead of measuring only one revolution, the user can measure several complete revolutions and calculate:

```text
RPM = 60 × Observed Revolutions / Elapsed Time
```

Measuring multiple revolutions significantly reduces the relative influence of ±1-frame timing uncertainty.

### Measurement values are preserved

Changing between:

- Distance
- Diameter
- Revolutions

no longer destroys and recreates the measurement inputs.

Previously entered values and selected units remain available when switching modes.

### Saved preferences

The application uses `localStorage` to retain useful interface preferences, including:

- Measurement mode.
- Distance value and unit.
- Diameter value and unit.
- Number of observed revolutions.
- Time unit.
- Interface language.

### Multilingual interface

V3.0 includes a complete interface in:

- English
- Spanish
- Italian

The selected language is saved locally. On a new browser profile, Spanish or Italian can also be selected automatically from the browser language when applicable.

---

## Main features

- Runs directly in a modern web browser.
- Single-file HTML5 application.
- No installation required.
- No server-side processing.
- Video remains local to the user's browser.
- Drag-and-drop video loading.
- Dedicated **Open video** button.
- Play/Pause controls.
- BWD/FWD frame stepping.
- Keyboard `←` / `→` frame stepping.
- Progress slider with controlled asynchronous seeking.
- Initial and Final timestamp markers.
- Automatic elapsed-time calculation.
- Linear speed calculation.
- Roller peripheral speed calculation from diameter.
- Rotational speed calculation in RPM.
- Multiple-revolution averaging.
- MP4/MOV container timing inspection.
- VFR indication.
- Manual FPS input.
- Manual FPS recalibration.
- Presented-frame timestamp tracking with `requestVideoFrameCallback()` when available.
- Approximate measurement uncertainty display.
- Video decoding/error messages inside the interface.
- English, Spanish, and Italian UI.
- Persistent preferences with `localStorage`.
- Responsive layout suitable for desktop and mobile browsers.

---

## Screenshot

![VirtualTachometer](assets/IMG01.JPG)

> The screenshot in the repository may show an earlier visual revision. The live GitHub Pages version reflects the current interface.

---

# Measurement modes

VirtualTachometer provides three measurement methods.

## 1. Distance

Use this mode when an object travels a known linear distance.

### Example applications

- Conveyor belt.
- Package transport.
- Bottle conveyor.
- Shuttle.
- Linear axis.
- Moving carriage.

### Procedure

1. Measure a known physical distance on the machine.
2. Place visible reference points at the beginning and end of that distance.
3. Record the moving object passing both reference points.
4. Open the video in VirtualTachometer.
5. Select **Distance**.
6. Enter the measured distance and unit.
7. Navigate to the frame where the moving reference reaches the first point.
8. Press **Initial**.
9. Navigate to the frame where it reaches the second point.
10. Press **Final**.
11. Press **Calculate**.

The application calculates:

```text
Elapsed Time = Final Time - Initial Time

Linear Speed = Distance / Elapsed Time
```

The selected time unit determines whether the result is displayed per second, per minute, or per hour.

Example:

```text
Distance = 1000 mm
Elapsed Time = 2.000 s

Speed = 1000 / 2
Speed = 500 mm/s
```

---

## 2. Diameter

Use this mode to calculate the peripheral or surface speed of a rotating roller, wheel, pulley, or similar component when its diameter is known.

### Procedure

1. Measure the roller diameter.
2. Put a visible reference mark on the rotating element.
3. Record one or more complete rotations.
4. Open the video.
5. Select **Diameter**.
6. Enter the diameter and unit.
7. Mark the start of one complete revolution with **Initial**.
8. Mark the end of that revolution with **Final**.
9. Press **Calculate**.

One circumference is:

```text
Circumference = π × Diameter
```

Therefore:

```text
Linear Speed = (π × Diameter) / Elapsed Time
```

Example:

```text
Diameter = 100 mm
Elapsed Time = 0.500 s

Circumference = π × 100
Circumference ≈ 314.159 mm

Speed ≈ 628.319 mm/s
```

---

## 3. Revolutions

Use this mode to calculate rotational speed directly in RPM.

### Procedure

1. Put a visible reference mark on the rotating element.
2. Record the rotation.
3. Open the video.
4. Select **Revolutions**.
5. Enter the number of complete observed revolutions.
6. Mark the beginning with **Initial**.
7. Mark the end with **Final**.
8. Press **Calculate**.

The calculation is:

```text
RPM = 60 × Revolutions / Elapsed Time
```

Example using multiple revolutions:

```text
Observed revolutions = 10
Elapsed Time = 4.000 s

RPM = 60 × 10 / 4
RPM = 150 RPM
```

For precision, measuring several revolutions is normally preferable to measuring only one.

---

# Supported units

Distance and diameter fields support:

| Metric | Imperial / US customary |
|---|---|
| mm | in |
| cm | ft |
| m | yd |
| km | mi |

Time basis:

- `/s`
- `/min`
- `/h`

Examples of possible linear-speed outputs:

```text
mm/s
mm/min
m/s
m/min
m/h
km/h
in/s
ft/min
yd/min
mi/h
```

Rotational measurements are displayed in:

```text
RPM
```

---

# FPS and frame navigation

## Why FPS matters

Frame stepping uses the video frame duration:

```text
Frame duration ≈ 1 / FPS
```

Examples:

| FPS | Approx. frame duration |
|---:|---:|
| 24 | 41.67 ms |
| 25 | 40.00 ms |
| 30 | 33.33 ms |
| 50 | 20.00 ms |
| 60 | 16.67 ms |
| 120 | 8.33 ms |
| 240 | 4.17 ms |

Using an incorrect FPS directly affects the requested ±1-frame seek interval.

For this reason, V3.0 does not silently substitute 30 FPS when the frame rate is unknown.

---

## MP4/MOV FPS detection

For compatible ISO Base Media / QuickTime-style files, VirtualTachometer inspects the video timing tables directly.

The application uses the video track timing information to calculate average FPS from:

```text
number of video samples × media timescale
-----------------------------------------
      total sample duration ticks
```

This approach avoids estimating the nominal FPS from computer playback speed during normal file loading.

---

## Variable Frame Rate (VFR)

Some smartphone and screen-recording videos use **Variable Frame Rate**.

For these files, individual frames may not all have the same duration.

VirtualTachometer detects this condition when the MP4 timing table contains multiple sample durations and marks the source as **VFR**.

The displayed FPS in this case is an **average FPS**.

### Important

For VFR video:

```text
1 / average FPS
```

is only an approximation for frame stepping.

The Initial/Final `mediaTime` tracking is therefore more meaningful than assuming every frame has exactly the same duration.

For highest measurement consistency, Constant Frame Rate (**CFR**) video is preferable.

---

# Accuracy and uncertainty

Video-based measurement is inherently limited by the temporal resolution of the source recording.

For an FPS value \(F\):

```text
Approximate one-frame timing resolution = 1 / F
```

VirtualTachometer reports an uncertainty based on approximately **±1 frame**.

The approximate relative contribution of this timing uncertainty is:

```text
Relative uncertainty ≈ Frame Duration / Measured Time
```

For example:

```text
Video = 30 FPS
Frame duration ≈ 0.0333 s

Measured event = 0.300 s
Relative timing uncertainty ≈ 11.1 %
```

but:

```text
Measured event = 10.000 s
Relative timing uncertainty ≈ 0.33 %
```

This demonstrates why longer observation intervals improve measurement quality.

---

# Recommendations for better measurements

For the best practical results:

1. **Record at the highest useful FPS available.**  
   60 FPS is better than 30 FPS, while 120 or 240 FPS can provide much finer timing resolution for fast motion.

2. **Measure over a longer interval.**  
   If possible, use a longer travel distance or measure several revolutions.

3. **Use multiple revolutions for RPM.**  
   Measuring 5 or 10 revolutions is usually more robust than timing a single revolution.

4. **Keep the camera stable.**  
   A fixed camera makes the reference crossing easier to identify.

5. **Use clear reference marks.**  
   High-contrast marks reduce ambiguity when selecting Initial and Final frames.

6. **Avoid motion blur when possible.**  
   Faster shutter speeds make frame selection more repeatable.

7. **Prefer CFR video for measurement work.**  
   Constant Frame Rate video provides more predictable frame intervals than VFR content.

8. **Verify FPS before frame stepping.**  
   Use the detected value, enter a known value manually, or run Recalibrate FPS.

9. **Be careful with long-GOP compressed video.**  
   MP4/H.264/H.265 video may contain keyframes separated by many dependent frames. Backward navigation can therefore take longer because the browser may need to decode again from an earlier keyframe.

---

# Frame-accurate behavior: important limitation

VirtualTachometer V3.0 significantly improves frame handling, but it still uses the browser's HTML `<video>` element for decoding and seeking.

A request such as:

```javascript
video.currentTime = target;
```

does not guarantee deterministic, sample-exact frame seeking on every codec, browser, or video file.

`requestVideoFrameCallback()` improves the application's knowledge of the frame actually presented by the browser, but it does not turn `<video>` seeking into a fully frame-indexed decoder.

Therefore:

- Frame stepping should be considered **best effort**.
- Long-GOP video can be slower when stepping backward.
- VFR files cannot be represented perfectly by a single fixed `1 / FPS` interval.
- Browser codec and container support still apply.

A future architecture requiring deterministic frame-level decoding would require a dedicated demuxer and frame decoder, for example a WebCodecs-based pipeline.

---

# Video compatibility

The application accepts video files supported by the current browser.

Actual playback support depends on:

- Browser.
- Operating system.
- Video container.
- Video codec.
- Audio codec.
- Device capabilities.

Common MP4/H.264 videos are generally the most portable choice across modern browsers.

If the browser cannot decode a file, V3.0 displays an error message inside the application.

Automatic container FPS extraction is specifically implemented for compatible:

```text
.mp4
.m4v
.mov
.qt
```

For other formats, video playback may still work if the browser supports them, but FPS may need to be entered manually or calibrated.

---

# Controls

## Video

- **Open video** — select a local video file.
- **PLAY / PAUSE** — start or stop playback.
- **BWD** — request one frame backward.
- **FWD** — request one frame forward.
- **← / →** — keyboard shortcuts for one-frame navigation.
- **Progress bar** — seek through the video.

## Measurement

- **Initial** — store the starting video timestamp.
- **Final** — store the ending video timestamp.
- **Calculate** — calculate elapsed time and speed.

## FPS

- **Frame rate (FPS)** — detected or manually entered frame rate.
- **Recalibrate FPS** — explicitly estimate FPS from presented video-frame timing when supported.

---

# Why the seek lock exists

Browser video seeking is asynchronous.

Without synchronization, holding an arrow key or rapidly moving the progress slider can generate many competing seek requests:

```text
Seek 1
Seek 2
Seek 3
Seek 4
...
```

The browser may complete those requests at different times, causing visible jumping or unstable frame positioning.

V3.0 uses a seek state:

```javascript
isSeeking
```

to avoid stacking BWD/FWD operations.

For slider movement, only the most recent pending position is retained while a seek is active. Once the current seek finishes, the latest requested position is applied.

This substantially reduces unnecessary seek contention.

---

# Privacy

VirtualTachometer processes the selected video locally in the browser.

The application does not require a backend service and does not intentionally upload the selected video to a server.

This makes it suitable for industrial environments where machine footage may be confidential.

As with any browser-based workflow, users should still follow their organization's security and data-handling policies.

---

# Installation

No installation is required.

## Option 1 — GitHub Pages

Open:

https://firerayo.github.io/VirtualTachometer/

## Option 2 — Run locally

Download:

```text
index.html
```

and open it in a modern browser.

Because the application is self-contained, no package manager, build process, or web server is required for normal use.

## Option 3 — Clone the repository

```bash
git clone https://github.com/FireRayo/VirtualTachometer.git
cd VirtualTachometer
```

Then open:

```text
index.html
```

in your browser.

---

# Repository structure

```text
VirtualTachometer/
├── index.html
├── VirtualTachometer.html
├── VirtualTachometer.zip
├── README.md
├── LICENSE
└── assets/
    └── IMG01.JPG
```

The main application is contained in a single HTML file with embedded CSS and JavaScript.

---

# Browser notes

VirtualTachometer is intended for modern browsers with HTML5 video support.

The best feature set is available when the browser supports:

```javascript
HTMLVideoElement.requestVideoFrameCallback()
```

If that API is not available, the application continues to operate using the available HTML video timing information, with reduced frame-presentation awareness.

Mobile behavior also depends on the browser's media implementation and the codecs available on the device.

---

# Version history

## V3.0

- Redesigned FPS handling.
- MP4/MOV timing-table FPS detection.
- VFR detection.
- Removed automatic FPS calibration during normal video loading.
- Removed hidden 30 FPS fallback.
- Added manual FPS entry.
- Added explicit FPS recalibration.
- Added seek locking for frame navigation.
- Added latest-wins progress-slider seeking.
- Added `requestVideoFrameCallback()` timestamp tracking.
- Improved Initial/Final timestamp capture.
- Added ±1-frame timing uncertainty.
- Added propagated speed/RPM uncertainty.
- Added multiple observed revolutions for RPM measurement.
- Preserved values and units while switching measurement modes.
- Added persistent preferences through `localStorage`.
- Added in-application video error reporting.
- Improved memory handling when replacing videos.
- Added complete English, Spanish, and Italian interfaces.
- Updated About information.
- Author identified as **Raymundo Ortiz**.

---

# License

This project is licensed under the **GNU General Public License v3.0 (GPL-3.0)**.

See:

[LICENSE](LICENSE)

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

Results can be affected by frame rate, variable frame timing, compression, dropped frames, motion blur, camera exposure, browser decoding behavior, seek accuracy, and the user's selection of Initial and Final frames.

For safety-critical calibration, certification, regulatory verification, or metrology applications, use properly calibrated measurement equipment and an appropriate validated measurement procedure.
