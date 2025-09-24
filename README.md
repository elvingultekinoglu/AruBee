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
```

## 3. Usage

To run the ArUco tag detection script:

```bash
python3 aruco_tag_detection.py
```
When executed, the script will guide you through several interactive steps:

**1. Video Path Selection**

At the start, you will be asked: 

```bash
Did you already set the paths inside the code? (yes/no):
```

* Type `yes` to use the predefined default paths in the script. 
* Type `no` if you want to manually enter: 
  - Path to the input video
  - Path to save the processed output video
  - Path to save the TXT log file

**2. ROI (Region of Interest) Selection**

* The first frame of the video will open in a window.
* Draw a circular ROI by clicking and dragging with your mouse. Start at the center of the circle you want to select, then expand it.
* Press **Enter** to confirm the ROI, or **Esc** to cancel and use the full frame.

**Tip:** Choose an ROI that closely covers the area where the bee and its ArUco tag are expected to move. This improves detection accuracy and reduces noise.

**3. Tag Calibration**

* The script waits until a stable tag detection is found (about 20 frames in long videos, fewer in short ones).
* Once detected, you will be asked to draw a rectangle around the tag.
* This step automatically adjusts detection thresholds and scaling for better tracking.
* If you skip this step, default safe thresholds are used.
**Tip:** Make sure the tag is clearly visible when you draw the rectangle. Avoid blurry or partially cut frames.

**4. Processing and Tracking** 

* The script processes the video frame by frame.
* It detects, tracks, and validates ArUco tags within the ROI.
* Optical flow and tracker fallback methods are used to maintain ID consistency even if a tag is temporarily lost.

**5. Outputs** 

The script generates two main outputs:
* **Processed video** with detected IDs drawn on each tag.
* **TXT log file** containing per-frame tag positions and metadata.
