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

## 5. Parameters

The following tables summarize the main parameters used in the script.

---

### Detection Parameters

| **Parameter** | **Description** | **Value / Behavior** |
|---------------|-----------------|----------------------|
| `params.cornerRefinementMethod` | Method for refining detected marker corners. Improves accuracy at sub-pixel level. | Default: `cv2.aruco.CORNER_REFINE_SUBPIX` |
| `params.adaptiveThreshWinSizeMin` | Minimum window size for adaptive thresholding. | Default: `3` |
| `params.adaptiveThreshWinSizeMax` | Maximum window size for adaptive thresholding. | Default: `71` |
| `params.adaptiveThreshWinSizeStep` | Step size for adaptive threshold window. | Default: `10` |
| `params.minMarkerPerimeterRate` | Minimum relative marker perimeter (w.r.t image size). | Default: `0.01` |
| `params.maxMarkerPerimeterRate` | Maximum relative marker perimeter (w.r.t image size). | Default: `2.5` |
| `params.perspectiveRemoveIgnoredMarginPerCell` | Margin ignored during perspective removal. | Default: `0.4` |
| `params.perspectiveRemovePixelPerCell` | Pixel resolution used per cell during perspective removal. | Default: `4` |
| `MIN_SIZE` | Minimum side length of detected tag (pixels). | Default: `8` → Recalibrated after tag selection (depends on tag size & scaling) |
| `MAX_SIZE` | Maximum side length of detected tag (pixels). | Default: `3000` → Recalibrated after tag selection |
| `MIN_AREA` | Minimum contour area of detected tag. | Default: `max(10, frame_area × 0.0001)` → Recalibrated after tag selection |
| `CENTER_THR` | Spatial tolerance for replacing unstable IDs with stable ones. | Default: `2% of frame size` → Recalibrated after tag selection |
| `AREA_THR` | Allowed relative area difference for ID stability. | Default: `0.25 (25%)` → Adjusted to `0.20` if tag selection performed |
| `UPSCALE` | ROI upscaling factor (improves detection of small tags). | Default: `1.5` → If tag diameter <50px → `2.5`, <80px → `2.0`, else stays `1.5` |
| `angle_tol` | Tolerance for quadrilateral corner angles (validity check). | Default: `20°` → During main loop: `30°` |


### Tracking Parameters

| **Parameter** | **Description** | **Value / Behavior** |
|---------------|-----------------|----------------------|
| `required_detections` | Number of stable detections required before tag calibration. | Default: `20` if frame_count ≥ 300, else `3` |
| `MAX_TRACK_TIME` | Max tracking duration with optical flow (seconds). | Default: `3.0` |
| `MAX_TRACK_FRAMES` | Same as above, in frames (FPS × time). | Depends on video FPS (~90 for 30 FPS) |
| `TRACKER_EXTRA_FRAMES` | Extra frames for CSRT tracker fallback. | `2 × FPS` (~60 for 30 FPS) |
| `STILL_THRESH` | Pixel threshold: below this, marker is considered stationary. | Default: `2 px` |
| `STILL_FRAMES` | Number of frames below `STILL_THRESH` before removal. | `0.2 × FPS` |
| `MAX_NO_DETECT` | Max frames allowed without detection before dropping an ID. | `2 × FPS` (~60 frames for 30 FPS) |
| `lk_params` | Parameters for Lucas–Kanade optical flow. | Default: `winSize=(31,31)`, `maxLevel=4`, `criteria=(30 iterations, ε=0.01)` |

## 6. Testing

This script was tested using video recordings obtained from **indoor laboratory experiments**.  
In these experiments, bees equipped with ArUco tags were observed while interacting with **artificial flowers** under controlled conditions. The controlled setup allowed for consistent lighting, camera positioning, and predictable tag visibility, making it possible to validate the reliability of the detection and tracking pipeline.

During testing:

- The script successfully detected ArUco tags across varying lighting conditions and different bee positions within the frame.  
- ROI selection and tag calibration steps were found to significantly improve detection stability by reducing background noise and adapting thresholds to the actual tag size.  
- The combined tracking strategy (**Optical Flow → CSRT fallback → Optical Flow recovery**) ensured that tags were not lost even when they became partially occluded or temporarily left the detection region.  
- The generated log files (TXT) provided frame-by-frame tracking data, which was compared against manual annotations to confirm accuracy.  
- Processed output videos were visually inspected to evaluate how well IDs remained stable over time.
- **The best results were observed when the artificial flower was recorded as closely as possible**, since a closer view increased tag resolution, reduced false detections, and improved tracking stability.

Overall, the tests demonstrated that the algorithm can **robustly detect, track, and log bee movements** in indoor settings, providing both qualitative (annotated videos) and quantitative (TXT logs) results for later analysis.

## 7. Limitations & Future Work

While the script has shown reliable performance in controlled indoor experiments, there are several limitations to consider:

- **Indoor Testing Only**: The algorithm was tested exclusively with indoor recordings. Outdoor conditions such as direct sunlight, shadows, and background complexity may reduce detection accuracy.  
- **Camera Distance**: Accuracy decreases when the camera is placed too far from the artificial flower, since tags appear too small for stable detection.  
- **Lighting Sensitivity**: Although CLAHE and preprocessing improve robustness, very low light or overexposed conditions can still impact detection quality.  
- **Short-Term Occlusions**: The combined tracking pipeline (Optical Flow → CSRT → Optical Flow recovery) works well, but very long occlusions may still cause ID loss.  
- **Computational Cost**: High-resolution videos and large ROIs increase processing time on limited hardware.

### Future Work
To address these limitations, several improvements can be considered:
- Testing with **outdoor recordings** to evaluate robustness under natural light and complex backgrounds.  
- Integration of **GPU acceleration** (e.g., CUDA) to improve processing speed.  
- Support for **different ArUco dictionaries** and multi-marker calibration for larger-scale experiments.  
- More advanced **occlusion handling techniques**, such as Kalman filters or deep-learning-based trackers.  

These improvements would make the script more versatile and better suited for diverse experimental conditions.


