
# Step 1 — Check GPU

Run:

```bash
nvidia-smi
```

You need ideally:

* 2× RTX 4090
* OR A100 80GB
* OR multiple enterprise GPUs

70B models need very high VRAM. Ollama can automatically split layers across multiple GPUs. ([localaiops.com][1])

---

# Step 2 — Install NVIDIA Container Toolkit

Required for Docker GPU passthrough. ([Ollama Documentation][2])

Ubuntu:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
| sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
| sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
| sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update

sudo apt install -y nvidia-container-toolkit

sudo nvidia-ctk runtime configure --runtime=docker

sudo systemctl restart docker
```

---

# Step 3 — Verify Docker GPU Access

Run:

```bash
docker run --rm --gpus all nvidia/cuda:12.3.0-base-ubuntu22.04 nvidia-smi
```

If GPU appears → Docker GPU passthrough works.

---

# Step 4 — Create Dokploy Project

Inside Dokploy:

* Create new Application
* Choose Docker Compose

---

# Step 5 — Use This Compose File

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped

    ports:
      - "11434:11434"

    volumes:
      - ollama_data:/root/.ollama

    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility

volumes:
  ollama_data:
```

This is the correct modern Docker GPU reservation format. ([glukhov.org][3])

---

# Step 6 — Deploy in Dokploy

Click:

```text
Deploy
```

Wait until container becomes healthy.

---

# Step 7 — Open Terminal in Container

Inside Dokploy terminal:

```bash
ollama pull llama3.1:70b
```

This downloads ~40GB+.

May take long depending on internet speed.

---

# Step 8 — Run Model

```bash
ollama run llama3.1:70b
```

---

# Step 9 — Verify GPU Usage

On host machine:

```bash
watch -n 1 nvidia-smi
```

You should see:

* VRAM usage increasing
* GPU compute usage active

---

# Step 10 — Verify Ollama Detects GPU

Inside container:

```bash
ollama ps
```

or:

```bash
docker logs ollama
```

You should see NVIDIA GPU detection. ([Stack Overflow][4])

---

# Multi-GPU Support

Ollama automatically distributes 70B models across GPUs. ([localaiops.com][1])

Example:

```bash
CUDA_VISIBLE_DEVICES=0,1 ollama serve
```

Usually not required — Ollama auto-detects GPUs.

---

# VERY IMPORTANT

If your GPU VRAM is insufficient:

The model will:

* spill into RAM
* become extremely slow
* or fail completely

Common issue reported by users running 70B locally. ([Reddit][5])

---

# Recommended Production Setup

| Component      | Recommendation |
| -------------- | -------------- |
| LLM            | `llama3.1:70b` |
| Embedding      | `bge-m3`       |
| Vector DB      | Qdrant         |
| Reverse Proxy  | Traefik        |
| Deployment     | Dokploy        |
| GPU Monitoring | `nvidia-smi`   |

---

# Recommended Folder Structure

```bash
/opt/ai/
 ├── ollama/
 ├── qdrant/
 ├── open-webui/
 └── dhangpt/
```

---

# Optional — Add Open WebUI

You can add:

Open WebUI

to chat with your 70B model visually.

Add service:

```yaml
open-webui:
  image: ghcr.io/open-webui/open-webui:main
  ports:
    - "3000:8080"
  environment:
    - OLLAMA_BASE_URL=http://ollama:11434
```

---

# Final Important Check

Before pulling 70B, run:

```bash
nvidia-smi
```

and check:

| GPU         | Can Run 70B?   |
| ----------- | -------------- |
| RTX 3060    | No             |
| RTX 4070    | No             |
| Single 4090 | Partial / slow |
| Dual 4090   | Yes            |
| A100 80GB   | Excellent      |

[1]: https://localaiops.com/posts/multi-gpu-ollama-setup-running-70b-models-on-dual-gpus/?utm_source=chatgpt.com "Multi-GPU Ollama Setup: Running 70B Models on Dual GPUs | Local AI Ops"
[2]: https://docs.ollama.com/docker?utm_source=chatgpt.com "Docker - Ollama"
[3]: https://www.glukhov.org/llm-hosting/ollama/ollama-in-docker-compose/?utm_source=chatgpt.com "Ollama in Docker Compose with GPU and Persistent Model Storage - Rost Glukhov | Personal site and technical blog"
[4]: https://stackoverflow.com/questions/78388016/run-ollama-with-docker-compose-and-using-gpu?utm_source=chatgpt.com "Run ollama with docker-compose and using gpu - Stack Overflow"
[5]: https://www.reddit.com/r/ollama/comments/1diawsq?utm_source=chatgpt.com "Ollama does not use GPU"
