# AruBee
## 1. General Information

**Aru Bee** is a Python tool for analyzing the movements of honeybees that carry **ArUco tags** on **artificial flowers**. It processes video recordings, detects the **unique IDs** of bees arriving at and leaving the flower area, tracks their positions over time, and writes outputs as:
- a **processed video** with overlaid IDs and polygons, and
- a **tab-separated log file** (frame, time, ID, and the four corner coordinates).

The workflow is simple: you select a **region of interest (ROI)** on the first frame, the script **calibrates** detection thresholds from a stable tag snapshot, and then performs **ArUco detection + optical-flow/CSRT tracking** to maintain ID continuity even when detections are intermittent.

### Why ArUco Tags with Honeybees?
ArUco tags are **lightweight, robust, and inexpensive** visual fiducials that encode an **ID in a binary pattern**. For honeybee studies, they offer several advantages:
- **Reliable re-identification:** Each bee’s tag maps to a stable numeric ID, enabling long-term, per-individual analysis.
- **Strong under real conditions:** The binary pattern is tolerant to partial occlusions, scale changes, and varying illumination—common in lab/field recordings.
- **Off-the-shelf tooling:** Detection is supported natively by **OpenCV** (no custom training needed), which simplifies setup and improves reproducibility.
- **Geometry-aware tracking:** Corner coordinates provide orientation and shape cues, improving association across frames compared to color-only methods.

This project aims to support **behavioral experiments** by providing a reproducible pipeline for **visit detection**, **dwell-time estimation**, and **movement analysis** of tagged honeybees on artificial flowers.

## 2. Requirements

To run **AruBee**, you will need the following environment and dependencies:

- **Python**: 3.9 or higher  
- **NumPy**: numerical operations and array handling  
- **OpenCV (opencv-python)**: core computer vision functions  
- **OpenCV Contrib (opencv-contrib-python)**: provides the `aruco` module for tag detection  
- **collections (deque)**: standard library module for efficient history tracking  
- **os**: standard library module for file and path operations  

### Installation

You can install the required libraries with:

```bash
pip install numpy opencv-python opencv-contrib-python
