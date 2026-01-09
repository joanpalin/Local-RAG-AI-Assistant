# Guide 3: Webex Integration
## Connecting Your RAG System to Enterprise Messaging

**Part 3 of 3: Implementation Guides**  
**Time Required:** 45 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Completed Parts 1 & 2 (Environment + RAG System)

---

## Table of Contents

1. [Introduction](#introduction)
2. [Part 3A: What Webex Integration Adds](#part-3a-what-webex-integration-adds)
3. [Part 3B: Create Webex Bot](#part-3b-create-webex-bot)
4. [Part 3C: Setup Tunnel](#part-3c-setup-tunnel)
5. [Part 3D: n8n Webex Workflow](#part-3d-n8n-webex-workflow)
6. [Part 3E: Register Webhook](#part-3e-register-webhook)
7. [Part 3F: Test Your Bot](#part-3f-test-your-bot)
8. [Part 3G: Daily Operations](#part-3g-daily-operations)
9. [Part 3H: Troubleshooting](#part-3h-troubleshooting-webex-integration)
10. [Part 3I: Final System Test](#part-3i-final-system-test)
11. [Next Steps](#next-steps)

---

## Introduction

🎉 **Welcome to the final guide!** You've built an impressive local RAG system that can answer questions about your documents using AI. Now you'll add one more capability: **access from anywhere via Webex Teams**.

**What you've already accomplished:**
- ✅ Complete RAG system running locally
- ✅ Documents loaded and searchable
- ✅ Python scripts for automation
- ✅ n8n chat interface for demos

**What this guide adds:**
- 📱 **Mobile access** - Query from phone/tablet via Webex app
- 👥 **Team collaboration** - Share bot with colleagues
- 💼 **Enterprise-ready** - IT-approved messaging platform
- 🌍 **Access anywhere** - As long as your local machine is on

**Time investment:** ~45 minutes to complete setup

---

### Before You Begin

**Verify prerequisites from Parts 1 & 2:**

```bash
# Quick system check
podman ps | grep -E "chromadb|n8n"  # Should show both running
curl http://localhost:11434/api/tags  # Should show models
ollama list  # Should show llama3.2:3b and nomic-embed-text

# Test RAG works
cd ~/cisco-rag-demo
./query_rag.py "test query"  # Should return AI answer
```

**All working?** ✅ Continue!  
**Issues?** Return to Parts 1 & 2 to fix.

---

## Part 3A: What Webex Integration Adds

**Time:** 10 minutes (reading and understanding)

### Why Add Webex Integration?

**Your current setup:**
- ✅ Works great on your local machine
- ✅ Chat interface in n8n
- ✅ Python scripts for queries
- ✗ Only accessible when sitting at your machine
- ✗ Not mobile-friendly
- ✗ Can't share with team easily

**After Webex integration:**
- ✅ Query from anywhere (via Webex mobile app)
- ✅ Share bot with team members
- ✅ Works in group spaces (with @mentions)
- ✅ Familiar interface (no training needed)
- ✅ Still 100% private (all processing on your local machine)

---

### How It Works - Simple Explanation

**The complete flow:**

```
[1] You type message in Webex Teams app
      ↓
[2] Message sent to Webex Cloud
      ↓
[3] Webex sends notification (webhook) to your machine
      ↓
[4] Tunnel forwards through your firewall
      ↓
[5] n8n on your machine receives notification
      ↓
[6] n8n fetches full message from Webex
      ↓
[7] n8n processes through RAG system
      ↓  (Same RAG system from Part 2!)
[8] AI generates answer using local Ollama
      ↓
[9] n8n sends answer back to Webex API
      ↓
[10] Answer appears in your Webex chat
```

**Total time:** 5-10 seconds  
**Where processing happens:** On your local machine  
**Data privacy:** Never leaves your infrastructure

---

### Architecture Diagram

**Visual representation:**

```
┌────────────────────────────────────────────────────┐
│                  WEBEX CLOUD                       │
│  (Cisco's servers - handles message routing)       │
└────────┬────────────────────────────────┬─────────-┘
         │                                │
         │ [3] Webhook POST               │ [9] API POST
         │                                │
         ↓                                ↑
┌────────────────────────────────────────────────────┐
│            TUNNEL (localhost.run/ngrok)            │
│  (Secure pathway through your firewall)            │
└────────┬──────────────────────────────────────────-┘
         │
         │ Forwards to localhost:5678
         │
         ↓
┌────────────────────────────────────────────────────┐
│               YOUR LOCAL MACHINE                   │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │              n8n (port 5678)                 │  │
│  │  Receives webhook, processes, responds       │  │
│  └──────┬───────────────────────────────────────┘  │
│         │                                          │
│         ├──→ [ChromaDB] Vector search              │
│         ├──→ [Ollama] AI processing                │
│         └──→ [Webex API] Send response             │
│                                                    │
│  ALL PROCESSING HAPPENS HERE (Private)             │
└────────────────────────────────────────────────────┘
```

---

### Key Concepts You'll Learn

**1. What is a Webex Bot?**
- Automated "team member" in Webex
- Can receive and send messages
- Controlled by your code/workflows
- Like a service account for messaging

**2. What is a Webhook?**
- Notification system (push, not pull)
- Webex says: "Here's a new message for you!"
- Like a doorbell vs checking mailbox
- HTTP POST to your endpoint

**3. What is a Tunnel?**
- Secure pathway through firewall
- Makes local service (n8n) accessible from internet
- Like port forwarding but easier
- Required because your machine is behind firewall

**Network analogy:**
- **Webhook** = SNMP trap (push notification)
- **Tunnel** = VPN tunnel (secure pathway)
- **Bot** = Automated monitoring script

---

### What's Different About This Part

**New complexity:**
- ⚠️ Requires internet connectivity (for tunnel)
- ⚠️ Free tunnels change URLs (normal behavior)
- ⚠️ Involves external API (Webex)
- ⚠️ More moving parts to coordinate

**But don't worry!**
- Still fully private (processing on your machine)
- Free tier is sufficient
- We'll walk through everything step-by-step
- Includes complete troubleshooting guide

---

### Success Criteria

**You'll know this part is complete when:**
- [ ] Webex bot created and token saved
- [ ] Tunnel running and accessible
- [ ] n8n workflow configured and active
- [ ] Webhook registered with Webex
- [ ] Can message bot from Webex mobile app
- [ ] Bot responds with accurate answers
- [ ] Response time under 10 seconds

Let's get started! ⚡

---

## Part 3B: Create Webex Bot

**Time:** 10 minutes  
**What you'll do:** Register a bot account with Cisco Webex  
**Why:** Bots can receive/send messages programmatically

---

### Understanding Bots vs User Accounts

**Bot account:**
- ✅ Programmatic control (API-driven)
- ✅ Always-on (no human login needed)
- ✅ Free tier available
- ✅ Can be in multiple spaces
- ✗ Can't initiate conversations (must be added)
- ✗ Different permissions than users

**Regular user account:**
- ✅ Full Webex features
- ✅ Can create spaces
- ✗ Requires human to login
- ✗ Can't be controlled by code

**For our purposes:** Bot account is perfect!

---

### Step-by-Step Bot Creation

**Step 1: Access Webex Developer Portal**

Open your browser and navigate to:
```
https://developer.webex.com
```

**Login with your Webex credentials:**
- If you have Webex: Use existing account
- If not: Create free account (no credit card needed)

**You'll land on the developer homepage.**

---

**Step 2: Navigate to Bot Creation**

**Click on your profile icon** (top right corner)

**Select "My Webex Apps"**

You'll see a page showing any existing apps/bots

**Click the "Create a New App" button**

---

**Step 3: Choose "Create a Bot"**

**You'll see several options:**
- Create a Bot ← Choose this one
- Create an Integration
- Create an OAuth Integration

**Click "Create a Bot"**

---

**Step 4: Fill in Bot Details**

**Form fields explained:**

**Bot name:** Display name users will see
```
Example: RAG Assistant
Tip: Choose a descriptive, professional name
```

**Bot username:** Unique identifier (becomes the email)
```
Example: rag-assistant
Rules:
- Lowercase only
- No spaces (use hyphens)
- Must be unique across all Webex bots
- Becomes: rag-assistant@webex.bot
```

**Icon:** Upload a logo (optional but professional)
```
Tips:
- 512x512 pixels recommended
- PNG or JPG format
- Use your company logo or generic AI icon
```

**Description:** What the bot does
```
Example: "AI assistant that answers questions about our network documentation using local RAG system. All processing happens privately on local infrastructure."

Tip: Clear description helps users know how to use it
```

---

**Step 5: Save Your Bot Token**

**CRITICAL:** After clicking "Add Bot", you'll see the bot access token.

**⚠️ THIS IS THE ONLY TIME YOU'LL SEE THIS TOKEN ⚠️**

**Your bot token looks like:**
```
YmJkYzJjZjAtMjZhNy00YTUwLWIwMzUtZGJmZmFhZjY1MDZjMmU3OTZmYzctNTI1_PF84_1eb65fdf-9643-417f-9974-ad72cae0e10f
```

**Action - Save it immediately:**

```bash
# Create secure credentials file
mkdir -p ~/cisco-rag-demo/credentials
chmod 700 ~/cisco-rag-demo/credentials

# Save bot token
cat > ~/cisco-rag-demo/credentials/webex_bot.txt << 'EOF'
BOT_TOKEN=<PASTE_YOUR_TOKEN_HERE>
BOT_EMAIL=<your-bot-username>@webex.bot
EOF

chmod 600 ~/cisco-rag-demo/credentials/webex_bot.txt
```

**Replace placeholders:**
- `<PASTE_YOUR_TOKEN_HERE>` - Your actual token
- `<your-bot-username>` - The username you chose

---

**Step 6: Verify Bot Creation**

**Check your bot works:**

```bash
# Source your credentials
source ~/cisco-rag-demo/credentials/webex_bot.txt

# Test API access
curl -X GET https://webexapis.com/v1/people/me \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json"
```

**Expected output:**
```json
{
  "id": "Y2lzY29zcGFyazovL3VzL1BFT1BMRS8...",
  "emails": ["your-bot@webex.bot"],
  "displayName": "RAG Assistant",
  "nickName": "RAG Assistant",
  "created": "2024-12-18T...",
  "status": "active",
  "type": "bot"
}
```

**What to verify:**
- ✅ `"type": "bot"` - Confirms it's a bot account
- ✅ `"status": "active"` - Bot is active
- ✅ Your bot name appears correctly

---

### Success Checkpoint

✓ Your Webex bot is ready when:
- Bot created on developer portal
- Bot token saved securely
- API test returns bot information
- Bot email address known

**Save this info - you'll need it throughout this guide!**

---

## Part 3C: Setup Tunnel

**Time:** 10 minutes  
**What you'll do:** Create secure tunnel to expose n8n to internet  
**Why:** Webex cloud needs to reach your local n8n instance

---

### Understanding Why We Need a Tunnel

**The problem:**
```
Webex Cloud (internet) ──✗──> Your machine (behind firewall/router)
```

**Your n8n runs on `localhost:5678`** but:
- Not accessible from internet
- Behind your firewall/NAT
- Doesn't have public IP

**The solution: Tunnel**
```
Webex Cloud ──✓──> Tunnel Service ──✓──> localhost:5678 (n8n)
```

**Tunnel provides:**
- Public HTTPS URL
- Automatic SSL certificate
- Forwards traffic to local port
- Works through firewalls

---

### Tunnel Options

**We'll cover two popular options:**

**1. localhost.run (Easiest)**
- ✅ No signup required
- ✅ One command to start
- ✅ Completely free
- ✗ URL changes each restart
- ✗ Limited to SSH connection

**2. ngrok (More features)**
- ✅ Free tier available
- ✅ Stable URL (with account)
- ✅ Web interface for monitoring
- ✗ Requires signup
- ✗ Free tier has connection limits

**Recommendation:** Start with localhost.run (simpler), upgrade to ngrok if needed.

---

### Option 1: localhost.run (Recommended for Beginners)

**No installation needed - uses SSH!**

**Step 1: Start Tunnel**

```bash
# Open new terminal window
ssh -R 80:localhost:5678 localhost.run
```

**What happens:**
1. Connects to localhost.run servers
2. Creates tunnel to your port 5678
3. Gives you public URL

**Expected output:**
```
Connect to http://abc123xyz.localhost.run
or https://abc123xyz.localhost.run
```

**⚠️ Save this URL! You'll need it for webhook registration.**

---

**Step 2: Verify Tunnel Works**

**Open new browser tab:**
```
https://abc123xyz.localhost.run
```

**You should see:** Your n8n login page

**If not:**
- Check n8n is running: `podman ps | grep n8n`
- Verify port 5678: `curl http://localhost:5678`
- Try restarting tunnel

---

**Step 3: Keep Tunnel Running**

**⚠️ Important:** Keep this terminal window open!

**Tunnel runs as long as SSH connection stays alive**

**To reconnect after close:**
```bash
ssh -R 80:localhost:5678 localhost.run
# Note: You'll get a NEW URL each time
```

---

### Option 2: ngrok (Advanced Option)

**Step 1: Install ngrok**

**Visit:** https://ngrok.com/download

**For macOS:**
```bash
brew install ngrok/ngrok/ngrok
```

**For Linux:**
```bash
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | \
  sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && \
  echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | \
  sudo tee /etc/apt/sources.list.d/ngrok.list && \
  sudo apt update && sudo apt install ngrok
```

---

**Step 2: Create Free Account**

1. Go to https://ngrok.com
2. Sign up (free tier)
3. Get your authtoken from dashboard

**Add authtoken:**
```bash
ngrok config add-authtoken <your-token-here>
```

---

**Step 3: Start Tunnel**

```bash
ngrok http 5678
```

**Expected output:**
```
ngrok                                                                                        
                                                                                             
Session Status                online                                                         
Account                       Your Name (Plan: Free)                                         
Version                       3.5.0                                                          
Region                        United States (us)                                             
Latency                       23ms                                                           
Web Interface                 http://127.0.0.1:4040                                          
Forwarding                    https://abc123def456.ngrok-free.app -> http://localhost:5678  
                                                                                             
Connections                   ttl     opn     rt1     rt5     p50     p90                    
                              0       0       0.00    0.00    0.00    0.00                   
```

**Your public URL:** `https://abc123def456.ngrok-free.app`

**⚠️ Save this URL!**

---

**Step 4: Access ngrok Web Interface**

**Open browser:** `http://127.0.0.1:4040`

**Shows:**
- Real-time requests
- Request/response details
- Replay requests
- Statistics

**Very helpful for debugging!**

---

**Step 5: Static URLs (Optional - Paid)**

**Free tier:** URL changes each restart
**Paid tiers:** Get custom domain that never changes

**Example:**
```bash
ngrok http 5678 --domain=my-rag-bot.ngrok.app
```

**Benefit:** Never update webhook URL again

---

### Which Tunnel Should You Use?

**Use localhost.run if:**
- ✅ Just testing/demoing
- ✅ Want simplest setup
- ✅ Don't mind URL changing
- ✅ Don't need monitoring

**Use ngrok if:**
- ✅ Need monitoring/debugging
- ✅ Want web interface
- ✅ Planning production use
- ✅ Want custom domain (paid)

**Both work perfectly for this guide!**

---

### Tunnel Management Script

**Create helper script:**

```bash
cat > ~/cisco-rag-demo/start-tunnel.sh << 'EOF'
#!/bin/bash
# Tunnel startup script

echo "Starting tunnel to n8n (port 5678)..."
echo ""
echo "Choose tunnel service:"
echo "1) localhost.run (simpler, free)"
echo "2) ngrok (more features)"
echo ""
read -p "Enter choice (1 or 2): " choice

if [ "$choice" = "1" ]; then
    echo ""
    echo "Starting localhost.run tunnel..."
    echo "Keep this window open!"
    echo ""
    ssh -R 80:localhost:5678 localhost.run
elif [ "$choice" = "2" ]; then
    echo ""
    echo "Starting ngrok tunnel..."
    echo "Keep this window open!"
    echo "Web interface: http://127.0.0.1:4040"
    echo ""
    ngrok http 5678
else
    echo "Invalid choice"
    exit 1
fi
EOF

chmod +x ~/cisco-rag-demo/start-tunnel.sh
```

**Usage:**
```bash
~/cisco-rag-demo/start-tunnel.sh
```

---

### Success Checkpoint

✓ Tunnel is working when:
- Tunnel service running (terminal shows connection)
- Public HTTPS URL received
- Can access n8n through tunnel URL
- n8n login page visible from phone

**⚠️ Remember: Tunnel must stay running for bot to work!**

---

## Part 3D: n8n Webex Workflow

**Time:** 15 minutes  
**What you'll do:** Create n8n workflow that handles Webex messages  
**Complexity:** Most complex workflow yet (12 nodes)

---

### Understanding the Workflow

**What this workflow does:**

```
[1] Webhook          → Receives notification from Webex
[2] Get Message      → Fetches full message content
[3] Extract Question → Parses user's question
[4] Generate Embed   → Converts to vector
[5] Get Collection   → Finds ChromaDB collection
[6] Search Vectors   → Finds relevant chunks
[7] Build Context    → Combines chunks
[8] Generate Answer  → AI creates response
[9] Send to Webex    → Posts answer back
```

**Visual workflow:**

```
┌─────────────────────────────────────────────────────┐
│           WEBEX BOT WORKFLOW                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Webex → Webhook → Get Msg → Extract → Embed        │
│                                           ↓         │
│                                    Get Collection   │
│                                           ↓         │
│                                    Search Vectors   │
│                                           ↓         │
│                                    Build Context    │
│                                           ↓         │
│                                    Generate Answer  │
│                                           ↓         │
│                                    Send to Webex    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### Create Workflow JSON

**This is a complete, working workflow:**

```bash
cat > ~/cisco-rag-demo/workflows/webex_rag_bot.json << 'EOF'
{
  "name": "Webex RAG Bot",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "webex-webhook",
        "responseMode": "responseNode",
        "options": {}
      },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [240, 300],
      "webhookId": "webex-bot"
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.body.data.personEmail }}",
              "operation": "notContains",
              "value2": "@webex.bot"
            }
          ]
        }
      },
      "name": "Filter Bot Messages",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [460, 300]
    },
    {
      "parameters": {
        "url": "=https://webexapis.com/v1/messages/{{ $json.body.data.id }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {}
      },
      "name": "Get Message Details",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [680, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "1",
          "name": "Webex Bot Token"
        }
      }
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "question",
              "value": "={{ $json.text }}"
            },
            {
              "name": "roomId",
              "value": "={{ $json.roomId }}"
            },
            {
              "name": "personEmail",
              "value": "={{ $json.personEmail }}"
            }
          ]
        },
        "options": {}
      },
      "name": "Extract Data",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [900, 300]
    },
    {
      "parameters": {
        "url": "http://host.containers.internal:11434/api/embeddings",
        "method": "POST",
        "jsonParameters": true,
        "options": {
          "timeout": 30000
        },
        "bodyParametersJson": "={\n  \"model\": \"nomic-embed-text\",\n  \"prompt\": \"{{ $json.question }}\"\n}"
      },
      "name": "Generate Embedding",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [1120, 300]
    },
    {
      "parameters": {
        "url": "http://host.containers.internal:8000/api/v1/collections",
        "options": {}
      },
      "name": "Get Collection",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [1340, 300]
    },
    {
      "parameters": {
        "url": "=http://host.containers.internal:8000/api/v1/collections/{{ $item(0).json[0].id }}/query",
        "method": "POST",
        "jsonParameters": true,
        "options": {},
        "bodyParametersJson": "={\n  \"query_embeddings\": [{{ $('Generate Embedding').item.json.embedding }}],\n  \"n_results\": 3\n}"
      },
      "name": "Search ChromaDB",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [1560, 300]
    },
    {
      "parameters": {
        "functionCode": "// Build context from retrieved documents\nconst results = items[0].json;\nconst documents = results.documents[0];\nconst metadatas = results.metadatas[0];\n\n// Format context\nconst context = documents.map((doc, i) => {\n  const meta = metadatas[i];\n  return `Source: ${meta.source}\\nChunk ${meta.chunk_index}/${meta.total_chunks}\\n\\n${doc}`;\n}).join('\\n\\n---\\n\\n');\n\n// Get question\nconst question = $('Extract Data').item.json.question;\n\n// Build prompt\nconst prompt = `Based on the following context from documents, answer the question. Keep the answer concise but informative. If the answer is not in the context, say \"I don't have enough information to answer that.\"\n\nContext:\n${context}\n\nQuestion: ${question}\n\nAnswer:`;\n\nreturn [{ json: { prompt: prompt } }];"
      },
      "name": "Build Prompt",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [1780, 300]
    },
    {
      "parameters": {
        "url": "http://host.containers.internal:11434/api/generate",
        "method": "POST",
        "jsonParameters": true,
        "options": {
          "timeout": 60000
        },
        "bodyParametersJson": "={\n  \"model\": \"llama3.2:3b\",\n  \"prompt\": \"{{ $json.prompt }}\",\n  \"stream\": false\n}"
      },
      "name": "Generate Answer",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [2000, 300]
    },
    {
      "parameters": {
        "url": "https://webexapis.com/v1/messages",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "jsonParameters": true,
        "options": {},
        "bodyParametersJson": "={\n  \"roomId\": \"{{ $('Extract Data').item.json.roomId }}\",\n  \"markdown\": \"{{ $json.response }}\"\n}"
      },
      "name": "Send to Webex",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [2220, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "1",
          "name": "Webex Bot Token"
        }
      }
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={ \"success\": true }"
      },
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [2440, 300]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={ \"success\": true, \"message\": \"Ignored bot message\" }"
      },
      "name": "Respond - Filtered",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [460, 500]
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{"node": "Filter Bot Messages", "type": "main", "index": 0}]]
    },
    "Filter Bot Messages": {
      "main": [
        [{"node": "Get Message Details", "type": "main", "index": 0}],
        [{"node": "Respond - Filtered", "type": "main", "index": 0}]
      ]
    },
    "Get Message Details": {
      "main": [[{"node": "Extract Data", "type": "main", "index": 0}]]
    },
    "Extract Data": {
      "main": [[{"node": "Generate Embedding", "type": "main", "index": 0}]]
    },
    "Generate Embedding": {
      "main": [[{"node": "Get Collection", "type": "main", "index": 0}]]
    },
    "Get Collection": {
      "main": [[{"node": "Search ChromaDB", "type": "main", "index": 0}]]
    },
    "Search ChromaDB": {
      "main": [[{"node": "Build Prompt", "type": "main", "index": 0}]]
    },
    "Build Prompt": {
      "main": [[{"node": "Generate Answer", "type": "main", "index": 0}]]
    },
    "Generate Answer": {
      "main": [[{"node": "Send to Webex", "type": "main", "index": 0}]]
    },
    "Send to Webex": {
      "main": [[{"node": "Respond to Webhook", "type": "main", "index": 0}]]
    }
  },
  "settings": {},
  "staticData": null,
  "tags": []
}
EOF
```

---

### Import Workflow into n8n

**Step 1: Open n8n**

```
http://localhost:5678
```

**Step 2: Import Workflow**

1. Click **"Add workflow"** (top right)
2. Click **"⋮"** menu → **"Import from file"**
3. Select: `~/cisco-rag-demo/workflows/webex_rag_bot.json`
4. Click **"Import"**

**You should see:** 12 nodes connected in sequence

---

### Configure Credentials

**The workflow needs your Webex bot token to:**
- Fetch message details from Webex
- Send responses back to Webex

**Step 1: Add Credentials**

1. Click **"Credentials"** in left sidebar
2. Click **"+ Add Credential"**
3. Search for **"Header Auth"**
4. Click **"Header Auth"**

**Step 2: Configure Header Auth**

**Name:** `Webex Bot Token`

**Header Name:** `Authorization`

**Header Value:** `Bearer <YOUR_BOT_TOKEN>`

**Example:**
```
Bearer YmJkYzJjZjAtMjZhNy00YTUwLWIwMzUtZGJmZmFhZjY1MDZjMmU3OTZmYzctNTI1_PF84_1eb65fdf-9643-417f-9974-ad72cae0e10f
```

**Click "Save"**

---

**Step 3: Assign Credentials to Nodes**

**Two nodes need credentials:**
1. **"Get Message Details"** node
2. **"Send to Webex"** node

**For each node:**
1. Click the node
2. Scroll to **"Authentication"** section
3. **Credential Type:** Generic Credential Type
4. **Generic Auth Type:** Header Auth
5. **Header Auth Credential:** Select "Webex Bot Token"
6. Click away to save

---

### Activate Workflow

**Step 1: Save Workflow**

**Click "Save"** button (top right)

**Step 2: Activate**

**Toggle "Active" switch** (top right)

Status should change from inactive to **Active**

**Step 3: Get Webhook URL**

1. Click **"Webhook"** node
2. Note the webhook URLs:
   - **Production URL:** This is what Webex will call
   - **Test URL:** For manual testing

**Example Production URL:**
```
http://localhost:5678/webhook/webex-webhook
```

**But remember:** This needs to be accessed through your tunnel!

**Your actual webhook URL for Webex:**
```
https://abc123xyz.localhost.run/webhook/webex-webhook
```

**⚠️ Save this complete URL - you'll need it next!**

---

### Success Checkpoint

✓ Workflow is ready when:
- All 12 nodes present and connected
- Credentials configured correctly
- Workflow activated (green toggle)
- Webhook URL known
- No red error indicators on nodes

---

## Part 3E: Register Webhook

**Time:** 5 minutes  
**What you'll do:** Tell Webex to notify your n8n when messages arrive  
**Why:** This connects Webex cloud to your local system

---

### Understanding Webhook Registration

**What we're doing:**
```
Webex Cloud: "When bot receives message, POST notification to this URL"
             ↓
Your URL: https://your-tunnel.localhost.run/webhook/webex-webhook
```

**What happens after registration:**
1. Someone messages your bot
2. Webex immediately POSTs to your URL
3. Your n8n receives notification
4. Workflow processes and responds

---

### Register Webhook via API

**Step 1: Prepare Registration Command**

```bash
# Source your bot token
source ~/cisco-rag-demo/credentials/webex_bot.txt

# Set your tunnel URL (REPLACE WITH YOUR ACTUAL URL!)
TUNNEL_URL="https://abc123xyz.localhost.run"

# Create webhook
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${TUNNEL_URL}/webhook/webex-webhook\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }"
```

**⚠️ Important:** Replace `abc123xyz.localhost.run` with YOUR actual tunnel URL!

---

**Step 2: Execute Registration**

**Run the curl command above.**

**Expected response:**
```json
{
  "id": "Y2lzY29zcGFyazovL3VzL1dFQkhPT0sv...",
  "name": "RAG Bot Messages",
  "targetUrl": "https://abc123xyz.localhost.run/webhook/webex-webhook",
  "resource": "messages",
  "event": "created",
  "filter": null,
  "secret": null,
  "status": "active",
  "created": "2024-12-18T10:30:00.000Z"
}
```

**What to verify:**
- ✅ `"status": "active"` - Webhook is active
- ✅ `"targetUrl"` matches your tunnel URL
- ✅ `"resource": "messages"` - Listens for messages
- ✅ `"event": "created"` - Triggers on new messages

**Save the webhook ID** from response - you might need it later.

---

### List Existing Webhooks

**Check what webhooks are registered:**

```bash
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**Shows all webhooks for your bot**

**Use this to:**
- Verify registration succeeded
- Find webhook IDs
- Check for duplicates

---

### Delete Webhook (If Needed)

**If you need to remove a webhook:**

```bash
# Get webhook ID from list command above
WEBHOOK_ID="<paste-webhook-id-here>"

curl -X DELETE https://webexapis.com/v1/webhooks/$WEBHOOK_ID \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**When to delete:**
- Changing tunnel URLs
- Webhook pointing to wrong endpoint
- Duplicates causing issues

---

### Webhook Management Script

**Create helper script:**

```bash
cat > ~/cisco-rag-demo/manage-webhook.sh << 'EOF'
#!/bin/bash
# Webhook management script

# Load credentials
source ~/cisco-rag-demo/credentials/webex_bot.txt

echo "Webex Webhook Manager"
echo "====================="
echo ""
echo "1) List webhooks"
echo "2) Register new webhook"
echo "3) Delete webhook"
echo "4) Test webhook"
echo ""
read -p "Choose option (1-4): " choice

case $choice in
  1)
    echo ""
    echo "Current webhooks:"
    curl -s -X GET https://webexapis.com/v1/webhooks \
      -H "Authorization: Bearer $BOT_TOKEN" | python3 -m json.tool
    ;;
  2)
    echo ""
    read -p "Enter your tunnel URL (e.g., https://abc.localhost.run): " tunnel_url
    echo ""
    echo "Registering webhook..."
    curl -X POST https://webexapis.com/v1/webhooks \
      -H "Authorization: Bearer $BOT_TOKEN" \
      -H "Content-Type: application/json" \
      -d "{
        \"name\": \"RAG Bot Messages\",
        \"targetUrl\": \"${tunnel_url}/webhook/webex-webhook\",
        \"resource\": \"messages\",
        \"event\": \"created\"
      }" | python3 -m json.tool
    ;;
  3)
    echo ""
    read -p "Enter webhook ID to delete: " webhook_id
    curl -X DELETE "https://webexapis.com/v1/webhooks/$webhook_id" \
      -H "Authorization: Bearer $BOT_TOKEN"
    echo ""
    echo "Webhook deleted"
    ;;
  4)
    echo ""
    echo "Testing webhook connection..."
    read -p "Enter your tunnel URL: " tunnel_url
    curl -X POST "${tunnel_url}/webhook/webex-webhook" \
      -H "Content-Type: application/json" \
      -d '{"test": true}'
    ;;
  *)
    echo "Invalid option"
    ;;
esac
EOF

chmod +x ~/cisco-rag-demo/manage-webhook.sh
```

**Usage:**
```bash
~/cisco-rag-demo/manage-webhook.sh
```

---

### Success Checkpoint

✓ Webhook is registered when:
- API returned success response
- Webhook ID received
- Status shows "active"
- targetUrl matches your tunnel
- Webhook appears in list command

---

## Part 3F: Test Your Bot

**Time:** 5 minutes  
**What you'll do:** Send first message and receive response  
**Excitement level:** 🎉 HIGH!

---

### Test Flow Checklist

**Before testing, verify everything is running:**

```bash
# 1. Services running
podman ps | grep -E "chromadb|n8n"  # Both should show

# 2. Ollama running
curl http://localhost:11434  # Should return "Ollama is running"

# 3. Models loaded
ollama list  # Should show llama3.2:3b and nomic-embed-text

# 4. Tunnel active
# Check terminal where tunnel is running - should show connection

# 5. n8n workflow active
# Check n8n web interface - workflow should show "Active"

# 6. Webhook registered
source ~/cisco-rag-demo/credentials/webex_bot.txt
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN"
# Should show webhook with status: active
```

**All green?** ✅ Proceed to testing!

---

### Test 1: Direct Message to Bot

**Step 1: Open Webex Teams**

- Desktop app, web app, or mobile app all work
- Make sure you're logged in

**Step 2: Find Your Bot**

1. Click **"New Message"** or **"+"**
2. In the **"To:"** field, type your bot's email
   ```
   your-bot-username@webex.bot
   ```
3. Select your bot from the list

**Step 3: Send First Message**

**Type a simple question:**
```
What is the network uptime?
```

**Press Enter/Send**

---

**Step 4: Watch the Magic! ✨**

**What happens (usually takes 5-10 seconds):**

1. **Message sent** - Shows in Webex
2. **Processing** - Bot shows "typing" indicator
3. **Response appears** - Bot replies with answer from your documents

**Expected response example:**
```
According to the Boston Network Assessment Q3 2024 report, 
the network uptime for Q3 2024 was 99.94% with a mean time 
to repair (MTTR) of 2.4 hours.
```

---

### Test 2: Query from Mobile

**Step 1: Open Webex Mobile App**

- iOS or Android
- Make sure you're logged in

**Step 2: Find Your Bot**

- Search for bot by name or email
- Open conversation

**Step 3: Ask Question**

**Try different question:**
```
What switches are installed?
```

**You should receive:** Accurate answer citing your documents

**This proves:** Mobile access works! 📱

---

### Test 3: Test in Group Space

**Step 1: Create Test Space**

1. In Webex, click **"Create a space"**
2. Name it: "RAG Bot Test"
3. Add your bot: Search for bot email, add to space

**Step 2: Send @mention**

**In the space, type:**
```
@RAG-Assistant What equipment needs upgrading?
```

**Bot should respond** with answer from documents

**This proves:** Group space functionality works! 👥

---

### Test 4: Monitor in n8n

**While testing, watch n8n:**

**Open n8n:** `http://localhost:5678`

**Go to:** "Executions" in left sidebar

**You'll see:**
- Each message as new execution
- Green checkmarks for success
- Click execution to see data flow through nodes
- View actual API responses

**This is invaluable for debugging!**

---

### Test 5: Monitor Tunnel Traffic

**If using ngrok:**

**Open:** `http://127.0.0.1:4040`

**You'll see:**
- Every HTTP request from Webex
- Request headers and body
- Response codes
- Timing information

**Perfect for troubleshooting!**

---

### Common First-Test Issues

**Issue: Bot doesn't respond at all**

**Check:**
1. Is tunnel still running? (Check terminal)
2. Is n8n workflow active? (Check toggle in n8n)
3. Is webhook registered? (Run list command)
4. Any errors in n8n executions?

---

**Issue: Bot responds slowly (>20 seconds)**

**Causes:**
1. First query loads model (normal)
2. Low RAM (check Activity Monitor)
3. Model swapping (close other apps)

**Pre-load models:**
```bash
ollama run llama3.2:3b "test"
# Press Ctrl+D to exit but keep model loaded
```

---

**Issue: Bot says "I don't have enough information"**

**Causes:**
1. No documents loaded in ChromaDB
2. Question doesn't match document content
3. ChromaDB collection not found

**Check:**
```bash
# Verify documents exist
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
# Should return number > 0
```

---

**Issue: Error in n8n execution**

**Check execution details:**
1. Open n8n
2. Go to "Executions"
3. Click failed execution
4. Look at node with red X
5. Read error message

**Common errors:**
- "Connection refused" → Ollama not running
- "Collection not found" → ChromaDB issue
- "401 Unauthorized" → Bot token wrong
- "Timeout" → Model too slow, increase timeout

---

### Success Checkpoint

✓ Your bot works when:
- Responds to direct messages
- Works from mobile app
- Responds in group spaces (with @mention)
- Answers are accurate
- Response time under 10 seconds
- Can see executions in n8n
- No errors in execution logs

**🎉 If all checks pass: YOUR BOT IS LIVE! 🎉**

---

## Part 3G: Daily Operations

**Time:** 5 minutes (learning procedures)  
**Purpose:** How to maintain and operate your bot

---

### Daily Startup Procedure

**When you start your machine or after restart:**

```bash
# 1. Start Podman machine
podman machine start

# 2. Start containers
podman start chromadb n8n

# 3. Start Ollama
ollama serve > /dev/null 2>&1 &

# 4. Wait for services
sleep 20

# 5. Pre-load models (optional but recommended)
ollama run llama3.2:3b "Ready"
# Press Ctrl+D

# 6. Start tunnel
~/cisco-rag-demo/start-tunnel.sh
# Keep terminal open!

# 7. Update webhook (if tunnel URL changed)
~/cisco-rag-demo/manage-webhook.sh
# Choose option 3 (delete old), then 2 (register new)
```

---

### Create Startup Script

**Automate most steps:**

```bash
cat > ~/cisco-rag-demo/startup.sh << 'EOF'
#!/bin/bash
# Complete system startup

echo "Starting RAG Bot System..."
echo ""

# Start Podman
echo "1. Starting Podman machine..."
podman machine start
sleep 5

# Start containers
echo "2. Starting containers..."
podman start chromadb n8n
sleep 10

# Start Ollama
echo "3. Starting Ollama..."
if ! pgrep -x "ollama" > /dev/null; then
    ollama serve > /dev/null 2>&1 &
    sleep 5
fi

# Pre-load models
echo "4. Pre-loading AI models..."
echo "Ready" | ollama run llama3.2:3b > /dev/null 2>&1

# Verify
echo ""
echo "5. Verifying services..."
echo "   Podman: $(podman machine list | grep -q 'running' && echo '✅' || echo '❌')"
echo "   ChromaDB: $(curl -s http://localhost:8000/api/v1/heartbeat &>/dev/null && echo '✅' || echo '❌')"
echo "   n8n: $(curl -s http://localhost:5678 &>/dev/null && echo '✅' || echo '❌')"
echo "   Ollama: $(curl -s http://localhost:11434 &>/dev/null && echo '✅' || echo '❌')"

echo ""
echo "✅ System ready!"
echo ""
echo "Next steps:"
echo "1. Start tunnel in separate terminal:"
echo "   ~/cisco-rag-demo/start-tunnel.sh"
echo ""
echo "2. If tunnel URL changed, update webhook:"
echo "   ~/cisco-rag-demo/manage-webhook.sh"
EOF

chmod +x ~/cisco-rag-demo/startup.sh
```

**Usage:**
```bash
~/cisco-rag-demo/startup.sh
```

---

### Daily Shutdown Procedure

**Clean shutdown when done for the day:**

```bash
# 1. Stop tunnel (Ctrl+C in tunnel terminal)

# 2. Stop containers
podman stop chromadb n8n

# 3. Stop Podman machine (optional)
podman machine stop

# 4. Stop Ollama (optional)
killall ollama
```

---

### Create Shutdown Script

```bash
cat > ~/cisco-rag-demo/shutdown.sh << 'EOF'
#!/bin/bash
# Clean system shutdown

echo "Shutting down RAG Bot System..."
echo ""

# Stop containers
echo "1. Stopping containers..."
podman stop chromadb n8n

# Stop Podman machine
echo "2. Stopping Podman machine..."
podman machine stop

# Stop Ollama
echo "3. Stopping Ollama..."
if pgrep -x "ollama" > /dev/null; then
    killall ollama
fi

echo ""
echo "✅ System shut down cleanly"
echo ""
echo "Note: You can close the tunnel terminal window"
EOF

chmod +x ~/cisco-rag-demo/shutdown.sh
```

**Usage:**
```bash
~/cisco-rag-demo/shutdown.sh
```

---

### Monitoring and Maintenance

**Check system health:**

```bash
cat > ~/cisco-rag-demo/health-check.sh << 'EOF'
#!/bin/bash
# System health check

echo "RAG Bot Health Check"
echo "===================="
echo ""

# Podman
if podman machine list | grep -q "running"; then
    echo "✅ Podman: Running"
else
    echo "❌ Podman: Not running"
fi

# Containers
if podman ps | grep -q "chromadb"; then
    echo "✅ ChromaDB: Running"
else
    echo "❌ ChromaDB: Not running"
fi

if podman ps | grep -q "n8n"; then
    echo "✅ n8n: Running"
else
    echo "❌ n8n: Not running"
fi

# Ollama
if curl -s http://localhost:11434 &>/dev/null; then
    echo "✅ Ollama: Running"
    echo "   Models: $(ollama list | grep -v NAME | wc -l | tr -d ' ')"
else
    echo "❌ Ollama: Not running"
fi

# ChromaDB documents
if curl -s http://localhost:8000/api/v1/collections &>/dev/null; then
    UUID=$(curl -s http://localhost:8000/api/v1/collections | \
           python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])" 2>/dev/null)
    if [ -n "$UUID" ]; then
        COUNT=$(curl -s "http://localhost:8000/api/v1/collections/$UUID/count")
        echo "✅ Documents: $COUNT chunks loaded"
    fi
fi

# n8n workflows
echo "   n8n workflows: Check http://localhost:5678"

# Tunnel (manual check)
echo ""
echo "Manual checks:"
echo "- Is tunnel running? (Check separate terminal)"
echo "- Is webhook registered? Run: ~/cisco-rag-demo/manage-webhook.sh"

echo ""
echo "===================="
EOF

chmod +x ~/cisco-rag-demo/health-check.sh
```

**Usage:**
```bash
~/cisco-rag-demo/health-check.sh
```

---

### Updating Documents

**Add new documents to your system:**

```bash
# Upload single document
python3 ~/cisco-rag-demo/scripts/load_document.py new_doc.txt

# Upload multiple documents
for doc in ~/documents/*.txt; do
    python3 ~/cisco-rag-demo/scripts/load_document.py "$doc"
done
```

---

### Backup Strategy

**Backup your ChromaDB data:**

```bash
cat > ~/cisco-rag-demo/backup.sh << 'EOF'
#!/bin/bash
# Backup ChromaDB data

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/cisco-rag-demo/backups
mkdir -p $BACKUP_DIR

echo "Creating backup..."
tar -czf $BACKUP_DIR/chromadb-backup-$DATE.tar.gz \
    ~/cisco-rag-demo/chroma-data/

echo "✅ Backup created: chromadb-backup-$DATE.tar.gz"
echo "   Location: $BACKUP_DIR"

# Optional: Delete backups older than 7 days
find $BACKUP_DIR -name "chromadb-backup-*.tar.gz" -mtime +7 -delete
EOF

chmod +x ~/cisco-rag-demo/backup.sh
```

**Run weekly:**
```bash
~/cisco-rag-demo/backup.sh
```

---

### Webhook Maintenance

**After tunnel restarts (URL changes):**

```bash
# List current webhooks
source ~/cisco-rag-demo/credentials/webex_bot.txt
curl -s -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN"

# Delete old webhook
WEBHOOK_ID="<old-webhook-id>"
curl -X DELETE "https://webexapis.com/v1/webhooks/$WEBHOOK_ID" \
  -H "Authorization: Bearer $BOT_TOKEN"

# Register new webhook with new tunnel URL
NEW_URL="https://new-tunnel-url.localhost.run"
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${NEW_URL}/webhook/webex-webhook\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }"
```

**Or use the script:**
```bash
~/cisco-rag-demo/manage-webhook.sh
```

---

### Performance Optimization

**Keep your system running smoothly:**

**1. Pre-load models at startup**
```bash
ollama run llama3.2:3b "Ready"
# Press Ctrl+D
```

**2. Monitor RAM usage**
```bash
# macOS:
# Open Activity Monitor → Memory tab

# Linux:
free -h
```

**3. Close unnecessary applications**
- Browser with many tabs
- Other AI tools
- Memory-intensive apps

**4. Restart periodically**
- Full system restart weekly
- Clears memory leaks
- Refreshes all services

---

## Part 3H: Troubleshooting Webex Integration

**Comprehensive troubleshooting guide for Webex-specific issues**

---

### Issue: Bot Not Responding

**Symptom:** Send message to bot, no response received

**Diagnostic steps:**

**1. Check all services running**
```bash
~/cisco-rag-demo/health-check.sh
```

**2. Check tunnel is active**
```bash
# In tunnel terminal, should see:
# "Connect to https://abc.localhost.run"
```

**3. Check webhook registered**
```bash
source ~/cisco-rag-demo/credentials/webex_bot.txt
curl -s -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN"
# Should show webhook with status: active
```

**4. Check n8n workflow active**
- Open n8n: `http://localhost:5678`
- Check workflow toggle is green
- Check no red error nodes

**5. Check n8n executions**
- Go to "Executions" in n8n
- Are new executions appearing?
- Click to see error details

---

**Solutions:**

**If no executions appearing in n8n:**
→ Webhook not reaching n8n
→ Check tunnel is running
→ Verify webhook targetUrl matches tunnel URL

**If executions failing:**
→ Click execution, find red node
→ Read error message
→ Apply specific fix below

---

### Issue: Tunnel URL Changed

**Symptom:** Webhook points to old URL, bot stops working

**Happens when:**
- Tunnel disconnected/reconnected
- Computer restarted
- Using free tunnel service

**Solution:**

**1. Get new tunnel URL**
```bash
# From tunnel terminal output
# Example: https://new-abc123.localhost.run
```

**2. Delete old webhook**
```bash
source ~/cisco-rag-demo/credentials/webex_bot.txt

# List webhooks
curl -s -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN"

# Delete old one
WEBHOOK_ID="<paste-old-webhook-id>"
curl -X DELETE "https://webexapis.com/v1/webhooks/$WEBHOOK_ID" \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**3. Register new webhook**
```bash
NEW_URL="https://new-abc123.localhost.run"
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${NEW_URL}/webhook/webex-webhook\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }"
```

**4. Test immediately**
- Send message to bot
- Should work with new URL

---

### Issue: Bot Token Invalid

**Symptom:** "401 Unauthorized" errors in n8n

**Causes:**
- Token expired (rare, bot tokens last indefinitely)
- Token copied incorrectly
- Wrong token in credentials

**Solution:**

**1. Verify token format**
```bash
cat ~/cisco-rag-demo/credentials/webex_bot.txt
```

Should look like:
```
BOT_TOKEN=YmJkYzJjZjAtMjZhNy00YTUwLWIwMzUtZGJmZmFhZjY1MDZjMmU3OTZmYzctNTI1_PF84_1eb65fdf-9643-417f-9974-ad72cae0e10f
```

**2. Test token works**
```bash
source ~/cisco-rag-demo/credentials/webex_bot.txt
curl -X GET https://webexapis.com/v1/people/me \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**Should return bot info, not 401 error**

**3. Update n8n credentials**
- Open n8n
- Go to "Credentials"
- Find "Webex Bot Token"
- Update with correct token
- Save

---

### Issue: Tunnel Connection Drops

**Symptom:** Bot works then stops, tunnel terminal shows disconnection

**Free tunnel services disconnect after:**
- Period of inactivity
- Network interruption
- Service maintenance

**Solutions:**

**1. Keep-alive script**
```bash
cat > ~/cisco-rag-demo/tunnel-keepalive.sh << 'EOF'
#!/bin/bash
# Ping tunnel to keep connection alive

while true; do
    sleep 300  # Every 5 minutes
    curl -s http://localhost:5678 > /dev/null
done
EOF

chmod +x ~/cisco-rag-demo/tunnel-keepalive.sh

# Run in background
~/cisco-rag-demo/tunnel-keepalive.sh &
```

**2. Auto-reconnect wrapper**
```bash
cat > ~/cisco-rag-demo/tunnel-persistent.sh << 'EOF'
#!/bin/bash
# Auto-reconnect tunnel if it drops

while true; do
    echo "Starting tunnel..."
    ssh -R 80:localhost:5678 localhost.run
    echo "Tunnel disconnected. Reconnecting in 10 seconds..."
    sleep 10
done
EOF

chmod +x ~/cisco-rag-demo/tunnel-persistent.sh

# Use this instead of direct tunnel command
~/cisco-rag-demo/tunnel-persistent.sh
```

**3. Upgrade to paid tunnel**
- ngrok Pro: Static URL, no disconnects
- Considered if using in production

---

### Issue: Slow Response Time

**Symptom:** Bot takes >15 seconds to respond

**Diagnostic:**

**1. Check where time is spent**
- Open n8n execution
- Look at node execution times
- Identify slowest node

**Common bottlenecks:**

**"Generate Embedding" or "Generate Answer" slow:**
```bash
# Pre-load models
ollama run llama3.2:3b "Ready"
# Keep model in RAM
```

**"Search ChromaDB" slow:**
```bash
# Check document count
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"

# If > 1000 chunks, consider splitting collections
```

**"Get Message Details" slow:**
- Network latency to Webex API
- Not much can be done (external API)
- Usually only 1-2 seconds

---

### Issue: Bot Loops (Responds to Itself)

**Symptom:** Bot sees its own messages and responds to them infinitely

**Cause:** "Filter Bot Messages" node not working

**Solution:**

**1. Check filter node**
- Open n8n workflow
- Click "Filter Bot Messages" node
- Verify condition: `personEmail` does NOT contain `@webex.bot`

**2. If filter exists but not working:**
- Bot token has wrong credentials
- Update credentials
- Reactivate workflow

**3. Test filter**
- Send message to bot
- Check n8n execution
- Verify "Respond - Filtered" path used for bot's own messages

---

### Issue: Bot Responds in Wrong Space

**Symptom:** Bot posts answer in different space than where question was asked

**Cause:** roomId not extracted correctly

**Solution:**

**1. Check "Extract Data" node**
- Should have: `roomId = {{ $json.roomId }}`

**2. Verify in execution**
- Open failed execution
- Check "Extract Data" output
- roomId should match space ID

**3. Check "Send to Webex" node**
- Should use: `{{ $('Extract Data').item.json.roomId }}`

---

### Debugging Tools

**Enable detailed logging:**

**1. n8n execution logs**
- Each node shows input/output data
- Click node to see full JSON
- Use "Execute workflow" to test

**2. Webex API tester**
- Visit: https://developer.webex.com/docs/api/getting-started
- Test individual API calls
- Verify bot has correct permissions

**3. Tunnel monitoring**
- ngrok: http://127.0.0.1:4040
- See all incoming requests
- Replay failed requests

**4. Container logs**
```bash
# n8n logs
podman logs n8n --tail 100

# ChromaDB logs
podman logs chromadb --tail 100

# Ollama logs (if running in background)
# Check system logs or restart with visible output
```

---

### Health Check Script

**Complete diagnostic script:**

```bash
cat > ~/cisco-rag-demo/webex-health-check.sh << 'EOF'
#!/bin/bash
# Comprehensive Webex bot health check

source ~/cisco-rag-demo/credentials/webex_bot.txt

echo "Webex Bot Health Check"
echo "======================"
echo ""

# 1. Bot token valid
echo "1. Testing bot token..."
if curl -s -X GET https://webexapis.com/v1/people/me \
    -H "Authorization: Bearer $BOT_TOKEN" | grep -q "id"; then
    echo "   ✅ Bot token valid"
else
    echo "   ❌ Bot token invalid or expired"
fi

# 2. Webhook registered
echo "2. Checking webhook..."
WEBHOOKS=$(curl -s -X GET https://webexapis.com/v1/webhooks \
    -H "Authorization: Bearer $BOT_TOKEN")
if echo "$WEBHOOKS" | grep -q "status"; then
    COUNT=$(echo "$WEBHOOKS" | grep -o "id" | wc -l)
    echo "   ✅ Webhooks registered: $COUNT"
    echo "$WEBHOOKS" | python3 -m json.tool
else
    echo "   ❌ No webhooks registered"
fi

# 3. Tunnel (manual check)
echo ""
echo "3. Tunnel check (manual):"
echo "   - Check tunnel terminal for connection status"
echo "   - Try accessing: [tunnel-url]/webhook/webex-webhook"

# 4. n8n workflow
echo ""
echo "4. n8n workflow:"
echo "   - Check http://localhost:5678"
echo "   - Verify workflow is Active (green toggle)"
echo "   - Check recent executions for errors"

# 5. Local services
echo ""
echo "5. Local services:"
~/cisco-rag-demo/health-check.sh

echo ""
echo "======================"
echo "Send test message to bot to verify end-to-end"
EOF

chmod +x ~/cisco-rag-demo/webex-health-check.sh
```

**Run anytime:**
```bash
~/cisco-rag-demo/webex-health-check.sh
```

---

### Getting Help

**Before asking for help, collect:**

1. **Error messages** (exact text)
2. **What you were doing** (steps taken)
3. **Health check output** (run script above)
4. **n8n execution details** (screenshot of failed node)
5. **What you've tried** (troubleshooting steps)

**Resources:**
- Part 1 & 2 troubleshooting (environment issues)
- Comprehensive troubleshooting chronicle
- Project documentation

---

## Part 3I: Final System Test

**Time:** 5 minutes  
**Purpose:** Verify complete end-to-end system

---

### Complete Verification Sequence

**Test all access methods:**

**1. Upload Document via n8n (Part 2)**
```
✓ Open n8n document upload workflow
✓ Click "Test workflow"
✓ Upload new test document
✓ Verify success message
```

**2. Query via Python Script (Part 2)**
```bash
cd ~/cisco-rag-demo
./query_rag.py "What is in the new document?"
✓ Receives accurate answer
```

**3. Ask in n8n Chat Interface (Part 2)**
```
✓ Open n8n chat workflow
✓ Click "Chat" button
✓ Ask question about document
✓ Receives accurate answer
```

**4. Message Bot in Webex (Part 3)**
```
✓ Open Webex Teams app
✓ Message bot
✓ Ask same question
✓ Receives same accurate answer
```

**5. Test from Mobile (Part 3)**
```
✓ Open Webex app on phone
✓ Message bot
✓ Receives answer
✓ Proves mobile access works
```

---

### Performance Benchmarks

**Record your system's performance:**

| Task | Target Time | Your Time |
|------|-------------|-----------|
| Document upload (5 chunks) | 30-60 sec | _____ |
| Python query | 5-8 sec | _____ |
| n8n chat query | 5-8 sec | _____ |
| Webex bot response | 5-10 sec | _____ |
| Mobile query | 5-10 sec | _____ |

**All within targets?** ✅ Excellent performance!

---

### Success Criteria - Final Checklist

**Environment (Part 1):**
- [ ] Podman containers running
- [ ] Ollama models loaded
- [ ] ChromaDB accessible
- [ ] n8n accessible
- [ ] Python environment ready

**RAG System (Part 2):**
- [ ] Python scripts work
- [ ] Documents loaded
- [ ] n8n upload workflow works
- [ ] n8n chat workflow works
- [ ] Answers are accurate

**Webex Integration (Part 3):**
- [ ] Webex bot created
- [ ] Tunnel running
- [ ] n8n workflow configured
- [ ] Webhook registered
- [ ] Bot responds in Webex
- [ ] Mobile access works
- [ ] Response time acceptable (<10 sec)

**System Integration:**
- [ ] Same documents accessible all methods
- [ ] Consistent answers across interfaces
- [ ] Can demo reliably
- [ ] Team members can use bot
- [ ] Ready for production use

---

### What You've Accomplished

🎉 **CONGRATULATIONS!** You've built a complete, production-ready system!

**Your achievement:**

✅ **Complete local RAG system**
- Document upload and processing
- Vector search and retrieval
- AI-powered question answering
- Multiple access interfaces

✅ **Enterprise messaging integration**
- Webex Teams bot
- Mobile access
- Team collaboration
- @mention support

✅ **Full privacy**
- All processing on your local machine
- No data sent to external AI
- HIPAA/SOC2 friendly
- Complete control

✅ **Multiple access methods**
- Python scripts (automation)
- n8n workflows (visual)
- Chat interface (demos)
- Webex bot (enterprise)

✅ **Production-ready**
- Comprehensive documentation
- Troubleshooting guides
- Daily operations procedures
- Performance optimized

---

### System Capabilities Summary

**What your system can do:**

**Document Processing:**
- Load .txt, .md, .doc, .docx files
- Automatic chunking and embedding
- Vector storage in ChromaDB
- Searchable knowledge base

**Query Processing:**
- Natural language questions
- Semantic search (meaning, not keywords)
- Context-aware AI responses
- Cite specific document sections

**Access Methods:**
- Command-line Python scripts
- Visual n8n workflows
- Interactive chat interface
- Mobile Webex app
- Desktop Webex client

**Enterprise Features:**
- Team collaboration
- @mention support in groups
- Mobile-friendly responses
- Privacy-compliant processing
- No external dependencies

---

## Next Steps

### Immediate Next Steps

**1. Add More Documents**
```bash
cd ~/cisco-rag-demo

# Load additional documents
python3 load_document.py documents/assessment1.md
python3 load_document.py documents/assessment2.md
python3 load_document.py documents/policy1.md
```

**2. Share Bot with Team**
```
1. Give colleagues bot email: your-bot@webex.bot
2. They add bot in Webex
3. They can ask questions
4. Everyone benefits!
```

---

### Advanced Topics

**See accompanying guides for:**

**Customization:**
- Adjust chunk sizes
- Modify prompt templates
- Change response format
- Add custom instructions

**Multiple Collections:**
- Separate document sets
- Topic-based collections
- Access control per collection

**Enhanced Features:**
- Conversation history
- Rate limiting
- Usage analytics
- User authentication

**Production Deployment:**
- Static tunnel URLs (ngrok Pro)
- Cloud deployment (no tunnel)
- Monitoring and alerts
- Backup strategies


---

**🎉 CONGRATULATIONS on completing the trilogy! You've built something amazing!** 🎉
