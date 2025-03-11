# 🚀 Run Your Own RL Swarm Node

**RL Swarm** is a fully open-source framework developed by **GensynAI** for building reinforcement learning (RL) training swarms over the internet. This guide will help you set up an RL Swarm node and a web UI dashboard to monitor swarm activity. Let’s get started! 🚀

---

## 🖥️ Hardware Requirements

🔹 **Minimum Requirements**:
- **CPU**: 16GB RAM (More recommended for larger models or datasets)
- **GPU (Optional, for better performance)**:
  - NVIDIA RTX 3090 / RTX 4090
  - NVIDIA A100 / H100
- **Note**: Running in CPU-only mode is possible (details in `docker-compose.yaml`).

---

## ⚙️ Install Dependencies

### 1️⃣ Update System Packages
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### 2️⃣ Install Essential Utilities
```bash
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### 3️⃣ Install Docker
```bash
# Remove old Docker versions
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test installation
sudo docker run hello-world
```

✅ **Tip:** Run Docker without `sudo` by adding your user to the Docker group:
```bash
sudo usermod -aG docker $USER
```

### 4️⃣ Install Python
```bash
sudo apt-get install python3 python3-pip
```

---

## 📂 Clone the Repository
```bash
git clone https://github.com/gensyn-ai/rl-swarm/
cd rl-swarm
```

---

## 🛠️ Create & Configure `docker-compose.yaml`
This file defines essential services like the RL Swarm node, telemetry collector, and web UI.

🔹 **Backup old file:**
```bash
mv docker-compose.yaml docker-compose.yaml.old
```
🔹 **Create a new one:**
```bash
nano docker-compose.yaml
```
🔹 **Paste the following content:**
```yaml
version: '3'

services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.120.0
    ports:
      - "4317:4317"  # OTLP gRPC
      - "4318:4318"  # OTLP HTTP
      - "55679:55679"  # Prometheus metrics (optional)
    environment:
      - OTEL_LOG_LEVEL=DEBUG

  swarm_node:
    image: europe-docker.pkg.dev/gensyn-public-b7d9/public/rl-swarm:v0.0.1
    command: ./run_hivemind_docker.sh
    runtime: nvidia  # Remove if running on CPU
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - PEER_MULTI_ADDRS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
      - HOST_MULTI_ADDRS=/ip4/0.0.0.0/tcp/38331
    ports:
      - "38331:38331"
    depends_on:
      - otel-collector

  fastapi:
    build:
      context: .
      dockerfile: Dockerfile.webserver
    environment:
      - OTEL_SERVICE_NAME=rlswarm-fastapi
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - INITIAL_PEERS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
    ports:
      - "8080:8000"
    depends_on:
      - otel-collector
      - swarm_node
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/healthz"]
      interval: 30s
      retries: 3
```
🔹 **GPU Users:** Leave `runtime: nvidia` intact.
🔹 **CPU Users:** Remove the `runtime: nvidia` line.


---

## 🚀 Run RL Swarm Node + Web UI Dashboard
```bash
docker compose up --build -d && docker compose logs -f
```

---

## 🌐 Access the Web UI Dashboard
🔹 **VPS Users:** `http://<your-vps-ip>:8080/`
🔹 **Local Users:** `http://localhost:8080` or `http://0.0.0.0:8080`


---

### 🎉 You're all set!
You’ve successfully deployed an RL Swarm node. 🚀 Have fun experimenting, and feel free to contribute to the project!