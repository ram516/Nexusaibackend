# Backend Architecture - NexusAI

**Detailed Architecture and System Design Documentation**

---

## Architecture Layers

```
┌─────────────────────────────────────────────┐
│         Presentation Layer                  │
│  (HTTP/REST API - FastAPI Routing)          │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Service Layer                       │
│  ├─ Face Detection Service                  │
│  ├─ Phone Detection Service                 │
│  ├─ Event Generation Service                │
│  └─ Recognition Service                     │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Model Layer                         │
│  ├─ Pydantic Models (Validation)            │
│  ├─ Data Models (Telemetry)                 │
│  └─ Response Schemas                        │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Integration Layer                   │
│  ├─ OpenCV (Face Detection)                 │
│  ├─ MediaPipe (Landmarks)                   │
│  ├─ InsightFace (Embeddings)                │
│  ├─ YOLOv8 (Phone Detection)                │
│  └─ SQLite (Persistence)                    │
└─────────────────────────────────────────────┘
```

---

## Service Layer Components

### Face Detection Service
- Image decoding & validation
- OpenCV cascade classifier
- MediaPipe 468 landmark extraction
- Drowsiness detection via eye distance
- Distraction detection via head pose
- Metric calculation (safety, attention, fatigue)

### Phone Detection Service
- YOLOv8 Nano object detection
- Cell phone class filtering
- Real-time inference

### Event Generation Service
- Event type categorization
- Cooldown-based throttling
- Severity assignment
- Event memory management

---

## Data Flow

```
Request (JPEG) → Decode → OpenCV Face → MediaPipe Mesh
                              │              │
                              └──────┬───────┘
                                     │
                        Phone Detection (YOLOv8)
                                     │
                              Calculate Metrics
                                     │
                        Generate Events & Response
                                     │
                          Return TelemetryResponse
```

---

## State Management

### Global Variables (Thread Safety Issue)
```python
drowsy_counter          # Incremented per drowsy frame
warning_counter         # Warning escalation level
drowsy_start_time       # Drowsiness duration tracker
looking_away_counter    # Consecutive away frames
blink_counter           # Total blink count
blink_detected          # Blink state flag
blink_start_time        # Reference for blink rate
gaze_history[]          # 20-frame direction buffer
```

### Production Recommendation
- Use Redis for distributed state
- Implement request-scoped state
- Add threading locks for concurrent requests

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29