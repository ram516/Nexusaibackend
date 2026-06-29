# NexusAI Backend - Smart Vehicle AI System

## 🎯 Project Overview

**NexusAI Backend** is a FastAPI-powered intelligent driver monitoring system designed for real-time vehicle safety. It combines advanced computer vision, face recognition, and behavioral analysis to detect driver drowsiness, distraction, and unsafe conditions in real-time.

**Status**: Production-Ready MVP  
**Language**: Python 100%  
**Framework**: FastAPI 0.136.1  
**Python Version**: 3.10+

---

## 🎬 Quick Start

### Prerequisites
- Python 3.10+
- CUDA 11.0+ (optional, for GPU acceleration)
- 4GB+ RAM

### Installation

```bash
# Clone repository
git clone https://github.com/ram516/Nexusaibackend.git
cd Nexusaibackend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Run Backend

```bash
# Start FastAPI server on localhost:8000
python run.py

# Server will be available at:
# http://localhost:8000
# Docs: http://localhost:8000/docs (Swagger UI)
```

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────┐
│         Frontend (React + Vite)                 │
│      (Driver Monitoring Dashboard)              │
└────────────────┬────────────────────────────────┘
                 │ HTTP/REST API (CORS enabled)
                 ▼
┌─────────────────────────────────────────────────┐
│    FastAPI Application (app/main.py)            │
├─────────────────────────────────────────────────┤
│  Core Endpoints:                                │
│  ├─ /detect-face (Real-time monitoring)         │
│  ├─ /register-driver (Face enrollment)          │
│  ├─ /recognize-driver (Driver identification)   │
│  └─ /clear-drivers (Database management)        │
└────────┬──────────────────────────────────────┬─┘
         │                                      │
    ┌────▼─────────┐                  ┌────────▼────┐
    │ AI Services  │                  │ Database    │
    ├──────────────┤                  ├─────────────┤
    │Face Detection│                  │SQLite DB    │
    │Drowsiness    │                  │(drivers.db) │
    │Distraction   │                  └─────────────┘
    │Phone Detect  │
    └──────────────┘
```

---

## 📁 Project Structure

```
Nexusaibackend/
├── app/
│   ├── __init__.py
│   ├── main.py                           # FastAPI app entry point
│   ├── api/
│   │   └── routes/
│   │       └── detection.py              # Detection endpoints
│   ├── models/
│   │   └── telemetry_models.py          # Pydantic data models
│   └── services/
│       ├── face_detection_service.py     # Core AI detection logic
│       ├── phone_detection_service.py    # Phone detection (YOLOv8)
│       ├── event_service.py              # Event generation logic
│       └── __init__.py
├── dataset/
│   ├── drowsiness/                       # Training dataset
│   │   ├── eyes_open/
│   │   ├── eyes_closed/
│   │   ├── yawning/
│   │   └── fatigued/
│   ├── distraction/
│   │   ├── focused/
│   │   ├── looking_away/
│   │   ├── phone_usage/
│   │   └── talking/
│   ├── face_recognition/                 # Face recognition dataset
│   │   ├── driver1/
│   │   ├── driver2/
│   │   └── unknown_driver/
│   ├── head_pose/                        # Head pose dataset
│   └── lighting_conditions/              # Lighting dataset
├── evaluation/
│   ├── evaluate_face_recognition.py
│   ├── evaluate_drowsiness.py
│   ├── evaluate_distraction.py
│   ├── evaluate_head_pose.py
│   ├── evaluate_lighting.py
│   ├── generate_csv.py
│   ├── generate_charts.py
│   ├── generate_report.py
│   ├── test_yawning_threshold.py
│   └── results/                          # Evaluation output
├── face_engine.py                        # InsightFace wrapper
├── database.py                           # SQLite connection
├── drivers.db                            # SQLite database
├── yolov8n.pt                            # YOLOv8 nano model
├── requirements.txt                      # Core dependencies
├── requirements-full.txt                 # Full dependency list
├── run.py                                # Entry point
├── test_face.py                          # Face embedding test
├── test_phone.py                         # Phone detection test
└── FRONTEND_REFERENCE_FOR_BACKEND.md    # API contract

```

---

## 🚀 Core Features

### 1. **Real-Time Driver Monitoring** (`/detect-face`)
- Continuous face detection (30 FPS target)
- Drowsiness detection via eye tracking
- Distraction detection (head pose, gaze direction)
- Yawning detection (fatigue indicator)
- Phone usage detection
- Safety scoring (0-100)
- Warning escalation system
- Emergency mode activation

### 2. **Face Recognition** (`/recognize-driver`)
- InsightFace embeddings (512-dim vectors)
- Cosine similarity matching
- Driver enrollment with preferences
- Unknown driver detection
- Confidence scoring

### 3. **AI Event System**
- Real-time event generation
- Event severity levels (info, warning, critical, monitoring)
- Cooldown-based event throttling
- Event logging for telemetry

### 4. **Telemetry Pipeline**
- Vision metrics (FPS, latency, tracking state)
- Driver metrics (attention, fatigue, safety)
- Vehicle state (risk level, safety mode)
- Event stream

---

## 📊 API Endpoints

### 1. Home Route
```
GET /
Response: {"message": "Smart Vehicle AI Backend Running"}
```

### 2. Detect Face (Real-time Monitoring)
```
POST /detect-face
Content-Type: multipart/form-data

Request:
  - file: JPEG image binary

Response:
{
  "driver": {
    "faceDetected": boolean,
    "isDrowsy": boolean,
    "isYawning": boolean,
    "isTalking": boolean,
    "phoneDetected": boolean,
    "attentionScore": 0-100,
    "attentionStatus": "Focused|Drowsy|Distracted|Phone Usage|No Face",
    "safetyScore": 0-100,
    "fatigueLevel": "Low|Medium|High",
    "warningCount": 0-5,
    "emergencyMode": boolean,
    "recommendedAction": string,
    "blinkRate": int,
    "gazeStability": 0-100,
    "headDirection": "Center|Left|Right|Up|Down",
    "lookingAway": boolean
  },
  "vision": {
    "trackingState": "Locked|Lost",
    "meshEnabled": boolean,
    "meshConfidence": 0-100,
    "pipelineStatus": "Operational",
    "fps": int,
    "latency": int (milliseconds)
  },
  "vehicle": {
    "riskLevel": "Low|Warning|Critical",
    "safetyMode": "Monitoring|Protection",
    "assistState": "Active|Intervention"
  },
  "events": [
    {
      "type": string,
      "severity": "info|warning|critical|monitoring"
    }
  ]
}
```

### 3. Register Driver
```
POST /register-driver
Content-Type: multipart/form-data

Request:
  - name: string (driver name)
  - driving_style: string
  - ac_temperature: string
  - ambient_mode: string
  - seat_position: string
  - assistant_voice: string
  - file: JPEG image (face photo)

Response:
{
  "success": boolean,
  "message": string
}
```

### 4. Recognize Driver
```
POST /recognize-driver
Content-Type: multipart/form-data

Request:
  - file: JPEG image binary

Response:
{
  "matched": boolean,
  "driver": string (driver name if matched),
  "confidence": 0-100
}
```

### 5. Clear Drivers
```
DELETE /clear-drivers

Response:
{
  "success": boolean,
  "message": "All driver profiles cleared"
}
```

---

## 🧠 AI Pipeline

### Face Detection & Landmark Extraction
1. **OpenCV Cascade Classifier**: Fast frontal face detection
2. **MediaPipe Face Mesh**: 468 facial landmarks
3. **InsightFace**: 512-dim face embeddings

### Drowsiness Detection
- **Eye Distance Metric**: Distance between top/bottom eyelid landmarks
- **Threshold**: < 0.015 = drowsy
- **Duration Tracking**: Accumulates drowsiness time
- **Warning System**: 2s → Warn 1, 4s → Warn 2, 6s+ → Warn 3

### Distraction Detection
- **Head Pose Estimation**: Left/right face distance balance
- **Gaze Direction**: 20-frame history of head direction
- **Gaze Stability**: % of center-looking frames (target: 80%+)
- **Alert**: Looking away > 3 frames = distracted

### Phone Detection
- **YOLOv8 Nano**: Real-time object detection
- **Class Filter**: "cell phone" detection
- **Single-shot inference**: Fast, lightweight

### Event Generation
- **Drowsiness**: 10-second cooldown, critical severity
- **Distraction**: 6-second cooldown, warning severity
- **Phone**: Every frame, critical severity
- **Yawning/Talking**: Every frame, warning/info severity
- **Emergency**: Cooldown-aware escalation

### Telemetry Calculation
```
Safety Score = 100
  - 50 (if drowsy)
  - 20 (if yawning)
  - 10 (if talking)
  - 25 (if looking away)
  - 40 (if no face)
  - 35 (if phone detected)
  → Clamped to [0, 100]

Fatigue Level:
  - fatigue_score = (is_yawning * 30) + (is_drowsy * 50)
  - High: score >= 70
  - Medium: 30 <= score < 70
  - Low: score < 30

Emergency Mode:
  - warning_counter >= 3
```

---

## 🗄️ Database Schema

### SQLite (drivers.db)

```sql
CREATE TABLE drivers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    embedding TEXT,              -- JSON array of 512 floats
    driving_style TEXT,
    ac_temperature TEXT,
    ambient_mode TEXT,
    seat_position TEXT,
    assistant_voice TEXT,
    created_at TEXT              -- ISO 8601 timestamp
);
```

**Driver Profile Storage:**
- Face embedding stored as JSON string: `json.dumps(embedding.tolist())`
- Preferences stored as TEXT (can expand later)
- Created at timestamp for audit trail

---

## ⚙️ Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Framework** | FastAPI | 0.136.1 |
| **Server** | Uvicorn | 0.46.0 |
| **Face Detection** | OpenCV | 4.13.0.92 |
| **Face Landmarks** | MediaPipe | 0.10.14 |
| **Face Embeddings** | InsightFace | 1.0.1 |
| **Object Detection** | YOLOv8 | ultralytics |
| **Computer Vision** | OpenCV | 4.13.0.92 |
| **Numerical Computing** | NumPy | 2.2.6 |
| **Data Validation** | Pydantic | 2.13.4 |
| **Runtime** | ONNX Runtime | 1.23.2 |
| **Database** | SQLite3 | Built-in |

---

## 🔧 Configuration

### CORS Configuration
```python
CORSMiddleware enabled for:
- Frontend: http://localhost:5173 (development)
- Methods: GET, POST, DELETE
- Credentials: Enabled
- Headers: *
```

### Performance Tuning
```python
# Face Mesh
static_image_mode = False      # Real-time mode
max_num_faces = 1              # Single driver
refine_landmarks = True        # Better accuracy

# Face Detection
scaleFactor = 1.1
minNeighbors = 5
minSize = (30, 30)

# Thresholds
DROWSY_EYE_THRESHOLD = 0.015
YAWNING_THRESHOLD = 0.05
TALKING_MIN_DISTANCE = 0.015
TALKING_MAX_DISTANCE = 0.05
```

---

## 📈 Performance Metrics

### Real-time Processing
- **Target FPS**: 24-30 (client-side 30 FPS)
- **Latency Target**: < 100ms per frame
- **Actual Performance**: Varies by system (CPU vs GPU)

### Model Sizes
- **YOLOv8 Nano**: ~6.5 MB
- **InsightFace Model**: ~130 MB (downloaded on first use)
- **MediaPipe Model**: ~1 MB (built-in)

### Throughput
- **Max Concurrent Requests**: Limited by CPU cores
- **Recommended**: Single-threaded for consistency
- **Scaling**: Use multiple instances behind reverse proxy

---

## 🛡️ Security Considerations

### Data Protection
- Face embeddings stored in local SQLite (no encryption)
- Recommendations for production:
  - Use encrypted database
  - Implement authentication
  - Add rate limiting
  - Use HTTPS only
  - Implement audit logging

### Input Validation
- Image size validation
- File type checking (JPEG only)
- Dimension constraints (30x30 minimum)

### CORS Security
- Restrict origins in production
- Use environment variables for allowed domains
- Implement CSRF tokens if needed

---

## 🚦 Deployment

### Local Development
```bash
python run.py
# Auto-reload enabled, runs on 0.0.0.0:8000
```

### Production (Gunicorn + Nginx)
```bash
# gunicorn with 4 workers
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app.main:app

# Nginx reverse proxy
server {
    listen 80;
    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

### Docker (Recommended)
```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "run.py"]
```

---

## 📝 Evaluation & Benchmarks

The project includes comprehensive evaluation scripts:

1. **Face Recognition**: `evaluate_face_recognition.py`
2. **Drowsiness Detection**: `evaluate_drowsiness.py`
3. **Distraction Detection**: `evaluate_distraction.py`
4. **Head Pose**: `evaluate_head_pose.py`
5. **Lighting Conditions**: `evaluate_lighting.py`

Run evaluations:
```bash
python evaluation/evaluate_drowsiness.py
python evaluation/evaluate_distraction.py
python evaluation/evaluate_face_recognition.py
```

---

## 🔍 Known Limitations

1. **Single Driver Detection**: Only monitors first/primary face
2. **Lighting Sensitivity**: Performance degrades in low light
3. **No Multi-Face Handling**: Ignores additional faces (for now)
4. **No GPU Requirement**: Uses CPU by default (slow)
5. **Database Not Encrypted**: SQLite plain text
6. **No Authentication**: Open API (add in production)
7. **Stateful Counters**: Global state in memory (not thread-safe)
8. **Head Pose Accuracy**: Simplified left/right detection

---

## 🎯 Future Enhancements

- [ ] GPU acceleration support (CUDA)
- [ ] Multi-threaded request handling
- [ ] Encrypted database support
- [ ] Authentication & authorization
- [ ] Rate limiting & throttling
- [ ] More sophisticated head pose estimation
- [ ] Micro-sleep detection
- [ ] Gaze direction accuracy improvement
- [ ] Time-series analytics
- [ ] Integration with vehicle APIs
- [ ] Cloud deployment templates
- [ ] Docker Compose setup
- [ ] Prometheus metrics export
- [ ] Detailed logging & monitoring

---

## 🐛 Debugging

Enable detailed logging:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

Test endpoints:
```bash
# Test face detection
curl -X POST http://localhost:8000/detect-face \
  -F "file=@test_image.jpg"

# Test recognition
curl -X POST http://localhost:8000/recognize-driver \
  -F "file=@driver_image.jpg"
```

---

## 📚 Documentation Files

- **README.md** - This file (Overview & Quick Start)
- **PROJECT_DOCUMENTATION.md** - Complete technical documentation
- **BACKEND_API_DOCUMENTATION.md** - Detailed API reference
- **BACKEND_ARCHITECTURE.md** - Architecture deep dive
- **AI_PIPELINE.md** - AI model details
- **DATABASE_REFERENCE.md** - Database schema & operations
- **DEPLOYMENT.md** - Deployment guide
- **QUICK_START_GUIDE.md** - Developer quick start
- **TESTING_AND_EVALUATION.md** - Evaluation methodology
- **FRONTEND_REFERENCE_FOR_BACKEND.md** - API contract

---

## 📞 Support

For issues, questions, or contributions:
1. Check existing documentation
2. Review evaluation scripts for expected behavior
3. Test with provided datasets
4. Enable debug logging
5. Create detailed issue reports

---

## 📄 License

Specify your license here (MIT, Apache 2.0, etc.)

---

**Last Updated**: 2026-06-29  
**Version**: 1.0.0  
**Status**: Production Ready (MVP)
