# AI Pipeline - NexusAI

**Detailed AI Model Architecture and Processing Pipeline**

---

## AI Pipeline Overview

```
Input Frame (JPEG)
    ↓
┌─────────────────────────────────┐
│ Stage 1: Face Detection         │
│ OpenCV Cascade Classifier       │
│ Output: Bounding boxes          │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ Stage 2: Face Landmarks         │
│ MediaPipe Face Mesh (468 pts)   │
│ Output: Facial landmarks        │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ Stage 3: Phone Detection        │
│ YOLOv8 Nano Object Detection    │
│ Output: Cell phone detected?    │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ Stage 4: Behavioral Analysis    │
│ ├─ Drowsiness (eye distance)    │
│ ├─ Distraction (head pose)      │
│ ├─ Yawning (mouth opening)      │
│ └─ Talking (mouth movement)     │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ Stage 5: Metric Calculation     │
│ ├─ Safety Score (0-100)         │
│ ├─ Attention Score (0-100)      │
│ ├─ Fatigue Level                │
│ └─ Warning Counter              │
└────────────┬────────────────────┘
             ↓
         TelemetryResponse
```

---

## Drowsiness Detection Algorithm

### Eye Distance Metric
```
Eye Distance = Distance(upper_eyelid, lower_eyelid)

if eye_distance < 0.015:  # Threshold
    is_drowsy = True
    drowsy_duration += frame_time
else:
    is_drowsy = False
    drowsy_duration = 0
```

### Warning Escalation
```
Warning Counter Logic:

if drowsy_duration > 2s:   warning_counter = 1
if drowsy_duration > 4s:   warning_counter = 2
if drowsy_duration > 6s:   warning_counter = 3

if warning_counter >= 3:
    emergency_mode = True
    recommended_action = "Pull Over"
```

---

## Distraction Detection Algorithm

### Head Pose Analysis
```
left_distance = |nose.x - left_face.x|
right_distance = |right_face.x - nose.x|

if left_distance > right_distance + 0.015:
    head_direction = "Right"
    looking_away = True
elif right_distance > left_distance + 0.015:
    head_direction = "Left"
    looking_away = True
else:
    head_direction = "Center"
    looking_away = False
```

### Gaze Stability
```
gaze_history = 20-frame buffer of head_direction
gaze_stability = (count("Center") / len(gaze_history)) * 100

if gaze_stability < 50:
    unstable_attention_event()
```

---

## Safety Score Calculation

```
safety_score = 100

if is_drowsy:           safety_score -= 50
if is_yawning:          safety_score -= 20
if is_talking:          safety_score -= 10
if looking_away:        safety_score -= 25
if not face_detected:   safety_score -= 40
if phone_detected:      safety_score -= 35

safety_score = max(0, min(100, safety_score))
```

---

## Models Used

| Model | Purpose | Size | Latency |
|-------|---------|------|----------|
| OpenCV Cascade | Face detection | ~1MB | 20-30ms |
| MediaPipe Mesh | Face landmarks | ~1MB | 15-25ms |
| YOLOv8 Nano | Object detection | 6.5MB | 10-20ms |
| InsightFace | Face embeddings | ~130MB | 5-15ms |

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29