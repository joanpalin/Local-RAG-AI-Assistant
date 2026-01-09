# Documentation Standards & Style Guide

## Purpose
This document defines the writing standards, formatting conventions, and style guidelines for all documentation in this RAG system repository. Following these standards ensures consistency and makes the documentation accessible to IT engineers regardless of their experience with AI, coding, or container technologies.

---

## Who This Documentation Is For

**Primary Audience:** IT engineers with networking/infrastructure experience but **NO** background in:
- Artificial Intelligence or Machine Learning
- Python programming or scripting
- Command-line interfaces beyond basic navigation
- Container technologies (Docker/Podman)
- n8n workflow automation
- RAG (Retrieval-Augmented Generation) systems

**Writing Philosophy:** If you wouldn't say it to a colleague who's great at networking but never touched Python, don't write it in our docs.

---

## 1. Language & Tone Guidelines

### Core Principles

**DO:**
- Use analogies to familiar IT concepts (routers, switches, databases, networks)
- Define every technical term on first use
- Explain the "why" before the "how"
- Write like you're helping a colleague over coffee, not writing a textbook
- Assume intelligence but not prior knowledge
- Use active voice ("You'll install..." not "Installation will be performed...")

**DON'T:**
- Assume knowledge of programming concepts
- Use jargon without explanation
- Say "simply" or "just" (it's not simple if they're reading docs)
- Skip steps because they seem obvious
- Write in passive voice
- Rush through explanations

### Example Comparisons

#### âŒ BAD: Too Technical
```
Deploy the Ollama container with the nomic-embed-text model to generate vector embeddings.
```

#### âœ… GOOD: Beginner-Friendly
```
We'll start a virtual computer (called a container) that runs Ollama. Think of Ollama 
as a translator that converts your documents into a special format (numbers) that 
computers can compare and search through quickly. This is similar to how a network 
device creates a routing table - it's organizing information in a way that makes 
lookups fast.
```

#### âŒ BAD: Assumes Knowledge
```
Use the REST API endpoint to query the RAG system.
```

#### âœ… GOOD: Explains Concept
```
You'll use a REST API (a way for programs to talk to each other over the network, 
like how two routers exchange routing information) to send questions to your RAG 
system and get answers back.
```

---

## 2. Command Documentation Format

Every command must follow this structure:

```markdown
### What This Command Does
[Plain English explanation of the purpose]

```bash
# The actual command
command --with --flags argument
```

**What each part means:**
- `command` - [explanation]
- `--with` - [explanation]
- `--flags` - [explanation]
- `argument` - [explanation]

**Expected output:**
```
[Exact text you should see]
```

**What this output means:**
[Interpretation of the output]

**Ã¢Å“â€¦ Success looks like:**
[What confirms it worked]

**âŒ Common errors:**
1. **Error:** `specific error message`
   - **Why:** [Plain English cause]
   - **Fix:** [Step-by-step solution]
```

### Real Example

#### What This Command Does
This checks if Ollama is running and ready to process documents.

```bash
# Check Ollama's status
curl http://localhost:11434/api/tags
```

**What each part means:**
- `curl` - A tool that fetches information from web addresses (like using a browser, but for the command line)
- `http://localhost:11434` - The "address" where Ollama is listening on your computer
- `/api/tags` - Asks Ollama "what AI models do you have installed?"

**Expected output:**
```json
{"models":[{"name":"llama3.2:3b","modified_at":"2024-11-20T10:30:00Z"}]}
```

**What this output means:**
Ollama is running and has the llama3.2:3b model installed (the AI "brain" that will answer questions).

**âœ… Success looks like:**
You see a list of models in curly braces. As long as you see `"models":[` you're good.

**âŒ Common errors:**
1. **Error:** `curl: (7) Failed to connect to localhost port 11434`
   - **Why:** Ollama isn't running yet
   - **Fix:** Start Ollama first (see Step 2: Installing Ollama)

---

## 3. Code Explanation Standards

### Before Showing Code

Always provide context:
1. **What** this code will do (outcome)
2. **Why** we need it (problem it solves)
3. **How** it fits into the bigger picture

### When Showing Code

**For short commands (1-3 lines):**
- Explain after the command
- Show expected output
- Note what success looks like

**For longer scripts (4+ lines):**
- Explain the overall purpose first
- Break down into logical sections
- Comment each section in plain English
- Show what each section produces

### Example: Script Explanation

#### âŒ BAD: Just Code Dumped
```python
import chromadb
client = chromadb.HttpClient(host="localhost", port=8000)
collections = client.list_collections()
print(collections)
```

#### âœ… GOOD: Properly Explained

**What this script does:**
This checks what document collections ChromaDB has stored. Think of it like running a "show interfaces" command - you're asking the system "what do you have?"

**Why we need this:**
Before we can add documents or ask questions, we need to know if our document storage is set up correctly and what collections (folders, essentially) exist.

```python
# Import the ChromaDB tools
import chromadb

# Connect to ChromaDB (like logging into a switch)
client = chromadb.HttpClient(host="localhost", port=8000)

# Ask "what collections exist?" (like "show vlans")
collections = client.list_collections()

# Display the results
print(collections)
```

**Expected output:**
```
[Collection(name=rag_demo)]
```

**What this means:**
You have one collection called "rag_demo" - this is where your documents will be stored.

---

## 4. Visual Indicators

Use these consistently throughout documentation:

### âœ… Success Checkpoints
Use when the user should verify something worked.
```markdown
âœ… **Checkpoint:** You should see "Container started" in the output
```

### âš ï¸ Important Warnings
Use for critical information that could cause problems if missed.
```markdown
âš ï¸ **Warning:** Do not close this terminal window while the container is running
```

### ðŸ’¡ Helpful Tips
Use for optimization, shortcuts, or "good to know" information.
```markdown
ðŸ’¡ **Tip:** You can press Ctrl+C to stop the container gracefully
```

### ðŸ” Verification Steps
Use when the user needs to check something.
```markdown
ðŸ” **Verify:** Run `podman ps` to confirm the container is running
```

### âŒ What NOT to Do
Use to prevent common mistakes.
```markdown
âŒ **Don't:** Don't use `sudo` with Podman commands - it's not needed and can cause permission issues
```

### ðŸ“‹ Prerequisites
Use at the start of sections that require prior steps.
```markdown
ðŸ“‹ **Prerequisites:** You must have completed Step 2 (Installing Ollama) before continuing
```

### ðŸŽ¯ Quick Reference
Use for summaries or key takeaways.
```markdown
ðŸŽ¯ **Key Point:** ChromaDB stores your documents as vectors (numbers), not as text files
```

---

## 5. Section Structure Template

Use this structure for every major step:

```markdown
## Step X: [Clear Action Title Using Verbs]

**What you'll do:** [One sentence summary in plain English]
**Why this matters:** [What this enables or solves]
**Time required:** [X-Y minutes, including reading]
**Prerequisites:** [What must be completed first]

---

### Background: Understanding [Concept]

[2-3 paragraphs explaining the concept using IT analogies]

Think of this like [familiar IT concept]. Just as [analogy explanation], 
this component [how it works].

**In practical terms:** [What this means for the user's system]

---

### What You'll Accomplish

By the end of this step, you will have:
- âœ… [Specific outcome 1]
- âœ… [Specific outcome 2]
- âœ… [Specific outcome 3]

---

### Instructions

#### Part A: [Substep Name]

1. **[Action in plain English]**
   
   ```bash
   # Command with explanation
   command here
   ```
   
   **What this does:** [Explanation]
   
   **Expected output:**
   ```
   [Output text]
   ```

2. **[Next action]**
   
   [Continue pattern]

#### Part B: [Next Substep]

[Continue pattern]

---

### Verification

âœ… **How to confirm this step worked:**

1. **Check [specific thing]:**
   ```bash
   verification command
   ```
   You should see: `[expected result]`

2. **Confirm [another thing]:**
   ```bash
   another verification
   ```
   Success looks like: `[expected output]`

ðŸŽ¯ **Success criteria:** [Summary of what "done" means]

---

### Troubleshooting This Step

> **Note:** These solutions are specific to Step X. For general issues, see the main Troubleshooting Guide.

#### Issue 1: [Specific Error Message]

**Symptoms:**
- [What the user sees]
- [Related problems]

**Cause:**
[Plain English explanation of why this happens]

**Solution:**
1. [Step to fix]
2. [Next step]
3. [Verification step]

**Verification:**
âœ… You'll know it's fixed when: [specific outcome]

---

#### Issue 2: [Another Common Problem]

[Same structure as Issue 1]

---

### Next Steps

You've now completed [what was accomplished]. 

**Where to go next:**
- âž¡ï¸ Continue to [Step X+1] to [what comes next]
- ðŸ”™ If something didn't work, review [related section]
- ðŸ’¡ Optional: [Enhancement or alternative path]

---
```

---

## 6. Beginner-Friendly Technical Explanations

### Core Concept Definitions

When introducing technical terms, always provide the beginner explanation first, then the technical definition.

#### Container â†’ Virtual Computer Running Inside Your Mac

**Beginner explanation:**
A container is like a virtual computer running inside your Mac. Imagine you have a completely separate computer with its own operating system and programs, but it's actually just software running on your Mac. It keeps everything self-contained so it won't interfere with your other programs. Think of it like a VLAN - it creates isolation while sharing the same physical hardware.

**Technical definition:** 
Containers are lightweight, standalone executable packages that include everything needed to run a piece of software: code, runtime, libraries, and settings. They provide OS-level virtualization without the overhead of full virtual machines.

**Why we use it:**
Containers let us run ChromaDB, n8n, and other tools without installing them directly on your Mac, which means they won't conflict with your other software and can be easily removed later.

---

#### Vector Database â†’ Specialized Search Engine for Finding Similar Documents

**Beginner explanation:**
A vector database stores your documents in a special numerical format that computers can compare mathematically. Think of it like this: instead of storing "The router has 10 interfaces," it stores something like [0.234, 0.891, 0.432...] - a list of numbers that represents the *meaning* of that sentence. 

When you ask a question, your question also gets converted to numbers, and the database finds documents whose numbers are "close" to your question's numbers. It's similar to how routers use metrics to find the best path - higher numbers = better match.

**Analogy:**
If traditional databases are like looking up exact phone numbers in a phonebook, vector databases are like asking "who in this phonebook lives near the hospital?" - you're searching by similarity and meaning, not exact matches.

**Technical definition:**
Vector databases store high-dimensional numerical representations (embeddings) of data and enable efficient similarity search using distance metrics like cosine similarity or Euclidean distance.

**Why we use it:**
ChromaDB (our vector database) lets the AI find relevant parts of your documents even when you ask questions using different words than what's in the document.

---

#### Embedding â†’ Converting Text into Numbers the AI Can Understand

**Beginner explanation:**
An embedding is what you get when you convert text (like a sentence or document) into a list of numbers. These numbers capture the *meaning* of the text in a way computers can work with mathematically.

Think of it like GPS coordinates: "123 Main Street" gets converted to latitude/longitude numbers (40.7128, -74.0060). Both represent the same place, but the numbers let computers calculate distances and find nearby locations. Embeddings do the same thing for text meaning instead of physical location.

**Example:**
- Original text: "The router crashed at 3am"
- Embedding: [0.234, 0.891, 0.432, ..., 0.123] (actually 768 numbers!)
- Similar text: "The network device failed overnight"
- Its embedding: [0.241, 0.885, 0.428, ..., 0.119] (very close numbers!)

**Technical definition:**
Embeddings are dense vector representations of text generated by machine learning models, where semantically similar text produces similar vectors in high-dimensional space.

**Why we use it:**
Embeddings let the AI understand that "router crashed" and "device failed" mean similar things, even though they use different words.

---

#### RAG (Retrieval-Augmented Generation) â†’ Smart AI That Reads Your Documents Before Answering

**Beginner explanation:**
RAG is a two-step process that makes AI answers more accurate and relevant to YOUR documents:

**Step 1 - Retrieval:** When you ask a question, the system searches your documents and finds the most relevant sections (like searching a knowledge base).

**Step 2 - Generation:** The AI reads those relevant sections and uses them to write an answer, rather than just making up an answer from its training.

**Analogy:**
Traditional AI is like asking someone a question when they only have their memory to rely on - they might forget details or make mistakes.

RAG is like asking someone a question when they have all your documentation open in front of them - they can reference specific pages and quote exact information.

**Without RAG:**
User: "What's the budget for our network upgrade?"
AI: "I don't have specific information about your network upgrade budget."

**With RAG:**
User: "What's the budget for our network upgrade?"
AI: "According to the Q3 Network Assessment document, the approved budget is $250,000, with $175,000 allocated for hardware and $75,000 for professional services."

**Technical definition:**
RAG combines information retrieval with text generation by first querying a vector database for relevant context, then providing that context to a language model to generate informed responses grounded in specific documents.

**Why we use it:**
RAG lets the AI answer questions using YOUR company's documents, network assessments, and technical reports - not just generic information from its training.

---

#### Webhook â†’ A Way for Programs to Notify Each Other Instantly

**Beginner explanation:**
A webhook is like a doorbell for programs. Instead of constantly checking "is there new information?" (polling), the program just waits. When something happens, the other program "rings the doorbell" (sends a webhook) to say "hey, you have a new message!"

**Analogy from Networking:**
Think of polling as continuously pinging a server every second to check status (wasteful). A webhook is like SNMP traps - the device only sends a message when something important happens.

**Real example in this project:**
When you send a message to the Webex bot, Cisco's servers immediately send a webhook to your n8n workflow saying "someone sent a message." Your workflow wakes up, reads the message, and responds. You're not constantly checking Webex every second asking "any new messages?"

**Traditional way (polling):**
```
Every 5 seconds: "Any new messages?" "Nope" "Any new messages?" "Nope" 
"Any new messages?" "Yes! Here's one." [Processes message]
```

**Webhook way:**
```
[Sitting idle] ... [Message arrives] ... [Cisco sends webhook] ... 
"New message!" [Processes message immediately]
```

**Technical definition:**
Webhooks are HTTP callbacks that enable real-time event-driven communication between services by pushing data to a specified URL when specific events occur, rather than requiring continuous polling.

**Why we use it:**
n8n uses webhooks to receive messages from Webex instantly, making the bot responsive and efficient without wasting resources constantly checking for new messages.

---

## 7. Acronym and Technical Term Handling

### First Use - Always Define and Explain

When introducing an acronym or technical term for the first time:

```markdown
We'll use RAG (Retrieval-Augmented Generation), which is a method that makes AI 
answers more accurate by having the AI read your documents before responding. 
Think of it as giving the AI a reference book before asking it questions.
```

**Format:**
1. Use the acronym with full name in parentheses
2. Provide plain English explanation
3. Add analogy or context
4. Use the acronym freely afterward

### Subsequent Uses - Use Freely

After the first definition, use the acronym without re-explaining:

```markdown
The RAG system will search your documents for relevant passages. When RAG finds 
matching content, it provides that to the AI along with your question.
```

### Create a Glossary Section

Every major document should include a glossary at the end:

```markdown
## Glossary

**API (Application Programming Interface):** A way for programs to communicate with each other. Like a menu at a restaurant - it lists what you can request and how to request it.

**ChromaDB:** The vector database we use to store documents as numerical embeddings. It's the "search engine" part of the RAG system.

**Container:** A virtual environment that runs programs in isolation. Like running a separate computer inside your Mac.

**Embedding:** Converting text into numbers that capture meaning. Allows computers to understand that "router failed" and "device crashed" mean similar things.

**LLM (Large Language Model):** The AI "brain" that generates text responses. In this project, we use llama3.2:3b.

**n8n:** A visual workflow automation tool. Instead of writing code, you connect boxes that represent different actions.

**Ollama:** The software that runs AI models locally on your Mac. It's like having ChatGPT running privately on your computer.

**Podman:** Container management software (alternative to Docker). It creates and runs isolated virtual environments.

**RAG (Retrieval-Augmented Generation):** A method where AI searches your documents for relevant information before generating an answer.

**Vector:** A list of numbers that represents meaning. Documents and questions are converted to vectors so they can be compared mathematically.

**Webhook:** A way for one program to notify another instantly when something happens, like a doorbell for software.
```

### Terms That Need Special Attention

Some terms are particularly confusing and need extra care:

**Vector** - Often confused with array or list. Emphasize it captures *meaning*, not just stores data.

**Embedding** - Often confused with "embedding a video." Emphasize it's a mathematical conversion.

**Model** - In AI context, not a 3D model or conceptual model. Emphasize it's the "brain" or "program" that generates responses.

**Endpoint** - Often confused with network endpoint. Emphasize it's a URL where programs can send requests.

**Local** - Be clear whether you mean "on this Mac" vs "in this network" vs "not cloud-based."

---

## 8. Error Message Documentation

Every documented error must follow this structure:

### Error Template

```markdown
### Error: [Exact Error Message in Code Format]

```
[Complete error message as it appears]
```

**What you're seeing:**
[Plain English description of what this error means]

**Why this happens:**
[Root cause explanation]

**Common causes:**
1. [Specific scenario 1]
2. [Specific scenario 2]
3. [Specific scenario 3]

**How to fix it:**

#### Step 1: [First thing to check]
```bash
# Command to check
command here
```

Expected result: `[what success looks like]`

If you see: `[problem output]`
Then: [what to do]

#### Step 2: [Next thing to check]
[Continue pattern]

#### Step 3: [Solution]
```bash
# Command to fix
fix command here
```

**Verification:**
âœ… Run this to confirm it's fixed:
```bash
verification command
```

You should see: `[success output]`

**Prevention:**
ðŸ’¡ To avoid this in the future: [preventive measure]
```

### Real Example

```markdown
### Error: `curl: (7) Failed to connect to localhost port 11434`

```
curl: (7) Failed to connect to localhost port 11434: Connection refused
```

**What you're seeing:**
Your computer tried to communicate with Ollama on port 11434, but nothing is listening at that address. It's like calling a phone number and getting "number not in service."

**Why this happens:**
Ollama isn't running. Either it was never started, or it crashed/stopped.

**Common causes:**
1. Ollama was never started after installation
2. Ollama was running but you closed the terminal window
3. Ollama crashed (rare)

**How to fix it:**

#### Step 1: Check if Ollama is actually running

```bash
# Check for Ollama process
ps aux | grep ollama
```

Expected result: You should see a line with `/usr/local/bin/ollama serve`

If you see: `grep ollama` (only your grep command), then Ollama isn't running.

#### Step 2: Start Ollama

Open a new terminal window and run:
```bash
# Start Ollama server
ollama serve
```

You should see:
```
Ollama server is running on http://localhost:11434
```

**Important:** Keep this terminal window open! Closing it stops Ollama.

#### Step 3: Verify it's working

In your original terminal (not the one running `ollama serve`), run:
```bash
# Test Ollama connection
curl http://localhost:11434/api/tags
```

**Verification:**
âœ… You should see JSON output listing your models:
```json
{"models":[{"name":"llama3.2:3b",...}]}
```

**Prevention:**
ðŸ’¡ To avoid this in the future:
- Keep the `ollama serve` terminal window open while working
- Or, set up Ollama as a background service (see Advanced Setup guide)
- Add a quick check to your startup process: `curl http://localhost:11434/api/tags`
```

---

## 9. Time Estimates

Provide realistic time estimates that account for beginners taking longer.

### Format

```markdown
**Time required:** 
- Reading: 5 minutes
- Execution: 10-15 minutes  
- Verification: 5 minutes
- **Total: 20-25 minutes**
```

### Estimation Guidelines

**Reading time:**
- Simple concept: 2-3 minutes per section
- Complex concept: 5-10 minutes per section
- Code-heavy section: 3-5 minutes per code block

**Execution time:**
- Simple command: 1-2 minutes (includes typing, running, checking output)
- Installing software: 5-15 minutes (depends on download speed)
- Creating/modifying files: 3-5 minutes per file
- Testing/verification: 2-5 minutes per test

**Troubleshooting buffer:**
- Add 50% to total time estimate
- Example: 20 minute task = estimate 30 minutes

### Time Estimate Examples

```markdown
## Step 3: Installing ChromaDB Container

**Time required:**
- Reading & understanding: 8 minutes
- Downloading container image: 3-5 minutes (depends on internet speed)
- Starting container: 2 minutes
- Verification: 3 minutes
- **Total: 16-18 minutes**
- **With troubleshooting buffer: 25-30 minutes**
```

### Multi-Part Sections

For sections with optional parts:

```markdown
**Time required:**
- Core setup (required): 15-20 minutes
- Optional: Webex integration: +30 minutes
- Optional: REST API testing: +10 minutes
- **Minimum total: 15-20 minutes**
- **Complete setup: 55-60 minutes**
```

---

## 10. Cross-Referencing Standards

### When to Link vs Explain

**Link to another document when:**
- The concept requires >3 paragraphs to explain
- It's covered comprehensively elsewhere
- The reader might want to skip it now and return later

**Explain in-line when:**
- The concept can be explained in 1-2 sentences
- It's essential to understanding current step
- It's a quick definition or reminder

### Link Format

Use descriptive link text, never "click here":

#### âŒ BAD
```markdown
To install Ollama, click [here](install-ollama.md).
```

#### âœ… GOOD
```markdown
See the [Ollama Installation Guide](install-ollama.md) for detailed setup instructions.
```

### Internal Document References

```markdown
For more details on [specific topic], see:
- **[Document Name](path/to/document.md#section-anchor)** - [One sentence about what they'll find]

Example: For more details on troubleshooting ChromaDB connection issues, see:
- **[ChromaDB Troubleshooting Guide](troubleshooting.md#chromadb-connection-errors)** - Solutions for common connection and permission problems
```

### Section References Within Same Document

```markdown
If you haven't completed this yet, see [Step 2: Installing Ollama](#step-2-installing-ollama) above.
```

### Avoiding Circular Dependencies

**Problem:** Doc A says "see Doc B first" â†’ Doc B says "see Doc A first"

**Solution:** Create a clear hierarchy:

```markdown
## Documentation Flow

This repository has a recommended reading order:

1. **[START_HERE.md](START_HERE.md)** - Overview and prerequisites
   â”œâ”€> 2. **[INSTALLATION.md](INSTALLATION.md)** - Installing all components
   â”‚   â”œâ”€> 3. **[RAG_SETUP.md](RAG_SETUP.md)** - Configuring RAG system
   â”‚   â””â”€> 3. **[N8N_SETUP.md](N8N_SETUP.md)** - Setting up workflows
   â””â”€> 4. **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** - Reference as needed

**Optional advanced topics:**
- **[WEBEX_INTEGRATION.md](WEBEX_INTEGRATION.md)** - Requires completing steps 1-3
- **[API_REFERENCE.md](API_REFERENCE.md)** - For programmatic access
```

### Reference Format Standards

```markdown
### Prerequisites

ðŸ“‹ Before starting this section, ensure you have completed:
- âœ… [Step 1: Environment Setup](setup.md) - Required
- âœ… [Step 2: Installing Ollama](ollama.md) - Required
- â­• [Optional: Custom Model Setup](custom-models.md) - Optional, for advanced users

### Related Resources

ðŸ’¡ You might also find these helpful:
- **[Troubleshooting Guide](troubleshooting.md#step3-errors)** - If you encounter errors
- **[FAQ](faq.md#ollama-questions)** - Common questions about Ollama
- **[Advanced Configuration](advanced.md#ollama-optimization)** - Performance tuning

### Next Steps

âž¡ï¸ **Recommended next step:** [Step 4: Loading Documents](loading-documents.md)

ðŸ”€ **Alternative paths:**
- Want to test first? â†’ [Testing Your Setup](testing.md)
- Need to customize? â†’ [Configuration Guide](configuration.md)
```

---

## 11. Code Block Standards

### Syntax Highlighting

Always specify the language for proper syntax highlighting:

```markdown
```bash
# Shell commands
curl http://localhost:8000
```

```python
# Python code
import chromadb
client = chromadb.HttpClient()
```

```json
// JSON output
{"status": "success"}
```

```javascript
// JavaScript / n8n code
const result = items[0].json;
return result;
```
```

### Indicating User Input vs Output

```markdown
```bash
# Command (you type this)
$ ollama list

# Output (what you'll see)
NAME              SIZE      MODIFIED
llama3.2:3b      2.0 GB    2 days ago
```
```

### Long Output - Show Key Parts Only

```markdown
```bash
$ podman ps

# Output (truncated for clarity):
CONTAINER ID  IMAGE                    STATUS
a1b2c3d4e5f6  chromadb/chromadb:0.4.24 Up 2 minutes
...
```
```

### Placeholder Values

Use angle brackets for values users must replace:

```markdown
```bash
# Replace <your-bot-token> with your actual token
export WEBEX_TOKEN="<your-bot-token>"
```

**Example with actual values:**
```bash
export WEBEX_TOKEN="YjI3ZTM0YTgtNzY5MS00OWY2LWI5ODYtZjQ0N2Y4YTdmMjQ0"
```
```

---

## 12. Screenshot and Diagram Standards

### When to Use Screenshots

**DO use screenshots for:**
- Complex UI navigation (n8n workflow builder)
- Showing where to click in an interface
- Confirming expected visual output
- Demonstrating error messages in context

**DON'T use screenshots for:**
- Terminal commands (use code blocks instead)
- Text-only content (use markdown)
- Things that change frequently
- Simple concepts that can be described

### Screenshot Format

```markdown
### Configuring the Webhook Node

In n8n, you'll see the workflow builder interface:

![n8n Webhook Configuration](images/n8n-webhook-setup.png)
*Figure 1: n8n webhook node configuration showing the HTTP Method dropdown*

**What you're looking at:**
- Left panel: List of available nodes
- Center: Your workflow canvas
- Right panel: Node settings

**What to do:**
1. Click the **"+" button** in the center canvas
2. Search for "Webhook"
3. Click **"Webhook"** to add it to your workflow
```

### Diagram Requirements

Keep diagrams simple and focused:

```markdown
## System Architecture

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   You       â”‚
â”‚  (User)     â”‚
â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜
       â”‚ Question
       â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚     RAG System                  â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚  â”‚ ChromaDB â”‚â—„â”€â–ºâ”‚  Ollama    â”‚ â”‚
â”‚  â”‚(Storage) â”‚   â”‚(AI Brain)  â”‚ â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                  â”‚ Answer
                  â–¼
             â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
             â”‚ Responseâ”‚
             â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```
*Figure 2: Information flow in the RAG system*
```

---

## 13. Version and Update Standards

### Documenting Version-Specific Information

When software versions matter:

```markdown
## Compatibility

This guide was written and tested with:
- **macOS:** 13.0 (Ventura) or later
- **Podman:** 4.9.0
- **ChromaDB:** 0.4.24
- **Ollama:** Latest (December 2024)
- **n8n:** 1.19.0

âš ï¸ **Note:** ChromaDB 0.5.0 introduced breaking API changes. This guide uses 0.4.24 specifically. Do not upgrade without consulting the migration guide.
```

### Update History

At the end of major documents:

```markdown
## Document History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0.0 | 2024-11-20 | Initial version | System |
| 1.0.1 | 2024-11-25 | Added Webex troubleshooting section | System |
| 1.1.0 | 2024-12-01 | Updated for ChromaDB 0.4.24 compatibility | System |

**Last updated:** December 18, 2024
```

---

## 14. FAQ Section Standards

Every major guide should include a FAQ section:

```markdown
## Frequently Asked Questions (FAQ)

### General Questions

**Q: Do I need to know Python to use this system?**

A: No! This system is designed for IT engineers with no programming experience. All the Python code is provided and explained. You'll copy and paste commands - no coding required.

---

**Q: Will this work on Windows?**

A: This guide is written for macOS. The concepts are the same on Windows, but the commands will be different (for example, Windows doesn't have the same terminal commands). A Windows version of this guide is planned for the future.

---

**Q: How much does this cost?**

A: Everything in this guide uses free, open-source software. There are no licensing fees or subscriptions. You do need a Mac with enough resources (8GB+ RAM recommended).

### Technical Questions

**Q: Why do we use Podman instead of Docker?**

A: Podman is similar to Docker but doesn't require root privileges and has better security. On Mac, both work fine, but Podman is easier to set up for this project.

---

**Q: What's the difference between Ollama and ChatGPT?**

A: Ollama runs AI models locally on your Mac - nothing leaves your computer. ChatGPT runs in the cloud. For sensitive company documents, local processing is often required for privacy/compliance reasons.

### Troubleshooting Questions

**Q: Why does my container keep stopping?**

A: Most commonly, this happens when you close the terminal window where the container is running. See the [Container Management Guide](container-management.md) for solutions.

---

**Q: How do I know if Ollama is working?**

A: Run this quick test:
```bash
curl http://localhost:11434/api/tags
```

If you see JSON output with model names, Ollama is working.
```

---

## 15. Testing and Validation Sections

Every major configuration should include validation steps:

```markdown
## Testing Your Setup

After completing the installation, validate that everything works:

### Test 1: Ollama Responds to Requests

**What we're testing:** Ollama can process simple requests

```bash
# Send a test question to Ollama
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Say hello in one word",
  "stream": false
}'
```

**âœ… Success looks like:**
```json
{"response":"Hello"}
```

**âŒ If it fails:** See [Ollama Troubleshooting](troubleshooting.md#ollama-not-responding)

---

### Test 2: ChromaDB Stores and Retrieves Data

**What we're testing:** ChromaDB can save and find documents

```bash
# Run the test script
python test_chromadb.py
```

**âœ… Success looks like:**
```
âœ… Collection created
âœ… Document added
âœ… Query returned 1 result
All tests passed!
```

**âŒ If it fails:** See [ChromaDB Troubleshooting](troubleshooting.md#chromadb-connection-issues)

---

### Test 3: End-to-End RAG Query

**What we're testing:** The complete system responds to questions

```bash
# Ask a question about your documents
python query_rag.py "What is the network budget?"
```

**âœ… Success looks like:**
```
Found 3 relevant passages
Answer: According to the Q3 assessment, the network upgrade budget is $250,000...
```

**âŒ If it fails:** See [RAG System Troubleshooting](troubleshooting.md#rag-not-finding-documents)

---

## âœ… Complete Test Checklist

Run through this checklist to confirm your system is fully operational:

- [ ] Ollama responds to API calls
- [ ] ChromaDB container is running (`podman ps` shows it)
- [ ] Documents are loaded (query returns results)
- [ ] RAG system generates answers
- [ ] n8n workflows execute without errors
- [ ] (Optional) Webex bot responds to messages

**All checked?** âœ… Your system is ready to use!
**Some failed?** ðŸ”§ See the [Troubleshooting Guide](troubleshooting.md)
```

---

## Checklist for Document Authors

Before submitting documentation, verify:

### Content Quality
- [ ] All technical terms defined on first use
- [ ] Analogies provided for complex concepts
- [ ] Commands explained before showing them
- [ ] Success criteria clearly stated
- [ ] Common errors anticipated and documented

### Structure
- [ ] Clear section hierarchy (## â†’ ### â†’ ####)
- [ ] Time estimates provided
- [ ] Prerequisites listed
- [ ] Verification steps included
- [ ] Next steps clearly indicated

### Formatting
- [ ] Code blocks have language specified
- [ ] Visual indicators used appropriately (âœ… âš ï¸ ðŸ’¡)
- [ ] Screenshots labeled and explained
- [ ] Links use descriptive text
- [ ] Placeholders use angle brackets `<like-this>`

### Beginner-Friendliness
- [ ] No unexplained jargon
- [ ] Active voice used ("you will" not "it shall")
- [ ] Plain English explanations
- [ ] IT analogies where helpful
- [ ] No assumptions about prior knowledge

### Completeness
- [ ] Glossary included
- [ ] FAQ section present
- [ ] Troubleshooting guide linked
- [ ] Related documents referenced
- [ ] Version information stated

---

## Examples: Good vs Bad Documentation

### Example 1: Explaining a Command

#### âŒ BAD
```markdown
Run `podman run -d -p 8000:8000 chromadb/chromadb:0.4.24`
```

**Problems:**
- No explanation
- No expected output
- No verification
- No troubleshooting

#### âœ… GOOD
```markdown
### Start the ChromaDB Container

**What this does:** Launches ChromaDB in a virtual environment on your Mac

```bash
# Start ChromaDB container
podman run -d \
  -p 8000:8000 \
  chromadb/chromadb:0.4.24
```

**What each part means:**
- `podman run` - Start a new container
- `-d` - Run in background (detached mode)
- `-p 8000:8000` - Make it accessible on port 8000
- `chromadb/chromadb:0.4.24` - Which program to run (ChromaDB version 0.4.24)

**Expected output:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```
This long string is the container ID - think of it like a MAC address for the container.

**âœ… Verify it's running:**
```bash
podman ps
```
You should see `chromadb/chromadb` in the IMAGE column.

**âŒ Common error:**
If you see `port 8000 already in use`, another program is using that port. See [Port Conflicts](troubleshooting.md#port-conflicts).
```

---

### Example 2: Explaining a Concept

#### âŒ BAD
```markdown
ChromaDB uses vector embeddings to enable semantic search over document collections through cosine similarity metrics.
```

**Problems:**
- Pure jargon
- No analogy
- No practical explanation
- Assumes technical knowledge

#### âœ… GOOD
```markdown
### How ChromaDB Finds Relevant Documents

ChromaDB is like a specialized search engine for finding documents that match the *meaning* of your question, not just matching keywords.

**Here's how it works:**

1. **Converting to numbers:** When you add a document, ChromaDB converts every sentence into a list of numbers (called vectors). These numbers capture the meaning of the sentence. Think of it like GPS coordinates - different representation, same information.

2. **Searching by meaning:** When you ask a question, it also gets converted to numbers. ChromaDB then finds documents whose numbers are "close" to your question's numbers.

**Example:**
- You ask: "What's our upgrade budget?"
- ChromaDB finds: "The network refresh costs $250,000"
- Even though the words are different, the *meaning* is similar, so ChromaDB finds it

**Why this matters:**
Traditional search requires exact keyword matches. ChromaDB finds relevant information even when different words are used. It's like how routers use metrics to find the best path - ChromaDB uses number comparison to find the best answer.

**Technical term:** This process is called "semantic search using vector embeddings" - but all you need to know is it lets the AI find relevant information even when you phrase things differently than the document.
```

---

## Conclusion

These standards ensure that anyone with IT networking experience can successfully set up and use this RAG system, regardless of their experience with AI, Python, or containers.

**Key principles to remember:**
1. Explain in plain English first, technical terms second
2. Use IT analogies (routers, switches, VLANs, etc.)
3. Show complete examples with expected output
4. Anticipate and document common errors
5. Verify every step with clear success criteria

**When in doubt:**
- Would you say this to a network engineer friend who's never coded?
- Can they follow these steps even if they don't understand the underlying technology?
- Have you given them a way to verify success?
- Have you documented the common ways this can fail?

If yes to all four, your documentation meets our standards.