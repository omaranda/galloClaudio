---
name: eo-pipeline
description: Scaffold a complete Earth Observation (EO) data processing pipeline — satellite imagery (Sentinel-2, Landsat, Planet), spectral index calculation, COG conversion, and storage to S3/QuestDB/PostgreSQL.
---

You are acting as the eo-cv-specialist agent. Scaffold an EO data processing pipeline.

**Step 1 — Define pipeline parameters (ask if not provided):**
- Satellite source? (Sentinel-2, Landsat 8/9, Planet Labs)
- Input location? (S3 prefix, STAC API endpoint, local directory)
- Spectral indices to compute? (NDVI, NDWI, EVI, SAVI, land cover classification)
- Output format and destination? (COG to S3, time-series to QuestDB, polygons to PostgreSQL)
- Area of Interest? (bounding box, shapefile, GeoJSON)
- Temporal range?

**Step 2 — Pipeline architecture:**
```
S3 raw tiles
    ↓ S3 event → SQS queue
    ↓ Lambda / ECS worker reads SQS message
    ↓ Download tile (windowed read — no full load)
    ↓ Validate (CRS, nodata, pixel size, spatial extent)
    ↓ Reproject to target CRS if needed
    ↓ Calculate spectral indices
    ↓ Convert to COG → upload to S3 processed/
    ↓ Extract time-series stats → write to QuestDB
    ↓ Extract polygon stats (zonal) → write to PostgreSQL
    ↓ Mark SQS message as processed (delete)
```

**Step 3 — Core implementation pattern:**
```python
import rasterio
from rasterio.windows import from_bounds
import numpy as np
import boto3

def process_tile(s3_key: str, aoi_bounds: tuple, output_crs: str = "EPSG:4326") -> dict:
    """Process a single satellite tile from S3 using windowed reads."""
    s3_url = f"s3://{BUCKET}/{s3_key}"

    with rasterio.open(s3_url) as src:
        # Validate
        assert src.crs is not None, f"No CRS in {s3_key}"

        # Windowed read — only load the AOI extent
        aoi_window = from_bounds(*aoi_bounds, transform=src.transform)
        data = src.read(window=aoi_window)
        transform = src.window_transform(aoi_window)
        nodata = src.nodata

        # Mask nodata
        mask = (data == nodata) if nodata is not None else np.zeros(data.shape[1:], dtype=bool)

    return {"data": data, "transform": transform, "crs": src.crs, "mask": mask}

def calculate_ndvi(red: np.ndarray, nir: np.ndarray, nodata_mask: np.ndarray) -> np.ndarray:
    """NDVI = (NIR - Red) / (NIR + Red), clipped to [-1, 1]."""
    with np.errstate(divide="ignore", invalid="ignore"):
        ndvi = np.where(
            ~nodata_mask & ((nir + red) != 0),
            (nir.astype(float) - red) / (nir.astype(float) + red),
            np.nan
        )
    return np.clip(ndvi, -1, 1)
```

**Step 4 — COG output:**
```python
from rio_cogeo.cogeo import cog_translate
from rio_cogeo.profiles import cog_profiles

def write_cog_to_s3(data, profile, output_s3_key):
    """Write array as Cloud-Optimized GeoTIFF directly to S3."""
    cog_profile = cog_profiles.get("deflate")
    profile.update(cog_profile)
    with MemoryFile() as mem:
        with mem.open(**profile) as dst:
            dst.write(data)
        cog_translate(mem, output_s3_key, cog_profile, in_memory=True, quiet=True)
```

**Step 5 — Provide:**
- Complete pipeline script with CLI entry point
- SQS message schema for triggering the pipeline
- QuestDB/PostgreSQL table DDL for results
- S3 folder structure (raw / processed / curated)
- `requirements.txt` additions needed (rasterio, rio-cogeo, rioxarray, etc.)
- Performance estimates for typical data volumes
