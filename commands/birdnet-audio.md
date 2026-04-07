---
name: birdnet-audio
description: Scaffold a BirdNET audio processing pipeline — S3 audio input, BirdNET-Analyzer Python API, SQS batch processing, and PostgreSQL detection storage.
---

You are acting as the eo-cv-specialist agent. Set up a BirdNET audio processing pipeline.

**Step 1 — Define pipeline parameters (ask if not provided):**
- S3 bucket and prefix containing WAV/MP3/FLAC recordings?
- Recording metadata available? (recorder ID, GPS coordinates, deployment start/end)
- Confidence threshold for storing detections? (default: 0.7)
- Temporal resolution? (full 3-second BirdNET segments or per-minute aggregation)
- Target location for filtering species predictions? (lat/lon for species filter)
- Processing environment: Lambda (< 512MB audio) or ECS worker (large files)?

**Step 2 — Pipeline architecture:**
```
S3 audio/<station-id>/YYYY-MM-DD/
    ↓ Nightly S3 scan or S3 event trigger
    ↓ SQS: one message per audio file
    ↓ ECS worker reads SQS message
    ↓ Download WAV to /tmp
    ↓ Run BirdNET-Analyzer Python API
    ↓ Filter detections by confidence threshold
    ↓ Write to PostgreSQL (audio_detections table)
    ↓ Delete SQS message on success
```

**Step 3 — Core implementation:**
```python
import os
import boto3
import psycopg2
import tempfile
from pathlib import Path
from birdnet_analyzer import predict  # BirdNET-Analyzer Python API

CONFIDENCE_THRESHOLD = float(os.environ.get("BIRDNET_THRESHOLD", "0.7"))
S3_BUCKET = os.environ["AUDIO_S3_BUCKET"]

def process_audio_file(s3_key: str, station_lat: float, station_lon: float, db_conn) -> int:
    """Download audio from S3, run BirdNET, store detections. Returns detection count."""
    s3 = boto3.client("s3")

    with tempfile.NamedTemporaryFile(suffix=Path(s3_key).suffix, delete=True) as tmp:
        s3.download_file(S3_BUCKET, s3_key, tmp.name)

        # BirdNET-Analyzer Python API
        detections = predict.predict_species_within_audio_file(
            audio_file=tmp.name,
            lat=station_lat,
            lon=station_lon,
            week=-1,         # -1 = use all weeks (no seasonal filter)
            sensitivity=1.0,
            min_conf=CONFIDENCE_THRESHOLD,
        )

    count = 0
    for detection in detections:
        start_sec, end_sec = detection["start_time"], detection["end_time"]
        species_code = detection["scientific_name"]
        common_name = detection["common_name"]
        confidence = detection["confidence"]

        write_detection(db_conn, s3_key, station_lat, station_lon,
                        start_sec, end_sec, species_code, common_name, confidence)
        count += 1

    return count

def write_detection(conn, s3_key, lat, lon, start_sec, end_sec,
                    species_code, common_name, confidence):
    """Store a BirdNET detection in PostgreSQL."""
    with conn.cursor() as cur:
        cur.execute("""
            INSERT INTO audio_detections
                (s3_key, station_location, start_sec, end_sec,
                 species_code, common_name, confidence, detected_at, pipeline_ver)
            VALUES (%s, ST_MakePoint(%s, %s)::GEOGRAPHY, %s, %s, %s, %s, %s, NOW(), %s)
        """, (s3_key, lon, lat, start_sec, end_sec,
              species_code, common_name, confidence, "birdnet-2.4"))
    conn.commit()
```

**Step 4 — PostgreSQL schema:**
```sql
CREATE TABLE audio_detections (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    s3_key          TEXT NOT NULL,
    station_location GEOGRAPHY(POINT, 4326),
    start_sec       FLOAT NOT NULL,
    end_sec         FLOAT NOT NULL,
    species_code    TEXT NOT NULL,
    common_name     TEXT NOT NULL,
    confidence      FLOAT NOT NULL,
    detected_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    pipeline_ver    TEXT NOT NULL
);
CREATE INDEX ON audio_detections (species_code, detected_at);
CREATE INDEX ON audio_detections USING GIST (station_location);
CREATE INDEX ON audio_detections (s3_key);
```

**Step 5 — SQS consumer loop:**
```python
def sqs_consumer_loop():
    """Long-running SQS consumer for audio processing."""
    sqs = boto3.client("sqs")
    queue_url = os.environ["SQS_QUEUE_URL"]
    db_conn = psycopg2.connect(os.environ["DATABASE_URL"])

    while True:
        response = sqs.receive_message(
            QueueUrl=queue_url,
            MaxNumberOfMessages=1,
            WaitTimeSeconds=20,      # long polling
            VisibilityTimeout=300,   # 5 min processing window
        )
        for message in response.get("Messages", []):
            body = json.loads(message["Body"])
            try:
                count = process_audio_file(body["s3_key"], body["lat"], body["lon"], db_conn)
                print(json.dumps({"status": "ok", "s3_key": body["s3_key"], "detections": count}))
                sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=message["ReceiptHandle"])
            except Exception as e:
                print(json.dumps({"status": "error", "s3_key": body["s3_key"], "error": str(e)}))
                # Do not delete — message returns to queue after visibility timeout
```

**Step 6 — Provide:**
- Complete script with SQS consumer and processing logic
- PostgreSQL DDL
- SQS message schema `{"s3_key": "...", "lat": 0.0, "lon": 0.0, "station_id": "..."}`
- Required environment variables
- `requirements.txt` additions: `birdnet-analyzer`, `psycopg2-binary`, `boto3`
