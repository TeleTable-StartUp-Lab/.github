# TeleTable

TeleTable is an end-to-end system for **autonomous indoor transport** in modern workplaces: a robot (“table”) that can be monitored in real time, driven manually when needed, and dispatched on named routes—backed by a secure backend and multiple client UIs.

## Mission

Make internal logistics in offices, labs, schools, and shared spaces **safer, faster, and more accessible** by providing a reliable autonomous transport robot with:

- clear real-time status,
- low-latency manual override,
- simple destination-based navigation,
- and a lightweight operational diary for traceability and team coordination.

## What TeleTable does (today)

### For operators

- **Live robot telemetry** (health, battery, current mode, position)
- **Manual driving** (low latency) with a **lock** to prevent conflicting control
- **Automatic navigation** between predefined **nodes** (e.g., “Home”, “Kitchen”, “Office”)
- **Route dispatching** (select start + destination)
- **Auth-protected access** (JWT) for user functions and admin-only user management
- **Project/operations diary** (CRUD) stored in PostgreSQL

### For the robot

- **Backend discovery** (UDP announce → backend learns robot base URL)
- **Telemetry + event reporting** (robot → backend via API key)
- **Command receiving** via WebSocket (backend → robot)

## Architecture (high-level)

TeleTable is split into four main parts:

- **Backend (Rust / Axum)** — API, auth, diary persistence, robot coordination, discovery, and WebSockets
- **Firmware / Robot runtime (ESP32 Arduino + simulator)** — robot HTTP endpoints, telemetry publishing, command execution
- **Web Frontend (React + Tailwind)** — browser dashboard for telemetry, control, diary
- **Mobile App (Flutter)** — joystick UI + route planning + diary UI (network integration currently stubbed/mocked in providers)

### Data flow

1. Robot announces itself via UDP → backend discovers robot URL
2. Robot sends telemetry/events to backend (`/table/state`, `/table/event`) using `X-Api-Key`
3. Clients fetch status from backend (`GET /status`) and operate using JWT-protected endpoints
4. Backend broadcasts robot commands over WebSocket (`/ws/robot/control`) to the robot
5. Manual driving is gated by a backend-enforced lock (`POST/DELETE /drive/lock`)

## Key functionality (backend API)

Backend runs on `http://localhost:3003` by default.

- Auth & users: `backend/docs/auth.md`
- Diary: `backend/docs/diary.md`
- Robot coordination (HTTP + WS + discovery): `backend/docs/robot.md`

## Repository layout

- `app/` — Flutter mobile app (UI + state management)
- `frontend/` — React web dashboard
- `backend/` — Rust service (Axum, SQLx, PostgreSQL, Redis)
- `firmware/` — ESP32 firmware project + Python robot simulator

## Quick start (local development)

### 1) Start backend (Docker)

From `backend/`:

```bash
docker compose up --build
```

This starts:

- PostgreSQL
- Redis
- Backend on `http://localhost:3003`

### 2) Start a robot simulator (recommended for dev)

From `firmware/`:

```bash
pip install -r requirements_robot.txt
python robot_simulator.py
```

The simulator will:

- expose a robot HTTP server (default `:8000`)
- announce itself via UDP to the backend (default UDP port `3001`)
- connect to the backend WebSocket and execute received commands

### 3) Start the web frontend

From `frontend/`:

```bash
npm install
npm start
```

The frontend uses `REACT_APP_API_URL` and defaults to `http://localhost:3003`.

### 4) (Optional) Run the Flutter app

From `app/`:

```bash
flutter pub get
flutter run
```

Note: the current Flutter providers contain mocked backend calls (login/diary/robot control). The UI is ready; wiring to the Rust backend can be done next.

## Configuration overview (backend)

Backend reads configuration from environment variables (see `backend/README.md` and `backend/.env.example`). Important ones:

- `DATABASE_URL` (required)
- `REDIS_URL` (required)
- `JWT_SECRET` (required)
- `SERVER_ADDRESS` (default `0.0.0.0:3003`)
- `ROBOT_API_KEY` (default `secret-robot-key`)
- UDP discovery listens on port `3001`

## Status and roadmap

TeleTable already includes:

- a working Rust backend with auth, diary, robot coordination, WS, and discovery
- a web frontend wired to the backend API URL
- firmware protocol documentation + a robot simulator

Next typical steps:

- connect the Flutter app providers to backend auth/diary/robot endpoints
- harden robot lock/timeout behavior end-to-end
- add richer route planning + map/node management UI

## Licenses

This monorepo contains multiple components; see the LICENSE files in each component directory:

- `backend/LICENSE`
- `firmware/LICENSE`
- `frontend/LICENSE`
