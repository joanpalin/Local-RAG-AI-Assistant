# Guide 1: Environment Setup
## Building Your Local AI Foundation

**Part 1 of 3: Implementation Guides**  
**Time Required:** 45-60 minutes  
**Difficulty:** Beginner-friendly (step-by-step)  
**Prerequisites:** Computer with sufficient resources, 16GB+ RAM

---

## Table of Contents

1. [Introduction](#introduction)
2. [What You'll Accomplish](#what-youll-accomplish)
3. [System Requirements](#system-requirements)
4. [Part 1: Understanding the Components](#part-1-understanding-the-components)
5. [Part 2: Installing Podman](#part-2-installing-podman)
6. [Part 3: Installing Ollama](#part-3-installing-ollama)
7. [Part 4: Setting Up ChromaDB](#part-4-setting-up-chromadb)
8. [Part 5: Setting Up n8n](#part-5-setting-up-n8n)
9. [Part 6: Python Setup](#part-6-python-setup)
10. [Part 7: Final Verification](#part-7-final-verification)
11. [Troubleshooting](#troubleshooting)
12. [Next Steps](#next-steps)

---

## Introduction

Welcome! This guide will help you set up the foundation for your local AI document assistant. By the end of this guide, you'll have all the necessary software installed and verified working on your local machine.

**What makes this guide different:** Every instruction has been tested on actual hardware. All common issues have been identified and documented. If you follow the steps exactly as written, you'll be very close to have a working system.

### What to Expect

- **Realistic time estimates** for each section
- **Plain English explanations** before technical terms
- **Verification steps** after each major installation
- **Troubleshooting help** for common issues
- **Success checkpoints** so you know when you're done

### If You Get Stuck

1. **Don't panic** - most issues are simple permission or path problems
2. **Check the Troubleshooting section** at the end of this guide
3. **Verify prerequisites** - did you complete previous steps?
4. **Read error messages carefully** - they usually tell you what's wrong
5. **Start fresh** - sometimes it's easier to remove and reinstall
6. Remember that you have the most powerful troubleshooting tool to assist you... your preferred AI GPT. Tell them what you are doing, share this guide, provide the step and the issue you are having, and will guide you through to solve it.

⚡ **Ready? Let's build your AI assistant!**

---

## What You'll Accomplish

By completing this guide, you will have:

✓ **Podman** - Container runtime (like Docker) running virtual environments  
✓ **Ollama** - AI model manager with two models downloaded  
✓ **ChromaDB** - Document storage system that AI can search  
✓ **n8n** - Visual workflow builder (no coding needed)  
✓ **Python setup** - For quick testing scripts  

**What this enables:**
- Process documents locally (no data leaves your machine)
- Run AI models without internet connection
- Build visual workflows without writing code
- Query documents through multiple interfaces

---

## System Requirements

### Hardware Requirements

**Minimum (Will work, but slower):**
- Modern computer with sufficient processing power
- 16GB RAM
- 50GB free disk space
- Modern operating system (Windows 10+, macOS 12+, or recent Linux)

**Recommended (Better performance):**
- Multi-core processor (6+ cores)
- 24GB+ RAM
- 100GB free disk space
- Latest operating system updates

**Note:** This guide focuses on macOS with Apple Silicon, but the principles apply to other platforms with appropriate adjustments. If you need Windows or other OS guides, give this guide to your AI GPT and ask them to replace the steps with the appropiate ones based on your system.

### Pre-Installation Checklist

Before you begin, verify:

- [ ] You have administrator access to your machine
- [ ] You can open a terminal/command line interface
- [ ] You have an internet connection for downloads
- [ ] You have 45-60 minutes of uninterrupted time
- [ ] You've read this introduction

💡 **Tip:** Close unnecessary applications to free up RAM before starting.

---

## Part 1: Understanding the Components

**Time:** 10 minutes (reading)

Before installing anything, let's understand what each component does and why we need it. Think of this like understanding the parts of a network before cabling everything together.

### Component 1: Podman

**What it is:** A tool that runs software in isolated "containers"  
**Why we need it:** ChromaDB and n8n need consistent environments  
**Real-world analogy:** Like a virtual machine, but lighter weight

**Think of it this way:** 
Imagine you're a network engineer setting up a demo in a conference room. Instead of bringing physical servers, you bring virtual machines on a laptop. Podman does something similar - it runs "virtual computers" (containers) inside your machine, each with its own isolated environment.

**Why containers matter:**
- Consistent behavior (works the same every time)
- Easy to start/stop
- Doesn't conflict with other software on your system
- Easy to remove if needed (no leftover files)

**Example:** 
If ChromaDB needs Python 3.9 but your system has Python 3.11, no problem - the container has its own Python 3.9 that doesn't interfere with your system's Python.

**Why Podman instead of Docker?**
- Free and open source (no licensing issues)
- Better security model (doesn't need root)
- Works well on modern systems
- Similar commands to Docker (easy to learn)

---

### Component 2: Ollama

**What it is:** Software that runs AI language models locally  
**Why we need it:** Provides the "intelligence" - reads and understands documents  
**Real-world analogy:** Like having a really smart colleague who never sleeps

**Think of it this way:**
Remember when network monitoring meant someone physically checking lights on routers? Then we got SNMP agents that automated the monitoring. Ollama is similar - instead of sending questions to a remote AI service, you have an AI "agent" running locally that can read documents and answer questions.

**What Ollama does:**
1. **Hosts AI models** - The actual "brains" that understand language
2. **Manages downloads** - Gets models from the internet (one-time)
3. **Provides API** - Other software can talk to it
4. **Handles memory** - Loads/unloads models as needed

**Models we'll use:**
- **llama3.2:3b** (2GB) - Answers questions in natural language
- **nomic-embed-text** (274MB) - Converts text into numbers for searching

**Why two models?**
- **Embedding model** (nomic) - Converts your documents into a special number format (vectors) that computers can compare quickly - think of it like creating routing tables from network topology
- **Language model** (llama) - Reads the retrieved information and writes human-friendly answers - like SNMP data being converted into readable reports

**Size note:** "3b" means 3 billion parameters - smaller than ChatGPT (175B) but perfect for running locally. It's like the difference between a core router and a branch router - one is for massive scale, the other is perfect for your specific needs.

---

### Component 3: ChromaDB

**What it is:** A specialized database for storing and searching document content  
**Why we need it:** Makes searching through documents lightning-fast  
**Real-world analogy:** Like a smart index that understands meaning, not just keywords

**Think of it this way:**
Traditional search is like looking for an exact switch model number in documentation - you have to know the exact term. ChromaDB is like asking "what switch supports 10G fiber?" and it finds relevant sections even if they use different words like "10 Gigabit optical interfaces."

**What makes ChromaDB special:**
Instead of storing text as-is, it stores it as "vectors" (lists of numbers that capture meaning). When you search, it finds vectors similar to your question's vector.

**Practical example:**
- **You ask:** "What's our upgrade budget?"
- **Document says:** "The network refresh costs $250,000"
- **Traditional search:** No match (different words)
- **ChromaDB:** Finds it (similar meaning)

**Technical term (for reference):**
This is called "semantic search using vector embeddings" - but all you need to know is: it finds relevant information even when you phrase things differently than the document.

**Why version 0.4.24 specifically?**
Through testing, we discovered that:
- Latest versions have incomplete APIs (some features don't work)
- Version 0.4.24 has the complete v1 API
- It's stable and well-tested
- All our workflows are designed for this version

**Storage:** Think of ChromaDB like a routing table, but instead of network prefixes and next hops, it stores text chunks and their mathematical representations (vectors). Just as routers quickly match IP addresses using longest prefix matching, ChromaDB quickly matches questions using vector similarity.

---

### Component 4: n8n

**What it is:** Visual workflow automation tool (like Zapier, but self-hosted)  
**Why we need it:** Build AI workflows without writing code  
**Real-world analogy:** Like a network diagram tool, but for automating tasks

**Think of it this way:**
Remember drawing network diagrams in Visio - boxes for routers, lines for connections? n8n is similar but for workflows. Instead of routers and switches, you have steps like "receive document," "break into chunks," "store in database." Instead of cables, you have arrows showing data flow.

**What n8n does:**
- **Visual interface** - Drag and drop boxes (called "nodes")
- **Connects services** - Makes Ollama, ChromaDB, and Webex talk to each other
- **No coding needed** - Click, configure, connect
- **Easy to modify** - Want to change something? Just edit the diagram

**Example workflow:**
```
[User uploads document] → [Break into chunks] → [Convert to vectors] → [Store in ChromaDB]
```

Each box is a "node" - you configure what it does, then connect the arrows.

**Why n8n specifically?**
- Self-hosted (runs locally, no cloud service needed)
- Free and open source
- Large library of pre-built nodes
- Active community
- Great documentation

**For network engineers:** Think of n8n nodes like CLI commands in a script - each node does one thing, and you chain them together. But instead of writing bash scripts, you're building visual diagrams.

---

### How It All Works Together

Here's the complete picture:

```
Local Machine
│
├── Ollama (running natively)
│   ├── llama3.2:3b model (answers questions)
│   └── nomic-embed-text model (converts text to vectors)
│
└── Podman (container engine)
    │
    ├── ChromaDB container (vector database)
    │   └── Stores your documents as searchable vectors
    │
    └── n8n container (workflow automation)
        └── Coordinates everything
```

**Data Flow Visualization:**

```
┌─────────────────────────────────────────────────────────┐
│                     YOUR LOCAL MACHINE                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────────┐         ┌─────────────────┐         │
│  │  You (User)    │────────>│   n8n Workflow  │         │
│  │  Ask Question  │         │   Orchestrates  │         │
│  └────────────────┘         └────────┬────────┘         │
│                                      │                  │
│                                      v                  │
│                          ┌───────────────────┐          │
│                          │   Question goes   │          │
│                          │   to Ollama for   │          │
│                          │   embedding       │          │
│                          └─────────┬─────────┘          │
│                                    │                    │
│                                    v                    │
│                          ┌───────────────────┐          │
│                          │   ChromaDB        │          │
│                          │   Searches for    │          │
│                          │   similar vectors │          │
│                          └─────────┬─────────┘          │
│                                    │                    │
│                                    v                    │
│                          ┌───────────────────┐          │
│                          │   Relevant chunks │          │
│                          │   sent to Ollama  │          │
│                          │   (llama model)   │          │
│                          └─────────┬─────────┘          │
│                                    │                    │
│                                    v                    │
│  ┌────────────────┐         ┌───────────────┐           │
│  │  You receive   │<────────│  Answer       │           │
│  │  answer        │         │  generated    │           │ 
│  └────────────────┘         └───────────────┘           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Example Query Flow:**
1. **You ask:** "What's the datacenter budget?"
2. **n8n receives** your question via web interface
3. **Ollama (embedding model)** converts question to vector
4. **ChromaDB searches** its database for similar vectors
5. **ChromaDB returns** relevant document chunks
6. **Ollama (language model)** reads chunks and generates answer
7. **n8n returns** the answer to you

**Network analogy:** 
- n8n is like your network controller (orchestrates traffic)
- ChromaDB is like your routing table (finds best paths)
- Ollama is like your packet processor (analyzes and handles data)
- You are the client sending requests

**Important:** Everything happens locally - no data leaves your machine.

---

## Part 2: Installing Podman

**Time:** 15 minutes  
**What you'll do:** Install Podman and verify it works

### Understanding What We're Installing

**Podman** is our container engine. It will:
- Run ChromaDB and n8n in isolated environments
- Manage container lifecycle (start, stop, restart)
- Handle networking between containers
- Map local folders into containers

### Installation Steps

#### Step 1: Download Podman Desktop

**Action:**
1. Visit: https://podman-desktop.io/downloads
2. Download for your operating system
3. Run the installer
4. Follow the installation wizard

**Expected time:** 5 minutes

**What happens:** Podman Desktop installs both:
- Podman CLI (command-line tool)
- Podman Desktop (graphical interface)

---

#### Step 2: Initialize Podman Machine

**Action:**
```bash
# Open Terminal and run:
podman machine init --cpus 4 --memory 12288 --disk-size 50
```

**What this does:**
- Creates a virtual machine for running containers
- Allocates 4 CPU cores
- Allocates 12GB RAM (12288 MB)
- Allocates 50GB disk space

**Expected output:**
```
Downloading VM image: fedora-coreos-XX.XXXXXX.X.X-qemu.aarch64.qcow2.xz: done
Extracting compressed file
Image resized.
Machine init complete
```

**Time:** 3-5 minutes (includes download)

---

#### Step 3: Start Podman Machine

**Action:**
```bash
podman machine start
```

**Expected output:**
```
Starting machine "podman-machine-default"
Waiting for VM...
Machine "podman-machine-default" started successfully
```

**Time:** 30 seconds

---

#### Step 4: Verify Installation

**Action:**
```bash
podman --version
podman ps
```

**Expected output:**
```
podman version 4.X.X or later

CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

**What this means:**
- First command: Shows Podman version (confirms installation)
- Second command: Lists running containers (should be empty now)

---

### Success Checkpoint

You've successfully installed Podman when:
- ✓ `podman --version` shows a version number
- ✓ `podman ps` returns without errors
- ✓ Podman Desktop app opens (if you installed it)

**If something failed:** See Troubleshooting section at end of guide

---

## Part 3: Installing Ollama

**Time:** 20 minutes  
**What you'll do:** Install Ollama, download AI models, verify they work

### Understanding What We're Installing

**Ollama** will:
- Run AI models directly on your machine
- Provide an API for other tools to use
- Manage model loading/unloading
- Handle all AI inference locally

### Installation Steps

#### Step 1: Download and Install Ollama

**For macOS:**
```bash
# Download and install
curl -fsSL https://ollama.com/install.sh | sh
```

**For Linux:**
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**For Windows:**
1. Visit: https://ollama.com/download
2. Download Windows installer
3. Run installer

**Expected time:** 2 minutes

---

#### Step 2: Start Ollama Service

**Action:**
```bash
# Start Ollama in background
ollama serve > /dev/null 2>&1 &
```

**What this does:**
- Starts Ollama service
- Runs in background (`&`)
- Redirects output to null (keeps terminal clean)

**Verify it's running:**
```bash
curl http://localhost:11434
```

**Expected output:**
```
Ollama is running
```

---

#### Step 3: Download Language Model

**Action:**
```bash
ollama pull llama3.2:3b
```

**What this does:**
- Downloads llama3.2 model (3 billion parameters)
- Stores it locally for future use
- This is the model that answers questions

**Expected output:**
```
pulling manifest
pulling 8eeb52dfb3bb... 100% ▕████████████████▏ 2.0 GB
pulling 73b313b5552d... 100% ▕████████████████▏ 1.4 KB
pulling 0ba8f0e314b4... 100% ▕████████████████▏  12 KB
pulling 56bb8bd477a5... 100% ▕████████████████▏   96 B
pulling 1a4c3c319823... 100% ▕████████████████▏  485 B
verifying sha256 digest
writing manifest
success
```

**Time:** 5-10 minutes (depends on internet speed)  
**File size:** ~2GB

---

#### Step 4: Download Embedding Model

**Action:**
```bash
ollama pull nomic-embed-text
```

**What this does:**
- Downloads embedding model
- This model converts text to vectors
- Required for document search

**Expected output:**
```
pulling manifest
pulling 0a109f422b47... 100% ▕████████████████▏ 274 MB
pulling 5f291a30d295... 100% ▕████████████████▏  426 B
pulling 88cce50c2536... 100% ▕████████████████▏  682 B
pulling f02dd72bb242... 100% ▕████████████████▏   30 B
verifying sha256 digest
writing manifest
success
```

**Time:** 2-3 minutes  
**File size:** ~274MB

---

#### Step 5: Verify Models

**Action:**
```bash
ollama list
```

**Expected output:**
```
NAME                     ID              SIZE      MODIFIED
llama3.2:3b             8eeb52dfb3bb    2.0 GB    X minutes ago
nomic-embed-text:latest 0a109f422b47    274 MB    X minutes ago
```

---

#### Step 6: Test Language Model

**Action:**
```bash
ollama run llama3.2:3b "What is 2+2?"
```

**Expected output:**
```
2+2 equals 4.
```

**Note:** First run takes 10-15 seconds (loading model). Subsequent runs are faster (3-5 seconds).

**To exit:** Type `/bye` or press Ctrl+D

---

#### Step 7: Test Embedding Model

**Action:**
```bash
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "The sky is blue"
}'
```

**Expected output:**
```json
{
  "embedding": [0.123, -0.456, 0.789, ... ] // 768 numbers
}
```

**What this means:** The model successfully converted text into a vector (list of numbers).

---

### Success Checkpoint

You've successfully installed Ollama when:
- ✓ `ollama list` shows both models
- ✓ Language model responds to test question
- ✓ Embedding model returns vectors
- ✓ `curl http://localhost:11434` returns "Ollama is running"

---

## Part 4: Setting Up ChromaDB

**Time:** 10 minutes  
**What you'll do:** Create data directory, start ChromaDB container, verify it works

### Understanding What We're Installing

**ChromaDB** will:
- Store document content as vectors
- Enable fast semantic search
- Persist data across restarts
- Provide REST API for queries

### Important Version Note

We're using **ChromaDB version 0.4.24** specifically because:
- It has the complete v1 API
- It's stable and well-tested
- Newer versions have incomplete APIs
- All our workflows are built for this version

---

### Installation Steps

#### Step 1: Create Data Directory

**Action:**
```bash
# Create project directory
mkdir -p ~/cisco-rag-demo/chroma-data

# Set permissions
chmod 777 ~/cisco-rag-demo/chroma-data
```

**Why this matters:**
- `~/cisco-rag-demo/chroma-data` - Stores all your vectors and documents
- `chmod 777` - Ensures container can read/write (required for Podman on macOS)
- Data persists even if container is deleted

**Verify:**
```bash
ls -la ~/cisco-rag-demo/
```

**Expected output:**
```
drwxrwxrwx  chroma-data
```

---

#### Step 2: Start ChromaDB Container

**Action:**
```bash
podman run -d \
  --name chromadb \
  -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  -e ANONYMIZED_TELEMETRY=FALSE \
  chromadb/chroma:0.4.24
```

**What each part means:**
- `-d` - Run in background (detached)
- `--name chromadb` - Name the container "chromadb"
- `-p 8000:8000` - Map port 8000 (host) to 8000 (container)
- `-v ~/cisco-rag-demo/chroma-data:/chroma/chroma` - Mount local folder into container
- `-e IS_PERSISTENT=TRUE` - Enable data persistence
- `-e ANONYMIZED_TELEMETRY=FALSE` - Disable telemetry
- `chromadb/chroma:0.4.24` - Use specific version

**Expected output:**
```
Trying to pull docker.io/chromadb/chroma:0.4.24...
Getting image source signatures
Copying blob...
...
Writing manifest to image destination
Storing signatures
[40-character container ID]
```

**Time:** 2-3 minutes (first time, includes download)

---

#### Step 3: Verify Container Running

**Action:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE                          COMMAND     CREATED        STATUS        PORTS                   NAMES
abc123def456  chromadb/chroma:0.4.24         uvicorn...  X seconds ago  Up X seconds  0.0.0.0:8000->8000/tcp  chromadb
```

**What to look for:**
- STATUS should be "Up X seconds"
- PORTS should show 8000->8000

---

#### Step 4: Test ChromaDB API

**Wait a moment for startup:**
```bash
sleep 10
```

**Test heartbeat:**
```bash
curl http://localhost:8000/api/v1/heartbeat
```

**Expected output:**
```json
{"nanosecond heartbeat":1234567890123456789}
```

**What this means:** ChromaDB is running and responding to API requests.

---

#### Step 5: Create Test Collection

**Action:**
```bash
curl -X POST http://localhost:8000/api/v1/collections \
  -H "Content-Type: application/json" \
  -d '{
    "name": "cisco_docs",
    "metadata": {"description": "Cisco network documentation"}
  }'
```

**Expected output:**
```json
{
  "name": "cisco_docs",
  "id": "12345678-1234-1234-1234-123456789abc",
  "metadata": {"description": "Cisco network documentation"}
}
```

**What this does:** Creates a "collection" - think of it as a database table for storing document vectors.

---

#### Step 6: Verify Collection Exists

**Action:**
```bash
curl http://localhost:8000/api/v1/collections
```

**Expected output:**
```json
[
  {
    "name": "cisco_docs",
    "id": "12345678-1234-1234-1234-123456789abc",
    "metadata": {"description": "Cisco network documentation"}
  }
]
```

---

### Success Checkpoint

You've successfully set up ChromaDB when:
- ✓ `podman ps` shows chromadb container running
- ✓ Heartbeat endpoint responds
- ✓ Can create collections
- ✓ Data directory exists: `~/cisco-rag-demo/chroma-data`

---

## Part 5: Setting Up n8n

**Time:** 10 minutes  
**What you'll do:** Create data directory, start n8n container, access web interface

### Understanding What We're Installing

**n8n** will:
- Provide visual workflow builder
- Connect Ollama, ChromaDB, and other services
- Enable document upload workflows
- Create chat interfaces
- No coding required

---

### Installation Steps

#### Step 1: Create Data Directory

**Action:**
```bash
# Create n8n data directory
mkdir -p ~/cisco-rag-demo/n8n-data

# Set permissions
chmod 777 ~/cisco-rag-demo/n8n-data
```

**Why:** Stores your workflows, credentials, and settings

**Verify:**
```bash
ls -la ~/cisco-rag-demo/
```

**Expected output:**
```
drwxrwxrwx  chroma-data
drwxrwxrwx  n8n-data
```

---

#### Step 2: Start n8n Container

**Action:**
```bash
podman run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  -e N8N_HOST=localhost \
  -e N8N_PORT=5678 \
  -e N8N_PROTOCOL=http \
  -e WEBHOOK_URL=http://localhost:5678/ \
  docker.n8n.io/n8nio/n8n:latest
```

**What each part means:**
- `-d` - Run in background
- `--name n8n` - Name the container "n8n"
- `-p 5678:5678` - Map port 5678
- `-v ~/cisco-rag-demo/n8n-data:/home/node/.n8n` - Mount data directory
- `-e N8N_HOST=localhost` - Set hostname
- `-e WEBHOOK_URL=http://localhost:5678/` - Set webhook base URL

**Expected output:**
```
Trying to pull docker.io/n8nio/n8n:latest...
[Download progress]
[Container ID]
```

**Time:** 3-5 minutes (first time, includes download)

---

#### Step 3: Wait for Startup

**Action:**
```bash
# Wait for n8n to fully start
sleep 30

# Check if it's running
podman logs n8n --tail 20
```

**Expected output (in logs):**
```
n8n ready on 0.0.0.0:5678
Editor is now accessible via:
http://localhost:5678/
```

---

#### Step 4: Access n8n Web Interface

**Action:**
1. Open browser
2. Go to: `http://localhost:5678`

**Expected result:**
- n8n welcome page appears
- Prompts for email/password setup

**First-time setup:**
1. **Email:** Enter any email (can be fake, only for local login)
2. **Password:** Choose a secure password
3. Click "Get started"

**What you see:**
- Clean interface with "+" button to create workflows
- Side menu with workflows, credentials, executions
- Welcome screen with tutorials

---

#### Step 5: Test n8n Connectivity

**Create a simple test workflow:**

1. Click "+" to create new workflow
2. Click "Add node"
3. Search for "Webhook"
4. Select "Webhook" node
5. Configure:
   - HTTP Method: GET
   - Path: test
6. Click "Test step" or "Listen for test event"
7. Open new browser tab
8. Visit: `http://localhost:5678/webhook-test/test`

**Expected result:**
- n8n shows "Webhook received!"
- Browser shows JSON response

**What this proves:** n8n can receive HTTP requests and execute workflows.

---

#### Step 6: Verify Both Containers

**Action:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE                      CREATED        STATUS        PORTS
abc123...     chromadb/chroma:0.4.24    X minutes ago  Up X minutes  0.0.0.0:8000->8000/tcp
def456...     n8n:latest                X minutes ago  Up X minutes  0.0.0.0:5678->5678/tcp
```

**What to verify:**
- Both containers show "Up" status
- Correct ports mapped
- No restart loops

---

### Success Checkpoint

You've successfully set up n8n when:
- ✓ `podman ps` shows n8n container running
- ✓ Can access web interface at `http://localhost:5678`
- ✓ Completed first-time setup
- ✓ Can create and test workflows

---

## Part 6: Python Setup

**Time:** 10 minutes  
**What you'll do:** Install Python libraries for quick testing

### Understanding Python Tools

While n8n provides visual workflows, Python scripts are useful for:
- Quick testing
- Debugging
- One-off operations
- Learning how the APIs work

**Note:** You don't need to be a Python expert - just copy/paste the commands.

---

### Installation Steps

#### Step 1: Check Python Installation

**Action:**
```bash
python3 --version
```

**Expected output:**
```
Python 3.X.X
```

**If Python not installed:**
- macOS: `brew install python3`
- Linux: `sudo apt install python3 python3-pip`
- Windows: Download from python.org

---

#### Step 2: Install Required Libraries

**Action:**
```bash
pip3 install requests chromadb==0.4.24
```

**What this installs:**
- `requests` - For making HTTP API calls
- `chromadb==0.4.24` - ChromaDB Python client (specific version)

**Expected output:**
```
Collecting requests
Collecting chromadb==0.4.24
[Download and installation progress]
Successfully installed requests-X.X.X chromadb-0.4.24 [and dependencies]
```

**Time:** 2-3 minutes

---

#### Step 3: Create Test Script

**Action:**
```bash
# Create script directory
mkdir -p ~/cisco-rag-demo/scripts

# Create test script
cat > ~/cisco-rag-demo/scripts/test_connection.py << 'EOF'
#!/usr/bin/env python3
"""
Test script to verify all services are accessible
"""
import requests
import sys

def test_ollama():
    """Test Ollama API"""
    try:
        response = requests.get("http://localhost:11434")
        if response.status_code == 200:
            print("✓ Ollama: Running")
            return True
        else:
            print("✗ Ollama: Not responding")
            return False
    except Exception as e:
        print(f"✗ Ollama: Error - {e}")
        return False

def test_chromadb():
    """Test ChromaDB API"""
    try:
        response = requests.get("http://localhost:8000/api/v1/heartbeat")
        if response.status_code == 200:
            print("✓ ChromaDB: Running")
            return True
        else:
            print("✗ ChromaDB: Not responding")
            return False
    except Exception as e:
        print(f"✗ ChromaDB: Error - {e}")
        return False

def test_n8n():
    """Test n8n web interface"""
    try:
        response = requests.get("http://localhost:5678", timeout=5)
        if response.status_code == 200:
            print("✓ n8n: Running")
            return True
        else:
            print("✗ n8n: Not responding")
            return False
    except Exception as e:
        print(f"✗ n8n: Error - {e}")
        return False

def main():
    print("Testing all services...\n")
    
    results = []
    results.append(test_ollama())
    results.append(test_chromadb())
    results.append(test_n8n())
    
    print("\n" + "="*50)
    if all(results):
        print("✓ All services are running!")
        return 0
    else:
        print("✗ Some services are not running")
        print("Run 'podman ps' to check container status")
        return 1

if __name__ == "__main__":
    sys.exit(main())
EOF

# Make executable
chmod +x ~/cisco-rag-demo/scripts/test_connection.py
```

---

#### Step 4: Test the Script

**Action:**
```bash
python3 ~/cisco-rag-demo/scripts/test_connection.py
```

**Expected output:**
```
Testing all services...

✓ Ollama: Running
✓ ChromaDB: Running
✓ n8n: Running

==================================================
✓ All services are running!
```

**If any service shows error:**
- Check container status: `podman ps`
- Check Ollama: `curl http://localhost:11434`
- Restart services if needed

---

#### Step 5: Create System Check Script

**Action:**
```bash
cat > ~/cisco-rag-demo/system-check.sh << 'EOF'
#!/bin/bash
# Quick system check script

echo "=========================================="
echo "System Status Check"
echo "=========================================="
echo ""

echo "1. Podman Machine Status:"
podman machine list
echo ""

echo "2. Running Containers:"
podman ps
echo ""

echo "3. Ollama Status:"
curl -s http://localhost:11434 || echo "Ollama not running"
echo ""

echo "4. ChromaDB Status:"
curl -s http://localhost:8000/api/v1/heartbeat | head -c 50 || echo "ChromaDB not running"
echo ""

echo "5. n8n Status:"
curl -s http://localhost:5678 | grep -q "n8n" && echo "n8n: Running" || echo "n8n: Not running"
echo ""

echo "6. Ollama Models:"
ollama list
echo ""

echo "=========================================="
echo "Detailed Python Test:"
echo "=========================================="
python3 ~/cisco-rag-demo/scripts/test_connection.py
EOF

chmod +x ~/cisco-rag-demo/system-check.sh
```

**Test it:**
```bash
~/cisco-rag-demo/system-check.sh
```

---

### Success Checkpoint

You've successfully set up Python tools when:
- ✓ Python libraries installed
- ✓ Test script runs successfully
- ✓ All services respond to Python requests
- ✓ System check script works

---

## Part 7: Final Verification

**Time:** 5 minutes  
**What you'll do:** Complete system test, verify everything works together

### Comprehensive System Check

#### Step 1: Run System Check Script

**Action:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Verify all sections show "Running" or "✓"**

---

#### Step 2: Test Complete Workflow

**Test embedding generation:**
```bash
curl -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nomic-embed-text",
    "prompt": "This is a test document about network switches"
  }' | head -c 200
```

**Expected:** JSON response with embedding array

---

#### Step 3: Test ChromaDB Storage

**Add a test document:**
```bash
curl -X POST http://localhost:8000/api/v1/collections/cisco_docs/add \
  -H "Content-Type: application/json" \
  -d '{
    "ids": ["test-doc-1"],
    "documents": ["Network switches enable connectivity between devices"],
    "metadatas": [{"source": "test"}]
  }'
```

**Query the document:**
```bash
curl -X POST http://localhost:8000/api/v1/collections/cisco_docs/query \
  -H "Content-Type: application/json" \
  -d '{
    "query_texts": ["What enables device connectivity?"],
    "n_results": 1
  }'
```

**Expected:** Returns the document we just added

---

#### Step 4: Verify Data Persistence

**Action:**
```bash
# Restart ChromaDB container
podman restart chromadb

# Wait for startup
sleep 10

# Check if collection still exists
curl http://localhost:8000/api/v1/collections
```

**Expected:** Collection "cisco_docs" still exists (proves data persists)

---

### Final Checklist

Verify everything before proceeding:

**Services:**
- [ ] Podman machine running (`podman machine list`)
- [ ] Ollama service running (`curl http://localhost:11434`)
- [ ] ChromaDB container running (`podman ps`)
- [ ] n8n container running (`podman ps`)

**Functionality:**
- [ ] Ollama models downloaded (`ollama list` shows 2 models)
- [ ] ChromaDB responds to API calls
- [ ] n8n web interface accessible
- [ ] Python test script passes

**Accessibility:**
- [ ] Can access ChromaDB API (`curl http://localhost:8000/api/v1/heartbeat`)
- [ ] Can access n8n web interface (`http://localhost:5678`)
- [ ] Can access Ollama API (`curl http://localhost:11434`)

**Data:**
- [ ] AI models downloaded (`ollama list` shows 2 models)
- [ ] ChromaDB collection exists (`cisco_docs` visible)
- [ ] Python environment ready (requests library installed)

**✓ If all boxes are checked, your environment is pilot-ready!**

---

## Troubleshooting

### Common Issues After System Restart

#### Issue: All containers stopped

**This is normal.** Podman doesn't auto-start after restart.

**Fix:**
```bash
# Start Podman machine first
podman machine start

# Then start containers
podman start chromadb n8n

# Verify
podman ps
```

---

#### Issue: Ollama not responding

**Cause:** Ollama doesn't auto-start either

**Fix:**
```bash
# Start Ollama service
ollama serve > /dev/null 2>&1 &

# Wait a moment
sleep 5

# Verify
curl http://localhost:11434
```

---

### Performance Issues

#### Issue: Everything is slow (>20 seconds per query)

**Possible causes:**
1. Low RAM - System is using swap memory
2. Other apps consuming resources
3. Models unloading/reloading frequently

**Check RAM usage:**
```bash
# On macOS: Open Activity Monitor
# Check "Memory" tab
# Look at "Memory Pressure"

# On Linux:
free -h
```

**If Memory Pressure is high:**
- Close other applications
- Restart system
- Consider reducing Podman memory allocation:
  ```bash
  podman machine stop
  podman machine set --memory 8192  # Reduce from 12GB to 8GB
  podman machine start
  ```

---

#### Issue: Ollama takes forever to respond (first query)

**This is normal!**
- First query: 10-15 seconds (loading model into RAM)
- Subsequent queries: 3-5 seconds (model stays in memory)

**If ALL queries are slow (>15 seconds):**
- Model is unloading between queries
- Not enough RAM available
- Close other applications

---

### Data Recovery

#### Issue: Lost all my documents

**Possible causes:**
1. ChromaDB container deleted
2. Volume data directory deleted
3. Using wrong collection name

**Check if data exists:**
```bash
ls -la ~/cisco-rag-demo/chroma-data/
```

**If empty:** Data is gone, no recovery possible. You'll need to:
1. Recreate collection
2. Re-load all documents
3. Consider implementing backups (see below)

**Backup strategy:**
```bash
# Create backup script
cat > ~/cisco-rag-demo/backup.sh << 'BACKUP'
#!/bin/bash
DATE=$(date +%Y%m%d)
tar -czf ~/chroma-backup-$DATE.tar.gz ~/cisco-rag-demo/chroma-data/
echo "Backup created: ~/chroma-backup-$DATE.tar.gz"
BACKUP

chmod +x ~/cisco-rag-demo/backup.sh

# Run weekly
# Add to crontab if desired
```

---

### Getting Help

**Before asking for help, gather:**

1. **System info:**
   ```bash
   # macOS:
   system_profiler SPHardwareDataType | grep -A 5 "Hardware Overview"
   sw_vers
   
   # Linux:
   uname -a
   lsb_release -a
   ```

2. **Service status:**
   ```bash
   ~/cisco-rag-demo/system-check.sh
   ```

3. **Container logs:**
   ```bash
   podman logs chromadb > ~/chromadb-logs.txt
   podman logs n8n > ~/n8n-logs.txt
   ```

4. **Error messages:**
   - Copy full error text
   - Note what you were doing when error occurred
   - Note what you tried already

5. **Configuration:**
   ```bash
   podman machine list
   podman ps -a
   ollama list
   ```

**Where to get help:**
1. Re-read relevant guide section
2. Search this troubleshooting section
3. Check the [Comprehensive Troubleshooting Guide](TROUBLESHOOTING.md)
4. Review [Lessons Learned Chronicle](LESSONS_LEARNED_Troubleshooting_Chronicle.md)

---

## Next Steps

🎉 **Congratulations!** You've completed the environment setup.

**What you've accomplished:**
- Installed and configured Podman for container management
- Set up Ollama with AI models for local processing
- Deployed ChromaDB for document storage and search
- Deployed n8n for visual workflow automation
- Configured Python for quick testing scripts
- Verified everything works together

**Your foundation is solid. Now you're ready to build on it.**

---

### What's Next: Part 2 - RAG System

**In the next guide, you'll:**
1. **Create Python scripts** for document loading and querying
2. **Load your first document** into ChromaDB
3. **Test basic queries** using command line
4. **Build n8n workflows** for:
   - Document upload interface
   - Interactive chat interface
   - REST API endpoint
5. **Understand RAG architecture** in depth

**Time required:** ~30 minutes  
**Difficulty:** Beginner-friendly (follow along)

---

### Important Reminders

**After system restart:**
```bash
podman machine start
ollama serve > /dev/null 2>&1 &
podman start chromadb n8n
sleep 20
~/cisco-rag-demo/system-check.sh
```

**Daily system check:**
```bash
~/cisco-rag-demo/system-check.sh
```

**If something breaks:**
1. Don't panic
2. Check logs: `podman logs [container-name]`
3. Try restarting: `podman restart [container-name]`
4. Verify permissions: `ls -la ~/cisco-rag-demo/`
5. Consult troubleshooting section

---

⚡ **Ready to build your RAG system? Proceed to [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)**
