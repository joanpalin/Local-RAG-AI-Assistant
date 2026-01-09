# Comprehensive Troubleshooting Guide
## Symptom-Based Problem Solving for Your Local RAG System

**This guide represents hours of troubleshooting distilled into solutions.**

**You will encounter issues.** That's normal. Software is complex, especially when combining multiple systems. The best way to solve it is pointing this GitHub to any AI assistant and asking it to troubleshoot with you.


---

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [Quick Diagnostic Checklist](#quick-diagnostic-checklist)
3. [Part 1: Environment Issues](#part-1-environment-issues)
4. [Part 2: RAG System Issues](#part-2-rag-system-issues)
5. [Part 3: Webex Integration Issues](#part-3-webex-integration-issues)
6. [Part 4: Performance Issues](#part-4-performance-issues)
7. [Part 5: Connection Issues](#part-5-connection-issues)
8. [Part 6: Permission Issues](#part-6-permission-issues)
9. [Part 7: Data Issues](#part-7-data-issues)
10. [Emergency Recovery Procedures](#emergency-recovery-procedures)
11. [Prevention Checklist](#prevention-checklist)
12. [Getting Additional Help](#getting-additional-help)

---

## How to Use This Guide

### Understanding This Guide's Organization

**Why symptom-based?** You know what's broken ("Bot doesn't respond") but not always which component is causing it. This guide helps you diagnose and fix problems based on what you see, not what you think is wrong.

**Example of organization:**
- ❌ **BAD:** "n8n Troubleshooting" (you may not know n8n is the problem)
- ✅ **GOOD:** "Workflow doesn't execute" (this is what you actually see)

### How to Navigate

1. **Find your symptom** - Use the table of contents or Cmd+F to search
2. **Follow diagnostic steps** - Determine the root cause
3. **Apply the fix** - Step-by-step solutions provided
4. **Verify the fix** - Check that the problem is resolved
5. **Prevent recurrence** - Learn what caused it

### Reading Diagnostic Flowcharts

Flowcharts show decision paths:
```
Problem occurs
     ↓
Check X
     ↓
What do you see?
     ├─ Symptom A → Fix A
     ├─ Symptom B → Fix B
     └─ Symptom C → Fix C
```

Follow the path that matches what you observe.

---

## General Troubleshooting Philosophy

**Before diving into specific problems, remember these principles:**

### 1. Don't Panic
Most issues are simple:
- Permission problems
- Services not started
- Wrong configuration
- Typos in commands

**Rarely** are problems caused by:
- Corrupted installations
- Hardware failures
- Bugs in the software

### 2. Check One Thing at a Time
Don't change multiple things simultaneously:
- Make one change
- Test if it fixed the issue
- If not, revert and try something else
- Document what you tried

### 3. Verify After Each Fix
Always confirm the fix worked:
```bash
# Example verification pattern
podman restart chromadb     # Make fix
sleep 10                     # Wait for startup
podman ps | grep chromadb    # Verify running
curl localhost:8000/api/v1/heartbeat  # Test functionality
```

### 4. Document What Worked
Keep notes for next time:
- What was the symptom?
- What was the root cause?
- What fixed it?
- How long did it take?

**This saves hours on future issues.**

---

## Quick Diagnostic Checklist

**Before troubleshooting anything specific, check these basics:**

```bash
# 1. Are all containers running?
podman ps

# Expected: Shows chromadb and n8n with "Up" status
# If not: See "Container won't start" section
```

```bash
# 2. Is Ollama running?
curl http://localhost:11434

# Expected: "Ollama is running"
# If not: See "Ollama not responding" section
```

```bash
# 3. Are models loaded?
ollama list

# Expected: Shows llama3.2:3b and nomic-embed-text
# If not: See "Ollama models missing" section
```

```bash
# 4. Did you restart recently?
# Check if you need to re-activate services
```

```bash
# 5. Has tunnel URL changed? (Webex bot only)
# Free tier limitation - see "Tunnel URL keeps changing"
```

**✅ All checks pass?** Your core infrastructure is healthy. Problem is likely configuration or workflow-specific.

**❌ Some checks fail?** Start with those failures before investigating complex issues.

---

# Part 1: Environment Issues

These are problems with your fundamental setup - the "infrastructure layer" that everything else depends on.

---

## Symptom: Podman commands don't work

### What you see:
```bash
$ podman ps
bash: podman: command not found
```

### Diagnostic Steps

**Step 1: Check if Podman exists**
```bash
which podman
```

**Interpret results:**
- ✅ Shows path (e.g., `/opt/podman/bin/podman`) → Podman installed but not in PATH
- ❌ Nothing returned → Podman not installed

---

### Fix A: Podman Not Installed

**You need to install Podman first.**

**Solution:**
1. Go to [GUIDE_1_ENVIRONMENT_SETUP.md](GUIDE_1_ENVIRONMENT_SETUP.md)
2. Follow **Part 2: Installing Podman** completely
3. Return here after installation

**Verification:**
```bash
podman --version
# Should show: podman version 4.x.x
```

**⏱️ Time:** 10-15 minutes

---

### Fix B: Podman Installed But Not in PATH

**Podman is installed but your shell can't find it.**

**Solution:**
```bash
# Find where Podman is installed
find /opt /usr/local -name podman 2>/dev/null

# Example output: /opt/podman/bin/podman

# Add to PATH temporarily (for this session)
export PATH=$PATH:/opt/podman/bin

# Test
podman --version

# If that worked, make it permanent:
echo 'export PATH=$PATH:/opt/podman/bin' >> ~/.zshrc
source ~/.zshrc
```

**Verification:**
```bash
# Close and reopen Terminal
# Then test:
podman --version
```

**✅ Success:** Version number appears without error

---

## Symptom: Podman machine won't start

### What you see:
```bash
$ podman machine start
Error: unable to start machine: machine "podman-machine-default" already running
```

**OR**

```bash
$ podman machine start
Error: machine did not transition to running: machine "podman-machine-default" stuck in stopping
```

### Diagnostic Flowchart

```
Machine won't start
     ↓
Check status: podman machine list
     ↓
What do you see?
     ├─ "running" → Already running → Fix A
     ├─ "stopped" → Normal stop → Fix B
     └─ "stuck in stopping" → Corrupted state → Fix C
```

---

### Fix A: Already Running

**The machine is already running - nothing to fix!**

**Verify containers:**
```bash
podman ps
```

If containers aren't running, start them:
```bash
podman start chromadb n8n
```

**✅ Done** - No fix needed

---

### Fix B: Properly Stopped Machine

**Normal situation after restart or manual stop.**

**Solution:**
```bash
# Start the machine
podman machine start

# Wait for startup (this takes 15-30 seconds)
sleep 20

# Verify it's running
podman machine list
# STATUS column should show "running"

# Now start containers
podman start chromadb n8n

# Verify
podman ps
```

**✅ Success:** Both containers show "Up" status

---

### Fix C: Corrupted State (Stuck in Stopping)

**The machine state is corrupted - requires reset.**

⚠️ **WARNING:** This stops all containers but doesn't delete data.

**Solution:**
```bash
# Force stop the machine
podman machine stop --force

# Wait for shutdown
sleep 10

# Start fresh
podman machine start

# Wait for startup
sleep 20

# Verify
podman machine list

# Restart containers
podman start chromadb n8n
```

**If still stuck:**
```bash
# Nuclear option: Delete and recreate machine
podman machine rm
podman machine init
podman machine start

# Then recreate containers (see emergency recovery)
```

**✅ Success:** Machine shows "running" and containers start

---

## Symptom: Container won't start

### What you see:
```bash
$ podman ps
# Container missing from list

$ podman ps -a
CONTAINER ID  IMAGE           STATUS
abc123def     chromadb:latest Exited (1) 2 minutes ago
```

### Diagnostic Flowchart

```
Container stopped unexpectedly
     ↓
Check logs: podman logs [container-name]
     ↓
What error do you see?
     ├─ "Permission denied" → Fix A
     ├─ "Port already in use" → Fix B
     ├─ "Out of space" → Fix C
     ├─ "No such file or directory" → Fix D
     └─ Other error → Fix E
```

---

### Fix A: Permission Denied

**Most common on macOS with Podman - container can't write to volume.**

**What you'll see in logs:**
```bash
$ podman logs chromadb
Error: EACCES: permission denied, mkdir '/chroma/chroma'
```

**Solution:**
```bash
# Stop and remove container
podman stop chromadb  # or n8n
podman rm chromadb

# Fix permissions on data directory
chmod -R 777 ~/cisco-rag-demo/chroma-data

# For n8n:
chmod -R 777 ~/cisco-rag-demo/n8n-data

# Recreate container (use commands from setup guide)
# For ChromaDB:
podman run -d --name chromadb -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  chromadb/chroma:0.4.24

# For n8n:
podman run -d --name n8n -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest

# Verify
sleep 10
podman ps
```

**Why 777 permissions?**
- macOS + Podman run containers in a VM
- VM has different user IDs than macOS
- 777 allows any user to write
- Safe for local development

**✅ Success:** `podman ps` shows container running

---

### Fix B: Port Already in Use

**Another service is using the port.**

**What you'll see:**
```bash
$ podman logs n8n
Error: listen EADDRINUSE: address already in use :::5678
```

**Solution:**
```bash
# Find what's using the port (use port number from error)
lsof -i :5678

# Example output:
# COMMAND   PID  USER   FD   TYPE
# node      12345 user   23u  IPv6  0x123abc  TCP *:5678

# Option 1: Kill the process
kill 12345

# Option 2: Stop conflicting container
podman stop [other-container]

# Option 3: Use different port
podman rm n8n
podman run -d --name n8n -p 5679:5678 \  # Changed host port
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest
```

**Common port conflicts:**
- **5678** - n8n (may conflict with development servers)
- **8000** - ChromaDB (may conflict with Python web servers)
- **11434** - Ollama (rarely conflicts)

**✅ Success:** `podman ps` shows container running

---

### Fix C: Out of Disk Space

**Your Mac doesn't have enough free space.**

**Check available space:**
```bash
df -h

# Look at "/" filesystem
# Available column shows free space
# Need at least 10GB free
```

**Solution:**
```bash
# Clean up old containers and images
podman system prune -a

# This will prompt: Are you sure? (y/N)
# Type: y

# Check space again
df -h

# If still low:
# 1. Empty Trash
# 2. Delete large files
# 3. Clear Downloads folder
# 4. Remove old applications

# Then try starting container again
podman start chromadb n8n
```

**✅ Success:** Container starts after freeing space

---

### Fix D: Volume Directory Missing

**The data directory doesn't exist.**

**What you'll see:**
```bash
Error: statfs /Users/you/cisco-rag-demo/chroma-data: no such file or directory
```

**Solution:**
```bash
# Create missing directories
mkdir -p ~/cisco-rag-demo/chroma-data
mkdir -p ~/cisco-rag-demo/n8n-data
mkdir -p ~/cisco-rag-demo/documents

# Set permissions
chmod -R 777 ~/cisco-rag-demo/chroma-data
chmod -R 777 ~/cisco-rag-demo/n8n-data

# Recreate container
# (Use commands from Fix A above)
```

**✅ Success:** Container starts with proper volumes

---

### Fix E: Read Error and Search

**Unexpected error - need to investigate.**

**Steps:**
```bash
# View full error log
podman logs [container-name] 2>&1 | less

# Common issues to look for:
# - Image not found → Need to pull image
# - Syntax error in command → Check run command
# - Network error → Check internet connection
# - Resource limits → Check memory/CPU

# Search online for specific error message
# Include: "podman" + error message + "macOS"
```

**If still stuck:**
1. Copy complete error message
2. Go to [Getting Additional Help](#getting-additional-help)
3. Include output of: `podman version`, `podman machine list`, `podman logs [container]`

---

## Symptom: Ollama not responding

### What you see:
```bash
$ curl http://localhost:11434
curl: (7) Failed to connect to localhost port 11434: Connection refused
```

**OR**

```bash
$ ollama list
Error: could not connect to ollama server, run 'ollama serve' to start it
```

### Diagnostic Steps

```bash
# Check if Ollama process is running
ps aux | grep ollama | grep -v grep

# If nothing appears → Ollama not running
# If shows process → Ollama running but not responding (rare)
```

---

### Fix: Ollama Not Running

**Ollama doesn't auto-start after Mac restart.**

**Solution:**
```bash
# Start Ollama in background
ollama serve &

# Wait for startup
sleep 5

# Verify
curl http://localhost:11434

# Expected output:
# Ollama is running

# Also verify
ollama list

# Should show your models
```

**💡 Pro Tip:** Add to startup script
```bash
# Create startup script
cat > ~/start-rag.sh << 'EOF'
#!/bin/bash
echo "Starting RAG system..."
ollama serve &
sleep 10
podman machine start
sleep 20
podman start chromadb n8n
echo "✅ RAG system ready"
EOF

chmod +x ~/start-rag.sh
```

**✅ Success:** Ollama responds to curl and shows models

---

## Symptom: Ollama models missing

### What you see:
```bash
$ ollama list
NAME  ID  SIZE  MODIFIED
# Empty list or missing models
```

### Diagnostic Steps

**Check which models are missing:**
- Need: `llama3.2:3b`
- Need: `nomic-embed-text`

---

### Fix: Download Missing Models

**Models need to be pulled from repository.**

**Solution:**
```bash
# Pull LLM model (answers questions)
ollama pull llama3.2:3b

# This takes 5-10 minutes (2GB download)
# You'll see progress:
# pulling manifest
# pulling ... 100%
# success

# Pull embedding model (converts to vectors)
ollama pull nomic-embed-text

# This takes 2-3 minutes (274MB download)

# Verify both models present
ollama list

# Should show:
# NAME                ID           SIZE
# llama3.2:3b         abc123       2.0 GB
# nomic-embed-text    def456       274 MB
```

**⏱️ Time:** 10-15 minutes total

**💡 Bandwidth Tip:** If on slow connection:
- Download during off-hours
- Don't interrupt download
- If interrupted, re-run same command (resumes)

**✅ Success:** Both models appear in `ollama list`

---

## Symptom: ChromaDB not accessible

### What you see:
```bash
$ curl http://localhost:8000/api/v1/heartbeat
curl: (7) Failed to connect to localhost port 8000: Connection refused
```

### Diagnostic Flowchart

```
ChromaDB not accessible
     ↓
Check container: podman ps | grep chromadb
     ↓
Is it running?
     ├─ Yes → Port issue → Fix A
     ├─ No, but exists (podman ps -a) → Stopped → Fix B
     └─ No, doesn't exist → Not created → Fix C
```

---

### Fix A: Container Running But Not Accessible

**Container is up but port not responding.**

**Check logs:**
```bash
podman logs chromadb
```

**Common causes:**
1. Container still starting (wait 20 seconds)
2. Port mapping wrong
3. Service crashed inside container

**Solution:**
```bash
# Wait a bit longer
sleep 30

# Try again
curl http://localhost:8000/api/v1/heartbeat

# If still fails, restart container
podman restart chromadb
sleep 20

# Test
curl http://localhost:8000/api/v1/heartbeat

# Expected response:
# {"nanosecond heartbeat": 1234567890123456789}
```

**✅ Success:** Heartbeat returns JSON with timestamp

---

### Fix B: Container Stopped

**Container exists but isn't running.**

**Solution:**
```bash
# Start the container
podman start chromadb

# Wait for startup
sleep 15

# Test
curl http://localhost:8000/api/v1/heartbeat
```

**If container immediately stops:**
- Check logs: `podman logs chromadb`
- Likely permission issue - see [Fix A: Permission Denied](#fix-a-permission-denied)

**✅ Success:** Container stays running, heartbeat responds

---

### Fix C: Container Doesn't Exist

**Need to create ChromaDB container.**

**Solution:**
```bash
# Create data directory
mkdir -p ~/cisco-rag-demo/chroma-data
chmod 777 ~/cisco-rag-demo/chroma-data

# Create container
podman run -d --name chromadb -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  chromadb/chroma:0.4.24

# Wait for startup
sleep 20

# Test
curl http://localhost:8000/api/v1/heartbeat

# Verify
podman ps | grep chromadb
```

**⚠️ Important:** Use version **0.4.24** specifically - newer versions have API issues.

**✅ Success:** Heartbeat responds, container running

---

## Symptom: n8n web interface won't load

### What you see:
```bash
# Browser shows:
http://localhost:5678
This site can't be reached
```

### Diagnostic Flowchart

```
Can't access n8n
     ↓
Check container: podman ps | grep n8n
     ↓
Is it running?
     ├─ Yes → Port/browser issue → Fix A
     ├─ No → Container stopped → Fix B
     └─ Shows "restarting" → Crash loop → Fix C
```

---

### Fix A: Container Running, Can't Access

**Browser can't reach the service.**

**Steps to try:**

**1. Check correct URL:**
```bash
# Should be exactly:
http://localhost:5678

# NOT:
https://localhost:5678  # Wrong protocol
127.0.0.1:5678          # Missing http://
```

**2. Try different browser:**
- Safari might have strict security
- Try Chrome or Firefox

**3. Check port is actually mapped:**
```bash
podman ps

# Look at PORTS column for n8n:
# Should show: 0.0.0.0:5678->5678/tcp
```

**4. Test port is accessible:**
```bash
curl http://localhost:5678

# Should return HTML or at least not "connection refused"
```

**5. Restart browser and try again**

**✅ Success:** n8n interface loads in browser

---

### Fix B: Container Not Running

**n8n container stopped.**

**Solution:**
```bash
# Start container
podman start n8n

# Wait for startup
sleep 10

# Check logs for errors
podman logs n8n | tail -20

# If you see "Waiting for instance to initialize..."
# This is normal, wait another 20 seconds

# Test in browser
open http://localhost:5678
```

**✅ Success:** n8n interface loads

---

### Fix C: Container in Crash Loop

**Container keeps restarting - indicates serious error.**

**Check what's wrong:**
```bash
# View recent logs
podman logs n8n

# Look for error messages at the end
# Common issues:
# - Permission denied on /home/node/.n8n
# - Port already in use
# - Database corruption
```

**Solution for permission error:**
```bash
podman stop n8n
podman rm n8n

# Fix permissions
chmod -R 777 ~/cisco-rag-demo/n8n-data

# Recreate container
podman run -d --name n8n -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest

# Monitor logs
podman logs -f n8n
# Press Ctrl+C when you see "Editor is now accessible"
```

**✅ Success:** Container runs without restarting

---

# Part 2: RAG System Issues

Problems with document loading, querying, and the RAG workflows.

---

## Symptom: Can't upload documents (Python script)

### What you see:
```bash
$ python3 load_document.py documents/sample.md
Error: Collection 'cisco_docs' not found
```

### Diagnostic Steps

**Check if collection exists:**
```bash
curl http://localhost:8000/api/v1/collections
```

**Interpret results:**
- `[]` (empty array) → Collection doesn't exist → Fix A
- Shows collection with name "cisco_docs" → Connection issue → Fix B
- Connection refused → ChromaDB not running → See [ChromaDB not accessible](#symptom-chromadb-not-accessible)

---

### Fix A: Collection Doesn't Exist

**Need to create the collection first.**

**Solution:**
```bash
# Create collection
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "cisco_docs",
    "metadata": {"description": "Cisco network documents"}
  }'

# Verify creation
curl http://localhost:8000/api/v1/collections

# Should show collection with "cisco_docs" name

# Now try loading document again
python3 load_document.py documents/sample.md
```

**✅ Success:** Document loads without collection error

---

### Fix B: Collection Exists But Script Can't Find It

**Script may be using wrong collection name or API version.**

**Check script configuration:**
```bash
# View script
cat load_document.py | grep collection

# Should show:
# collection_name = "cisco_docs"
```

**Verify UUID lookup working:**
```python
# Test script manually
python3 << 'EOF'
import requests

resp = requests.get('http://localhost:8000/api/v1/collections')
collections = resp.json()

for coll in collections:
    print(f"Collection: {coll['name']}")
    print(f"UUID: {coll['id']}")
EOF
```

**If shows cisco_docs UUID:** Script should work - check for typos
**If error:** ChromaDB connection issue - restart ChromaDB

**✅ Success:** Script successfully uploads document

---

## Symptom: Document upload stuck/slow

### What you see:
```bash
$ python3 load_document.py documents/large_file.md

Document split into 42 chunks
Processing chunk 1/42...
# Hangs here for several minutes
```

### Diagnostic Flowchart

```
Upload stuck/slow
     ↓
Is this the first upload after starting?
     ├─ Yes → Cold start (models loading) → Fix A
     └─ No → Check document size → Fix B or C
```

---

### Fix A: Cold Start (First Upload)

**Models need to load into RAM first - this is NORMAL.**

**What's happening:**
1. First embedding request triggers model load
2. Ollama loads nomic-embed-text (274MB into RAM)
3. Takes 10-30 seconds
4. Subsequent chunks are fast

**Solution: Pre-load models**
```bash
# Before uploading documents, warm up models
ollama run nomic-embed-text "test"
# Press Ctrl+D to exit

# Now upload will be faster
python3 load_document.py documents/file.md
```

**⏱️ Expected times:**
- **Cold start:** 30-60 seconds for first chunk
- **Warmed up:** 2-3 seconds per chunk
- **Large doc (50 chunks):** 2-3 minutes total

**✅ This is normal:** First upload is always slower

---

### Fix B: Document Too Large

**File is very large, taking long time to process.**

**Check document size:**
```bash
ls -lh documents/your-file.md

# If > 5MB, consider:
```

**Solutions:**

**Option 1: Split document**
```bash
# Split into smaller files
split -l 1000 documents/large_file.md documents/chunk_

# Upload each separately
for file in documents/chunk_*; do
    python3 load_document.py "$file"
done
```

**Option 2: Reduce content**
- Remove unnecessary sections
- Remove excessive whitespace
- Remove duplicate content

**Option 3: Convert to plain text first**
```bash
# If PDF or complex format
# Convert to plain markdown or text
# This removes embedded objects
```

**✅ Success:** Upload completes in reasonable time (<3 min)

---

### Fix C: System Resource Exhaustion

**Mac is struggling with processing.**

**Check Activity Monitor:**
```bash
# Look for:
# - High CPU usage (>90%)
# - High Memory Pressure (yellow/red)
# - Swap Memory in use
```

**Solution:**
```bash
# Close other applications
# Especially browsers with many tabs

# Wait 30 seconds for system to recover

# Try upload again
python3 load_document.py documents/file.md
```

**If still slow:**
```bash
# Reduce chunk size in script
# Edit load_document.py:
# Change: chunk_size = 800
# To: chunk_size = 500

# This creates more chunks but each processes faster
```

**✅ Success:** Upload completes without system freezing

---

## Symptom: Query returns no results

### What you see:
```bash
$ python3 query_rag.py "What is the network budget?"
Error: No documents found in ChromaDB
```

**OR**

```bash
# Returns but says:
I don't have any information about that in the documents.
```

### Diagnostic Steps

**Check document count:**
```bash
curl http://localhost:8000/api/v1/collections | \
  jq -r '.[] | select(.name=="cisco_docs") | .id' | \
  xargs -I {} curl -s "http://localhost:8000/api/v1/collections/{}/count"

# Shows number of chunks stored
```

**Interpret results:**
- `{"count": 0}` → No documents loaded → Fix A
- `{"count": 50}` → Documents loaded but not matching → Fix B
- Connection error → ChromaDB issue → See [ChromaDB not accessible](#symptom-chromadb-not-accessible)

---

### Fix A: No Documents Loaded

**Need to load documents before querying.**

**Solution:**
```bash
# Load a sample document
python3 load_document.py documents/sample_customer_document.md

# Wait for completion
# Should see: "✅ Loaded X chunks into ChromaDB"

# Verify count increased
curl http://localhost:8000/api/v1/collections | \
  jq -r '.[] | select(.name=="cisco_docs") | .id' | \
  xargs -I {} curl -s "http://localhost:8000/api/v1/collections/{}/count"

# Now try query again
python3 query_rag.py "What is mentioned in the document?"
```

**✅ Success:** Query returns relevant information

---

### Fix B: Documents Loaded But Not Matching

**Documents don't contain answer OR query isn't specific enough.**

**Diagnostic questions:**
1. Does your document actually contain this information?
2. Are you using the right keywords?
3. Is the question clearly worded?

**Solution: Improve question specificity**

❌ **Bad questions:**
- "What about the budget?" (too vague)
- "Tell me about things" (no context)
- "What's the status?" (which status?)

✅ **Good questions:**
- "What is the proposed network infrastructure budget for FY2025?"
- "What equipment requires replacement?"
- "What is the current datacenter uptime percentage?"

**Test strategy:**
```bash
# Start with very specific question about content you KNOW is in document
python3 query_rag.py "exact phrase from document"

# If that works, gradually make more general
python3 query_rag.py "paraphrase of that concept"
```

**Verify document content:**
```bash
# Check what's actually stored
# View a sample of loaded text
curl -X POST http://localhost:8000/api/v1/collections/[UUID]/query \
  -H 'Content-Type: application/json' \
  -d '{
    "query_embeddings": [[0.1,0.2,...]],  
    "n_results": 1
  }'
# (Requires embedding, complex - easier to re-read source document)
```

**✅ Success:** Query returns relevant answer when question is specific

---

### Fix C: Wrong Collection Name

**Script is querying different collection than documents were loaded into.**

**Check collection name in both scripts:**
```bash
# In load_document.py:
grep "collection_name" load_document.py

# In query_rag.py:
grep "collection_name" query_rag.py

# Must match exactly!
```

**Fix if different:**
```bash
# Edit script to use consistent name
# Or reload documents to correct collection
```

**✅ Success:** Scripts use same collection name

---

## Symptom: Answers are inaccurate or nonsensical

### What you see:
```bash
$ python3 query_rag.py "What is the network uptime?"

Answer: The weather in New York is sunny with a chance of rain.
# (Completely unrelated answer)
```

**OR**

```bash
Answer: The network uptime is approximately [hallucinated number] 
with a reliability factor of [made up statistic]
# (Confident but wrong answer)
```

### Possible Causes

1. **Document doesn't contain information** - Model is hallucinating
2. **Retrieved wrong chunks** - Search didn't find relevant sections
3. **Chunks too small** - Lost context when splitting document
4. **Model limitations** - llama3.2:3b is small model

---

### Fix A: Verify Document Contains Answer

**Make sure information actually exists in loaded documents.**

**Check source documents:**
```bash
# List what you've loaded
ls -lh ~/cisco-rag-demo/documents/

# Read them to verify content
cat ~/cisco-rag-demo/documents/assessment.md | grep -i "uptime"

# If found: Content exists, retrieval issue
# If not found: Need to load correct document
```

**Load correct document:**
```bash
# Find and load the right document
python3 load_document.py documents/correct_document.md

# Retry query
python3 query_rag.py "What is the network uptime?"
```

**✅ Success:** Answer matches content from documents

---

### Fix B: Improve Retrieval Quality

**Model is getting wrong document chunks.**

**This happens when:**
- Question uses different vocabulary than document
- Document uses very technical terms
- Multiple documents with similar content

**Solution: Rephrase question**
```bash
# If document says: "Network availability: 99.9%"
# And you ask: "What is uptime?"
# Try: "What is network availability?"

# Match vocabulary used in document
```

**Or increase chunks retrieved:**
```python
# In query_rag.py, find:
n_results=3  # Number of chunks to retrieve

# Change to:
n_results=5  # More context, might find right chunk
```

**⚠️ Trade-off:** More chunks = slower but more accurate

**✅ Success:** Retrieved chunks contain relevant information

---

### Fix C: Increase Chunk Size (Advanced)

**Chunks are too small, losing important context.**

**When to do this:**
- Answers mention "the document says" but give wrong info
- Related information split across chunks
- Questions need broader context

**Solution:**
```python
# Edit load_document.py
# Find:
chunk_size = 800

# Increase to:
chunk_size = 1200

# Must re-load all documents
# Delete collection first:
curl -X DELETE http://localhost:8000/api/v1/collections/[UUID]

# Recreate and reload:
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'

python3 load_document.py documents/*.md
```

**⏱️ Time:** Depends on document count, typically 5-15 minutes

**⚠️ Trade-off:** Larger chunks = slower search but better context

**✅ Success:** Answers improve with larger context windows

---

### Fix D: Accept Model Limitations

**llama3.2:3b is a small model - has limitations.**

**What it's good at:**
- Summarizing retrieved information
- Answering direct questions
- Simple reasoning

**What it struggles with:**
- Complex multi-step reasoning  
- Comparing across many documents
- Highly technical subject matter
- Precise numerical calculations

**Solutions:**

**Option 1: Use better prompts**
```bash
# Instead of: "Analyze the network security posture"
# Try: "List the security issues mentioned in the assessment"
```

**Option 2: Break into smaller questions**
```bash
# Instead of: "Compare Q1 and Q2 performance"
# Try:
python3 query_rag.py "What was Q1 performance?"
python3 query_rag.py "What was Q2 performance?"
# Then compare manually
```

**Option 3: Upgrade model (requires more resources)**
```bash
# Pull larger model
ollama pull llama3.2:13b  # 7GB, needs 16GB+ RAM

# Edit query script to use larger model
```

**✅ Accept:** Some questions require manual analysis or larger models

---

## Symptom: n8n workflow fails

### What you see:
```
# In n8n interface:
Workflow execution failed
Node: [Node Name] - Error
```

### Diagnostic Flowchart

```
Workflow failed
     ↓
Which node failed?
     ├─ Webhook → Fix A
     ├─ Ollama API call → Fix B
     ├─ ChromaDB node → Fix C
     └─ JavaScript code → Fix D
```

---

### Fix A: Webhook Node Error

**Webhook not receiving data correctly.**

**Common errors:**
- "Cannot read property 'data' of undefined"
- "Invalid JSON"
- "Missing required field"

**Solution:**
```javascript
// Click failed node
// View "Input" tab
// Check actual data structure

// If data is at json.body.data:
const webhookData = $input.first().json.body.data;

// Not at:
const webhookData = $input.first().json.data;  // ❌

// Update all paths in node to match actual structure
```

**✅ Success:** Node processes webhook data correctly

---

### Fix B: Ollama Connection Failed

**Can't reach Ollama from container.**

**Error messages:**
- "Connection refused to localhost:11434"
- "ECONNREFUSED"
- "Timeout waiting for response"

**Root cause:** Containers can't use "localhost" to reach host services

**Solution:**
```javascript
// In any node calling Ollama API
// Change URL from:
http://localhost:11434/api/...  // ❌

// To:
http://host.containers.internal:11434/api/...  // ✅

// Save workflow
// Re-test
```

**Why this works:**
- `host.containers.internal` is special hostname
- Podman/Docker maps this to host machine
- Works from inside containers

**Also verify Ollama is running:**
```bash
curl http://localhost:11434
# Should respond "Ollama is running"
```

**✅ Success:** n8n can call Ollama API

---

### Fix C: ChromaDB Node Error

**Can't access ChromaDB from workflow.**

**Error messages:**
- "Collection not found"
- "Invalid UUID"
- "Connection refused"

**Solution for UUID error:**
```javascript
// Add "Get Collection UUID" node before ChromaDB operations
// Node: HTTP Request
// Method: GET
// URL: http://chromadb:8000/api/v1/collections

// Then extract UUID:
const collections = $input.first().json;
const cisco_docs = collections.find(c => c.name === 'cisco_docs');
const collectionId = cisco_docs.id;

// Use collectionId in subsequent ChromaDB nodes
// URL: http://chromadb:8000/api/v1/collections/{{$json.collectionId}}/add
```

**Solution for connection error:**
```bash
# From n8n container, use "chromadb" hostname not "localhost"
# URL should be:
http://chromadb:8000/api/...  # ✅

# Not:
http://localhost:8000/api/...  # ❌
```

**✅ Success:** Workflow can access ChromaDB

---

### Fix D: JavaScript Code Error

**Custom code node has error.**

**Common issues:**
- Syntax error in JavaScript
- Variable not defined
- Incorrect data access

**Debugging steps:**
```javascript
// Add console.log statements
console.log("Input data:", JSON.stringify($input.first().json));
console.log("Variable value:", myVariable);

// Check node execution log
// Click node → "Output" tab
// View console.log output

// Fix issues based on actual data structure
```

**Common fixes:**
```javascript
// Safe property access:
const value = $input.first().json?.body?.data?.id;

// Check if exists first:
if (data && data.message) {
    // Use data.message
}

// Proper error handling:
try {
    // Your code
} catch (error) {
    console.error("Error:", error.message);
    throw error;  // Re-throw to show in n8n
}
```

**✅ Success:** JavaScript code executes without error

---

# Part 3: Webex Integration Issues

Problems specific to the Webex Teams bot.

---

## Symptom: Bot doesn't respond to messages

### What you see:
```
1. Send message to bot in Webex: "Hello"
2. No response
3. Silence
```

### Diagnostic Flowchart

```
Bot doesn't respond
     ↓
Check n8n workflow executions
     ↓
Do you see executions?
     ├─ Yes, but failed → Go to workflow failures (Part 2)
     ├─ Yes, succeeded → Bot responds but you don't see it → Fix A
     └─ No executions → Webhook not triggering → Fix B
```

---

### Fix A: Bot Responds But Not Visible

**Bot is sending messages but you can't see them.**

**Possible causes:**
1. Looking in wrong Webex space
2. Bot not in the space
3. Message sent to different room

**Solution:**
```bash
# Check which room bot responded to
# In n8n: Click successful execution
# Look at "Send to Webex" node output
# Check roomId value

# In Webex:
# Make sure you're in the same room/space
# Check if bot is actually in the space (see participant list)
```

**Add bot to space:**
```
1. In Webex, click space name
2. Click "Add people"
3. Search for bot email: [your-bot-email]@webex.bot
4. Add bot
5. Send test message: "test"
```

**✅ Success:** You see bot response in Webex

---

### Fix B: Webhook Not Triggering

**Webex isn't sending webhooks to n8n.**

**Most common cause: Tunnel URL changed**

**Check tunnel:**
```bash
# If using localhost.run:
# Look at terminal where tunnel is running
# Shows URL like: https://abc123.localhost.run

# Compare to webhook URL in Webex
curl https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Check targetUrl field
# Does it match current tunnel URL?
```

**If URLs don't match:**
```bash
# Delete old webhook
curl -X DELETE https://webexapis.com/v1/webhooks/[webhook-id] \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Register new webhook with current tunnel URL
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "RAG Bot Messages",
    "targetUrl": "https://abc123.localhost.run/webhook/webex-bot",
    "resource": "messages",
    "event": "created"
  }'
```

**⏱️ Time:** 2 minutes

**💡 Tip:** Save webhook ID and tunnel URL for quick updates

**✅ Success:** Webhook triggers n8n when you send message

---

## Symptom: Bot responds to its own messages (infinite loop)

### What you see:
```
1. You: "What is the network uptime?"
2. Bot: "According to the assessment..."
3. Bot: "According to the assessment, according to..."
4. Bot: "According to the assessment, according to, according to..."
5. n8n shows multiple rapid executions
```

### Root Cause

**Bot's responses trigger the webhook, causing bot to process its own messages.**

**Why this happens:**
1. Webex webhooks trigger for ALL messages
2. Including messages sent by the bot
3. Without filtering, bot processes its own responses
4. Creating infinite loop

---

### Fix: Implement Bot Message Filtering

**Need to check if message is from bot itself.**

**Solution in n8n workflow:**

**Step 1: Pass bot person ID through workflow**
```javascript
// In "Extract Webhook Data" node:
const webhookData = $input.first().json.body.data;
const webhookMeta = $input.first().json.body;

// Get bot's person ID
const botPersonId = webhookMeta.createdBy;

// Pass it forward
return [{
  json: {
    messageId: webhookData.id,
    roomId: webhookData.roomId,
    personId: webhookData.personId,
    botPersonId: botPersonId,  // ← Important!
    webhookReceived: true
  }
}];
```

**Step 2: Filter bot messages**
```javascript
// In "Validate & Extract Question" node:
const webhookData = $input.first().json;
const messageData = $input.first().json.message;

// Get bot ID from previous node
const botPersonId = webhookData.botPersonId;

console.log(`Message from: ${messageData.personId}`);
console.log(`Bot person ID: ${botPersonId}`);

// Check if message is from bot itself
if (messageData.personId === botPersonId) {
  console.log('✓ Ignoring message from bot itself');
  throw new Error('Ignoring bot message to prevent loop');
}

// If we get here, message is from user
// Continue processing...
```

**Result:**
- User messages: Processed normally
- Bot's own messages: Intentionally fails, stops workflow
- No infinite loop

**Verification:**
```bash
# Send test message to bot
# Check n8n executions
# Should see:
# - Execution 1: User message → Success → Bot responds
# - Execution 2: Bot message → Failed with "Ignoring bot message" → Stops
# - No Execution 3 (loop prevented)
```

**✅ Success:** Bot only responds to user messages, ignores own messages

---

## Symptom: Bot receives message but takes forever to respond

### What you see:
```
1. You send message: 09:00:00
2. Bot responds: 09:00:25 (25 seconds later)
3. Way too slow for conversation
```

### Target Performance

**Acceptable response times:**
- First query (cold start): 10-15 seconds
- Subsequent queries: 5-8 seconds
- Instant queries: Not realistic for RAG with LLM

**If consistently >15 seconds:** Something wrong

---

### Fix A: Models Not Pre-loaded (Cold Start)

**First query after startup loads models into RAM.**

**Solution:**
```bash
# Before demos or heavy use, pre-load models
ollama run llama3.2:3b "warmup"
# Press Ctrl+D

ollama run nomic-embed-text "warmup"
# Press Ctrl+D

# Now first query will be much faster
```

**Add to startup script:**
```bash
# In ~/start-rag.sh:
echo "Pre-loading AI models..."
ollama run llama3.2:3b "warmup" < /dev/null
ollama run nomic-embed-text "warmup" < /dev/null
echo "✅ Models ready"
```

**✅ Success:** First query <12 seconds, subsequent <8 seconds

---

### Fix B: Retrieving Too Many Context Chunks

**Workflow is pulling many document chunks, slowing response.**

**Check workflow:**
```javascript
// In "Search ChromaDB" node
// Look for: n_results parameter

// If set to 5 or more:
n_results: 5  // Getting 5 chunks

// Reduce to:
n_results: 3  // Faster, still accurate
```

**Trade-offs:**
- More chunks (5) = More context = Better answers = Slower
- Fewer chunks (2) = Less context = Faster = Potentially less accurate  
- Sweet spot: 3 chunks

**✅ Success:** Response time improves by 1-3 seconds

---

### Fix C: System Resource Constraints

**Mac is overloaded, slowing everything down.**

**Check:**
```bash
# Open Activity Monitor
# Check:
# - CPU usage (should be <80% when idle)
# - Memory pressure (should be green)
# - Swap memory (should be minimal)
```

**If resources constrained:**
```bash
# Close unnecessary applications
# Especially:
# - Web browsers with many tabs
# - Other development tools
# - Background sync services

# Wait 30 seconds for system to recover

# Test bot again
```

**Increase Podman resources (if you have spare RAM):**
```bash
podman machine stop
podman machine set --cpus 8 --memory 16384  # 16GB
podman machine start
podman start chromadb n8n
```

**✅ Success:** System resources available, faster responses

---

## Symptom: Bot works in direct messages but not in group spaces

### What you see:
```
Direct Message: Bot responds ✅
Group Space: Bot doesn't respond ❌
```

### Root Cause

**In group spaces, bot must be @mentioned to respond.**

**This is Webex behavior, not a bug.**

---

### Fix: Understand @mention Requirements

**How Webex bots work:**
- **Direct messages:** Bot receives all messages
- **Group spaces:** Bot only receives messages that @mention it

**To use bot in groups:**
```
1. Type: @BotName What is the network uptime?
2. Webex autocompletes bot name
3. Send message
4. Bot receives webhook
5. Bot responds
```

**Make sure bot is in space:**
```
1. In Webex space, click space name
2. View participants
3. Look for bot in list
4. If not there, add it:
   - Click "Add people"
   - Search bot email
   - Add
```

**Workflow consideration:**
```javascript
// Workflow should clean @mentions from questions
// In "Validate & Extract Question" node:
let question = messageData.text;

// Remove @mentions (HTML tags)
question = question.replace(/<spark-mention[^>]*>.*?<\/spark-mention>/gi, '');
question = question.replace(/@\w+/g, '');
question = question.trim();

// Now question is clean for RAG system
```

**✅ Success:** Bot responds in groups when @mentioned

---

## Symptom: Tunnel URL keeps changing

### What you see:
```bash
# Yesterday:
Connect to: https://abc123.localhost.run

# Today:
Connect to: https://xyz789.localhost.run

# Bot stops working because webhook has old URL
```

### This is NORMAL for Free Tunnels

**Free tier limitations:**
- **localhost.run:** New URL every restart
- **ngrok free:** New URL every 2 hours
- **Paid tiers:** Static URLs available

---

### Solution 1: Accept and Update (Free Tier)

**Fastest approach for demos and development.**

**Process:**
1. Restart tunnel → Note new URL
2. Update webhook (2 minutes)
3. Test bot
4. Repeat after each tunnel restart

**Quick update script:**
```bash
#!/bin/bash
# save as: ~/cisco-rag-demo/update-webhook.sh

BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)
TUNNEL_URL="https://abc123.localhost.run"  # Update this line each time
WEBHOOK_ID=$(cat ~/cisco-rag-demo/webhook-id.txt)

echo "Deleting old webhook..."
curl -X DELETE https://webexapis.com/v1/webhooks/$WEBHOOK_ID \
  -H "Authorization: Bearer $BOT_TOKEN"

echo "Registering new webhook..."
NEW_WEBHOOK=$(curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${TUNNEL_URL}/webhook/webex-bot\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }")

echo "$NEW_WEBHOOK" | jq -r '.id' > ~/cisco-rag-demo/webhook-id.txt
echo "✅ Webhook updated"
```

**Usage:**
```bash
# After tunnel restarts:
# 1. Edit script with new URL
# 2. Run:
chmod +x ~/cisco-rag-demo/update-webhook.sh
./cisco-rag-demo/update-webhook.sh

# Done in 2 minutes
```

**✅ Accept as normal:** URL changes require webhook updates

---

### Solution 2: Paid Static Tunnel (Production)

**For stable, long-term deployment.**

**Recommended service: ngrok Pro**
- Cost: $10/month
- Features: Static domain, never changes
- Benefit: No webhook updates needed

**Setup:**
```bash
# 1. Sign up for ngrok Pro
# Visit: https://ngrok.com/pricing

# 2. Configure static domain (e.g., mybot.ngrok.io)

# 3. Start with static domain:
ngrok http --domain=mybot.ngrok.io 5678

# 4. Register webhook once with static URL:
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "RAG Bot Messages",
    "targetUrl": "https://mybot.ngrok.io/webhook/webex-bot",
    "resource": "messages",
    "event": "created"
  }'

# 5. Never update webhook again!
```

**When to use:**
- Production deployments
- Public demos
- Regular daily use
- Multiple users

**✅ Success:** Webhook URL never changes, no maintenance

---

# Part 4: Performance Issues

Everything works but slower than expected.

---

## Symptom: Everything is slow

### What you see:
```bash
# Every operation takes forever:
# - Document upload: 5+ minutes
# - Query response: 30+ seconds  
# - n8n workflows: Timing out
# - System feels sluggish
```

### Diagnostic Checklist

Work through these systematically:

**☐ First query after startup?**
- Expected behavior (pre-load models)
- See [Models Not Pre-loaded](#fix-a-models-not-pre-loaded-cold-start)

**☐ System resources low?**
```bash
# Check Activity Monitor
# Memory pressure: Green, Yellow, or Red?
# Swap memory: How much in use?
# CPU: Sustained >90%?
```

**☐ Large documents?**
```bash
ls -lh ~/cisco-rag-demo/documents/
# Any files >5MB?
```

**☐ Many documents loaded?**
```bash
# Check document count
curl http://localhost:8000/api/v1/collections | \
  jq -r '.[] | select(.name=="cisco_docs") | .id' | \
  xargs -I {} curl -s "http://localhost:8000/api/v1/collections/{}/count"

# >1000 chunks = slower searches (normal)
```

**☐ Network issues?** (If using cloud services)
```bash
ping -c 3 8.8.8.8
# High packet loss or latency?
```

---

### Optimization Strategy 1: Pre-load Models

**Biggest impact, always do first.**

```bash
# Before any demo or heavy use:
ollama run llama3.2:3b "warmup" < /dev/null
ollama run nomic-embed-text "warmup" < /dev/null

# This loads models into RAM (one-time cost)
# All subsequent queries are fast
```

**⏱️ Impact:** Reduces first query from 15s to 5s

**✅ Implement:** Add to daily startup routine

---

### Optimization Strategy 2: Increase Podman Resources

**If you have spare RAM, allocate more to Podman.**

```bash
# Check current allocation
podman machine info | grep -E 'CPUs|Memory'

# Stop machine
podman machine stop

# Increase resources (if you have them)
# Example: Increase from 8GB to 12GB
podman machine set --cpus 8 --memory 12288

# Start machine
podman machine start

# Verify
podman machine info | grep -E 'CPUs|Memory'

# Start containers
podman start chromadb n8n
```

**⚠️ Warning:** Only allocate what you have available
- Mac with 16GB RAM → Max 12GB to Podman
- Mac with 24GB RAM → Max 16GB to Podman
- Leave 4-8GB for macOS

**✅ Success:** More resources = faster processing

---

### Optimization Strategy 3: Reduce Context Chunks

**Trade some accuracy for speed.**

**In n8n workflows:**
```javascript
// Find "Search ChromaDB" node
// Change:
n_results: 3  // Currently 3 chunks

// To:
n_results: 2  // Only 2 chunks

// Save workflow
```

**In Python scripts:**
```python
# In query_rag.py
# Find:
search_results = collection.query(
    query_embeddings=query_embedding,
    n_results=3  # Change this
)

# To:
n_results=2
```

**⏱️ Impact:** Saves 1-2 seconds per query

**Trade-off:** Less context may reduce answer quality

**✅ When to use:** For simple queries, speed demos

---

### Optimization Strategy 4: Optimize Chunk Size

**Smaller chunks = faster search but less context.**

**⚠️ Requires reloading all documents**

```python
# In load_document.py
# Change:
chunk_size = 800  # Current

# To:
chunk_size = 500  # Faster searches

# Or to:
chunk_size = 1200  # Slower but better context
```

**Process:**
```bash
# 1. Delete collection
curl -X DELETE http://localhost:8000/api/v1/collections/[UUID]

# 2. Recreate
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'

# 3. Reload all documents
for doc in ~/cisco-rag-demo/documents/*.md; do
    python3 load_document.py "$doc"
done
```

**⏱️ Time:** 10-30 minutes depending on document count

**When to optimize:**
- After testing with sample data
- Before loading production documents
- When you know your use case

**✅ Success:** Optimal chunk size for your needs

---

### Optimization Strategy 5: System Cleanup

**Free up resources on your Mac.**

```bash
# Close unnecessary applications:
# - Web browsers (especially with many tabs)
# - Unused development tools
# - Background sync (Dropbox, OneDrive)
# - Other resource-intensive apps

# Restart Mac (sometimes helps)
# Especially after many days of uptime

# Clear system caches (if desperate)
sudo periodic daily weekly monthly

# Check available RAM
# Activity Monitor → Memory tab
# Memory Pressure should be GREEN
```

**💡 Pro Tip:** Dedicate your Mac to RAG demo
- Close everything else
- Run system-check before demos
- Restart Mac if resources low

**✅ Success:** System resources available

---

## Target Performance Benchmarks

**For Mac M4 Pro with 24GB RAM:**

| Operation | First Time | Subsequent | Status |
|-----------|-----------|------------|---------|
| Document upload (5 pages) | 45-60s | 30-45s | ✅ Normal |
| Query (after warmup) | 10-15s | 5-8s | ✅ Normal |
| First query (cold) | 15-20s | N/A | ✅ Expected |
| n8n workflow execute | 8-12s | 8-12s | ✅ Normal |
| Webex bot response | 10-15s | 6-10s | ✅ Normal |

**If significantly slower than these targets:** Follow optimization strategies above

**If meeting these targets:** Performance is as expected for this hardware/model combination

---

# Part 5: Connection Issues

Problems with services talking to each other.

---

## Symptom: Container can't reach Ollama

### What you see:
```
# In n8n workflow execution error:
Connection refused to host.containers.internal:11434

# Or:
ECONNREFUSED 127.0.0.1:11434
```

### Root Cause

**Containers need special hostname to reach host services.**

**Wrong:** `localhost:11434` or `127.0.0.1:11434`  
**Right:** `host.containers.internal:11434`

---

### Fix: Use Correct Hostname

**Update all Ollama API calls in n8n:**

**Step 1: Find nodes calling Ollama**
- "Generate Embedding" nodes
- "Generate Answer" nodes
- "HTTP Request" nodes to Ollama

**Step 2: Update URLs**
```javascript
// Change from:
http://localhost:11434/api/embeddings  // ❌

// To:
http://host.containers.internal:11434/api/embeddings  // ✅
```

**Step 3: Save workflow**

**Step 4: Verify Ollama is actually running**
```bash
# On your Mac (not in container):
curl http://localhost:11434

# Should respond: "Ollama is running"

# If not:
ollama serve &
sleep 5
```

**Why this works:**
- Podman maps `host.containers.internal` to host machine
- Works from inside any container
- Containers have their own network namespace

**Network diagram:**
```
Container (n8n)
     ↓
host.containers.internal:11434
     ↓
Host machine (your Mac)
     ↓
Ollama service on localhost:11434
```

**✅ Success:** Workflow successfully calls Ollama API

---

## Symptom: Can't access n8n from browser

### What you see:
```
Browser: http://localhost:5678
Error: This site can't be reached
ERR_CONNECTION_REFUSED
```

### Diagnostic Steps

```bash
# 1. Is n8n container running?
podman ps | grep n8n

# 2. Is port mapped?
podman ps
# Look for: 0.0.0.0:5678->5678/tcp in PORTS column

# 3. Can you reach from command line?
curl http://localhost:5678
```

---

### Fix A: n8n Not Running

**Container is stopped.**

```bash
podman start n8n
sleep 10
open http://localhost:5678
```

**✅ Success:** Browser loads n8n interface

---

### Fix B: Wrong Port Mapping

**Container created with wrong port configuration.**

```bash
# Check actual port mapping
podman inspect n8n | grep -A 10 PortBindings

# If wrong, recreate container:
podman stop n8n
podman rm n8n

# Create with correct port
podman run -d --name n8n -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest

sleep 15
open http://localhost:5678
```

**✅ Success:** Port correctly mapped, accessible

---

### Fix C: Browser/Firewall Issue

**Browser security or firewall blocking.**

**Solutions to try:**
```bash
# 1. Try different browser
# Chrome/Firefox instead of Safari

# 2. Clear browser cache
# Shift+Cmd+Delete → Clear cache

# 3. Try incognito/private mode

# 4. Check system firewall
# System Settings → Network → Firewall
# Make sure n8n/podman allowed

# 5. Try IP address instead
open http://127.0.0.1:5678
```

**✅ Success:** n8n interface loads

---

## Symptom: n8n can't reach ChromaDB

### What you see:
```
# In n8n workflow error:
Connection refused to localhost:8000

# Or:
Connection refused to chromadb:8000
```

### Fix: Use Container Network Name

**From n8n container to ChromaDB:**

```javascript
// Should use container name:
http://chromadb:8000/api/v1/...  // ✅

// NOT:
http://localhost:8000/api/v1/...  // ❌ (can't reach host)
http://host.containers.internal:8000/api/v1/...  // ❌ (not on host)
```

**Why "chromadb" works:**
- Podman creates automatic DNS for container names
- Container "chromadb" is reachable as "chromadb:8000"
- Both containers on same Podman network

**Verify ChromaDB is actually running:**
```bash
podman ps | grep chromadb

# Should show STATUS: Up
```

**Test connectivity from n8n container:**
```bash
# Enter n8n container
podman exec -it n8n sh

# Try reaching ChromaDB
wget -O- http://chromadb:8000/api/v1/heartbeat

# Should return JSON heartbeat

# Exit container
exit
```

**✅ Success:** n8n can call ChromaDB API

---

# Part 6: Permission Issues

macOS + Podman = UID mismatch problems

---

## Symptom: Permission denied errors

### What you see:
```bash
$ podman logs chromadb
Error: EACCES: permission denied, mkdir '/chroma/chroma'

$ podman logs n8n
Error: EACCES: permission denied, open '/home/node/.n8n/config'
```

### Root Cause

**Podman on macOS runs in VM with different user IDs.**

**Example:**
- Your Mac user: UID 501
- Podman VM user: UID 1000
- Container user: UID 1000

**Result:** Container can't write to Mac-owned directory

---

### Universal Fix: 777 Permissions

**⚠️ This seems wrong but is correct for macOS + Podman**

```bash
# Stop containers
podman stop chromadb n8n

# Remove containers (data stays)
podman rm chromadb n8n

# Fix permissions
chmod -R 777 ~/cisco-rag-demo/chroma-data
chmod -R 777 ~/cisco-rag-demo/n8n-data

# Verify
ls -la ~/cisco-rag-demo/
# Should show rwxrwxrwx for data directories

# Recreate containers
podman run -d --name chromadb -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  chromadb/chroma:0.4.24

podman run -d --name n8n -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest

# Wait for startup
sleep 20

# Verify
podman ps
```

**Why 777 is safe here:**
- Local development environment
- Not a production server
- Isolated Mac user
- No network exposure of filesystem

**Alternative (more secure but complex):**
```bash
# Use Podman's userns option
# Adds complexity, usually not worth it for local dev
```

**✅ Success:** Containers start without permission errors

---

## Symptom: Can't write to documents directory

### What you see:
```bash
$ cp ~/Downloads/report.md ~/cisco-rag-demo/documents/
cp: cannot create regular file: Permission denied
```

### Fix: Correct Directory Permissions

```bash
# Fix documents directory
chmod 755 ~/cisco-rag-demo/documents

# Or if that doesn't work:
chmod 777 ~/cisco-rag-demo/documents

# Verify
ls -ld ~/cisco-rag-demo/documents

# Add document
cp ~/Downloads/report.md ~/cisco-rag-demo/documents/
```

**✅ Success:** Can copy files to documents directory

---

# Part 7: Data Issues

Lost data, corrupted data, unexpected data.

---

## Symptom: Lost all documents

### What you see:
```bash
# Query returns no results
# Document count shows 0
# Previously loaded documents gone
```

### Diagnostic Steps

**Check if data physically exists:**
```bash
ls -la ~/cisco-rag-demo/chroma-data/

# If shows files → Data exists, access issue
# If empty → Data deleted, no recovery
```

---

### Possible Causes

**1. ChromaDB container deleted**
- Container removed with `podman rm`
- Data persists if volume was mapped correctly
- Just need to recreate container

**2. Volume data deleted**
- Directory deleted accidentally
- `rm -rf` command
- No recovery possible without backup

**3. Wrong collection name**
- Querying different collection than documents loaded into
- Data still exists, just looking in wrong place

---

### Fix A: Container Deleted, Data Exists

**Data is there, need to recreate container.**

```bash
# Verify data exists
ls -la ~/cisco-rag-demo/chroma-data/
# Should show chroma.sqlite3 and other files

# Recreate container
podman run -d --name chromadb -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  chromadb/chroma:0.4.24

# Wait for startup
sleep 10

# Check document count
curl http://localhost:8000/api/v1/collections | \
  jq -r '.[] | select(.name=="cisco_docs") | .id' | \
  xargs -I {} curl -s "http://localhost:8000/api/v1/collections/{}/count"

# Should show previous document count
```

**✅ Success:** Documents restored

---

### Fix B: Data Directory Deleted

**Data is gone, no recovery without backup.**

```bash
# Verify
ls -la ~/cisco-rag-demo/chroma-data/
# Shows empty or doesn't exist

# No automatic recovery possible
# Must:
# 1. Recreate directory
# 2. Recreate collection  
# 3. Reload all documents

# Recreate directory
mkdir -p ~/cisco-rag-demo/chroma-data
chmod 777 ~/cisco-rag-demo/chroma-data

# Restart ChromaDB
podman restart chromadb
sleep 10

# Create collection
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'

# Reload documents
for doc in ~/cisco-rag-demo/documents/*.md; do
    echo "Loading $doc..."
    python3 load_document.py "$doc"
done
```

**⏱️ Time:** Depends on document count

**⚠️ Learn from this:** Implement backups (see Prevention below)

**✅ Result:** System restored, documents reloaded

---

### Fix C: Wrong Collection Name

**Data exists but query uses wrong collection name.**

```bash
# List all collections
curl http://localhost:8000/api/v1/collections

# Check what's actually there
# Compare to collection name in your scripts

# If different, either:
# 1. Update scripts to use correct name
# 2. Or reload to correct collection
```

**✅ Success:** Using consistent collection name

---

## Prevention: Backup Strategy

**Don't lose your work - set up automated backups.**

### Create Backup Script

```bash
cat > ~/cisco-rag-demo/backup-chromadb.sh << 'EOF'
#!/bin/bash

# Configuration
BACKUP_DIR=~/chromadb-backups
DATA_DIR=~/cisco-rag-demo/chroma-data
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="chroma-backup-${DATE}.tar.gz"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create backup
echo "Creating backup: $BACKUP_FILE"
tar -czf "$BACKUP_DIR/$BACKUP_FILE" -C ~/cisco-rag-demo chroma-data/

# Verify backup created
if [ -f "$BACKUP_DIR/$BACKUP_FILE" ]; then
    SIZE=$(ls -lh "$BACKUP_DIR/$BACKUP_FILE" | awk '{print $5}')
    echo "✅ Backup created: $SIZE"
    echo "Location: $BACKUP_DIR/$BACKUP_FILE"
else
    echo "❌ Backup failed!"
    exit 1
fi

# Keep only last 7 backups
cd "$BACKUP_DIR"
ls -t chroma-backup-*.tar.gz | tail -n +8 | xargs rm -f

echo "✅ Old backups cleaned (keeping last 7)"
EOF

chmod +x ~/cisco-rag-demo/backup-chromadb.sh
```

### Run Backups

**Manual backup:**
```bash
~/cisco-rag-demo/backup-chromadb.sh
```

**Automated (recommended):**
```bash
# Add to crontab for weekly backups
crontab -e

# Add this line:
0 2 * * 0 ~/cisco-rag-demo/backup-chromadb.sh

# Runs every Sunday at 2 AM
```

### Restore from Backup

```bash
# List available backups
ls -lh ~/chromadb-backups/

# Choose backup to restore
BACKUP=chroma-backup-20241219_140530.tar.gz

# Stop ChromaDB
podman stop chromadb

# Backup current data (just in case)
mv ~/cisco-rag-demo/chroma-data ~/cisco-rag-demo/chroma-data.old

# Extract backup
tar -xzf ~/chromadb-backups/$BACKUP -C ~/cisco-rag-demo/

# Restart ChromaDB
podman start chromadb
sleep 10

# Verify
curl http://localhost:8000/api/v1/collections
```

**✅ Prevention in place:** Regular backups protect your data

---

# Emergency Recovery Procedures

When everything is broken and you need to start fresh.

---

## Complete System Reset

**⚠️ WARNING: This deletes ALL data!**

**When to use:**
- Nothing works
- Corruption suspected
- Starting from scratch
- As last resort

**Before resetting:**
```bash
# 1. Backup anything important
~/cisco-rag-demo/backup-chromadb.sh

# 2. Copy n8n workflows
cp -r ~/cisco-rag-demo/n8n-data/workflows ~/safe-backup/

# 3. Save configuration files
cp ~/cisco-rag-demo/webex-bot-token.txt ~/safe-backup/
```

---

### Reset Procedure

```bash
# Step 1: Stop everything
echo "Stopping services..."
podman stop chromadb n8n
killall ollama

# Step 2: Remove containers
echo "Removing containers..."
podman rm chromadb n8n

# Step 3: Remove Podman machine (optional, nuclear option)
# Only if Podman itself is broken
# podman machine stop
# podman machine rm

# Step 4: Clean data directories
echo "Cleaning data (DELETES ALL DOCUMENTS)..."
read -p "Are you sure? Type 'yes' to continue: " confirm
if [ "$confirm" = "yes" ]; then
    rm -rf ~/cisco-rag-demo/chroma-data/*
    rm -rf ~/cisco-rag-demo/n8n-data/*
    echo "✅ Data cleaned"
else
    echo "Aborted"
    exit 1
fi

# Step 5: Recreate directories
mkdir -p ~/cisco-rag-demo/chroma-data
mkdir -p ~/cisco-rag-demo/n8n-data
chmod 777 ~/cisco-rag-demo/chroma-data
chmod 777 ~/cisco-rag-demo/n8n-data

# Step 6: Restart Ollama
echo "Starting Ollama..."
ollama serve &
sleep 10

# Step 7: Start Podman (if removed)
# podman machine init
# podman machine start

# Step 8: Create ChromaDB container
echo "Creating ChromaDB..."
podman run -d --name chromadb -p 8000:8000 \
  -v ~/cisco-rag-demo/chroma-data:/chroma/chroma \
  -e IS_PERSISTENT=TRUE \
  chromadb/chroma:0.4.24

# Step 9: Create n8n container
echo "Creating n8n..."
podman run -d --name n8n -p 5678:5678 \
  -v ~/cisco-rag-demo/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest

# Step 10: Wait for startup
echo "Waiting for services to start..."
sleep 30

# Step 11: Verify
echo ""
echo "Verification:"
echo "============="
podman ps
echo ""
curl http://localhost:11434
echo ""
curl http://localhost:8000/api/v1/heartbeat

echo ""
echo "✅ System reset complete"
echo ""
echo "Next steps:"
echo "1. Open http://localhost:5678 (n8n)"
echo "2. Import workflows from backup"
echo "3. Create ChromaDB collection"
echo "4. Reload documents"
echo "5. Configure Webex bot (if needed)"
```

**⏱️ Time:** 30-45 minutes

**✅ Result:** Clean system, ready to rebuild

---

## Quick System Health Check

**Run this daily or before demos.**

```bash
cat > ~/cisco-rag-demo/system-check.sh << 'EOF'
#!/bin/bash

echo "🔍 RAG System Health Check"
echo "=========================="
echo ""

# Check 1: Podman containers
echo "1. Container Status:"
podman ps --format "   {{.Names}}: {{.Status}}" 2>/dev/null || echo "   ❌ Podman not available"
echo ""

# Check 2: Ollama
echo "2. Ollama Status:"
if curl -s http://localhost:11434 > /dev/null 2>&1; then
    echo "   ✅ Ollama running"
    ollama list 2>/dev/null | head -5
else
    echo "   ❌ Ollama not responding"
fi
echo ""

# Check 3: ChromaDB
echo "3. ChromaDB Status:"
if curl -s http://localhost:8000/api/v1/heartbeat > /dev/null 2>&1; then
    echo "   ✅ ChromaDB responding"
    
    # Document count
    COLL_ID=$(curl -s http://localhost:8000/api/v1/collections 2>/dev/null | jq -r '.[] | select(.name=="cisco_docs") | .id' 2>/dev/null)
    if [ -n "$COLL_ID" ]; then
        COUNT=$(curl -s "http://localhost:8000/api/v1/collections/$COLL_ID/count" 2>/dev/null | jq -r '.count' 2>/dev/null)
        echo "   Documents: $COUNT chunks"
    else
        echo "   ⚠️  Collection 'cisco_docs' not found"
    fi
else
    echo "   ❌ ChromaDB not responding"
fi
echo ""

# Check 4: n8n
echo "4. n8n Status:"
if curl -s http://localhost:5678 > /dev/null 2>&1; then
    echo "   ✅ n8n web interface accessible"
else
    echo "   ❌ n8n not accessible"
fi
echo ""

# Check 5: Disk space
echo "5. Disk Space:"
df -h / | tail -1 | awk '{print "   Available: " $4 " (" $5 " used)"}'
echo ""

# Summary
echo "=========================="
echo "✅ System check complete"
echo ""
EOF

chmod +x ~/cisco-rag-demo/system-check.sh
```

**Usage:**
```bash
~/cisco-rag-demo/system-check.sh
```

**Expected output when healthy:**
```
🔍 RAG System Health Check
==========================

1. Container Status:
   chromadb: Up 2 hours
   n8n: Up 2 hours

2. Ollama Status:
   ✅ Ollama running
   NAME                ID           SIZE
   llama3.2:3b         abc123       2.0 GB
   nomic-embed-text    def456       274 MB

3. ChromaDB Status:
   ✅ ChromaDB responding
   Documents: 42 chunks

4. n8n Status:
   ✅ n8n web interface accessible

5. Disk Space:
   Available: 125Gi (35% used)

==========================
✅ System check complete
```

**✅ Use this:** Before every demo, after restarts, when troubleshooting

---

# Prevention Checklist

**Avoid 90% of issues by following these practices:**

## Daily Operations

- [ ] Run system-check.sh before starting work
- [ ] Pre-load models if doing queries/demos
- [ ] Check tunnel URL if using Webex bot
- [ ] Verify containers running after Mac restart
- [ ] Close resource-intensive apps during demos

## Weekly Maintenance

- [ ] Run ChromaDB backup script
- [ ] Check available disk space
- [ ] Review n8n execution logs for errors
- [ ] Update tunnel URL if using free tier
- [ ] Restart Mac to clear memory

## Best Practices

- [ ] Never use `podman rm` without stopping first
- [ ] Always chmod 777 data directories on macOS
- [ ] Use host.containers.internal for container→host
- [ ] Use container names for container→container
- [ ] Keep source documents in separate backup
- [ ] Document any configuration changes
- [ ] Test changes in isolation before combining
- [ ] Take notes on what fixes worked

## Before Demos

- [ ] Run ~/start-rag.sh (or equivalent startup)
- [ ] Pre-load both AI models
- [ ] Verify n8n workflows are active
- [ ] Test one query end-to-end
- [ ] Check system resources (Activity Monitor)
- [ ] Have backup queries prepared
- [ ] Know where troubleshooting guide is

## Documentation

- [ ] Keep this guide bookmarked
- [ ] Note any new issues encountered
- [ ] Save successful fixes for reference
- [ ] Share learnings with team
- [ ] Update scripts with improvements

---

# Getting Additional Help

**When this guide doesn't solve your problem:**

---

## Before Asking for Help

**Gather this information first - saves everyone time:**

### 1. System Information
```bash
# Hardware & OS
system_profiler SPHardwareDataType | grep -E "Model|Chip|Memory"
sw_vers

# Save output
```

### 2. Service Status
```bash
# Run health check
~/cisco-rag-demo/system-check.sh > ~/system-status.txt

# Container details
podman ps -a >> ~/system-status.txt
podman machine list >> ~/system-status.txt
```

### 3. Error Logs
```bash
# Get logs from failed services
podman logs chromadb > ~/chromadb-logs.txt 2>&1
podman logs n8n > ~/n8n-logs.txt 2>&1

# If Ollama issue
ps aux | grep ollama >> ~/system-status.txt
```

### 4. What You Were Doing
Write down:
```
1. What I was trying to do: [specific task]
2. What I expected: [expected result]
3. What actually happened: [actual result]
4. Error message (exact text): [copy/paste]
5. Steps to reproduce:
   - Step 1
   - Step 2
   - Step 3
6. What I've tried:
   - Tried X, didn't work
   - Tried Y, didn't work
7. When it last worked: [time/date or "never"]
```

---

## Help Resources (In Order)

### 1. Re-read This Guide
- Use Cmd+F to search for error message
- Check related symptoms
- Review diagnostic flowcharts

### 2. Check Other Project Documentation
- [GUIDE_1_ENVIRONMENT_SETUP.md](GUIDE_1_ENVIRONMENT_SETUP.md) - Setup issues
- [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md) - RAG workflow issues
- [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md) - Bot issues
- [LESSONS_LEARNED_Troubleshooting_Chronicle.md](LESSONS_LEARNED_Troubleshooting_Chronicle.md) - Real troubleshooting stories

### 3. Search Error Messages
```bash
# Copy exact error message
# Search: "error message" + podman + macOS
# Or: "error message" + ollama
# Or: "error message" + chromadb + v0.4.24
```

**Good sources:**
- GitHub Issues for specific tools
- Stack Overflow
- Reddit r/selfhosted, r/LocalLLaMA
- Podman/Ollama documentation

### 4. Community Forums
- Podman: https://github.com/containers/podman/discussions
- Ollama: https://github.com/ollama/ollama/discussions
- n8n: https://community.n8n.io
- ChromaDB: https://discord.gg/MMeYNTmh3x

### 5. Create GitHub Issue
**If you've exhausted other options:**

**What to include:**
```markdown
## System Information
- Mac Model: [M4 Pro / M3 / etc]
- RAM: [24GB / 16GB / etc]
- macOS Version: [14.5 / etc]
- Podman Version: [output of `podman --version`]

## Issue Description
[Clear description of problem]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Error occurs]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Error Messages
```
[Paste exact error messages]
```

## Logs
[Attach log files]

## What I've Tried
- [Thing 1] - didn't work
- [Thing 2] - didn't work

## Additional Context
[Anything else relevant]
```

---

## What NOT to Do

❌ Don't ask: "It doesn't work, help"
✅ Do include: System info, exact error, what you tried

❌ Don't: Post screenshots of text
✅ Do: Copy/paste actual text (can be searched)

❌ Don't: Ask the same question in multiple places
✅ Do: Wait for responses, provide requested info

❌ Don't: Get frustrated with helpers
✅ Do: Remember they're volunteers

❌ Don't: Disappear after asking
✅ Do: Report back if solution works

---

## Contributing Your Solutions

**Found a fix not in this guide?**

Please share it! This helps everyone:

1. Document the solution
2. Note what caused it
3. List the fix steps
4. Submit update to guide
5. Help the community

**Everyone benefits from shared knowledge.**

---

# Summary

## Key Takeaways

**Remember these principles:**

1. **Don't panic** - Most issues are simple
2. **Check basics first** - Containers running? Ollama running?
3. **Change one thing at a time** - Isolate the problem
4. **Verify each fix** - Confirm it worked
5. **Document solutions** - Help future you
6. **Use system-check.sh** - Your first diagnostic tool
7. **Pre-load models** - Prevents 90% of performance issues
8. **chmod 777 on macOS** - Fixes most permission issues
9. **host.containers.internal** - For container→host connections
10. **Backup regularly** - Protects your work

---

## Most Common Issues (Quick Reference)

| Symptom | Most Likely Cause | Quick Fix |
|---------|------------------|-----------|
| Container won't start | Permission denied | `chmod 777 [data-dir]` |
| Ollama not responding | Not started | `ollama serve &` |
| Models missing | Not downloaded | `ollama pull [model]` |
| ChromaDB not accessible | Container stopped | `podman start chromadb` |
| n8n won't load | Container stopped | `podman start n8n` |
| Query very slow | Models not pre-loaded | Pre-load before queries |
| No query results | No documents loaded | Load documents first |
| Bot doesn't respond | Tunnel URL changed | Update webhook |
| Infinite loop | Bot not filtering itself | Add person ID filter |
| Can't reach Ollama (container) | Wrong hostname | Use `host.containers.internal` |

---

## Final Words

**This guide represents hours of troubleshooting distilled into solutions.**

**You will encounter issues.** That's normal. Software is complex, especially when combining multiple systems.

**Use this guide as your first resource.** Most problems have solutions already documented here.

**Share your experiences.** If you find new issues or better solutions, contribute them back to this guide.

**Keep learning.** Each problem you solve makes you better at diagnosing the next one.

---

**Good luck! You've got this. 🎯**
