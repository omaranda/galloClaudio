---
name: docker-setup
description: Generate or review Dockerfile and Docker Compose configuration — Django API, data processing workers, EO pipeline containers.
---

You are acting as the devops-aws agent. Generate or review Docker configuration.

**Step 1 — Gather context:**
- Which service? (Django API, FastAPI, EO processor, data pipeline worker, etc.)
- Is this for local development, CI testing, or production on ECS/EC2?
- Does it need GDAL/rasterio? (requires special base image)
- Python version? (default: 3.11)
- Any GPU requirements?

**Step 2 — Read existing code:**
Check for existing `Dockerfile`, `docker-compose.yml`, `requirements.txt` / `Pipfile` / `pyproject.toml`.

**Step 3 — Generate Dockerfile (multi-stage build):**

**Standard Django/FastAPI API:**
```dockerfile
# Stage 1: build deps
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: runtime
FROM python:3.11-slim AS runtime
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

**EO/GDAL processing container:**
```dockerfile
FROM osgeo/gdal:ubuntu-small-3.6.3 AS base
RUN apt-get update && apt-get install -y python3-pip python3-venv && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python3", "-m", "processor.main"]
```

**Step 4 — Generate docker-compose.yml for local development:**
Include:
- Application service with volume mounts for hot reload
- PostgreSQL with PostGIS extension (`postgis/postgis:15-3.3`)
- QuestDB if needed (`questdb/questdb:latest`)
- LocalStack for AWS service simulation (S3, SQS) if needed
- Environment variables loaded from `.env` file (never hardcoded)

**Step 5 — Best practices checklist:**
- [ ] Multi-stage build — dev dependencies not in final image
- [ ] Non-root user in final stage
- [ ] `.dockerignore` excludes `.git`, `__pycache__`, `.env`, `node_modules`
- [ ] No secrets in Dockerfile or image layers
- [ ] Health check defined (`HEALTHCHECK CMD curl -f http://localhost:8000/health/`)
- [ ] Pinned base image tags — no `latest` in production Dockerfiles

Provide Dockerfile, docker-compose.yml, and .dockerignore contents.
