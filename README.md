# Documentation: Custom n8n + Python Docker Build

This documentation provides a step-by-step guide to building, extending, and pushing a custom **n8n** image equipped with **Python 3, FFmpeg, Git, and system utilities** using **Google Cloud Shell**.

---

## 1. Overview & Capabilities

The standard `n8n` image is lightweight but lacks native runtime tools for complex workflows. This custom image extends n8n with:

* **Python 3 & Pip (`python3`, `py3-pip`):** Run custom Python scripts, AI automation, and data processing inside n8n's **Code** or **Execute Command** nodes.
* **FFmpeg (`ffmpeg`):** Process audio and video files (useful for OpenAI Whisper transcriptions).
* **PDF Utilities (`poppler-utils`):** Convert and extract text from PDFs (`pdftotext`).
* **CLI Tools (`curl`, `wget`, `jq`, `git`):** Download files, manipulate JSON payloads, and clone GitHub repositories.

---

## 2. Cloud Environment Setup (Google Cloud Shell)

Google Cloud Shell provides a free terminal in your browser with Docker pre-installed.

1. Open **[shell.cloud.google.com](https://shell.cloud.google.com)** in your browser.
2. (Optional) Create a workspace folder to organize your project files:
   ```bash
   mkdir n8n-custom && cd n8n-custom
   ```
3. Create or edit a new file named `Dockerfile`:
   ```bash
   nano Dockerfile
   ```

---

## 3. The `Dockerfile` Code

Because official n8n images use a stripped-down Alpine base, this **Multi-Stage Dockerfile** compiles all binaries and shared libraries cleanly inside a temporary Alpine environment before copying them into n8n.

```dockerfile
# Stage 1: Install all required packages inside a clean Alpine builder
FROM alpine:3.20 AS builder

RUN apk add --no-cache \
    python3 \
    py3-pip \
    curl \
    wget \
    jq \
    git \
    ffmpeg \
    poppler-utils

# Stage 2: Target official n8n image and copy dependencies
FROM n8nio/n8n:latest

USER root

# Copy installed binaries, libraries, and configs from builder stage
COPY --from=builder /usr/bin /usr/bin
COPY --from=builder /usr/lib /usr/lib
COPY --from=builder /usr/share /usr/share
COPY --from=builder /etc /etc

# Switch back to non-root 'node' user for security
USER node
```

---

## 4. Build, Login & Push Commands

Run these commands in your Google Cloud Shell terminal to compile and upload the image to Docker Hub.

### Step A: Authenticate with Docker Hub
```bash
docker login
```
> Enter your Docker Hub username (`none344sd`) and password (or Access Token) when prompted.

### Step B: Build the Image
```bash
docker build -t none344sd/python_n8n:latest .
```

### Step C: Push to Docker Hub
```bash
docker push none344sd/python_n8n:latest
```

---

## 5. Docker Hub Repository

* **Repository Name:** `none344sd/python_n8n`
* **Full Image Tag:** `none344sd/python_n8n:latest`
* **Docker Hub Link:** [https://hub.docker.com/r/none344sd/python_n8n](https://hub.docker.com/r/none344sd/python_n8n)

---

## 6. How to Run Your Image Anywhere

Once pushed, you can run your custom image on any server, VPS, or cloud host using Docker or Docker Compose.

### Quick Run Command
```bash
docker run -d \
  --name n8n \
  --restart always \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  none344sd/python_n8n:latest
```

### Docker Compose Configuration (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  n8n:
    image: none344sd/python_n8n:latest
    container_name: n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_OPTIONS=--max-old-space-size=384
      - EXECUTIONS_PROCESS=main
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```
