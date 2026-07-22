# Final Summary of the Space Scan Experiment

## Status

As of 2026-07-08, the space scanning and automatic floor-plan generation experiment was fully stopped.

The following project folders and large experiment outputs were deleted.

```text
/Users/apple/Desktop/ungskitchen/uk_manger_app
/Users/apple/Desktop/ungskitchen/uk_scan_lab
```

Only the experiment result documents were preserved.

## Preserved Documents

```text
/Users/apple/Desktop/ungskitchen/experiment_results/uk_manger_app_docs/POC_ROADMAP.md
/Users/apple/Desktop/ungskitchen/experiment_results/uk_manger_app_docs/TECH_FEASIBILITY_LOG.md
/Users/apple/Desktop/ungskitchen/experiment_results/uk_manger_app_docs/SPACE_SCAN_TECH_PLAN.md
/Users/apple/Desktop/ungskitchen/experiment_results/uk_scan_lab_docs/VALIDATION_PLAN.md
/Users/apple/Desktop/ungskitchen/experiment_results/uk_scan_lab_docs/IOS_AR_CAPTURE_POC.md
```

Any `/Users/apple/Desktop/ungskitchen/...` paths that remain inside the documents are historical paths from the time of the experiment.
The original project folders and analysis output files have been deleted.

## Final Decision

The core conclusion of this experiment was:

```text
Based on the current results, the assumption that free-form video or ARKit frames alone can automatically generate reliable store floor plans or wall outlines was not valid.
```

Limitations observed during the experiment:

- The COLMAP-based video approach reconstructed only short partial segments and could not reliably connect the full spatial path.
- ARKit camera pose was useful as a path and scale hint, but ARPlaneAnchor alone was not enough to create full wall outlines.
- Depth Anything V2 was useful for separating nearby furniture and foreground objects, but was not enough to directly confirm wall lines.
- RGB/depth edges and floor-line back-projection produced many candidates, but the noise level was too high to use them directly for floor-plan generation.
- YOLO was meaningful for detecting table, chair, and furniture candidates, but it did not solve the wall-structure estimation problem.

## Summary by Experiment

### Video / COLMAP

The general video-based SfM/COLMAP approach was unstable in finding a good initial pair and maintaining a continuous path.
On real store and indoor footage, it either failed to connect the full path into a single structure or reconstructed only partial sections.

### ARKit

ARKit was useful for:

```text
camera movement path
rough scale
some floor and vertical plane candidates
```

However, it was insufficient for:

```text
full wall-outline generation
room-structure estimation
stable separation of doors, entrances, furniture, and walls
```

### Depth

Depth Anything V2 represented front/back relationships better than RGB alone.
However, it reacted strongly to nearby furniture, shelves, and objects, and was not enough to directly create wall outlines.

Useful purposes:

```text
foreground obstacle/furniture masks
wall-candidate validation filters
capture-quality diagnostic support
```

Unsuitable purposes:

```text
direct wall-line generation
direct floor-plan coordinate extraction
```

### YOLO

YOLO was meaningful for generating object candidate layers such as tables, chairs, plants, refrigerators, and sinks.
However, using only the default COCO classes made it difficult to reliably distinguish store-structure elements such as doors, door frames, shelves, partitions, and counters.

## Reason for Stopping

The previous approach was close to the following assumption:

```text
capture data
-> extract edge/depth/plane candidates
-> project to top-down view
-> promote repeated candidates into walls
-> generate a draft floor plan
```

The results showed that this approach was closer to forcing 2D candidates into a floor-plan shape than actually understanding the space.
Repeatedly visible furniture, door frames, shelves, and floor lines survived alongside actual walls.

For that reason, continuing to refine this flow was judged unlikely to reach production quality.

## If This Work Is Resumed

Do not resume with the same approach.

Any resumed effort should move in one of the following directions:

```text
1. A separate product based on structure-estimation APIs such as RoomPlan, LiDAR, or Scene Reconstruction
2. Structured scanning where the app controls the capture route and angles instead of accepting free-form video
3. Redefining the goal as automated store reservation-screen creation, not accurate floor-plan generation
4. Building a spatial graph of seats, tables, zones, and paths before attempting wall-line extraction
```

Based on the current results, the most realistic product direction is:

```text
Redefine the product as an app for automatically creating store reservation screens,
not as an app for automatically generating floor plans.

Treat the floor plan as an optional supporting output, not as a mandatory intermediate artifact.
The core data should be reservable table/seat objects, zones, paths, and background imagery.
```

## Preservation Principle

This experiment should be preserved not as a failed implementation, but as a record of validating a flawed assumption.
Before starting the next experiment, read this document first and avoid repeating the same approach.
