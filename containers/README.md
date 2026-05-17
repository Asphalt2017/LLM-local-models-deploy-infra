# Ollama (Docker) Guide

This folder runs an Ollama server in Docker using Compose.

## Model compatibility by GPU

Use these tables as practical starting points for Ollama model selection.

### 12 GB (RTX 3060)

| Model family | Example Ollama tag | Size class | Recommended quantization | Fit | Notes |
|---|---|---|---|---|---|
| Llama 3.2 | `llama3.2:3b` | 3B | Q4_K_M / Q5_K_M | Excellent | Fast and reliable for chat and utility tasks. |
| Llama 3.1 | `llama3.1:8b` | 8B | Q4_K_M | Excellent | Great quality/performance balance. |
| Mistral | `mistral:7b` | 7B | Q4_K_M | Excellent | Strong general-purpose model. |
| Qwen 2.5 | `qwen2.5:7b` | 7B | Q4_K_M | Excellent | Good reasoning and coding quality for size. |
| DeepSeek R1 Distill | `deepseek-r1:7b` | 7B | Q4_K_M | Excellent | Better reasoning style than many 7B models. |
| Llama 3.1 | `llama3.1:13b` | 13B | Q4_K_M | Good | Works with tighter context/settings. |
| Qwen 2.5 | `qwen2.5:14b` | 14B | Q4_K_M | Fair | Usable, but slower on longer contexts. |
| DeepSeek R1 Distill | `deepseek-r1:14b` | 14B | Q4_K_M | Fair | Better reserved for non-latency-sensitive tasks. |
| Any 30B+ model | varies | 30B+ | Q4/Q5 | Not recommended | Usually too slow or memory constrained. |

### 6 GB (RTX 1060 Ti)

| Model family | Example Ollama tag | Size class | Recommended quantization | Fit | Notes |
|---|---|---|---|---|---|
| Llama 3.2 | `llama3.2:3b` | 3B | Q4_K_M / Q5_K_M | Excellent | Best responsiveness and stability. |
| Phi | `phi3:mini` | ~3.8B | Q4_K_M | Excellent | Efficient model for lightweight tasks. |
| Gemma | `gemma2:2b` | 2B | Q4_K_M / Q5_K_M | Excellent | Very fast for simple prompts. |
| Mistral | `mistral:7b` | 7B | Q4_K_M | Good | Usable, but speed depends on context size. |
| Qwen 2.5 | `qwen2.5:7b` | 7B | Q4_K_M | Good | Good quality; reduce context if needed. |
| Llama 3.1 | `llama3.1:8b` | 8B | Q4_K_M | Limited | May require smaller context and occasional CPU offload. |
| Any 13B+ model | varies | 13B+ | Q4/Q5 | Not recommended | Often memory constrained on 6 GB VRAM. |

Legend:

- Excellent: smooth for daily use.
- Good: usable, sometimes lower throughput.
- Fair: works but slower, may need smaller context.
- Limited: may require aggressive quantization or CPU offload.

Quick recommendations:

- Best overall for 12 GB (RTX 3060): 7B to 8B models in Q4_K_M.
- Best overall for 6 GB (RTX 1060 Ti): 2B to 7B models in Q4_K_M.
- For coding: try `qwen2.5:7b` or `deepseek-r1:7b` first.

## What is running

- Service name: `ollama`
- Image: `ollama/ollama:latest`
- Host port: `11000`
- Container port: `11434`
- API base URL from host: `http://localhost:11000`
- Persistent model storage: named volume `ollama_data` mounted at `/root/.ollama`

## Start and stop the container

Run from this folder (`models/`):

```bash
docker compose up -d
```

Stop:

```bash
docker compose down
```

See logs:

```bash
docker compose logs -f ollama
```

## How to connect

### 1) From your host using HTTP API

Check server and list models:

```bash
curl http://localhost:11000/api/tags
```

### 2) From inside the container using CLI

Open shell:

```bash
docker compose exec ollama bash
```

Then run Ollama commands, for example:

```bash
ollama list
```

### 3) Run one-off commands without opening shell

```bash
docker compose exec ollama ollama list
```

## Useful Ollama commands (with explanation)

### Model management

```bash
ollama list
```
- Lists all locally available models.

```bash
ollama pull llama3
```
- Downloads a model into local storage.

```bash
ollama show llama3
```
- Shows metadata/details for a model.

```bash
ollama rm llama3
```
- Removes a local model.

### Inference

```bash
ollama run llama3
```
- Starts an interactive chat with the model.

```bash
ollama run llama3 "Explain transformers in simple terms"
```
- Runs a single prompt and prints the response.

### Server/API checks

```bash
curl http://localhost:11000/api/tags
```
- Returns model tags visible through the API.

```bash
curl http://localhost:11000/api/generate \
  -H "Content-Type: application/json" \
  -d '{"model":"llama3","prompt":"Hello"}'
```
- Sends a direct text generation request.

## Docker/Compose helper commands

```bash
docker compose ps
```
- Shows service status.

```bash
docker compose restart ollama
```
- Restarts the service.

```bash
docker compose exec ollama nvidia-smi
```
- Confirms GPU visibility inside container (if NVIDIA GPU is configured).

## Common configuration points

Current setup is defined in `docker-compose.yml`:

- `ports`: maps host `11000` to container `11434`
- `volumes`: keeps models in `ollama_data` so they survive restarts
- `deploy.resources.reservations.devices`: requests NVIDIA GPU
- `OLLAMA_HOST=0.0.0.0`: allows binding on all interfaces inside container

## Quick troubleshooting

- `docker compose up` fails with GPU-related error:
  - Verify NVIDIA drivers and container toolkit are installed.
  - Try removing GPU reservation block temporarily to test CPU mode.

- API not reachable on host:
  - Check service status: `docker compose ps`
  - Check logs: `docker compose logs -f ollama`
  - Verify port conflict on `11000`.

- Model not found:
  - Pull it first: `docker compose exec ollama ollama pull <model>`
  - List installed models: `docker compose exec ollama ollama list`
