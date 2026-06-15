# 🏁 PITWALL — F1 OVERTAKE PREDICTOR

A decoupled, low-latency **real-time Formula 1 racing telemetry dashboard and predictive engine**. Powered by a multi-headed **Mixture-of-Experts (MoE)** machine learning framework on an asynchronous Python backend, it forecasts overtaking probabilities during live tracking battles.

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Next.js 14](https://img.shields.io/badge/Next.js-14-000000?logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-Retro%20Motorsport-38B2AC?logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)](https://redis.io/)

---

## What It Does

- **Real-Time Battle Tracking** — Continuously updates live racing wheel-to-wheel duels where the performance gap falls under 2.0 seconds.
- **Mixture-of-Experts Inference** — Breaks down aggregate predictive probabilities into dedicated sub-component expert system weights:
  - `Overtake Mode (OM)` — Tracks localized DRS availability and engine energy maps.
  - `ERS Boost` — Processes kinetic state deployment differentials between cars.
  - `Aero Delta` — Calculates downforce loss from aerodynamic clean/dirty air wake parameters.
- **Historical Replay Engine** — Persists comprehensive multi-lap execution records for post-race scrubbing and telemetry review.
- **Relational Telemetry Data Exports** — Grants authenticated clients structured CSV pipeline compilation endpoints for independent statistical evaluation.

---

## Technical Stack

- **Frontend:** Next.js 14 (App Router), Tailwind CSS, Framer Motion, TanStack React Query, Zustand.
- **Backend:** FastAPI (Python), SQLAlchemy 2.0 Async, AsyncPG, Pydantic v2.
- **Infrastructure / Data Layer:** PostgreSQL 15, Redis Cache, Docker & Docker Compose.

---

## System Architecture

```text
[ OpenF1 API / FastF1 Feed ]
             │
             ▼
┌─────────────────────────────┐
│   Background Data Ingestor  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│        FASTAPI ML INFERENCE ENGINE      │
│  ┌───────────────────────────────────┐  │
│  │     Mixture-of-Experts (MoE) Pool │  │  ┌─────────────────┐
│  │  ├── [OM]    LightGBM Classifier  │◀─┼──▶│  Redis Cache    │
│  │  ├── [BOOST] XGBoost Classifier   │  │  │ (Live Telemetry)│
│  │  └── [AERO]  LightGBM / XGBoost  │  │  └─────────────────┘
│  └─────────────────┬─────────────────┘  │
│                    ▼                    │
│    [Meta-Model Stacker  LogReg / MLP]   │
└────────────────────┬────────────────────┘
                     │
                     ▼
        ┌─────────────────────────────┐
        │      PostgreSQL Database    │
        │  (Persisted Historic Replays│
        └─────────────┬───────────────┘
                      │  (Async Web API)
                      ▼
        ┌─────────────────────────────┐
        │      NEXT.JS 14 WEB UI      │
        │    (Retro Telemetry Panel)  │
        └─────────────────────────────┘
```

---

## Core Operational API Gateways

| Method | Endpoint | Access Level | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/health` | Guest / Anonymous | Verifies underlying service state and model compilation indices. |
| `GET` | `/live-battles` | Guest / Anonymous | Yields transient real-time parameters for gaps < 2.0s. |
| `GET` | `/predict` | Registered / API User | Consumes real-time streams to execute active MoE matrix predictions. |
| `GET` | `/history/{race_id}` | Registered User (`user`) | Retrieves deep database records for full-session playback telemetry. |

---

## Database Architecture Blueprint

The persistent storage topology decouples volatile real-time prediction data streams from core user identities to preserve historical telemetry sets:

```text
┌───────────────┐        ┌────────────────┐        ┌─────────────────┐
│     users     │───────<│    sessions    │        │    circuits     │
├───────────────┤        ├────────────────┤        ├─────────────────┤
│ id (UUID, PK) │        │ id (UUID, PK)  │        │ id (VARCHAR, PK)│
└───────┬───────┘        │ user_id (FK)   │        └────────┬────────┘
        │                └────────────────┘                 │
        │                                                   ▼
        │                ┌────────────────┐        ┌─────────────────┐
        │                │    drivers     │        │     races       │
        │                ├────────────────┤        ├─────────────────┤
        │                │ id (VARCHAR,PK)│        │ id (VARCHAR, PK)│
        │                └───────┬────────┘        └────────┬────────┘
        │                        │                          │
        ▼                        ▼                          ▼
┌────────────────────────────────────────────────────────────────────┐
│                            predictions                             │
├────────────────────────────────────────────────────────────────────┤
│ id (BIGINT, PK)     race_id (FK)       attacker_id (FK)            │
│ defender_id (FK)    lap (INT)          p_overtake (FLOAT)          │
└────────────────────────────────────────────────────────────────────┘
```

---

## Getting Started

### Local Setup Requirements

Ensure **Docker Desktop** is running locally before deployment.

**1. Clone the repository and enter the workspace root:**

```bash
D:
cd your-repo-name
```

**2. Initialize the Next.js frontend scaffolding inside the monorepo:**

```bash
npx create-next-app@latest frontend --ts --tailwind --app --src-dir
```

**3. Orchestrate and start all system services:**

```bash
docker-compose up --build
```

**4. Verify connection status:**

- Frontend Dashboard:
- FastAPI Healthcheck Gateway: 

---

## 📂 Repository Structure

```
pitwall/
│
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── database.py       # Async SQLAlchemy session configuration
│   │   ├── models.py         # Declarative Postgres table schemas
│   │   └── main.py           # FastAPI lifecycle hooks & core routers
│   └── requirements.txt      # Core backend dependencies
│
├── frontend/
│   ├── src/
│   │   └── app/              # Next.js App Router workspace
│   ├── tailwind.config.ts    # Custom Retro Motorsport palette rules
│   └── package.json
│
└── docker-compose.yml        # Root architecture orchestration file
```

---

## Author
https://github.com/sharr-catalyst
