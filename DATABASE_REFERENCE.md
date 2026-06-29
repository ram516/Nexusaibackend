# Database Reference - NexusAI

**SQLite Schema and Data Persistence Documentation**

---

## Database Configuration

**Type**: SQLite3  
**File**: `drivers.db`  
**Connection**: Non-thread-safe (use in single-threaded context)  
**Thread Check**: `check_same_thread=False` (enabled for FastAPI)

```python
import sqlite3

conn = sqlite3.connect(
    "drivers.db",
    check_same_thread=False
)
cursor = conn.cursor()
```

---

## Schema

### Table: `drivers`

```sql
CREATE TABLE IF NOT EXISTS drivers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    embedding TEXT NOT NULL,
    driving_style TEXT,
    ac_temperature TEXT,
    ambient_mode TEXT,
    seat_position TEXT,
    assistant_voice TEXT,
    created_at TEXT NOT NULL
);
```

### Column Definitions

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| `id` | INTEGER | Auto | Primary key, auto-increment |
| `name` | TEXT | Yes | Driver name (e.g., "John Driver") |
| `embedding` | TEXT | Yes | Face embedding as JSON array |
| `driving_style` | TEXT | No | "eco", "comfort", "sport" |
| `ac_temperature` | TEXT | No | Temperature preference |
| `ambient_mode` | TEXT | No | "off", "dim", "medium", "bright" |
| `seat_position` | TEXT | No | Seat configuration |
| `assistant_voice` | TEXT | No | "male", "female" |
| `created_at` | TEXT | Yes | ISO 8601 timestamp |

---

## Operations

### INSERT Driver

```python
import json
from datetime import datetime

cursor.execute("""
    INSERT INTO drivers (
        name, embedding, driving_style, ac_temperature,
        ambient_mode, seat_position, assistant_voice, created_at
    )
    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
""", (
    "John Driver",
    json.dumps(embedding.tolist()),  # Convert numpy array to JSON
    "comfort",
    "22",
    "dim",
    "upright",
    "male",
    datetime.now().isoformat()
))
conn.commit()
```

### SELECT All Drivers

```python
cursor.execute("SELECT name, embedding FROM drivers")
drivers = cursor.fetchall()

for name, embedding_json in drivers:
    embedding = np.array(json.loads(embedding_json))
    # Use embedding for recognition
```

### SELECT By ID

```python
cursor.execute("SELECT * FROM drivers WHERE id = ?", (driver_id,))
driver = cursor.fetchone()
```

### DELETE Driver

```python
cursor.execute("DELETE FROM drivers WHERE id = ?", (driver_id,))
conn.commit()
```

### DELETE All Drivers

```python
cursor.execute("DELETE FROM drivers")
conn.commit()
```

---

## Embedding Storage Format

### Saving
```python
import numpy as np
import json

embedding = np.array([0.123, 0.456, ...])  # 512 floats
embedding_json = json.dumps(embedding.tolist())
# Result: "[0.123, 0.456, ...]"
```

### Loading
```python
embedding_json = "[0.123, 0.456, ...]"
embedding = np.array(json.loads(embedding_json))
```

---

## Performance Considerations

### Query Performance
- **Indexed column needed**: `name` (for frequent lookups)
- **Current bottleneck**: Linear scan through all drivers for recognition

### Recommended Index
```sql
CREATE INDEX idx_drivers_name ON drivers(name);
```

### Scalability Limits
- **SQLite Limit**: ~10 million rows
- **Practical Limit**: 1,000+ drivers without noticeable slowdown
- **Recommendation**: Use PostgreSQL for >10K drivers

---

## Production Recommendations

1. **Encryption**: Encrypt embedding column
2. **Backup**: Regular database snapshots
3. **Connection Pool**: Use database connection pool for multiple workers
4. **Migration**: Use Alembic for schema versioning
5. **Audit Trail**: Add `updated_at` column for tracking changes

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29