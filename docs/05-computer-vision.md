# RoShopSmart AI Retail Kiosk — Computer Vision Specification

**Document:** `05-computer-vision.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the Computer Vision (CV) subsystem of the **RoShopSmart AI Retail Kiosk**.

The CV subsystem provides the kiosk with visual perception capabilities:

- Person detection.
- Face detection.
- Face recognition.
- Product detection.
- Object tracking.
- Hand/gesture recognition.
- Product interaction analysis.
- Product verification.
- Suspicious transaction detection.

The objective is not simply to run pretrained models.

The objective is to demonstrate the complete Computer Vision engineering workflow:

```text
Problem
 ↓
Data
 ↓
Preprocessing
 ↓
Model
 ↓
Inference
 ↓
Post-processing
 ↓
Tracking
 ↓
Event Generation
 ↓
Business Integration
 ↓
Evaluation
 ↓
Optimization
```

---

# 2. Computer Vision Goals

The CV subsystem must satisfy the following goals:

1. Detect customers reliably.
2. Detect kiosk products.
3. Track products across video frames.
4. Recognize known customers when enabled.
5. Detect supported hand gestures.
6. Convert raw model predictions into structured events.
7. Support product verification.
8. Provide measurable accuracy and latency.
9. Support CPU inference for development.
10. Support GPU acceleration when available.
11. Allow models to be replaced without changing business logic.
12. Provide deterministic testing through mocked inference results.

---

# 3. Scope

## In Scope

```text
Person Detection
Face Detection
Face Recognition
Product Detection
Object Tracking
Gesture Recognition
Product Verification
CV Event Generation
CV Benchmarking
CV Testing
```

## Out of Scope for Initial Version

```text
Advanced behavior prediction
Full human pose estimation
Emotion recognition
Advanced anti-shoplifting behavior analysis
Multi-camera 3D reconstruction
Depth-camera fusion
Advanced anomaly detection
```

These capabilities may be considered in future versions.

---

# 4. CV Architecture

The CV subsystem follows a modular pipeline.

```text
                    CAMERA
                       │
                       ▼
                Frame Capture
                       │
                       ▼
                Frame Scheduler
                       │
                       ▼
                 Preprocessing
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
       Person        Product       Face
      Detection     Detection    Detection
          │            │            │
          │            ▼            ▼
          │         Tracking    Recognition
          │            │            │
          └────────────┼────────────┘
                       ▼
                Post-processing
                       │
                       ▼
                 Event Builder
                       │
                       ▼
              Application Layer
```

Gesture recognition is an independent pipeline:

```text
Camera
  ↓
Hand Detection
  ↓
Landmarks
  ↓
Gesture Recognition
  ↓
Gesture Event
```

---

# 5. Camera Input

The camera is the primary visual sensor.

Initial implementation should support:

```text
USB Webcam
Laptop Camera
IP Camera
```

The camera abstraction should hide the actual hardware.

Conceptual interface:

```python
class Camera:
    def start(self):
        ...

    def read(self):
        ...

    def stop(self):
        ...
```

Possible implementations:

```text
Camera
├── OpenCVCamera
├── VideoFileCamera
└── MockCamera
```

`VideoFileCamera` is particularly useful for reproducible testing.

---

# 6. Camera Configuration

Camera configuration should be externalized.

Example:

```yaml
camera:
  source: 0
  width: 1280
  height: 720
  fps: 30
```

Possible configuration:

```text
Resolution:
1280 × 720

Capture FPS:
30

Processing FPS:
5–15
```

The display FPS and AI processing FPS do not need to be identical.

---

# 7. Frame Scheduler

Running every AI model on every frame is inefficient.

Therefore:

```text
Camera FPS
    ↓
Frame Scheduler
    ↓
AI Processing FPS
```

Example:

```text
Camera:
30 FPS

Detection:
10 FPS

Face Recognition:
5 FPS

Gesture:
15 FPS
```

The scheduler should be configurable.

---

# 8. Frame Preprocessing

Typical preprocessing:

```text
Raw Frame
   ↓
Resize
   ↓
Color Conversion
   ↓
Normalization
   ↓
Model Input
```

Potential operations:

- Resolution scaling.
- RGB/BGR conversion.
- Normalization.
- Letterboxing.
- Image quality checks.

Preprocessing should be separated from model inference.

---

# 9. Frame Quality

The system should optionally detect poor-quality frames.

Possible conditions:

```text
Blur
Low brightness
Overexposure
Heavy occlusion
Invalid frame
```

Example:

```text
Frame Quality
      ↓
 ┌────┴─────┐
Good       Poor
 ↓           ↓
Inference   Skip
```

Skipping unusable frames can reduce unnecessary computation.

---

# 10. Person Detection

Person detection identifies customers in the kiosk area.

Pipeline:

```text
Camera
 ↓
Person Detector
 ↓
Bounding Boxes
 ↓
Confidence Filtering
 ↓
Person Events
```

Example:

```json
{
  "class": "person",
  "confidence": 0.97,
  "bbox": [100, 80, 420, 680]
}
```

---

# 11. Customer Detection

A customer should not be considered "present" simply because one frame contains a person.

Temporal confirmation should be used.

Example:

```text
Frame 1 → Person detected
Frame 2 → Person detected
Frame 3 → Person detected
             ↓
      CUSTOMER_DETECTED
```

Similarly:

```text
No person
for N consecutive frames
        ↓
CUSTOMER_LEFT
```

This reduces state-machine noise.

---

# 12. Customer Detection Event

Example event:

```json
{
  "event_type": "CUSTOMER_DETECTED",
  "timestamp": "2026-10-28T10:00:00Z",
  "confidence": 0.96,
  "bbox": {
    "x1": 100,
    "y1": 80,
    "x2": 420,
    "y2": 680
  }
}
```

The Kiosk State Machine consumes the event.

---

# 13. Product Detection

Product detection is the most important CV capability for automated checkout.

Pipeline:

```text
Frame
 ↓
Product Detector
 ↓
Bounding Boxes
 ↓
Class Prediction
 ↓
Confidence Filtering
 ↓
Product Detection
```

Example:

```text
Coca Cola
Confidence: 0.95

Water
Confidence: 0.92
```

---

# 14. Product Classes

Initial product classes should be deliberately small.

Example:

```text
P001 → Coca Cola
P002 → Water
P003 → Pepsi
P004 → Chips
P005 → Chocolate
```

The project should begin with approximately **5–10 products** rather than attempting hundreds of classes.

The goal of the first version is engineering validation.

---

# 15. Product Identity

The CV subsystem should return a stable product identifier.

Example:

```text
Detection
 ↓
class_name = "coca_cola"
 ↓
Product Mapping
 ↓
product_id = "P001"
```

The Product Service remains the authoritative source for:

```text
Name
SKU
Price
Inventory
Description
Category
```

CV only determines:

> "What object appears to be present?"

---

# 16. Product Detection Interface

Conceptual interface:

```python
class ProductDetector:

    def detect(self, frame) -> list:
        ...
```

Output:

```python
Detection(
    class_name="water",
    confidence=0.94,
    bbox=BoundingBox(...)
)
```

The application must not depend on YOLO-specific output structures.

---

# 17. YOLO Integration

Initial product detector:

```text
YOLO
```

Architecture:

```text
ProductDetector
       │
       ▼
YOLOProductDetector
       │
       ▼
YOLO Runtime
       │
       ▼
Trained Model
```

Possible future implementations:

```text
YOLO
RT-DETR
TensorRT model
ONNX model
Custom detector
```

The interface should remain stable.

---

# 18. Product Dataset

A dedicated dataset should be created for the kiosk products.

Suggested structure:

```text
datasets/
└── products/
    ├── raw/
    ├── annotations/
    ├── train/
    ├── val/
    └── test/
```

Dataset split:

```text
Train
Validation
Test
```

A typical initial split:

```text
70% Train
20% Validation
10% Test
```

The exact split may change depending on dataset size.

---

# 19. Dataset Collection

Images should represent realistic kiosk conditions.

Collect variations in:

```text
Camera angle
Distance
Lighting
Background
Rotation
Occlusion
Product orientation
Multiple products
Partial visibility
Hand interaction
```

Example:

```text
Product on shelf
Product in hand
Product partially hidden
Product under weak lighting
Product close to camera
Product far from camera
```

---

# 20. Dataset Quality

Before training, inspect:

```text
Duplicate images
Incorrect labels
Missing labels
Incorrect bounding boxes
Class imbalance
Low-quality images
Data leakage
```

A high-quality dataset is often more important than simply increasing model complexity.

---

# 21. Annotation Format

The initial detector may use YOLO annotation format.

Example:

```text
class_id center_x center_y width height
```

Example:

```text
0 0.512 0.443 0.231 0.582
```

The repository should maintain a mapping:

```text
0 → coca_cola
1 → water
2 → pepsi
3 → chips
4 → chocolate
```

---

# 22. Data Leakage Prevention

The same physical scene should not accidentally appear in both train and test sets.

For example:

```text
Same video
 ├── Frame 001 → train
 ├── Frame 002 → train
 └── Frame 003 → test
```

is undesirable.

Instead, split at the source-video or scene level where possible.

---

# 23. Data Augmentation

Potential augmentations:

```text
Horizontal Flip
Rotation
Scale
Crop
Brightness
Contrast
Blur
Noise
```

Augmentation should simulate realistic kiosk conditions.

Do not use transformations that create unrealistic product appearances.

---

# 24. Detection Threshold

The detector should expose a configurable confidence threshold.

Example:

```yaml
product_detection:
  confidence_threshold: 0.5
```

The threshold should be tuned using validation data.

A lower threshold may improve recall but increase false positives.

A higher threshold may reduce false positives but miss products.

---

# 25. Non-Maximum Suppression

Object detectors may produce overlapping predictions.

NMS is used to reduce duplicate boxes.

Conceptually:

```text
Multiple Boxes
      ↓
Confidence Ranking
      ↓
IoU Comparison
      ↓
Remove Duplicates
      ↓
Final Detections
```

The detector implementation should expose relevant NMS configuration when supported.

---

# 26. Object Tracking

Tracking provides temporal identity.

```text
Detection
   ↓
Tracker
   ↓
Track ID
```

Example:

```text
Frame 1:
Water → Track 01

Frame 2:
Water → Track 01

Frame 3:
Water → Track 01
```

The system can therefore distinguish between:

```text
One product seen across 10 frames
```

and:

```text
10 different products
```

---

# 27. Tracking Requirements

A track should contain:

```text
track_id
product_id
class_name
bounding_box
confidence
first_seen
last_seen
frame_count
status
```

Example:

```json
{
  "track_id": 17,
  "product_id": "P001",
  "confidence": 0.93,
  "frame_count": 12,
  "status": "ACTIVE"
}
```

---

# 28. Track Lifecycle

```text
NEW
 ↓
ACTIVE
 ↓
TEMPORARILY_LOST
 ↓
REACQUIRED
 ↓
REMOVED
```

This allows temporary occlusion.

Example:

```text
Product visible
 ↓
Hand covers product
 ↓
Tracker temporarily loses object
 ↓
Product reappears
 ↓
Same track recovered
```

---

# 29. Stable Detection

A product should not immediately become a verified product based on one detection.

Example rule:

```text
Product detected
      ↓
Collect N frames
      ↓
Confidence aggregation
      ↓
Stable detection
```

Possible aggregation:

```text
Mean confidence
Median confidence
Majority class
Track duration
```

The exact algorithm should be benchmarked.

---

# 30. Temporal Smoothing

Example:

```text
Frame 1 → Water 0.62
Frame 2 → Water 0.84
Frame 3 → Water 0.89
Frame 4 → Water 0.93
```

Instead of:

```text
Detected = true
```

on frame 1, the system can wait until:

```text
StableDetection = true
```

This improves robustness.

---

# 31. Face Detection

Face detection pipeline:

```text
Camera
 ↓
Face Detector
 ↓
Face Bounding Box
 ↓
Face Quality
 ↓
Face Recognition
```

Face detection should be independent from recognition.

---

# 32. Face Recognition

Recognition pipeline:

```text
Face
 ↓
Alignment
 ↓
Embedding
 ↓
Similarity Search
 ↓
Threshold
 ↓
Identity
```

Example:

```text
Embedding
      ↓
Database / Vector Store
      ↓
Nearest Candidate
      ↓
Similarity = 0.91
      ↓
Threshold = 0.85
      ↓
MATCH
```

The threshold must be evaluated experimentally.

---

# 33. Face Recognition Output

Example:

```json
{
  "identity": "customer_123",
  "confidence": 0.91,
  "match": true
}
```

For unknown users:

```json
{
  "identity": null,
  "confidence": 0.41,
  "match": false
}
```

---

# 34. Face Recognition Failure Modes

Potential failures:

```text
Poor lighting
Face angle
Occlusion
Multiple people
Low resolution
False match
False rejection
```

The system should return uncertainty rather than forcing an identity.

---

# 35. Face Recognition Optionality

The kiosk must remain functional if face recognition is disabled.

Possible configuration:

```yaml
face:
  enabled: false
```

The kiosk can then use:

```text
Anonymous Customer Session
```

instead of:

```text
Recognized Customer Session
```

This is important for privacy and system resilience.

---

# 36. Gesture Recognition

Gesture pipeline:

```text
Camera
 ↓
Hand Detection
 ↓
Landmark Extraction
 ↓
Gesture Classification
 ↓
Temporal Validation
 ↓
Gesture Event
```

Possible gestures:

```text
THUMBS_UP
THUMBS_DOWN
OPEN_HAND
POINT_LEFT
POINT_RIGHT
CONFIRM
CANCEL
```

---

# 37. Gesture Stability

A gesture should be observed over multiple frames.

Example:

```text
Frame 1 → THUMBS_UP
Frame 2 → THUMBS_UP
Frame 3 → THUMBS_UP
       ↓
CONFIRM
```

This prevents accidental gestures.

---

# 38. Gesture Event

Example:

```json
{
  "event_type": "GESTURE_DETECTED",
  "gesture": "CONFIRM",
  "confidence": 0.94
}
```

The State Machine can then decide whether the gesture is valid in the current state.

For example:

```text
SHOPPING + CONFIRM
→ valid

IDLE + CONFIRM
→ ignored
```

---

# 39. CV Event Model

All CV modules should produce normalized events.

Base event:

```json
{
  "event_id": "uuid",
  "event_type": "PRODUCT_DETECTED",
  "timestamp": "2026-10-28T10:00:00Z",
  "kiosk_id": "KIOSK-001",
  "session_id": "SESSION-001",
  "payload": {}
}
```

Possible event types:

```text
PERSON_DETECTED
CUSTOMER_DETECTED
CUSTOMER_LEFT
FACE_DETECTED
FACE_RECOGNIZED
PRODUCT_DETECTED
PRODUCT_STABLE
PRODUCT_REMOVED
GESTURE_DETECTED
CV_ERROR
```

---

# 40. Product Interaction Detection

The system should eventually identify interactions such as:

```text
Product Appeared
Product Picked
Product Held
Product Moved
Product Returned
Product Removed
```

Initial implementation should avoid overcomplicating this.

A practical first version can use:

```text
Product detection
+
Tracking
+
Spatial/temporal rules
```

---

# 41. Product Verification

Product verification compares:

```text
EXPECTED
vs
DETECTED
```

Example:

```text
Expected:
Coca Cola × 1
Water × 1

Detected:
Coca Cola × 1
Water × 1

Result:
MATCH
```

---

# 42. Product Mismatch

Example:

```text
Expected:
Coca Cola × 1

Detected:
Coca Cola × 2
```

Result:

```text
MISMATCH
```

Another:

```text
Expected:
Water × 1

Detected:
Water × 0
```

Result:

```text
MISMATCH
```

---

# 43. Verification Result

Normalized result:

```json
{
  "status": "MISMATCH",
  "expected": [
    {
      "product_id": "P001",
      "quantity": 1
    }
  ],
  "detected": [
    {
      "product_id": "P001",
      "quantity": 2
    }
  ],
  "confidence": 0.94
}
```

Possible statuses:

```text
MATCH
MISMATCH
UNCERTAIN
ERROR
```

---

# 44. Verification State

Verification should be temporal.

```text
PRODUCT_VERIFY
      ↓
Collect detections
      ↓
Track objects
      ↓
Aggregate results
      ↓
Compare expected/detected
      ↓
 ┌────┼───────────┐
 ▼    ▼           ▼
MATCH MISMATCH  UNCERTAIN
```

`UNCERTAIN` should not automatically become `SUSPICIOUS_TRANSACTION`.

---

# 45. Suspicious Transaction

A transaction becomes suspicious when evidence exceeds a defined rule threshold.

Initial examples:

```text
Quantity mismatch
Unknown product
Repeated inconsistent detection
Unexpected product
```

Example:

```text
Expected:
Water × 1

Detected:
Water × 1
Unknown Object × 1

→ SUSPICIOUS_TRANSACTION
```

---

# 46. Rule-Based First Version

The first version should use deterministic rules.

Example:

```python
if detected_quantity > expected_quantity:
    status = "MISMATCH"
```

This provides:

- Explainability.
- Easy testing.
- Deterministic behavior.
- Easy debugging.

Machine-learning-based anomaly detection can be introduced later.

---

# 47. CV Decision Boundary

The CV subsystem should not directly execute checkout.

Correct architecture:

```text
CV
 ↓
Detection
 ↓
Verification Result
 ↓
Verification Service
 ↓
State Machine
 ↓
Checkout
```

Incorrect architecture:

```text
YOLO
 ↓
checkout()
```

This separation is critical.

---

# 48. Multi-Person Scenario

The kiosk should consider multiple people.

Example:

```text
Camera
 ↓
Person Detection
 ↓
Person Count = 2
```

Possible behavior:

```text
Primary customer
Secondary person
```

The initial MVP may restrict operation to one active customer.

Configuration:

```yaml
customer:
  max_active_customers: 1
```

If more people are detected:

```text
MULTIPLE_CUSTOMERS_DETECTED
```

can be generated.

---

# 49. Occlusion Handling

Products may be temporarily hidden.

Example:

```text
Frame 1 → Product visible
Frame 2 → Hand occludes product
Frame 3 → Hand occludes product
Frame 4 → Product visible
```

The tracker should avoid immediately treating the product as removed.

Use:

```text
Lost Track Timeout
```

before deleting the track.

---

# 50. Lighting Robustness

The test dataset should include:

```text
Bright
Normal
Dim
Backlit
Artificial lighting
```

The system should be evaluated separately under these conditions.

Example benchmark:

| Condition | Precision | Recall | Latency |
|---|---:|---:|---:|
| Normal | TBD | TBD | TBD |
| Low Light | TBD | TBD | TBD |
| Bright | TBD | TBD | TBD |

---

# 51. Camera Position

Camera placement directly affects CV performance.

Recommended initial setup:

```text
Customer Camera
        ↓
       Kiosk
        ↓
   Product Area
```

The project should document:

```text
Camera height
Camera angle
Field of View
Detection region
```

---

# 52. Region of Interest

Instead of processing the entire image for every task, define regions.

Example:

```text
┌──────────────────────────────┐
│ Customer Area                │
│                              │
├──────────────────────────────┤
│ Product Interaction Area     │
│                              │
├──────────────────────────────┤
│ Checkout Area                │
└──────────────────────────────┘
```

Product detection can focus on:

```text
Product Interaction Area
```

This can reduce false positives and computation.

---

# 53. CV Pipeline Configuration

Example:

```yaml
vision:
  enabled: true

  detection:
    model: product_detector_v1
    confidence_threshold: 0.5

  tracking:
    enabled: true
    max_lost_frames: 15

  face:
    enabled: false

  gesture:
    enabled: true

  processing:
    target_fps: 10
```

---

# 54. Error Handling

CV errors should be explicit.

Possible errors:

```text
CAMERA_UNAVAILABLE
INVALID_FRAME
MODEL_LOAD_FAILED
INFERENCE_FAILED
TRACKER_FAILED
FACE_MODEL_UNAVAILABLE
GESTURE_MODEL_UNAVAILABLE
```

The kiosk should degrade gracefully.

---

# 55. Camera Failure

If camera fails:

```text
Camera
 ↓
Failure
 ↓
CV_ERROR
 ↓
Application
 ↓
Fallback UI
```

Example:

```text
"Camera unavailable. Please select products manually."
```

The kiosk should not crash.

---

# 56. Model Loading

Models should be loaded once where possible.

Bad:

```text
For every frame:
    load_model()
    inference()
```

Correct:

```text
Application Startup
      ↓
Load Model
      ↓
Keep Runtime
      ↓
Process Frames
```

This significantly reduces unnecessary overhead.

---

# 57. Inference Runtime

The inference layer should abstract execution.

```text
InferenceEngine
│
├── CPUInferenceEngine
├── GPUInferenceEngine
└── MockInferenceEngine
```

Potential future runtime:

```text
ONNX Runtime
TensorRT
OpenVINO
```

The choice should be benchmark-driven.

---

# 58. Batch vs Streaming Inference

The kiosk is primarily a streaming application.

Therefore:

```text
Frame
 ↓
Inference
 ↓
Result
```

is the default.

Batch inference may be useful for:

```text
Offline evaluation
Dataset benchmarking
Model testing
```

---

# 59. CV Performance Metrics

Minimum metrics:

```text
Inference latency
End-to-end latency
FPS
CPU usage
GPU usage
RAM
VRAM
Precision
Recall
mAP
Tracking stability
```

---

# 60. End-to-End Latency

Important metric:

```text
Camera Frame
     ↓
Inference
     ↓
Post-processing
     ↓
Event
     ↓
Application
```

Measure:

```text
T_end_to_end
```

not only model inference time.

Example:

```text
Capture:       5 ms
Preprocess:    3 ms
Inference:    25 ms
Postprocess:   4 ms
Event:         2 ms
--------------------
Total:        39 ms
```

Numbers above are illustrative only.

---

# 61. Accuracy Metrics

For object detection:

```text
Precision
Recall
mAP@50
mAP@50:95
```

For face recognition:

```text
False Acceptance Rate
False Rejection Rate
Recognition Accuracy
```

For gesture recognition:

```text
Accuracy
Precision
Recall
F1-score
```

---

# 62. Tracking Metrics

Possible metrics:

```text
Track continuity
ID switches
Track loss rate
Track recovery rate
Average track duration
```

The initial MVP can focus on:

```text
Track stability
Track loss
ID consistency
```

---

# 63. Benchmark Protocol

Every benchmark should specify:

```text
Model
Model version
Dataset
Image resolution
Hardware
Runtime
Batch size
Confidence threshold
Input FPS
```

Example:

```text
Model:
product_detector_v1

Resolution:
640 × 640

Hardware:
CPU

Runtime:
PyTorch

Batch:
1

Dataset:
products_test_v1
```

---

# 64. Benchmark Table

The repository should eventually contain:

| Experiment | Model | Hardware | mAP50 | Latency | FPS | Memory |
|---|---|---|---:|---:|---:|---:|
| EXP-001 | Detector v1 | CPU | TBD | TBD | TBD | TBD |
| EXP-002 | Detector v1 | GPU | TBD | TBD | TBD | TBD |
| EXP-003 | Detector v2 | GPU | TBD | TBD | TBD | TBD |

---

# 65. CV Testing Strategy

Testing should be divided into:

```text
Unit Tests
Integration Tests
Dataset Tests
Model Evaluation
End-to-End Tests
```

---

# 66. Unit Tests

Examples:

```text
BoundingBox
Detection
Track
ConfidenceFilter
TemporalSmoothing
ProductMapping
VerificationLogic
```

Example:

```python
def test_product_mapping():
    assert map_class("water") == "P002"
```

---

# 67. Verification Tests

Example:

```text
Expected:
Water × 1

Detected:
Water × 1

Expected:
MATCH
```

Test mismatch:

```text
Expected:
Water × 1

Detected:
Water × 2

Expected:
MISMATCH
```

Test uncertainty:

```text
Low confidence
+
insufficient frames

→ UNCERTAIN
```

---

# 68. Mock Detection Tests

Use deterministic detections:

```json
{
  "product_id": "P001",
  "confidence": 0.95
}
```

Then verify application behavior.

This allows the entire checkout workflow to be tested without running YOLO.

---

# 69. Dataset Regression Tests

When a model is updated:

```text
Model v1
 ↓
Benchmark
 ↓
Model v2
 ↓
Benchmark
 ↓
Compare
```

Do not replace the model simply because v2 is newer.

The new model should demonstrate an acceptable trade-off between:

```text
Accuracy
Latency
Resource Usage
Robustness
```

---

# 70. Model Promotion Criteria

A model can be promoted when:

```text
Accuracy ≥ target
AND
Latency ≤ target
AND
No critical regression
AND
Integration tests pass
```

Example:

```text
mAP50 >= target
Latency <= target
False positive rate <= target
```

Exact values should be established experimentally.

---

# 71. CV Observability

Every inference may record:

```text
timestamp
model
model_version
processing_time
confidence
object_count
device
```

Example:

```json
{
  "model": "product_detector",
  "version": "v1",
  "latency_ms": 31,
  "object_count": 3
}
```

---

# 72. CV Logging

Logs should be structured.

Example:

```text
INFO PRODUCT_DETECTION
model=product_detector_v1
objects=3
latency_ms=32
```

Avoid logging raw image data unnecessarily.

---

# 73. Privacy

The CV subsystem should follow data minimization.

Do not store:

```text
Every camera frame
Every face image
Every raw video stream
```

unless explicitly required for development or evaluation.

For debugging, use controlled datasets and temporary recordings.

---

# 74. Security

Potential security concerns:

```text
Camera access
Face embeddings
Recorded images
Model files
Inference APIs
```

Controls:

```text
Access control
Secure storage
Environment configuration
Input validation
Logging
Retention policies
```

---

# 75. Initial MVP

The first CV implementation should intentionally be small.

### MVP CV

```text
1. Camera
2. Person Detection
3. Product Detection
4. Object Tracking
5. Product Verification
```

Then:

```text
6. Gesture Recognition
7. Face Detection
8. Face Recognition
```

This ordering prioritizes the core retail workflow.

---

# 76. Recommended Implementation Order

```text
Step 1
Camera abstraction
        ↓
Step 2
Frame pipeline
        ↓
Step 3
YOLO person detection
        ↓
Step 4
YOLO product detection
        ↓
Step 5
Product dataset
        ↓
Step 6
Product training
        ↓
Step 7
Object tracking
        ↓
Step 8
Temporal smoothing
        ↓
Step 9
Product verification
        ↓
Step 10
CV events
        ↓
Step 11
Gesture recognition
        ↓
Step 12
Face recognition
        ↓
Step 13
Benchmark
        ↓
Step 14
Optimization
```

---

# 77. Future Computer Vision Extensions

Potential future features:

```text
Pose Estimation
Depth Camera
Multi-camera Tracking
Shelf Monitoring
Advanced Anomaly Detection
Visual Search
OCR
Barcode Recognition
Object Re-identification
```

These should only be implemented if they provide measurable product value.

---

# 78. Engineering Principles

The CV implementation should follow:

```text
Modularity
+
Replaceability
+
Testability
+
Measurability
+
Performance
+
Privacy
```

Avoid:

```text
Model-specific business logic
Hardcoded thresholds
Hardcoded camera paths
Unmeasured optimization
Uncontrolled data storage
```

---

# 79. Definition of Done

The Computer Vision phase is considered complete when:

```text
☐ Camera abstraction implemented
☐ Frame pipeline implemented
☐ Person detection implemented
☐ Product detection implemented
☐ Product dataset created
☐ Dataset split documented
☐ Product annotations validated
☐ Object tracking implemented
☐ Temporal smoothing implemented
☐ Product verification implemented
☐ CV events implemented
☐ Gesture recognition implemented
☐ Face detection implemented
☐ Face recognition implemented or explicitly deferred
☐ Unit tests implemented
☐ Integration tests implemented
☐ Model evaluation implemented
☐ Benchmark recorded
☐ Performance bottlenecks identified
☐ Documentation updated
☐ Demo evidence recorded
☐ Git commit created
```

---

# 80. Final Computer Vision Pipeline

The complete CV workflow is:

```text
                         CAMERA
                            │
                            ▼
                     FRAME CAPTURE
                            │
                            ▼
                    FRAME SCHEDULER
                            │
                            ▼
                     PREPROCESSING
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          PERSON         PRODUCT          FACE
        DETECTION       DETECTION       DETECTION
             │              │              │
             │              ▼              ▼
             │           TRACKING     RECOGNITION
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                    TEMPORAL ANALYSIS
                            │
                            ▼
                     POST-PROCESSING
                            │
                            ▼
                       CV EVENTS
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        Customer        Product         Gesture
        Events          Events          Events
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                   APPLICATION LAYER
                            │
                            ▼
                  VERIFICATION SERVICE
                            │
                            ▼
                      STATE MACHINE
                            │
                            ▼
                       CHECKOUT
```

The central engineering principle is:

```text
RAW SENSOR DATA
      ↓
PERCEPTION
      ↓
STRUCTURED VISUAL INFORMATION
      ↓
VERIFICATION
      ↓
APPLICATION DECISION
```

Computer Vision does not own the transaction.

It provides **measurable perception signals** that the application layer can validate, combine, and use to operate the smart retail workflow.