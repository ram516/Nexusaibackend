# Known Limitations - NexusAI Backend

**Documented Constraints and Edge Cases**

---

## Critical Limitations

### 1. Single Face Detection ⚠️
**Issue**: Only one face per frame is analyzed

**Impact**:
- Multiple faces (passengers): Only driver analyzed
- Occluded faces: No fallback
- Side faces: May not detect if angle >45°

**Workaround**:
- Ensure clear driver visibility
- Position camera for frontal face
- Remove obstructions

**Future Fix**: Multi-face handling with face tracking

---

### 2. Thread Safety ⚠️
**Issue**: Global state not thread-safe

**Affected Variables**:
- `drowsy_counter`
- `warning_counter`
- `gaze_history`
- `blink_counter`

**Impact**:
- Race conditions possible with >1 concurrent request
- State corruption possible
- Inconsistent metrics

**Current Mitigation**:
- Use single worker (Uvicorn default)
- Sequential request processing

**Production Workaround**:
- Use threading.Lock() for state access
- Migrate to Redis for distributed state
- One worker per GPU

---

### 3. CPU-Bound Processing ⚠️
**Issue**: No GPU acceleration

**Impact**:
- Latency: 60-110ms (ideal)
- Throughput: Limited to ~10 req/s per worker
- High CPU usage: 60-75%

**Current System**:
- CPU implementation only
- No CUDA support configured
- OpenCV CPU mode

**Performance Issues**:
- Slow on weak CPUs (<4 cores)
- Cannot handle >40 req/s (4 workers)
- Expensive for high-volume deployments

**Solution**:
- Enable CUDA for GPU servers
- Use TensorRT for inference
- Upgrade to GPU hardware

---

### 4. SQLite Concurrency ⚠️
**Issue**: SQLite not designed for concurrent writes

**Impact**:
- Single writer at a time
- Read locks during writes
- Potential slowdown with high concurrency
- Not suitable for >1000 concurrent users

**Current Setup**:
- `check_same_thread=False` enabled
- No connection pool
- No query optimization

**Scalability Concern**:
- Breaks down with >100 concurrent drivers
- Driver registration may queue
- Driver recognition may stall

**Production Solution**:
- Migrate to PostgreSQL
- Implement connection pooling
- Add caching (Redis)
- Use read replicas

---

## Functional Limitations

### 5. No Encrypted Database
**Issue**: Face embeddings stored as plain text

**Risk**: Unauthorized access to embeddings

**Mitigation**:
- Implement AES-256 encryption
- Use encrypted file system
- Restrict database file permissions

---

### 6. No Authentication
**Issue**: Any client can call all endpoints

**Risk**:
- Unauthorized data access
- DoS attacks
- Data tampering

**Recommendation**:
- Add JWT authentication
- Implement OAuth2 flow
- API key validation

---

### 7. No Rate Limiting
**Issue**: Unlimited requests per client

**Risk**:
- DoS vulnerability
- Resource exhaustion
- Unfair resource sharing

**Solution**:
- Implement token bucket algorithm
- Use Redis for distributed rate limiting
- Configure per-IP limits

---

### 8. No Request Logging
**Issue**: No audit trail of API calls

**Risk**:
- Can't debug issues
- No compliance audit trail
- Security investigation impossible

**Solution**:
- Add structured logging
- Log all requests/responses
- Ship logs to centralized system

---

## Performance Limitations

### 9. Latency > 100ms Possible
**Issue**: Latency can reach 110-115ms in p99

**Causes**:
- GC pauses
- Model loading
- System load
- I/O operations

**Impact**:
- 30 FPS frontend gets 3-5 stale frames
- Reduced real-time responsiveness
- May miss rapidly changing conditions

**Improvement**:
- Use GPU acceleration
- Preload all models at startup
- Optimize OpenCV parameters
- Use faster inference engines

---

### 10. Memory Usage
**Issue**: 200-400 MB per worker

**Breakdown**:
- InsightFace model: ~130 MB
- OpenCV + MediaPipe: ~40 MB
- Runtime overhead: ~30-100 MB

**Limitation**:
- Can't deploy on <512 MB RAM devices
- Container size large
- Slow startup

---

## Detection Accuracy Limitations

### 11. Low Lighting Performance ⚠️
**Issue**: Accuracy degrades in darkness

**Problem**:
- Night driving scenarios
- Dashboard lights only
- Shadows and backlighting

**Impact**:
- Face detection fails: <80% accuracy
- Landmark extraction unreliable
- Metric calculations inaccurate

**Workaround**:
- Use IR cameras
- Improve vehicle lighting
- Accept lower accuracy at night

---

### 12. Extreme Head Angles
**Issue**: Detection fails for angles >45°

**Scenario**:
- Looking far left/right
- Leaning back
- Extreme yawn

**Impact**:
- Face may not detect
- Landmarks unreliable
- Metrics invalid

---

### 13. Glasses/Sunglasses
**Issue**: Eye detection affected by eyewear

**Problem**:
- Dark lenses obscure eyes
- Reflections cause issues
- Reflections interfere with MediaPipe

**Impact**:
- Eye distance metric unreliable
- Drowsiness detection fails
- Blink rate invalid

---

### 14. Facial Hair/Makeup
**Issue**: Landmarks shift with facial hair

**Problem**:
- Heavy beards change landmark positions
- Makeup alters face shape
- Expressions amplified by makeup

**Impact**:
- Yawning detection threshold off
- Head pose estimation inaccurate
- False positives possible

---

## Edge Cases

### 15. Multiple Faces
**Current**: Only analyzes largest face
**Issue**: Passengers or reflections cause problems
**Fix**: Need multi-face tracking

### 16. Occlusion
**Current**: No handling
**Issue**: Hands, steering wheel, phone block face
**Fix**: Implement temporal tracking

### 17. Motion Blur
**Current**: Not handled
**Issue**: Fast head movements blur image
**Fix**: Use video processing instead of frames

### 18. Very Young/Old Drivers
**Current**: Dataset biased toward 18-65
**Issue**: Child or elderly drivers may have different metrics
**Fix**: Expand dataset and recalibrate

---

## Deployment Limitations

### 19. No Horizontal Scaling Ready
**Issue**: Global state prevents easy scaling

**Current**: Works with single worker or multiple independent instances

**Limitation**: Can't share state across workers

**Solution**: Refactor to stateless service with Redis

---

### 20. No Kubernetes Support
**Issue**: No orchestration configuration

**Missing**:
- Helm charts
- Deployment manifests
- Service mesh integration

**Effort**: 2-3 days of work

---

## Workarounds and Recommendations

### Immediate (For Production)
1. Add authentication (JWT)
2. Implement rate limiting
3. Add request logging
4. Enable HTTPS/TLS
5. Set up database backups

### Short Term (1-2 months)
1. Refactor global state to Redis
2. Add GPU support
3. Implement multi-worker safety
4. Add comprehensive monitoring
5. Set up CI/CD pipeline

### Medium Term (3-6 months)
1. Multi-face support
2. Temporal tracking
3. Advanced head pose estimation
4. Micro-sleep detection
5. Kubernetes deployment

### Long Term (6+ months)
1. Custom model training
2. Real-time 3D face modeling
3. Gaze direction accuracy
4. Integration with vehicle APIs
5. Advanced analytics

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29

**Status**: Production Ready with Known Limitations