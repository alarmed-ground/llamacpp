# LlamaCPP with Monitoring Stack

This repository contains a Docker Compose setup to run LlamaCPP, OpenWebUI, Grafana, Prometheus, and DCGM Exporter.

## Prerequisites

- Docker and Docker Compose installed
- NVIDIA GPU (for DCGM and GPU-accelerated LlamaCPP)
- NVIDIA Container Toolkit installed

## Setup

1. Clone or use this repository.

2. Place your Llama model file (e.g., `your-model.gguf`) in the `models/` directory.

3. Update the `command` in `docker-compose.yml` for LlamaCPP to point to your model file.

4. Run `docker-compose up -d` to start the services.

## Services

- **LlamaCPP**: Runs the Llama model server on port 8080.
- **OpenWebUI**: Web UI for interacting with the model on port 3000.
- **Unsloth Studio**: Fine-tuning interface on port 3002.
- **Grafana**: Monitoring dashboard on port 3001 (admin/admin).
- **Prometheus**: Metrics collection on port 9090.
- **DCGM Exporter**: GPU metrics exporter on port 9400.

## Accessing

- OpenWebUI: http://localhost:3000
- Unsloth Studio: http://localhost:3002
- Grafana: http://localhost:3001
- Prometheus: http://localhost:9090

## Notes

- Change the default Grafana password in production.
- Ensure the model path in LlamaCPP command is correct.
- For GPU support, the runtime is set to nvidia.