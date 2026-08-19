# Smart Energy Management & Grid Balancing Platform

A production-ready, interactive web platform addressing Gujarat's solar-heavy grid challenges — featuring real-time telemetry, multi-agent AI dispatch simulation, BESS management, and a visual grid topology map.

---

## Author

| | |
|---|---|
| **Name**   | Mihir Parmar |
| **Email**  | [mihirparmar1230@gmail.com](mailto:mihirparmar1230@gmail.com) |
| **GitHub** | [@mihirparmar0007](https://github.com/mihirparmar0007) |

---

## Architecture Overview

```
smart-grid-platform/
├── backend/                    # FastAPI simulation engine
│   ├── app/
│   │   ├── main.py             # FastAPI app, WebSocket endpoints, REST API
│   │   ├── simulation/
│   │   │   └── engine.py       # Solar/demand/BESS mock telemetry generator
│   │   └── agents/
│   │       └── engine.py       # 5-agent AI reasoning engine
│   └── requirements.txt
│
└── frontend/                   # Next.js 14 App Router frontend
    └── src/
        ├── app/
        │   ├── layout.tsx       # Root layout + GridProvider
        │   ├── page.tsx         # Main page with sidebar routing
        │   └── globals.css      # Dark mode control-room theme
        ├── components/
        │   ├── dashboard/       # Executive Command Dashboard
        │   │   ├── DashboardView.tsx
        │   │   ├── KpiBar.tsx              # 5 KPI cards with live data
        │   │   ├── LoadGenerationChart.tsx # Recharts area/line chart
        │   │   └── DuckCurveBanner.tsx     # Evening ramp alert
        │   ├── agents/          # Multi-Agent AI Control Center
        │   │   ├── AgentsView.tsx
        │   │   ├── AgentMatrix.tsx         # 5-agent status matrix
        │   │   └── ActionLogTerminal.tsx   # Terminal-style action stream
        │   ├── bess/            # BESS & Storage Management
        │   │   └── BessView.tsx            # SoC gauge, telemetry, override
        │   ├── topology/        # Outage Risk & Geo-Topology
        │   │   └── TopologyView.tsx        # SVG substation map + fault sim
        │   └── ui/
        │       └── Sidebar.tsx             # Navigation sidebar
        ├── lib/
        │   ├── GridContext.tsx   # WebSocket state + control API
        │   └── utils.ts         # Formatting helpers
        └── types/
            └── index.ts          # Shared TypeScript types
```

---

## Prerequisites

| Tool      | Version     |
|-----------|-------------|
| Node.js   | 18+ (LTS)   |
| Python    | 3.11+       |
| pip       | latest      |
| npm / pnpm | latest     |

---

## Quick Start

### 1 — Backend (FastAPI)

```bash
cd smart-grid-platform/backend

# Create and activate virtual environment
python -m venv .venv

# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

The API will be live at `http://localhost:8000`.
Interactive docs available at `http://localhost:8000/docs`.

---

### 2 — Frontend (Next.js)

```bash
cd smart-grid-platform/frontend

# Install dependencies
npm install
# or: pnpm install

# Run development server
npm run dev
```

Open `http://localhost:3000` in your browser.

---

## Environment Variables (Optional)

Create `frontend/.env.local` to override API URLs:

```env
NEXT_PUBLIC_WS_TELEMETRY=ws://localhost:8000/ws/telemetry
NEXT_PUBLIC_WS_AGENTS=ws://localhost:8000/ws/agents
NEXT_PUBLIC_REST_BASE=http://localhost:8000
```

---

## API Reference

| Method | Endpoint               | Description                                  |
|--------|------------------------|----------------------------------------------|
| GET    | `/api/history`         | 60-point synthetic history for chart seed    |
| GET    | `/api/snapshot`        | Single telemetry + agent reasoning snapshot  |
| POST   | `/api/fault/{mode}`    | Inject fault: `cloud_cover`, `load_spike`, `feeder_trip`, `clear` |
| POST   | `/api/bess/override`   | Body: `{"enabled": true, "setpoint_mw": 150}` |
| WS     | `/ws/telemetry`        | Live telemetry stream (2s interval)          |
| WS     | `/ws/agents`           | Live agent action log stream (3s interval)   |

### WebSocket Control (via `/ws/telemetry`)

Send JSON messages mid-stream to inject faults or change BESS mode:

```json
{ "fault": "cloud_cover" }
{ "fault": "clear" }
{ "manual_override": true, "setpoint_mw": 150 }
```

---

## Feature Guide

### Executive Command Dashboard
- **5 KPI cards**: Grid Load, Solar Generation, Net Imbalance ΔP, BESS Reserve, Grid Frequency
- **Real-time chart**: Solar area + Demand area + BESS line + Frequency line (dual Y-axis, Recharts)
- **Duck Curve Banner**: Auto-appears 17:00–21:00 IST when solar < 25% of demand
- **Fault Injection Buttons**: Trigger cloud cover, load spike, or feeder trip from the header

### Multi-Agent AI Control Center
- **Agent Status Matrix**: Cards for all 5 agents with live status badges (OK / WARN / CRITICAL), reasoning text, confidence bar, and last metric
- **Action Log Terminal**: Terminal-style scrolling feed of autonomous agent decisions with color-coded severity

### BESS & Storage Management
- **SoC Radial Gauge**: Color-coded (green / amber / red) with animated needle
- **Full telemetry table**: Cell temperature, degradation index, cycle count, efficiency
- **Manual Override Toggle**: Bypass AI agents and set custom dispatch setpoint via slider (−250 → +250 MW)
- **SoC History Chart**: 60-point area chart of battery level over time

### Outage Risk & Geo-Topology Map
- **SVG Gujarat Substation Map**: 12 substations projected to real lat/lng coordinates
- **Stress visualization**: Green/amber/red nodes with pulsing animation for critical sites
- **Feeder lines**: Connect nearby substations with stress-colored edges
- **Fault simulation controls**: Dedicated buttons with self-healing demonstration

---

## Simulation Engine Details

| Parameter         | Model                                        |
|-------------------|----------------------------------------------|
| Solar irradiance  | Gaussian bell curve, peak 13:00 IST, 1200 W/m² max |
| Consumer demand   | Dual-peak (08:30, 20:00), 2200 MW base load  |
| BESS dispatch     | Proportional droop control on imbalance      |
| Grid frequency    | 50 Hz ± 0.5 Hz droop per 1000 MW imbalance  |
| Solar capacity    | 5040 MW (Gujarat rooftop + utility scale)    |
| BESS capacity     | 500 MWh / 250 MW                             |
| Telemetry interval| 2 seconds (WebSocket), 3 seconds (agents)    |

### Fault Modes
| Fault         | Effect                                                                  |
|---------------|-------------------------------------------------------------------------|
| `cloud_cover` | Solar drops to 10–35% of baseline; Solar Agent raises confidence interval |
| `load_spike`  | Demand surges 18–35%; Load Agent flags emergency reserves               |
| `feeder_trip` | Artificial 300–600 MW load imbalance; G-12 / G-07 stress spikes to critical |

---

## Production Deployment Notes

- **Backend**: Use `gunicorn -k uvicorn.workers.UvicornWorker` behind nginx, or deploy to any container platform (Docker, Cloud Run, etc.)
- **Frontend**: `npm run build && npm start` or deploy to Vercel / Netlify
- Replace mock simulation engine with real SCADA/DERMS integration at `app/simulation/engine.py`
- Add authentication middleware (JWT/OAuth2) to FastAPI before exposing to production

---

## Tech Stack

| Layer       | Technology                        |
|-------------|-----------------------------------|
| Frontend    | Next.js 14, React 18, TypeScript  |
| Styling     | Tailwind CSS (custom dark theme)  |
| Charts      | Recharts (ComposedChart, AreaChart, RadialBarChart) |
| Icons       | Lucide React                      |
| State       | React Context + native WebSocket  |
| Backend     | FastAPI (Python 3.11)             |
| Transport   | WebSocket (telemetry + agents)    |
| Simulation  | Pure Python in-memory engine      |
