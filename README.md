## Lifeline Microservices Backend

A microservices backend to support blood donation coordination in Nigeria.

### Quick start (one command)
Prereqs: Docker Desktop installed and running.

1) Copy your local env (optional; defaults work):
   - The stack will start Postgres and RabbitMQ with defaults. If you want to change DB creds, set these in your shell before starting:
     - `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`

2) Start the stack:
```
docker compose up -d --build
```

This brings up:
- Postgres (port 5432)
- RabbitMQ (ports 5672, 15672)
- Auth service (port 8000)
- Hospital service (port 8888)
- Blood Request service (port 8989)
- Notification service (port 8099)

Swagger UIs:
- Auth: http://localhost:8000/docs
- Hospital: http://localhost:8888/docs
- Requests: http://localhost:8989/docs
- Notifications: http://localhost:8099/docs

Gateway note:
- The Gateway currently calls services using hardcoded URLs like `http://0.0.0.0:8000`. Inside a Docker container, that won’t reach other containers. Until we switch Gateway to use env-based service URLs, run Gateway locally:
  - Open a terminal:
    - `cd gateway`
    - `python -m venv .venv && .\.venv\Scripts\activate`
    - `pip install -r requirements.txt`
    - `python main.py`
  - Then visit http://localhost:8088/docs

### Local development (without Docker)
You can also run each service directly with Python (requires Python 3.10+ and a local Postgres + RabbitMQ). Create a `.env` per service with database settings (example for `auth/.env`):
```
DB_TYPE=postgresql+psycopg2
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=lifeline
```

Then for each service:
- Create venv, install deps: `python -m venv .venv && .\.venv\Scripts\activate && pip install -r requirements.txt`
- Apply migrations (where available): `alembic upgrade head`
- Start service: `python main.py`

Ports by default:
- Auth: 8000
- Hospital: 8888
- Requests: 8989
- Gateway: 8088
- Notifications: 8099

### Architecture overview
- Auth: user auth/profile
- Hospital service: hospital CRUD and metadata (incl. lat/long)
- Blood request service: create/manage blood requests
- Matching service: locational matching utilities (Haversine)
- Notification service: listens to RabbitMQ and simulates notifications
- Gateway: HTTP aggregator that proxies to the other services

### Next steps recommended
- Switch Gateway to read service base URLs from environment variables
- Add `.env.example` files per service
- Add CI and tests
- Wire end-to-end events for request matching and notification delivery
