# Store Space Scan Feasibility Study

> This feasibility study was derived from the UngsKitchen reservation-map project.

Original project: [UngsKitchen Reservation APP](https://github.com/dnddlek8275/UngsKitchen_Reservation_APP)

This repository preserves the experiment results and technical notes from an automated store-space scanning study.

The original goal was to help store or restaurant operators generate a reservation-ready aerial layout from photos, videos, or mobile scan data. The generated layout would eventually support reservable tables, seats, zones, and customer-facing reservation screens.

## Repository Metadata

Recommended GitHub description:

```text
A feasibility study derived from the UngsKitchen reservation-map project, exploring mobile video-based store layout reconstruction.
```

Recommended GitHub topics:

```text
ungskitchen
reservation-system
floorplan
indoor-mapping
computer-vision
arkit
depth-estimation
colmap
poc
feasibility-study
```

## Main Conclusion

The tested pipeline was not reliable enough to continue as a fully automated "video to floor plan" product.

Camera tracking, depth estimation, and object detection provided useful partial signals, but they did not consistently recover accurate wall outlines, room boundaries, or full store structure from unconstrained mobile footage.

## Preserved Documents

- `EXPERIMENT_SUMMARY.md`: Final experiment summary and decision.
- `uk_manger_app_docs/POC_ROADMAP.md`: Early proof-of-concept roadmap.
- `uk_manger_app_docs/TECH_FEASIBILITY_LOG.md`: Technical feasibility log.
- `uk_manger_app_docs/SPACE_SCAN_TECH_PLAN.md`: Product and technical plan for store scanning.
- `uk_scan_lab_docs/VALIDATION_PLAN.md`: AR scan validation plan.
- `uk_scan_lab_docs/IOS_AR_CAPTURE_POC.md`: iOS ARKit capture proof-of-concept notes.

## Repository Status

This repository is an experiment archive, not a production application.

Its value is to preserve what was tested, what failed, what showed partial promise, and what should be avoided if this product direction is revisited later.
