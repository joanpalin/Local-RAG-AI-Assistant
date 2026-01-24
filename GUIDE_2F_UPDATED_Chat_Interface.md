# Part 2F: n8n Chat Interface

**Time Required:** 10-15 minutes  
**What you'll build:** Interactive chat interface for querying your documents

---

## Overview

In this section, you'll create the chat interface that allows you to ask questions about your documents directly in n8n. This workflow connects all the pieces: fetching documents from ChromaDB, generating embeddings, searching for relevant content, and using the LLM to create human-friendly answers.

**What the workflow does:**
1. Receives your question through chat interface
2. Looks up your collection UUID dynamically
3. Generates an embedding for your question
4. Searches ChromaDB for relevant document chunks
5. Builds a prompt with retrieved context
6. Sends to LLM for answer generation
7. Formats and returns the response

---

## Step 1: Create Workflows Directory

Before importing the workflow, create the directory to organize your workflow files:

```bash
# Create workflows directory
mkdir -p ~/cisco-rag-demo/workflows

# Verify it was created
ls -la ~/cisco-rag-demo/
```

**Expected output:**
```
drwxr-xr-x  ... workflows
```

✓ **Checkpoint:** The workflows directory exists

---

## Step 2: Download the Workflow File

The workflow JSON file is included in this repository and has been tested to work with current n8n versions.

**Download the workflow:**

```bash
# Navigate to your project directory
cd ~/cisco-rag-demo/workflows/

# Download the workflow JSON
# (Replace with actual GitHub raw URL once repo is published)
curl -O https://raw.githubusercontent.com/joanpalin/Local-RAG-AI-Assistant/main/RAG_Chat_Interface_WORKING.json
```

**Or manually download:**
1. Go to the GitHub repository
2. Navigate to the workflows folder
3. Download `RAG_Chat_Interface_WORKING.json`
4. Save to `~/cisco-rag-demo/workflows/`

✓ **Checkpoint:** File downloaded to workflows directory

---

## Step 3: Import the Workflow into n8n

### Open n8n Interface

1. Open your browser to: **http://localhost:5678**
2. You should see the n8n interface

### Import the Workflow

1. Click the **menu icon** (☰) in the top-left corner
2. Select **"Import from file"** or **"Import workflow"**
3. Click **"Browse"** or **"Select file"**
4. Navigate to: `~/cisco-rag-demo/workflows/RAG_Chat_Interface_WORKING.json`
5. Select the file and click **"Open"**
6. The workflow will import with all 9 nodes connected

**You should see:**
- **9 nodes** arranged in a flow
- All nodes connected with arrows
- Node names:
  1. Chat Trigger
  2. Get Collection UUID
  3. Prepare Query
  4. Generate Query Embedding
  5. Prepare Search
  6. Search ChromaDB
  7. Build Prompt
  8. Generate Answer
  9. Format Response

✓ **Checkpoint:** Workflow imported successfully, all 9 nodes visible

---

## Step 4: Save and Activate the Workflow

### Save the Workflow

1. Click **"Save"** button (top right)
2. The workflow is now saved

### Activate the Workflow

1. Find the **toggle switch** near the top-right (labeled "Inactive" or "Active")
2. Click to turn it **ON** (should turn blue/green and say "Active")
3. Wait 5-10 seconds for activation to complete

**What activation does:**
- Starts the Chat Trigger webhook
- Makes the chat interface available
- Enables the workflow to receive messages

✓ **Checkpoint:** Workflow shows "Active" status

---

## Step 5: Access the Chat Interface

### Find the Chat Button

1. Look for a **"Chat"** button in the bottom-right corner of the n8n interface
2. If you don't see it immediately:
   - **Refresh** your browser page
   - Wait 10 seconds
   - Check that workflow is Active

### Open the Chat

1. Click the **"Chat"** button
2. A chat panel opens on the right side of the screen
3. You should see:
   - Chat input box at the bottom
   - Session ID at the top
   - Empty conversation area

✓ **Checkpoint:** Chat interface is open and ready

---

## Step 6: Test Your Chat Interface

### Ask Your First Question

In the chat input box, type:
```
What was the network uptime in Q3?
```

Press **Enter** or click **Send**

### What Should Happen

**Processing (5-10 seconds):**
- You'll see a "thinking" or loading indicator
- The workflow executes through all 9 nodes
- Progress shows in the main n8n interface

**Response:**
The AI should respond with specific information from your document, for example:
```
The network uptime for Q3 2024 was 99.94%. This figure was mentioned in 
both Source 1 and Source 2, which reported that the network demonstrated 
high reliability during this quarter. According to these documents, the 
mean time to repair (MTTR) was 2.4 hours.
```

**Note:** Your response will vary based on the documents you loaded earlier.

✓ **Checkpoint:** Received an accurate answer citing your documents

---

## Step 7: Try Additional Questions

Test the system with various question types:

**Specific factual questions:**
```
What is the budget for datacenter upgrades?
How many switches are deployed in Boston?
What were the Q3 incidents?
```

**Analytical questions:**
```
Summarize the main network issues
What improvements are recommended?
Compare Q2 and Q3 performance
```

**Negative tests (to verify error handling):**
```
What is the weather in Paris?
(Should respond that information isn't in documents)
```

✓ **Checkpoint:** Chat handles various question types appropriately

---

## Understanding the Workflow

### The 9-Node Architecture

**Visual flow:**
```
Chat Trigger → Get Collection UUID → Prepare Query → Generate Query Embedding → 
Prepare Search → Search ChromaDB → Build Prompt → Generate Answer → Format Response
```

**What each node does:**

1. **Chat Trigger** - Receives your question from the chat interface
2. **Get Collection UUID** - Fetches list of collections from ChromaDB
3. **Prepare Query** - Extracts UUID, prepares embedding request
4. **Generate Query Embedding** - Converts question to vector (768 numbers)
5. **Prepare Search** - Formats search request with embedding
6. **Search ChromaDB** - Finds 3 most relevant document chunks
7. **Build Prompt** - Combines chunks with question into LLM prompt
8. **Generate Answer** - Sends to llama3.2:3b for answer generation
9. **Format Response** - Cleans and formats answer for chat display

### Key Features

**Dynamic collection lookup:**
- Workflow automatically finds your collection UUID
- No hardcoding required
- Works even if collection is recreated

**Semantic search:**
- Converts question to embedding vector
- Finds relevant chunks by meaning, not keywords
- Returns top 3 most similar passages

**Context-aware answers:**
- LLM reads retrieved document chunks
- Answers specifically based on your documents
- Cites sources when available

**Error handling:**
- Validates collection exists
- Checks for documents in ChromaDB
- Provides clear error messages

---

## Troubleshooting

### Chat Button Not Appearing

**Cause:** Workflow not activated or needs time to start

**Solutions:**
1. Verify workflow is **Active** (toggle in top-right)
2. **Refresh** browser page completely
3. Wait 15-20 seconds after activation
4. Check browser console for errors (F12)

---

### Error: "Collection 'cisco_docs' not found"

**Cause:** ChromaDB doesn't have documents loaded

**Solution:**
Return to Part 2E and load documents:
```bash
python3 ~/cisco-rag-demo/scripts/load_document.py ~/path/to/document.txt
```

Verify documents exist:
```bash
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
```

---

### Response Says "I don't have information"

**Possible causes:**
1. Question is about content not in your documents
2. Documents don't contain relevant information
3. Question phrasing doesn't match document language

**Solutions:**
- Rephrase question to match document terminology
- Verify document content covers the topic
- Try more specific questions
- Check that documents loaded correctly

---

### Very Slow Responses (>30 seconds)

**Cause:** System resources constrained

**Solutions:**

**Check RAM usage:**
```bash
# macOS: Open Activity Monitor, check Memory Pressure
# Should be green, not yellow/red
```

**Pre-load models:**
```bash
# Keeps models in RAM
ollama run llama3.2:3b "test"
# Press Ctrl+D to exit
```

**Reduce load:**
- Close unnecessary applications
- Restart system to free memory
- Consider smaller documents

---

### Workflow Shows Errors on Nodes

**If you see red error indicators on nodes:**

1. **Click the node** showing error
2. **Read the error message** in the right panel
3. **Common fixes:**
   - Ollama not running: Start with `ollama serve`
   - ChromaDB connection: Verify container running with `podman ps`
   - Timeout: Increase timeout in node Options

4. **Check logs:**
   ```bash
   # Check if services are running
   ~/cisco-rag-demo/system-check.sh
   
   # View container logs
   podman logs chromadb
   podman logs n8n
   ```

---

## Workflow Customization (Optional)

Once the workflow is working, you can customize it:

### Adjust Number of Results

In **"Prepare Search"** Code node, change:
```javascript
n_results: 3  // Change to 5 or 10 for more context
```

**Trade-off:** More results = more context but slower processing

### Modify LLM Parameters

In **"Build Prompt"** Code node, adjust:
```javascript
temperature: 0.7,  // Lower (0.3) = more focused, Higher (0.9) = more creative
top_p: 0.9         // Lower = more deterministic
```

### Change System Prompt

In **"Build Prompt"** Code node, modify the `systemPrompt`:
```javascript
const systemPrompt = `You are a helpful AI assistant analyzing Cisco network assessment documents. 
Provide accurate, concise answers based on the context provided. 
If the context doesn't contain enough information to answer fully, say so.`;
```

**Customize for your use case:**
- Medical records: "You are a medical assistant..."
- Legal documents: "You are a legal research assistant..."
- Custom format: "Answer in bullet points..."

---

## Success Checklist

Before moving to Part 3 (Webex Integration), verify:

- [ ] Workflows directory created
- [ ] Workflow JSON downloaded
- [ ] Workflow imported into n8n
- [ ] Workflow saved and activated
- [ ] Chat button appears in interface
- [ ] Chat responds to test questions
- [ ] Answers cite specific document content
- [ ] Multiple questions work correctly
- [ ] Error handling works (tested with irrelevant question)

**✓✓✓ If all boxes checked: Your chat interface is WORKING!**

---

## What You've Accomplished

🎉 **Congratulations!** You now have:

✅ **Complete RAG System:**
- Document upload (Python & n8n)
- Vector storage in ChromaDB
- Semantic search capability
- LLM-powered answers

✅ **Interactive Interface:**
- Chat-based interaction
- Real-time responses
- Context-aware answers
- Error handling

✅ **Private & Local:**
- No data leaves your machine
- Runs completely offline
- HIPAA/SOC2 compatible architecture
- Full control over data

---

## Next Steps

**You're now ready for Part 3: Webex Integration**

In the next guide, you'll:
- Create a Webex bot
- Connect your RAG system to Webex Teams
- Enable team members to query documents via chat
- Set up secure webhooks with localhost.run

**Time required:** ~45 minutes  
**Prerequisites:** Completed Parts 1 and 2 (✓)

**⚡ Continue to Part 3: [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)**

---

## Additional Resources

**Troubleshooting:**
- See main [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for comprehensive solutions
- Review [LESSONS_LEARNED_Troubleshooting_Chronicle.md](LESSONS_LEARNED_Troubleshooting_Chronicle.md) for known issues

**n8n Documentation:**
- [n8n Chat Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.chattrigger/)
- [n8n HTTP Request](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)
- [n8n Code Node](https://docs.n8n.io/code-examples/overview/)

---

**End of Part 2F: n8n Chat Interface**
