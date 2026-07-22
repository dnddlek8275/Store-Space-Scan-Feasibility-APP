# UK Manager App - Space Scan Technical Plan

## Purpose

This document summarizes the product idea and technical direction for `uk_manger_app`.

The goal is to help store operators or managers create customer-facing reservation maps quickly from photos, videos, or drawings, instead of building them manually from scratch.

The reference project, `uk_app`, lets customers reserve seats or tables on top of an aerial-style store image. The new project explores automating the creation of those reservation maps.

## Core Product Idea

The app should accept store materials and generate an editable reservation map.

Possible inputs:

- Store photos
- Store videos
- Rough hand-drawn floor plans
- Later stage: guided real-time scanning

Possible outputs:

- Editable floor plan
- Wall, room, door, and object layers
- Reservable table or seat objects
- Aerial-view style background image
- JSON data usable by the customer-facing reservation app

Important product distinction:

- AI-generated imagery is visual background material.
- Reservation objects must be structured data.
- Wall, room, door, and table positions must be editable and confirmable by the user.

## Current Judgment

Asking an image generation model to directly convert photos or videos into an accurate aerial view is not reliable enough.

Observed problems:

- An L-shaped structure can be changed into a straight corridor.
- Unverified areas behind doors can be hallucinated.
- Door, wall, and structure positions can shift.
- Even with a rough floor-plan sketch, results can look visually plausible while remaining structurally wrong.

This does not mean the project is impossible. It means the generated image must not be used as the source of truth for the reservation structure.

## Source-of-Truth Data

The source of truth should be structured layout data, not a generated image.

Recommended layout model:

```json
{
  "rooms": [],
  "walls": [],
  "doors": [],
  "objects": [
    {
      "id": "table_1",
      "type": "table",
      "xRatio": 0.42,
      "yRatio": 0.31,
      "widthRatio": 0.12,
      "heightRatio": 0.08,
      "reservable": true
    }
  ],
  "background": {
    "type": "generated_image",
    "assetPath": ""
  }
}
```

This matches the direction of the existing `uk_app`, where table positions are defined as ratio coordinates over an aerial-view image.

## Recommended Technical Direction

Use a hybrid pipeline:

```text
video or photos
-> frame extraction
-> object and structure detection
-> draft layout JSON generation
-> user correction
-> confirmed reservation object layer
-> optional aerial-view style background image generation
-> customer reservation app data generation
```

The first technical goal is not to create a beautiful aerial-view image. The first goal is to create a useful editable draft.

## Target Technical Pipeline

The long-term ideal technical flow is:

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

This is the core technical direction for the product.

Role separation:

- Camera movement and spatial-structure estimation creates spatial hints.
- Detection finds visible elements such as walls, doors, floors, and furniture.
- Floor-plan coordinate conversion creates editable geometry data.
- User correction confirms the geometry.
- Aerial-view image generation creates visual background material.
- Reservation object layer generation creates the actual selectable reservation data.

The reservation layer must operate from user-confirmed coordinates, not from pixels in a generated aerial-view image.

## Video Analysis vs Real-Time Analysis

### Video Analysis

Video analysis is recommended for the first PoC.

Advantages:

- Lower implementation difficulty
- Easier debugging
- Can be processed locally or on a server after upload
- Easier to combine multiple models
- Less dependent on mobile device performance
- Easier to support both iOS and Android later

Risks:

- Poor capture quality may only be discovered after upload.
- It is difficult to provide enough real-time guidance while the user is recording.

### Real-Time Analysis

Real-time analysis should be treated as a later-stage feature.

Advantages:

- Capture quality can be guided immediately.
- The app can warn about fast movement, dark lighting, shake, or missing angles.
- Guided scan UX becomes better.

Risks:

- Implementation difficulty increases significantly.
- iOS and Android support different capabilities.
- ARKit, ARCore, LiDAR, Depth, and SLAM support vary by device.
- Battery, heat, latency, and model size become product issues.

Recommended order:

```text
1. Offline video analysis
2. Manual correction UI
3. Capture quality checks
4. Semi-real-time capture guidance
5. Advanced real-time space scanning
```

## Technical Candidates

### Frame Extraction

Purpose:

- Convert video into representative frames.
- Remove duplicate, blurry, or unusable frames.

Possible tools:

- OpenCV
- FFmpeg
- Native iOS/Android video APIs

### Object Detection

Purpose:

- Detect layout hints such as doors, tables, chairs, windows, kitchen counters, sinks, and shelves.

Possible tools:

- YOLO-family models
- MediaPipe object detection
- Vision-language models for frame descriptions

### Segmentation

Purpose:

- Separate floors, walls, doors, furniture, and large structural surfaces.

Possible tools:

- SAM-family models
- Mask R-CNN-style segmentation
- Mobile segmentation models for later real-time features

### Spatial Reconstruction

Purpose:

- Estimate camera movement and rough spatial relationships from video.

Possible tools:

- SLAM
- Visual-inertial odometry
- COLMAP
- ARKit
- ARCore
- Apple RoomPlan on supported iOS devices

Important note:

Spatial reconstruction can help, but it does not automatically create a clean reservation-ready floor plan. Post-processing and user correction are still required.

### Image Generation

Purpose:

- Generate an aerial-view style background image after the layout is confirmed.

Recommended role:

- Visual quality improvement only
- Not used as source-of-truth data
- Must follow confirmed wall, room, door, and object positions

## MVP Scope

The first MVP should focus on:

```text
uploaded video
-> extracted frames
-> detected structure/object hints
-> editable floor-plan draft
-> confirmed reservation object JSON
```

Do not include in the first MVP:

- Fully automatic perfect floor-plan generation
- Fully automatic 3D reconstruction
- Real-time AR scanning
- Floor-plan results created only by image generation
