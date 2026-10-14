
# URBREATH Flooding Monitoring Tool

Provided by: **EXUS**

### Contact:
- Ettore Ettenzi: e.ettenzi@exus.ai
- Cristina Nichiforov: c.nichiforov@exus.ai

## Description

The URBREATH Flooding Monitoring Tool is a flood early-warning system for the Parma river basin (Emilia-Romagna, Italy). It is trained on hourly hydro-meteorological sensor data from Arpae-SIMC (water levels at S. Siro, Ponte Verdi and Ponte Nuovo, precipitation, humidity and temperature) and provides:

- **Flood alert prediction**: the probability that an official flood alert (Yellow/Orange/Red) will be active 6, 12 or 24 hours ahead (XGBoost classifier).
- **Water level forecasting**: predicted water level in metres at Ponte Verdi and Ponte Nuovo from 1 to 24 hours ahead, with graded severity against the official thresholds (XGBoost regressors).
- **Explainability**: the top drivers behind each prediction (SHAP).
- **Monitoring dashboard**: live station levels, forecast hydrographs, what-if rainfall scenarios, historical flood events (2016-2026) and model performance.
- **REST API**: every prediction is also available programmatically (OpenAPI/Swagger docs included).

The tool is distributed as a pre-built Docker image with the trained models included. No source code or build steps are needed.

## Installation Prerequisites

- Docker and Docker Compose installed and running (Docker Desktop on Windows/macOS)
- About 3 GB of free disk space
- Internet access, so the dashboard can show live sensor readings from the Emilia-Romagna alert portal (allertameteo.regione.emilia-romagna.it). If it can't reach the portal, it falls back to the bundled historical data.

## Installation Instructions

#### 1. Clone the Repository

```bash
git clone https://github.com/URBREATH/flooding-model.git
cd flooding-model
```

#### 2. Download and load the Docker image

Download `urbreath-flooding-model.tar.gz` from the [latest release](https://github.com/URBREATH/flooding-model/releases/latest) into the `flooding-model` folder, then load it:

```bash
docker load < urbreath-flooding-model.tar.gz

# Confirm it loaded
docker images urbreath-flooding-model
```

#### 3. Run the tool
```bash
docker compose up -d

# Check container status (urbreath-flooding-api should be "healthy")
docker compose ps

# Check logs
docker compose logs -f
```

With the older standalone Compose, use `docker-compose` instead of `docker compose`.

#### 4. Access the tool:

- Dashboard: `http://localhost:8501`
- API documentation (Swagger UI): `http://localhost:8001/docs`
- API health check: `http://localhost:8001/health`

#### 5. Stop the tool
```bash
docker compose down
```

## Troubleshooting

- **Port already in use**: change the left-hand port in the `ports:` section of `docker-compose.yml` (e.g. `"18501:8501"`).
- **Dashboard shows a "placeholder data" warning**: the live feed is unreachable, so the dashboard is using the bundled historical readings instead. Predictions are still real model outputs.
- **Dashboard doesn't start**: it waits for the API to become healthy. Check `docker compose logs api`.
- **"image not found"**: re-run step 2 and confirm `urbreath-flooding-model:latest` appears in `docker images`.

## Built Image

- Distribution: GitHub release asset ([latest release](https://github.com/URBREATH/flooding-model/releases/latest))
- Image Name: `urbreath-flooding-model:latest`
- Version: `v1.0.0`

## License
This tool is licensed under Proprietary Software Licence - End-User License Agreement (EULA).
