# Crowd Monitoring and Loitering Detection Pipeline

## Project Overview
This project implements a real-time surveillance system using artificial intelligence to detect, track, count people crossing through geofenced areas, and detect loitering behavior. The system combines the power of YOLOv8 (object detection) and ByteTrack algorithm (multi-object tracking) with velocity and region-based logic to detect anomalies.

### Enhanced Features & Modularity
- **Object Tracking:** Uses YOLOv8 and ByteTrack to detect and assign stable IDs to each person.
- **Geofencing (Virtual Fence):** Flexibly count people in geographic areas (polygons).
- **Loitering Detection:** Detect people staying too long or remaining in an area with extremely low movement speed based on time thresholds.
- **Modular Design:** Easy to maintain, develop, and run individual functions through command-line structure in Terminal.

## System Requirements
- Python 3.8+
- Webcam or IP camera (for real-time video)
- RAM: 4GB+ (8GB recommended)
- GPU: Optional (for higher performance)

## Installation

### 1. Clone Repository
```bash
https://github.com/TLio2407/AI-Powered-Smart-Home-Surveillance-and-Crowd-Management-System.git```

### 2. Create Virtual Environment
```bash
python3 -m venv venv
# On Windows:
venv\Scripts\activate
# On Linux/Mac:
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

## Usage Guide

The main source code of the system is located in the `BytetrackCountingLoitering/` folder and the web backend is located in the `web/` folder.

To start the system (Redis, backend and Celery workers), run the following commands on 4 separate terminals:

Run:

```bash
# Terminal 1 — Redis:
docker compose -f web\docker-compose.yml up -d redis

# Terminal 2 — Backend:
python -m uvicorn web.app.main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 3 — Celery (stream_queue):
python -m celery -A web.app.core.celery_app worker --loglevel=INFO --pool=solo -Q stream_queue

# Terminal 4 — Celery (alert/notification/hardware/stats queues):
python -m celery -A web.app.core.celery_app worker --loglevel=INFO --pool=solo -Q alert_queue,notification_queue,hardware_queue,stats_queue
```

Notes:
- Run Redis before starting backend and workers.
- On Windows, keep terminals open to view logs; use PowerShell or CMD with virtual environment enabled if needed.

## Project Structure
Summary of the main folder structure in the repository (only important items listed):

```
.
├── BytetrackCountingLoitering/    # Tracking & loitering pipeline (core)
├── CrowdCounting/                  # Experimental counting modules
├── web/                            # Web backend, API, Celery tasks, frontend assets
│   ├── app/
│   ├── frontend/
│   └── docker-compose.yml
├── requirements.txt
├── README.md
└── yolov8s.pt                       # Sample YOLOv8 model
```

The `web/` folder contains FastAPI backend, Celery worker, Docker configuration and static frontend (`web/frontend/`).

Detailed structure inside `web/app/`:

```
web/app/
├── __init__.py
├── main.py                  # FastAPI app entrypoint
├── adapters/                # Adapters for AI, camera, hardware, notifier
│   ├── ai/                  # AI implementations (factory, mock, model wrappers)
│   ├── camera/              # Camera adapters (video file, mock, hardware clients)
│   ├── hardware/            # Hardware device communication
│   └── notifier/            # Notifier implementations (e.g., Telegram)
├── api/                     # FastAPI routes (routes_*.py)
├── core/                    # Core app: `celery_app.py`, `config.py`, `database.py`, `redis.py`
├── models/                  # ORM/DB models (e.g., SQLAlchemy)
├── schemas/                 # Pydantic schemas for request/response
├── services/                # Business logic and service layer
├── tasks/                   # Celery tasks (alert_tasks, stream_tasks, stats_tasks,...)
├── ws/                      # WebSocket routes/handlers
└── static/                  # Static assets: `app.js`, `style.css`, snapshots/
```

Quick editing/observation tips:
- API Entrypoint: `web/app/main.py` (initializes FastAPI + mounts routers).
- Celery Configuration: `web/app/core/celery_app.py` and tasks in `web/app/tasks/`.
- Add/adjust environment variables in `web/.env` and `web/.env.example`.
- To extend adapters or add new cameras, start from `web/app/adapters/camera/`.

## Important Notes
- **Docker & Redis:** The file `web/docker-compose.yml` contains the `redis` service — start Redis before running backend/celery (see commands in the Usage Guide section).
- **Environment Variables:** Copy `web/.env.example` to `web/.env` and adjust if needed before starting services.
- **Running on Windows:** Use PowerShell/CMD with virtual environment activated or Docker Desktop; keep terminals open to view logs.
- **Celery:** Celery uses Redis as a broker (by default) — ensure Redis is reachable from workers; queues are separated by task (stream_queue, alert_queue, ...).
- **Customize Parameters:** `BytetrackCountingLoitering/config.py` contains thresholds, polygon coordinates and loitering parameters.

## Development Roadmap (To-do)
- **1. Dockerize full stack (high):** create images for backend, worker and deployment instructions (compose/stack).
- **2. API/Streaming (high):** add APIs returning stream results in real-time and websockets for frontend.
- **3. Automation & CI (medium):** unit tests for pipeline, linting, basic CI/CD pipeline.
- **4. Model Management (medium):** add scripts to download/check model versions, storage for checkpoints.
- **5. Improve Detection (low):** add fall detection logic, crowd run detection, optimize parameters based on real data.

## License
The project uses mostly open-source libraries distributed freely under YOLOv8 and other open Python library standards. Please comply with the copyrights of third-party libraries being applied.
