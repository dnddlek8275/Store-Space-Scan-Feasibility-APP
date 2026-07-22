# UK Manager App - PoC Roadmap

## Goal

The first PoC verifies whether uploaded video can provide layout hints that are useful for an editable reservation map.

The goal is not to generate a perfect aerial-view image.

The goal is to validate the following flow:

```text
video -> key frames -> detection hints -> editable layout draft
```

The long-term pipeline is:

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

The first PoC covers only the front part of this flow:

```text
video input
-> frame extraction
-> frame quality analysis
-> object and structure hints
-> editable preview draft
```

## Why Start in This Order

Previous tests showed that image generation could create visually appealing aerial-view images, but could not reliably preserve the actual room shape, door positions, or object placement.

For a reservation app, accurate structure data is more important than visual polish.

Therefore, the first technical validation should focus on structured data generation rather than image generation.

## Step 1 - Local Video Frame Pipeline

Input:

- One `.mov` or `.mp4` file

Output:

- Frames extracted at regular time intervals
- A contact sheet for human review
- Basic metadata such as video duration, frame size, and sampled timestamps

Success criteria:

- Representative frames can be extracted reliably from user-captured videos.

Default extraction strategy:

- The default is `1 frame per second`.
- Around 8 sampled frames can be used for quick visual review.
- For real structure analysis, per-second extraction or scene-change-based extraction is more appropriate.
- Only frames that pass quality analysis should be sent to object detection models.

## Step 2 - Frame Quality Analysis

Analyze the following for each frame:

- Shake or blur
- Darkness
- Overexposure
- Duplicate or near-duplicate scenes
- Fast motion
- Lack of usable scene information

Example output:

```json
{
  "frameId": "frame_004",
  "timeSeconds": 8.2,
  "quality": {
    "blur": 0.12,
    "brightness": 0.64,
    "usable": true
  }
}
```

Success criteria:

- Low-quality frames can be filtered before object or structure analysis.

## Step 3 - Object and Structure Hint Detection

Detect candidate elements:

- Doors
- Windows
- Walls
- Room entrances or openings
- Tables
- Chairs
- Counters
- Sinks
- Shelves
- Bathroom fixtures
- Kitchen equipment

Example output:

```json
{
  "frameId": "frame_006",
  "objects": [
    {
      "type": "door",
      "confidence": 0.81,
      "box": {
        "x": 0.12,
        "y": 0.18,
        "width": 0.21,
        "height": 0.62
      }
    }
  ]
}
```

Success criteria:

- Usable object candidates can be produced even if they are imperfect.

## Step 4 - Layout Draft Generation

Combine frame-level hints into a rough draft:

- Rooms
- Doors
- Major fixed structures
- Reservable object candidates

Example output:

```json
{
  "layoutDraft": {
    "rooms": [
      {
        "id": "room_1",
        "label": "main_area",
        "xRatio": 0.2,
        "yRatio": 0.2,
        "widthRatio": 0.6,
        "heightRatio": 0.5
      }
    ],
    "doors": [],
    "objects": []
  }
}
```

Success criteria:

- The draft should be faster for the user to correct than drawing everything from scratch.

## Step 5 - Simple Editable Preview

Before building the full app, create a minimal preview.

Recommended first version:

- Local HTML/SVG preview
- Draggable handles for rooms, walls, doors, and objects
- JSON export

Success criteria:

- The edited JSON can be used as source data for the reservation map.

## Step 6 - Aerial-View Background Image Generation

Proceed only after the geometry data is confirmed.

```text
confirmed layout JSON + selected reference frames
-> aerial-view style background image
```

The generated image should follow the confirmed structure. However, reservation functionality should still operate from JSON coordinates.

Success criteria:

- The generated background image improves visual quality without changing the actual reservation object positions.

## Proposed Initial File Outputs

For each test video, generate:

```text
analysis/
  video_metadata.json
  frames/
  contact_sheet.jpg
  frame_quality.json
  object_hints.json
  layout_draft.json
  preview.html
```

## Technical Risk Notes

High-risk areas:

- Reconstructing accurate walls and room structure from ordinary smartphone video
- Matching the same object across multiple frames
- Handling L-shaped, U-shaped, square-loop, multi-room, and irregular spaces
- Distinguishing visible doors from actually accessible spaces
- Avoiding hallucinated rooms or objects

Lower-risk areas:

- Frame extraction
- Correction editor implementation
- Ratio-based object storage
- Rendering a reservation map from confirmed JSON

## Recommended First Implementation

Start with a local prototype script.

Tasks:

1. Accept a video path as input.
2. Extract representative frames.
3. Generate a contact sheet.
4. Generate `video_metadata.json`.
5. Generate a temporary `object_hints.json`.
6. Render a simple `preview.html`.

Once this pipeline exists, object detection and AI analysis can be added step by step.
