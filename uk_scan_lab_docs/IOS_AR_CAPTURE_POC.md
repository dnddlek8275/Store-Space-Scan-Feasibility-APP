# iOS AR Capture POC

## Purpose

This POC validates whether ARKit world tracking data from a regular iPhone can produce a more stable 2D path than the previous video-based COLMAP approach.

RoomPlan and LiDAR are not required for this POC.

## Created App

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture
```

Main files:

```text
project.yml
Sources/ARScanCapture/ARScanCaptureApp.swift
Sources/ARScanCapture/ContentView.swift
Sources/ARScanCapture/ARScanView.swift
Sources/ARScanCapture/ARSessionRecorder.swift
Sources/ARScanCapture/Info.plist
```

## Collected Data

During recording, the app saves the following roughly every 0.75 seconds:

```text
capturedImage -> frames/frame_000000.jpg
camera.transform -> cameraTransform
camera.intrinsics -> cameraIntrinsics
camera.trackingState -> trackingState
ARPlaneAnchor -> planes
```

Output location:

```text
App Documents/scan_sessions/{sessionId}/scan_session.json
App Documents/scan_sessions/{sessionId}/frames/*.jpg
```

## Current Local Validation State

At the beginning of this POC, the Mac had only Command Line Tools enabled and did not have the full Xcode app installed.

Checked state:

```text
xcode-select -p -> /Library/Developer/CommandLineTools
/Applications/Xcode.app -> missing
xcodegen -> not installed
Info.plist -> OK
```

Therefore, iOS app build validation could not be performed in that environment at that time.

## Device Validation Procedure

1. Install Xcode or open the project on a Mac with Xcode installed.
2. If using XcodeGen:

```bash
cd /Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture
xcodegen generate
open ARScanCapture.xcodeproj
```

3. If not using XcodeGen, create a new iOS App in Xcode and add the files from `Sources/ARScanCapture`.
4. Run on a physical iPhone.
5. In a store or corridor, tap `Start Scan` and walk slowly for 20 to 40 seconds.
6. Tap `Stop & Export` to save JSON.
7. Copy the app Documents `scan_sessions` folder to the Mac through Finder Devices, Xcode Devices, or file sharing.

## Post-Processing

Inspect the copied JSON with the existing renderer:

```bash
python3 /Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/render_scan_session_topdown.py \
  --input /path/to/scan_session.json \
  --outdir /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_scan_001
```

Things to check:

```text
whether the camera path resembles the actual walking path
whether L-shaped or turning sections remain stable
whether ARPlaneAnchor accumulates as wall/floor candidates
whether the path is more stable than ordinary MOV+COLMAP
```

## Passing Criteria

Primary pass:

```text
scan_session.json is generated successfully
frames/*.jpg are saved successfully
SVG is generated through render_scan_session_topdown.py
camera path roughly matches the actual movement direction
```

Secondary pass:

```text
some wall/floor plane candidates accumulate
the X/Z path does not jump significantly when the capture direction changes
```

## 2026-07-07 First Physical Device Result

Test device:

```text
iPhone 16
iOS 26.5
```

Container:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-07 16:26.03.102.xcappdata
```

Sessions:

```text
ios_arkit_20260707_161801
ios_arkit_20260707_161809
```

Main analysis target:

```text
ios_arkit_20260707_161809
```

Results:

```text
frameCount: 62
durationMs: about 45,757
planeCount: 625
vertical planes: 524
horizontal planes: 101
classification wall: 482
classification floor: 101
classification seat: 42
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_161809/scan_path_topdown.svg
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_161809/scan_path_topdown.png
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_161809/frame_contact_sheet.jpg
```

Judgment:

```text
Top-down path generation from ARKit cameraTransform succeeded.
The camera path accumulated continuously without COLMAP.
ARPlaneAnchor also collected enough wall/floor candidates.
However, the current planes are raw anchors with heavy overlap and noise, so wall-line merging post-processing is required.
Saved frame orientation was rotated, so image orientation correction is needed.
```

Next fixes:

```text
1. Correct orientation when saving capturedImage.
2. Merge raw ARPlaneAnchor rectangles into wall_line_candidate.
3. Render horizontal and vertical planes separately in the top-down renderer.
4. Re-capture in an indoor corridor or store interior using the same procedure.
```

## 2026-07-07 Capture Format Fix

The first physical device result revealed the following issues:

```text
portrait capture frames were saved in landscape orientation
ARPlaneAnchor was drawn only with center + extent, making vertical plane position/direction inaccurate
```

Fixes:

```text
ARSessionRecorder:
- save frame.imageOrientation
- save capturedImage as portrait JPEG
- save ARPlaneAnchor.transform
- save four ARPlaneAnchor.worldCorners

render_scan_session_topdown.py:
- if worldCorners exist, render polygons based on corners instead of center/extent
- render planes as polygons in both SVG and PNG

schema:
- add imageOrientation
- add plane.transform
- add plane.worldCorners
```

Validation:

```text
renderer py_compile: OK
synthetic sample render: OK
iPhone device build: OK
```

Next capture guide:

```text
capture in portrait orientation
walk slowly for about 20 to 40 seconds
hold the phone so walls and floor are visible together
do not rotate quickly in one spot
move slowly through corners and L-shaped sections
do not look at only one wall; alternate between left and right walls
after Stop & Export, import the new AppData again
```

## 2026-07-08 Indoor Re-Capture Result

Container:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata
```

New session:

```text
ios_arkit_20260707_195951
```

Results:

```text
frameCount: 60
durationMs: about 44,154
planeCount: 406
worldCorners planes: 406
imageOrientation: portrait
horizontal planes: 307
vertical planes: 99
classification none: 313
classification seat: 49
classification floor: 43
classification door: 1
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951/scan_path_topdown.png
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951/scan_path_topdown.svg
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951/frame_contact_sheet.jpg
```

Judgment:

```text
Frame orientation correction succeeded.
Plane rendering based on worldCorners was less distorted than the previous center/extent approach.
The camera path stayed connected as a single movement path.
However, in an indoor environment with many objects, ARKit plane classification produced almost no wall classifications.
The current result is still insufficient for automatic floor-plan generation.
```

Next fix candidates:

```text
1. Use floor/large horizontal planes to create candidates for usable indoor area.
2. Judge vertical planes as wall candidates using geometric conditions rather than classification.
3. Remove planes that are too small or too short.
4. Promote only vertical planes that repeat across multiple frames into wall_line_candidate.
5. Consider collecting ARFrame.rawFeaturePoints or hit-test samples.
```

## 2026-07-08 Plane Candidate Post-Processing Result

Added tool:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/extract_arkit_plane_candidates.py
```

Purpose:

```text
Extract wall-line candidates from ARKit raw planes using alignment + worldCorners + repeated observation count,
instead of treating raw planes as object bounding boxes.
```

Execution:

```bash
python3 /Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/extract_arkit_plane_candidates.py \
  --input "/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata/AppData/Documents/scan_sessions/ios_arkit_20260707_195951/scan_session.json" \
  --outdir /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_plane_candidates_flip_z \
  --orientation flip_z
```

Results:

```text
orientation: flip_z
rawPlaneObservations: 406
dedupedPlaneAnchors: 12
dedupedHorizontalPlanes: 9
dedupedVerticalPlanes: 3
rawWallLineCandidateObservations: 98
dedupedWallLineCandidateCount: 2
wallLineClusterCount: 2
```

Wall-line candidate clusters:

```text
1. support 53, frames 7~59, length 0.90m, angle 23.0deg, classification none
2. support 45, frames 15~59, length 1.41m, angle 25.0deg, classification none
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_plane_candidates_flip_z/arkit_plane_candidates.json
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_plane_candidates_flip_z/arkit_plane_candidates.png
```

Judgment:

```text
Based on the user's visual judgment, flip_z was the correct path orientation.
ARKit planes are plane candidates, not object bounding boxes.
In this session, two repeatedly observed vertical planes were extracted as stable wall-line candidates.
However, there were not enough wall candidates to create the full space outline.
Therefore, the next step should not be automatic floor-plan generation using ARKit planes alone.
It needs to combine ARKit path/scale with image-based wall boundary and object candidates.
```

## 2026-07-08 Depth-Based Wall Outline Review

Concern:

```text
If wall outlines are detected only from RGB edges, furniture, shelves, lighting, and floor patterns can easily mislead the system.
Walls can also be occluded by furniture, so depth signals should be considered together.
```

Execution:

```bash
/Users/apple/Desktop/ungskitchen/uk_manger_app/.venv_depth/bin/python \
  /Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/video_poc/run_depth_anything.py \
  --repo-dir /Users/apple/Desktop/ungskitchen/uk_manger_app/vendor/Depth-Anything-V2 \
  --checkpoint /Users/apple/Desktop/ungskitchen/uk_manger_app/vendor/Depth-Anything-V2/checkpoints/depth_anything_v2_vits.pth \
  --img-dir "/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata/AppData/Documents/scan_sessions/ios_arkit_20260707_195951/frames" \
  --outdir /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_vits \
  --encoder vits \
  --input-size 518
```

Depth Anything V2 result:

```text
model: Depth Anything V2
encoder: vits
device: mps
inputSize: 518
imageCount: 60
```

First depth boundary result:

```text
processedFrameCount: 60
totalLineCandidates: 0
totalRegionCandidates: 1037
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_vits
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_boundaries/depth_boundary_contact_sheet.jpg
```

Hough line detection did not produce good wall lines from the depth maps.
Instead, depth contours detected many object, furniture, and foreground regions.

Additional tool:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/extract_depth_lsd_candidates.py
```

OpenCV LSD-based depth line result:

```text
analyzedFrameCount: 21
vertical_depth_boundary: 27
lower_diagonal_depth_boundary: 17
lower_horizontal_depth_boundary: 1
other_depth_boundary: 2
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_lsd/depth_lsd_contact_sheet.jpg
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_lsd/depth_lsd_candidates.json
```

Judgment:

```text
Depth Anything V2 shows front/back relationships better than RGB edges.
However, the current result is not enough to directly generate wall outlines.
Depth line candidates react strongly to nearby furniture, shelves, and object boundaries rather than walls.
Therefore, depth should first be used to mask foreground obstacles/furniture and exclude them from wall candidates,
rather than to directly confirm wall lines.
```

Next candidates:

```text
1. Create nearby foreground object masks from depth maps.
2. Exclude RGB/depth edges inside foreground mask regions from wall candidates.
3. Re-evaluate ARKit vertical planes, floor boundaries, and image-line candidates in the remaining background region.
4. Promote only candidates that repeatedly project near the same world coordinates across multiple frames.
```

## 2026-07-08 Re-Testing Existing COLMAP Method on ARKit-Saved Frames

Question:

```text
Can the same capture data be tested with the previous video experiment method without ARKit coordinates?
```

Input:

```text
60 frames/*.jpg saved by the ARKit app, not the original MOV
frame save interval was about 0.75 seconds
```

Execution:

```bash
/opt/homebrew/bin/colmap feature_extractor \
  --database_path /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/database.db \
  --image_path "/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata/AppData/Documents/scan_sessions/ios_arkit_20260707_195951/frames" \
  --ImageReader.single_camera 1

/opt/homebrew/bin/colmap sequential_matcher \
  --database_path /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/database.db

/opt/homebrew/bin/colmap mapper \
  --database_path /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/database.db \
  --image_path "/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata/AppData/Documents/scan_sessions/ios_arkit_20260707_195951/frames" \
  --output_path /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/sparse
```

Results:

```text
feature extraction: all 60 frames processed
sequential matching: completed
mapper: created one sparse/0 model
registered cameras: 5
sparse points: 628
registered frames:
- frame_000005.jpg
- frame_000006.jpg
- frame_000007.jpg
- frame_000008.jpg
- frame_000009.jpg
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/sparse/0
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_colmap_frames/projections/projection_xz.png
```

Judgment:

```text
The existing COLMAP method could not connect all 60 saved frames into one path.
Only the first 5 frames were registered, so it could not be used for full spatial structure or wall-outline generation.
Many frames had enough features, but failures repeated during middle-section connection and good initial pair construction.
For this dataset, using video frames alone without ARKit pose/scale is not suitable as the main path.
```

## 2026-07-08 ARKit + Depth Foreground Mask Fusion

Goal:

```text
1. Mask nearby furniture/obstacle regions with depth.
2. Exclude RGB/depth edges from those regions from wall candidates.
3. Re-evaluate ARKit vertical planes + floor boundaries + image lines in the remaining background area.
4. Promote only candidates that repeat near the same world coordinates across multiple frames.
```

Added tool:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/fuse_arkit_depth_wall_candidates.py
```

Input:

```text
scan_session.json
Depth Anything V2 grayscale depth maps
orientation: flip_z
```

Main execution:

```bash
python3 /Users/apple/Desktop/ungskitchen/uk_scan_lab/tools/fuse_arkit_depth_wall_candidates.py \
  --input "/Users/apple/Desktop/ungskitchen/uk_scan_lab/ios/ARScanCapture/com.ungskitchen.scanlab.ARScanCapture 2026-07-08 09:08.43.657.xcappdata/AppData/Documents/scan_sessions/ios_arkit_20260707_195951/scan_session.json" \
  --depth-dir /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_depth_vits/grayscale \
  --outdir /Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_fused_wall_candidates_p85_floorlines_strict \
  --orientation flip_z \
  --near-percentile 85 \
  --debug-every 12
```

Results:

```text
estimatedFloorY: -0.65632415
foregroundMeanPixelRatio: 0.1524
verticalPlaneProjectionCount: 99
verticalPlaneProjectionSuccessCount: 20
acceptedWallObservationCount: 95
rejectedWallObservationCount: 4
wallClusterCount: 2
promotedWallClusterCount: 2
floorCandidateCount: 5
floorImageLineCandidateCount: 219
floorImageLineClusterCount: 39
promotedFloorImageLineClusterCount: 14
```

Promoted ARKit wall clusters:

```text
1. support 51, frames 7~59, length 0.90m, meanForegroundOverlapRatio 0.024
2. support 44, frames 15~59, length 1.41m, meanForegroundOverlapRatio 0.000
```

Outputs:

```text
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_fused_wall_candidates_p85_floorlines_strict/fused_wall_candidates.json
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_fused_wall_candidates_p85_floorlines_strict/fused_wall_candidates_topdown.png
/Users/apple/Desktop/ungskitchen/uk_scan_lab/analysis/ios_arkit_20260707_195951_fused_wall_candidates_p85_floorlines_strict/foreground_mask_contact_sheet.jpg
```

Judgment:

```text
Depth foreground masks are useful for separating nearby furniture and obstacles.
Two ARKit vertical planes had low depth foreground overlap and remained as wall candidates.
However, no additional wall clusters were created.
Back-projecting image floor-lines onto the ARKit floor plane increases candidates, but it is currently too noisy to promote directly into wall candidates.
Therefore, image floor-lines should remain auxiliary/diagnostic signals for now, not confirmed wall signals.
```

Next technical direction:

```text
1. Keep foreground masks for furniture/foreground removal.
2. Promote ARKit vertical planes using repeated observation + foreground-overlap rules.
3. Apply stronger constraints to image-line floor back-projection:
   - distance limit around camera path
   - whether the line is inside or near the floor plane
   - whether it re-projects to the same world coordinate from multiple angles
   - relation to ARKit horizontal/floor candidates
4. Mark sections with insufficient wall candidates as automatic re-capture guidance targets.
```
