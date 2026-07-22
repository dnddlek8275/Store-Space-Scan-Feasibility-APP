# Space Scan Validation Plan

## Background

The previous experiment used ordinary MOV video as input and attempted the following:

```text
frame extraction
COLMAP camera path reconstruction
Depth Anything V2 depth estimation
OpenCV structural line candidate extraction
YOLO object candidate extraction
Open3D plane segmentation
2D floor-plan candidate generation
```

As a result, YOLO object candidates and some wall-line candidates were obtained, but the output was not sufficient to be considered an automatic floor plan.

Main causes:

```text
ordinary video does not provide stable camera poses
there is no real-world scale
structure estimation is weak unless walls are sufficiently observed
it is difficult to structurally separate doors, walls, and furniture from 2D video alone
```

## New Hypothesis

Data captured through in-app AR scanning is more suitable for floor-plan generation than ordinary video files.

Validation questions:

```text
Can ARKit/ARCore camera pose alone produce a more stable 2D path than COLMAP?
Can plane anchors and hit-test samples improve wall/floor boundary candidates?
Can general ARCore/ARKit devices without LiDAR produce usable structure candidates?
Does attaching YOLO object candidates to the AR coordinate system improve table/seat draft quality?
```

## MVP Validation Scope

The first goal is not a fully automatic floor plan.

Primary validation goals:

```text
define an app scan data collection format
create a sample scan_session.json
visualize the camera path through AR coordinate X/Z projection
display plane, hit-test, and object candidates in the same 2D coordinate system
compare stability against previous COLMAP results
```

## Input Data

Required:

```text
session metadata
frame timestamp
frame image path
camera transform 4x4
camera intrinsics
tracking state
```

Collect if possible:

```text
plane anchors
hit-test points
raw feature points
depth map path
YOLO detections
```

## Output Data

Primary outputs:

```text
scan_path_topdown.png
scan_session_summary.json
layout_candidate.json
```

`layout_candidate.json` should include only candidates:

```text
camera_path
floor_plane_candidates
wall_line_candidates
table_candidates
seat_candidates
coverage_gaps
```

## Success Criteria

Continue if at least two of the following improve compared to ordinary MOV + COLMAP:

```text
camera path does not break
rotation or L-shaped movement matches the actual capture flow
scale is preserved in AR world units instead of arbitrary units
wall/floor/object candidates accumulate stably in the same coordinate system
the amount of human correction is reduced
```

## Failure Criteria

Reduce or pause the AR scan path if:

```text
AR tracking is not stable enough on general devices in store environments
plane/depth data is barely usable on non-LiDAR devices
floor-plan quality improvement is small compared to app implementation complexity
```

## Next Tasks

1. Create a fake sample session based on `scan_session.schema.json`.
2. Create a tool that renders the sample session as a 2D top-down image.
3. Build a real AR capture POC on either iOS or Android.
4. Visualize the real captured JSON with the same renderer.

## 2026-07-07 Initial Validation

New experiment folder:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab
```

Created files:

```text
README.md
docs/VALIDATION_PLAN.md
schemas/scan_session.schema.json
samples/synthetic_l_shape_scan_session.json
tools/render_scan_session_topdown.py
```

The previous video analysis tools were copied while preserving the original.

```text
uk_manger_app/tools/video_poc
-> uk_scan_lab/tools/video_poc
```

Minimum validation:

```text
input: samples/synthetic_l_shape_scan_session.json
tool: tools/render_scan_session_topdown.py
output: analysis/synthetic_l_shape_001/scan_path_topdown.svg
summary: analysis/synthetic_l_shape_001/scan_session_summary.json
```

Result:

```text
frameCount: 5
cameraPathPointCount: 5
simplifiedCameraPathPointCount: 5
planeCount: 4
hitTestPointCount: 10
detectionCount: 4
```

Judgment:

```text
An AR-world-coordinate session can be projected directly to 2D X/Z top-down view without COLMAP.
Because this was synthetic data, it does not prove real-world performance.
However, if iOS/Android can generate the same scan_session.json format in the next POC, it can connect directly to the post-processing pipeline.
```

Next real validation:

```text
1. Build a frame + cameraTransform capture POC with either iOS ARKit or Android ARCore.
2. Scan a real store or corridor for 20 to 40 seconds.
3. Generate scan_session.json.
4. Use the same renderer to inspect the top-down path and accumulated plane/hit-test data.
5. Compare path stability against the previous MOV+COLMAP results.
```

## 2026-07-07 iOS POC Start

The team decided to validate iOS first.

Created POC:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture
```

Technical choices:

```text
ARKit ARWorldTrackingConfiguration
horizontal/vertical planeDetection
ARSCNView + ARSessionDelegate
SwiftUI control overlay
```

Current local state:

```text
Info.plist lint: OK
Xcode.app: missing
xcode-select: /Library/Developer/CommandLineTools
xcodegen: missing
```

iOS build validation could not be performed locally. Real validation requires an environment with Xcode installed and a physical iPhone.

Detailed procedure:

```text
docs/IOS_AR_CAPTURE_POC.md
```
