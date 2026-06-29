# Dataset Documentation - NexusAI

**Training and Evaluation Dataset Reference**

---

## Dataset Overview

### Purpose
Provide diverse, labeled images for training and evaluating driver monitoring AI models.

### Total Images
- **Drowsiness**: ~400 images
- **Distraction**: ~400 images
- **Face Recognition**: ~150 images
- **Head Pose**: ~200 images
- **Lighting**: ~150 images
- **Total**: ~1,300+ images

### Distribution
- 80% training/evaluation
- 20% reserved for testing

---

## Drowsiness Dataset

### Categories

#### 1. Eyes Open (~100 images)
**Description**: Normal driving posture, eyes clearly open
**Characteristics**:
- Full eye visibility
- Normal eyelid position
- Alert expression
- Frontal face angle

**Use**: Negative samples for drowsiness detection

#### 2. Eyes Closed (~100 images)
**Description**: Sleeping or drowsy state
**Characteristics**:
- Eyelids fully or partially closed
- Drooping eyelids
- Eye distance < 0.015
- Various lighting conditions

**Use**: Positive samples for drowsiness detection

#### 3. Yawning (~100 images)
**Description**: Yawning motion (fatigue indicator)
**Characteristics**:
- Mouth wide open
- Natural yawning expression
- Eye squinting
- Head tilted back

**Use**: Fatigue level classification

#### 4. Fatigued (~100 images)
**Description**: General fatigue indicators
**Characteristics**:
- Multiple fatigue signs combined
- Reduced alertness
- Slack facial muscles
- Various head positions

**Use**: Medium/High fatigue classification

---

## Distraction Dataset

### Categories

#### 1. Focused (~100 images)
**Description**: Driver looking straight ahead
**Characteristics**:
- Eyes forward
- Head centered
- Gaze on road
- Normal posture

**Use**: Negative samples for distraction detection

#### 2. Looking Away (~100 images)
**Description**: Head turned, not looking at road
**Characteristics**:
- Head turned left/right
- Eyes off-center
- Looking at instruments/passengers
- Various angles (15-45 degrees)

**Use**: Positive samples for distraction detection

#### 3. Phone Usage (~100 images)
**Description**: Phone/handheld device in hand
**Characteristics**:
- Phone visible in frame
- Hand holding device
- Downward gaze
- Attention diverted

**Use**: Phone detection classification

#### 4. Talking (~100 images)
**Description**: Mouth movement, speaking
**Characteristics**:
- Lips moving
- Mouth partially open
- Natural conversation
- Various head positions

**Use**: Talking detection classification

---

## Face Recognition Dataset

### Structure
```
face_recognition/
├── driver1/          (~50 images)
│   ├── front/
│   ├── left/
│   ├── right/
│   ├── up/
│   └── down/
├── driver2/          (~50 images)
│   ├── front/
│   ├── left/
│   ├── right/
│   ├── up/
│   └── down/
└── unknown_driver/   (~50 images)
    ├── person_A/
    ├── person_B/
    └── person_C/
```

### Characteristics
- **Clear face visibility**: 95%+
- **Lighting**: Varied (day, night, office)
- **Angles**: Multiple head poses
- **Resolution**: 480x640 minimum
- **Quality**: JPEG 70-85 quality

### Use Cases
- Face embedding extraction
- Cosine similarity matching
- Threshold validation (0.45)

---

## Head Pose Dataset

### Categories

#### Forward (~50 images)
- Head centered
- Eyes forward
- Nose at center

#### Left (~50 images)
- Head rotated left
- 15-45 degree angles
- Eyes tracking left

#### Right (~50 images)
- Head rotated right
- 15-45 degree angles
- Eyes tracking right

#### Up (~25 images)
- Head tilted up
- Looking at ceiling/mirror

#### Down (~25 images)
- Head tilted down
- Looking at dashboard/lap

---

## Lighting Conditions Dataset

### Categories

#### Daylight (~50 images)
- Natural sunlight
- High brightness
- Clear shadows
- Color accurate

#### Office Light (~40 images)
- Fluorescent lighting
- Even illumination
- Low contrast
- Neutral color temp

#### Night/Low Light (~40 images)
- Dark conditions
- Dashboard lights only
- Low contrast
- Challenging detection

#### Mixed Lighting (~20 images)
- Multiple light sources
- Shadows and highlights
- Variable color temp
- Real-world scenarios

---

## Data Acquisition

### Methods
1. **Recorded Video**: Drive footage cropped to frames
2. **Staged Scenarios**: Controlled environment captures
3. **Public Datasets**: Licensed face/behavior datasets
4. **Synthetic Generation**: AI-generated variations

### Quality Checks
- ✅ Face detection passes
- ✅ Image quality > 480p
- ✅ Label accuracy verified
- ✅ No duplicates
- ✅ Balanced distribution

---

## Dataset Statistics

### Image Statistics
| Category | Count | Accuracy |
|----------|-------|----------|
| Drowsiness | 400 | 91.25% |
| Distraction | 400 | 92.00% |
| Face Recog | 150 | 96.00% |
| Head Pose | 200 | 85.00% |
| Lighting | 150 | 88.00% |

### Face Coverage
- Frontal: 60%
- Side angles: 25%
- Up/Down: 15%

### Age/Gender Distribution
- Adult (18-65): 85%
- Young adult (18-30): 40%
- Middle age (30-50): 35%
- Senior (50+): 25%
- Male: 55%
- Female: 45%

---

## Usage Guidelines

### Training
- Use 80% of images
- Augment with rotations/brightness
- Shuffle before training
- Validate on held-out 20%

### Evaluation
- Use separate test set
- Run all evaluation scripts
- Generate accuracy reports
- Compare to baselines

### Privacy
- Ensure consent for data use
- Anonymize if needed
- Follow GDPR/CCPA guidelines
- Audit trail for compliance

---

## Limitations

⚠️ **Known Issues**:
- Limited diversity (mostly indoors)
- Few extreme angles
- Limited nighttime data
- Small number of drivers
- Synthetic data presence

---

## Future Expansion

- [ ] Add 1000+ more images
- [ ] Include diverse ethnicities
- [ ] Add more driving conditions
- [ ] Include weather variations
- [ ] Add video sequences
- [ ] Include multi-face scenarios

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29