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

#### ✗ BAD: Too Technical
```
Deploy the Ollama container with the nomic-embed-text model to generate vector embeddings.
```

#### ✓ GOOD: Beginner-Friendly
```
We'll start a virtual computer (called a container) that runs Ollama. Think of Ollama 
as a translator that converts your documents into a special format (numbers) that 
computers can compare and search through quickly. This is similar to how a network 
device creates a routing table - it's organizing information in a way that makes 
lookups fast.
```

#### ✗ BAD: Assumes Knowledge
```
Use the REST API endpoint to query the RAG system.
```

#### ✓ GOOD: Explains Concept
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

**✓ Success looks like:**
[What confirms it worked]

**✗ Common errors:**
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

**✓ Success looks like:**
You see a list of models in curly braces. As long as you see `"models":[` you're good.

**✗ Common errors:**
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

#### ✗ BAD: Just Code Dumped
```python
import chromadb
client = chromadb.HttpClient(host="localhost", port=8000)
collections = client.list_collections()
print(collections)
```

#### ✓ GOOD: Properly Explained

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

### ✓ Success Checkpoints
Use when the user should verify something worked.
```markdown
✓ **Checkpoint:** You should see "Container started" in the output
```

### ⚠️ Important Warnings
Use for critical information that could cause problems if missed.
```markdown
⚠️ **Warning:** Do not close this terminal window while the container is running
```

### 💡 Helpful Tips
Use for optimization, shortcuts, or "good to know" information.
```markdown
💡 **Tip:** You can press Ctrl+C to stop the container gracefully
```

### 🔍 Verification Steps
Use when the user needs to check something.
```markdown
🔍 **Verify:** Run `podman ps` to confirm the container is running
```

### ✗ What NOT to Do
Use to prevent common mistakes.
```markdown
✗ **Don't:** Don't use `sudo` with Podman commands - it's not needed and can cause permission issues
```

### 📋 Prerequisites
Use at the start of sections that require prior steps.
```markdown
📋 **Prerequisites:** You must have completed Step 2 (Installing Ollama) before continuing
```

### 🎯 Quick Reference
Use for summaries or key takeaways.
```markdown
🎯 **Key Point:** ChromaDB stores your documents as vectors (numbers), not as text files
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
- ✓ [Specific outcome 1]
- ✓ [Specific outcome 2]
- ✓ [Specific outcome 3]

---

### Implementation Steps

#### Step 1: [First Action]

**Action:** [What to do]

```bash
# Command to run
command here
```

**Expected result:**
```
Output you should see
```

**✓ Verification:**
[How to confirm this step worked]

---

#### Step 2: [Second Action]

[Continue pattern for each sub-step]

---

### Success Checkpoint

You've successfully completed this step when:
- ✓ [Specific success criterion 1]
- ✓ [Specific success criterion 2]
- ✓ [Specific success criterion 3]

**✗ If something failed:** See [Troubleshooting Section]

---

### What You Just Built

[Recap what they accomplished and why it matters]

**Next step:** [Link to next section]
```

---

## 6. Analogies and Examples

### Good Analogies to IT Concepts

**Recommended analogies for common RAG concepts:**

| RAG Concept | IT Analogy |
|-------------|------------|
| **Containers** | Virtual machines but lighter weight; like VLANs for applications |
| **Vector embeddings** | Converting addresses to GPS coordinates; different format, same info |
| **ChromaDB** | Routing table for documents; organized for fast lookups |
| **Ollama** | SNMP agent for AI; runs locally and processes requests |
| **RAG system** | Three-tier architecture; presentation, logic, data layers |
| **Webhooks** | SNMP traps; push notifications instead of polling |
| **API endpoints** | Management interfaces; how services communicate |
| **Chunking** | Packet fragmentation; breaking large data into processable pieces |
| **Semantic search** | QoS matching; finding best fit, not just exact match |

### How to Create Good Analogies

**Structure:**
1. State the RAG concept
2. Name a familiar IT concept
3. Explain the parallel
4. Show practical example

**Example:**
```markdown
**Vector embeddings** work like MAC address tables in switches. Just as a switch 
converts device names to MAC addresses for fast forwarding decisions, our system 
converts document text to number vectors for fast similarity matching. When you 
search, it's finding the "closest MAC addresses" to your question.
```

---

## 7. Error Documentation Format

Every error should be documented using this template:

```markdown
### Error: [Exact Error Message]

**Symptoms:**
- [What the user sees]
- [What behavior occurs]

**Cause:**
[Plain English explanation of root cause]

**Solution:**

**Quick fix:**
```bash
# Commands to resolve
fix-command here
```

**Detailed steps:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Prevention:**
[How to avoid this error in future]

**Related errors:**
- [Similar error A]
- [Similar error B]
```

### Example Error Documentation

```markdown
### Error: `Connection refused on port 8000`

**Symptoms:**
- Python script fails with connection error
- Cannot access ChromaDB
- Browser shows "This site can't be reached"

**Cause:**
ChromaDB container is not running or hasn't started yet.

**Solution:**

**Quick fix:**
```bash
# Check if container is running
podman ps | grep chromadb

# If not running, start it
podman start chromadb

# Wait 10 seconds for startup
sleep 10

# Verify it's accessible
curl http://localhost:8000/api/v1/heartbeat
```

**Detailed steps:**
1. Open terminal
2. Run `podman ps` to check running containers
3. If chromadb is not listed, run `podman start chromadb`
4. Wait 10 seconds for the service to start
5. Test connection with curl command above
6. If still failing, check logs: `podman logs chromadb`

**Prevention:**
Add ChromaDB to your startup script so it starts automatically when you boot your system.

**Related errors:**
- Port 8000 already in use
- ChromaDB container not found
```

---

## 8. Glossary Requirements

Every guide must include a glossary defining all technical terms.

### Glossary Entry Format

```markdown
## Glossary

### [Term]
**Simple definition:** [One sentence in plain English]
**IT analogy:** [How it relates to familiar IT concepts]
**In this system:** [Specific role in our RAG system]
**Example:** [Concrete example of the term in use]

---
```

### Example Glossary Entries

```markdown
## Glossary

### API (Application Programming Interface)
**Simple definition:** A way for programs to communicate with each other over a network.
**IT analogy:** Like SNMP for applications - a standardized way for different systems to exchange information.
**In this system:** Our RAG components (Ollama, ChromaDB, n8n) use APIs to pass questions and documents between each other.
**Example:** When you ask a question, n8n uses the ChromaDB API to search for relevant documents.

---

### Vector Embedding
**Simple definition:** Converting text into a list of numbers that represents its meaning.
**IT analogy:** Like converting hostnames to IP addresses - different format, but represents the same thing. Just as routers work with IPs not hostnames, our AI works with vectors not text.
**In this system:** Every document and question is converted to vectors so ChromaDB can find similar meanings quickly.
**Example:** "What's the network uptime?" becomes [0.23, -0.15, 0.87, ...] (768 numbers total).

---

### Container
**Simple definition:** A lightweight virtual environment that runs a program with all its dependencies.
**IT analogy:** Like a VLAN for applications - isolated environment sharing the same physical infrastructure.
**In this system:** ChromaDB and n8n run in containers so they don't interfere with other programs on your machine.
**Example:** The ChromaDB container has its own Python version and libraries, separate from your system's Python.

---

### RAG (Retrieval-Augmented Generation)
**Simple definition:** AI that looks up information in documents before answering questions.
**IT analogy:** Like how a router checks its routing table before forwarding packets - the AI checks your documents before generating answers.
**In this system:** When you ask a question, the system retrieves relevant document sections then generates an answer using that context.
**Example:** You ask "What's the budget?" → System finds budget document → AI reads it and answers "The Q3 budget is $250,000."
```

---

## 9. Screenshots and Visuals

### When to Include Screenshots

**Required screenshots:**
- First-time interface views (n8n dashboard, Webex bot)
- Configuration screens with non-obvious settings
- Success states that aren't obvious from text output
- Error messages that might be confusing

**NOT needed:**
- Terminal output that's shown in code blocks
- Standard file browsers or system dialogs
- Things that change frequently (URLs, dates)

### Screenshot Requirements

**Every screenshot must have:**
1. **Caption** explaining what's shown
2. **Callouts** highlighting important areas (use arrows/boxes)
3. **Context** explaining when user will see this
4. **File name** descriptive: `n8n-workflow-success.png`, not `screenshot1.png`

### Example Screenshot Documentation

```markdown
### Verify the Workflow is Active

After saving your workflow, you should see the activation toggle:

![n8n workflow activation toggle](images/n8n-workflow-active.png)

**What you're looking at:**
- The workflow name in the top-left
- The "Active" toggle (should be green/on)
- Last execution time (if workflow has run)

**✓ Success indicator:** Toggle is green and shows "Active"
**✗ Not activated:** Toggle is gray and shows "Inactive"

**If toggle won't turn on:** Check for red error nodes in the workflow - fix those first.
```

---

## 10. Time Estimates

### Guidelines for Time Estimates

**Always include realistic time estimates** for each section.

**Format:**
- **Reading time:** 5-10 minutes
- **Hands-on time:** 15-20 minutes
- **Total time:** 20-30 minutes

**What to include in estimates:**
- Reading the instructions
- Running commands and waiting for output
- Verification steps
- Reasonable troubleshooting time
- NOT included: downloading large files (note separately)

### Time Estimate Categories

```markdown
**⚡ Quick (< 5 minutes)**
- Running a single command
- Checking status
- Quick verification

**🕐 Short (5-15 minutes)**
- Installing a single component
- Basic configuration
- Simple testing

**🕑 Medium (15-45 minutes)**
- Multi-step installation
- Configuration with testing
- Complete feature setup

**🕐 Long (45+ minutes)**
- Complete system setup
- Multiple interdependent components
- Extensive testing and verification

**Note downloads separately:**
"Plus 10-30 minutes for initial model download (depends on internet speed)"
```

---

## 11. Prerequisites Documentation

### How to Document Prerequisites

**For every major section, clearly state:**

```markdown
📋 **Prerequisites**

**Before starting this section, you must have:**
- ✓ [Completed step X]
- ✓ [Specific software installed]
- ✓ [System in specific state]
- ✓ [Access to specific resource]

**Verify you're ready:**
```bash
# Quick verification command
verification-command
```

**Expected output:**
```
What success looks like
```

**If verification fails:** [Link to setup instructions]
```

### Example Prerequisites Section

```markdown
📋 **Prerequisites**

**Before starting this section, you must have:**
- ✓ Completed Part 1: Environment Setup
- ✓ Podman running with containers started
- ✓ Ollama installed with models downloaded
- ✓ At least one document loaded in ChromaDB

**Verify you're ready:**
```bash
# Check all services are running
podman ps
curl http://localhost:11434/api/tags
curl http://localhost:8000/api/v1/heartbeat
```

**Expected output:**
- Podman shows chromadb and n8n containers
- Ollama returns list of models
- ChromaDB returns heartbeat with timestamp

**If any verification fails:**
- Podman containers not showing → Return to [Part 1, Step 4](part1.md#step-4)
- Ollama not responding → Return to [Part 1, Step 3](part1.md#step-3)
- ChromaDB not responding → Return to [Part 1, Step 5](part1.md#step-5)
```

---

## 12. Cross-Referencing and Links

### Link Format

**Always use descriptive link text:**

✓ **Good:**
```markdown
See the [ChromaDB installation guide](chromadb-setup.md) for detailed steps.
```

✗ **Bad:**
```markdown
Click [here](chromadb-setup.md) for more information.
```

### Internal Document References

**Format:**
```markdown
[Descriptive Text](document-name.md#section-anchor)
```

**Examples:**
```markdown
- Detailed in [Part 2: RAG System Setup](GUIDE_2_RAG_SYSTEM.md#installation)
- See [Troubleshooting: Container Issues](TROUBLESHOOTING.md#container-not-starting)
- Review [Prerequisites](PREREQUISITES_AND_LEARNING.md#system-requirements)
```

### External Resource Links

**Always provide context for external links:**

```markdown
For more details on ChromaDB v1 API, see the [official ChromaDB documentation](https://docs.trychroma.com/api/v1).
```

---

## 13. Version and Update Information

### Document Header Template

Every document should start with:

```markdown
# Document Title

**Version:** X.Y.Z
**Last Updated:** Month Day, Year
**Status:** [Draft | Active | Archived]
**Applies to:** [Software version or system state]

---
```

### Changelog Section

Include at end of document:

```markdown
## Document History

### Version 2.0.0 (January 2026)
- Updated for Ollama 0.3.x
- Added troubleshooting for ChromaDB v1 API
- Expanded error documentation

### Version 1.0.0 (December 2024)
- Initial documentation
- Covers basic setup and configuration
```

---

## 14. FAQ Format

### FAQ Structure

Group FAQs by category and use consistent format:

```markdown
## Frequently Asked Questions

### General Questions

**Q: [Question in user's voice]**

A: [Direct answer first, then details]

[Optional: code example or additional context]

---

**Q: [Next question]**

A: [Answer]

---

### [Next Category]
```

### Example FAQ Section

```markdown
## Frequently Asked Questions

### General Questions

**Q: Do I need to know Python to use this system?**

A: No! You can use the system through the n8n visual interface (no coding required) or through the Webex bot (just send messages). The Python scripts are optional for advanced users who want automation.

---

**Q: How much does this cost?**

A: Everything in this guide uses free, open-source software. There are no licensing fees or subscriptions. You do need a computer with enough resources (8GB+ RAM recommended).

### Technical Questions

**Q: Why do we use Podman instead of Docker?**

A: Podman is similar to Docker but doesn't require root privileges and has better security. On most systems, both work fine, but Podman is easier to set up for this project.

---

**Q: What's the difference between Ollama and ChatGPT?**

A: Ollama runs AI models locally on your machine - nothing leaves your computer. ChatGPT runs in the cloud. For sensitive company documents, local processing is often required for privacy/compliance reasons.

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

**✓ Success looks like:**
```json
{"response":"Hello"}
```

**✗ If it fails:** See [Ollama Troubleshooting](troubleshooting.md#ollama-not-responding)

---

### Test 2: ChromaDB Stores and Retrieves Data

**What we're testing:** ChromaDB can save and find documents

```bash
# Run the test script
python test_chromadb.py
```

**✓ Success looks like:**
```
✓ Collection created
✓ Document added
✓ Query returned 1 result
All tests passed!
```

**✗ If it fails:** See [ChromaDB Troubleshooting](troubleshooting.md#chromadb-connection-issues)

---

### Test 3: End-to-End RAG Query

**What we're testing:** The complete system responds to questions

```bash
# Ask a question about your documents
python query_rag.py "What is the network budget?"
```

**✓ Success looks like:**
```
Found 3 relevant passages
Answer: According to the Q3 assessment, the network upgrade budget is $250,000...
```

**✗ If it fails:** See [RAG System Troubleshooting](troubleshooting.md#rag-not-finding-documents)

---

## ✓ Complete Test Checklist

Run through this checklist to confirm your system is fully operational:

- [ ] Ollama responds to API calls
- [ ] ChromaDB container is running (`podman ps` shows it)
- [ ] Documents are loaded (query returns results)
- [ ] RAG system generates answers
- [ ] n8n workflows execute without errors
- [ ] (Optional) Webex bot responds to messages

**All checked?** ✓ Your system is ready to use!
**Some failed?** 🔧 See the [Troubleshooting Guide](troubleshooting.md)
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
- [ ] Clear section hierarchy (## → ### → ####)
- [ ] Time estimates provided
- [ ] Prerequisites listed
- [ ] Verification steps included
- [ ] Next steps clearly indicated

### Formatting
- [ ] Code blocks have language specified
- [ ] Visual indicators used appropriately (✓ ⚠️ 💡)
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

#### ✗ BAD
```markdown
Run `podman run -d -p 8000:8000 chromadb/chromadb:0.4.24`
```

**Problems:**
- No explanation
- No expected output
- No verification
- No troubleshooting

#### ✓ GOOD
```markdown
### Start the ChromaDB Container

**What this does:** Launches ChromaDB in a virtual environment on your machine

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

**✓ Verify it's running:**
```bash
podman ps
```
You should see `chromadb/chromadb` in the IMAGE column.

**✗ Common error:**
If you see `port 8000 already in use`, another program is using that port. See [Port Conflicts](troubleshooting.md#port-conflicts).
```

---

### Example 2: Explaining a Concept

#### ✗ BAD
```markdown
ChromaDB uses vector embeddings to enable semantic search over document collections through cosine similarity metrics.
```

**Problems:**
- Pure jargon
- No analogy
- No practical explanation
- Assumes technical knowledge

#### ✓ GOOD
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

If yes to all four, your documentation meets the standards.
