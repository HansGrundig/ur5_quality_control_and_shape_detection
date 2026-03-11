# Automated Object Detection & Robotic Pick-and-Place System

**Course:** Entwicklung Automatisierter und Autonomer Systeme (5. Semester)

## Description

This project implements an automated pick-and-place pipeline using a Raspberry Pi camera and a UR5 robot arm. A YOLO OBB (Oriented Bounding Box) model detects objects (bags) in real time, computes their 2D pose (position + rotation), and sends the coordinates to the UR5 via TCP socket so the robot can autonomously pick them up.

### Pipeline Overview

```
Pi Camera → Undistort → YOLO OBB Inference → Homography Transform → TCP Socket → UR5 Robot
```

1. Capture a frame with the Raspberry Pi camera (Picamera2)
2. Undistort the image using pre-calibrated camera intrinsics
3. Detect objects using a custom-trained YOLO OBB model (single class: `bag`)
4. Extract center coordinates and rotation angle from the bounding box
5. Transform pixel coordinates to robot coordinates via a homography matrix
6. Send pose data (x, y, shape type, angle) to the UR5 over a TCP socket connection

---

## Project Structure

```
Projekt/
├── src/
│   ├── rbp_script/               # Main runtime scripts (Raspberry Pi)
│   │   ├── live_shape_detection_rbp.py   # Main detection + robot communication
│   │   ├── prediction_shape_detection.py # Streamlined YOLO-only detection
│   │   ├── parameter_setup.py            # Interactive calibration tool (trackbars)
│   │   ├── parameters.json               # Canny edge detection thresholds
│   │   ├── camera_matrix.txt             # Camera intrinsics
│   │   ├── centerpoints.json             # Homography reference points
│   └── ur_script/                # UR5 robot program files (.urp / .script)
├── runs/                         # YOLO training run outputs & weights
├── models/
    ├── best.pt                  # Trained YOLO OBB model
├── README.md                    # Project documentation
```

---

## Required Packages

Install dependencies on the Raspberry Pi:

```bash
pip install ultralytics opencv-python numpy picamera2 
```

| Package | Purpose |
|---|---|
| `ultralytics` | YOLO OBB model inference and training |
| `opencv-python` | Image processing, Canny edge detection, homography |
| `numpy` | Numerical operations and matrix math |
| `picamera2` | Raspberry Pi camera interface |
| `socket` | TCP communication with the UR5 robot (stdlib) |
| `pickle` | Serialize/deserialize homography matrix and calibration data (stdlib) |
| `json` | Load/save parameters and configuration files (stdlib) |

---

## Usage

### 1. Camera Calibration (one-time setup)
Run `parameter_setup.py` on the Raspberry Pi to tune Canny edge detection thresholds interactively. Results are saved to `parameters.json`.

### 2. Live Detection & Robot Control
```bash
python src/rbp_script/live_shape_detection_rbp.py
```

After calculating disstortion matrix and homography matrix, start the  script for live detection. Detected bag poses are automatically sent to the UR5 robot.
