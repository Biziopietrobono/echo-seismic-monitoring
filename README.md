# E.C.H.O.: Earthquake & Conflict Hazard Observer

A distributed, fault-tolerant platform that classifies seismic events in real time. It reads 20 Hz streams from simulated seismic sensors, runs an FFT over a sliding window for each sensor and labels each event from its dominant frequency: earthquake (0.5 to 3 Hz), conventional explosion (3 to 8 Hz) or nuclear-like event (8 Hz and above).

Group project (5 people) for Advanced Programming, MSc in Engineering in Computer Science and AI, Sapienza University of Rome (2025/26).

> Portfolio copy of the team's official repository [enoughpaladin00/2003424_ECHO](https://github.com/enoughpaladin00/2003424_ECHO), with full commit history.

## My role

I built the ingestion broker (`source/broker`): sensor discovery, one ingestion loop per sensor, reconnection with exponential backoff, a dead-letter queue for invalid messages and fan-out to all active processing replicas. I also worked on the persistence API (`source/backend`) that stores the events in PostgreSQL.

## Architecture

![Architecture](booklets/ECHO_Architecture_diagrams.jpeg)

Seven containerised services on one Docker network:

| Service | Role | Stack |
|---|---|---|
| `ingestion-broker` | Receives the sensor streams from the simulator and fans them out to the replicas | Python, websockets, aiohttp |
| `processing-replica` (×3) | Keeps a 5 s window per sensor (100 samples), computes the FFT, classifies the event and scores its severity | Python, FastAPI, NumPy |
| `backend-api` | Stores events idempotently, serves the history and pushes new events to the dashboard | FastAPI, asyncpg, Pydantic |
| `database` | Event storage, indexed by time and location | PostgreSQL 15 |
| `dashboard` | Live event feed, alerts and sensor map | React, Leaflet |
| `api-gateway` | Single entry point that routes `/api` and `/ws` traffic | Nginx |
| `seismic-simulator` | External sensor simulator provided by the course | |

**No duplicate events.** All three replicas see the same signal, so they detect the same event. Instead of coordinating them, each replica builds the event ID deterministically from sensor, event type and a timestamp rounded to 10-second buckets, and the database keeps the first insert (`ON CONFLICT DO NOTHING`).

**Fault tolerance.** The simulator can shut down a replica at any time (via an SSE control stream); the other replicas keep processing. The broker reconnects to the simulator on its own if the connection drops, and malformed measurements go to a dead-letter queue instead of reaching the replicas.

## Running it

The simulator image is provided by the course and is not included in this repository.

1. Put `seismic-signal-simulator-oci.tar` in `source/`
2. From `source/`, run `./start.sh` (it loads the simulator image and runs `docker compose up -d --build`)
3. Open the dashboard at http://localhost

Unit tests for the processing logic are in `source/tests` and run on every push through GitHub Actions.

## Repository layout

| Path | Content |
|---|---|
| `source/broker` | Ingestion broker |
| `source/processing-engine` | FFT and classification replicas |
| `source/backend` | Persistence and real-time API |
| `source/frontend` | React dashboard |
| `source/gateway` | Nginx configuration |
| `source/tests` | Unit tests |
| `Student_doc.md` | Full design document: user stories, containers, endpoints |
| `booklets/` | Architecture diagram and presentation slides |

## Known limitations

Authentication on the dashboard is mocked client-side, and the database credentials in `docker-compose.yml` are for local development only.

## Team

Fabiano Cacioli · Jacopo Rossi · Fabrizio Pietrobono · Emanuele Smisi · Luca Buonomini
