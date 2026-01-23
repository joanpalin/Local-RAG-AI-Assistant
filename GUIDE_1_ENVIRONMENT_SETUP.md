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

**What makes this guide different:** Every instruction has been tested on actual hardware. All common issues have been identified and documented. If you follow the steps exactly as written, you'll have a working system.

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

**Note:** This guide focuses on macOS with Apple Silicon, but the principles apply to other platforms with appropriate adjustments.

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

**Important v1 API Change:**
ChromaDB 0.4.24 uses the v1 API which has different requirements than older versions:
- Collections are accessed by **UUID** (unique identifier), not by name
- All operations require **pre-computed embeddings** (you must generate vectors before adding/querying)
- This is different from older tutorials which use collection names directly

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
│                                                         │
│  1. Upload Document                                     │
│     ↓                                                   │
│  2. n8n breaks into chunks                              │
│     ↓                                                   │
│  3. Ollama converts to vectors (embeddings)             │
│     ↓                                                   │
│  4. ChromaDB stores vectors                             │
│     ↓                                                   │
│  5. Query: "What's the budget?"                         │
│     ↓                                                   │
│  6. Ollama converts query to vector                     │
│     ↓                                                   │
│  7. ChromaDB finds similar vectors                      │
│     ↓                                                   │
│  8. Ollama generates answer                             │
│     ↓                                                   │
│  9. "The network refresh costs $250,000"                │
│                                                         │
│  ALL PROCESSING STAYS ON YOUR MACHINE                   │
│  NO DATA SENT TO EXTERNAL SERVERS                       │
└─────────────────────────────────────────────────────────┘
```

**Privacy Note:** Everything runs locally. Your documents never leave your machine (unless you explicitly add Webex integration later).

---

## Part 2: Installing Podman

**Time:** 10 minutes  
**What you'll do:** Download Podman, initialize virtual machine, verify it works

### Understanding What We're Installing

**Podman Desktop** provides:
- Easy-to-use graphical interface
- Container management
- Image management
- Built-in terminal

**Podman Machine** (the backend):
- Runs containers
- Manages resources
- Provides isolation

---

### Installation Steps

#### Step 1: Download and Install

**For macOS:**
1. Visit: https://podman.io/getting-started/installation
2. Download "Podman Desktop" for macOS
3. Open the downloaded `.dmg` file
4. Drag Podman to Applications folder
5. Open Podman from Applications

**For Linux:**
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install podman

# Fedora/RHEL
sudo dnf install podman
```

**For Windows:**
1. Visit: https://podman.io/getting-started/installation
2. Download Podman Desktop for Windows
3. Run installer
4. Follow installation wizard

**Expected time:** 5 minutes

---

#### Step 2: Initialize Podman Machine

**Action:**
```bash
# Initialize Podman virtual machine
podman machine init --cpus 4 --memory 12288 --disk-size 50

# Start the machine
podman machine start
```

**What these commands do:**
- Creates a lightweight VM to run containers
- Allocates 4 CPU cores
- Allocates 12GB RAM (12288 MB)
- Creates 50GB virtual disk

**Expected output:**
```
Downloading VM image...
Extracting compressed file...
Machine init complete
Starting machine "podman-machine-default"...
Machine "podman-machine-default" started successfully
```

**Time:** 3-5 minutes (downloads ~500MB image)

---

#### Step 3: Verify Installation

**Action:**
```bash
# Check Podman version
podman --version

# Check machine status
podman machine list

# Verify we can run containers
podman ps
```

**Expected output:**
```
podman version 4.X.X

NAME                     VM TYPE     CREATED     LAST UP              CPUS   MEMORY      DISK SIZE
podman-machine-default*  libkrun     X min ago   Currently running    4      12GiB       50GiB

CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

**What this means:**
- First command: Shows Podman version (confirms installation)
- Second command: Shows machine is running
- Third command: Lists running containers (should be empty now)

---

### Success Checkpoint

You've successfully installed Podman when:
- ✓ `podman --version` shows a version number
- ✓ `podman machine list` shows a running machine
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

---

### Installation Steps

#### Step 1: Download and Install Ollama

**⚠️ IMPORTANT: Different installation methods for different operating systems**

**For macOS (Option 1 - Recommended):**
```bash
# Open the download page in your browser
open https://ollama.com/download

# Steps:
# 1. Click "Download for macOS"
# 2. Wait for .zip file to download
# 3. Double-click the .zip file to extract
# 4. Drag Ollama.app to Applications folder
# 5. Open Ollama from Applications (it will appear in menu bar)
```

**For macOS (Option 2 - Homebrew):**
```bash
# If you have Homebrew installed
brew install ollama

# Verify installation
ollama --version
```

**For Linux:**
```bash
# This script works on Linux only
curl -fsSL https://ollama.com/install.sh | sh
```

**For Windows:**
1. Visit: https://ollama.com/download
2. Download Windows installer
3. Run installer
4. Follow installation wizard

**Expected time:** 2-5 minutes

**⚠️ Common Error:** If you're on macOS and try the Linux command (`curl -fsSL https://ollama.com/install.sh | sh`), you'll get an error: "This script is intended to run on Linux only." Use the macOS options above instead.

---

#### Step 2: Start Ollama Service

**For macOS (if you downloaded manually):**
- Ollama starts automatically when you open it from Applications
- You'll see the Ollama icon in your menu bar at the top of the screen
- No command needed!

**For macOS/Linux (if you installed via Homebrew or script):**
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
- **CRITICAL:** You MUST have this model for ChromaDB operations

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

**To exit interactive mode:** Type `/bye` or press Ctrl+D

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

**What this means:** The model successfully converted text into a vector (list of 768 numbers). This is the foundation of semantic search.

**⚠️ IMPORTANT:** You'll use this embedding API extensively when working with ChromaDB, as the v1 API requires pre-computed embeddings for all operations.

---

### Success Checkpoint

You've successfully installed Ollama when:
- ✓ `ollama list` shows both models (llama3.2:3b and nomic-embed-text)
- ✓ Language model responds to test question
- ✓ Embedding model returns vectors (array of 768 numbers)
- ✓ `curl http://localhost:11434` returns "Ollama is running"

---

## Part 4: Setting Up ChromaDB

**Time:** 10 minutes  
**What you'll do:** Create data directory, start ChromaDB container, create collection, verify it works

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

**⚠️ CRITICAL v1 API Changes:**
ChromaDB 0.4.24 uses the v1 API which works differently from older versions:
1. **UUID-based access:** Collections must be accessed by UUID, not name
2. **Embeddings required:** You must provide pre-computed embeddings for ALL operations (add, query)
3. **No auto-embedding:** Unlike older versions, ChromaDB will not generate embeddings automatically

This means every workflow follows this pattern:
```
Text → Ollama (generate embedding) → ChromaDB (store/query with embedding)
```

---

### Installation Steps

#### Step 1: Create Data Directory

**Action:**
```bash
# Create project directory
mkdir -p ~/cisco-rag-demo/chroma-data

# Set permissions (required for Podman on macOS)
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

#### Step 2: Create Container Network

**Action:**
```bash
# Create custom network for container communication
podman network create rag-network
```

**Why this matters:**
- Allows containers to communicate with each other
- Provides DNS resolution between containers
- Isolates our containers from other projects

**Expected output:**
```
rag-network
```

**Verify:**
```bash
podman network ls
```

---

#### Step 3: Start ChromaDB Container

**Action:**
```bash
podman run -d \
  --name chromadb \
  --network rag-network \
  -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma:Z \
  -e IS_PERSISTENT=TRUE \
  -e ANONYMIZED_TELEMETRY=FALSE \
  docker.io/chromadb/chroma:0.4.24
```

**What each line does:**
- `-d` - Run in background (detached mode)
- `--name chromadb` - Name the container
- `--network rag-network` - Connect to our custom network
- `-p 8000:8000` - Map port 8000 (host:container)
- `-v ~/cisco-rag-demo/chroma-data:/chroma/chroma:Z` - Mount data directory
- `-e IS_PERSISTENT=TRUE` - Enable data persistence
- `-e ANONYMIZED_TELEMETRY=FALSE` - Disable telemetry
- `docker.io/chromadb/chroma:0.4.24` - Specific version to download

**Expected output:**
```
[Container ID - long string of characters]
```

**Time:** 2-3 minutes (downloads ~500MB image on first run)

---

#### Step 4: Verify ChromaDB is Running

**Action:**
```bash
# Check container status
podman ps

# Test API
curl http://localhost:8000/api/v1/heartbeat
```

**Expected output:**
```
# From podman ps:
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS        PORTS                   NAMES
[ID]          docker.io/chromadb/chroma:0.4.24          X seconds ago  Up X seconds  0.0.0.0:8000->8000/tcp  chromadb

# From curl:
{"nanosecond heartbeat":1234567890123456789}
```

**If you see the heartbeat response, ChromaDB is working!**

---

#### Step 5: Create Collection

**⚠️ IMPORTANT:** This step is crucial and different from older tutorials.

**Action:**
```bash
# Create collection
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
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "metadata": {"description": "Cisco network documentation"},
  "tenant": "default_tenant",
  "database": "default_database"
}
```

**⚠️ CRITICAL:** Save the `"id"` value from the response! This is your collection UUID. You'll need it for ALL future operations.

---

#### Step 6: Get Collection UUID

**Action:**
```bash
# List all collections to get UUID
curl http://localhost:8000/api/v1/collections | jq

# Store UUID in environment variable for easy access
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
echo "Your collection UUID: $COLLECTION_UUID"
```

**Expected output:**
```json
[
  {
    "name": "cisco_docs",
    "id": "b787cfbb-e6c8-4ade-833c-e8ca6ce5c275",
    "metadata": {"description": "Cisco network documentation"},
    "tenant": "default_tenant",
    "database": "default_database"
  }
]

Your collection UUID: b787cfbb-e6c8-4ade-833c-e8ca6ce5c275
```

**💡 Pro Tip:** Write down your collection UUID or save it somewhere. You'll use it in all your workflows and scripts.

---

#### Step 7: Verify Collection Count

**Action:**
```bash
# Check collection count (should be 0 initially)
curl http://localhost:8000/api/v1/collections/$COLLECTION_UUID/count
```

**Expected output:**
```
0
```

**⚠️ Note:** If you try `GET /api/v1/collections/{uuid}` (without /count), it may return an error saying "Collection does not exist." This is a known quirk in ChromaDB 0.4.24 - the endpoint doesn't work reliably. Use the `/count` endpoint or the list endpoint instead.

---

### Success Checkpoint

You've successfully set up ChromaDB when:
- ✓ `podman ps` shows chromadb container running
- ✓ Heartbeat API responds with JSON
- ✓ Collection created and UUID obtained
- ✓ Count endpoint returns 0 (empty collection ready for data)
- ✓ Data directory created with correct permissions

---

## Part 5: Setting Up n8n

**Time:** 10 minutes  
**What you'll do:** Start n8n container, access web interface, complete setup

### Understanding What We're Installing

**n8n** will:
- Provide visual workflow builder
- Connect Ollama and ChromaDB
- Handle document processing
- Provide web interface for uploads

---

### Installation Steps

#### Step 1: Create n8n Data Directory

**Action:**
```bash
# Create directory for n8n data
mkdir -p ~/cisco-rag-demo/n8n-data

# Set permissions
chmod 777 ~/cisco-rag-demo/n8n-data
```

---

#### Step 2: Start n8n Container

**Action:**
```bash
podman run -d \
  --name n8n \
  --network rag-network \
  -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n:Z \
  -e N8N_HOST=localhost \
  -e N8N_PORT=5678 \
  -e N8N_PROTOCOL=http \
  -e WEBHOOK_URL=http://localhost:5678/ \
  docker.n8n.io/n8nio/n8n:latest
```

**What each line does:**
- `-d` - Run in background
- `--name n8n` - Name the container
- `--network rag-network` - Connect to same network as ChromaDB
- `-p 5678:5678` - Map web interface port
- `-v ~/cisco-rag-demo/n8n-data:/home/node/.n8n:Z` - Persist workflows
- Environment variables - Configure n8n URLs

**Expected output:**
```
[Container ID]
```

**Time:** 2-3 minutes (downloads image on first run)

---

#### Step 3: Verify n8n is Running

**Action:**
```bash
# Check container status
podman ps

# Wait for startup (n8n takes ~20 seconds to fully start)
sleep 20
```

**Expected output:**
```
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS        PORTS                   NAMES
[ID]          docker.n8n.io/n8nio/n8n:latest            X seconds ago  Up X seconds  0.0.0.0:5678->5678/tcp  n8n
[ID]          docker.io/chromadb/chroma:0.4.24          X minutes ago  Up X minutes  0.0.0.0:8000->8000/tcp  chromadb
```

---

#### Step 4: Access n8n Web Interface

**Action:**
```bash
# Open n8n in your browser
open http://localhost:5678

# Or manually navigate to: http://localhost:5678
```

**Expected:** Browser opens showing n8n setup page

---

#### Step 5: Complete n8n First-Time Setup

**Actions in browser:**

1. **Create account:**
   - Email: (your choice, can be fake for local use)
   - Password: (choose a secure password)
   - Click "Next"

2. **Setup preferences:**
   - Workflow name: "Local RAG Assistant"
   - Industry: "Technology"
   - Click "Next"

3. **Skip additional setup:**
   - Click "Skip" or "Continue to n8n"

4. **You should now see the n8n canvas:**
   - Blank workspace with "+" button
   - Left sidebar with nodes
   - This is where you'll build workflows

---

#### Step 6: Test n8n

**Action:** Create a simple test workflow

1. Click the "+" button on canvas
2. Search for "Schedule Trigger"
3. Add "Schedule Trigger" node
4. Click "+" after it
5. Search for "Set" node
6. Add "Set" node
7. Click "Execute Workflow" (top right)

**Expected:** Workflow executes successfully, shows green checkmarks

---

### Success Checkpoint

You've successfully set up n8n when:
- ✓ `podman ps` shows n8n container running
- ✓ Can access web interface at `http://localhost:5678`
- ✓ Completed first-time setup
- ✓ Can create and test workflows
- ✓ Both containers (chromadb and n8n) are running

---

## Part 6: Python Setup

**Time:** 10-15 minutes  
**What you'll do:** Install Python libraries for testing, handle multiple Python installations

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
# Check if Python is installed
python3 --version

# Check which Python will be used by scripts
/usr/bin/env python3 --version

# Check where Python is installed
which python3
/usr/bin/env python3 -c "import sys; print(sys.executable)"
```

**Expected output:**
```
Python 3.X.X
Python 3.X.X
/usr/bin/python3  (or /opt/homebrew/bin/python3)
/opt/homebrew/opt/python@3.XX/bin/python3.XX  (or similar)
```

**⚠️ IMPORTANT:** If the version numbers are different or the paths are different, you have multiple Python installations. This is common on macOS.

**If Python not installed:**
- macOS: `brew install python3`
- Linux: `sudo apt install python3 python3-pip`
- Windows: Download from python.org

---

#### Step 2: Determine Correct Python for Package Installation

**⚠️ CRITICAL:** This step addresses the most common installation issue.

**Problem:** On macOS, you often have multiple Python installations:
- Apple's Python (old, from Command Line Tools)
- Homebrew's Python (newer, preferred)
- Scripts use `#!/usr/bin/env python3` which finds the first Python in PATH

**Solution:** Install packages to the Python that scripts actually use.

**Action:**
```bash
# Find which Python scripts will use
SCRIPT_PYTHON=$(/usr/bin/env python3 -c "import sys; print(sys.executable)")
echo "Scripts will use: $SCRIPT_PYTHON"

# Check Python version
/usr/bin/env python3 --version
```

**Expected output:**
```
Scripts will use: /opt/homebrew/opt/python@3.14/bin/python3.14
Python 3.14.2
```

---

#### Step 3: Install Required Libraries

**⚠️ For Python 3.12+ (Modern Homebrew Python):**

Modern Python versions have PEP 668 protection that prevents global package installation. You'll need to use the `--break-system-packages` flag for development/demo environments.

**Action:**
```bash
# Try standard installation first
python3 -m pip install requests

# If you get "externally-managed-environment" error, use this:
python3 -m pip install requests --break-system-packages

# Verify installation
python3 -c "import requests; print('✓ requests installed correctly')"
```

**⚠️ For Python 3.9-3.11 (Older Python):**

If you have an older Python version:

**Action:**
```bash
# Standard installation (should work without flags)
python3 -m pip install requests

# Verify
python3 -c "import requests; print('✓ requests installed correctly')"
```

**What this installs:**
- `requests` - For making HTTP API calls to Ollama and ChromaDB

**Expected output (with --break-system-packages):**
```
Collecting requests
Downloading requests-X.X.X-py3-none-any.whl
Installing collected packages: certifi, charset-normalizer, idna, urllib3, requests
Successfully installed requests-X.X.X ...

✓ requests installed correctly
```

**Expected output (without flag, older Python):**
```
Collecting requests
[Download progress]
Successfully installed requests-X.X.X

✓ requests installed correctly
```

**⚠️ If you get "no such option: --break-system-packages" error:**
Your Python is too old (< 3.12) and doesn't need the flag. Just use: `python3 -m pip install requests`

**💡 Why `--break-system-packages` is safe here:**
- This is a development/demo environment on your local machine
- Not a production server
- We're not installing system-critical packages
- The flag allows installation to Homebrew Python without creating conflicts

---

#### Step 4: Handle Multiple Python Installations (If Needed)

**If your test script still can't find `requests` after installation:**

**Diagnostic:**
```bash
# Check where pip3 installs to
pip3 --version

# Check where python3 runs from
which python3

# Check where /usr/bin/env python3 points
/usr/bin/env python3 --version
which -a python3  # Show ALL python3 installations
```

**If they point to different Python installations:**

**Solution Option 1 - Install to the correct Python:**
```bash
# Use the specific Python that scripts use
/usr/bin/env python3 -m pip install requests --break-system-packages

# Verify
/usr/bin/env python3 -c "import requests; print('✓ Success!')"
```

**Solution Option 2 - Use pip from the correct Python:**
```bash
# If you have Homebrew Python
/opt/homebrew/bin/python3 -m pip install requests --break-system-packages

# Verify
/opt/homebrew/bin/python3 -c "import requests; print('✓ Success!')"
```

---

#### Step 5: Create Test Script

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

#### Step 6: Test the Script

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

**⚠️ If you get "ModuleNotFoundError: No module named 'requests'":**

This means the packages weren't installed to the right Python. Go back to Step 3 and use the `python3 -m pip` commands with `--break-system-packages` flag.

**Quick fix:**
```bash
# Install directly with the Python that scripts use
/usr/bin/env python3 -m pip install requests --break-system-packages

# Try again
python3 ~/cisco-rag-demo/scripts/test_connection.py
```

---

#### Step 7: Create System Check Script

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

**Expected output:**
```
==========================================
System Status Check
==========================================

1. Podman Machine Status:
NAME                     VM TYPE     CREATED     LAST UP              CPUS   MEMORY   DISK SIZE
podman-machine-default*  libkrun     X min ago   Currently running    4      12GiB    50GiB

2. Running Containers:
CONTAINER ID  IMAGE                             COMMAND  CREATED     STATUS      PORTS                   NAMES
[ID]          docker.io/chromadb/chroma:0.4.24          X min ago   Up X min    0.0.0.0:8000->8000/tcp  chromadb
[ID]          docker.n8n.io/n8nio/n8n:latest            X min ago   Up X min    0.0.0.0:5678->5678/tcp  n8n

3. Ollama Status:
Ollama is running

4. ChromaDB Status:
{"nanosecond heartbeat":1234567890123456789}

5. n8n Status:
n8n: Running

6. Ollama Models:
NAME                       ID              SIZE      MODIFIED
llama3.2:3b               8eeb52dfb3bb    2.0 GB    X minutes ago
nomic-embed-text:latest   0a109f422b47    274 MB    X minutes ago

==========================================
Detailed Python Test:
==========================================
Testing all services...

✓ Ollama: Running
✓ ChromaDB: Running
✓ n8n: Running

==================================================
✓ All services are running!
```

---

### Success Checkpoint

You've successfully set up Python tools when:
- ✓ Python 3.9+ installed and accessible
- ✓ `requests` library installed to correct Python
- ✓ Test script runs without "ModuleNotFoundError"
- ✓ All services respond to Python requests
- ✓ System check script shows all green checkmarks

**Common issues resolved:**
- ✓ Multiple Python installations handled
- ✓ PEP 668 externally-managed-environment bypassed for dev use
- ✓ Correct Python identified for script execution

---

## Part 7: Final Verification

**Time:** 10 minutes  
**What you'll do:** Test complete workflow with embeddings, verify everything works together

### Comprehensive System Check

#### Step 1: Run System Check Script

**Action:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Verify all sections show "Running" or "✓"**

If anything shows errors, troubleshoot that component before proceeding.

---

#### Step 2: Test Embedding Generation

**⚠️ CRITICAL:** This tests the foundation of all RAG operations.

**Action:**
```bash
curl -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nomic-embed-text",
    "prompt": "This is a test document about network switches"
  }' | head -c 200
```

**Expected output:**
```json
{"embedding":[0.123,-0.456,0.789,...]}
```

**What this proves:**
- Ollama is working
- Embedding model is loaded
- You can generate vectors from text

---

#### Step 3: Get Collection UUID

**⚠️ REQUIRED:** You need the collection UUID for all ChromaDB operations.

**Action:**
```bash
# Get collection UUID
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
echo "Collection UUID: $COLLECTION_UUID"

# Save it for this session
export COLLECTION_UUID
```

**Expected output:**
```
Collection UUID: b787cfbb-e6c8-4ade-833c-e8ca6ce5c275
```

**💡 Write this UUID down!** You'll need it for testing and later guides.

---

#### Step 4: Test Complete Add Operation (with Embeddings)

**⚠️ CRITICAL:** ChromaDB v1 API requires embeddings for add operations.

**Action:**
```bash
# Generate embedding for document
DOC_EMBEDDING=$(curl -s -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "nomic-embed-text", "prompt": "Network switches enable connectivity between devices"}' | jq -c '.embedding')

# Add document with embedding to ChromaDB
curl -X POST http://localhost:8000/api/v1/collections/$COLLECTION_UUID/add \
  -H "Content-Type: application/json" \
  -d "{
    \"ids\": [\"test-doc-1\"],
    \"documents\": [\"Network switches enable connectivity between devices\"],
    \"embeddings\": [$DOC_EMBEDDING],
    \"metadatas\": [{\"source\": \"test\"}]
  }"
```

**Expected output:**
```
true
```

**What this proves:**
- You can generate embeddings
- You can add documents to ChromaDB with embeddings
- Data is being stored

**⚠️ Common mistake:** Trying to add without embeddings will fail silently or cause errors in queries.

---

#### Step 5: Verify Document Count

**Action:**
```bash
# Check how many documents are in collection
curl -s http://localhost:8000/api/v1/collections/$COLLECTION_UUID/count
```

**Expected output:**
```
1
```

**This proves the document was successfully added.**

---

#### Step 6: Test Complete Query Operation (with Embeddings)

**⚠️ CRITICAL:** ChromaDB v1 API requires embeddings for query operations.

**Action:**
```bash
# Generate embedding for query
QUERY_EMBEDDING=$(curl -s -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "nomic-embed-text", "prompt": "What enables device connectivity?"}' | jq -c '.embedding')

# Query ChromaDB with embedding
curl -X POST http://localhost:8000/api/v1/collections/$COLLECTION_UUID/query \
  -H "Content-Type: application/json" \
  -d "{\"query_embeddings\": [$QUERY_EMBEDDING], \"n_results\": 1}" | jq
```

**Expected output:**
```json
{
  "ids": [
    [
      "test-doc-1"
    ]
  ],
  "distances": [
    [
      124.72855022742878
    ]
  ],
  "metadatas": [
    [
      {
        "source": "test"
      }
    ]
  ],
  "embeddings": null,
  "documents": [
    [
      "Network switches enable connectivity between devices"
    ]
  ],
  "uris": null,
  "data": null
}
```

**What this proves:**
- Semantic search is working!
- Query "What enables device connectivity?" found document about "Network switches enable connectivity"
- The words are different but the meaning is similar (semantic matching)
- Distance score shows similarity (lower = more similar)

---

#### Step 7: Test Complete Workflow (All-in-One)

**For convenience, here's a one-line command that does everything:**

**Action:**
```bash
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id') && \
QUERY_EMBEDDING=$(curl -s -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "nomic-embed-text", "prompt": "What enables device connectivity?"}' | jq -c '.embedding') && \
curl -X POST http://localhost:8000/api/v1/collections/$COLLECTION_UUID/query \
  -H "Content-Type: application/json" \
  -d "{\"query_embeddings\": [$QUERY_EMBEDDING], \"n_results\": 1}" | jq
```

**This command:**
1. Gets collection UUID
2. Generates query embedding
3. Queries ChromaDB
4. Returns results

**Save this pattern!** This is the core workflow you'll use repeatedly.

---

#### Step 8: Verify Data Persistence

**Action:**
```bash
# Restart ChromaDB container
podman restart chromadb

# Wait for startup
sleep 10

# Check if collection still exists
curl -s http://localhost:8000/api/v1/collections | jq

# Check if document count persists
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
curl -s http://localhost:8000/api/v1/collections/$COLLECTION_UUID/count
```

**Expected output:**
```json
[
  {
    "name": "cisco_docs",
    "id": "b787cfbb-e6c8-4ade-833c-e8ca6ce5c275",
    ...
  }
]

1
```

**What this proves:**
- Data persists across container restarts
- Your storage setup is working correctly
- You won't lose data if containers restart

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
- [ ] Can generate embeddings (Step 2 successful)
- [ ] Can add documents with embeddings (Step 4 returns `true`)
- [ ] Can query with embeddings (Step 6 returns documents)
- [ ] Document count accurate (Step 5 shows 1)

**Data:**
- [ ] Collection created with valid UUID
- [ ] Test document added successfully
- [ ] Query returns correct document
- [ ] Data persists after container restart

**Accessibility:**
- [ ] Can access ChromaDB API (`curl http://localhost:8000/api/v1/heartbeat`)
- [ ] Can access n8n web interface (`http://localhost:5678`)
- [ ] Can access Ollama API (`curl http://localhost:11434`)
- [ ] Python test script passes

**✓ If all boxes are checked, your environment is production-ready!**

---

### Important Notes on ChromaDB v1 API

**What works:**
- ✅ `GET /api/v1/collections` - List all collections
- ✅ `POST /api/v1/collections/{uuid}/add` - Add documents (with embeddings)
- ✅ `POST /api/v1/collections/{uuid}/query` - Query documents (with embeddings)
- ✅ `GET /api/v1/collections/{uuid}/count` - Count documents

**What doesn't work (known bug in 0.4.24):**
- ❌ `GET /api/v1/collections/{uuid}` - Get individual collection details (returns "does not exist" error)

**Workaround:** Always use the list endpoint (`GET /api/v1/collections`) and filter client-side.

---

## Troubleshooting

### Common Issues After System Restart

#### Issue: All containers stopped

**This is normal.** Podman doesn't auto-start after restart.

**Fix:**
```bash
# Start Podman machine first
podman machine start

# Wait for machine to fully start
sleep 10

# Then start containers
podman start chromadb n8n

# Verify
podman ps
```

---

#### Issue: Ollama not responding

**Cause:** Ollama doesn't auto-start either (if installed via Homebrew/script)

**Fix for macOS (manual installation):**
- Open Ollama from Applications
- Check menu bar for Ollama icon

**Fix for Homebrew/script installation:**
```bash
# Start Ollama service
ollama serve > /dev/null 2>&1 &

# Wait a moment
sleep 5

# Verify
curl http://localhost:11434
```

---

#### Issue: Python script shows "ModuleNotFoundError: No module named 'requests'"

**Cause:** Packages installed to wrong Python installation

**Fix:**
```bash
# Check which Python scripts use
/usr/bin/env python3 --version
/usr/bin/env python3 -c "import sys; print(sys.executable)"

# Install to correct Python with flag for modern Python
python3 -m pip install requests --break-system-packages

# Or for older Python (if above fails)
python3 -m pip install requests

# Verify
python3 -c "import requests; print('✓ Success!')"
```

---

#### Issue: ChromaDB query returns empty results

**Cause:** Documents weren't added with embeddings, or wrong collection UUID

**Fix:**
```bash
# Verify collection UUID
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
echo "UUID: $COLLECTION_UUID"

# Check document count
curl -s http://localhost:8000/api/v1/collections/$COLLECTION_UUID/count

# If count is 0, re-add document with embedding (see Step 4 in Part 7)
```

---

#### Issue: curl command fails with "Could not parse cisco_docs as a UUID"

**Cause:** Trying to use collection name instead of UUID

**Fix:**
```bash
# Get the actual UUID
curl http://localhost:8000/api/v1/collections | jq

# Use the "id" field (not "name") in your commands
COLLECTION_UUID="paste-uuid-here"
```

---

#### Issue: externally-managed-environment error when installing Python packages

**Cause:** Modern Python (3.12+) protects against global package installation

**Fix:**
```bash
# For development/demo use, override protection
python3 -m pip install requests --break-system-packages

# Verify
python3 -c "import requests; print('✓ Installed')"
```

**Alternative (proper way for production):**
```bash
# Create virtual environment
python3 -m venv ~/cisco-rag-demo/venv

# Activate it
source ~/cisco-rag-demo/venv/bin/activate

# Install packages
pip install requests

# Update scripts to use venv Python
# (requires editing shebang lines)
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
3. Using wrong collection UUID

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
   python3 --version
   /usr/bin/env python3 --version
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
- Configured Python for testing scripts
- Verified everything works together
- Successfully tested the complete RAG workflow with embeddings

**Your foundation is solid. Now you're ready to build on it.**

---

### What's Next: Part 2 - RAG System

**In the next guide, you'll:**
1. **Create Python scripts** for document loading and querying (with proper UUID and embedding handling)
2. **Load your first real document** into ChromaDB
3. **Test semantic search** using command line
4. **Build n8n workflows** for:
   - Document upload interface (with automatic embedding generation)
   - Interactive chat interface
   - REST API endpoint
5. **Understand RAG architecture** in depth

**Time required:** ~30-45 minutes  
**Difficulty:** Beginner-friendly (follow along)

**Important reminders for next guide:**
- Always get collection UUID before operations
- Always generate embeddings before add/query operations
- Use the two-step pattern: Text → Embedding → ChromaDB
- Your collection UUID: (write yours here: _________________)

---

### Important Reminders

**After system restart:**
```bash
# 1. Start Podman machine
podman machine start

# 2. Start Ollama (if needed)
# macOS manual install: Open from Applications
# Homebrew install:
ollama serve > /dev/null 2>&1 &

# 3. Start containers
podman start chromadb n8n

# 4. Wait for services
sleep 20

# 5. Verify everything
~/cisco-rag-demo/system-check.sh
```

**Daily system check:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Get your collection UUID:**
```bash
curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id'
```

**If something breaks:**
1. Don't panic
2. Check logs: `podman logs [container-name]`
3. Try restarting: `podman restart [container-name]`
4. Verify permissions: `ls -la ~/cisco-rag-demo/`
5. Check if using correct collection UUID
6. Verify embeddings are being generated
7. Consult troubleshooting section

---

### Key Patterns to Remember

**Pattern 1: Get Collection UUID**
```bash
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
```

**Pattern 2: Generate Embedding**
```bash
EMBEDDING=$(curl -s -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "nomic-embed-text", "prompt": "your text here"}' | jq -c '.embedding')
```

**Pattern 3: Add with Embedding**
```bash
curl -X POST http://localhost:8000/api/v1/collections/$COLLECTION_UUID/add \
  -H "Content-Type: application/json" \
  -d "{\"ids\": [\"id\"], \"documents\": [\"text\"], \"embeddings\": [$EMBEDDING]}"
```

**Pattern 4: Query with Embedding**
```bash
curl -X POST http://localhost:8000/api/v1/collections/$COLLECTION_UUID/query \
  -H "Content-Type: application/json" \
  -d "{\"query_embeddings\": [$EMBEDDING], \"n_results\": 1}"
```

---

⚡ **Ready to build your RAG system? Proceed to [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)**

---

## Appendix: Quick Reference

### Essential Commands

**Start everything:**
```bash
podman machine start
ollama serve > /dev/null 2>&1 &  # if needed
podman start chromadb n8n
```

**Check status:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Stop everything:**
```bash
podman stop chromadb n8n
podman machine stop
pkill ollama  # if needed
```

**Get collection UUID:**
```bash
curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id'
```

**Count documents:**
```bash
COLLECTION_UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[0].id')
curl -s http://localhost:8000/api/v1/collections/$COLLECTION_UUID/count
```

---

### Service URLs

- **Ollama API:** http://localhost:11434
- **ChromaDB API:** http://localhost:8000
- **n8n Interface:** http://localhost:5678

---

### Important Files

- **Project directory:** `~/cisco-rag-demo/`
- **ChromaDB data:** `~/cisco-rag-demo/chroma-data/`
- **n8n data:** `~/cisco-rag-demo/n8n-data/`
- **Scripts:** `~/cisco-rag-demo/scripts/`
- **System check:** `~/cisco-rag-demo/system-check.sh`

---

**End of Guide 1: Environment Setup**
