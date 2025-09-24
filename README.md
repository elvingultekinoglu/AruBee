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

## 4. Algorithm Overview

The script follows a structured pipeline to detect and track ArUco tags:

1. **Input Handling**  
   - User specifies video/log paths or uses predefined defaults.  
   - Video properties (resolution, FPS, frame count) are read for later calibration.  

2. **ROI Selection**  
   - A circular ROI is selected on the first frame (optional).  
   - Restricts computation to a focused area, improving accuracy and efficiency.  

3. **Tag Calibration**  
   - The script waits for a stable tag detection (e.g., 20 consecutive frames in long videos).  
   - The user then selects the tag with a rectangle.  
   - Tag size is used to dynamically adjust thresholds (min/max size, area, and upscale factor).  
   - This ensures the detector is tuned to the actual marker size in the video.  

4. **Frame Processing & Detection**
- Each frame undergoes preprocessing: grayscale conversion, CLAHE (contrast enhancement), Gaussian blur, and sharpening.  
- ROI is upscaled to improve detection of small tags.  
- ArUco markers are detected and validated through:  
  - **Size constraints** (too small/large markers are discarded).  
  - **Area thresholds** (removes noisy detections).  
  - **Geometric checks** (angles between corners must be ~90° to filter distorted tags).  
- For each detected ID, only the *best candidate* (largest, most reliable detection) is kept per frame.  

5. **Tracking & ID Consistency**
- If a marker is detected, it is stored with its corners, center, and feature points.  
- If detection fails in the next frames, the algorithm switches to **tracking modes**:
  - **Optical Flow (Lucas–Kanade)** estimates marker corner movement across frames.  
  - If optical flow becomes unreliable, a **CSRT tracker** is initialized as a fallback.  
- The tracker is not final — if new good feature points are found, the script **switches back from CSRT to optical flow** for smoother and more accurate tracking.  
- ID stability is preserved by checking:  
  - **Center proximity** (IDs close to each other are considered the same).  
  - **Area similarity** (avoids false ID swaps when tags overlap).  
- If a marker remains undetected for too long (e.g., >1 second), its track is dropped.  
- Stationary markers are also removed if their movement stays below a pixel threshold for a set duration.  

6. **Logging & Output**
- For each frame, the following are saved:  
  - Frame number and timestamp  
  - Tag ID  
  - Corner coordinates (x0,y0 … x3,y3)  
- A processed video is generated with polygons, center dots, and IDs drawn on tags.  
- A TXT log file stores all tracking data in tabular format for later analysis.  

7. **Termination**
- Once the video ends, resources are released.  
- The script prints the locations of the saved output files (processed video and log).

