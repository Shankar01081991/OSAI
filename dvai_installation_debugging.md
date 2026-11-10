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
<img width="637" height="167" alt="image" src="https://github.com/user-attachments/assets/a5e9f4ef-3cf4-4fd8-b151-7e5b72f0163a" />

### Step 2: Install Node.js, Python 3.13, and Dependencies
```bash
# Install Node.js & NPM for frontend
sudo apt install -y nodejs npm

# Install Python 3.13 environment tools
sudo apt install -y python3.13 python3.13-venv

# Install Python requirements
python3 -m pip install --break-system-packages -r requirements.txt
```
<img width="641" height="173" alt="image" src="https://github.com/user-attachments/assets/1c4652ae-bc61-46d7-a2d2-27609c24af13" />
<img width="506" height="152" alt="image" src="https://github.com/user-attachments/assets/b09bfe02-a96a-437e-95ed-c6e591ad4b17" />
<img width="741" height="216" alt="image" src="https://github.com/user-attachments/assets/8fc72b61-48f2-4a51-a477-af3790c43661" />

---

## 3. Ollama Installation & Service Configuration

### Step 1: Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
```
<img width="631" height="292" alt="image" src="https://github.com/user-attachments/assets/77830afa-4d34-4525-9f78-059dbbce47e6" />

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
<img width="627" height="155" alt="image" src="https://github.com/user-attachments/assets/e33b8c7e-3dce-4385-a856-d6ee51ebc884" />

Verify model execution:
```bash
ollama run qwen2.5:7b
```
<img width="627" height="277" alt="image" src="https://github.com/user-attachments/assets/914ff9bd-374c-4f15-9a9c-57ab0084229b" />

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
<img width="387" height="341" alt="image" src="https://github.com/user-attachments/assets/55a4b125-1555-4aa6-abb2-bed4455ca5ec" />


Once running:
* **Frontend UI:** `http://localhost:3000`
* **Backend API:** `http://localhost:8000`
<img width="864" height="329" alt="image" src="https://github.com/user-attachments/assets/c48eec8f-f6bc-43ab-bede-7d616001c286" />
