# Quick Start Guide - NexusAI Backend

**Get Started in 5 Minutes**

---

## 1. Clone Repository

```bash
git clone https://github.com/ram516/Nexusaibackend.git
cd Nexusaibackend
```

---

## 2. Setup Python Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate
source venv/bin/activate  # macOS/Linux
# or
venv\Scripts\activate  # Windows
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

**Installation Time**: ~5 minutes (first time includes model downloads)

---

## 4. Start Backend

```bash
python run.py
```

**Expected Output**:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
INFO:     Application startup complete
```

---

## 5. Test Endpoints

### Health Check
```bash
curl http://localhost:8000/
```

### API Documentation
```
http://localhost:8000/docs
```

### Test with Sample Image

```bash
# Register driver
curl -X POST http://localhost:8000/register-driver \
  -F "name=Test Driver" \
  -F "driving_style=comfort" \
  -F "ac_temperature=22" \
  -F "ambient_mode=dim" \
  -F "seat_position=upright" \
  -F "assistant_voice=male" \
  -F "file=@sample_face.jpg"

# Detect face
curl -X POST http://localhost:8000/detect-face \
  -F "file=@sample_face.jpg"

# Recognize driver
curl -X POST http://localhost:8000/recognize-driver \
  -F "file=@sample_face.jpg"
```

---

## 6. Run Tests

```bash
# Test face detection
python test_face.py

# Test phone detection
python test_phone.py
```

---

## 7. Run Evaluation

```bash
# Evaluate drowsiness detection
python evaluation/evaluate_drowsiness.py

# Evaluate distraction detection
python evaluation/evaluate_distraction.py

# Evaluate face recognition
python evaluation/evaluate_face_recognition.py
```

---

## Troubleshooting

### Issue: "ModuleNotFoundError: No module named 'cv2'"
**Solution**: Run `pip install opencv-python` again

### Issue: "Port 8000 already in use"
**Solution**: Change port in `run.py` or kill existing process

### Issue: Face not detected
**Reason**: Poor image quality, lighting, or face angle
**Solution**: Use clear frontal face image with good lighting

### Issue: Slow performance
**Reason**: CPU-based processing
**Solution**: Check `requirements-full.txt` for GPU support

---

## Next Steps

1. Read **README.md** for full overview
2. Review **BACKEND_API_DOCUMENTATION.md** for API details
3. Check **PROJECT_DOCUMENTATION.md** for deep dive
4. Review **DEPLOYMENT.md** for production setup

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29