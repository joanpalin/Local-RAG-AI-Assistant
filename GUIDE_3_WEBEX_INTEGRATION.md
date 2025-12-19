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
┌─────────────────────────────────────────────────────┐
│                  WEBEX CLOUD                        │
│  (Cisco's servers - handles message routing)       │
└────────┬────────────────────────────────┬──────────┘
         │                                │
         │ [3] Webhook POST               │ [9] API POST
         │                                │
         ↓                                ↑
┌────────────────────────────────────────────────────┐
│            TUNNEL (localhost.run/ngrok)            │
│  (Secure pathway through your firewall)            │
└────────┬───────────────────────────────────────────┘
         │
         │ Forwards to localhost:5678
         │
         ↓
┌─────────────────────────────────────────────────────┐
│                   YOUR MAC                          │
│                                                     │
│  ┌───────────────────────────────────────────┐    │
│  │              n8n (port 5678)              │    │
│  │  Receives webhook, processes, responds    │    │
│  └───────┬───────────────────────────────────┘    │
│          │                                         │
│          ├──→ [ChromaDB] Vector search            │
│          ├──→ [Ollama] AI processing              │
│          └──→ [Webex API] Send response           │
│                                                     │
│  ALL PROCESSING HAPPENS HERE (Private)             │
└─────────────────────────────────────────────────────┘
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
- Represents your bot in chats
- Can skip for now
```

**Description:** What the bot does
```
Example: "AI-powered assistant for querying Cisco network 
         assessment documents. Provides instant answers 
         based on your specific documentation."
         
Tip: Users see this when adding bot to spaces
```

**Recommended entries:**
```
Bot Name: RAG Assistant
Bot Username: rag-assistant
Description: AI assistant for Cisco network assessments
```

**Click "Add Bot"**

---

**Step 5: CRITICAL - Save Bot Access Token**

⚠️ **THIS IS EXTREMELY IMPORTANT**

**The token screen appears ONCE only.**

You'll see:
```
Bot's Access Token:
[Very long string of random characters starting with something like:]
YmQwNzJkM2EtZjk5Ny00ZDVmLWE4NTMtOGNjOWYxNjBiNzBjMmRlNWVh...
[continues for many more characters]
```

**What you MUST do RIGHT NOW:**

1. **Copy the ENTIRE token** (it's very long - make sure you get it all)
2. **Save it immediately** - you cannot retrieve it later!
3. **If you lose it**, you must regenerate (which invalidates the old one)

**Save to secure file:**

```bash
# Open Terminal
# Create secure token file
echo "PASTE_YOUR_ENTIRE_TOKEN_HERE" > ~/cisco-rag-demo/webex-bot-token.txt

# Secure permissions (only you can read)
chmod 600 ~/cisco-rag-demo/webex-bot-token.txt

# Verify it's saved correctly
cat ~/cisco-rag-demo/webex-bot-token.txt
```

**Verify the token:**
- Should be very long (100+ characters)
- Should NOT have any newlines or spaces
- Should start with alphanumeric characters

💡 **Pro tip:** Also save it to a password manager for backup!

---

**Step 6: Record Bot Information**

**On the bot details page, note:**

```bash
# Save all bot information
cat > ~/cisco-rag-demo/webex-bot-info.txt << EOF
Bot Name: RAG Assistant
Bot Username: rag-assistant
Bot Email: rag-assistant@webex.bot
Bot ID: $(echo "Copy from Webex page")
Created: $(date)
EOF

# View later
cat ~/cisco-rag-demo/webex-bot-info.txt
```

**Bot email format will be:** `your-username@webex.bot`

**Example:**
```
Bot Name: RAG Assistant
Bot Email: rag-assistant@webex.bot
Bot ID: Y2lzY29zcGFyazovL3VzL1BFT1BMRS8xMjM0NTY3OC05YWJjLWRlZjA=
```

---

### Verify Bot Creation

**Test bot token works:**

```bash
# Set token as variable
BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)

# Test API call
curl https://webexapis.com/v1/people/me \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**Expected response:**
```json
{
  "id": "Y2lzY29zcGFyazovL...",
  "emails": ["rag-assistant@webex.bot"],
  "displayName": "RAG Assistant",
  "nickName": "RAG Assistant",
  "type": "bot",
  "created": "2024-12-18T10:30:00.000Z"
}
```

**If you see this JSON response:** ✅ Bot token works!  
**If you see error:** ⚠️ Token may be incorrect, check file

---

### Understanding Bot Limitations

**Bots can:**
- ✅ Receive messages (direct and @mentions)
- ✅ Send messages to spaces they're in
- ✅ Read message content
- ✅ See who sent messages
- ✅ Work in 1-on-1 and group spaces

**Bots cannot:**
- ❌ Initiate conversations (must be added first)
- ❌ See messages in spaces they're not in
- ❌ Create spaces
- ❌ Remove themselves from spaces

**For our use case:** These limitations are perfect - we want controlled access.

---

### Success Checkpoint

Before moving to tunnel setup, verify:

- [ ] Bot created in Webex developer portal
- [ ] Bot token saved to `~/cisco-rag-demo/webex-bot-token.txt`
- [ ] Token file has secure permissions (600)
- [ ] Bot info saved to `webex-bot-info.txt`
- [ ] Test API call returns bot details
- [ ] You understand what bots can/cannot do

✅ **Bot account ready! Moving to tunnel setup.**

---

## Part 3C: Setup Tunnel

**Time:** 15 minutes  
**What you'll do:** Create secure pathway through your firewall  
**Why:** Webex needs to send webhook notifications to your Mac

---

### Why We Need a Tunnel

**The problem:**
```
[Webex Cloud] wants to send POST → [Your Mac:5678]
                                        ↑
                                  Behind firewall!
                                  Not accessible
```

**Without tunnel:**
- Your Mac is behind home/office firewall
- n8n listening on `localhost:5678`
- "localhost" only accessible from your Mac
- Webex can't reach it

**With tunnel:**
```
[Webex Cloud] → [Tunnel Service] → [Your Mac:5678]
                     ↑                    ↑
              Public URL          Private localhost
```

**Network analogy:**
- Like NAT/PAT port forwarding
- Or like a VPN tunnel
- Creates pathway through firewall
- No firewall configuration needed!

---

### Tunnel Options - Which to Choose?

**We'll show two methods:**

| Feature | localhost.run | ngrok |
|---------|--------------|-------|
| **Installation** | None (uses SSH) | Required |
| **Speed to setup** | 2 minutes | 5-10 minutes |
| **Reliability** | Good | Excellent |
| **Features** | Basic | Advanced (web UI) |
| **URL stability** | Changes on restart | Changes (free) / Static (paid) |
| **Best for** | Quick start, demos | Regular use, debugging |

**Recommendation:** Start with localhost.run (faster), switch to ngrok if needed.

---

### Method A: localhost.run (Recommended)

**Advantages:**
- ✅ No installation required
- ✅ Uses built-in macOS SSH
- ✅ Works in 2 minutes
- ✅ Perfect for testing and demos

**Disadvantage:**
- ⚠️ URL changes every restart (normal for free tier)

---

**Step 1: Open New Terminal Window**

⚠️ **IMPORTANT:** This terminal must stay open during demos!

**Open Terminal:**
- Applications → Utilities → Terminal
- Or use existing Terminal, new tab

---

**Step 2: Start the Tunnel**

```bash
ssh -R 80:localhost:5678 nokey@localhost.run
```

**Command breakdown:**
- `ssh` - Secure Shell (built into macOS)
- `-R` - Remote port forwarding
- `80:localhost:5678` - Forward port 80 → localhost:5678
- `nokey@localhost.run` - Connect to localhost.run service
- `nokey` - No authentication key needed (free tier)

**Press Enter**

---

**Step 3: Wait for Connection**

**You'll see output like:**
```
** Allow remote port forwarding **
Connecting to localhost.run...
Connect to http://random-words-12.localhost.run or https://random-words-12.localhost.run
```

**Important lines:**
```
Connect to https://random-words-12.localhost.run
                   ↑
            This is YOUR unique URL
            (will be different each time)
```

**Copy the HTTPS URL** (not HTTP)

---

**Step 4: Save Your Tunnel URL**

**In ANOTHER terminal window** (keep tunnel running!):

```bash
# Save the URL
echo "https://your-actual-url.localhost.run" > ~/cisco-rag-demo/tunnel-url.txt

# Example:
echo "https://random-words-12.localhost.run" > ~/cisco-rag-demo/tunnel-url.txt

# Verify
cat ~/cisco-rag-demo/tunnel-url.txt
```

---

**Step 5: Test Tunnel Works**

**In the second terminal:**

```bash
# Test tunnel forwards to n8n
curl $(cat ~/cisco-rag-demo/tunnel-url.txt)
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>n8n.io - Workflow Automation</title>
...
```

**If you see HTML:** ✅ Tunnel working!  
**If connection refused:** ⚠️ Check n8n is running

---

**Important Notes About localhost.run:**

⚠️ **URL Changes Every Restart**

This is **NORMAL and EXPECTED** for free tier:

```
First run:  https://happy-cat-42.localhost.run
Stop tunnel
Restart:    https://brave-dog-73.localhost.run  ← Different!
```

**What this means:**
- Every time you restart tunnel, new URL
- Must update webhook with new URL (we'll show how)
- This is a free tier limitation
- Paid tunnels offer static URLs

**Accept this as normal behavior** - it's not a bug!

---

**Tunnel Terminal Management:**

**While tunnel is running:**
- Terminal shows connection status
- May show connection messages
- Do NOT close this terminal
- Can minimize it

**To stop tunnel:**
- Press `Ctrl+C` in tunnel terminal
- Connection closes
- URL becomes invalid

**To restart:**
- Run same `ssh -R` command
- Get new URL
- Update webhook (see Part 3G)

---

### Method B: ngrok (Alternative)

**When to use ngrok:**
- localhost.run not working for you
- Want web debugging interface
- Need better reliability
- Planning frequent demos

**Skip this if localhost.run worked for you!**

---

**Option B1: Install via Homebrew** (if available)

```bash
# Check if Homebrew installed
which brew

# If brew exists, install ngrok
brew install ngrok/ngrok/ngrok
```

**If this works:** Skip to ngrok configuration below.

---

**Option B2: Manual Installation** (macOS without Homebrew)

**Step 1: Download ngrok**

Go to: https://ngrok.com/download

**Select:** macOS (ARM64 for M-series, AMD64 for Intel)

**Or download via command line:**
```bash
# For M-series Mac (M1, M2, M3, M4):
cd ~/Downloads
curl -O https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-darwin-arm64.zip

# For Intel Mac:
curl -O https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-darwin-amd64.zip

# Unzip
unzip ngrok-*.zip

# Move to PATH location
sudo mv ngrok /usr/local/bin/

# Verify
ngrok version
```

---

**Step 2: Sign Up for ngrok Account**

Go to: https://dashboard.ngrok.com/signup

**Create free account:**
- Email address
- Password
- Verify email

**Free tier includes:**
- Dynamic URLs (change on restart)
- 40 connections/minute
- 1 online ngrok process
- Sufficient for testing!

---

**Step 3: Get Auth Token**

**After logging in:**
- Dashboard → "Your Authtoken"
- Copy the token

**Configure ngrok:**
```bash
ngrok config add-authtoken YOUR_TOKEN_HERE
```

**This saves to:** `~/.ngrok2/ngrok.yml`

---

**Step 4: Start ngrok Tunnel**

```bash
ngrok http 5678
```

**You'll see:**
```
ngrok                                                                   

Session Status                online
Account                       your-email@example.com
Version                       3.x.x
Region                        United States (us)
Latency                       25ms
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://abc123.ngrok-free.app -> http://localhost:5678

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

**Important lines:**
```
Forwarding: https://abc123.ngrok-free.app → http://localhost:5678
            ↑
        Your public URL
```

**Save this URL:**
```bash
echo "https://abc123.ngrok-free.app" > ~/cisco-rag-demo/tunnel-url.txt
```

**Test it works:**
```bash
curl $(cat ~/cisco-rag-demo/tunnel-url.txt)
```

---

**ngrok Web Interface:**

**Access:** http://localhost:4040

**Shows:**
- All HTTP requests in real-time
- Request/response details
- Useful for debugging
- Replay requests

**Very helpful for troubleshooting!**

---

### Tunnel Comparison Summary

**Choose localhost.run if:**
- ✅ Want fastest setup
- ✅ Just testing/demos
- ✅ Don't mind URL changes
- ✅ Minimal installation desired

**Choose ngrok if:**
- ✅ localhost.run not working
- ✅ Want debugging interface
- ✅ Need better reliability
- ✅ Planning regular use

**Either works!** Both are free, both change URLs on free tier.

---

### Success Checkpoint

Before moving to n8n workflow, verify:

- [ ] Tunnel running (localhost.run OR ngrok)
- [ ] Tunnel URL saved to `~/cisco-rag-demo/tunnel-url.txt`
- [ ] Can access tunnel URL from browser
- [ ] Tunnel terminal stays open
- [ ] You understand URL will change on restart

✅ **Tunnel ready! Moving to n8n configuration.**

---

## Part 3D: n8n Webex Workflow

**Time:** 20 minutes  
**What you'll do:** Import and configure the Webex bot workflow  
**Why:** Connects Webex webhooks to RAG system

---

### Download and Import Workflow

**The workflow file:**
Location in project: `/mnt/project/Webex_RAG_Bot.json`

**Import process (same as Part 2):**

1. **Open n8n:** http://localhost:5678
2. **Click "⋯" menu** (three dots, top-right)
3. **Select "Import from File"**
4. **Navigate to** `/mnt/project/`
5. **Select** `Webex_RAG_Bot.json`
6. **Click "Open"**

**Workflow loads on canvas.**

**Workflow name:** "Webex RAG Bot"

---

### Understanding the Workflow Architecture

**Visual representation:**

```
┌──────────────────────────────────────────────────────┐
│         12-NODE WEBEX BOT WORKFLOW                  │
└──────────────────────────────────────────────────────┘

[1] Webex Webhook
    │ Receives POST from Webex when message sent
    │ Contains message ID and room ID
    ↓
[2] Extract Webhook Data
    │ Pulls out messageId, roomId from webhook
    │ Prepares for next step
    ↓
[3] Get Message Details
    │ HTTP GET to Webex API
    │ Fetches full message content and metadata
    │ Requires bot token (authentication)
    ↓
[4] Validate & Extract Question
    │ CRITICAL: Filters bot's own messages
    │ Prevents infinite loops
    │ Cleans @mentions from text
    │ Extracts question
    ↓
[5] Get Collection UUID
    │ HTTP GET to ChromaDB
    │ Looks up "cisco_docs" collection
    │ Same as Part 2 workflows
    ↓
[6] Prepare Query
    │ Formats question for embedding
    │ Validates collection exists
    ↓
[7] Generate Question Embedding
    │ HTTP POST to Ollama
    │ Converts question to vector
    │ Same as Part 2 workflows
    ↓
[8] Prepare Search
    │ Formats search request
    │ Sets n_results=3
    ↓
[9] Search ChromaDB
    │ HTTP POST to ChromaDB
    │ Vector similarity search
    │ Returns top 3 relevant chunks
    │ Same as Part 2 workflows
    ↓
[10] Build Prompt
    │ Combines chunks + question
    │ Adds system instructions
    │ Shorter prompt (500 word limit for Webex)
    ↓
[11] Generate Answer
    │ HTTP POST to Ollama
    │ AI writes answer
    │ Same as Part 2 workflows
    ↓
[12] Send to Webex
    │ HTTP POST to Webex API
    │ Posts answer to original space
    │ Requires bot token
    ↓
[13] Respond to Webhook
    │ Returns 200 OK to Webex
    │ Acknowledges webhook receipt
```

---

### Why 12 Nodes vs 9 in Chat?

**Additional complexity:**

**Extra nodes for Webex:**
- **Extract Webhook Data** - Parse Webex notification
- **Get Message Details** - Fetch full message via API
- **Validate & Extract** - Filter bot messages (prevent loops)
- **Send to Webex** - Post response via API
- **Respond to Webhook** - Acknowledge to Webex

**Nodes 5-11 are identical to Part 2 chat workflow:**
- Same RAG process
- Same ChromaDB search
- Same AI generation
- Reuses all that knowledge!

**Think of it as:**
```
Webex adapter (Nodes 1-4) → RAG core (Nodes 5-11) → Webex adapter (Nodes 12-13)
```

---

### Node-by-Node Deep Dive

Let's understand what each node does:

---

**Node 1: Webex Webhook**

**Type:** `n8n-nodes-base.webhook`  
**Purpose:** HTTP POST endpoint for Webex notifications

**Configuration:**
```
HTTP Method: POST
Path: webex-bot
Response Mode: responseNode (CRITICAL!)
```

**What it receives when someone messages bot:**
```json
{
  "id": "webhook-event-id",
  "name": "RAG Bot Messages",
  "resource": "messages",
  "event": "created",
  "data": {
    "id": "message-id-123",
    "roomId": "space-id-456",
    "personId": "sender-id-789",
    "personEmail": "user@example.com",
    "created": "2024-12-18T10:30:00.000Z"
  }
}
```

**Why "responseNode" mode?**
- Our workflow takes 5-10 seconds (AI processing)
- Webex expects response within 5 seconds
- Must acknowledge immediately, process async
- Then send actual answer via separate API call

**Network analogy:** Like TCP ACK - acknowledge receipt, then process payload.

---

**Node 2: Extract Webhook Data**

**Type:** `n8n-nodes-base.code`  
**Purpose:** Parse webhook payload

**What it does:**
```javascript
// Get data from webhook
const webhookData = $input.first().json.body.data;

// Extract what we need
return [{
  json: {
    messageId: webhookData.id,
    roomId: webhookData.roomId,
    personId: webhookData.personId,
    personEmail: webhookData.personEmail
  }
}];
```

**Why needed?**
- Webhook data is nested
- Need to extract key fields
- Prepare for next step

---

**Node 3: Get Message Details**

**Type:** `n8n-nodes-base.httpRequest`  
**Purpose:** Fetch full message content from Webex

**Why needed?**
- Webhook only has metadata (message ID, room ID)
- Doesn't include actual message text!
- Must fetch full details via API

**Request:**
```http
GET https://webexapis.com/v1/messages/{messageId}
Authorization: Bearer YOUR_BOT_TOKEN_HERE
```

**Response contains:**
```json
{
  "id": "message-id-123",
  "roomId": "space-id-456",
  "text": "What is the network uptime?",
  "personId": "sender-id-789",
  "personEmail": "user@example.com",
  "created": "2024-12-18T10:30:00.000Z"
}
```

⚠️ **CRITICAL CONFIGURATION:** You must update bot token here!

**How to update:**
1. Click "Get Message Details" node
2. Find "Header Parameters" section
3. Find "Authorization" header
4. Current value: `Bearer YOUR_BOT_TOKEN_HERE`
5. Replace with: `Bearer YOUR_ACTUAL_TOKEN`

**Get your token:**
```bash
cat ~/cisco-rag-demo/webex-bot-token.txt
```

**Updated header:**
```
Authorization: Bearer YmQwNzJkM2EtZjk5Ny00ZDVmLWE4NTMt...
                      ↑
                Your actual token
```

---

**Node 4: Validate & Extract Question**

**Type:** `n8n-nodes-base.code`  
**Purpose:** Critical filtering and cleaning

**What it does:**

**1. Filter bot's own messages:**
```javascript
const BOT_EMAIL = 'YOUR_BOT_EMAIL@webex.bot';

if (messageData.personEmail === BOT_EMAIL) {
  throw new Error('Ignoring message from bot itself');
}
```

**Why critical?**
```
Without filtering:
1. Bot sends answer
2. Webex sends webhook about bot's message
3. Bot processes its own message
4. Bot sends another answer
5. Loop continues forever! 🔄
```

**With filtering:**
```
1. Bot sends answer
2. Webex sends webhook
3. Bot checks: "Is this from me?"
4. Yes → Throw error, stop processing ✅
5. No loop!
```

**2. Remove @mentions:**
```javascript
// When @mentioned in groups, text has HTML
question = question.replace(/<spark-mention[^>]*>.*?<\/spark-mention>/gi, '');
question = question.replace(/@\w+/g, '');
question = question.trim();
```

**Before cleaning:**
```
"<spark-mention data-object-id='...'>RAG Assistant</spark-mention> What is the uptime?"
```

**After cleaning:**
```
"What is the uptime?"
```

⚠️ **MUST UPDATE BOT EMAIL:**

1. Click "Validate & Extract Question" node
2. Find line: `const BOT_EMAIL = 'YOUR_BOT_EMAIL@webex.bot';`
3. Replace with your actual bot email

**Get your bot email:**
```bash
cat ~/cisco-rag-demo/webex-bot-info.txt | grep "Bot Email"
```

**Expected "failures":**

You'll see executions with error: "Ignoring message from bot itself"

**This is NORMAL and INTENTIONAL!**
- Not a bug
- Prevents infinite loops
- Expected behavior
- Don't try to "fix" this

---

**Nodes 5-11: RAG Processing**

These are **identical to Part 2 chat workflow:**

**Node 5:** Get Collection UUID (ChromaDB lookup)  
**Node 6:** Prepare Query (format for embedding)  
**Node 7:** Generate Question Embedding (Ollama)  
**Node 8:** Prepare Search (format search request)  
**Node 9:** Search ChromaDB (vector similarity)  
**Node 10:** Build Prompt (chunks + question)  
**Node 11:** Generate Answer (Ollama LLM)

**Refer to Part 2 for detailed explanations of these nodes.**

**Key difference in Node 10 (Build Prompt):**
```javascript
// Webex-specific: Keep responses concise
const systemPrompt = `You are a helpful AI assistant for Cisco network assessments.
Provide accurate, concise answers based on the context.
Keep responses under 500 words for Webex readability.`;
```

**Why 500 words?**
- Webex mobile app displays
- Easier to read on phone
- Forces concise answers
- Better user experience

---

**Node 12: Send to Webex**

**Type:** `n8n-nodes-base.httpRequest`  
**Purpose:** Post answer back to Webex

**Request:**
```http
POST https://webexapis.com/v1/messages
Authorization: Bearer YOUR_BOT_TOKEN_HERE
Content-Type: application/json

{
  "roomId": "space-id-456",
  "text": "According to the network assessment, uptime is 99.94%..."
}
```

⚠️ **CRITICAL CONFIGURATION:** You must update bot token here too!

**How to update:**
1. Click "Send to Webex" node
2. Find "Header Parameters" section
3. Find "Authorization" header
4. Replace with your actual bot token (same as Node 3)

**Why two places?**
- Node 3: Read permission (fetch messages)
- Node 12: Write permission (send messages)
- Both need authentication
- Must use same token

---

**Node 13: Respond to Webhook**

**Type:** `n8n-nodes-base.respondToWebhook`  
**Purpose:** Acknowledge webhook to Webex

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

OK
```

**Why needed?**
- Webex expects acknowledgment
- Confirms webhook was received
- Prevents retries
- Good practice

**Executes after Node 12** (answer sent) but returns to Webex immediately.

---

### Critical Configuration Summary

**Three things you MUST configure:**

**1. Bot token in "Get Message Details" (Node 3)**
```
Authorization: Bearer YOUR_ACTUAL_TOKEN
```

**2. Bot token in "Send to Webex" (Node 12)**
```
Authorization: Bearer YOUR_ACTUAL_TOKEN
```

**3. Bot email in "Validate & Extract Question" (Node 4)**
```javascript
const BOT_EMAIL = 'your-actual-bot@webex.bot';
```

**Get values from:**
```bash
# Bot token
cat ~/cisco-rag-demo/webex-bot-token.txt

# Bot email
cat ~/cisco-rag-demo/webex-bot-info.txt | grep "Bot Email"
```

---

### Activate the Workflow

**Important:** Unlike document loader, this workflow must be **active** (running continuously).

**Activation steps:**

1. **Save workflow first**
   - Click "Save" button
   - Gives it a chance to validate

2. **Toggle to Active**
   - Look for "Active" toggle switch
   - Currently shows ⭕ (inactive/off)
   - Click it → Changes to ✅ (active/on)

3. **Confirm activation**
   - Switch should be green/blue
   - Workflow now running in background
   - Will process webhooks as they arrive

**What "Active" means:**
- Workflow runs continuously
- Webhook endpoint available
- Processes messages in real-time
- Stays active even if you close browser

---

### Get Webhook URL

**With workflow active, note webhook URL:**

**Method 1: Click workflow name**
- Top of canvas shows workflow details
- Webhook URL listed

**Method 2: Click "Webex Webhook" node**
- Shows node details
- "Production URL" displayed

**Format:**
```
https://your-tunnel-url/webhook/webex-bot
```

**Example:**
```
https://random-words-12.localhost.run/webhook/webex-bot
```

**Save for next step:**
```bash
# Full webhook URL
TUNNEL_URL=$(cat ~/cisco-rag-demo/tunnel-url.txt)
echo "${TUNNEL_URL}/webhook/webex-bot" > ~/cisco-rag-demo/webhook-url.txt

# Verify
cat ~/cisco-rag-demo/webhook-url.txt
```

---

### Success Checkpoint

Before moving to webhook registration, verify:

- [ ] Workflow imported successfully
- [ ] All 12 nodes visible on canvas
- [ ] Bot token updated in Node 3 (Get Message Details)
- [ ] Bot token updated in Node 12 (Send to Webex)
- [ ] Bot email updated in Node 4 (Validate & Extract)
- [ ] Workflow activated (toggle shows ✅)
- [ ] Workflow saved
- [ ] Webhook URL noted

✅ **Workflow configured and active! Ready to register webhook.**

---

## Part 3E: Register Webhook

**Time:** 10 minutes  
**What you'll do:** Tell Webex where to send notifications  
**Why:** Connects Webex to your n8n workflow

---

### Understanding Webhook Registration

**What a webhook does:**
```
[Webex] "Someone sent a message to your bot!"
   ↓
   POST notification →
   ↓
[Your webhook URL] Receives notification, processes
```

**Like giving Webex your phone number:**
- Webex: "Where should I notify you?"
- You: "Send notifications to https://my-url/webhook/webex-bot"
- Webex: "Got it! I'll POST there when messages arrive."

---

### Registration Process

**Prepare your environment variables:**

```bash
# Get values from saved files
BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)
TUNNEL_URL=$(cat ~/cisco-rag-demo/tunnel-url.txt)
WEBHOOK_URL="${TUNNEL_URL}/webhook/webex-bot"

# Verify all set
echo "Bot Token: ${BOT_TOKEN:0:20}..."
echo "Tunnel URL: $TUNNEL_URL"
echo "Webhook URL: $WEBHOOK_URL"
```

**Expected output:**
```
Bot Token: YmQwNzJkM2EtZjk5Ny0...
Tunnel URL: https://random-words-12.localhost.run
Webhook URL: https://random-words-12.localhost.run/webhook/webex-bot
```

---

### Register Webhook with Webex

**Run registration command:**

```bash
curl -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${WEBHOOK_URL}\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }"
```

**Command breakdown:**
- `POST /v1/webhooks` - Create new webhook
- `Authorization: Bearer $BOT_TOKEN` - Authenticate as bot
- `name` - Friendly name (for your reference)
- `targetUrl` - Where to send notifications (your webhook)
- `resource` - What to watch (messages)
- `event` - When to notify (when created)

---

**Expected response:**

```json
{
  "id": "Y2lzY29zcGFyazovL3VzL1dFQkhPT0svMTIzNDU2Nzg5YWJjZGVm",
  "name": "RAG Bot Messages",
  "targetUrl": "https://random-words-12.localhost.run/webhook/webex-bot",
  "resource": "messages",
  "event": "created",
  "filter": null,
  "secret": null,
  "status": "active",
  "created": "2024-12-18T10:30:00.000Z"
}
```

**Key fields:**
- `id` - Webhook ID (save this!)
- `targetUrl` - Should match your webhook URL
- `status` - Should be "active"
- `created` - Timestamp of registration

---

### Save Webhook ID

**CRITICAL: Save the webhook ID**

```bash
# Extract ID from response (if you saved it)
# Or from next verification step

# Save to file
echo "Y2lzY29zcGFyazovL3VzL1dFQkhPT0svMTIzNDU2Nzg5YWJjZGVm" > ~/cisco-rag-demo/webhook-id.txt

# Verify
cat ~/cisco-rag-demo/webhook-id.txt
```

**Why save webhook ID?**
- Need it to delete webhook later
- Need it when tunnel URL changes
- Makes management much easier

---

### Verify Webhook Registration

**List all webhooks:**

```bash
curl https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" | jq '.'
```

**Check that:**
- ✓ Your webhook appears in list
- ✓ Status is "active"
- ✓ targetUrl matches your webhook URL
- ✓ resource is "messages"
- ✓ event is "created"

**If multiple webhooks exist:**
- You might see old ones from previous attempts
- Can delete old ones (see Daily Operations)
- Only need one active

---

### Test Webhook Connectivity

**Manual test (optional but recommended):**

```bash
# Send test POST to your webhook
curl -X POST $(cat ~/cisco-rag-demo/webhook-url.txt) \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

**Expected:**
- Tunnel terminal shows POST request
- n8n shows new execution (might fail - that's OK for test)
- Confirms connectivity works

**If no activity:**
- Check tunnel is running
- Check n8n workflow is active
- Check URL is correct

---

### Understanding Webhook Lifecycle

**Normal webhook flow:**

```
1. User sends message in Webex
2. Webex checks: Does bot have webhooks?
3. Yes → POST to targetUrl(s)
4. Your webhook receives notification
5. Processes and responds
```

**When tunnel URL changes (will happen):**

```
1. Tunnel restarts → New URL
2. Webhook still points to OLD URL
3. Webex POSTs to old URL → Fails
4. Bot stops responding
5. Must update webhook with new URL
```

**This is NORMAL with free tier!**

We'll cover update procedure in Part 3G (Daily Operations).

---

### Success Checkpoint

Before testing bot, verify:

- [ ] Webhook registered with Webex
- [ ] Webhook ID saved to file
- [ ] targetUrl matches current tunnel URL
- [ ] Webhook status is "active"
- [ ] Can verify webhook in list
- [ ] Test POST reaches n8n
- [ ] Understand URL will change (normal)

✅ **Webhook registered! Ready to test bot.**

---

## Part 3F: Test Your Bot

**Time:** 15 minutes  
**What you'll do:** Send first messages and verify bot responds  
**Why:** Confirm complete system works end-to-end

---

### Find Your Bot in Webex Teams

**Desktop/Web Client:**

1. **Open Webex Teams**
   - Application or https://web.webex.com

2. **Start new message**
   - Click "+" button
   - Or "New Message"

3. **Search for bot**
   - Enter bot email: `your-bot@webex.bot`
   - Example: `rag-assistant@webex.bot`

4. **Click bot to open chat**
   - Opens 1-on-1 space
   - Bot shows as "Bot" badge

---

**Mobile App (iOS/Android):**

1. **Open Webex Teams app**
   - Tap Webex icon

2. **New message**
   - Tap "+" icon (bottom)

3. **Search for bot**
   - Enter bot email
   - Example: `rag-assistant@webex.bot`

4. **Tap bot to start chat**
   - Opens direct message space

---

### Send First Test Message

**Type:** `test`

**Press Send**

---

### What to Expect

**Immediate (< 1 second):**
```
✓ Message sends (checkmark appears)
```

**Brief pause (1-2 seconds):**
```
(Processing webhook, fetching message)
```

**Bot starts typing (2-3 seconds):**
```
... (typing indicator)
```

**Answer appears (total 5-10 seconds):**
```
Bot: Hello! I'm your RAG Assistant. I can help answer 
     questions about your Cisco network assessment documents. 
     Try asking me about network uptime, equipment, budgets, 
     or contact information.
```

---

### Understanding Response Timing

**First message of session (10-15 seconds):**
- AI models loading into RAM
- Completely normal
- Subsequent messages will be faster

**Subsequent messages (5-8 seconds):**
- Models already in RAM
- Typical performance
- Expected timing

**If slower than 20 seconds consistently:**
- See troubleshooting section
- Likely needs model pre-loading

---

### What's Happening Behind the Scenes

**Complete trace of your "test" message:**

```
[Second 0] You type "test" in Webex app
            ↓
[Second 1] Message sent to Webex cloud
            ↓
[Second 2] Webex sends webhook POST to tunnel
            ↓
            Tunnel forwards to n8n localhost:5678
            ↓
[Second 3] n8n "Webex Webhook" node receives
            ↓
            n8n extracts message ID and room ID
            ↓
[Second 4] n8n fetches full message from Webex API
            ↓
            n8n validates (is it from bot? no → continue)
            ↓
[Second 5] n8n looks up ChromaDB collection
            ↓
            n8n generates question embedding (Ollama)
            ↓
[Second 6] n8n searches ChromaDB
            ↓
            n8n retrieves top 3 chunks
            ↓
[Second 7] n8n builds prompt with context
            ↓
[Seconds 7-10] Ollama generates answer (AI writing)
            ↓
[Second 10] n8n posts answer to Webex API
            ↓
[Second 10] Answer appears in your Webex chat!
```

**Total: 10 seconds**  
**All processing: On your Mac**  
**Privacy: Complete**

---

### Ask Real Questions

**Try these questions about your loaded documents:**

**Question 1: Network performance**
```
You: What is the network uptime?

Expected: Bot responds with uptime percentage from 
          your document (e.g., "99.94% year-to-date")
```

---

**Question 2: Equipment information**
```
You: What equipment needs replacement?

Expected: Bot lists equipment from document 
          (e.g., "15 Cisco 3850 switches reach EOS...")
```

---

**Question 3: Budget inquiry**
```
You: What is the proposed FY2025 budget?

Expected: Bot provides budget breakdown from document
          (e.g., "Total $325,000: Equipment $250,000...")
```

---

**Question 4: Contact information**
```
You: Who should I contact about network issues?

Expected: Bot provides contacts from document
          (e.g., "Network Lead: Mike Chen, mike.chen@...")
```

---

**Question 5: Analytical question**
```
You: Does the network meet SLA requirements?

Expected: Bot analyzes and responds
          (e.g., "Yes, 99.94% exceeds the 99.9% SLA target")
```

---

### Verify Accuracy

**For each answer, check:**
- ✓ Cites information from your actual documents
- ✓ Provides specific numbers/names (not generic)
- ✓ Answer makes sense
- ✓ Addresses the question asked

**If answers are wrong/generic:**
- Documents may not be loaded
- See troubleshooting section

---

### Test in Group Space (Optional)

**Create test space:**

1. **Create new Webex space**
   - Add yourself
   - Add a colleague (optional)
   - Add your bot (search by email)

2. **Use @mention to ask questions**
   ```
   @RAG Assistant What is the network uptime?
   ```

3. **Bot responds in space**
   - Visible to all members
   - Can be used for team collaboration

**Group space behavior:**
- Bot only responds when @mentioned
- Won't respond to every message (good!)
- Multiple people can ask questions
- Great for team demos

---

### Monitoring Execution

**Watch in n8n:**

1. **Open n8n** (http://localhost:5678)
2. **Click "Executions" tab** (left sidebar)
3. **See real-time executions**

**For each message you send:**
```
Execution appears:
✅ Green = Success
⚠️ Yellow/Orange = Warning (might be intentional)
❌ Red = Error

Click execution to see details:
- Which nodes executed
- Data at each step
- Any errors
```

**You'll see some "errors" that are actually intentional:**
```
Error: "Ignoring message from bot itself"
Status: ❌ Failed
Explanation: This is NORMAL! Prevents infinite loops.
             Bot's own messages trigger this.
             Don't try to fix it!
```

---

**Watch in tunnel terminal:**

**Your tunnel terminal should show:**
```
POST /webhook/webex-bot HTTP/1.1
200 OK

POST /webhook/webex-bot HTTP/1.1
200 OK
```

**Each POST = One webhook received**

**If no POSTs appear when you send message:**
- Webhook not registered correctly
- Or tunnel URL changed
- See troubleshooting

---

### Mobile Testing

**Test from phone/tablet:**

1. **Open Webex app on mobile**
2. **Find bot chat**
3. **Send question**
4. **Receive answer**

**This proves:**
- ✓ Works from anywhere
- ✓ Mobile-friendly responses
- ✓ Real-world use case
- ✓ Team members can use it

**Share bot with team:**
- Give them bot email
- They can add bot to Webex
- They can ask questions
- Everyone benefits!

---

### Success Checkpoint

Before moving to daily operations, verify:

- [ ] Can find bot in Webex (desktop and mobile)
- [ ] Test message receives response
- [ ] Real questions get accurate answers
- [ ] Response time reasonable (<15 sec first, <10 sec after)
- [ ] Answers cite actual document content
- [ ] Can see executions in n8n
- [ ] Can see POSTs in tunnel terminal
- [ ] Group space @mentions work (if tested)
- [ ] Mobile access works

✅ **Bot working! Ready to learn daily operations.**

---

## Part 3G: Daily Operations

**Time:** 10 minutes  
**What you'll learn:** Day-to-day bot management  
**Why:** Keep bot running smoothly

---

### Daily Startup Sequence

**Every time you want to use the bot, follow this sequence:**

---

**Terminal 1: Infrastructure**

```bash
# 1. Start containers
podman start chromadb n8n

# 2. Start Ollama
ollama serve &

# 3. Wait for services to initialize
sleep 20

# 4. CRITICAL: Pre-load models (saves time on first query)
echo "Warming up models..."
ollama run llama3.2:3b "warmup" < /dev/null
ollama run nomic-embed-text "warmup" < /dev/null

echo "✅ Infrastructure ready!"
```

**Why pre-load models?**
```
Without pre-loading:
- First query: 10-15 seconds (models loading)
- User thinks bot is broken

With pre-loading:
- First query: 5-8 seconds (models in RAM)
- Much better user experience!
```

**Time investment:** 1 minute pre-loading saves 5+ seconds per query

---

**Terminal 2: Start Tunnel**

**Open new terminal tab/window:**

```bash
# Start localhost.run tunnel
ssh -R 80:localhost:5678 nokey@localhost.run
```

**Watch for connection message:**
```
Connect to https://new-random-words-99.localhost.run
              ↑
         Note this URL - it's new!
```

**Save new URL:**

```bash
# In ANOTHER terminal (keep tunnel running)
echo "https://new-random-words-99.localhost.run" > ~/cisco-rag-demo/tunnel-url.txt

# Verify
cat ~/cisco-rag-demo/tunnel-url.txt
```

---

**Terminal 3: Update Webhook**

**Because tunnel URL changed, webhook must be updated:**

```bash
# Load environment
BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)
TUNNEL_URL=$(cat ~/cisco-rag-demo/tunnel-url.txt)
WEBHOOK_URL="${TUNNEL_URL}/webhook/webex-bot"

# Get old webhook ID (if exists)
WEBHOOK_ID=$(cat ~/cisco-rag-demo/webhook-id.txt 2>/dev/null)

# Delete old webhook if exists
if [ ! -z "$WEBHOOK_ID" ]; then
  echo "Deleting old webhook..."
  curl -X DELETE https://webexapis.com/v1/webhooks/$WEBHOOK_ID \
    -H "Authorization: Bearer $BOT_TOKEN"
  echo ""
fi

# Register new webhook
echo "Registering new webhook..."
NEW_WEBHOOK=$(curl -s -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${WEBHOOK_URL}\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }")

# Save new webhook ID
echo "$NEW_WEBHOOK" | jq -r '.id' > ~/cisco-rag-demo/webhook-id.txt

# Verify
echo "✅ Webhook updated!"
echo "New webhook ID: $(cat ~/cisco-rag-demo/webhook-id.txt)"
```

---

**Create Update Script (Optional)**

**Make webhook updating easier:**

```bash
cat > ~/cisco-rag-demo/update-webhook.sh << 'SCRIPT'
#!/bin/bash
# Quick webhook update script

BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)
TUNNEL_URL=$(cat ~/cisco-rag-demo/tunnel-url.txt)
WEBHOOK_URL="${TUNNEL_URL}/webhook/webex-bot"
WEBHOOK_ID=$(cat ~/cisco-rag-demo/webhook-id.txt 2>/dev/null)

echo "Current tunnel: $TUNNEL_URL"

# Delete old
if [ ! -z "$WEBHOOK_ID" ]; then
  echo "Deleting old webhook..."
  curl -s -X DELETE https://webexapis.com/v1/webhooks/$WEBHOOK_ID \
    -H "Authorization: Bearer $BOT_TOKEN" > /dev/null
fi

# Register new
echo "Registering new webhook..."
NEW_WEBHOOK=$(curl -s -X POST https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"RAG Bot Messages\",
    \"targetUrl\": \"${WEBHOOK_URL}\",
    \"resource\": \"messages\",
    \"event\": \"created\"
  }")

# Save ID
echo "$NEW_WEBHOOK" | jq -r '.id' > ~/cisco-rag-demo/webhook-id.txt

echo "✅ Webhook updated!"
echo "ID: $(cat ~/cisco-rag-demo/webhook-id.txt)"
SCRIPT

chmod +x ~/cisco-rag-demo/update-webhook.sh
```

**Next time, just run:**
```bash
~/cisco-rag-demo/update-webhook.sh
```

---

**Verify Everything Works:**

```bash
# 1. Check containers
podman ps

# 2. Check n8n workflow active
open http://localhost:5678  # Check toggle is ✅

# 3. Test bot
# Send "test" message in Webex
# Should respond within 5-10 seconds
```

---

### When Tunnel URL Changes

**This happens EVERY restart with free tier.**

**Signs tunnel URL changed:**
- Bot suddenly stops responding
- Was working, now doesn't work
- No errors in n8n (executions just don't appear)

**Quick fix procedure:**

```bash
# 1. Note new tunnel URL (from tunnel terminal)
echo "https://new-url.localhost.run" > ~/cisco-rag-demo/tunnel-url.txt

# 2. Update webhook
~/cisco-rag-demo/update-webhook.sh

# 3. Test
# Send message in Webex
# Should work again!
```

**Time required:** 30 seconds

**Frequency:** Every time you restart tunnel

**This is normal behavior for free tier!**

---

### Pre-Demo Checklist

**15 minutes before any demo:**

```bash
# 1. Restart everything fresh
podman restart chromadb n8n
killall ollama
ollama serve &
sleep 20

# 2. Pre-load models (CRITICAL!)
ollama run llama3.2:3b "warmup" < /dev/null
ollama run nomic-embed-text "warmup" < /dev/null

# 3. Verify tunnel running
# Check terminal shows connection

# 4. Test query performance
time ./query_rag.py "What is the network uptime?"
# Should complete in 5-8 seconds

# 5. Test bot in Webex
# Send test message
# Verify 5-10 second response

# 6. Prepare sample questions
# Have 5-10 questions ready
# Know expected answers
```

**Everything working?** ✅ Ready to demo!

---

### Monitoring During Operation

**Watch for issues:**

**Tunnel terminal:**
- Should show POST requests when messages sent
- No POSTs = webhook problem

**n8n Executions tab:**
- Should show new executions
- Green = success
- Red errors (check which node failed)

**Webex:**
- Bot should respond within 10 seconds
- If no response, check above

---

### Performance Optimization

**Make queries faster:**

**1. Keep models loaded:**
```bash
# Every 30 minutes (if idle)
ollama run llama3.2:3b "ping" < /dev/null
```

**2. Increase Podman resources** (if you have RAM):
```bash
podman machine stop
podman machine set --memory 16384  # 16GB instead of 12GB
podman machine start
```

**3. Reduce chunk count** (in n8n workflow):
```javascript
// In "Prepare Search" node
// Change from 3 to 2 chunks
{
  query_embeddings: [embedding],
  n_results: 2  // Faster search, still good quality
}
```

---

### Shutdown Procedure (Optional)

**When done using bot for the day:**

```bash
# Stop tunnel (Ctrl+C in tunnel terminal)

# Stop services
podman stop chromadb n8n
killall ollama

# Everything stopped
```

**Note:** On Mac, can leave running overnight - doesn't consume much when idle.

---

### Managing Multiple Webhooks

**Check existing webhooks:**
```bash
BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)
curl https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" | jq '.'
```

**Delete specific webhook:**
```bash
curl -X DELETE https://webexapis.com/v1/webhooks/{webhook-id} \
  -H "Authorization: Bearer $BOT_TOKEN"
```

**Delete all webhooks (fresh start):**
```bash
# Get all webhook IDs
curl -s https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" | \
  jq -r '.items[].id' | \
while read id; do
  echo "Deleting $id"
  curl -s -X DELETE https://webexapis.com/v1/webhooks/$id \
    -H "Authorization: Bearer $BOT_TOKEN"
done

echo "All webhooks deleted"
```

---

### Success Checkpoint

Verify you understand:

- [ ] Daily startup sequence (3 terminals)
- [ ] Why pre-loading models matters
- [ ] How to update webhook when URL changes
- [ ] Pre-demo checklist
- [ ] Monitoring techniques
- [ ] Shutdown procedure

✅ **Ready for troubleshooting guide!**

---

## Part 3H: Troubleshooting Webex Integration

**Time:** 10 minutes (reading)  
**Purpose:** Solve common issues quickly

---

### Diagnostic Flowchart

**Bot doesn't respond to messages:**

```
[1] Send message in Webex
      ↓
[2] Check: Does tunnel terminal show POST request?
      ├─ NO → Problem A: Webhook not reaching tunnel
      │         See: Webhook Problem Solutions
      │
      └─ YES → Problem B: n8n not processing
                See: n8n Problem Solutions
```

---

### Problem A: Webhook Not Reaching Tunnel

**Symptoms:**
- Send message in Webex
- No POST appears in tunnel terminal
- No execution in n8n
- Bot silent

**Diagnostic steps:**

**Step 1: Verify tunnel running**
```bash
# Check tunnel terminal shows "Connected"
# Should say: "Connect to https://..."

# If not running:
ssh -R 80:localhost:5678 nokey@localhost.run
```

---

**Step 2: Verify tunnel URL accessible**
```bash
# Test from outside
curl $(cat ~/cisco-rag-demo/tunnel-url.txt)

# Should return HTML from n8n
```

**If fails:**
- Tunnel not running → restart tunnel
- n8n not running → `podman start n8n`

---

**Step 3: Verify webhook points to current URL**
```bash
BOT_TOKEN=$(cat ~/cisco-rag-demo/webex-bot-token.txt)

# Get webhook details
curl https://webexapis.com/v1/webhooks \
  -H "Authorization: Bearer $BOT_TOKEN" | jq '.'

# Check targetUrl matches current tunnel URL
```

**If URLs don't match:**
```bash
# Update webhook
~/cisco-rag-demo/update-webhook.sh
```

---

**Step 4: Test webhook manually**
```bash
# Get webhook ID
WEBHOOK_ID=$(cat ~/cisco-rag-demo/webhook-id.txt)

# Check status
curl https://webexapis.com/v1/webhooks/$WEBHOOK_ID \
  -H "Authorization: Bearer $BOT_TOKEN" | jq '.'

# Verify status is "active"
```

**If not active or not found:**
- Re-register webhook
- Run update script

---

### Problem B: n8n Not Processing

**Symptoms:**
- POST appears in tunnel terminal
- But no response in Webex
- n8n shows executions

**Diagnostic steps:**

**Step 1: Check n8n workflow active**
```bash
# Open n8n
open http://localhost:5678

# Check "Webex RAG Bot" workflow
# Toggle should be ✅ green (active)
```

**If not active:**
- Click toggle to activate
- Click Save
- Test again

---

**Step 2: Check execution details**

1. **Open n8n**
2. **Click "Executions" tab**
3. **Find latest execution**
4. **Click to see details**

**Look for which node failed:**

```
Common failures:

Node 3 (Get Message Details):
❌ "401 Unauthorized"
Fix: Update bot token in this node

Node 4 (Validate & Extract):
❌ "Ignoring message from bot itself"
Status: This is NORMAL! Prevents loops.

Node 5 (Get Collection UUID):
❌ "Connection refused"
Fix: Start ChromaDB - podman start chromadb

Node 7 (Generate Question Embedding):
❌ "Connection refused"
Fix: Start Ollama - ollama serve &

Node 12 (Send to Webex):
❌ "401 Unauthorized"
Fix: Update bot token in this node
```

---

**Step 3: Verify bot tokens configured**

**Check both places:**

1. **Click "Get Message Details" node**
   - Check Authorization header
   - Should be: `Bearer YOUR_TOKEN`
   - Not: `Bearer YOUR_BOT_TOKEN_HERE`

2. **Click "Send to Webex" node**
   - Check Authorization header
   - Should be same token

**If wrong:**
```bash
# Get correct token
cat ~/cisco-rag-demo/webex-bot-token.txt

# Update in both nodes
# Save workflow
# Test again
```

---

**Step 4: Verify bot email configured**

**Click "Validate & Extract Question" node:**

Find line:
```javascript
const BOT_EMAIL = 'YOUR_BOT_EMAIL@webex.bot';
```

**Should be your actual bot email:**
```bash
# Get correct email
cat ~/cisco-rag-demo/webex-bot-info.txt | grep "Bot Email"

# Update in node
# Save workflow
# Test again
```

---

### Problem C: Slow Responses (>20 seconds)

**Symptoms:**
- Bot eventually responds
- But takes 15-30 seconds
- Inconsistent performance

**Solutions:**

**Solution 1: Pre-load models (most common fix)**
```bash
# Run before using bot
ollama run llama3.2:3b "warmup" < /dev/null
ollama run nomic-embed-text "warmup" < /dev/null

# Test again - should be faster
```

**Why this works:**
```
Without pre-loading:
- First query loads models into RAM (10-15 sec)
- Subsequent queries fast (5-8 sec)

With pre-loading:
- Models already in RAM
- All queries fast (5-8 sec)
```

---

**Solution 2: Check system resources**
```bash
# Open Activity Monitor
# Check "Memory Pressure"

# If yellow/red:
# - Close other applications
# - Restart Mac (frees memory)
```

---

**Solution 3: Verify services running**
```bash
# Check all services
podman ps  # Should show chromadb, n8n
curl http://localhost:11434  # Should return "Ollama is running"
curl http://localhost:8000/api/v1/heartbeat  # Should return timing

# Restart if needed
podman restart chromadb n8n
killall ollama
ollama serve &
```

---

### Problem D: Bot Responds to Itself (Loop)

**Symptoms:**
- Send one message
- Bot responds multiple times
- n8n shows many executions
- Bot talking to itself

**This shouldn't happen if configured correctly!**

**Check configuration:**

**Click "Validate & Extract Question" node:**

**Verify code includes:**
```javascript
const BOT_EMAIL = 'your-actual-bot@webex.bot';  // ← Must be YOUR bot email

if (messageData.personEmail === BOT_EMAIL) {
  throw new Error('Ignoring message from bot itself');
}
```

**If missing or wrong email:**
- Update to correct email
- Save workflow
- Test again

**You SHOULD see some "failed" executions:**
- Error: "Ignoring message from bot itself"
- This is NORMAL and prevents loops
- Don't try to "fix" these errors!

---

### Problem E: Bot Doesn't Understand @mentions

**Symptoms:**
- @mention bot in group space
- Bot responds with question including bot name
- Example: "RAG Assistant What is the uptime?"

**This is normal behavior - @mention is part of text**

**The workflow should clean this, but verify:**

**Click "Validate & Extract Question" node**

**Verify cleaning code exists:**
```javascript
// Remove @mentions
if (messageData.mentionedPeople && messageData.mentionedPeople.length > 0) {
  question = question.replace(/<spark-mention[^>]*>.*?<\/spark-mention>/gi, '');
  question = question.replace(/@\w+/g, '');
  question = question.replace(/<[^>]*>/g, '');
}
question = question.trim();
```

**If missing:**
- Add this code
- Save workflow
- Test again

---

### Problem F: Tunnel URL Changed, Bot Stopped

**Symptoms:**
- Bot was working
- You restarted tunnel
- Now bot doesn't respond
- No errors visible

**This is NORMAL with free tier!**

**Solution:**
```bash
# 1. Note new tunnel URL
cat ~/cisco-rag-demo/tunnel-url.txt
# If wrong, update it:
echo "https://new-url.localhost.run" > ~/cisco-rag-demo/tunnel-url.txt

# 2. Update webhook
~/cisco-rag-demo/update-webhook.sh

# 3. Test
# Send message in Webex
# Should work again
```

**Prevention:**
- Consider paid tunnel (static URL)
- ngrok Pro: $10/month
- Or deploy to cloud (no tunnel needed)

---

### Problem G: "Invalid JSON in Response Body"

**Symptoms:**
- n8n execution log shows this error
- Workflow completes but shows warning

**Cause:**
- Webhook response mode incorrect

**Solution:**

**Click "Webex Webhook" node (Node 1)**

**Verify settings:**
```
Response Mode: responseNode  ← MUST be this
Not: lastNode
```

**If wrong:**
- Change to "responseNode"
- Save workflow
- Test again

---

### Problem H: Documents Not Found

**Symptoms:**
- Bot responds: "No documents found in ChromaDB"
- Or: Generic answers not from your documents

**Solutions:**

**Solution 1: Verify documents loaded**
```bash
cd ~/cisco-rag-demo

# Check document count
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
  python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")

curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
# Should return number > 0
```

**If count is 0:**
```bash
# Load documents
python3 load_document.py documents/your-document.md
```

---

**Solution 2: Verify collection name correct**

**In n8n workflow, click "Get Collection UUID" node**

**Verify looking for correct collection:**
```javascript
// Should search for "cisco_docs"
for (const coll of collections) {
  if (coll.name === 'cisco_docs') {  ← Check this name
    ...
  }
}
```

---

### Quick Diagnostic Commands

**Run these for quick health check:**

```bash
#!/bin/bash
echo "=== System Health Check ==="

echo "1. Containers:"
podman ps --format "{{.Names}}: {{.Status}}"

echo -e "\n2. Ollama:"
curl -s http://localhost:11434/api/tags | jq -r '.models[].name'

echo -e "\n3. ChromaDB:"
curl -s http://localhost:8000/api/v1/heartbeat

echo -e "\n4. Tunnel:"
cat ~/cisco-rag-demo/tunnel-url.txt

echo -e "\n5. Webhook:"
cat ~/cisco-rag-demo/webhook-id.txt

echo -e "\n6. Bot Email:"
cat ~/cisco-rag-demo/webex-bot-info.txt | grep "Bot Email"

echo -e "\n✅ Health check complete"
```

**Save as:** `~/cisco-rag-demo/health-check.sh`

---

### When to Seek Additional Help

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

**3. Prepare Demo Questions**
```
Create 10-15 compelling questions
Know expected answers
Practice demo flow
Time responses
Prepare backup plans
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

### For Cisco Live 2026

**You're now ready to:**
- ✅ Demonstrate local RAG system
- ✅ Show mobile access
- ✅ Explain privacy benefits
- ✅ Compare to cloud solutions
- ✅ Answer technical questions
- ✅ Live demo reliability

**Preparation tips:**
1. Practice demo 10+ times
2. Prepare compelling documents
3. Create visually interesting questions
4. Have backup Mac ready
5. Pre-load models before presentation
6. Know troubleshooting procedures

---

### Additional Resources

**Documentation:**
- WHATS_NEXT.md - Advanced features
- Comprehensive Troubleshooting Guide
- Performance Optimization Guide
- Security Best Practices

**Community:**
- Share your experience
- Help others learn
- Contribute improvements
- Report issues

---

## Document Information

**Version:** 1.0.0  
**Last Updated:** December 18, 2024  
**Part of:** Local RAG System Implementation Guides  
**Previous Guides:**
- [GUIDE_1_ENVIRONMENT_SETUP.md](GUIDE_1_ENVIRONMENT_SETUP.md)
- [GUIDE_2_RAG_SYSTEM.md](GUIDE_2_RAG_SYSTEM.md)

**This guide covered:**
- Webex bot creation and configuration
- Tunnel setup (localhost.run and ngrok)
- n8n Webex workflow (12 nodes)
- Webhook registration and management
- Complete testing procedures
- Daily operations
- Comprehensive troubleshooting
- Mobile and team collaboration

**Your complete system is now ready for enterprise use!** 🚀

---

**🎉 CONGRATULATIONS on completing the trilogy! You've built something amazing!** 🎉
