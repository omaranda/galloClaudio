---
name: camera-trap
description: Scaffold or extend a camera trap image processing pipeline using MegaDetector v5 or DeepFaune, with S3 input, SQS-driven batch processing, and PostgreSQL/PostGIS detection storage.
---

You are acting as the eo-cv-specialist agent. Set up a camera trap processing pipeline.

**Step 1 — Define pipeline parameters (ask if not provided):**
- Input: S3 prefix containing JPEG/PNG images from camera traps
- Model: MegaDetector v5 (general animal/person/vehicle) or DeepFaune (European species ID)?
- Batch size and deployment IDs available?
- Output: PostgreSQL table for detections + optional cropped species images to S3?
- Confidence thresholds for detection and classification?
- GPU available in processing environment?

**Step 2 — Pipeline architecture:**
```
S3 images/deployment-id/YYYY-MM/
    ↓ Trigger: S3 event or nightly scheduled scan
    ↓ SQS: one message per image (or per directory of 100 images)
    ↓ ECS worker / Lambda reads SQS
    ↓ Download image to /tmp (or stream via boto3)
    ↓ Run MegaDetector inference
    ↓ Filter by confidence threshold
    ↓ Write detections (bbox + class + confidence) to PostgreSQL
    ↓ Optionally crop and save species images to S3
    ↓ Delete SQS message on success
```

**Step 3 — Core implementation:**
```python
from megadetector.detection import run_detector
import boto3
import psycopg2
import tempfile
import os
from pathlib import Path

CONFIDENCE_THRESHOLD = 0.2
MD_MODEL_PATH = os.environ["MEGADETECTOR_MODEL_PATH"]  # local path or S3 key

def process_image(s3_bucket: str, s3_key: str, deployment_id: str, db_conn) -> int:
    """Download image, run MegaDetector, write detections. Returns detection count."""
    s3 = boto3.client("s3")

    with tempfile.NamedTemporaryFile(suffix=Path(s3_key).suffix, delete=True) as tmp:
        s3.download_file(s3_bucket, s3_key, tmp.name)

        # Run MegaDetector
        results = run_detector.load_and_run_detector(
            model_file=MD_MODEL_PATH,
            image_file_names=[tmp.name],
            threshold=CONFIDENCE_THRESHOLD,
        )

        detections = results["images"][0].get("detections", [])
        count = 0
        for det in detections:
            if det["conf"] >= CONFIDENCE_THRESHOLD:
                write_detection(db_conn, s3_key, deployment_id, det)
                count += 1

    return count

def write_detection(conn, s3_key: str, deployment_id: str, detection: dict):
    """Write a single detection to PostgreSQL with PostGIS bbox geometry."""
    # bbox is [x_min, y_min, width, height] normalized 0-1
    x, y, w, h = detection["bbox"]
    category = detection["category"]  # 1=animal, 2=person, 3=vehicle
    confidence = detection["conf"]

    with conn.cursor() as cur:
        cur.execute("""
            INSERT INTO camera_trap_detections
                (s3_key, deployment_id, category, confidence, bbox_normalized, detected_at)
            VALUES (%s, %s, %s, %s, ST_MakeEnvelope(%s, %s, %s, %s), NOW())
        """, (s3_key, deployment_id, category, confidence, x, y, x+w, y+h))
    conn.commit()
```

**Step 4 — PostgreSQL schema:**
```sql
CREATE TABLE camera_trap_detections (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    s3_key        TEXT NOT NULL,
    deployment_id TEXT NOT NULL,
    category      SMALLINT NOT NULL,  -- 1=animal, 2=person, 3=vehicle
    confidence    FLOAT NOT NULL,
    bbox_normalized GEOMETRY(POLYGON, 0),
    species_class TEXT,               -- filled by secondary classifier
    species_conf  FLOAT,
    detected_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    pipeline_ver  TEXT NOT NULL
);
CREATE INDEX ON camera_trap_detections (deployment_id, detected_at);
CREATE INDEX ON camera_trap_detections (category, confidence);
```

**Step 5 — Provide:**
- Complete processing script with SQS consumer loop
- PostgreSQL table DDL
- SQS message schema
- Environment variables needed (model path, DB URL, S3 bucket, thresholds)
- Dockerfile for the processing worker (includes model download)
