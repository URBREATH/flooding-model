
# URBREATH Flooding Monitoring Tool

Provided by: **EXUS**

### Contact:
- Ettore Ettenzi: e.ettenzi@exus.ai
- Cristina Nichiforov: c.nichiforov@exus.ai

## Description

The URBREATH Flooding Monitoring Tool is a flood early-warning system for the Parma river basin (Emilia-Romagna, Italy). It is trained on hourly hydro-meteorological sensor data from Arpae-SIMC (water levels at S. Siro, Ponte Verdi and Ponte Nuovo, precipitation, humidity and temperature) and provides:

- **Flood alert prediction**: the probability that an official flood alert (Yellow/Orange/Red) will be active 6, 12 or 24 hours ahead (XGBoost classifier).
- **Water level forecasting**: predicted water level in metres at Ponte Verdi and Ponte Nuovo from 1 to 24 hours ahead, with graded severity against the official thresholds (XGBoost regressors).
- **Upstream monitoring**: the observed severity at S. Siro against its own official thresholds.
- **Explainability**: the top drivers behind each prediction (SHAP).
- **Monitoring dashboard**: live station levels and severity on a map, forecast hydrographs, what-if rainfall scenarios, historical flood events (2016-2026) and model performance.
- **REST API**: every prediction is also available programmatically (OpenAPI/Swagger docs included).

The tool is distributed as a pre-built Docker image with the trained models included. No source code or build steps are needed.

Official alert thresholds used (metres above each gauge's own datum, source Arpae-SIMC):

| Station | Yellow | Orange | Red |
|---|---|---|---|
| Ponte Verdi | 1.80 m | 2.15 m | 3.00 m |
| Ponte Nuovo | 1.20 m | 2.00 m | 3.00 m |
| S. Siro | 9.20 m | 10.70 m | 11.70 m |

## Installation Prerequisites

- Docker and Docker Compose installed and running (Docker Desktop on Windows/macOS). Check with `docker compose version`.
- An x86-64 (amd64) machine with about 3 GB of free disk space and 2 GB of RAM.
- **Access to the image.** The image is hosted privately on GitHub Container Registry. Ask EXUS (see Contact) to grant your GitHub account read access to the `urbreath-flooding-model` package.
- Internet access to:
  - `ghcr.io`, to download the image;
  - `allertameteo.regione.emilia-romagna.it`, so the dashboard can show live sensor readings. If it can't reach the portal, it falls back to the bundled historical data.

## Installation Instructions

#### 1. Clone the Repository

```bash
git clone https://github.com/URBREATH/flooding-model.git
cd flooding-model
```

#### 2. Log in to GitHub Container Registry (once per machine)

Create a GitHub personal access token **(classic)** with only the **`read:packages`** scope: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic). Copy the token (it starts with `ghp_`) when GitHub shows it; it is shown only once.

Then log in, pasting the token when asked for a password:

```bash
docker login ghcr.io -u <your-github-username>
```

You should see `Login Succeeded`. On Git Bash for Windows, use `winpty docker login ghcr.io -u <your-github-username>`.

#### 3. Configure

Open `.env` in the `flooding-model` folder and set an API key, which protects the tool's maintenance endpoints (`/pipeline/*`). Generate a random value:

```bash
openssl rand -hex 32
```

and paste it after `URBREATH_PIPELINE_API_KEY=`. The tool will not start while it is empty. The other settings can stay at their defaults:

| Variable | Default | Purpose |
|---|---|---|
| `URBREATH_VERSION` | `1.1.0` | Image version to run. |
| `URBREATH_BIND_ADDRESS` | `0.0.0.0` | Host interface for the published ports. Use `127.0.0.1` when a reverse proxy on the same host fronts the tool. |
| `URBREATH_API_PORT` | `8001` | Host port for the API. |
| `URBREATH_UI_PORT` | `8501` | Host port for the dashboard. |
| `URBREATH_PIPELINE_API_KEY` | (empty, required) | Secret for `/pipeline/*`, sent as the `X-API-Key` header. |
| `HTTPS_PROXY` / `NO_PROXY` | (empty) | Outbound proxy, if your network needs one for the live feed. |

#### 4. Run the tool
```bash
docker compose up -d

# Check container status (urbreath-flooding-api should be "healthy")
docker compose ps

# Check logs
docker compose logs -f
```

The first start downloads the image (about 0.9 GB). With the older standalone Compose, use `docker-compose` instead of `docker compose`.

#### 5. Access the tool:

- Dashboard: `http://localhost:8501`
- API documentation (Swagger UI): `http://localhost:8001/docs`
- API health check: `http://localhost:8001/health` should return `{"status":"ok","model_loaded":true,"model_tag":"6h"}`

Replace `localhost` with the machine's name or address when connecting from another computer.

#### 6. Stop the tool
```bash
docker compose down
```

#### Updating to a new version

Set `URBREATH_VERSION` in `.env` to the new version, then:

```bash
docker compose pull
docker compose up -d
```

**Coming from v1.0.0** (the image loaded from `urbreath-flooding-model.tar.gz`): run `docker compose down` with your old `docker-compose.yml` first, then follow steps 2 to 4. The old local image can be removed with `docker rmi urbreath-flooding-model:latest`.

## Security Notes

- The dashboard and the prediction endpoints have no login. Publish them only on a trusted network, or put them behind a reverse proxy that adds TLS and authentication (and set `URBREATH_BIND_ADDRESS=127.0.0.1`).
- `/pipeline/*` requires the `X-API-Key` header. It is not needed for normal operation; the key keeps it closed.
- The containers run as an unprivileged user and keep no state: stopping or recreating them loses nothing.

## Troubleshooting

- **`required variable URBREATH_PIPELINE_API_KEY is missing a value`**: set the key in `.env` (step 3).
- **`denied`, `unauthorized` or `not found` when downloading the image**: you are not logged in, or your account has no access yet. Repeat step 2 and check with EXUS that access was granted. The token must be a classic token with `read:packages`.
- **Port already in use**: change `URBREATH_API_PORT` or `URBREATH_UI_PORT` in `.env`.
- **Dashboard shows a "Live feed unavailable" warning**: the live feed is unreachable, so the dashboard is using the bundled historical readings instead. Predictions are still real model outputs. Check the firewall, or set `HTTPS_PROXY` in `.env`.
- **Dashboard doesn't start**: it waits for the API to become healthy. Check `docker compose logs api`.
- **`exec format error`**: the machine is not x86-64. The image is built for amd64 only.

## Built Image

- Distribution: GitHub Container Registry (private, access on request)
- Image Name: `ghcr.io/exus-ai-labs/urbreath-flooding-model`
- Version: `1.1.0`

## License
This tool is licensed under Proprietary Software Licence - End-User License Agreement (EULA).
