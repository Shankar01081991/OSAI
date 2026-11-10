# DVAI Environment Setup & Debugging Guide

This document provides a complete guide for setting up **Damn Vulnerable AI (DVAI)** on a fresh Kali Linux environment, configuring local LLM backends with Ollama, and resolving common backend execution errors.

---

## 1. System Requirements & Recommendations

Running local LLMs alongside web applications requires adequate hardware allocation:

| Resource | Minimum | Recommended | Notes |
| :--- | :--- | :--- | :--- |
| **Kali VM RAM** | 8 GB | 16 GB | Base memory for system & models |
| **Free RAM for Model** | ~6 GB | 8 GB+ | Required during active token generation |
| **Disk Space** | 8 GB | 15–20 GB | Accommodates model weights (`qwen2.5:7b` ~4.7 GB) |
| **CPU Cores** | 4 Cores | 4–8 Cores | Multi-threading speeds up CPU inference |
| **GPU** | Not required | Optional | Accelerates response times if available |

---

## 2. Base System & Dependency Installation

### Step 1: Update System & Clone Repository
```bash
sudo apt update && sudo apt upgrade -y
git clone https://github.com/offensiveai26/DVAI.git
cd DVAI
```

### Step 2: Install Node.js, Python 3.13, and Dependencies
```bash
# Install Node.js & NPM for frontend
sudo apt install -y nodejs npm

# Install Python 3.13 environment tools
sudo apt install -y python3.13 python3.13-venv

# Install Python requirements
python3 -m pip install --break-system-packages -r requirements.txt
```

---

## 3. Ollama Installation & Service Configuration

### Step 1: Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
```

### Step 2: Enable and Start Service
```bash
sudo systemctl enable --now ollama
sudo systemctl status ollama --no-pager
```

### Step 3: Pull Local Language Models
Depending on hardware resources, choose a lightweight or standard model:

```bash
# Lightweight model (Low RAM)
ollama pull llama3.2:1b

# Recommended model for logic and cybersecurity analysis (~4.7 GB download)
ollama pull qwen2.5:7b
```

Verify model execution:
```bash
ollama run qwen2.5:7b
```

---

## 4. Backend Script Modification & Debugging

If startup issues occur or the backend fails to activate the virtual environment properly, update `start.sh`.

### Patching `start.sh` Backend Setup Section
Open `start.sh` in a text editor:
```bash
nano start.sh
```

Replace the `# Backend setup` section with the following block:

```bash
# Backend setup
echo -e "${GREEN}Starting backend...${NC}"
cd backend

if [ ! -d ".venv" ]; then
    python3.13 -m venv .venv
fi

source .venv/bin/activate
python -m pip install -q -r requirements.txt
python -m uvicorn app.main:app --port 8000 --reload &
BACKEND_PID=$!
cd ..
```

---

## 5. Model Aliasing & Application Launch

DVAI's default backend looks for model tags matching `llama3`. If using `qwen2.5:7b`, create a model alias within Ollama:

```bash
# Create model alias
ollama cp qwen2.5:7b llama3

# Confirm model creation
ollama list
```

### Launching the Application
Export the environment variables and run the startup script:

```bash
cd ~/Desktop/DVAI
sudo systemctl restart ollama
export OLLAMA_MODEL=llama3
export MODEL_NAME=llama3
./start.sh
```

Once running:
* **Frontend UI:** `http://localhost:3000`
* **Backend API:** `http://localhost:8000`