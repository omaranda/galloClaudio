---
name: s3-data-flow
description: Design or review S3 data flow architecture — bucket organization, data lake zones, event notifications, SQS triggers, lifecycle policies, and access patterns.
---

You are acting as the data-engineer agent. Design the S3 data flow architecture.

**Step 1 — Understand the data flow:**
- What types of data need to be stored? (raw satellite tiles, processed COGs, camera trap images, audio, model outputs, exports)
- What triggers processing? (upload event, scheduled batch, manual)
- What are the downstream consumers? (EO pipeline, camera trap processor, BirdNET, API, analytics)
- What are the data retention and compliance requirements?
- Estimated volume per month? (affects storage class and lifecycle decisions)

**Step 2 — Standard S3 data lake architecture:**

```
<project>-data-<env>/
├── raw/                          # Immutable source data — never modified
│   ├── satellite/
│   │   └── sentinel2/<tile>/<date>/
│   ├── camera-trap/
│   │   └── <deployment-id>/<YYYY-MM-DD>/
│   └── audio/
│       └── <station-id>/<YYYY-MM-DD>/
│
├── processed/                    # Validated, format-converted data
│   ├── satellite/
│   │   └── ndvi/<tile>/<date>/   # COG format
│   ├── camera-trap/
│   │   └── detections/<deployment-id>/
│   └── audio/
│       └── spectrograms/<station-id>/
│
├── curated/                      # Analysis-ready, aggregated datasets
│   ├── species-timeseries/
│   ├── habitat-maps/
│   └── exports/                  # Outputs for clients or APIs
│
└── models/                       # ML model artifacts
    └── <model-name>/<version>/
```

**Step 3 — S3 Event Notification → SQS wiring:**
```json
{
  "QueueConfigurations": [
    {
      "QueueArn": "arn:aws:sqs:<region>:<account>:camera-trap-processing",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {"Name": "prefix", "Value": "raw/camera-trap/"},
            {"Name": "suffix", "Value": ".jpg"}
          ]
        }
      }
    }
  ]
}
```

**Step 4 — S3 Lifecycle policies:**
```json
[
  {
    "ID": "raw-transition-to-glacier",
    "Filter": {"Prefix": "raw/satellite/"},
    "Status": "Enabled",
    "Transitions": [
      {"Days": 90,  "StorageClass": "STANDARD_IA"},
      {"Days": 365, "StorageClass": "GLACIER_IR"}
    ]
  },
  {
    "ID": "expire-temp-exports",
    "Filter": {"Prefix": "curated/exports/tmp/"},
    "Status": "Enabled",
    "Expiration": {"Days": 30}
  }
]
```

**Step 5 — Terraform for bucket with full config:**
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "${var.project_name}-data-${var.environment}"
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket                  = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

**Step 6 — Provide:**
- Full S3 folder structure with naming conventions
- S3 event → SQS wiring configuration
- Lifecycle policy recommendations with cost rationale
- Terraform bucket configuration
- Access pattern summary: who reads/writes each zone
