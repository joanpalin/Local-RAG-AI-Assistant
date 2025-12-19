# Guide 1: Environment Setup
## Building Your Local AI Foundation

**Part 1 of 3: Implementation Guides**  
**Time Required:** 45-60 minutes  
**Difficulty:** Beginner-friendly (step-by-step)  
**Prerequisites:** Mac with Apple Silicon, 16GB+ RAM

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

Welcome! This guide will help you set up the foundation for your local AI document assistant. By the end of this guide, you'll have all the necessary software installed and verified working on your Mac.

**What makes this guide different:** Every instruction has been tested on actual hardware (Mac M4 Pro). All common issues have been identified and documented. If you follow the steps exactly as written, you'll have a working system.

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

âž¡ï¸ **Ready? Let's build your AI assistant!**

---

## What You'll Accomplish

By completing this guide, you will have:

âœ… **Podman** - Container runtime (like Docker) running virtual environments  
âœ… **Ollama** - AI model manager with two models downloaded  
âœ… **ChromaDB** - Document storage system that AI can search  
âœ… **n8n** - Visual workflow builder (no coding needed)  
âœ… **Python setup** - For quick testing scripts  

**What this enables:**
- Process documents locally (no data leaves your Mac)
- Run AI models without internet connection
- Build visual workflows without writing code
- Query documents through multiple interfaces

---

## System Requirements

### Hardware Requirements

**Minimum (Will work, but slower):**
- Mac with Apple Silicon (M1 or newer)
- 16GB RAM
- 50GB free disk space
- macOS Monterey (12.0) or later

**Recommended (Better performance):**
- Mac with M2, M3, or M4 chip
- 24GB+ RAM
- 100GB free disk space
- macOS Sonoma (14.0) or later

### Pre-Installation Checklist

Before you begin, verify:

- [ ] You have administrator access to your Mac
- [ ] You can open Terminal (Applications > Utilities > Terminal)
- [ ] You have an internet connection for downloads
- [ ] You have 45-60 minutes of uninterrupted time
- [ ] You've read this introduction

ðŸ'¡ **Tip:** Close unnecessary applications to free up RAM before starting.

---

## Part 1: Understanding the Components

**Time:** 10 minutes (reading)

Before installing anything, let's understand what each component does and why we need it. Think of this like understanding the parts of a network before cabling everything together.

### Component 1: Podman

**What it is:** A tool that runs software in isolated "containers"  
**Why we need it:** ChromaDB and n8n need consistent environments  
**Real-world analogy:** Like a virtual machine, but lighter weight

**Think of it this way:** 
Imagine you're a network engineer setting up a demo in a conference room. Instead of bringing physical servers, you bring virtual machines on a laptop. Podman does something similar - it runs "virtual computers" (containers) inside your Mac, each with its own isolated environment.

**Why containers matter:**
- Consistent behavior (works the same every time)
- Easy to start/stop
- Doesn't conflict with other software on your Mac
- Easy to remove if needed (no leftover files)

**Example:** 
If ChromaDB needs Python 3.9 but your Mac has Python 3.11, no problem - the container has its own Python 3.9 that doesn't interfere with your system's Python.

**Why Podman instead of Docker?**
- Free and open source (no licensing issues)
- Better security model (doesn't need root)
- Works well on Apple Silicon Macs
- Similar commands to Docker (easy to learn)

---

### Component 2: Ollama

**What it is:** Software that runs AI language models on your Mac  
**Why we need it:** Provides the "intelligence" - reads and understands documents  
**Real-world analogy:** Like having a really smart colleague who never sleeps

**Think of it this way:**
Remember when network monitoring meant someone physically checking lights on routers? Then we got SNMP agents that automated the monitoring. Ollama is similar - instead of sending questions to a remote AI service, you have an AI "agent" running locally on your Mac that can read documents and answer questions.

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
[User uploads document] â†' [Break into chunks] â†' [Convert to vectors] â†' [Store in ChromaDB]
```

Each box is a "node" - you configure what it does, then connect the arrows.

**Why n8n specifically?**
- Self-hosted (runs on your Mac, no cloud service needed)
- Free and open source
- Large library of pre-built nodes
- Active community
- Great documentation

**For network engineers:** Think of n8n nodes like CLI commands in a script - each node does one thing, and you chain them together. But instead of writing bash scripts, you're building visual diagrams.

---

### How It All Works Together

Here's the complete picture:

```
Your Mac
â"‚
â"œâ"€ Ollama (running natively on macOS)
â"‚   â"œâ"€ llama3.2:3b model (answers questions)
â"‚   â""â"€ nomic-embed-text model (converts text to vectors)
â"‚
â""â"€ Podman (container engine)
    â"‚
    â"œâ"€ ChromaDB container (vector database)
    â"‚   â""â"€ Stores your documents as searchable vectors
    â"‚
    â""â"€ n8n container (workflow automation)
        â""â"€ Coordinates everything
```

**Example Query Flow:**
1. **You ask:** "What's the datacenter budget?"
2. **n8n receives** your question via web interface
3. **Ollama (embedding model)** converts question to vector
4. **ChromaDB searches** for similar vectors
5. **ChromaDB returns** relevant document chunks
6. **Ollama (language model)** reads chunks and writes answer
7. **n8n shows** answer to you

**All of this happens in 5-10 seconds**, completely on your Mac, with no data sent to the internet.

---

### Why This Architecture?

**Separation of concerns:**
- Ollama = AI capabilities (runs on your Mac for performance)
- ChromaDB = Data storage (runs in container for isolation)
- n8n = Workflow orchestration (runs in container for easy management)
- Podman = Container runtime (manages isolated environments)

**Benefits:**
- âœ… Each component does one thing well
- âœ… Easy to update individual pieces
- âœ… Failure in one doesn't break others
- âœ… Can replace components if needed

**Network analogy:** Just like how you separate routing, switching, and security into different devices rather than putting everything on one box, we separate AI models, data storage, and workflow orchestration into different components.

---

## Part 2: Installing Podman

**Time:** 15 minutes  
**What you'll do:** Install Podman and create a virtual machine for running containers  
**Why:** Provides the foundation for ChromaDB and n8n containers

---

### Background: How Podman Works on macOS

**Important to understand:**
Containers are a Linux technology. Since your Mac runs macOS (not Linux), Podman creates a tiny Linux virtual machine behind the scenes. This VM runs inside your Mac and hosts the containers.

**What this means for you:**
- Podman commands work just like on Linux
- Small startup time when you first start Podman (~5 seconds)
- VM needs CPU, RAM, and disk space (we'll allocate these)

**Don't worry:** This all happens automatically. You won't see the VM - you'll just use simple Podman commands.

---

### Step 2.1: Install Homebrew (If Not Already Installed)

**What is Homebrew?**  
A package manager for macOS - think of it as an app store for command-line tools. It makes installing software like Podman much easier.

**Check if you already have it:**
```bash
# Run this command in Terminal
brew --version
```

**If you see a version number:** âœ… Skip to Step 2.2  
**If you see "command not found:"** Continue below

**Install Homebrew:**
```bash
# Copy and paste this entire command:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**What this does:**
- Downloads the Homebrew installer
- Installs Homebrew to your Mac
- Sets up necessary paths

**During installation:**
- You'll be prompted for your Mac password
- Installation takes 5-10 minutes
- You'll see lots of text scroll by (this is normal)

**Verify installation:**
```bash
brew --version
```

**Expected output:**
```
Homebrew 4.x.x
```

âœ… **Success checkpoint:** Homebrew version displays

---

### Step 2.2: Install Podman

**Now install Podman using Homebrew:**
```bash
brew install podman
```

**What this does:**
- Downloads Podman software
- Installs it to your Mac
- Sets up the `podman` command

**This takes:** 3-5 minutes

**During installation, you'll see:**
```
==> Downloading podman...
==> Installing podman...
==> Summary
🍺  /opt/homebrew/Cellar/podman/5.x.x: XX files, XXMB
```

**Verify installation:**
```bash
podman --version
```

**Expected output:**
```
podman version 5.x.x
```

ðŸ'¡ **Tip:** Exact version doesn't matter as long as it's 5.0 or higher.

âœ… **Success checkpoint:** Podman version displays

---

### Step 2.3: Initialize the Podman Machine

**What's happening:** Creating the Linux virtual machine that will run your containers.

**Think of it like:** Provisioning a new virtual server - you need to specify how many CPUs, how much RAM, and how much disk space it gets.

**Create the machine:**
```bash
podman machine init --cpus 6 --memory 12288 --disk-size 60
```

**What each parameter means:**

- `--cpus 6` → Allocate 6 CPU cores to the VM
  - **Why 6?** Leaves cores for macOS and other apps
  - Your Mac likely has 8-12 cores, so 6 is a good balance
  
- `--memory 12288` → Allocate 12GB of RAM (12288 MB)
  - **Why 12GB?** Enough for ChromaDB + n8n + AI models
  - If you have 16GB total: 12GB for VM, 4GB for macOS
  - If you have 24GB total: 12GB for VM, 12GB for macOS
  
- `--disk-size 60` → Allocate 60GB of disk space
  - **Why 60GB?** Space for containers, AI models, and your documents
  - **This isn't immediately used** - it's a maximum limit
  - Actual usage starts around 10GB

**During initialization:**
```
Downloading VM image: fedora-coreos...
Extracting compressed file
Creating machine podman-machine-default
```

**This takes:** 5-10 minutes (downloading the Linux VM image)

**Expected completion message:**
```
Machine "podman-machine-default" has been successfully initialized
```

âœ… **Success checkpoint:** Initialization completes without errors

---

### Step 2.4: Start the Podman Machine

**Start the virtual machine:**
```bash
podman machine start
```

**What this does:**
- Boots the Linux VM
- Sets up networking
- Prepares to run containers

**During startup:**
```
Starting machine "podman-machine-default"
API forwarding listening on: /var/run/...
...
Machine "podman-machine-default" started successfully
```

**This takes:** 15-30 seconds

**Expected completion message:**
```
Machine "podman-machine-default" started successfully
```

ðŸ'¡ **Good to know:** After a Mac restart, you'll need to run `podman machine start` again. The VM doesn't auto-start.

---

### Step 2.5: Verify Podman is Working

**Check that everything is running:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

**What this means:**
- Empty list = Good! (We haven't created any containers yet)
- If you see an error, see Troubleshooting section

**Check machine status:**
```bash
podman machine list
```

**Expected output:**
```
NAME                     VM TYPE     CREATED      LAST UP            CPUS        MEMORY      DISK SIZE
podman-machine-default   qemu        X mins ago   Currently running  6           12.00GB     60.00GB
```

**What to verify:**
- âœ… "Currently running" status
- âœ… CPUs: 6
- âœ… Memory: 12.00GB
- âœ… Disk: 60.00GB

---

### Understanding Podman Commands (Quick Reference)

You'll use these commands frequently:

```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# Start a stopped container
podman start [container-name]

# Stop a running container
podman stop [container-name]

# View container logs (for troubleshooting)
podman logs [container-name]

# Remove a container
podman rm [container-name]

# Remove an image
podman rmi [image-name]
```

**Network analogy:** Think of Podman commands like CLI commands on a switch:
- `podman ps` = `show interfaces status` (what's running?)
- `podman logs` = `show logging` (what happened?)
- `podman stop/start` = `shutdown/no shutdown` (control state)

---

### Step 2.6: Success Checkpoint

Before moving to the next section, verify:

- [ ] Podman is installed (`podman --version` works)
- [ ] Podman machine exists (`podman machine list` shows it)
- [ ] Machine is running (status shows "Currently running")
- [ ] Can run podman commands (`podman ps` works, even if empty)

âœ… **If all checkboxes are checked, you're ready for Ollama!**

---

### Troubleshooting Podman Installation

#### Issue: "podman: command not found"

**Cause:** Homebrew didn't add Podman to your PATH

**Fix:**
```bash
# Add Homebrew to PATH
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Try again
podman --version
```

---

#### Issue: Machine won't start - "port already in use"

**Cause:** Previous Podman installation or Docker is using the same ports

**Fix:**
```bash
# Stop any Docker desktop
# Then try starting Podman again
podman machine start
```

---

#### Issue: "Machine already exists"

**Cause:** You ran `podman machine init` twice

**Fix:**
```bash
# Remove existing machine
podman machine rm podman-machine-default

# Create fresh machine
podman machine init --cpus 6 --memory 12288 --disk-size 60
podman machine start
```

---

#### Issue: Low disk space warning

**Cause:** Mac doesn't have 60GB free

**Fix Option 1:** Free up space on your Mac  
**Fix Option 2:** Use smaller disk allocation
```bash
podman machine rm podman-machine-default
podman machine init --cpus 6 --memory 12288 --disk-size 40
podman machine start
```

ðŸ'¡ **Note:** 40GB is minimum - system will be tight on space.

---

## Part 3: Installing Ollama

**Time:** 10 minutes  
**What you'll do:** Install Ollama and download two AI models  
**Why:** Provides the AI "intelligence" for reading and answering questions

---

### Background: What Ollama Does

Ollama is software that:
1. **Downloads AI models** from the internet (one-time)
2. **Runs them locally** on your Mac
3. **Provides an API** that other software (like n8n) can use
4. **Manages memory** - loads/unloads models as needed

**Unlike ChatGPT or Claude:**
- No internet required after initial download
- No monthly fees
- No data sent to external servers
- Runs on your Mac's hardware

**Performance:**
- First question: 10-15 seconds (loading model into RAM)
- Subsequent questions: 5-8 seconds
- Perfectly acceptable for document queries

---

### Step 3.1: Install Ollama

**Install using Homebrew:**
```bash
brew install ollama
```

**What this does:**
- Downloads Ollama software (small, ~50MB)
- Installs it to your Mac
- Sets up the `ollama` command

**This takes:** 1-2 minutes

**Verify installation:**
```bash
ollama --version
```

**Expected output:**
```
ollama version is 0.x.x
```

âœ… **Success checkpoint:** Ollama version displays

---

### Step 3.2: Start the Ollama Service

**Important:** Ollama needs to run as a background service.

**Start Ollama:**
```bash
ollama serve &
```

**What this does:**
- Starts Ollama in the background
- The `&` keeps it running even when you close Terminal
- Ollama listens on port 11434 for requests

**During startup:**
```
Ollama is running
```

**Verify it's running:**
```bash
curl http://localhost:11434
```

**Expected output:**
```
Ollama is running
```

ðŸ'¡ **Good to know:** 
- After Mac restart, run `ollama serve &` again
- Or, it will auto-start when you first use it
- You can safely run `ollama serve &` multiple times (it just says "already running")

---

### Step 3.3: Download AI Models

**Now for the bigger downloads - the actual AI models.**

**Download the language model (answers questions):**
```bash
ollama pull llama3.2:3b
```

**What's happening:**
- Downloading llama 3.2 model with 3 billion parameters
- Size: ~2GB
- Time: 3-7 minutes (depending on internet speed)

**During download:**
```
pulling manifest
pulling 6a0746a1ec1a... 100% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ 2.0 GB                          
pulling 4fa551d4f938... 100% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ 1.4 KB                          
...
success
```

**Expected completion:** "success"

---

**Download the embedding model (converts text to vectors):**
```bash
ollama pull nomic-embed-text
```

**What's happening:**
- Downloading nomic embedding model
- Size: ~274MB
- Time: 30-90 seconds

**During download:**
```
pulling manifest
pulling 0a109f422b47... 100% â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ 274 MB
...
success
```

**Expected completion:** "success"

---

### Step 3.4: Verify Models Are Downloaded

**List installed models:**
```bash
ollama list
```

**Expected output:**
```
NAME                    ID              SIZE      MODIFIED
llama3.2:3b            abc123def...    2.0 GB    X minutes ago
nomic-embed-text       def456ghi...    274 MB    X minutes ago
```

**What to verify:**
- âœ… Both models listed
- âœ… Sizes approximately match (2.0GB and 274MB)
- âœ… Modified time is recent

---

### Step 3.5: Test the Models (Optional but Recommended)

**Test the language model:**
```bash
ollama run llama3.2:3b "What is the capital of France?"
```

**What this does:**
- Loads llama3.2 into memory (first time only, 5-10 seconds)
- Sends the question
- Gets an answer
- Unloads after a minute of inactivity

**Expected response:**
```
The capital of France is Paris.
```

**First run timing:**
- First question: 10-15 seconds (loading model)
- Answer generation: 2-3 seconds

**If you run another question immediately:**
- Model stays in memory
- Responds in 3-5 seconds

---

**Test the embedding model:**
```bash
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "hello world"
}'
```

**Expected response (shortened for display):**
```json
{
  "embedding": [0.123, -0.456, 0.789, ... (274 more numbers)]
}
```

**What this means:**
- âœ… Embedding model works
- The numbers are the "vector" representation of "hello world"
- You won't normally see these - ChromaDB uses them automatically

---

### Understanding Model Sizes and Performance

**llama3.2:3b (2GB):**
- **3b** = 3 billion parameters (the "intelligence")
- **Smaller than ChatGPT** (175B parameters) but perfectly capable
- **Tradeoff:** Smaller = faster on your Mac, less complex reasoning
- **Perfect for:** Document Q&A, summarization, information extraction

**Think of it like:**
- ChatGPT = Core datacenter router (massive capacity, handles everything)
- llama3.2:3b = Branch router (right-sized for specific needs)

**nomic-embed-text (274MB):**
- **Specialized model** - only does one thing (text → vectors)
- **Fast** - converts text in milliseconds
- **Essential** - without this, document search wouldn't work

---

### Step 3.6: Success Checkpoint

Before moving to the next section, verify:

- [ ] Ollama is installed (`ollama --version` works)
- [ ] Ollama service is running (`curl http://localhost:11434` responds)
- [ ] Both models downloaded (`ollama list` shows 2 models)
- [ ] Models respond to test queries (optional but recommended)

âœ… **If all checkboxes are checked, you're ready for ChromaDB!**

---

### Troubleshooting Ollama Installation

#### Issue: "ollama: command not found"

**Cause:** Homebrew path issue (same as Podman)

**Fix:**
```bash
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
ollama --version
```

---

#### Issue: Model download fails - "connection reset"

**Cause:** Network interruption or firewall

**Fix:**
```bash
# Try again - Ollama resumes partial downloads
ollama pull llama3.2:3b
```

**If repeatedly fails:**
```bash
# Try different model (might be temporarily unavailable)
ollama pull llama3.1:latest

# Or check firewall settings
```

---

#### Issue: "Error: could not connect to ollama app"

**Cause:** Ollama service not running

**Fix:**
```bash
# Start Ollama service
ollama serve &

# Wait 5 seconds
sleep 5

# Try command again
ollama list
```

---

#### Issue: Model is slow (30+ seconds per query)

**Possible causes:**
1. First query (model loading into RAM) - normal
2. Mac is low on RAM - close other applications
3. Mac is using swap memory - restart Mac

**Check RAM usage:**
```bash
# Open Activity Monitor
# Check "Memory" tab
# If "Memory Pressure" is red, you need to close applications
```

---

## Part 4: Setting Up ChromaDB

**Time:** 15 minutes  
**What you'll do:** Deploy ChromaDB in a container, set permissions, and create your first collection  
**Why:** Provides the "smart filing cabinet" where documents are stored and searched

---

### Background: Understanding ChromaDB

**What ChromaDB does:**
1. **Stores documents** as text chunks with vector representations
2. **Searches quickly** - finds relevant chunks in milliseconds
3. **Returns context** - gives the AI what it needs to answer questions

**Version note (IMPORTANT):**
We use **ChromaDB 0.4.24** specifically (not latest). Here's why:

- **Latest versions** have incomplete v2 APIs (some endpoints return 404 errors)
- **Version 0.4.24** has the complete v1 API - everything works
- **Tested and proven** - this is the version used in actual deployments

ðŸ'¡ **Key insight:** In production, "latest" isn't always best. Pinning to tested versions prevents unexpected breaks.

---

### Step 4.1: Create Project Directory

**Create the directory structure:**
```bash
mkdir -p ~/cisco-rag-demo/{chroma-data,n8n-data,documents}
```

**What this does:**
- Creates `~/cisco-rag-demo/` as the main project folder
- Creates `chroma-data/` for ChromaDB's database files
- Creates `n8n-data/` for n8n's workflow files
- Creates `documents/` for your test documents

**Verify it worked:**
```bash
ls -la ~/cisco-rag-demo/
```

**Expected output:**
```
drwxr-xr-x  5 you  staff  160 Dec 18 10:30 .
drwxr-xr-x  20 you  staff  640 Dec 18 10:30 ..
drwxr-xr-x  2 you  staff   64 Dec 18 10:30 chroma-data
drwxr-xr-x  2 you  staff   64 Dec 18 10:30 documents
drwxr-xr-x  2 you  staff   64 Dec 18 10:30 n8n-data
```

---

### Step 4.2: Set Permissions (CRITICAL for macOS + Podman)

**Why this matters:**
Podman runs containers in a Linux VM. The Linux user inside the container has a different user ID (UID) than your Mac user. This causes permission conflicts when the container tries to write to macOS folders.

**The solution:** Set permissions to 777 (read, write, execute for everyone)

**Is 777 safe?**
- **For local development: YES** - only you have access to your Mac
- **For servers: NO** - but this isn't a server
- **Alternative:** Complex UID mapping (not worth the hassle for local dev)

**Set permissions:**
```bash
chmod -R 777 ~/cisco-rag-demo/chroma-data
chmod -R 777 ~/cisco-rag-demo/n8n-data
```

**What this does:**
- `-R` = Recursive (applies to all files and subdirectories)
- `777` = Full permissions for user, group, and others
- Allows the container to create/modify files

**Verify:**
```bash
ls -la ~/cisco-rag-demo/
```

**Look for:** `drwxrwxrwx` (that's 777 permissions)

ðŸ'¡ **Remember:** If you ever see "permission denied" errors with containers, this is the first thing to check.

---

### Step 4.3: Pull ChromaDB Image (Specific Version)

**Pull the specific working version:**
```bash
podman pull chromadb/chroma:0.4.24
```

**What this does:**
- Downloads ChromaDB container image
- Version 0.4.24 specifically (not latest!)
- Size: ~500MB
- Time: 2-4 minutes

**During download:**
```
Trying to pull chromadb/chroma:0.4.24...
Getting image source signatures
Copying blob 123abc456def done
...
Writing manifest to image destination
```

**Expected completion:**
```
123abc456def789...
```
(This is the image ID - a long hexadecimal string)

**Verify image is downloaded:**
```bash
podman images | grep chroma
```

**Expected output:**
```
docker.io/chromadb/chroma  0.4.24  123abc456def  2 weeks ago  500 MB
```

---

### Step 4.4: Create ChromaDB Container

**Create and start the container:**
```bash
podman run -d \
  --name chromadb \
  -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  -e ANONYMIZED_TELEMETRY=FALSE \
  chromadb/chroma:0.4.24
```

**Let's break down each part:**

`podman run` - Create and start a new container

`-d` - **Detached mode** (runs in background)
  - Container keeps running even after command completes
  - Like running a daemon process

`--name chromadb` - **Name the container** "chromadb"
  - You'll use this name in other commands
  - Easier than remembering container IDs

`-p 8000:8000` - **Port mapping**
  - First 8000: Port on your Mac
  - Second 8000: Port inside container
  - Makes ChromaDB accessible at `http://localhost:8000`

`-v ~/cisco-rag-demo/chroma-data:/chroma/chroma` - **Volume mount**
  - Left side: Folder on your Mac
  - Right side: Folder inside container
  - Database files persist on your Mac (survive container restarts)

`-e IS_PERSISTENT=TRUE` - **Environment variable**
  - Tells ChromaDB to save data to disk (not just RAM)
  - Without this, data disappears when container stops

`-e ANONYMIZED_TELEMETRY=FALSE` - **Environment variable**
  - Disables anonymous usage stats
  - Keeps everything local

`chromadb/chroma:0.4.24` - **Which image to use**
  - Must match what you pulled earlier

---

**During creation:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
```
(This is the container ID - save this just in case)

**Verify container is running:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE                            COMMAND  CREATED         STATUS         PORTS                   NAMES
a1b2c3d4e5f6  docker.io/chromadb/chroma:0.4.24 ...      10 seconds ago  Up 10 seconds  0.0.0.0:8000->8000/tcp  chromadb
```

**What to verify:**
- âœ… STATUS shows "Up"
- âœ… PORTS shows "8000->8000"
- âœ… NAMES shows "chromadb"

---

### Step 4.5: Wait for ChromaDB to Start

**Why wait?**
ChromaDB takes 10-15 seconds to fully initialize after the container starts.

**Wait for startup:**
```bash
sleep 15
```

**Or, check logs to confirm it's ready:**
```bash
podman logs chromadb
```

**Look for these lines:**
```
Starting component ChromaServer
Running migrations...
ChromaDB is ready to use
```

ðŸ'¡ **Tip:** If you see error messages, see Troubleshooting section.

---

### Step 4.6: Verify ChromaDB is Accessible

**Test the API endpoint:**
```bash
curl http://localhost:8000/api/v1/heartbeat
```

**Expected response:**
```json
{
  "nanosecond heartbeat": 1702909876543210000
}
```

**What this means:**
- âœ… ChromaDB is running
- âœ… API is accessible
- âœ… Ready to accept requests

**If you see connection refused:**
```bash
# Check if container is still running
podman ps | grep chromadb

# If not running, check logs
podman logs chromadb

# Look for permission errors
```

---

### Step 4.7: Create Your First Collection

**What's a collection?**
Think of it like a database table or a VLAN - a logical grouping for organizing data. In ChromaDB, each collection stores related documents.

**For this project:**
- **Collection name:** `cisco_docs`
- **Purpose:** Store all your document chunks
- **You could have multiple collections** (e.g., "policies", "assessments", "tickets")

**Create the collection:**
```bash
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'
```

**What each part means:**

`curl -X POST` - Send a POST request  
`http://localhost:8000/api/v1/collections` - ChromaDB collections endpoint  
`-H 'Content-Type: application/json'` - Tell server we're sending JSON  
`-d '{"name":"cisco_docs"}'` - The JSON data (collection name)

---

**Expected response:**
```json
{
  "name": "cisco_docs",
  "id": "12345678-1234-1234-1234-123456789abc",
  "metadata": null,
  "tenant": "default_tenant",
  "database": "default_database"
}
```

**Important parts:**
- `"name": "cisco_docs"` - Your collection name
- `"id": "12345678-1234-1234-1234-123456789abc"` - **The UUID** (unique identifier)

ðŸ'¡ **Key concept:** ChromaDB v1 API uses UUIDs internally, not names. Your scripts will look up the UUID by name automatically - you don't need to memorize it.

---

**Verify collection was created:**
```bash
curl http://localhost:8000/api/v1/collections
```

**Expected response:**
```json
[
  {
    "name": "cisco_docs",
    "id": "12345678-1234-1234-1234-123456789abc",
    "metadata": null,
    "tenant": "default_tenant",
    "database": "default_database"
  }
]
```

**What this shows:**
- Array with one collection
- Your cisco_docs collection
- Ready to store documents

---

### Understanding ChromaDB's v1 API (Important for Later)

**Key insight discovered through troubleshooting:**

ChromaDB v1 API requires **UUIDs** in URLs, not collection names.

**Wrong (doesn't work):**
```
POST /api/v1/collections/cisco_docs/add
```

**Right (works):**
```
POST /api/v1/collections/12345678-1234-1234-1234-123456789abc/add
```

**Don't worry:** All the Python scripts and n8n workflows in this project handle this automatically. They:
1. Look up the UUID by collection name
2. Use the UUID in the API call
3. You just use the friendly "cisco_docs" name

**Why this matters:**
- If you ever get "InvalidUUID" errors, this is why
- The working code in this project solves this automatically
- Just know that UUIDs are being used behind the scenes

---

### Step 4.8: Success Checkpoint

Before moving to the next section, verify:

- [ ] ChromaDB container is running (`podman ps` shows it)
- [ ] Permissions are set correctly (777 on chroma-data folder)
- [ ] ChromaDB API responds (`curl http://localhost:8000/api/v1/heartbeat`)
- [ ] Collection exists (`curl http://localhost:8000/api/v1/collections`)
- [ ] You see "cisco_docs" collection in the response

âœ… **If all checkboxes are checked, you're ready for n8n!**

---

### Troubleshooting ChromaDB Setup

#### Issue: Container stops immediately after starting

**Symptoms:**
```bash
podman ps
# chromadb not listed

podman ps -a
# Shows chromadb with "Exited (1)" status
```

**Most common cause:** Permission denied

**Fix:**
```bash
# Check logs for "permission denied"
podman logs chromadb

# Fix permissions
chmod -R 777 ~/cisco-rag-demo/chroma-data

# Remove failed container
podman rm chromadb

# Recreate container (re-run podman run command from Step 4.4)
```

---

#### Issue: "Connection refused" to port 8000

**Diagnostic:**
```bash
# Is container running?
podman ps | grep chromadb

# Is something else on port 8000?
lsof -i :8000
```

**Fix option 1:** Start the container
```bash
podman start chromadb
sleep 15
curl http://localhost:8000/api/v1/heartbeat
```

**Fix option 2:** Port conflict - use different port
```bash
podman stop chromadb
podman rm chromadb

# Use port 8001 instead
podman run -d \
  --name chromadb \
  -p 8001:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  -e ANONYMIZED_TELEMETRY=FALSE \
  chromadb/chroma:0.4.24

# Test with new port
curl http://localhost:8001/api/v1/heartbeat
```

---

#### Issue: Collection creation fails - "Already exists"

**This is actually fine!** It means you already created it.

**Verify:**
```bash
curl http://localhost:8000/api/v1/collections
# Should show cisco_docs
```

**If you want to delete and recreate:**
```bash
# Get the UUID
UUID=$(curl -s http://localhost:8000/api/v1/collections | jq -r '.[] | select(.name=="cisco_docs") | .id')

# Delete collection
curl -X DELETE http://localhost:8000/api/v1/collections/$UUID

# Create again
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'
```

---

#### Issue: Wrong ChromaDB version - API doesn't work

**Symptoms:**
- 404 errors when using collections
- Missing endpoints

**Check version:**
```bash
podman inspect chromadb | grep Image
```

**Should show:** `chromadb/chroma:0.4.24`

**If wrong version:**
```bash
# Stop and remove
podman stop chromadb
podman rm chromadb

# Pull correct version
podman pull chromadb/chroma:0.4.24

# Recreate (re-run podman run command from Step 4.4)
```

---

## Part 5: Setting Up n8n

**Time:** 10 minutes  
**What you'll do:** Deploy n8n workflow automation platform  
**Why:** Provides visual interface for building AI workflows without coding

---

### Background: What n8n Provides

**n8n is your "no-code" interface for:**
- Building workflows visually (drag and drop)
- Connecting ChromaDB, Ollama, and Webex
- Creating interactive chat interfaces
- Building document upload forms
- Testing and debugging flows

**Think of it like:**
- Visio for network diagrams → n8n for workflow diagrams
- Each box (node) = one action
- Arrows show data flow
- Configure settings visually (no code needed)

---

### Step 5.1: Create n8n Container

**Deploy n8n:**
```bash
podman run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  -e N8N_HOST=localhost \
  -e N8N_PORT=5678 \
  -e N8N_PROTOCOL=http \
  docker.io/n8nio/n8n:latest
```

**Let's break down each part:**

`podman run -d` - Create container in background

`--name n8n` - Name it "n8n"

`-p 5678:5678` - **Port mapping**
  - Access n8n at `http://localhost:5678`
  - Port 5678 is n8n's default

`-v ~/cisco-rag-demo/n8n-data:/home/node/.n8n` - **Volume mount**
  - Workflows and credentials persist on your Mac
  - Survives container restarts

`-e N8N_HOST=localhost` - **Set host** for local access

`-e N8N_PORT=5678` - **Set port** (matches our mapping)

`-e N8N_PROTOCOL=http` - **Use HTTP** (not HTTPS for local dev)

`docker.io/n8nio/n8n:latest` - **Official n8n image**
  - Note: We use latest for n8n (gets updates)
  - Unlike ChromaDB where we pinned a version

---

**During creation:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
```
(Container ID)

**Note:** n8n takes 15-30 seconds to fully start. Be patient!

---

**Verify container is running:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE                        COMMAND  CREATED         STATUS         PORTS                   NAMES
a1b2c3d4e5f6  docker.io/n8nio/n8n:latest  ...      20 seconds ago  Up 20 seconds  0.0.0.0:5678->5678/tcp  n8n
...
```

**What to verify:**
- âœ… STATUS shows "Up"
- âœ… PORTS shows "5678->5678"
- âœ… NAMES shows "n8n"

ðŸ'¡ **If n8n stops immediately:** Check permissions on n8n-data folder (same fix as ChromaDB)

---

### Step 5.2: Wait for n8n to Start

**n8n needs time to initialize:**
```bash
sleep 20
```

**Or watch the logs:**
```bash
podman logs -f n8n
```
(Press Ctrl+C to stop watching)

**Look for:**
```
Version: X.XX.X
Editor is now accessible via:
http://localhost:5678/
```

---

### Step 5.3: Access n8n Web Interface

**Open n8n in your browser:**
```bash
open http://localhost:5678
```

**Or manually navigate to:** `http://localhost:5678`

**What you'll see:**
- Welcome screen
- Setup wizard

---

### Step 5.4: First-Time Setup

**n8n will ask you to create an account.**

**âš ï¸ IMPORTANT - Choose "Local" account:**

1. **Enter your information:**
   - First name, Last name
   - Email (can be anything - not verified)
   - Password (create a secure password)

2. **â›" Skip cloud/sync features:**
   - When asked about cloud sync: Click "Skip" or "Not now"
   - **Why?** For privacy, keep everything local
   - No data should go to n8n's cloud

3. **â›" Decline telemetry:**
   - When asked about usage statistics: Click "No" or "Decline"
   - Keeps your workflows private

**What you should NOT do:**
- Don't create n8n cloud account (not needed)
- Don't enable synchronization (breaks privacy model)
- Don't share workflows to cloud (keep local)

---

### Step 5.5: Quick n8n Interface Tour

After setup, you'll see:

**Left Sidebar:**
- **Workflows** - Your saved workflows
- **Credentials** - Saved API keys (we'll add Webex token later)
- **Executions** - Workflow run history
- **Settings** - Configuration options

**Top Menu:**
- **New** - Create new workflow
- **Templates** - Pre-built workflows (we'll import ours)
- **Help** - Documentation

**Main Canvas:**
- Empty workspace where you'll build workflows
- Drag nodes from left panel
- Connect them with arrows

**Don't worry about learning everything now** - detailed n8n usage comes in Part 2 (RAG System Guide).

---

### Step 5.6: Verify n8n Can Reach Other Services

**Important:** n8n runs in a container, so it needs special URLs to reach other services.

**Network architecture:**
```
n8n container
 â"‚
 â"œâ"€â"€ To Ollama (on Mac): http://host.containers.internal:11434
 â"œâ"€â"€ To ChromaDB (in container): http://chromadb:8000
 â""â"€â"€ To internet: Works normally
```

**Key URLs to remember:**

**From n8n to Ollama:**
- âœ… Use: `http://host.containers.internal:11434`
- âŒ Never use: `http://localhost:11434` (won't work)

**From n8n to ChromaDB:**
- âœ… Use: `http://chromadb:8000`
- Also works: `http://host.containers.internal:8000`

**Why "host.containers.internal"?**
- Special hostname that Podman provides
- Points to your Mac (the host machine)
- Allows containers to access services running on your Mac

**Network analogy:** Think of it like static routes:
- "localhost" = loopback (container talking to itself)
- "host.containers.internal" = default gateway (container talking to host Mac)
- "chromadb" = hostname in same subnet (container to container)

---

### Step 5.7: Success Checkpoint

Before moving to the next section, verify:

- [ ] n8n container is running (`podman ps` shows it)
- [ ] Can access web interface (`http://localhost:5678` opens)
- [ ] Completed first-time setup (created local account)
- [ ] See the n8n workflow canvas

âœ… **If all checkboxes are checked, you're ready for Python setup!**

---

### Troubleshooting n8n Setup

#### Issue: n8n container stops immediately

**Most common cause:** Permission denied (same as ChromaDB)

**Fix:**
```bash
# Check logs
podman logs n8n

# If you see "EACCES: permission denied":
chmod -R 777 ~/cisco-rag-demo/n8n-data

# Remove and recreate
podman rm n8n
# Re-run podman run command from Step 5.1
```

---

#### Issue: Can't access web interface

**Diagnostic:**
```bash
# Is container running?
podman ps | grep n8n

# Can you reach the port?
curl http://localhost:5678
```

**Fix option 1:** Wait longer
```bash
# n8n can take 30+ seconds to start
sleep 30
open http://localhost:5678
```

**Fix option 2:** Check logs
```bash
podman logs n8n
# Look for errors
```

---

#### Issue: Forgot password

**Fix:**
```bash
# Reset by removing n8n database
podman stop n8n
rm ~/cisco-rag-demo/n8n-data/database.sqlite
podman start n8n

# Navigate to http://localhost:5678
# Go through setup again
```

---

## Part 6: Python Setup

**Time:** 5 minutes  
**What you'll do:** Verify Python is installed and add required libraries  
**Why:** Enables quick testing scripts for document loading and querying

---

### Background: Why Python?

While n8n provides visual workflows (which you'll use most of the time), Python scripts are useful for:

- **Quick testing** - Verify system works before building workflows
- **Bulk operations** - Load multiple documents at once
- **Debugging** - Isolate problems to specific components
- **Automation** - Schedule document updates

**You don't need to know Python** - we'll provide complete working scripts in Part 2. This section just sets up the environment.

---

### Step 6.1: Check Python Version

**Your Mac comes with Python pre-installed.** Verify:

```bash
python3 --version
```

**Expected output:**
```
Python 3.9.x
```
or
```
Python 3.10.x
```
or
```
Python 3.11.x
```

**Any Python 3.9 or newer works fine.**

**If you see "command not found":**
- Highly unusual on modern Macs
- Install Python from python.org
- Or: `brew install python3`

âœ… **Success checkpoint:** Python 3.9+ is installed

---

### Step 6.2: Install Required Library

**We need one Python library:** `requests` (for making HTTP calls to ChromaDB and Ollama)

**Install it:**
```bash
pip3 install requests
```

**What this does:**
- Downloads the `requests` library
- Installs it to your Python environment
- Takes 10-30 seconds

**During installation:**
```
Collecting requests
  Downloading requests-2.31.0-py3-none-any.whl
Installing collected packages: requests
Successfully installed requests-2.31.0
```

---

**Verify installation:**
```bash
python3 -c "import requests; print('âœ" requests library installed')"
```

**Expected output:**
```
âœ" requests library installed
```

**If you see "No module named 'requests'":**
```bash
# Try with sudo (sometimes needed)
sudo pip3 install requests

# Or use user installation
pip3 install --user requests
```

âœ… **Success checkpoint:** requests library imports successfully

---

### Understanding Python Scripts (Preview)

**In Part 2 (RAG System Guide), you'll create two Python scripts:**

1. **load_document.py** - Uploads documents to ChromaDB
   - Reads text files
   - Breaks them into chunks (800 characters each)
   - Converts to vectors using Ollama
   - Stores in ChromaDB

2. **query_rag.py** - Asks questions about your documents
   - Takes your question
   - Finds relevant document chunks
   - Uses Ollama to generate answer
   - Displays response

**Don't worry:** Complete, working scripts will be provided. You'll just copy-paste them.

---

### Step 6.3: Test Python + requests Library

**Quick test to ensure everything works:**

```bash
python3 << 'TESTSCRIPT'
import requests
import json

# Test Ollama
print("Testing Ollama...")
response = requests.get("http://localhost:11434")
print(f"Ollama: {response.text.strip()}")

# Test ChromaDB
print("\nTesting ChromaDB...")
response = requests.get("http://localhost:8000/api/v1/heartbeat")
data = response.json()
print(f"ChromaDB: Connected (heartbeat: {data.get('nanosecond heartbeat', 'N/A')})")

print("\nâœ… Python environment is ready!")
TESTSCRIPT
```

**Expected output:**
```
Testing Ollama...
Ollama: Ollama is running

Testing ChromaDB...
ChromaDB: Connected (heartbeat: 1702909876543210000)

âœ… Python environment is ready!
```

**What this test does:**
- Imports necessary libraries
- Connects to Ollama
- Connects to ChromaDB
- Confirms both are accessible from Python

**If you see errors:**
- Ollama not running → `ollama serve &`
- ChromaDB not running → `podman start chromadb`
- Network issues → Check firewall settings

---

### Step 6.4: Success Checkpoint

Before moving to final verification, confirm:

- [ ] Python 3.9+ is installed
- [ ] requests library is installed
- [ ] Test script runs successfully
- [ ] Both Ollama and ChromaDB are reachable from Python

âœ… **If all checkboxes are checked, you're ready for final verification!**

---

### Troubleshooting Python Setup

#### Issue: "pip3: command not found"

**Fix:**
```bash
# Use python3 -m pip instead
python3 -m pip install requests

# Verify
python3 -c "import requests; print('âœ" works')"
```

---

#### Issue: "Permission denied" when installing

**Fix option 1:** Use --user flag
```bash
pip3 install --user requests
```

**Fix option 2:** Use sudo (not recommended but works)
```bash
sudo pip3 install requests
```

---

#### Issue: Test script can't reach Ollama

**Diagnostic:**
```bash
# Check if Ollama is running
curl http://localhost:11434

# If nothing, start Ollama
ollama serve &
sleep 5
```

---

#### Issue: Test script can't reach ChromaDB

**Diagnostic:**
```bash
# Check if container is running
podman ps | grep chromadb

# If not running
podman start chromadb
sleep 15
```

---

## Part 7: Final Verification

**Time:** 5 minutes  
**What you'll do:** Run comprehensive checks on entire environment  
**Why:** Confirm everything is working before building the RAG system

---

### Step 7.1: Check All Services Status

**Check running containers:**
```bash
podman ps
```

**Expected output:**
```
CONTAINER ID  IMAGE                            COMMAND  CREATED  STATUS      PORTS                   NAMES
abc123def456  docker.io/chromadb/chroma:0.4.24 ...      X mins   Up X mins   0.0.0.0:8000->8000/tcp  chromadb
def456ghi789  docker.io/n8nio/n8n:latest       ...      X mins   Up X mins   0.0.0.0:5678->5678/tcp  n8n
```

**What to verify:**
- âœ… Both containers listed
- âœ… STATUS shows "Up"
- âœ… PORTS are mapped correctly

---

**Check Ollama models:**
```bash
ollama list
```

**Expected output:**
```
NAME                    ID              SIZE      MODIFIED
llama3.2:3b            abc123def...    2.0 GB    X minutes ago
nomic-embed-text       def456ghi...    274 MB    X minutes ago
```

**What to verify:**
- âœ… Both models listed
- âœ… Sizes are correct (2.0 GB and 274 MB)

---

### Step 7.2: Test Network Connectivity

**Test 1: Ollama accessibility**
```bash
curl http://localhost:11434
```

**Expected:** `Ollama is running`

---

**Test 2: ChromaDB accessibility**
```bash
curl http://localhost:8000/api/v1/heartbeat
```

**Expected:** JSON with heartbeat timestamp

---

**Test 3: n8n accessibility**
```bash
curl -s http://localhost:5678 | grep "n8n"
```

**Expected:** Some HTML mentioning "n8n"

---

**Test 4: ChromaDB collection exists**
```bash
curl http://localhost:8000/api/v1/collections
```

**Expected:** JSON array showing cisco_docs collection

---

### Step 7.3: Comprehensive System Check Script

**Create a system check script for daily use:**

```bash
cat > ~/cisco-rag-demo/system-check.sh << 'CHECKSCRIPT'
#!/bin/bash
echo "====================================="
echo "RAG System Status Check"
echo "====================================="
echo ""

# Check Podman
echo "ðŸ"¦ Podman Containers:"
podman ps --format "  {{.Names}}: {{.Status}}"
echo ""

# Check Ollama
echo "ðŸ¤– Ollama Models:"
ollama list | tail -n +2 | awk '{print "  " $1 " (" $3 " " $4 ")"}'
echo ""

# Check Ollama service
echo "ðŸŒ Network Services:"
if curl -s http://localhost:11434 > /dev/null; then
    echo "  Ollama (11434): âœ…"
else
    echo "  Ollama (11434): âŒ"
fi

if curl -s http://localhost:8000/api/v1/heartbeat > /dev/null; then
    echo "  ChromaDB (8000): âœ…"
else
    echo "  ChromaDB (8000): âŒ"
fi

if curl -s http://localhost:5678 > /dev/null; then
    echo "  n8n (5678): âœ…"
else
    echo "  n8n (5678): âŒ"
fi
echo ""

# Check Python
echo "ðŸ Python Environment:"
if python3 -c "import requests" 2>/dev/null; then
    echo "  Python 3 + requests: âœ…"
else
    echo "  Python 3 + requests: âŒ"
fi
echo ""

# Check collections
echo "ðŸ"š ChromaDB Collections:"
curl -s http://localhost:8000/api/v1/collections | \
    python3 -c "import sys, json; data = json.load(sys.stdin); print('  ' + data[0]['name'] if data else '  None found')"
echo ""

echo "====================================="
echo "âœ… System check complete"
echo "====================================="
CHECKSCRIPT

# Make executable
chmod +x ~/cisco-rag-demo/system-check.sh
```

---

**Run the system check:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Expected output:**
```
=====================================
RAG System Status Check
=====================================

ðŸ"¦ Podman Containers:
  chromadb: Up X minutes
  n8n: Up X minutes

ðŸ¤– Ollama Models:
  llama3.2:3b (2.0 GB)
  nomic-embed-text (274 MB)

ðŸŒ Network Services:
  Ollama (11434): âœ…
  ChromaDB (8000): âœ…
  n8n (5678): âœ…

ðŸ Python Environment:
  Python 3 + requests: âœ…

ðŸ"š ChromaDB Collections:
  cisco_docs

=====================================
âœ… System check complete
=====================================
```

**What to verify:**
- All services show âœ…
- Both Ollama models present
- Both containers running
- cisco_docs collection exists

---

### Step 7.4: Environment Summary

**At this point, you have:**

âœ… **Podman** - Container runtime configured and running  
âœ… **Ollama** - AI models downloaded and accessible  
  - llama3.2:3b (answers questions)  
  - nomic-embed-text (creates vectors)  
âœ… **ChromaDB 0.4.24** - Vector database with collection created  
âœ… **n8n** - Workflow automation platform ready  
âœ… **Python + requests** - Testing environment configured  

**Your system is ready for:**
- Loading documents (Part 2)
- Building chat interface (Part 2)
- Creating Webex bot (Part 3)

---

### Step 7.5: Post-Restart Checklist

**Important:** After restarting your Mac, you'll need to:

```bash
# 1. Start Podman machine
podman machine start

# 2. Start Ollama service
ollama serve &

# 3. Start containers (if not auto-started)
podman start chromadb n8n

# 4. Wait for services
sleep 20

# 5. Run system check
~/cisco-rag-demo/system-check.sh
```

ðŸ'¡ **Tip:** Save these commands in `~/cisco-rag-demo/startup.sh` for quick restarts.

---

### Step 7.6: Final Success Checkpoint

**Complete environment checklist:**

**Core Services:**
- [ ] Podman machine running (`podman machine list`)
- [ ] Ollama service running (`curl http://localhost:11434`)
- [ ] ChromaDB container running (visible in `podman ps`)
- [ ] n8n container running (visible in `podman ps`)

**Accessibility:**
- [ ] Can access ChromaDB API (`curl http://localhost:8000/api/v1/heartbeat`)
- [ ] Can access n8n web interface (`http://localhost:5678`)
- [ ] Can access Ollama API (`curl http://localhost:11434`)

**Data:**
- [ ] AI models downloaded (`ollama list` shows 2 models)
- [ ] ChromaDB collection exists (`cisco_docs` visible)
- [ ] Python environment ready (requests library installed)

**âœ… If all boxes are checked, your environment is production-ready!**

---

## Troubleshooting

### Common Issues After Mac Restart

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
ollama serve &

# Wait a moment
sleep 5

# Verify
curl http://localhost:11434
```

---

### Performance Issues

#### Issue: Everything is slow (>20 seconds per query)

**Possible causes:**
1. Low RAM - Mac is using swap memory
2. Other apps consuming resources
3. Models unloading/reloading frequently

**Check RAM usage:**
```bash
# Open Activity Monitor
# Check "Memory" tab
# Look at "Memory Pressure"
```

**If Memory Pressure is yellow/red:**
- Close other applications
- Restart Mac
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
   system_profiler SPHardwareDataType | grep -A 5 "Hardware Overview"
   sw_vers
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

ðŸŽ‰ **Congratulations!** You've completed the environment setup.

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

**After Mac restart:**
```bash
podman machine start
ollama serve &
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

## Document Information

**Version:** 1.0.0  
**Last Updated:** December 18, 2024  
**Part of:** Local RAG System Implementation Guides  
**Next Guide:** [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)  

**This guide is based on:**
- Actual deployment on Mac M4 Pro with 24GB RAM
- Real troubleshooting experiences documented
- Testing with Cisco network assessment documents
- Production-ready configurations

**Feedback:** If you encounter issues not covered here, please document them for future updates.

---

âž¡ï¸ **Ready to build your RAG system? Proceed to [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)**
