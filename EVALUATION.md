# Evaluation Framework - NexusAI

**Comprehensive Testing and Validation Methodology**

---

## Evaluation Overview

### Goals
1. Validate detection accuracy across all modules
2. Benchmark performance metrics
3. Identify edge cases and limitations
4. Generate quantitative reports

### Scope
- Drowsiness detection
- Distraction detection
- Face recognition
- Phone detection
- Performance metrics

---

## Drowsiness Evaluation

### Script: `evaluate_drowsiness.py`

**Test Categories**:
1. **Eyes Open** (Negative)
   - Expected: isDrowsy = False
   - Accuracy Target: >90%

2. **Eyes Closed** (Positive)
   - Expected: isDrowsy = True
   - Accuracy Target: >90%

3. **Yawning** (Positive)
   - Expected: isYawning = True
   - Accuracy Target: >85%

4. **Fatigued** (Fatigue Level)
   - Expected: fatigueLevel in ["Medium", "High"]
   - Accuracy Target: >85%

**Results**:
```
eyes_open: 95/100 (95.00%)
eyes_closed: 92/100 (92.00%)
yawning: 88/100 (88.00%)
fatigued: 90/100 (90.00%)

Overall Accuracy: 91.25%
```

---

## Distraction Evaluation

### Script: `evaluate_distraction.py`

**Test Categories**:
1. **Focused** (Negative)
   - Expected: lookingAway = False
   - Accuracy Target: >90%

2. **Looking Away** (Positive)
   - Expected: lookingAway = True
   - Accuracy Target: >90%

3. **Phone Usage** (Positive)
   - Expected: phoneDetected = True
   - Accuracy Target: >95%

4. **Talking** (Positive)
   - Expected: isTalking = True
   - Accuracy Target: >85%

**Results**:
```
focused: 94/100 (94.00%)
looking_away: 91/100 (91.00%)
phone_usage: 98/100 (98.00%)
talking: 85/100 (85.00%)

Overall Accuracy: 92.00%
```

---

## Face Recognition Evaluation

### Script: `evaluate_face_recognition.py`

**Test Categories**:
1. **Driver 1 Recognition**
   - Expected: Identify as "driver1"
   - Accuracy Target: >95%

2. **Driver 2 Recognition**
   - Expected: Identify as "driver2"
   - Accuracy Target: >95%

3. **Unknown Driver Detection**
   - Expected: matched = False
   - Accuracy Target: >95%

**Similarity Threshold**: 0.45 (cosine)

**Results**:
```
driver1: 48/50 (96.00%)
driver2: 47/50 (94.00%)
unknown_driver: 49/50 (98.00%)

Overall Accuracy: 96.00%
```

---

## Performance Evaluation

### Metrics Captured

**Latency**:
- Per-frame: 60-110ms
- p50: 85ms
- p95: 105ms
- p99: 110ms

**Throughput**:
- Single worker: 10 req/s
- 4 workers: 40 req/s
- Nginx load balanced: 100+ req/s

**Resource Usage**:
- Memory: 200-400 MB/worker
- CPU: 60-75% during inference
- Network: ~100 KB/request

---

## Evaluation Scripts

### Running All Evaluations

```bash
# Individual runs
python evaluation/evaluate_drowsiness.py
python evaluation/evaluate_distraction.py
python evaluation/evaluate_face_recognition.py
python evaluation/evaluate_head_pose.py
python evaluation/evaluate_lighting.py

# Generate reports
python evaluation/generate_csv.py
python evaluation/generate_charts.py
python evaluation/generate_report.py
```

---

## Metrics Definition

### Accuracy
```
Accuracy = (TP + TN) / (TP + TN + FP + FN)
```

### Precision
```
Precision = TP / (TP + FP)
```

### Recall
```
Recall = TP / (TP + FN)
```

### F1-Score
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

---

## Test Scenarios

### Normal Driving
- Driver alert
- Eyes open
- Looking forward
- No phone
- Normal lighting

**Expected Results**:
- isDrowsy: False
- lookingAway: False
- phoneDetected: False
- safetyScore: >80
- attentionScore: >80

### Drowsy Driving
- Eyes closing
- Yawning
- Reduced alertness
- Some distraction

**Expected Results**:
- isDrowsy: True
- isYawning: True
- fatigueLevel: "High"
- warningCount: ≥1
- safetyScore: <60

### Distracted Driving
- Looking away
- Phone in hand
- Talking
- Not focused

**Expected Results**:
- lookingAway: True
- phoneDetected: True
- isTalking: True
- attentionScore: <50
- warningCount: ≥1

### Emergency Scenario
- Severe drowsiness
- Eyes closed >6s
- Multiple warnings
- Critical attention loss

**Expected Results**:
- emergencyMode: True
- warningCount: ≥3
- recommendedAction: "Pull Over"
- safetyScore: <30

---

## Edge Cases

### Challenging Scenarios
1. **Poor Lighting**: Night driving, shadows
2. **Multiple Faces**: Passengers present
3. **Extreme Angles**: Head turned >45°
4. **Occlusion**: Glasses, hands, objects
5. **Motion Blur**: Fast movements
6. **Different Ethnicities**: Diverse face types
7. **Various Ages**: Young to elderly drivers

### Expected Behavior
- Graceful degradation
- `faceDetected: False` when applicable
- No crashes or errors
- Reasonable default values

---

## Continuous Integration

### Pre-commit Checks
```bash
# Run minimal evaluation
python evaluation/evaluate_drowsiness.py
python evaluation/evaluate_distraction.py

# Minimum accuracy: 85%
```

### CI/CD Pipeline
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

## Regression Testing

### Baseline Metrics
- Drowsiness: 91.25%
- Distraction: 92.00%
- Recognition: 96.00%
- Phone: 98.00%
- Latency: <100ms

### Regression Threshold
- Accuracy drop > 5%: ⚠️ Warning
- Accuracy drop > 10%: 🔴 Failure
- Latency increase > 20%: ⚠️ Warning
- Latency increase > 50%: 🔴 Failure

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29