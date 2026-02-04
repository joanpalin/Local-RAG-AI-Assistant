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
- 🌍 **Access anywhere** - As long as your Mac is on

**Time investment:** ~45 minutes to complete setup

---

### Before You Begin

**Verify prerequisites from Parts 1 & 2:**

```bash
# Quick system check
podman ps | grep -E "chromadb|n8n"  # Should show both running
curl http://localhost:11434/api/tags  # Should show models
ollama list  # Should show llama3.2:3b and nomic-embed-text

# Test RAG works - IMPORTANT: Navigate to scripts directory first!
cd ~/cisco-rag-demo/scripts
./query_rag.py "test query"  # Should return AI answer
```

**All working?** ✅ Continue!  
**Issues?** Return to Parts 1 & 2 to fix.

---

## Part 3A: What Webex Integration Adds

**Time:** 10 minutes (reading and understanding)

### Why Add Webex Integration?

**Your current setup:**
- ✅ Works great on your Mac
- ✅ Chat interface in n8n
- ✅ Python scripts for queries
- ❌ Only accessible when sitting at your Mac
- ❌ Not mobile-friendly
- ❌ Can't share with team easily

**After Webex integration:**
- ✅ Query from anywhere (via Webex mobile app)
- ✅ Share bot with team members
- ✅ Works in group spaces (with @mentions)
- ✅ Familiar interface (no training needed)
- ✅ Still 100% private (all processing on your Mac)

---

### How It Works - Simple Explanation

**The complete flow:**

```
[1] You type message in Webex Teams app
      ↓
[2] Message sent to Webex Cloud
      ↓
[3] Webex sends notification (webhook) to your Mac
      ↓
[4] Tunnel forwards through your firewall
      ↓
[5] n8n on your Mac receives notification
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
**Where processing happens:** On your Mac  
**Data privacy:** Never leaves your infrastructure

---

### Architecture Diagram

**Visual representation:**

```
┌───────────────────────────────────────────────────┐
│                  WEBEX CLOUD                      │
│  (Cisco's servers - handles message routing)      │
└─────────┬───────────────────────────────┬─────────┘
         │                                │
         │ [3] Webhook POST               │ [9] API POST
         │                                │
         ↓                                ↑
┌───────────────────────────────────────────────────┐
│               TUNNEL (ngrok)                      │
│  (Secure pathway through your firewall)           │
└─────────┬─────────────────────────────────────────┘
         │
         │ Forwards to localhost:5678
         │
         ↓
┌───────────────────────────────────────────────────┐
│                   YOUR MAC                        │
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │              n8n (port 5678)                │  │
│  │  Receives webhook, processes, responds      │  │
│  └───────┬─────────────────────────────────────┘  │
│          │                                        │
│          ├──→ [ChromaDB] Vector search            │
│          ├──→ [Ollama] AI processing              │
│          └──→ [Webex API] Send response           │
│                                                   │
│  ALL PROCESSING HAPPENS HERE (Private)            │
└───────────────────────────────────────────────────┘
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
- Required because your Mac is behind firewall

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
- Still fully private (processing on your Mac)
- Free tier is sufficient
- We'll walk through everything step-by-step
- Includes complete troubleshooting guide

---

### Success Criteria

**You'll know this part is complete when:**
- [ ] Webex bot created and token saved
- [ ] Tunnel running and accessible
- [ ] n8n workflow configured and published
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
- ❌ Can't initiate conversations (must be added)
- ❌ Different permissions than users

**Regular user account:**
- ✅ Full Webex features
- ✅ Can create spaces
- ❌ Requires human to login
- ❌ Can't be controlled by code

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
- Makes bot look professional
- Users see this in chat
```

**Description:** What your bot does
```
Example: AI-powered assistant that answers questions about Cisco network documentation using local RAG (Retrieval Augmented Generation). All processing happens on-premises for maximum privacy.
```

**Click "Add Bot"**

---

**Step 5: Save Your Bot Access Token**

**CRITICAL - DO THIS IMMEDIATELY!**

After creating the bot, you'll see a page with:
- Bot email address (e.g., rag-assistant@webex.bot)
- **Bot Access Token** (long string starting with "Bearer ")

**⚠️ IMPORTANT:** This token is only shown ONCE. You cannot retrieve it later!

**Copy and save the token securely:**

```bash
# Create credentials file
cd ~/cisco-rag-demo
nano bot-credentials.txt

# Paste this information:
Bot Name: [Your bot name]
Bot Email: [your-bot@webex.bot]
Bot Access Token: [paste the full token here]

# Save the file (Ctrl+O, Enter, Ctrl+X)

# Secure the file
chmod 600 bot-credentials.txt
```

**If you lose the token:** You'll need to regenerate it (which invalidates the old one).

---

**Step 6: Verify Bot Email**

**Your bot email format:**
```
[username]@webex.bot
```

**Examples:**
- rag-assistant@webex.bot
- cisco-docs-bot@webex.bot
- network-ai@webex.bot

**You'll need this email to:**
- Add bot to Webex spaces
- Test direct messages
- Share with team members

**Write it down somewhere easy to find!**

---

### Test Your Bot Account

**Quick verification that bot was created:**

**Option 1: Via Webex App**
```
1. Open Webex Teams app
2. Click "+" to start new conversation
3. Search for your bot email
4. If found → Bot created successfully!
5. Don't message yet (no backend connected)
```

**Option 2: Via API Test**
```bash
# Test that token works
curl -X GET https://webexapis.com/v1/people/me \
  -H "Authorization: Bearer YOUR_BOT_TOKEN_HERE"

# Should return JSON with bot details
# If error → token invalid or expired
```

**✅ Bot created and verified!** Let's set up the tunnel next.

---

## Part 3C: Setup Tunnel

**Time:** 10 minutes  
**What you'll do:** Create secure tunnel for webhooks  
**Why:** Webex needs to reach your Mac through firewall

---

### Understanding the Tunnel

**The problem:**
- Your Mac is behind a firewall
- Webex servers can't directly reach localhost:5678
- Port forwarding requires router access (often restricted)

**The solution:**
- Tunnel service creates public URL
- Routes traffic to your localhost
- No router configuration needed
- Works on company networks

**We'll use ngrok** (reliable, widely used, works on company-managed Macs)

---

### Install ngrok

**Why direct download:**
- ✅ Works without Homebrew (often not available on company Macs)
- ✅ No admin privileges required
- ✅ Simple and reliable
- ✅ Works on both Intel and Apple Silicon Macs

**Step 1: Check Your Mac's Processor**

```bash
# Determine which version you need
uname -m

# Output meanings:
# arm64 = Apple Silicon (M1/M2/M3/M4) → Use ARM version
# x86_64 = Intel processor → Use AMD64 version
```

---

**Step 2: Download ngrok**

**For Apple Silicon (M1/M2/M3/M4) Macs:**
```bash
cd ~/Downloads
curl -O https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-darwin-arm64.zip
unzip ngrok-v3-stable-darwin-arm64.zip
```

**For Intel Macs:**
```bash
cd ~/Downloads
curl -O https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-darwin-amd64.zip
unzip ngrok-v3-stable-darwin-amd64.zip
```

---

**Step 3: Install ngrok to Your PATH**

```bash
# Create personal bin directory
mkdir -p ~/bin

# Move ngrok to bin
mv ngrok ~/bin/

# Add to PATH (if not already there)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc

# Reload shell configuration
source ~/.zshrc

# Verify installation
ngrok version
```

**Expected output:** `ngrok version 3.x.x`

**✅ ngrok installed successfully!**

---

### Create ngrok Account

**Why you need an account:**
- ✅ Free tier is sufficient
- ✅ More stable connections than anonymous
- ✅ Gets you an auth token
- ✅ No credit card required

**Step 1: Sign Up**

Open browser and navigate to:
```
https://dashboard.ngrok.com/signup
```

**Sign up with:**
- Google account (easiest)
- GitHub account
- Or email/password

**Complete registration** (email verification if using email)

---

**Step 2: Get Your Auth Token**

After login, you'll see your dashboard.

**Click "Your Authtoken"** in the left sidebar

**Copy the token** (long string like: `2abc...xyz123`)

---

**Step 3: Configure ngrok**

```bash
# Add your auth token to ngrok
ngrok config add-authtoken YOUR_AUTHTOKEN_HERE

# Example:
# ngrok config add-authtoken 2abc...xyz123

# Verify configuration
cat ~/.config/ngrok/ngrok.yml
```

**Expected output:** Should show your authtoken in the config

**✅ ngrok configured!**

---

### Start the Tunnel

**IMPORTANT: Understanding Tunnel Behavior**

⚠️ **The tunnel MUST stay running for webhooks to work!**
- Do NOT close the terminal window
- If terminal closes, tunnel disconnects
- When tunnel restarts, you get a NEW URL
- You'll need to update the webhook URL in Webex

**Step 1: Start ngrok Tunnel**

```bash
# Start tunnel to n8n
ngrok http 5678
```

**You'll see output like:**

```
ngrok

Session Status: online
Account: your-email@example.com
Version: 3.x.x
Region: United States (us)
Latency: 25ms
Web Interface: http://127.0.0.1:4040
Forwarding: https://1a2b-3c4d-5e6f.ngrok-free.app -> http://localhost:5678

Connections       ttl     opn     rt1     rt5     p50     p90
                  0       0       0.00    0.00    0.00    0.00
```

**Key information:**
- **Forwarding URL:** `https://1a2b-3c4d-5e6f.ngrok-free.app`
- This is your public webhook URL
- **This URL changes every time you restart ngrok!**

---

**Step 2: Copy Your Tunnel URL**

**⚠️ CRITICAL:** Save this URL - you'll need it multiple times!

```bash
# Copy the HTTPS forwarding URL
# Example: https://1a2b-3c4d-5e6f.ngrok-free.app

# Save it to a file
echo "https://1a2b-3c4d-5e6f.ngrok-free.app" > ~/cisco-rag-demo/tunnel-url.txt
```

**✅ Tunnel is now running!**

---

### ⚠️ IMPORTANT: Keep This Terminal Open!

**CRITICAL TUNNEL MANAGEMENT:**

1. **DO NOT close this terminal window!**
   - Closing it stops the tunnel
   - Webhooks will stop working immediately
   - You'll need to restart and update webhook URL

2. **Open a NEW terminal for remaining commands:**
   - Press `Cmd+N` for new window
   - Or Terminal → New Window
   - Use the new terminal for all future steps

3. **Monitor tunnel status:**
   - This window shows live webhook requests
   - Useful for debugging
   - You'll see each message arrive

4. **If you need to restart ngrok:**
   - Note down when you do this
   - You'll get a NEW URL
   - Must update webhook in Webex (Part 3E)

**Keep this terminal visible but minimized so you can see tunnel is running!**

---

### Verify Tunnel Works

**In a NEW terminal window:**

```bash
# Test tunnel is accessible
curl https://YOUR-NGROK-URL.ngrok-free.app

# Replace YOUR-NGROK-URL with your actual URL
# Example: curl https://1a2b-3c4d-5e6f.ngrok-free.app
```

**Expected:** HTML response from n8n (login page)

**If connection refused:**
- Check n8n is running: `podman ps | grep n8n`
- Check tunnel is running (look at tunnel terminal)
- Verify URL is correct (no typos)

**✅ Tunnel verified and working!**

---

## Part 3D: n8n Webex Workflow

**Time:** 15 minutes  
**What you'll do:** Import and configure workflow  
**Why:** Connects all the pieces together

---

### Understanding the Workflow

**This workflow is the brain of your bot!**

**What it does:**
1. **Receives** webhook from Webex (new message notification)
2. **Fetches** full message content from Webex API
3. **Validates** message isn't from bot itself (prevents loops)
4. **Extracts** the question from the message
5. **Looks up** ChromaDB collection UUID
6. **Generates** embedding for the question using Ollama
7. **Searches** ChromaDB for relevant document chunks
8. **Builds** prompt with context for LLM
9. **Generates** AI answer using Ollama
10. **Sends** response back to Webex
11. **Acknowledges** webhook to Webex

**⏱️ Total execution time:** 5-10 seconds

---

### Import the Workflow

**Step 1: Download Workflow JSON**

The workflow JSON is provided in the project files. Save the following as `Webex_RAG_Bot.json`:

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
      "id": "098e2029-897b-44c4-bbd4-d31f31becdfd",
      "name": "Webex Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1.1,
      "position": [
        32,
        64
      ],
      "webhookId": "webex-bot-webhook",
      "notes": "Receives webhooks from Webex when messages are posted"
    },
    {
      "parameters": {
        "jsCode": "// Extract webhook data from correct location (body.data)\nconst webhookData = $input.first().json.body.data;\nconst webhookMeta = $input.first().json.body;\n\n// Validate we received the required fields\nif (!webhookData || !webhookData.id) {\n  throw new Error('Missing webhook data. Check that Webex webhook is configured correctly.');\n}\n\n// Extract key information\nconst messageId = webhookData.id;\nconst roomId = webhookData.roomId;\nconst personId = webhookData.personId;\nconst personEmail = webhookData.personEmail;\n\n// CRITICAL: Extract bot's person ID from webhook metadata\nconst botPersonId = webhookMeta.createdBy;\n\n// Validate all required fields exist\nif (!messageId || !roomId || !personEmail) {\n  throw new Error(`Missing required fields: messageId=${!!messageId}, roomId=${!!roomId}, personEmail=${!!personEmail}`);\n}\n\nconsole.log(`Processing message from ${personEmail} in room ${roomId}`);\nconsole.log(`Bot person ID: ${botPersonId}`);\n\n// Store for use in next nodes\nreturn [{\n  json: {\n    messageId: messageId,\n    roomId: roomId,\n    personId: personId,\n    personEmail: personEmail,\n    botPersonId: botPersonId,  // ← ADD THIS!\n    webhookReceived: true\n  }\n}];"
      },
      "id": "6ea2b827-c1ee-45ac-8037-379e44453dc6",
      "name": "Extract Webhook Data",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        192,
        64
      ],
      "notes": "Extracts message metadata from Webex webhook"
    },
    {
      "parameters": {
        "url": "=https://webexapis.com/v1/messages/{{ $json.messageId }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {
          "timeout": 10000
        }
      },
      "id": "fdad13af-ba8e-4f50-a38b-102be736bbe9",
      "name": "Get Message Details",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        368,
        64
      ],
      "credentials": {
        "httpHeaderAuth": {
          "id": "D0YLYycSvpB9tjy3",
          "name": "Header Auth account 2"
        }
      },
      "notes": "Fetches full message content from Webex API"
    },
    {
      "parameters": {
        "jsCode": "// Check if message is from bot itself and extract question\nconst messageData = $input.first().json;\nconst webhookData = $('Extract Webhook Data').first().json;\n\n// IMPORTANT: Replace this with your actual bot email from credentials file\nconst BOT_EMAIL = 'ragdemotest1@webex.bot';\n\nconsole.log(`Message from: ${messageData.personEmail}`);\nconsole.log(`Bot email: ${BOT_EMAIL}`);\n\n// CRITICAL: Check if message is from the bot itself\nif (messageData.personEmail === BOT_EMAIL) {\n  console.log('✓ Ignoring message from bot itself to prevent loop');\n  return [];\n}\n\n// Extract message text\nlet question = messageData.text;\n\nconsole.log(`Original message text: \"${question}\"`);\n\n// Remove @mentions (appears as <spark-mention> HTML in text)\nif (messageData.mentionedPeople && messageData.mentionedPeople.length > 0) {\n  // Remove HTML spark-mention tags\n  question = question.replace(/<spark-mention[^>]*>.*?<\\/spark-mention>/gi, '');\n  \n  // Also remove plain @mentions\n  question = question.replace(/@\\w+/g, '');\n  \n  // Remove any remaining HTML tags\n  question = question.replace(/<[^>]*>/g, '');\n}\n\n// Clean up whitespace\nquestion = question.trim();\n\n// Validate we have content\nif (!question || question.length === 0) {\n  throw new Error('Empty message received after cleaning');\n}\n\nconsole.log(`Cleaned question: \"${question}\"`);\n\nreturn [{\n  json: {\n    question: question,\n    roomId: webhookData.roomId,\n    messageId: webhookData.messageId,\n    personEmail: messageData.personEmail\n  }\n}];"
      },
      "id": "e32754ce-769d-4416-9c17-e69b6e2a7201",
      "name": "Validate & Extract Question",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        528,
        64
      ],
      "notes": "Filters bot's own messages and extracts clean question"
    },
    {
      "parameters": {
        "url": "http://host.docker.internal:8000/api/v1/collections",
        "options": {
          "timeout": 5000
        }
      },
      "id": "4320c7dc-2bea-4566-95ed-1d641153faa6",
      "name": "Get Collection UUID",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        704,
        64
      ],
      "notes": "Looks up ChromaDB collection UUID"
    },
    {
      "parameters": {
        "jsCode": "// Get UUID and prepare query\nconst collections = $input.all().map(item => item.json);\nconst question = $('Validate & Extract Question').first().json.question;\n\nlet collectionUuid = null;\nfor (const coll of collections) {\n  if (coll.name === 'cisco_docs') {\n    collectionUuid = coll.id;\n    break;\n  }\n}\n\nif (!collectionUuid) {\n  throw new Error(\"Collection 'cisco_docs' not found! Load documents first.\");\n}\n\nreturn [{\n  json: {\n    collection_uuid: collectionUuid,\n    question: question,\n    embedding_request: {\n      model: 'nomic-embed-text',\n      prompt: question\n    }\n  }\n}];"
      },
      "id": "29a55788-458e-4311-ae5d-f10e4d53e7fb",
      "name": "Prepare Query",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        848,
        64
      ],
      "notes": "Prepares embedding request for question"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "http://host.containers.internal:11434/api/embeddings",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json.embedding_request) }}",
        "options": {
          "timeout": 10000
        }
      },
      "id": "fb306b9f-c90d-4725-9886-e55de1efb687",
      "name": "Generate Question Embedding",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        1008,
        64
      ],
      "notes": "Generates embedding using Ollama"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=http://host.docker.internal:8000/api/v1/collections/{{ $('Prepare Query').first().json.collection_uuid }}/query",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify({\n  query_embeddings: [$json.embedding],\n  n_results: 3\n}) }}",
        "options": {
          "timeout": 5000
        }
      },
      "id": "7434c7b9-41a2-4459-99fd-be88261ece41",
      "name": "Search ChromaDB",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        1168,
        64
      ],
      "notes": "Searches for relevant document chunks"
    },
    {
      "parameters": {
        "jsCode": "// Build LLM prompt with context\nconst searchResults = $input.first().json;\nconst question = $('Prepare Query').first().json.question;\n\nconst contexts = searchResults.documents[0];\n\nif (!contexts || contexts.length === 0) {\n  throw new Error('No documents found in ChromaDB. Load documents first.');\n}\n\nconst contextText = contexts.map((ctx, i) => \n  `[Source ${i + 1}]\\n${ctx}`\n).join('\\n\\n---\\n\\n');\n\nconst systemPrompt = `You are a helpful AI assistant analyzing Cisco network assessment documents.\nProvide accurate, concise answers suitable for messaging.\nKeep responses under 500 words.\nIf context doesn't contain enough information, say so clearly.`;\n\nconst prompt = `${systemPrompt}\\n\\nCONTEXT FROM DOCUMENTS:\\n${contextText}\\n\\nQUESTION: ${question}\\n\\nAnswer (be specific and concise for messaging):`;\n\nconsole.log(`Retrieved ${contexts.length} chunks for question`);\n\nreturn [{\n  json: {\n    llm_request: {\n      model: 'llama3.2:3b',\n      prompt: prompt,\n      stream: false,\n      options: {\n        temperature: 0.7,\n        top_p: 0.9,\n        num_predict: 500\n      }\n    }\n  }\n}];"
      },
      "id": "ef6be37e-be10-4721-91f6-36edc08cd842",
      "name": "Build Prompt",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1328,
        64
      ],
      "notes": "Creates prompt with context for LLM"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "http://host.containers.internal:11434/api/generate",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json.llm_request) }}",
        "options": {
          "timeout": 30000
        }
      },
      "id": "410f6380-4191-460b-b284-366b98a465f6",
      "name": "Generate Answer",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        1472,
        64
      ],
      "notes": "Generates AI answer using Ollama"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://webexapis.com/v1/messages",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify({\n  roomId: $('Validate & Extract Question').first().json.roomId,\n  text: $json.response.trim()\n}) }}",
        "options": {
          "timeout": 10000
        }
      },
      "id": "42ae94c1-249c-4e2e-a83a-3814c8689aaf",
      "name": "Send to Webex",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [
        1616,
        64
      ],
      "credentials": {
        "httpHeaderAuth": {
          "id": "D0YLYycSvpB9tjy3",
          "name": "Header Auth account 2"
        }
      },
      "notes": "Sends AI response back to Webex room"
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ JSON.stringify({ success: true, message: 'Response sent to Webex' }) }}",
        "options": {}
      },
      "id": "2e658f26-397b-46e4-8b07-2ad09853b637",
      "name": "Webhook Response",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [
        1760,
        64
      ],
      "notes": "Acknowledges webhook to Webex"
    },
    {
      "parameters": {
        "content": "##                       Receive Message and Processing",
        "height": 224,
        "width": 656,
        "color": 3
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        0,
        0
      ],
      "typeVersion": 1,
      "id": "4bd05af4-7709-49e1-9cea-ee4f91601c25",
      "name": "Sticky Note"
    },
    {
      "parameters": {
        "content": "## Database Search",
        "height": 224,
        "width": 624,
        "color": 2
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        672,
        0
      ],
      "typeVersion": 1,
      "id": "8cc6b713-963b-4460-9d79-a7b7b9a9f2c4",
      "name": "Sticky Note1"
    },
    {
      "parameters": {
        "content": "## Generate answer",
        "height": 224,
        "width": 624,
        "color": 4
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        1312,
        0
      ],
      "typeVersion": 1,
      "id": "4a3d9108-13df-4104-85f2-e2c430139479",
      "name": "Sticky Note2"
    }
  ],
  "pinData": {},
  "connections": {
    "Webex Webhook": {
      "main": [
        [
          {
            "node": "Extract Webhook Data",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Extract Webhook Data": {
      "main": [
        [
          {
            "node": "Get Message Details",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Message Details": {
      "main": [
        [
          {
            "node": "Validate & Extract Question",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Validate & Extract Question": {
      "main": [
        [
          {
            "node": "Get Collection UUID",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Collection UUID": {
      "main": [
        [
          {
            "node": "Prepare Query",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Prepare Query": {
      "main": [
        [
          {
            "node": "Generate Question Embedding",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Generate Question Embedding": {
      "main": [
        [
          {
            "node": "Search ChromaDB",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Search ChromaDB": {
      "main": [
        [
          {
            "node": "Build Prompt",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Build Prompt": {
      "main": [
        [
          {
            "node": "Generate Answer",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Generate Answer": {
      "main": [
        [
          {
            "node": "Send to Webex",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Send to Webex": {
      "main": [
        [
          {
            "node": "Webhook Response",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1",
    "availableInMCP": false
  },
  "versionId": "3169feaf-c142-4dfd-81fc-df5dcc05be3d",
  "meta": {
    "instanceId": "e01397b3da8cc95dd77600e45d38feaf0925f5a60febb9aaa5856cd3706790a9"
  },
  "id": "90lJ7Jo_zdejn5uohmljk",
  "tags": []
}
EOF
```

---

**Step 2: Import into n8n**

```bash
# Open n8n in your browser
open http://localhost:5678

# Or navigate manually to: http://localhost:5678
```

**In n8n interface:**

1. Click **"Workflows"** in left sidebar
2. Click **"+"** button (top right) → **"Import from File"**
3. Select the `Webex_RAG_Bot.json` file
4. Workflow will open automatically

**✅ Workflow imported!**

---

### Configure Webex Bot Credential

**This is CRITICAL - the workflow needs your bot token!**

**Step 1: Create Webex Bot Token Credential**

1. In the workflow, click on **"Get Message Details"** node
2. Under **"Authentication"**, click the credential dropdown
3. Click **"+ Create New Credential"**
4. Select **"Header Auth"**

**Step 2: Configure the Credential**

**CRITICAL - Use these EXACT values:**

```
Credential Name: Webex Bot Token
Header Name: Authorization
Header Value: Bearer YOUR_BOT_ACCESS_TOKEN
```

**⚠️ COMMON MISTAKES TO AVOID:**
- ❌ Header Name = "Webex Bot Token" → WRONG! Will fail!
- ✅ Header Name = "Authorization" → CORRECT!
- ❌ Header Value = "YOUR_BOT_ACCESS_TOKEN" → WRONG! Missing "Bearer "
- ✅ Header Value = "Bearer YOUR_BOT_ACCESS_TOKEN" → CORRECT!

**Where to get YOUR_BOT_ACCESS_TOKEN:**
```bash
# Look in your saved credentials file
cat ~/cisco-rag-demo/bot-credentials.txt
```

**Example correct configuration:**
```
Header Name: Authorization
Header Value: Bearer ZTI1ZDgzMzctZWVjMy00OTlkLWFkNjctZjY2ZTc3ODJkNjMyNmU1YzgxMzctNDFl_PF84_1eb65fdf-9643-417f-9974-ad72cae0e10f
```

**Click "Save"**

---

**Step 3: Apply Credential to Both Nodes**

The credential is needed in TWO nodes:
1. **Get Message Details** (already done ✅)
2. **Send to Webex**

**For "Send to Webex" node:**
1. Click the **"Send to Webex"** node
2. Under **"Authentication"** dropdown
3. Select your **"Webex Bot Token"** credential
4. Click away to save

**✅ Credential configured for both nodes!**

---

### Update Bot Email in Code

**CRITICAL: Prevent infinite loops!**

**Step 1: Open "Validate & Extract Question" Node**

1. Click the **"Validate & Extract Question"** node
2. You'll see JavaScript code
3. Find this line near the top:
   ```javascript
   const BOT_EMAIL = 'YOUR-BOT-EMAIL@webex.bot';
   ```

**Step 2: Replace with Your Bot Email**

```javascript
// Change from:
const BOT_EMAIL = 'YOUR-BOT-EMAIL@webex.bot';

// To (example):
const BOT_EMAIL = 'rag-assistant@webex.bot';
```

**Use YOUR actual bot email from Part 3B!**

**Why this is critical:**
- Without this, bot responds to its own messages
- Creates infinite loop
- Burns through API calls
- Drains your resources

**Click away to save the change**

**✅ Bot email configured!**

---

### Understanding Host References (Informational)

**You'll notice different URLs in the workflow nodes:**

**For Ollama (runs natively on macOS):**
- Generate Question Embedding: `http://host.containers.internal:11434/api/embeddings`
- Generate Answer: `http://host.containers.internal:11434/api/generate`

**For ChromaDB (runs in Podman container):**
- Get Collection UUID: `http://host.docker.internal:8000/api/v1/collections`
- Search ChromaDB: `http://host.docker.internal:8000/api/v1/collections/{uuid}/query`

**Why different hosts?**
- Ollama runs directly on your Mac → accessible via `host.containers.internal`
- ChromaDB runs in a container → accessible via `host.docker.internal`
- This is correct and intentional!
- Do NOT change these URLs unless you know what you're doing

**ℹ️ This is for your understanding - no changes needed!**

---

### Publish the Workflow

**⚠️ IMPORTANT: n8n UI has changed!**

**Old n8n versions had:**
- "Save" button
- "Activate" toggle

**New n8n (v2.4.6+) has:**
- **"Publish"** button only
- Publishing both saves AND activates

**To publish your workflow:**

1. **Click the "Publish" button** (top right, orange/red button)
2. Workflow is now saved AND active
3. Webhook is now listening for requests

**✅ Workflow is published and active!**

**Note:** You can edit published workflows anytime by clicking on the workflow name.

---

### Verify Workflow is Active

**Check workflow status:**

```bash
# In n8n web interface
# Look at the workflow list (click "Workflows" in sidebar)
# Your "Webex RAG Bot" should show status indicator
```

**Status indicators:**
- 🟢 Green dot = Active and running
- ⚪ Gray dot = Inactive
- 🔴 Red dot = Error

**Should be 🟢 Green!**

---

### Get Your Webhook URL

**CRITICAL: You need this for registering with Webex!**

**Step 1: Open the Webhook Node**

1. Click the **"Webex Webhook"** node (first node in workflow)
2. Look at the **"Webhook URLs"** section
3. You'll see **"Test URL"** and **"Production URL"**

**Step 2: Use the Production URL**

**Format:**
```
https://YOUR-NGROK-URL.ngrok-free.app/webhook/webex-webhook
```

**Example:**
```
https://1a2b-3c4d-5e6f.ngrok-free.app/webhook/webex-webhook
```

**⚠️ Note the path:** `/webhook/webex-webhook` is added to your ngrok URL

**Copy this complete URL** - you'll need it in Part 3E!

**✅ Webhook URL ready!**

---

## Part 3E: Register Webhook

**Time:** 10 minutes  
**What you'll do:** Tell Webex where to send notifications  
**Why:** Connects Webex to your n8n workflow

---

### Understanding Webhook Registration

**What you're doing:**
- Telling Webex: "When messages arrive, notify this URL"
- Registering your bot to receive message events
- Creating persistent connection between Webex and your Mac

**What happens after:**
- Someone messages your bot → Webex sends POST to your webhook
- Your n8n workflow receives notification
- Workflow processes and responds

**One-time setup:** Unless you change ngrok URL

---

### Register Webhook via API

**Method 1: Using curl (Recommended)**

**Step 1: Prepare the Command**

```bash
# Set variables
BOT_TOKEN="YOUR_BOT_ACCESS_TOKEN_HERE"
WEBHOOK_URL="https://YOUR-NGROK-URL.ngrok-free.app/webhook/webex-webhook"

# Register webhook
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer ${BOT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "RAG Bot Message Webhook",
    "targetUrl": "'"${WEBHOOK_URL}"'",
    "resource": "messages",
    "event": "created"
  }'
```

**Step 2: Replace Placeholders**

**Replace `YOUR_BOT_ACCESS_TOKEN_HERE` with your bot token:**
```bash
# Get from your credentials file
cat ~/cisco-rag-demo/bot-credentials.txt
```

**Replace `YOUR-NGROK-URL` with your ngrok URL:**
```bash
# Get from tunnel terminal or:
cat ~/cisco-rag-demo/tunnel-url.txt
```

**Step 3: Execute Command**

```bash
# Run the complete curl command with your values filled in
```

**Expected response:**
```json
{
  "id": "Y2lzY29zcGFy...",
  "name": "RAG Bot Message Webhook",
  "targetUrl": "https://YOUR-NGROK-URL.ngrok-free.app/webhook/webex-webhook",
  "resource": "messages",
  "event": "created",
  "orgId": "...",
  "createdBy": "...",
  "appId": "...",
  "ownedBy": "creator",
  "status": "active",
  "created": "2024-01-15T10:30:00.000Z"
}
```

**Save the webhook ID** (the "id" field) - you'll need it to delete/update later

**✅ Webhook registered!**

---

### Method 2: Using Webex Developer Portal

**Alternative if curl doesn't work:**

1. Go to https://developer.webex.com/docs/api/v1/webhooks/create-a-webhook
2. **Sign in** with your Webex account
3. **Use the interactive API tool** on right side
4. **Fill in:**
   - name: "RAG Bot Message Webhook"
   - targetUrl: Your complete webhook URL
   - resource: "messages"
   - event: "created"
5. **Click "Run"**
6. **Check response** shows webhook created

**✅ Webhook registered!**

---

### Verify Webhook Registration

**List all webhooks:**

```bash
# List registered webhooks
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN_HERE"
```

**Expected:** JSON array showing your webhook(s)

**Check:**
- ✅ targetUrl matches your ngrok URL + path
- ✅ resource = "messages"
- ✅ event = "created"
- ✅ status = "active"

**✅ Webhook verified!**

---

### When to Update Webhook

**You need to re-register webhook when:**
- ❌ Ngrok tunnel restarts (new URL)
- ❌ Changed n8n workflow webhook path
- ❌ Switched to different machine

**You DON'T need to re-register when:**
- ✅ Workflow code changes (same webhook path)
- ✅ Bot token is the same
- ✅ n8n restarts (same URL)

---

### Delete Old Webhooks (If Needed)

**If you need to clean up:**

```bash
# Delete specific webhook
curl -X DELETE https://webexapis.com/v1/webhooks/WEBHOOK_ID_HERE \
  -H "Authorization: Bearer YOUR_BOT_TOKEN_HERE"

# Or delete ALL webhooks for your bot
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN_HERE" | \
  grep -o '"id":"[^"]*"' | \
  cut -d'"' -f4 | \
  while read id; do
    curl -X DELETE https://webexapis.com/v1/webhooks/$id \
      -H "Authorization: Bearer YOUR_BOT_TOKEN_HERE"
  done
```

**✅ Webhook management complete!**

---

## Part 3F: Test Your Bot

**Time:** 5-10 minutes  
**What you'll do:** Send messages and verify responses  
**Why:** Confirm end-to-end integration works

---

### Pre-Flight Check

**Before testing, verify ALL components:**

```bash
# Quick health check
cd ~/cisco-rag-demo

# 1. Check containers running
podman ps | grep -E "chromadb|n8n"
# Expected: Both containers showing "Up"

# 2. Check Ollama
ollama list
# Expected: llama3.2:3b and nomic-embed-text

# 3. Check documents loaded
curl -s http://localhost:8000/api/v1/collections | grep cisco_docs
# Expected: Should show cisco_docs collection

# 4. Check tunnel running
# Look at tunnel terminal - should show "online"

# 5. Check n8n workflow
# Open http://localhost:5678
# Workflow should show green "active" indicator
```

**All checks pass?** ✅ Ready to test!

---

### Test 1: Direct Message to Bot

**Most basic test - direct 1:1 message**

**Step 1: Add Bot in Webex**

Open Webex Teams app (desktop or mobile)

**Click "+" to start new conversation**

**Search for your bot email:**
```
your-bot-name@webex.bot
```

**Click on bot to start chat**

---

**Step 2: Send Test Message**

**Type a simple question:**
```
Hello! Can you help me?
```

**Press Enter/Send**

---

**Step 3: Watch for Response**

**What should happen:**

1. **Immediately** (~1 second):
   - Message sent to Webex
   - Webex sends webhook to your n8n
   - Check ngrok terminal - should show incoming request

2. **Processing** (~3-5 seconds):
   - n8n workflow executes
   - Fetches message from Webex
   - Generates embedding
   - Searches ChromaDB
   - Generates AI response

3. **Response** (~5-8 seconds total):
   - Bot sends reply in Webex
   - You see answer in chat

**Expected response:**
```
Based on the Cisco network documentation, I can help you with questions about network assessments, configurations, and recommendations. What would you like to know?
```

**✅ If you got a response:** SUCCESS! Bot is working!

**❌ If no response:** Check troubleshooting section below

---

### Test 2: Ask Document Question

**Test RAG system integration**

**Send this message:**
```
What are the main recommendations in the network assessment?
```

**Expected:**
- Bot retrieves relevant chunks from your documents
- Provides specific answer based on document content
- Response should cite information from documents

**Example response:**
```
Based on the network assessment documents, the main recommendations include:

1. Upgrade aging network infrastructure
2. Implement network segmentation for security
3. Deploy SD-WAN for branch offices
4. Standardize on Cisco Catalyst 9000 series
5. Implement network automation with DNA Center

These recommendations are from the 2024 network assessment report.
```

**✅ If answer references your documents:** RAG is working!

---

### Test 3: Group Space with @mentions

**Test team collaboration features**

**Step 1: Create Test Space**

1. In Webex, click "+" → "Create a Space"
2. Name it "RAG Bot Test"
3. Add your bot: Search for bot email and add
4. Add a colleague (optional)

---

**Step 2: Test @mention**

**In the space, type:**
```
@RAG-Assistant What is in the documentation?
```

**Replace @RAG-Assistant with your bot's actual name**

**Expected:**
- Bot only responds when @mentioned
- Doesn't respond to messages not directed at it
- Multiple people can ask questions

**✅ If bot responds only when @mentioned:** Group mode working!

---

### Test 4: Mobile Access

**Test from phone/tablet**

**Step 1: Install Webex App**

- iOS: https://apps.apple.com/app/cisco-webex/id833967564
- Android: https://play.google.com/store/apps/details?id=com.cisco.wx2.android

**Step 2: Login**

Use same Webex account

---

**Step 3: Find Your Bot**

- Search for bot email
- Open direct message
- Send test question

**Expected:**
- Same fast response
- Works on cellular or WiFi
- Can use voice-to-text for questions

**✅ If mobile works:** Full mobile access confirmed!

---

### Test 5: Response Time

**Measure performance**

**Send this:**
```
Quick test question about the network
```

**Time from send to response:**

**Target:** < 10 seconds  
**Good:** 5-8 seconds  
**Excellent:** 3-5 seconds

**If > 15 seconds:**
- Check Mac CPU (Activity Monitor)
- Verify Ollama model loaded: `ollama list`
- Check network latency to Webex

**✅ Performance acceptable!**

---

### Monitoring Webhook Activity

**Watch the tunnel terminal:**

Every webhook request shows up as a log entry:
```
POST /webhook/webex-webhook  200 OK
```

**You can see:**
- Each incoming message notification
- Response status codes
- Request timing

**Useful for debugging!**

---

### What to Do If Tests Fail

**See Part 3H: Troubleshooting** for detailed solutions

**Quick checks:**

**No response at all:**
```bash
# Check webhook registered
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Check n8n workflow active
# Visit: http://localhost:5678

# Check tunnel running
# Look at ngrok terminal
```

**Error response:**
```bash
# Check n8n execution log
# In n8n web interface: Executions → Click failed execution

# Check which node failed
# Red X indicates problem node
```

**Slow response:**
```bash
# Check Ollama loaded
ollama list

# Check Mac resources
top

# Check document count
curl http://localhost:8000/api/v1/collections
```

**✅ All tests passing?** Congratulations! Bot is fully operational!

---

## Part 3G: Daily Operations

**Time:** 5 minutes  
**What you'll learn:** Day-to-day bot management  
**Why:** Keep bot running reliably

---

### Daily Startup Procedure

**When you start your Mac each day:**

**Step 1: Start Core Services** (if not auto-starting)

```bash
# Check what's running
podman ps

# If ChromaDB not running:
podman start chromadb

# If n8n not running:
podman start n8n

# Verify both running
podman ps | grep -E "chromadb|n8n"
```

---

**Step 2: Start Tunnel**

```bash
# Start ngrok tunnel
ngrok http 5678
```

**⚠️ CRITICAL:** Note the new ngrok URL!

**If URL changed from yesterday:**
- You MUST update webhook (Part 3E)
- Old URL won't work
- Messages won't reach your bot

---

**Step 3: Update Webhook (If URL Changed)**

```bash
# Delete old webhook
curl -X DELETE https://webexapis.com/v1/webhooks/OLD_WEBHOOK_ID \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Register new webhook with new URL
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "RAG Bot Message Webhook",
    "targetUrl": "https://NEW-NGROK-URL.ngrok-free.app/webhook/webex-webhook",
    "resource": "messages",
    "event": "created"
  }'
```

---

**Step 4: Quick Test**

```bash
# Send yourself a test message in Webex
# Verify bot responds
```

**✅ Daily startup complete!** (3-5 minutes)

---

### Helper Script for Startup

**Save this as `~/cisco-rag-demo/start-system.sh`:**

```bash
#!/bin/bash

echo "🚀 Starting Local RAG System..."

# Start containers
echo "📦 Starting containers..."
podman start chromadb
podman start n8n
sleep 5

# Check status
echo "✅ Checking container status..."
podman ps | grep -E "chromadb|n8n"

# Check Ollama
echo "🤖 Checking Ollama models..."
ollama list

echo ""
echo "🎯 Next steps:"
echo "1. Start ngrok tunnel: ngrok http 5678"
echo "2. Note the new URL"
echo "3. If URL changed, update webhook (see docs)"
echo "4. Send test message to bot"
echo ""
echo "✨ System ready!"
```

**Make executable:**
```bash
chmod +x ~/cisco-rag-demo/start-system.sh
```

**Run daily:**
```bash
cd ~/cisco-rag-demo
./start-system.sh
```

---

### Monitor Bot Health

**Quick health check command:**

```bash
#!/bin/bash
# Save as: ~/cisco-rag-demo/check-health.sh

echo "🏥 RAG System Health Check"
echo "=========================="

# Check containers
echo ""
echo "📦 Container Status:"
podman ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E "NAME|chromadb|n8n"

# Check Ollama
echo ""
echo "🤖 Ollama Status:"
curl -s http://localhost:11434/api/tags | grep -q "llama3.2" && echo "✅ Ollama responding" || echo "❌ Ollama not responding"

# Check ChromaDB
echo ""
echo "💾 ChromaDB Status:"
curl -s http://localhost:8000/api/v1/heartbeat | grep -q "200" && echo "✅ ChromaDB responding" || echo "❌ ChromaDB not responding"

# Check n8n
echo ""
echo "⚙️  n8n Status:"
curl -s http://localhost:5678 | grep -q "n8n" && echo "✅ n8n responding" || echo "❌ n8n not responding"

# Check documents
echo ""
echo "📄 Documents Loaded:"
curl -s http://localhost:8000/api/v1/collections | grep -o '"name":"[^"]*"' | wc -l | xargs echo "Collections:"

echo ""
echo "=========================="
```

**Make executable and run:**
```bash
chmod +x ~/cisco-rag-demo/check-health.sh
./check-health.sh
```

---

### Handling Tunnel URL Changes

**Ngrok free tier changes URL every restart:**

**Option 1: Update Webhook Each Time** (Free)
- Restart ngrok → Get new URL
- Update webhook registration
- Takes 2-3 minutes

**Option 2: Paid ngrok Plan** (~$8/month)
- Get persistent URL (doesn't change)
- No webhook updates needed
- Worth it for daily use

**Option 3: localhost.run Alternative**
- ⚠️ Note: localhost.run was removed from this guide due to compatibility issues with company-managed Macs
- If you want stable URLs, upgrade to ngrok paid plan

---

### Stopping the System

**When you're done for the day:**

```bash
# Stop tunnel (Ctrl+C in ngrok terminal)

# Optionally stop containers to save resources
podman stop chromadb
podman stop n8n

# Or leave running for instant startup next time
```

**Data persists!**
- Documents stay in ChromaDB
- Workflows saved in n8n
- No data loss

---

### Adding New Documents

**While system is running:**

```bash
cd ~/cisco-rag-demo

# Upload via Python script
python3 load_document.py path/to/new-document.md

# Or use n8n upload workflow
# Open: http://localhost:5678
# Use document upload form
```

**Documents immediately searchable!**

---

### Sharing Bot with Team

**Give colleagues:**
```
Bot email: your-bot-name@webex.bot
```

**They can:**
1. Search for bot in Webex
2. Add bot to spaces
3. Ask questions immediately

**No configuration needed on their end!**

---

## Part 3H: Troubleshooting Webex Integration

**Common issues and solutions**

---

### Issue: Bot Doesn't Respond

**Symptom:** Message sent, no response from bot

**Check 1: Is webhook registered?**
```bash
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Should return webhook with your ngrok URL
```

**Fix if not registered:**
```bash
# Re-register webhook (Part 3E)
```

---

**Check 2: Is tunnel running?**
```bash
# Look at ngrok terminal
# Should show "Session Status: online"
```

**Fix if not running:**
```bash
# Restart tunnel
ngrok http 5678
# Update webhook with new URL
```

---

**Check 3: Is n8n workflow active?**
```bash
# Open http://localhost:5678
# Check workflow shows green "active" indicator
```

**Fix if not active:**
```bash
# Click workflow → Click "Publish" button
```

---

**Check 4: Are containers running?**
```bash
podman ps | grep -E "chromadb|n8n"
```

**Fix if not running:**
```bash
podman start chromadb
podman start n8n
```

---

### Issue: Bot Responds to Own Messages (Loop)

**Symptom:** Bot keeps responding to itself repeatedly

**Root cause:** Bot email not configured in workflow

**Fix:**

1. Open n8n: `http://localhost:5678`
2. Open "Webex RAG Bot" workflow
3. Click "Validate & Extract Question" node
4. Find line: `const BOT_EMAIL = '...'`
5. Change to YOUR bot email
6. Click "Publish"

**Verify fix:**
```javascript
// Should match your bot from developer.webex.com
const BOT_EMAIL = 'your-actual-bot@webex.bot';
```

---

### Issue: Credential Errors

**Symptom:** "invalid HTTP token" or "unauthorized" errors

**Common mistake:** Header Name set incorrectly

**Check credential:**

1. Open n8n workflow
2. Click "Get Message Details" node
3. Click credential dropdown → Edit
4. Verify:
   - Header Name = `Authorization` (NOT "Webex Bot Token")
   - Header Value = `Bearer YOUR_TOKEN` (must start with "Bearer ")

**Correct format:**
```
Header Name: Authorization
Header Value: Bearer ZTI1ZDgzMzctZWVjMy00OTlkLWFkNjctZjY2ZTc...
```

**Apply same credential to "Send to Webex" node!**

---

### Issue: Slow Responses (>15 seconds)

**Symptom:** Bot responds but takes too long

**Check 1: Ollama models loaded**
```bash
ollama list
# Should show llama3.2:3b and nomic-embed-text
```

**Fix:**
```bash
# Pre-load models
ollama pull llama3.2:3b
ollama pull nomic-embed-text
```

---

**Check 2: Mac resources**
```bash
# Check CPU usage
top
# If > 90%, close other apps
```

---

**Check 3: Document count**
```bash
# Too many documents can slow queries
curl http://localhost:8000/api/v1/collections
# If > 100 docs, consider splitting collections
```

---

### Issue: "Collection not found" Error

**Symptom:** Workflow fails at "Search ChromaDB" node

**Root cause:** No documents loaded

**Fix:**
```bash
cd ~/cisco-rag-demo

# Load a test document
echo "This is a test document about networking." > test.md
python3 load_document.py test.md

# Verify
curl http://localhost:8000/api/v1/collections | grep cisco_docs
```

---

### Issue: Tunnel Keeps Disconnecting

**Symptom:** ngrok connection drops frequently

**Common causes:**
- Unstable internet connection
- Free tier timeout
- Firewall interference

**Solutions:**

**Option 1: Upgrade to paid ngrok**
- More stable connections
- Persistent URLs
- Worth it for production use

**Option 2: Monitor and restart**
```bash
# Keep tunnel terminal visible
# Restart when drops
# Update webhook with new URL
```

**Option 3: Add monitoring**
```bash
# Create restart script
#!/bin/bash
while true; do
  ngrok http 5678
  echo "Tunnel disconnected, restarting..."
  sleep 5
done
```

---

### Issue: Webhook URL Mismatch

**Symptom:** Webhook registered but no messages received

**Check webhook URL exactly matches:**

```bash
# List webhooks
curl -X GET https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Compare with n8n webhook URL
# Must match: https://YOUR-URL.ngrok-free.app/webhook/webex-webhook
```

**Fix if mismatch:**
```bash
# Delete old webhook
curl -X DELETE https://webexapis.com/v1/webhooks/WEBHOOK_ID \
  -H "Authorization: Bearer YOUR_BOT_TOKEN"

# Register with correct URL
# (See Part 3E)
```

---

### Issue: Host Connection Errors

**Symptom:** "Connection refused" or "service offline" in workflow

**Check which node fails:**
- Get Collection UUID → ChromaDB issue
- Generate Embedding → Ollama issue
- Search ChromaDB → ChromaDB issue
- Generate Answer → Ollama issue

**Verify URLs in failing node:**

**For ChromaDB nodes** (should use):
```
http://host.docker.internal:8000/...
```

**For Ollama nodes** (should use):
```
http://host.containers.internal:11434/...
```

**Don't mix these up!**

---

### Debug Mode

**Enable verbose logging:**

1. Open n8n workflow
2. Click Settings (gear icon)
3. Enable "Save Execution Progress"
4. Click "Publish"

**Now you can:**
- See data flowing between nodes
- Identify exact failure point
- View actual API responses

**Check execution details:**
1. In n8n, click "Executions" (left sidebar)
2. Click failed execution
3. Click nodes with red X
4. View error messages

---

### Getting Help

**Before asking for help, collect:**

1. **Error messages** (exact text)
2. **What you were doing** (steps taken)
3. **Health check output** (run check-health.sh)
4. **n8n execution details** (screenshot of failed node)
5. **Webhook status** (list webhooks output)

**Check:**
- This troubleshooting guide
- GUIDE_1 and GUIDE_2 troubleshooting
- Comprehensive troubleshooting chronicle in project docs

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
cd ~/cisco-rag-demo/scripts
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
- [ ] n8n workflow published and active
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
- All processing on your Mac
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

**3. Prepare Demo Questions**
```
Create 10-15 compelling questions
Know expected answers
Practice demo flow
Time responses
Prepare backup plans
```

---

### For Cisco Live 2026

**You're now ready to:**
- ✅ Demonstrate local RAG system
- ✅ Show mobile access
- ✅ Explain privacy benefits
- ✅ Compare to cloud solutions
- ✅ Answer technical questions
- ✅ Deliver reliable live demo

**Preparation tips:**
1. Practice demo 10+ times
2. Prepare compelling documents
3. Create visually interesting questions
4. Have backup Mac ready
5. Pre-load models before presentation
6. Know troubleshooting procedures

---

## Document Information

**Version:** 2.0.0 (UPDATED)  
**Last Updated:** January 31, 2026  
**Part of:** Local RAG System Implementation Guides  

**Major Changes in v2.0:**
- ✅ Removed localhost.run (compatibility issues)
- ✅ Updated ngrok installation (direct download method)
- ✅ Fixed n8n UI instructions (Publish button)
- ✅ Added Webex credential setup details
- ✅ Updated workflow JSON (working, tested version)
- ✅ Added tunnel terminal management
- ✅ Clarified host references (Ollama vs ChromaDB)
- ✅ Fixed path issues in verification commands

**Previous Guides:**
- [GUIDE_1_ENVIRONMENT_SETUP.md](GUIDE_1_ENVIRONMENT_SETUP.md)
- [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)

---

**🎉 CONGRATULATIONS on completing the trilogy! You've built something amazing!** 🎉
