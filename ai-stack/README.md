# Local AI Stack – Setup Guide

## Services

| Service | Port | Purpose |
|---|---|---|
| **Open WebUI** | 3000 | Main chat UI |
| **Ollama** | 11434 | Primary LLM runner |
| **llama.cpp server** | 8081 | Optional second backend (GGUF models) |
| **SearXNG** | 8080 | Web search engine |
| **openedai-speech** | 8000 | Local text-to-speech |
| **Redis** | — | SearXNG cache (internal only) |

---

## Prerequisites

- Docker + Docker Compose v2
- (Recommended) NVIDIA GPU with [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

---

## First-time setup

### 1. Prepare config files

```bash
cp .env.example .env
# Edit .env and set real secret keys:
#   openssl rand -hex 32   (run twice, paste into each secret field)
nano .env
```

### 2. First-run SearXNG bootstrap

SearXNG needs to write `uwsgi.ini` on first launch, which requires extra capabilities.
Temporarily edit `docker-compose.yml` and **comment out** `cap_drop: - ALL` under the `searxng` service, then:

```bash
docker compose up searxng redis -d
sleep 20
docker compose down
```

Now **restore** `cap_drop: - ALL` in the compose file, then add JSON format support so Open WebUI can query SearXNG:

```bash
# Linux / macOS:
sed -i 's/  - html/  - html\n  - json/' searxng/settings.yml

# Or edit manually – find the 'formats:' block and make it look like:
# formats:
#   - html
#   - json
```

Also add Redis as the cache backend by ensuring `searxng/settings.yml` contains:

```yaml
server:
  secret_key: "your-searxng-secret"   # must match SEARXNG_SECRET_KEY in .env
  limiter: true
  image_proxy: true

redis:
  url: redis://redis:6379/0
```

### 3. Start the full stack

```bash
docker compose up -d
```

### 4. Pull your first model

```bash
# General purpose (fast, fits in 8 GB VRAM):
docker exec -it ollama ollama pull qwen2.5:7b

# Coding (recommended for dev tasks):
docker exec -it ollama ollama pull qwen2.5-coder:7b

# Reasoning:
docker exec -it ollama ollama pull deepseek-r1:8b

# Vision (multimodal):
docker exec -it ollama ollama pull llava:7b
```

Open **http://localhost:3000**, create an admin account on first login, and select your model.

---

## Using llama.cpp server (optional)

The `llama-server` service is gated behind the `llama-cpp` Docker Compose profile so it doesn't start by default.

1. Place a GGUF file in the `llama_models` volume (or bind-mount `./models:/models`).
2. Set `LLAMA_MODEL=yourfile.Q4_K_M.gguf` in `.env`.
3. Start it:
   ```bash
   docker compose --profile llama-cpp up -d llama-server
   ```
4. In Open WebUI → **Admin Panel → Settings → Connections**, add:
   - URL: `http://llama-server:8080/v1`
   - API Key: (leave blank)

---

## Enabling web search in a chat

1. Click the **+** icon to the left of the message box.
2. Toggle **Web Search** on.
3. The model will now search SearXNG before responding.

---

## Text-to-speech

TTS is pre-configured to use `openedai-speech`. To enable it per-chat:

- Click the **speaker icon** in the chat controls, or
- Go to **Settings → Audio** and set your preferred voice (`nova`, `alloy`, `echo`, `fable`, `onyx`, `shimmer`).

Models are downloaded from Hugging Face on first use (~500 MB).

---

## GPU configuration

### NVIDIA
The `ollama` service has the GPU block enabled by default. Requires:
```bash
# Ubuntu/Debian
sudo apt install nvidia-container-toolkit
sudo systemctl restart docker
```

### AMD (ROCm)
Change the Ollama image and add device passthrough:
```yaml
ollama:
  image: ollama/ollama:rocm
  devices:
    - /dev/kfd
    - /dev/dri
```
Remove the `deploy.resources` block.

### CPU only
Remove or comment out the entire `deploy:` block from the `ollama` service.

---

## Useful commands

```bash
# View logs
docker compose logs -f open-webui
docker compose logs -f ollama

# List downloaded models
docker exec -it ollama ollama list

# Remove a model
docker exec -it ollama ollama rm modelname

# Update all images
docker compose pull && docker compose up -d

# Stop everything
docker compose down

# Full reset (removes volumes/data!)
docker compose down -v
```

---

## Recommended models for coding tasks

| Model | Size | Best for |
|---|---|---|
| `qwen2.5-coder:7b` | ~4 GB | Fast everyday coding |
| `qwen2.5-coder:14b` | ~8 GB | Better code reasoning |
| `deepseek-r1:8b` | ~5 GB | Step-by-step problem solving |
| `codellama:13b` | ~7 GB | Code completion & infill |
| `phi4:14b` | ~8 GB | General + strong at math/code |
