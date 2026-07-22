# Technical Feasibility Log

## 2026-07-06 - Video Analysis PoC 1

### Validation Goal

This test checked whether the first stage of the video-based space analysis pipeline could run locally.

Validation scope:

```text
extracted frames
-> frame quality analysis
-> structure hint generation
-> JSON output generation
-> contact sheet generation
-> HTML preview generation
```

This validation did not include a real object detection model, spatial-structure estimation, or floor-plan coordinate conversion.

### Input Used

- Original video: `/Users/apple/Downloads/IMG_0914.mov`
- Analysis frames: `/Users/apple/PycharmProjects/ungskitchen/tmp_video_frames/IMG_0914_frames`
- Frame count: 8

### Generated Results

Result location:

```text
/Users/apple/Desktop/ungskitchen/uk_manger_app/analysis/IMG_0914
```

Generated files:

```text
video_metadata.json
frame_quality.json
object_hints.json
layout_draft.json
contact_sheet.jpg
preview.html
frames/
```

### Result Summary

Frame quality analysis:

- Total frames: 8
- Usable: 7
- Needs review: 1

Frame needing review:

- `frame_005`
- Reason: the frame was close to a plain wall and lacked edge information.

### What Is Currently Possible

- A folder of extracted frames can be analyzed for quality scores.
- Brightness, contrast, edge score, and sharpness score can be calculated.
- Usable frames and frames needing review can be separated for indoor videos.
- A dominant orientation value can be generated as a frame-level structure hint.
- Analysis results can be generated as JSON, contact sheet, and HTML preview.

### Current Limitations

- Direct frame extraction from a video file was not stabilized yet.
- The local default Python environment did not include OpenCV, Pillow, or NumPy, but the Codex bundled Python included Pillow and NumPy and was used instead.
- The Swift-based frame extractor did not run immediately because of a CommandLineTools Swift/SDK mismatch.
- The current `object_hints.json` is a placeholder for the next stage, not a real object detection result.
- The current `layout_draft.json` is an empty draft, not a real floor-plan coordinate conversion result.

### Judgment

The preprocessing stage of the video analysis pipeline is feasible.

However, in the full target flow:

```text
video input
-> frame extraction
-> camera movement and spatial-structure estimation
-> wall, door, floor, and furniture detection
-> conversion to floor-plan coordinates
-> user correction
-> aerial-view image generation
-> reservation object layer generation
```

Only the following range was validated:

```text
extracted frames
-> frame quality analysis
-> basic structure hints
-> analysis output files
```

### Next Validation Steps

1. Stabilize direct frame extraction from video files.
   - Options: install FFmpeg, fix the Swift/Xcode toolchain, or process on a server.

2. Attach an object detection model.
   - Initial candidates: doors, windows, sinks, kitchen counters, washing machines, room entrances.

3. Create logic to group the same object across frames.
   - Check whether the same door, window, or kitchen fixture appears repeatedly across multiple frames.

4. Add real candidate data to `layout_draft.json`.
   - Start as a candidate layer for user correction, not as exact floor-plan coordinates.

5. Create a simple editable preview.
   - HTML/SVG-based
   - Drag-to-correct rooms, doors, and object candidates

## 2026-07-06 - Per-Second Frame Extraction Expansion

### Changes

Installed `ffmpeg` and added an integrated runner script that extracts frames directly from video files.

Added file:

```text
tools/video_poc/run_video_analysis.py
```

Execution flow:

```text
video file
-> extract 1 frame per second with ffmpeg
-> frame quality analysis
-> generate object_hints.json
-> generate layout_draft.json
-> generate contact_sheet.jpg
-> generate preview.html
```

### Example Command

```text
/Users/apple/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3 \
  /Users/apple/Desktop/ungskitchen/uk_manger_app/tools/video_poc/run_video_analysis.py \
  /Users/apple/Downloads/IMG_0914.mov \
  /Users/apple/Desktop/ungskitchen/uk_manger_app/analysis/IMG_0914_per_second \
  --fps 1 \
  --width 720
```

### Result Summary

Result location:

```text
/Users/apple/Desktop/ungskitchen/uk_manger_app/analysis/IMG_0914_per_second
```

Results:

- Extracted frames: 36
- Usable: 14
- Needs review: 22

### Judgment

An 8-frame sample is enough for quick visual review, but insufficient for structure estimation and object matching.

For real analysis, `1 frame per second` is more appropriate as the default. Later stages should not send every frame to the model; only frames that pass quality analysis should move to object detection or structure estimation.

Possible improvements:

- Scene-change-based frame extraction
- Duplicate frame removal
- Frame prioritization based on doors, windows, or furniture presence
- Frame selection using both quality scores and object detection results

## 2026-07-06 - OpenAI Vision Analysis Script

### Purpose

Only frames that pass quality analysis are sent to the OpenAI Vision API to generate frame-level structure and object hints.

Added file:

```text
tools/video_poc/analyze_with_openai.py
```

Input:

```text
analysis/IMG_0914_per_second/frame_quality.json
analysis/IMG_0914_per_second/frames/
```

Output:

```text
analysis/IMG_0914_per_second/object_hints_openai.json
analysis/IMG_0914_per_second/raw_openai_responses/
```

### Execution Condition

The `OPENAI_API_KEY` environment variable is required.

Example:

```text
export OPENAI_API_KEY="YOUR_API_KEY"

/Users/apple/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3 \
  /Users/apple/Desktop/ungskitchen/uk_manger_app/tools/video_poc/analyze_with_openai.py \
  /Users/apple/Desktop/ungskitchen/uk_manger_app/analysis/IMG_0914_per_second \
  --max-frames 8
```

### Design Judgment

- Do not send all frames to the API.
- Among frames with `usable: true`, send the highest-quality frames first.
- Analyze up to 8 frames by default.
- Limit analyzed frames with `--max-frames` to reduce cost and errors.
- The output is frame-level observation hints, not floor-plan coordinates.

### 8-Frame API Analysis Result

Execution result:

```text
model: gpt-5.4-mini
analyzedFrameCount: 8
output: analysis/IMG_0914_per_second/object_hints_openai.json
```

Frame summaries:

```text
frame_033: storage  | washing_machine, fixture, window, wall, floor, corner
frame_032: bathroom | washing_machine, fixture, window, wall, floor, corner
frame_007: kitchen  | appliance, fixture, shelf, window, wall, floor, ceiling, corner
frame_029: bathroom | washing_machine, fixture, floor, wall, corner, window
frame_002: kitchen  | cabinet, kitchen_counter, fixture, doorway, door, wall, floor, corner
frame_006: storage  | fixture, wall, floor, window, corner
frame_001: corridor | wall, floor, corner, room_opening
frame_015: corridor | door, wall, floor, corner, room_opening
```

Judgment:

- Structural elements such as doors, windows, walls, floors, and corners were detected to some degree.
- Major objects such as washing machines, fixtures, kitchen counters, and cabinets were also detected.
- Some space-type labels were ambiguous, such as `bathroom` vs `storage`.
- The current result should be treated as candidate data for post-processing and user correction, not as confirmed data for floor-plan generation.
