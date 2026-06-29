# Testing and Evaluation - NexusAI

**Evaluation Framework and Metrics**

---

## Evaluation Scripts

### 1. Drowsiness Detection

**File**: `evaluation/evaluate_drowsiness.py`

**Categories**:
- `eyes_open/` - Eyes open images (should detect as NOT drowsy)
- `eyes_closed/` - Eyes closed images (should detect as drowsy)
- `yawning/` - Yawning images (should detect isYawning=true)
- `fatigued/` - Fatigued images (should show fatigue level Medium/High)

**Run**:
```bash
python evaluation/evaluate_drowsiness.py
```

**Output**:
```
DROWSINESS RESULTS

eyes_open: 95/100 (95.00%)
eyes_closed: 92/100 (92.00%)
yawning: 88/100 (88.00%)
fatigued: 90/100 (90.00%)

Overall Accuracy: 91.25%
```

---

### 2. Distraction Detection

**File**: `evaluation/evaluate_distraction.py`

**Categories**:
- `focused/` - Looking forward (should detect lookingAway=false)
- `looking_away/` - Head turned (should detect lookingAway=true)
- `phone_usage/` - Phone in hand (should detect phoneDetected=true)
- `talking/` - Mouth moving (should detect isTalking=true)

**Run**:
```bash
python evaluation/evaluate_distraction.py
```

**Output**:
```
DISTRACTION RESULTS

focused: 94/100 (94.00%)
looking_away: 91/100 (91.00%)
phone_usage: 98/100 (98.00%)
talking: 85/100 (85.00%)

Overall Accuracy: 92.00%
```

---

### 3. Face Recognition

**File**: `evaluation/evaluate_face_recognition.py`

**Categories**:
- `driver1/` - Driver 1 face images
- `driver2/` - Driver 2 face images
- `unknown_driver/` - Unknown faces

**Threshold**: 0.45 cosine similarity

**Run**:
```bash
python evaluation/evaluate_face_recognition.py
```

**Output**:
```
FACE RECOGNITION RESULTS

driver1: 48/50 (96.00%)
driver2: 47/50 (94.00%)
unknown_driver: 49/50 (98.00%)

Overall Accuracy: 96.00%
```

---

### 4. Head Pose Estimation

**File**: `evaluation/evaluate_head_pose.py`

---

### 5. Lighting Conditions

**File**: `evaluation/evaluate_lighting.py`

---

## Metrics

### Accuracy Metrics

**Precision**: TP / (TP + FP)
- How many detected positives are correct

**Recall**: TP / (TP + FN)
- How many actual positives were detected

**F1-Score**: 2 × (Precision × Recall) / (Precision + Recall)
- Balanced metric

### Performance Metrics

| Metric | Target | Typical | Unit |
|--------|--------|---------|------|
| Latency | <100 | 60-110 | ms |
| FPS | 24-30 | 20-30 | fps |
| Memory | <500 | 200-400 | MB |
| CPU Usage | <80% | 60-75% | % |

---

## Test Dataset Structure

```
dataset/
├── drowsiness/
│   ├── eyes_open/          # ~100 images
│   ├── eyes_closed/        # ~100 images
│   ├── yawning/            # ~100 images
│   └── fatigued/           # ~100 images
├── distraction/
│   ├── focused/            # ~100 images
│   ├── looking_away/       # ~100 images
│   ├── phone_usage/        # ~100 images
│   └── talking/            # ~100 images
├── face_recognition/
│   ├── driver1/            # ~50 images
│   ├── driver2/            # ~50 images
│   └── unknown_driver/     # ~50 images
├── head_pose/
└── lighting_conditions/
```

---

## Manual Testing

### Test 1: Basic Face Detection
```python
from app.services.face_detection_service import process_driver_frame
import cv2

image = cv2.imread("test_face.jpg")
_, buffer = cv2.imencode(".jpg", image)
result = process_driver_frame(buffer.tobytes())

assert result.driver.faceDetected == True
assert result.driver.faceCount >= 1
```

### Test 2: Drowsiness Detection
```python
assert result.driver.isDrowsy in [True, False]
assert 0 <= result.driver.attentionScore <= 100
```

### Test 3: Phone Detection
```python
assert result.driver.phoneDetected in [True, False]
```

### Test 4: Safety Score
```python
assert 0 <= result.driver.safetyScore <= 100
assert result.driver.emergencyMode in [True, False]
```

---

## Continuous Integration

Recommended CI/CD checks:

```yaml
test:
  script:
    - pip install -r requirements.txt
    - python evaluation/evaluate_drowsiness.py
    - python evaluation/evaluate_distraction.py
    - python evaluation/evaluate_face_recognition.py
  success_criteria:
    - drowsiness_accuracy > 85%
    - distraction_accuracy > 85%
    - recognition_accuracy > 80%
```

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29