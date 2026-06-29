# Backend API Documentation - NexusAI

**Complete REST API Reference for Backend Developers**

---

## Base URL

**Development**: `http://localhost:8000`  
**Production**: `https://api.example.com`

---

## Authentication

Currently: **None (Open API)**  
Future: JWT Bearer tokens

---

## Response Format

All responses are JSON. Success responses include the requested data. Error responses include an error message.

---

## Endpoints

### 1. Health Check

**GET** `/`

Health check endpoint to verify backend is running.

**Request:**
```
GET / HTTP/1.1
Host: localhost:8000
```

**Response:**
```json
{
  "message": "Smart Vehicle AI Backend Running"
}
```

**Status Code**: `200 OK`

---

### 2. Detect Face (PRIMARY ENDPOINT)

**POST** `/detect-face`

Real-time driver monitoring endpoint. Analyzes a frame for drowsiness, distraction, phone usage, and other safety metrics.

**Request:**
```
POST /detect-face HTTP/1.1
Host: localhost:8000
Content-Type: multipart/form-data

file: <JPEG image binary>
```

**Image Specifications:**
- **Format**: JPEG
- **Size**: 50-200 KB
- **Resolution**: 640x480 to 1280x720 (recommended)
- **Quality**: 70-85 JPEG quality
- **Content**: Clear driver face, preferably frontal

**Response Schema:**
```json
{
  "driver": {
    "faceDetected": boolean,
    "faceCount": integer,
    "isDrowsy": boolean,
    "isYawning": boolean,
    "isTalking": boolean,
    "phoneDetected": boolean,
    "attentionScore": integer (0-100),
    "attentionStatus": string,
    "safetyScore": integer (0-100),
    "fatigueLevel": string,
    "warningCount": integer (0-5),
    "emergencyMode": boolean,
    "recommendedAction": string,
    "blinkRate": integer,
    "gazeStability": integer (0-100),
    "headDirection": string,
    "lookingAway": boolean
  },
  "vision": {
    "trackingState": string,
    "meshEnabled": boolean,
    "meshConfidence": float (0-100),
    "pipelineStatus": string,
    "fps": integer,
    "latency": integer
  },
  "vehicle": {
    "riskLevel": string,
    "safetyMode": string,
    "assistState": string
  },
  "events": [
    {
      "type": string,
      "severity": string
    }
  ]
}
```

**Status Code**: `200 OK`

**Example cURL:**
```bash
curl -X POST http://localhost:8000/detect-face \
  -F "file=@driver_frame.jpg"
```

**Python Example:**
```python
import requests

with open("driver_frame.jpg", "rb") as f:
    files = {"file": f}
    response = requests.post(
        "http://localhost:8000/detect-face",
        files=files
    )
    data = response.json()
    print(f"Drowsy: {data['driver']['isDrowsy']}")
    print(f"Safety Score: {data['driver']['safetyScore']}")
```

**Field Reference:**

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| `faceDetected` | bool | - | Is driver face visible? |
| `faceCount` | int | 0+ | Number of faces in frame |
| `isDrowsy` | bool | - | Eyes closed detected? |
| `isYawning` | bool | - | Yawning motion detected? |
| `isTalking` | bool | - | Mouth movement detected? |
| `phoneDetected` | bool | - | Cell phone detected? |
| `attentionScore` | int | 0-100 | Focus level (80+ Good, <50 Critical) |
| `safetyScore` | int | 0-100 | Overall safety (80+ Safe, <60 Unsafe) |
| `fatigueLevel` | str | Low\|Medium\|High | Fatigue assessment |
| `warningCount` | int | 0-5 | Cumulative warnings |
| `emergencyMode` | bool | - | Critical conditions? |
| `blinkRate` | int | 0-60 | Blinks per minute |
| `gazeStability` | int | 0-100 | Eye movement smoothness |
| `trackingState` | str | Locked\|Lost | Face tracking quality |
| `fps` | int | 0-60 | Frames processed/second |
| `latency` | int | 0-500ms | Processing time |

---

### 3. Register Driver

**POST** `/register-driver`

Enroll a new driver with face photo and preferences.

**Request:**
```
POST /register-driver HTTP/1.1
Host: localhost:8000
Content-Type: multipart/form-data

name: string
driving_style: string
ac_temperature: string
ambient_mode: string
seat_position: string
assistant_voice: string
file: <JPEG image binary>
```

**Form Parameters:**

| Parameter | Type | Required | Example |
|-----------|------|----------|---------|
| `name` | string | Yes | "John Driver" |
| `driving_style` | string | Yes | "eco" \| "comfort" \| "sport" |
| `ac_temperature` | string | Yes | "22" |
| `ambient_mode` | string | Yes | "off" \| "dim" \| "bright" |
| `seat_position` | string | Yes | "upright" \| "reclined" |
| `assistant_voice` | string | Yes | "female" \| "male" |
| `file` | file | Yes | Clear face photo (JPEG) |

**Response:**
```json
{
  "success": boolean,
  "message": string
}
```

**Success Response:**
```json
{
  "success": true,
  "message": "John Driver registered"
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "No face detected"
}
```

**Status Code**: `200 OK` (success) or `400 Bad Request` (no face)

**Example cURL:**
```bash
curl -X POST http://localhost:8000/register-driver \
  -F "name=John Driver" \
  -F "driving_style=comfort" \
  -F "ac_temperature=22" \
  -F "ambient_mode=dim" \
  -F "seat_position=upright" \
  -F "assistant_voice=male" \
  -F "file=@profile_photo.jpg"
```

**Python Example:**
```python
import requests

data = {
    "name": "John Driver",
    "driving_style": "comfort",
    "ac_temperature": "22",
    "ambient_mode": "dim",
    "seat_position": "upright",
    "assistant_voice": "male"
}

files = {"file": open("profile_photo.jpg", "rb")}

response = requests.post(
    "http://localhost:8000/register-driver",
    data=data,
    files=files
)

result = response.json()
print(f"Success: {result['success']}")
print(f"Message: {result['message']}")
```

---

### 4. Recognize Driver

**POST** `/recognize-driver`

Identify a driver from a face image.

**Request:**
```
POST /recognize-driver HTTP/1.1
Host: localhost:8000
Content-Type: multipart/form-data

file: <JPEG image binary>
```

**Response:**
```json
{
  "matched": boolean,
  "driver": string (optional),
  "confidence": number (0-100, optional)
}
```

**Matched Response:**
```json
{
  "matched": true,
  "driver": "John Driver",
  "confidence": 92
}
```

**Unknown Driver Response:**
```json
{
  "matched": false,
  "message": "Unknown driver"
}
```

**Status Code**: `200 OK`

**Matching Logic:**
- Confidence > 45: Considered a match
- Confidence 45-70: Weak match (may need verification)
- Confidence > 70: Strong match (recommended threshold)
- Confidence <= 45: Unknown driver

**Example cURL:**
```bash
curl -X POST http://localhost:8000/recognize-driver \
  -F "file=@current_driver.jpg"
```

**Python Example:**
```python
import requests

with open("current_driver.jpg", "rb") as f:
    files = {"file": f}
    response = requests.post(
        "http://localhost:8000/recognize-driver",
        files=files
    )
    
    data = response.json()
    
    if data["matched"]:
        print(f"Driver: {data['driver']}")
        print(f"Confidence: {data['confidence']}%")
    else:
        print("Unknown driver")
```

---

### 5. Clear Drivers

**DELETE** `/clear-drivers`

Remove all registered driver profiles from database.

**Request:**
```
DELETE /clear-drivers HTTP/1.1
Host: localhost:8000
```

**Response:**
```json
{
  "success": true,
  "message": "All driver profiles cleared"
}
```

**Status Code**: `200 OK`

**⚠️ Warning**: This operation is irreversible!

**Example cURL:**
```bash
curl -X DELETE http://localhost:8000/clear-drivers
```

---

## Error Responses

### 400 Bad Request
```json
{
  "detail": "Invalid image format or no face detected"
}
```

### 422 Unprocessable Entity
```json
{
  "detail": [
    {
      "loc": ["body", "file"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

### 500 Internal Server Error
```json
{
  "detail": "Internal server error"
}
```

---

## Rate Limiting

Currently: **No rate limiting**

Recommended Production:
- 1000 requests per minute per IP
- 30 requests per minute per driver ID

---

## CORS Headers

**Development:**
```
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: GET, POST, DELETE, OPTIONS
Access-Control-Allow-Headers: *
```

**Production:**
```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
```

---

## Testing with cURL

### Test All Endpoints

```bash
# 1. Health check
curl http://localhost:8000/

# 2. Register driver
curl -X POST http://localhost:8000/register-driver \
  -F "name=Test Driver" \
  -F "driving_style=comfort" \
  -F "ac_temperature=22" \
  -F "ambient_mode=dim" \
  -F "seat_position=upright" \
  -F "assistant_voice=male" \
  -F "file=@test_face.jpg"

# 3. Recognize driver
curl -X POST http://localhost:8000/recognize-driver \
  -F "file=@test_face.jpg"

# 4. Detect face
curl -X POST http://localhost:8000/detect-face \
  -F "file=@test_face.jpg"

# 5. Clear all drivers
curl -X DELETE http://localhost:8000/clear-drivers
```

---

## Testing with Python

```python
import requests
import json

BASE_URL = "http://localhost:8000"

# Health check
resp = requests.get(f"{BASE_URL}/")
print(resp.json())

# Register driver
with open("test_face.jpg", "rb") as f:
    resp = requests.post(
        f"{BASE_URL}/register-driver",
        data={
            "name": "Test Driver",
            "driving_style": "comfort",
            "ac_temperature": "22",
            "ambient_mode": "dim",
            "seat_position": "upright",
            "assistant_voice": "male"
        },
        files={"file": f}
    )
    print("Register:", resp.json())

# Recognize driver
with open("test_face.jpg", "rb") as f:
    resp = requests.post(
        f"{BASE_URL}/recognize-driver",
        files={"file": f}
    )
    print("Recognize:", resp.json())

# Detect face
with open("test_face.jpg", "rb") as f:
    resp = requests.post(
        f"{BASE_URL}/detect-face",
        files={"file": f}
    )
    data = resp.json()
    print(f"Safety Score: {data['driver']['safetyScore']}")
    print(f"Attention: {data['driver']['attentionStatus']}")

# Clear drivers
resp = requests.delete(f"{BASE_URL}/clear-drivers")
print("Clear:", resp.json())
```

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29
