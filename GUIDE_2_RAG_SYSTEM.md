# Guide 2: RAG System Setup
## Building Your Document Intelligence System

**Part 2 of 3: Implementation Guides**  
**Time Required:** 45 minutes  
**Difficulty:** Beginner-friendly  
**Prerequisites:** Completed Part 1 (Environment Setup)

---

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding RAG Systems](#understanding-rag-systems)
3. [Part 2A: What is RAG?](#part-2a-what-is-rag)
4. [Part 2B: Python Script Setup](#part-2b-python-script-setup)
5. [Part 2C: Load Your First Document](#part-2c-load-your-first-document)
6. [Part 2D: Test with Python Queries](#part-2d-test-with-python-queries)
7. [Part 2E: n8n Document Upload Workflow](#part-2e-n8n-document-upload-workflow)
8. [Part 2F: n8n Chat Interface](#part-2f-n8n-chat-interface)
9. [Part 2G: System Verification](#part-2g-system-verification)
10. [Troubleshooting](#troubleshooting)
11. [Next Steps](#next-steps)

---

## Introduction

Welcome back! In Part 1, you set up the foundation - all the software components are now installed and running. In this guide, you'll build the actual RAG (Retrieval-Augmented Generation) system that lets AI answer questions about your documents.

**What you'll build:**
- Document upload system (both Python and visual interface)
- Chat interface for asking questions
- Understanding of how RAG actually works

**Why this matters:**
Instead of sending your sensitive documents to ChatGPT or other cloud AI services, you'll have your own local AI assistant that can read and understand your specific documents - all while keeping data completely private.

---

### Before You Begin

**Verify your environment is ready:**

```bash
# Run the system check from Part 1
~/cisco-rag-demo/system-check.sh
```

**You should see all green checkmarks (✅):**
- Podman containers running (chromadb, n8n)
- Ollama models present (llama3.2:3b, nomic-embed-text)
- All services accessible
- cisco_docs collection exists

**If anything shows ❌:** Return to Part 1 and fix issues before continuing.

---

## Understanding RAG Systems

Before we start building, let's understand what you're creating and why it's powerful.

### What "RAG" Means

**RAG = Retrieval-Augmented Generation**

Let's break that down in plain English:

- **Retrieval** - Finding relevant information from your documents
- **Augmented** - Adding that information to enhance something
- **Generation** - AI creating an answer

**Simple explanation:** RAG teaches AI about your specific documents by finding relevant parts and including them when answering questions.

---

### Why RAG vs Regular AI

**ChatGPT without RAG:**
```
You: "What's our Q3 datacenter budget?"
ChatGPT: "I don't have access to your specific documents. 
         I can only provide general information about datacenter budgets."
```

**Your RAG system:**
```
You: "What's our Q3 datacenter budget?"
Your AI: "According to the FY2024 budget proposal, Q3 datacenter 
         spending is allocated at $187,500, with $125,000 for equipment 
         refresh and $62,500 for facility upgrades."
```

**The difference:** Your RAG system read your actual budget document and can cite specific numbers from it.

---

### Real-World Analogy

**Think of RAG like a research assistant:**

**Without RAG (like ChatGPT):**
- Knows general world knowledge
- Can't access your specific files
- Makes educated guesses
- Can't cite sources from your documents

**With RAG (what you're building):**
- Reads your specific documents first
- Searches through them when you ask questions
- Quotes directly from your files
- Cites which document and section it found information

**Network analogy:** 
- Regular AI = Generic ISP router (works for everyone, knows general routing)
- RAG = Custom-configured enterprise router (knows your specific network topology, VLANs, policies)

---

## Part 2A: What is RAG?

**Time:** 15 minutes (reading and understanding)

### The Four-Step RAG Process

**Here's exactly what happens when you use your RAG system:**

```
Step 1: DOCUMENT UPLOAD (Done Once)
─────────────────────────────────────
Your Document
  â†"
Split into Chunks (small pieces, ~800 characters each)
  â†"
Convert to Vectors (numbers that represent meaning)
  â†"
Store in ChromaDB (searchable database)


Step 2: ASK A QUESTION (Every Query)
──────────────────────────────────────
Your Question: "What's the network uptime?"
  â†"
Convert to Vector (same process as documents)
  â†"
Search ChromaDB for Similar Vectors
  â†"
Find 3 Most Relevant Chunks


Step 3: BUILD CONTEXT (Behind the Scenes)
───────────────────────────────────────────
Retrieved Chunks + Your Question
  â†"
Combine into a Single Prompt
  â†"
Send to AI Model


Step 4: GENERATE ANSWER (The Magic)
──────────────────────────────────────
AI Reads the Context
  â†"
Writes Human-Friendly Answer
  â†"
Returns Response to You
```

---

### Deep Dive: Why Chunks?

**Question:** Why break documents into pieces? Why not give AI the whole document?

**Answer:** AI models have limits on how much text they can process at once.

**Think of it like packet fragmentation:**
- Just as large network packets get fragmented for transmission
- Large documents get "fragmented" into chunks for AI processing
- Each chunk is independently searchable
- Relevant chunks are "reassembled" for the AI to read

**Our settings:**
- **Chunk size: 800 characters** (about 2-3 paragraphs)
- **Overlap: 100 characters** (prevents splitting mid-sentence)
- **Why overlap?** Maintains context between chunks

**Example from a network assessment:**
```
Chunk 1 (chars 0-800):
"...The Boston datacenter currently operates 24 Cisco Catalyst 
switches across three distribution layers. Network uptime for 
Q3 2024 was 99.94% with a mean time to repair of 2.4 hours..."

Chunk 2 (chars 700-1500): [100 char overlap starts here]
"...of 2.4 hours. The incident log shows five outages: two 
planned maintenance windows and three unplanned events caused 
by power fluctuations..."
```

Notice how the overlap ensures "2.4 hours" context isn't lost between chunks.

---

### Deep Dive: What Are Embeddings?

**Embeddings** are how we convert text into numbers that represent meaning.

**The problem:** Computers don't understand "datacenter" vs "data center" vs "facility" are similar concepts.

**The solution:** Convert words into mathematical vectors (lists of numbers) where similar meanings have similar numbers.

**Real example:**
```
Text: "network uptime"
Embedding: [0.23, -0.15, 0.87, 0.42, -0.31, ... 768 more numbers]

Text: "network availability"  
Embedding: [0.25, -0.14, 0.85, 0.44, -0.29, ... 768 more numbers]
        ↑     ↑     ↑     ↑     ↑
     Very similar numbers = Similar meaning!

Text: "pizza toppings"
Embedding: [-0.62, 0.91, -0.13, 0.08, 0.77, ... 768 more numbers]
        ↑     ↑     ↑     ↑     ↑
     Very different numbers = Different meaning!
```

**Network analogy:** Think of embeddings like GPS coordinates:
- "Network uptime" might be at coordinates (40.7128° N, 74.0060° W)
- "Network availability" might be at (40.7129° N, 74.0061° W) - very close!
- "Pizza toppings" might be at (51.5074° N, 0.1278° W) - far away!

When you search, ChromaDB finds the "closest coordinates" to your question.

---

### Deep Dive: How Vector Search Works

**Traditional search (keyword matching):**
```
Question: "What's our uptime?"
Document chunk: "The network availability was 99.94%"
Match: ❌ NO - doesn't contain word "uptime"
```

**Vector search (semantic matching):**
```
Question: "What's our uptime?"
  â†' Vector: [0.23, -0.15, 0.87, ...]

Document chunk: "The network availability was 99.94%"
  â†' Vector: [0.25, -0.14, 0.85, ...]

Distance calculation: CLOSE! (similar meaning)
Match: âœ… YES - semantically similar
```

**The math (you don't need to know this, but it's interesting):**
```
Cosine Similarity = How similar two vectors are
- Score of 1.0 = Identical meaning
- Score of 0.8 = Very similar
- Score of 0.5 = Somewhat related
- Score of 0.0 = Completely different
```

ChromaDB returns the top 3 chunks with highest similarity scores.

---

### Why Local RAG vs Cloud AI?

**Comparison table:**

| Feature | Your RAG System | ChatGPT/Cloud AI |
|---------|----------------|------------------|
| **Privacy** | âœ… Docs never leave your Mac | âŒ Uploaded to external servers |
| **Cost** | âœ… Free after setup | âŒ $0.002-0.06 per 1K tokens |
| **Compliance** | âœ… HIPAA/SOC2 friendly | âŒ May violate policies |
| **Customization** | âœ… Your docs, your rules | âŒ Generic knowledge |
| **Offline** | âœ… Works without internet | âŒ Requires connection |
| **Speed** | âœ… 5-10 seconds | âœ… 2-5 seconds (faster) |
| **Accuracy** | âœ… Cites your documents | ⚠️ May hallucinate |
| **Knowledge** | ⚠️ Only your documents | âœ… Knows everything |

**Best use cases for YOUR RAG system:**
- Network assessments and reports
- Internal policies and procedures  
- Financial documents and budgets
- Customer data and account information
- Any sensitive or proprietary documents

**When cloud AI is better:**
- General knowledge questions
- Current events and news
- Programming help (unless you have company code docs)
- Creative writing tasks

---

### What You'll Build Today

By the end of this guide, you'll have:

**Two ways to upload documents:**
1. **Python script** - Fast, command-line based
2. **n8n web form** - Visual, user-friendly

**Two ways to query:**
1. **Python script** - Quick testing, automation-friendly
2. **n8n chat interface** - Interactive, demo-friendly

**Why both?**
- Python = Quick testing, proves concept works
- n8n = Beautiful demos, easier for non-technical users

**Total time investment:** About 45 minutes to get everything working.

---

## Part 2B: Python Script Setup

**Time:** 15 minutes  
**What you'll do:** Create two Python scripts for loading documents and querying  
**Why:** Quick testing before building visual workflows

---

### Why Start with Python?

**Benefits:**
- âœ… Faster to test (no UI to navigate)
- âœ… Easier to debug (see exact errors)
- âœ… Proves the concept works
- âœ… Can be automated later

**Don't worry if you're not a programmer** - you'll just copy-paste complete, working scripts. We'll explain what each part does as we go.

---

### Script 1: Document Loader

**What this script does:**
1. Reads a text file from your computer
2. Breaks it into chunks (800 characters each)
3. Converts each chunk to a vector (using Ollama)
4. Stores everything in ChromaDB

**Create the script:**

```bash
cat > ~/cisco-rag-demo/load_document.py << 'SCRIPT_END'
#!/usr/bin/env python3
"""
RAG Document Loader - UUID-Aware Version
Loads documents into ChromaDB for RAG system
"""

import requests
import json
import sys
import warnings

warnings.filterwarnings('ignore', message='.*OpenSSL.*')

def get_collection_uuid(collection_name):
    """
    Get collection UUID by name.
    
    ChromaDB v1 API requires UUIDs in URLs, not names.
    This function looks up the UUID for our collection.
    """
    try:
        # Get all collections
        resp = requests.get('http://localhost:8000/api/v1/collections', timeout=5)
        
        if resp.status_code == 200:
            collections = resp.json()
            
            # Find our collection by name
            for coll in collections:
                if coll.get('name') == collection_name:
                    return coll.get('id')
        
        print(f"❌ Collection '{collection_name}' not found")
        return None
        
    except Exception as e:
        print(f"❌ Error getting collection: {e}")
        return None

def chunk_document(text, chunk_size=800, overlap=100):
    """
    Split document into overlapping chunks.
    
    Args:
        text: Full document text
        chunk_size: Size of each chunk (characters)
        overlap: How much chunks overlap
    
    Returns:
        List of text chunks
    
    Why overlap? Prevents important information from being 
    split across chunk boundaries.
    """
    chunks = []
    start = 0
    
    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end]
        
        if chunk.strip():  # Only add non-empty chunks
            chunks.append(chunk.strip())
        
        # Move forward, but overlap
        start = end - overlap
    
    return chunks

def main():
    """Main execution function"""
    
    # Check command line arguments
    if len(sys.argv) < 2:
        print("Usage: python3 load_document.py <document.md>")
        print("Example: python3 load_document.py documents/network-assessment.md")
        sys.exit(1)
    
    file_path = sys.argv[1]
    
    # Step 1: Read the document
    print(f"\nðŸ"„ Reading document: {file_path}")
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            document = f.read()
    except FileNotFoundError:
        print(f"❌ File not found: {file_path}")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Error reading file: {e}")
        sys.exit(1)
    
    print(f"âœ… Read {len(document)} characters")
    
    # Step 2: Get collection UUID (CRITICAL for v1 API)
    print(f"\nðŸ" Getting collection UUID...")
    collection_uuid = get_collection_uuid('cisco_docs')
    
    if not collection_uuid:
        print("\n❌ Cannot proceed without collection UUID")
        print("Make sure you created the collection in Part 1:")
        print("curl -X POST http://localhost:8000/api/v1/collections \\")
        print("  -H 'Content-Type: application/json' \\")
        print("  -d '{\"name\":\"cisco_docs\"}'")
        sys.exit(1)
    
    print(f"âœ… Collection UUID: {collection_uuid[:8]}...")
    
    # Step 3: Chunk the document
    print(f"\nâœ‚ï¸  Chunking document...")
    chunks = chunk_document(document, chunk_size=800, overlap=100)
    print(f"âœ… Document split into {len(chunks)} chunks")
    
    if len(chunks) == 0:
        print("❌ No content to process")
        sys.exit(1)
    
    # Step 4: Process each chunk
    print(f"\nðŸ"„ Processing chunks...\n")
    
    ids = []
    embeddings = []
    documents = []
    metadatas = []
    
    for i, chunk in enumerate(chunks):
        print(f"Processing chunk {i+1}/{len(chunks)}...", end=" ")
        
        try:
            # Generate embedding for this chunk
            embed_resp = requests.post(
                'http://localhost:11434/api/embeddings',
                json={
                    'model': 'nomic-embed-text',
                    'prompt': chunk
                },
                timeout=30
            )
            
            if embed_resp.status_code != 200:
                print(f"❌ Error: {embed_resp.status_code}")
                continue
            
            embedding = embed_resp.json()['embedding']
            
            # Store chunk data
            chunk_id = f"chunk_{i}"
            ids.append(chunk_id)
            embeddings.append(embedding)
            documents.append(chunk)
            metadatas.append({
                'chunk_index': i,
                'chunk_size': len(chunk),
                'source_file': file_path
            })
            
            print("âœ…")
            
        except Exception as e:
            print(f"❌ Error: {e}")
            continue
    
    # Step 5: Store in ChromaDB (batch operation)
    if len(ids) == 0:
        print("\n❌ No chunks successfully processed")
        sys.exit(1)
    
    print(f"\nðŸ'¾ Storing {len(ids)} chunks in ChromaDB...")
    
    try:
        # Use UUID in URL - this is the critical fix!
        add_resp = requests.post(
            f'http://localhost:8000/api/v1/collections/{collection_uuid}/add',
            json={
                'ids': ids,
                'embeddings': embeddings,
                'documents': documents,
                'metadatas': metadatas
            },
            timeout=30
        )
        
        if add_resp.status_code == 201:
            print(f"âœ… Success! Loaded {len(ids)} chunks into ChromaDB")
            print(f"\nYou can now query this document with:")
            print(f"./query_rag.py \"What is this document about?\"")
        else:
            print(f"❌ Storage failed: {add_resp.status_code}")
            print(f"Response: {add_resp.text}")
            
    except Exception as e:
        print(f"❌ Error storing in ChromaDB: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
SCRIPT_END

# Make executable
chmod +x ~/cisco-rag-demo/load_document.py
```

---

### Understanding the Document Loader Script

Let's break down what each section does:

**Section 1: Imports**
```python
import requests  # For HTTP requests to ChromaDB and Ollama
import json      # For working with JSON data
import sys       # For command line arguments
import warnings  # To suppress SSL warnings
```

**Section 2: UUID Lookup Function**
```python
def get_collection_uuid(collection_name):
    # Gets all collections from ChromaDB
    # Finds the one matching our name
    # Returns its UUID
    # This is critical - ChromaDB v1 API needs UUIDs!
```

**Why this matters:** Remember from Part 1, ChromaDB v1 API uses UUIDs (like `12345678-1234-...`) in URLs, not collection names. This function does the lookup automatically.

**Section 3: Chunking Function**
```python
def chunk_document(text, chunk_size=800, overlap=100):
    # Splits text into 800-character pieces
    # Each piece overlaps by 100 characters
    # Prevents mid-sentence splits
```

**Visual example:**
```
Full text: "The network uptime is 99.94%. This exceeds our SLA..."
           [Chunk 1: 800 chars starting at 0]
                        [Chunk 2: 800 chars starting at 700]
                        ↑ 100 char overlap
```

**Section 4: Main Processing**
```python
1. Read file from disk
2. Get collection UUID
3. Break into chunks
4. For each chunk:
   - Send to Ollama for embedding
   - Store the embedding
5. Upload all chunks to ChromaDB at once (batch)
```

**Why batch upload?** Faster than uploading one chunk at a time. Like sending one large packet vs many small packets.

---

### Script 2: Query System

**What this script does:**
1. Takes your question as input
2. Converts question to a vector
3. Searches ChromaDB for similar chunks
4. Builds a prompt with those chunks
5. Asks Ollama AI to generate an answer

**Create the script:**

```bash
cat > ~/cisco-rag-demo/query_rag.py << 'SCRIPT_END'
#!/usr/bin/env python3
"""
RAG Query System - UUID-Aware Version
Query documents using natural language
"""

import requests
import json
import sys
import warnings

warnings.filterwarnings('ignore', message='.*OpenSSL.*')

def get_collection_uuid(collection_name):
    """Get collection UUID by name"""
    try:
        resp = requests.get('http://localhost:8000/api/v1/collections', timeout=5)
        
        if resp.status_code == 200:
            collections = resp.json()
            for coll in collections:
                if coll.get('name') == collection_name:
                    return coll.get('id')
        return None
    except:
        return None

def query_rag(question):
    """
    Query the RAG system with a natural language question.
    
    Process:
    1. Convert question to embedding
    2. Search ChromaDB for similar chunks
    3. Build prompt with retrieved chunks
    4. Generate answer with Ollama
    """
    
    # Step 1: Get collection UUID
    print(f"\nðŸ" Searching documents...")
    collection_uuid = get_collection_uuid('cisco_docs')
    
    if not collection_uuid:
        print("❌ Collection 'cisco_docs' not found")
        print("Load documents first with: ./load_document.py documents/your-file.md")
        return
    
    # Step 2: Generate question embedding
    try:
        embed_resp = requests.post(
            'http://localhost:11434/api/embeddings',
            json={
                'model': 'nomic-embed-text',
                'prompt': question
            },
            timeout=30
        )
        
        if embed_resp.status_code != 200:
            print(f"❌ Embedding error: {embed_resp.status_code}")
            return
        
        question_embedding = embed_resp.json()['embedding']
        
    except Exception as e:
        print(f"❌ Error generating embedding: {e}")
        return
    
    # Step 3: Search ChromaDB using UUID
    try:
        # Use UUID in URL - critical for v1 API!
        search_resp = requests.post(
            f'http://localhost:8000/api/v1/collections/{collection_uuid}/query',
            json={
                'query_embeddings': [question_embedding],
                'n_results': 3  # Get top 3 most similar chunks
            },
            timeout=10
        )
        
        if search_resp.status_code != 200:
            print(f"❌ Search error: {search_resp.text}")
            return
        
        results = search_resp.json()
        
        # Check if we found any documents
        if not results.get('documents') or not results['documents'][0]:
            print("❌ No documents found. Load documents first.")
            return
        
        contexts = results['documents'][0]
        distances = results.get('distances', [[]])[0]
        
        print(f"âœ… Found {len(contexts)} relevant chunks")
        
        # Show similarity scores if available
        if distances:
            print(f"   Similarity scores: {', '.join([f'{1-d:.3f}' for d in distances[:3]])}")
        
    except Exception as e:
        print(f"❌ Search error: {e}")
        return
    
    # Step 4: Build prompt with retrieved context
    context_text = "\n\n".join([
        f"[Source {i+1}]\n{ctx}" 
        for i, ctx in enumerate(contexts)
    ])
    
    prompt = f"""Based on the following context from a Cisco network assessment document, answer the question accurately and concisely.

Context:
{context_text}

Question: {question}

Answer:"""
    
    # Step 5: Generate answer with AI
    print(f"\nðŸ¤– Generating answer...\n")
    
    try:
        llm_resp = requests.post(
            'http://localhost:11434/api/generate',
            json={
                'model': 'llama3.2:3b',
                'prompt': prompt,
                'stream': False
            },
            timeout=60
        )
        
        if llm_resp.status_code != 200:
            print(f"❌ Generation error: {llm_resp.status_code}")
            return
        
        answer = llm_resp.json()['response']
        
    except Exception as e:
        print(f"❌ Error generating answer: {e}")
        return
    
    # Step 6: Display formatted answer
    print("=" * 60)
    print(answer.strip())
    print("=" * 60)
    print(f"\nâœ… Answer generated from {len(contexts)} document chunks")

def main():
    """Main execution"""
    
    if len(sys.argv) < 2:
        print("Usage: ./query_rag.py \"Your question here\"")
        print("\nExamples:")
        print("  ./query_rag.py \"What is the network uptime?\"")
        print("  ./query_rag.py \"What equipment needs replacement?\"")
        print("  ./query_rag.py \"What is the proposed budget?\"")
        sys.exit(1)
    
    question = " ".join(sys.argv[1:])
    query_rag(question)

if __name__ == "__main__":
    main()
SCRIPT_END

# Make executable
chmod +x ~/cisco-rag-demo/query_rag.py
```

---

### Understanding the Query Script

**Flow diagram:**
```
Your Question
    â†"
Convert to Embedding (vector)
    â†"
Search ChromaDB (find similar chunks)
    â†"
Retrieve Top 3 Chunks
    â†"
Build Prompt (chunks + question)
    â†"
Send to Ollama AI
    â†"
Get Answer
    â†"
Display to You
```

**Key sections:**

**UUID Lookup** (again!)
```python
collection_uuid = get_collection_uuid('cisco_docs')
# Same pattern as loader - must get UUID first
```

**Embedding Generation**
```python
# Converts your question to same format as document chunks
embed_resp = requests.post(
    'http://localhost:11434/api/embeddings',
    json={'model': 'nomic-embed-text', 'prompt': question}
)
```

**Vector Search**
```python
# Finds chunks with closest vector match
search_resp = requests.post(
    f'.../{collection_uuid}/query',
    json={
        'query_embeddings': [question_embedding],
        'n_results': 3  # Top 3 matches
    }
)
```

**Prompt Building**
```python
# Combines retrieved chunks with question
prompt = f"""Based on the following context... 

Context:
{chunk1}
{chunk2}
{chunk3}

Question: {your_question}

Answer:"""
```

This prompt is what the AI actually sees. It gets your question AND relevant chunks together.

**Answer Generation**
```python
# Ollama reads the prompt and writes answer
llm_resp = requests.post(
    'http://localhost:11434/api/generate',
    json={'model': 'llama3.2:3b', 'prompt': prompt}
)
```

---

### Verify Scripts Are Ready

**Check both scripts exist:**
```bash
ls -lh ~/cisco-rag-demo/*.py
```

**Expected output:**
```
-rwxr-xr-x  1 you  staff   5.2K Dec 18 10:30 load_document.py
-rwxr-xr-x  1 you  staff   4.8K Dec 18 10:30 query_rag.py
```

**What to verify:**
- âœ… Both files present
- âœ… Size looks reasonable (4-6 KB each)
- âœ… Executable permission (the 'x' in -rwxr-xr-x)

ðŸ'¡ **Tip:** If not executable, run: `chmod +x ~/cisco-rag-demo/*.py`

---

### Success Checkpoint

Before proceeding, confirm:

- [ ] `load_document.py` script created
- [ ] `query_rag.py` script created
- [ ] Both scripts are executable
- [ ] You understand (conceptually) what each script does

âœ… **Ready to load your first document!**

---

## Part 2C: Load Your First Document

**Time:** 10 minutes  
**What you'll do:** Upload a document and see it processed into chunks  
**Why:** Proves the system works before building visual interfaces

---

### Get a Sample Document

**Option 1: Create a test document**

```bash
cat > ~/cisco-rag-demo/documents/test-network-assessment.md << 'DOC_END'
# Acme Corporation Network Assessment
## Executive Summary

Date: December 2024
Location: Boston Office
Prepared by: Network Engineering Team

## Current Infrastructure

Acme Corporation's Boston office network consists of:
- 24 Cisco Catalyst 3850 switches
- 3 Cisco ASR 1001 routers
- 450 active network endpoints
- 1 Gbps internet connectivity

## Network Performance

Year-to-date network uptime: 99.94%
Mean time to repair: 2.4 hours
Total incidents: 5 (2 planned, 3 unplanned)

The network performance exceeds our SLA requirements of 99.9% uptime. 
All unplanned outages were resolved within 4 hours.

## Equipment Status

End-of-Support Equipment:
- 15 Cisco 3850 switches reach EOS in October 2025
- Recommended replacement: Cisco Catalyst 9300 series
- Estimated cost: $187,500

## Budget Proposal

FY2025 Network Upgrade Budget:
- Equipment: $250,000
- Installation: $50,000
- Licensing: $25,000
- Total: $325,000

## Recommendations

1. Replace EOS equipment in Q1 2025
2. Upgrade internet to 2 Gbps by Q2 2025
3. Implement SD-WAN across all branch offices
4. Budget allocation approved for $325,000

## Contact Information

Project Manager: Sarah Johnson (sarah.johnson@acme.com)
Network Lead: Mike Chen (mike.chen@acme.com)
Budget Approval: Jane Smith, CFO
DOC_END
```

**Option 2: Use your own document**

Requirements:
- Plain text format (.txt or .md)
- Size: 1 KB to 50 KB works best
- Content: Technical documents work well

---

### Load the Document

**Run the loading script:**

```bash
cd ~/cisco-rag-demo
python3 load_document.py documents/test-network-assessment.md
```

---

### Understanding the Output

**You'll see output like this (with explanation):**

```
ðŸ"„ Reading document: documents/test-network-assessment.md
âœ… Read 1847 characters
```
**What this means:** Successfully read the file. Character count tells you document size.

```
ðŸ" Getting collection UUID...
âœ… Collection UUID: 12345678...
```
**What this means:** Found the cisco_docs collection. The UUID lookup worked!

```
âœ‚ï¸  Chunking document...
âœ… Document split into 4 chunks
```
**What this means:** Your 1847-character document became 4 chunks of ~800 characters each.

**Why 4 chunks?** 
```
Chunk 1: chars 0-800
Chunk 2: chars 700-1500 (100 char overlap)
Chunk 3: chars 1400-2200 (100 char overlap)
Chunk 4: chars 2100-1847 (partial final chunk)
```

```
ðŸ"„ Processing chunks...

Processing chunk 1/4... âœ…
Processing chunk 2/4... âœ…
Processing chunk 3/4... âœ…
Processing chunk 4/4... âœ…
```
**What this means:** Each chunk is being:
1. Sent to Ollama
2. Converted to a 768-number vector
3. Prepared for storage

**Why it takes time:** Each embedding generation calls Ollama, which processes the text through the AI model. Takes ~2-3 seconds per chunk.

```
ðŸ'¾ Storing 4 chunks in ChromaDB...
âœ… Success! Loaded 4 chunks into ChromaDB

You can now query this document with:
./query_rag.py "What is this document about?"
```
**What this means:** All 4 chunks stored successfully. Your document is now searchable!

---

### Verify Documents Were Stored

**Check document count in ChromaDB:**

```bash
# Get collection UUID first
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; data = json.load(sys.stdin); print([c['id'] for c in data if c['name']=='cisco_docs'][0])")

# Check count
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
```

**Expected output:**
```json
4
```

**What this means:** ChromaDB confirms it has 4 documents (chunks) stored.

---

### What Just Happened (Technical Detail)

**Let's trace what happened to your document:**

**Before RAG:**
```
File: test-network-assessment.md
Content: Plain text, 1847 characters
Location: Your disk
Searchable: No
AI-readable: No
```

**After RAG:**
```
In ChromaDB:
- Chunk 1: "# Acme Corporation..." → Vector [0.23, -0.15, ...]
- Chunk 2: "Year-to-date network..." → Vector [0.18, -0.22, ...]
- Chunk 3: "End-of-Support Equipment..." → Vector [0.31, -0.08, ...]
- Chunk 4: "Project Manager: Sarah..." → Vector [0.09, -0.31, ...]

Searchable: Yes (by meaning, not just keywords)
AI-readable: Yes (can retrieve and read relevant chunks)
```

**Storage structure:**
```
ChromaDB Collection: cisco_docs (UUID: 12345678...)
â"‚
â"œâ"€ chunk_0
â"‚  â"œâ"€ ID: "chunk_0"
â"‚  â"œâ"€ Text: "# Acme Corporation Network Assessment..."
â"‚  â"œâ"€ Vector: [0.23, -0.15, 0.87, ... 765 more numbers]
â"‚  â""â"€ Metadata: {chunk_index: 0, source_file: "test-network..."}
â"‚
â"œâ"€ chunk_1
â"‚  â"œâ"€ ID: "chunk_1"
â"‚  â"œâ"€ Text: "Year-to-date network uptime: 99.94%..."
â"‚  â"œâ"€ Vector: [0.18, -0.22, 0.91, ... 765 more numbers]
â"‚  â""â"€ Metadata: {chunk_index: 1, source_file: "test-network..."}
â"‚
... (chunks 2 and 3)
```

---

### Success Checkpoint

Before proceeding, verify:

- [ ] Document loaded without errors
- [ ] Saw âœ… for each chunk processed
- [ ] Final "Success!" message appeared
- [ ] Count shows correct number of chunks
- [ ] You understand what chunking and embedding mean

âœ… **Your document is now ready to query!**

---

## Part 2D: Test with Python Queries

**Time:** 10 minutes  
**What you'll do:** Ask questions about your document and see AI-generated answers  
**Why:** Verify RAG system works end-to-end

---

### Your First Query

**Ask about the document:**

```bash
cd ~/cisco-rag-demo
./query_rag.py "What is the network uptime?"
```

---

### Understanding Query Output

**You'll see:**

```
ðŸ" Searching documents...
âœ… Found 3 relevant chunks
   Similarity scores: 0.892, 0.845, 0.823
```

**What this means:**
- **Found 3 chunks** - ChromaDB returned top 3 most similar chunks
- **Similarity scores** - How closely each chunk matches your question:
  - 1.0 = Perfect match
  - 0.8-0.9 = Very relevant
  - 0.6-0.8 = Somewhat relevant
  - <0.6 = Loosely relevant

**In this case:** 0.892 means very high relevance - ChromaDB found exactly the right chunk!

```
ðŸ¤– Generating answer...
```

**What's happening now:**
1. Building prompt with question + 3 retrieved chunks
2. Sending to Ollama AI
3. AI reading context and writing answer

**Why it takes 5-10 seconds:**
- First query: AI model loading into RAM (10-15 sec)
- Subsequent queries: Model stays in RAM (5-8 sec)

```
============================================================
According to the network assessment, Acme Corporation's 
Boston office network has a year-to-date uptime of 99.94%, 
which exceeds the SLA requirement of 99.9%. The mean time 
to repair is 2.4 hours, and there were 5 total incidents 
(2 planned and 3 unplanned outages).
============================================================

âœ… Answer generated from 3 document chunks
```

**What this means:**
- AI found the information in your document
- Cited specific numbers (99.94%, 2.4 hours, 5 incidents)
- Provided context (exceeds SLA, incident breakdown)
- Used 3 chunks to form complete answer

**This is RAG working!** The AI didn't hallucinate or guess - it read your specific document and extracted the information.

---

### Try Different Question Types

**Factual question (simple lookup):**
```bash
./query_rag.py "What equipment needs replacement?"
```

**Expected answer:**
```
15 Cisco 3850 switches need replacement as they reach 
end-of-support in October 2025. The recommended replacement 
is the Cisco Catalyst 9300 series, with an estimated cost 
of $187,500.
```

---

**Budget question (specific numbers):**
```bash
./query_rag.py "What is the proposed budget?"
```

**Expected answer:**
```
The FY2025 Network Upgrade Budget totals $325,000, broken 
down as: Equipment ($250,000), Installation ($50,000), and 
Licensing ($25,000).
```

---

**Analytical question (requires understanding):**
```bash
./query_rag.py "Does the network meet SLA requirements?"
```

**Expected answer:**
```
Yes, the network exceeds SLA requirements. The current 
uptime of 99.94% surpasses the required 99.9% SLA target.
```

**Why this is impressive:** The AI understood:
1. What "SLA requirements" means
2. That 99.94% > 99.9%
3. How to frame this as "yes, exceeds"

---

**Contact information (extraction):**
```bash
./query_rag.py "Who is the project manager?"
```

**Expected answer:**
```
Sarah Johnson is the Project Manager. Her email is 
sarah.johnson@acme.com.
```

---

**Summarization question:**
```bash
./query_rag.py "Summarize the main recommendations"
```

**Expected answer:**
```
The main recommendations are:
1. Replace end-of-support equipment in Q1 2025
2. Upgrade internet from 1 Gbps to 2 Gbps by Q2 2025
3. Implement SD-WAN across all branch offices
4. Allocate approved budget of $325,000
```

---

### What Makes Good vs Bad Questions?

**Good questions (work well):**
- âœ… Specific: "What is the network uptime?"
- âœ… Focused: "What equipment needs replacement?"
- âœ… Factual: "Who is the project manager?"
- âœ… Contextual: "Does the network meet requirements?"

**Questions that might not work as well:**
- ⚠️ Too broad: "Tell me everything"
- ⚠️ Multiple topics: "What's the budget and who's the manager and when..."
- ⚠️ Outside document: "What's the weather?" (not in document)
- ⚠️ Requires external knowledge: "Is Cisco Catalyst good?" (needs opinion, not just facts from doc)

**Why the difference?**
- RAG excels at finding and citing specific information FROM your documents
- RAG struggles with general knowledge or creative tasks
- Keep questions focused on what's IN the document

---

### Understanding Response Times

**Expected performance:**

| Scenario | Time | Explanation |
|----------|------|-------------|
| **First query** | 10-15 sec | Loading AI model into RAM |
| **Second query** | 5-8 sec | Model already in RAM |
| **After idle** | 10-15 sec | Model unloaded, reloading |
| **Simple question** | 4-6 sec | Less text to generate |
| **Complex question** | 8-12 sec | More text to generate |

**What influences speed:**
- **Model size** - llama3.2:3b is optimized for speed
- **RAM available** - More RAM = model stays loaded longer
- **Question complexity** - Longer answers take more time
- **First query** - Always slower (model loading)

**Network analogy:** Like the difference between:
- Cold routing table = First packet (10-15 sec)
- Hot routing table = Subsequent packets (5-8 sec)
- Cache timeout = Model unloaded after idle (back to 10-15 sec)

---

### Trace: What Happens Behind the Scenes

**Let's trace a complete query:**

**Your question:** "What is the network uptime?"

**Step 1: Embedding** (2-3 seconds)
```
Text: "What is the network uptime?"
  â†"
Ollama (nomic-embed-text model)
  â†"
Vector: [0.31, -0.18, 0.92, 0.44, -0.27, ... 763 more]
```

**Step 2: Search** (<1 second)
```
Question vector: [0.31, -0.18, 0.92, ...]
  â†"
Compare to ALL chunk vectors in ChromaDB
  â†"
Find 3 closest matches:
  - Chunk 1: Distance 0.108 (similarity 0.892)
  - Chunk 2: Distance 0.155 (similarity 0.845)
  - Chunk 3: Distance 0.177 (similarity 0.823)
  â†"
Return these 3 chunks
```

**Step 3: Prompt Building** (<1 second)
```
Combine:
- System instructions ("You are a helpful assistant...")
- Retrieved chunks (the 3 matched chunks)
- User question
  â†"
Create full prompt for AI
```

**Step 4: Generation** (3-8 seconds)
```
Full prompt:
"Based on the following context...
[Chunk 1 text]
[Chunk 2 text]
[Chunk 3 text]
Question: What is the network uptime?
Answer:"
  â†"
Ollama (llama3.2:3b model)
  â†"
Generated answer: "According to the network assessment, 
Acme Corporation's Boston office network has a year-to-date 
uptime of 99.94%..."
```

**Total time: 5-10 seconds**

---

### Performance Optimization Tips

**If queries are slow (>15 seconds consistently):**

**1. Pre-load AI models:**
```bash
# Run once before demos
ollama run llama3.2:3b "test" 
# Press Ctrl+D to exit
# Model now stays in RAM
```

**2. Check available RAM:**
```bash
# Open Activity Monitor
# Check "Memory Pressure"
# If yellow/red → close other apps
```

**3. Verify all services:**
```bash
~/cisco-rag-demo/system-check.sh
# All should show âœ…
```

**4. Increase Podman memory** (if you have RAM to spare):
```bash
podman machine stop
podman machine set --memory 16384  # 16GB instead of 12GB
podman machine start
```

---

### Success Checkpoint

Before moving to n8n workflows, verify:

- [ ] Can run queries successfully
- [ ] Answers are accurate and cite document content
- [ ] Response times are acceptable (<15 sec)
- [ ] Different question types work
- [ ] You understand how the RAG flow works

âœ… **Python RAG system is working! Ready for visual workflows.**

---

## Part 2E: n8n Document Upload Workflow

**Time:** 20 minutes  
**What you'll do:** Import a visual workflow that creates a web form for document uploads  
**Why:** Makes document loading user-friendly for demos and non-technical users

---

### Why n8n Workflows?

**You just proved it works with Python. Why add n8n?**

**Benefits of n8n:**
- âœ… Visual interface (no command line)
- âœ… Web forms (click and upload)
- âœ… Easy to demo (impressive visuals)
- âœ… Non-technical friendly
- âœ… Can customize without coding

**When to use Python vs n8n:**
- **Python** - Bulk operations, automation, testing
- **n8n** - Demos, presentations, end-user access

---

### Download the Workflow File

**The workflow JSON is in your project files.**

**Check if you have it:**
```bash
ls -lh /mnt/project/Document_Upload_-_DOC_Support.json
```

**If you don't have it, here's where to get it:**
- From the project repository
- Or create it using the template (advanced)

For this guide, we'll assume you have the file at:
`/mnt/project/Document_Upload_-_DOC_Support.json`

---

### Open n8n Interface

**Access n8n:**
```bash
open http://localhost:5678
```

**Or manually navigate to:** `http://localhost:5678` in your browser

**You should see:**
- n8n main interface
- Left sidebar with workflows
- Empty canvas (or existing workflows if you created any)

---

### Import the Workflow

**Step 1: Access import menu**

Click the **"⋯"** (three dots) menu in the top-right corner

Select **"Import from File"**

---

**Step 2: Select the file**

A file picker dialog appears

Navigate to: `/mnt/project/`

Select: `Document_Upload_-_DOC_Support.json`

Click **"Open"**

---

**Step 3: Workflow loads**

You'll see the workflow appear on the canvas

**What you're looking at:**
- 9 nodes (boxes) connected by arrows
- Each node is a step in the process
- Arrows show data flow direction

**Workflow name:** "RAG - Document Upload (Popup) - DOC Support"

---

### Understanding the Workflow Architecture

**Visual representation:**

```
â""â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"¬â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€
   [1] Form Trigger              
       â"œâ"€ Creates popup form
       â"œâ"€ Accepts .txt, .doc, .docx
       â""â"€ User fills and submits
          â†"
   [2] Extract File Data         
       â"œâ"€ Gets uploaded file
       â"œâ"€ Reads metadata
       â""â"€ Validates format
          â†"
   [3] Parse Document Content   
       â"œâ"€ Handles .txt (direct)
       â"œâ"€ Handles .docx (extract)
       â""â"€ Handles .doc (extract)
          â†"
   [4] Get Collection UUID       
       â"œâ"€ Queries ChromaDB
       â"œâ"€ Finds "cisco_docs"
       â""â"€ Returns UUID
          â†"
   [5] Process Document          
       â"œâ"€ Chunks text (800 chars)
       â"œâ"€ Creates unique IDs
       â""â"€ Prepares metadata
          â†"
   [6] Generate Embeddings       
       â"œâ"€ Calls Ollama
       â"œâ"€ One per chunk
       â""â"€ Returns vectors
          â†"
   [7] Prepare Storage           
       â"œâ"€ Combines chunks + vectors
       â"œâ"€ Formats for ChromaDB
       â""â"€ Batch prepares
          â†"
   [8] Store in ChromaDB         
       â"œâ"€ Sends batch request
       â"œâ"€ Uses UUID in URL
       â""â"€ Saves all chunks
          â†"
   [9] Success                   
       â"œâ"€ Confirms completion
       â"œâ"€ Shows chunk count
       â""â"€ Returns to user
â""â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"´â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€
```

---

### Node-by-Node Explanation

Let's understand what each node does:

**Node 1: Form Trigger**
- **What:** Creates a popup web form when you click "Test workflow"
- **Fields:** 
  - File upload (accepts .txt, .doc, .docx)
  - Document name (optional)
  - Collection (defaults to "cisco_docs")
- **Why needed:** User-friendly way to upload files

**When you click "Test workflow":**
```
[Popup Form Appears]
┌─────────────────────────────────┐
│  Upload Document to RAG         │
├─────────────────────────────────┤
│                                 │
│  ðŸ"„ Select Document File       │
│  [Choose File...]               │
│  Accepts: .txt, .doc, .docx     │
│                                 │
│  ðŸ"ï¸ Document Name (optional)   │
│  [_________________________]    │
│                                 │
│  ðŸ"š Collection                 │
│  [cisco_docs â–¼]                │
│                                 │
│  [Submit]                       │
└─────────────────────────────────┘
```

---

**Node 2: Extract File Data**
- **What:** JavaScript code that extracts the uploaded file
- **Why needed:** Form data comes in complex format, need to extract the actual file
- **Does:**
  - Gets file from form submission
  - Extracts filename, file type, file data
  - Validates file was actually uploaded

**Technical detail:**
```javascript
// Simplified version of what it does
const uploadedFile = formData['Select Document File'];
const filename = uploadedFile.fileName;
const mimeType = uploadedFile.mimeType;
const fileData = uploadedFile.data; // The actual file bytes
```

---

**Node 3: Parse Document Content**
- **What:** Extracts text from different file formats
- **Why needed:** .doc and .docx aren't plain text - need special parsing
- **Does:**
  - **.txt files** - Read directly (simple)
  - **.docx files** - Use adm-zip library to extract XML, parse text (medium complexity)
  - **.doc files** - Basic text extraction (best effort - format is complex)

**Quality levels:**
- .txt → âœ…âœ…âœ…âœ… Perfect (100% accuracy)
- .docx → âœ…âœ…âœ… Excellent (95%+ accuracy)
- .doc → âœ…âœ… Good (80-90% accuracy - old format, tricky)

**Example extraction:**
```
Input: sample.docx
Output (extracted text):
"# Network Assessment
Executive Summary
The network consists of..."
```

---

**Node 4: Get Collection UUID**
- **What:** HTTP request to ChromaDB to get all collections
- **Why needed:** ChromaDB v1 API requires UUID, not collection name
- **Does:**
  - GET request to `/api/v1/collections`
  - Receives list of all collections with their UUIDs
  - Next node will extract the specific UUID we need

**Request:**
```http
GET http://chromadb:8000/api/v1/collections
```

**Response:**
```json
[
  {
    "name": "cisco_docs",
    "id": "12345678-1234-1234-1234-123456789abc",
    "metadata": null,
    ...
  }
]
```

---

**Node 5: Process Document**
- **What:** JavaScript code that chunks the document
- **Why needed:** Same chunking logic as Python script
- **Does:**
  - Breaks text into 800-character chunks
  - 100-character overlap between chunks
  - Creates unique chunk IDs
  - Prepares embedding requests

**Output (for each chunk):**
```json
{
  "chunk": "The network consists of...",
  "index": 0,
  "document_id": "doc_20241218_103045",
  "document_name": "network-assessment.txt",
  "collection_uuid": "12345678...",
  "embedding_request": {
    "model": "nomic-embed-text",
    "prompt": "The network consists of..."
  }
}
```

**Important:** This node splits into MULTIPLE outputs - one per chunk.

If document creates 5 chunks → Node outputs 5 items

---

**Node 6: Generate Embeddings**
- **What:** HTTP request to Ollama to create embeddings
- **Why needed:** Convert text chunks to vectors
- **Does:**
  - POST request to Ollama for EACH chunk
  - Receives 768-number vector for each
  - Runs in parallel for efficiency

**For each chunk:**
```http
POST http://host.containers.internal:11434/api/embeddings
{
  "model": "nomic-embed-text",
  "prompt": "The network consists of..."
}
```

**Response:**
```json
{
  "embedding": [0.23, -0.15, 0.87, ..., (765 more numbers)]
}
```

**Note:** Uses `host.containers.internal` because n8n container needs to reach Ollama running on your Mac.

---

**Node 7: Prepare Storage**
- **What:** JavaScript code that formats data for ChromaDB
- **Why needed:** ChromaDB expects specific batch format
- **Does:**
  - Collects ALL chunks and their embeddings
  - Formats into ChromaDB batch structure
  - Creates arrays of IDs, embeddings, documents, metadata

**Output format:**
```json
{
  "ids": ["doc_123_chunk_0", "doc_123_chunk_1", ...],
  "embeddings": [[0.23, -0.15, ...], [0.18, -0.22, ...], ...],
  "documents": ["chunk 0 text", "chunk 1 text", ...],
  "metadatas": [
    {"chunk_id": 0, "document_name": "..."}, 
    {"chunk_id": 1, "document_name": "..."},
    ...
  ]
}
```

This is the exact format ChromaDB's batch add endpoint expects.

---

**Node 8: Store in ChromaDB**
- **What:** HTTP POST request to ChromaDB
- **Why needed:** Actually save the data
- **Does:**
  - POST to `/api/v1/collections/{UUID}/add`
  - Sends all chunks in one request (efficient)
  - Waits for confirmation

**Request:**
```http
POST http://chromadb:8000/api/v1/collections/12345678.../add
Content-Type: application/json

{
  "ids": [...],
  "embeddings": [...],
  "documents": [...],
  "metadatas": [...]
}
```

**Note the URL uses UUID!** This is why we looked it up earlier.

**Response:** 201 Created (success)

---

**Node 9: Success**
- **What:** Formats success message for user
- **Why needed:** User feedback
- **Does:**
  - Shows how many chunks were stored
  - Confirms document name
  - Provides friendly message

**Example output shown to user:**
```
âœ… Success! 5 chunks stored

Document: network-assessment.txt
Document ID: doc_20241218_103045  
Collection: cisco_docs

Message: Document successfully uploaded!
Now searchable in your RAG system.
```

---

### File Format Support Details

**What the workflow can handle:**

**.txt files (Plain Text)**
- **Processing:** Direct read, no parsing needed
- **Quality:** 100% perfect
- **Speed:** Fastest
- **Limitations:** None
- **Best for:** Simple documents, markdown, code

**.docx files (Modern Word)**
- **Processing:** ZIP extraction → XML parsing → text extraction
- **Quality:** 95%+ accurate
- **Speed:** Fast
- **Limitations:** 
  - Complex formatting may be lost
  - Tables converted to text
  - Images ignored
- **Best for:** Most Word documents created after 2007

**.doc files (Legacy Word)**
- **Processing:** Binary format → best-effort text extraction
- **Quality:** 80-90% accurate
- **Speed:** Medium
- **Limitations:**
  - Complex documents may have errors
  - Some formatting may cause issues
  - Tables might not parse well
- **Best for:** Old Word documents when .docx not available

**Recommendation:** Convert .doc → .docx before uploading for best results.

---

### Test the Workflow

**Step 1: Click "Test workflow" button**

Located at the bottom of the n8n interface

A popup form appears (this is the Form Trigger node)

---

**Step 2: Fill the form**

```
ðŸ"„ Select Document File
   [Click to choose...]
   → Navigate to ~/cisco-rag-demo/documents/
   → Select test-network-assessment.md
   → Click Open

ðŸ"ï¸ Document Name (optional)
   [Leave blank or enter "Test Assessment"]

ðŸ"š Collection
   [cisco_docs] (already selected)
```

---

**Step 3: Submit**

Click the **"Submit"** button

**Watch the execution:**
- Nodes light up in green as they execute
- You'll see progress through all 9 nodes
- Each node shows execution time

**Typical timeline:**
```
Node 1: Form Trigger → Instant (form submitted)
Node 2: Extract File → <1 sec (read file)
Node 3: Parse Content → 1-2 sec (extract text)
Node 4: Get UUID → <1 sec (query ChromaDB)
Node 5: Process Doc → 1-2 sec (chunk text)
Node 6: Embeddings → 5-10 sec (Ollama processing)
Node 7: Prepare → <1 sec (format data)
Node 8: Store → 1-2 sec (save to ChromaDB)
Node 9: Success → Instant (display message)

Total: 10-20 seconds
```

---

**Step 4: Verify success**

**Success message appears:**
```
âœ… Success! 4 chunks stored

Document: test-network-assessment.md
Document ID: doc_20241218_103045
Collection: cisco_docs

Document successfully uploaded!
Now searchable in your RAG system.
```

**Check the execution log:**
- All nodes should be green (âœ…)
- No red nodes (âŒ = error)
- Click any node to see its output data

---

### Verify Documents Were Loaded

**Query via Python script:**

```bash
cd ~/cisco-rag-demo
./query_rag.py "What document was just uploaded?"
```

**You should get an answer referencing your uploaded document!**

---

### Common Issues and Solutions

**Issue: Form doesn't appear**
- **Check:** Is workflow saved?
- **Fix:** Click "Save" button, then "Test workflow" again

**Issue: "File parsing failed"**
- **Check:** File format supported?
- **Fix:** Ensure file is .txt, .doc, or .docx (not .pdf!)

**Issue: Node shows error (red)**
- **Check:** Click the red node to see error message
- **Fix:** Most common - Ollama not running
  ```bash
  ollama serve &
  ```

**Issue: Takes very long (>60 seconds)**
- **Check:** Large document? Many chunks?
- **Normal:** 5 chunks = 10-20 sec, 20 chunks = 40-60 sec
- **Fix:** If truly stuck, click "Stop execution" and try again

**Issue: "Collection not found"**
- **Check:** Did you create cisco_docs collection in Part 1?
- **Fix:**
  ```bash
  curl -X POST http://localhost:8000/api/v1/collections \
    -H 'Content-Type: application/json' \
    -d '{"name":"cisco_docs"}'
  ```

---

### Understanding n8n's Network Configuration

**Important concept: Container networking**

**n8n runs in a container, so URLs are different:**

```
From n8n container TO Ollama (on your Mac):
âœ… Use: http://host.containers.internal:11434
âŒ Don't use: http://localhost:11434

From n8n container TO ChromaDB container:
âœ… Use: http://chromadb:8000
Also works: http://host.containers.internal:8000

From your Mac TO n8n:
âœ… Use: http://localhost:5678
```

**Why "host.containers.internal"?**
- Special hostname that Podman provides
- Points to your Mac (the "host" machine)
- Allows containers to access services on the host
- Like a default gateway in networking terms

---

### Success Checkpoint

Before moving to chat interface, verify:

- [ ] Workflow imported successfully
- [ ] Can see all 9 nodes on canvas
- [ ] Test execution completes (all green)
- [ ] Success message appears with correct chunk count
- [ ] Can query uploaded document with Python script
- [ ] You understand what each node does (conceptually)

âœ… **Document upload workflow working! Ready for chat interface.**

---

## Part 2F: n8n Chat Interface

**Time:** 20 minutes  
**What you'll do:** Import a chat interface workflow for interactive Q&A  
**Why:** Makes querying documents visual, interactive, and demo-friendly

---

### Download and Import Chat Workflow

**The chat workflow file:**
`/mnt/project/RAG_-_Query_Chat.json`

**Import process (same as before):**

1. Click **"⋯"** menu → **"Import from File"**
2. Navigate to `/mnt/project/`
3. Select `RAG_-_Query_Chat.json`
4. Click **"Open"**

**Workflow loads on canvas**

**Workflow name:** "RAG - Query Chat (OPTIMIZED)"

---

### Understanding the Chat Workflow Architecture

**Visual representation:**

```
â""â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"¬â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€
   [1] Chat Trigger               
       â"œâ"€ Creates chat interface
       â"œâ"€ Listens for input
       â""â"€ Continuous operation
          â†"
   [2] Get Collection UUID        
       â"œâ"€ Queries ChromaDB
       â""â"€ Gets cisco_docs UUID
          â†"
   [3] Prepare Query              
       â"œâ"€ Extracts question
       â"œâ"€ Validates collection
       â""â"€ Preps embedding request
          â†"
   [4] Generate Question Embedding
       â"œâ"€ Converts question to vector
       â""â"€ Uses Ollama
          â†"
   [5] Prepare Search             
       â"œâ"€ Formats search request
       â""â"€ Sets n_results=3
          â†"
   [6] Search ChromaDB            
       â"œâ"€ Vector similarity search
       â"œâ"€ Returns top 3 chunks
       â""â"€ With similarity scores
          â†"
   [7] Build Prompt               
       â"œâ"€ Combines chunks + question
       â"œâ"€ Adds system instructions
       â""â"€ Creates full prompt
          â†"
   [8] Generate Answer            
       â"œâ"€ Sends to Ollama LLM
       â"œâ"€ AI writes answer
       â""â"€ Returns response
          â†"
   [9] Format Response            
       â"œâ"€ Cleans up text
       â"œâ"€ Formats for chat
       â""â"€ Sends back to user
â""â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"´â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€â"€
```

**Key differences from document loader:**
- This runs **continuously** (always listening)
- Document loader runs **once per upload**
- Chat processes **user input** in real-time
- Chat must **respond** to same interface

---

### Node-by-Node Explanation (Chat Workflow)

**Node 1: Chat Trigger**
- **Type:** Special n8n node (`@n8n/n8n-nodes-langchain.chatTrigger`)
- **What:** Creates interactive chat window
- **Provides:** User's typed question as `$json.chatInput`
- **Runs:** Continuously (listens for chat messages)

**What the chat interface looks like:**
```
┌─────────────────────────────────────┐
│  Chat with RAG System               │
├─────────────────────────────────────┤
│                                     │
│  You: What is the network uptime?   │
│                                     │
│  AI: According to the network       │
│      assessment, Acme Corporation's │
│      Boston office network has...   │
│                                     │
│  [Type your question here...]       │
└─────────────────────────────────────┘
```

---

**Node 2: Get Collection UUID**
- **What:** HTTP GET to ChromaDB collections
- **Why:** Need UUID for searching (same reason as document loader)
- **Note:** Happens on EVERY query (ensures collection still exists)

**Network analogy:** Like ARP request before sending packet - verify destination exists.

---

**Node 3: Prepare Query**
- **What:** JavaScript code that validates and prepares query
- **Does:**
  - Extracts question from chat input
  - Finds UUID in collection list
  - Validates collection exists
  - Prepares embedding request

**Code logic:**
```javascript
// Get question from chat
const question = $('Chat Trigger').first().json.chatInput;

// Find UUID
for (const coll of collections) {
  if (coll.name === 'cisco_docs') {
    collectionUuid = coll.id;
  }
}

// Prepare for embedding
return {
  collection_uuid: collectionUuid,
  question: question,
  embedding_request: {
    model: 'nomic-embed-text',
    prompt: question
  }
};
```

---

**Node 4: Generate Question Embedding**
- **What:** POST to Ollama embeddings endpoint
- **Why:** Convert question to vector (same space as document chunks)
- **Important:** Uses same model (nomic-embed-text) as document chunks

**Request:**
```http
POST http://host.containers.internal:11434/api/embeddings
{
  "model": "nomic-embed-text",
  "prompt": "What is the network uptime?"
}
```

**Response:**
```json
{
  "embedding": [0.31, -0.18, 0.92, ... (768 numbers total)]
}
```

**Why same model matters:** 
- Document chunks → nomic-embed-text → vectors
- Question → nomic-embed-text → vector
- Vectors are in same "space" → comparable
- Like using same coordinate system for two GPS points

---

**Node 5: Prepare Search**
- **What:** Formats ChromaDB search request
- **Does:**
  - Takes question embedding from previous node
  - Formats as ChromaDB query
  - Sets n_results=3 (return top 3 chunks)

**Output:**
```json
{
  "collection_uuid": "12345678...",
  "question": "What is the network uptime?",
  "search_request": {
    "query_embeddings": [[0.31, -0.18, 0.92, ...]],
    "n_results": 3
  }
}
```

---

**Node 6: Search ChromaDB**
- **What:** POST to ChromaDB query endpoint
- **Does:**
  - Sends question vector
  - ChromaDB calculates similarities
  - Returns top 3 closest chunks

**Request:**
```http
POST http://chromadb:8000/api/v1/collections/{UUID}/query
{
  "query_embeddings": [[0.31, -0.18, ...]],
  "n_results": 3
}
```

**Response:**
```json
{
  "documents": [
    ["Network uptime is 99.94%...", "The mean time...", "All incidents..."]
  ],
  "distances": [[0.108, 0.155, 0.177]],
  "metadatas": [[{...}, {...}, {...}]]
}
```

**Distances explained:**
- Lower distance = More similar
- 0.108 → Very similar (0.892 similarity score)
- Distance converted to similarity: similarity = 1 - distance

---

**Node 7: Build Prompt**
- **What:** JavaScript code that constructs AI prompt
- **Does:**
  - Takes retrieved chunks
  - Formats them nicely
  - Adds system instructions
  - Combines with user question

**Prompt structure:**
```
System: You are a helpful AI assistant analyzing Cisco 
        network assessment documents. Provide accurate, 
        concise answers based on context provided.

Context from documents:
[Source 1]
Network uptime is 99.94% year-to-date...

[Source 2]
Mean time to repair is 2.4 hours...

[Source 3]
All incidents were resolved within 4 hours...

User question: What is the network uptime?

Your answer:
```

**Why this format?**
- System instructions guide AI behavior
- Sources labeled for clarity
- Question clearly stated
- "Your answer:" prompts AI to respond

---

**Node 8: Generate Answer**
- **What:** POST to Ollama generate endpoint
- **Does:**
  - Sends complete prompt
  - AI model (llama3.2:3b) reads and writes answer
  - Returns generated text

**Request:**
```http
POST http://host.containers.internal:11434/api/generate
{
  "model": "llama3.2:3b",
  "prompt": "[Full prompt from Node 7]",
  "stream": false
}
```

**Response:**
```json
{
  "model": "llama3.2:3b",
  "response": "According to the network assessment, Acme Corporation's Boston office network has a year-to-date uptime of 99.94%, which exceeds the SLA requirement of 99.9%. The mean time to repair is 2.4 hours.",
  "done": true
}
```

**Why this takes 5-10 seconds:**
- AI reading full prompt (chunks + question)
- Generating human-friendly response
- Writing 50-200 words typically

---

**Node 9: Format Response**
- **What:** Final JavaScript code
- **Does:**
  - Extracts just the answer text
  - Trims whitespace
  - Formats for chat interface
  - Returns as `{output: "text"}` (required by Chat Trigger)

**Critical format:**
```javascript
// Chat Trigger requires this exact format:
return [{
  json: {
    output: cleanAnswer  // Must be named "output"
  }
}];
```

**If format wrong:** Chat won't display response.

---

### UUID Handling in Chat vs Document Loader

**Key architectural difference:**

**Document Loader:**
- Gets UUID once at start
- Uses same UUID for all chunks
- One-time lookup

**Chat:**
- Gets UUID on EVERY query
- Ensures collection exists each time
- Handles edge cases (collection deleted, recreated, etc.)

**Why this design?**
- Chat runs continuously for hours/days
- Collection might change during that time
- Safe to verify on each query
- Minimal performance impact (< 1 second)

**Network analogy:** 
- Document loader = Static route (set once)
- Chat = Dynamic routing (verify each packet)

---

### Activate the Workflow

**Important:** Chat must be **activated** to work (unlike document loader which runs on-demand).

**Activation steps:**

1. **Save the workflow first**
   - Click the **"Save"** button

2. **Toggle to Active**
   - Look for the **"Active" toggle** switch
   - Currently shows as ⭕ (inactive/off)
   - Click it → Changes to âœ… (active/on)

3. **Confirm it's active**
   - Switch should be green/blue
   - Workflow name shows **"Active"** badge
   - Workflow will now process chat messages

**What "Active" means:**
- Workflow runs in background continuously
- Chat Trigger listens for messages
- Processes queries in real-time
- Stays active even if you close browser

**To deactivate:**
- Click toggle again → ⭕
- Workflow stops processing
- Chat button disappears

---

### Open the Chat Interface

**With workflow active:**

1. **Look for Chat button**
   - Top toolbar in n8n
   - Shows **"💬 Chat"** button
   - Appears only when chat workflow is active

2. **Click the Chat button**
   - New window/tab opens
   - Shows chat interface
   - Ready to receive questions

**Chat interface opens:**
```
┌───────────────────────────────────────────┐
│  RAG System Chat                          │
├───────────────────────────────────────────┤
│                                           │
│  ðŸ'¬ Ask me anything about your documents │
│                                           │
│  [Type your question here...]             │
│  [Send â†']                                 │
└───────────────────────────────────────────┘
```

---

### Ask Questions in Chat

**Try these questions:**

**Simple factual:**
```
You: What is the network uptime?

AI: According to the network assessment, Acme 
    Corporation's Boston office network has a 
    year-to-date uptime of 99.94%, which exceeds 
    the SLA requirement of 99.9%. The mean time 
    to repair is 2.4 hours.

⏱ Response time: 5-8 seconds
```

---

**Budget inquiry:**
```
You: What is the proposed budget?

AI: The FY2025 Network Upgrade Budget totals 
    $325,000, broken down as follows:
    - Equipment: $250,000
    - Installation: $50,000
    - Licensing: $25,000
    
    This budget has been approved.

⏱ Response time: 6-9 seconds
```

---

**Equipment question:**
```
You: What equipment needs replacement?

AI: 15 Cisco 3850 switches need replacement as 
    they reach end-of-support in October 2025. 
    The recommended replacement is the Cisco 
    Catalyst 9300 series, with an estimated cost 
    of $187,500.

⏱ Response time: 5-8 seconds
```

---

**Analytical question:**
```
You: Does the network meet SLA requirements?

AI: Yes, the network exceeds SLA requirements. 
    The current uptime of 99.94% surpasses the 
    required 99.9% SLA target, demonstrating 
    excellent reliability.

⏱ Response time: 7-10 seconds
```

---

**Contact info:**
```
You: Who should I contact about this project?

AI: For this project, you should contact:
    - Project Manager: Sarah Johnson 
      (sarah.johnson@acme.com)
    - Network Lead: Mike Chen 
      (mike.chen@acme.com)
    - Budget Approval: Jane Smith, CFO

⏱ Response time: 6-9 seconds
```

---

### Understanding Chat Responses

**What influences answer quality:**

**Good answers when:**
- âœ… Information is IN the uploaded documents
- âœ… Question is specific and focused
- âœ… Relevant chunks are retrieved (high similarity)
- âœ… Context is sufficient to answer

**Weaker answers when:**
- ⚠️ Information not in documents (AI may say "not found in context")
- ⚠️ Question too broad or vague
- ⚠️ Retrieved chunks not relevant (low similarity)
- ⚠️ Multiple unrelated topics in one question

**Example - Good question:**
```
âœ… "What is the network uptime?"
- Specific
- Likely in assessment document
- Single topic
- Measurable answer
```

**Example - Weaker question:**
```
⚠️ "Tell me everything about the network, budget, and recommendations"
- Too broad
- Multiple topics
- Hard to answer concisely
- May miss important details
```

**Best practice:** Ask focused, specific questions. If you need multiple topics, ask multiple questions.

---

### Response Time Expectations

**Typical performance:**

| Scenario | Time | Reason |
|----------|------|--------|
| First chat question | 10-15 sec | AI model loading into RAM |
| Subsequent questions | 5-8 sec | Model already in RAM |
| Simple question | 4-6 sec | Shorter answer to generate |
| Complex question | 8-12 sec | Longer answer to generate |
| After 5 min idle | 10-15 sec | Model may have unloaded |

**What happens during those 5-8 seconds:**

```
Second 0: You press Enter
Second 1: Question embedding generated (Ollama)
Second 2: ChromaDB search completes
Second 3: Prompt built and sent to AI
Seconds 3-7: AI reading context and generating answer
Second 8: Answer returned and displayed
```

**The longest part:** Answer generation (60-70% of time)

**Why?** AI model has to:
1. Read full prompt (question + 3 chunks = ~1000 words)
2. Understand context
3. Formulate answer
4. Generate text word-by-word
5. Ensure accuracy

---

### Chat Workflow vs Python Script

**Comparison:**

| Feature | Chat Workflow | Python Script |
|---------|---------------|---------------|
| **Interface** | Visual chat window | Command line |
| **Best for** | Demos, presentations | Testing, automation |
| **Activation** | Must be "Active" | Run on demand |
| **Multi-user** | Could support multiple | Single user |
| **History** | Shown in chat | Not saved |
| **Speed** | Same (5-10 sec) | Same (5-10 sec) |
| **Debugging** | Node-by-node | Python errors |

**When to use each:**

**Use Chat Workflow:**
- Demonstrating to stakeholders
- Interactive Q&A sessions
- Non-technical user access
- "Wow factor" presentations

**Use Python Script:**
- Quick verification tests
- Automated batch queries
- Integration with other scripts
- When chat workflow isn't running

**Both use the same underlying RAG system** - just different interfaces!

---

### Success Checkpoint

Before moving to final verification, confirm:

- [ ] Chat workflow imported successfully
- [ ] Workflow activated (toggle shows âœ…)
- [ ] Chat button appears in n8n toolbar
- [ ] Chat window opens
- [ ] Can ask questions and get accurate answers
- [ ] Response times are acceptable (<15 sec)
- [ ] Answers cite document content correctly
- [ ] You understand the 9-node flow (conceptually)

âœ… **Chat interface working! Ready for comprehensive testing.**

---

## Part 2G: System Verification & Testing

**Time:** 10 minutes  
**What you'll do:** Complete end-to-end testing of your RAG system  
**Why:** Ensure everything works together reliably

---

### Complete Test Sequence

**Test 1: Upload via n8n, Query via Python**

**Purpose:** Verify document upload workflow and Python query work together

```bash
# 1. Upload a new document via n8n form
# (Open n8n, click "Test workflow" on Document Upload workflow)
# Upload a different document

# 2. Query it via Python
cd ~/cisco-rag-demo
./query_rag.py "What is in the new document?"
```

**Expected result:** Python script finds and answers questions about the newly uploaded document.

âœ… **Confirms:** Upload workflow successfully stored document in ChromaDB.

---

**Test 2: Upload via Python, Query via Chat**

**Purpose:** Verify Python script and chat workflow work together

```bash
# 1. Create a new test document
cat > ~/cisco-rag-demo/documents/test-policy.md << 'EOF'
# IT Security Policy

## Password Requirements
All passwords must be:
- At least 12 characters
- Include uppercase, lowercase, numbers, symbols
- Changed every 90 days

## Access Control
Network access is granted on role-based basis.
Contractors receive temporary 30-day access.

## Data Backup
Critical data backed up daily to encrypted storage.
Retention period: 7 years for compliance.
EOF

# 2. Load via Python
python3 load_document.py documents/test-policy.md

# 3. Query via Chat
# (Open n8n chat interface)
# Ask: "What are the password requirements?"
```

**Expected result:** Chat interface answers with password policy details.

âœ… **Confirms:** Python loader and chat workflow both access same ChromaDB collection.

---

**Test 3: Multiple Documents**

**Purpose:** Verify system handles multiple documents simultaneously

```bash
# Upload 2-3 different documents
python3 load_document.py documents/test-network-assessment.md
python3 load_document.py documents/test-policy.md

# Query about specific documents
./query_rag.py "What is the network uptime?"  # Should get network assessment answer
./query_rag.py "What are password rules?"     # Should get policy answer
```

**Expected result:** System finds correct information from correct document.

âœ… **Confirms:** RAG can distinguish between different documents and retrieve appropriate context.

---

**Test 4: File Format Support**

**Purpose:** Verify .txt, .docx, .doc support

**If you have test files:**

```
# Upload .txt file via n8n
→ Should process quickly, 100% accuracy

# Upload .docx file via n8n  
→ Should extract text correctly

# Upload .doc file via n8n
→ Should extract text (may have minor issues)
```

**Expected results:**
- All formats process without errors
- Text extraction quality: txt > docx > doc
- All become queryable

âœ… **Confirms:** File format handling works as designed.

---

**Test 5: Error Handling**

**Purpose:** Verify system handles problems gracefully

**Test scenarios:**

**Scenario A: Query before loading documents**
```bash
# In fresh collection:
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"test_empty"}'

# Try to query (should fail gracefully)
# Modify script to use "test_empty" collection
```

**Expected:** Error message "No documents found, load documents first"

---

**Scenario B: Invalid file format**
```
# Try to upload .pdf via n8n
→ Form validation prevents it (only allows .txt, .doc, .docx)
```

**Expected:** Cannot even select .pdf file

---

**Scenario C: Ollama service stopped**
```bash
# Stop Ollama temporarily
killall ollama

# Try to query or upload
→ Should show connection error
```

**Expected:** Clear error message about Ollama connectivity

**Fix:**
```bash
ollama serve &
# Try again, should work
```

---

### Performance Benchmarks

**Document your system's performance:**

**Upload performance:**
```
Document size: _____ characters
Number of chunks: _____
Upload time: _____ seconds
Embedding generation: _____ sec per chunk
```

**Typical values:**
- 2KB document (4 chunks) → 15-20 seconds
- 5KB document (10 chunks) → 35-45 seconds
- 10KB document (20 chunks) → 60-80 seconds

**Query performance:**
```
First query: _____ seconds
Subsequent queries: _____ seconds
Complex questions: _____ seconds
```

**Typical values:**
- First query: 10-15 seconds (model loading)
- Subsequent: 5-8 seconds (model in RAM)
- Complex: 8-12 seconds (longer answer generation)

---

### System Health Check

**Run comprehensive check:**

```bash
# 1. Check all containers
podman ps --format "{{.Names}}: {{.Status}}"

# 2. Check Ollama models
ollama list

# 3. Check ChromaDB collections
curl -s http://localhost:8000/api/v1/collections | \
  python3 -c "import sys, json; [print(f\"  - {c['name']}\") for c in json.load(sys.stdin)]"

# 4. Check document count
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"

# 5. Test query
./query_rag.py "test query"
```

**All should return success!**

---

### What You've Built - Summary

**Two Upload Methods:**

âœ… **Python Script** (`load_document.py`)
- Command-line based
- Fast and direct
- Good for automation
- Handles .txt and .md files

âœ… **n8n Workflow** (Form upload)
- Web form interface
- Visual and user-friendly
- Good for demos
- Handles .txt, .doc, .docx files

---

**Two Query Methods:**

âœ… **Python Script** (`query_rag.py`)
- Command-line based
- Quick testing
- Automation-friendly
- Shows detailed process

âœ… **n8n Chat** (Interactive interface)
- Visual chat window
- Real-time conversation
- Demo-friendly
- Non-technical user friendly

---

**Complete RAG Pipeline:**

```
Documents
    â†"
Your Choice: Python or n8n Upload
    â†"
Chunking (800 chars, 100 overlap)
    â†"
Embedding (Ollama nomic-embed-text)
    â†"
Storage (ChromaDB with UUIDs)
    â†"
Your Questions
    â†"
Your Choice: Python or n8n Chat
    â†"
Search (Vector similarity)
    â†"
Retrieved Chunks (Top 3)
    â†"
Prompt Building (Chunks + Question)
    â†"
Answer Generation (Ollama llama3.2:3b)
    â†"
Response to You
```

**Everything runs locally. No data leaves your Mac. Fully private.**

---

### Final Success Checklist

**Environment (from Part 1):**
- [ ] Podman running with containers
- [ ] Ollama with both models
- [ ] ChromaDB accessible
- [ ] n8n interface accessible
- [ ] Python environment ready

**Python Components (this guide):**
- [ ] load_document.py works
- [ ] query_rag.py works
- [ ] Documents load successfully
- [ ] Queries return accurate answers

**n8n Workflows (this guide):**
- [ ] Document upload workflow imported
- [ ] Document upload workflow tested
- [ ] Chat workflow imported
- [ ] Chat workflow activated
- [ ] Chat interface accessible
- [ ] Chat answers accurately

**System Integration:**
- [ ] Can upload via n8n, query via Python
- [ ] Can upload via Python, query via chat
- [ ] Multiple documents work correctly
- [ ] Different file formats supported
- [ ] Error handling works gracefully
- [ ] Performance is acceptable

**âœ…âœ…âœ… If all boxes checked: Your RAG system is PRODUCTION READY!**

---

## Troubleshooting

### Common Issues and Solutions

**Issue: "Collection not found" error**

**Symptoms:**
- Python script or n8n workflow can't find cisco_docs
- Error message mentions collection doesn't exist

**Diagnostic:**
```bash
curl http://localhost:8000/api/v1/collections
```

**If empty `[]`:**
```bash
# Create collection
curl -X POST http://localhost:8000/api/v1/collections \
  -H 'Content-Type: application/json' \
  -d '{"name":"cisco_docs"}'
```

---

**Issue: Embeddings fail - can't connect to Ollama**

**Symptoms:**
- "Connection refused" errors
- Timeout when generating embeddings

**Diagnostic:**
```bash
curl http://localhost:11434
```

**If connection refused:**
```bash
# Start Ollama
ollama serve &

# Wait a moment
sleep 5

# Verify
curl http://localhost:11434
# Should return: "Ollama is running"
```

---

**Issue: n8n workflow shows red error nodes**

**Diagnostic:**
- Click the red node
- Read error message in panel

**Common errors:**

**"host.containers.internal not found"**
- Wrong URL in node
- Should use: `http://host.containers.internal:11434`
- Check HTTP Request nodes

**"Collection UUID not found"**
- Collection doesn't exist
- Create it (see above)

**"Embedding timeout"**
- Ollama not responding
- Check Ollama is running
- Increase timeout in node (default 30 sec)

---

**Issue: Chat workflow activated but no Chat button**

**Solutions:**
1. **Refresh browser** - Sometimes UI doesn't update
2. **Deactivate and reactivate** - Toggle off, save, toggle on
3. **Check workflow type** - Must use Chat Trigger node
4. **Restart n8n** - `podman restart n8n`, wait 20 sec

---

**Issue: Queries return irrelevant answers**

**Possible causes:**

**1. Wrong documents retrieved**
```bash
# Check what's in collection
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")

curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
# Should match number of chunks you expect
```

**2. Question too vague**
- Try more specific questions
- Focus on one topic per question

**3. Information not in documents**
- Verify document actually contains that information
- Re-upload if needed

---

**Issue: Very slow performance (>20 seconds per query)**

**Diagnostic steps:**

**1. Check RAM usage**
```bash
# Open Activity Monitor
# Look at "Memory Pressure"
```

**If pressure is high (yellow/red):**
- Close other applications
- Restart Mac to free memory

**2. Pre-load models**
```bash
# Keep models in RAM
ollama run llama3.2:3b "test"
# Press Ctrl+D
```

**3. Increase Podman resources** (if you have RAM):
```bash
podman machine stop
podman machine set --memory 16384  # 16GB
podman machine start
```

---

**Issue: Document upload fails with parsing error**

**For .doc files:**
- Try converting to .docx first (better support)
- Use: Pages, Word, or online converter

**For .docx files:**
- File might be corrupted
- Try opening and re-saving
- Check file isn't password-protected

**For all files:**
- Ensure file isn't empty
- Check file size (< 5MB recommended)
- Verify correct file extension

---

### Getting Additional Help

**Before asking for help, collect:**

1. **Error messages** (full text, not paraphrased)
2. **What you were doing** (exact steps)
3. **System check output:**
   ```bash
   ~/cisco-rag-demo/system-check.sh > system-status.txt
   ```
4. **Container logs:**
   ```bash
   podman logs chromadb > chromadb.log
   podman logs n8n > n8n.log
   ```
5. **What you tried** (attempted fixes)

**Resources:**
- Part 1 Troubleshooting (Environment issues)
- Comprehensive Troubleshooting Guide (detailed solutions)
- Project documentation (specific to your setup)

---

## Next Steps

ðŸŽ‰ **Congratulations!** You've built a complete RAG system!

**What you've accomplished:**
- âœ… Understand RAG concepts deeply
- âœ… Built document upload system (2 methods)
- âœ… Built query system (2 methods)
- âœ… Created visual workflows in n8n
- âœ… Everything runs locally and privately

**You now have:**
- Working Python scripts for automation
- Visual n8n workflows for demonstrations
- Interactive chat interface for users
- Fully local AI document assistant

---

### Ready for Part 3: Webex Integration

**In the next guide, you'll:**
- Connect your RAG system to Webex Teams
- Create a bot that responds to messages
- Enable team members to query documents via Webex
- Build enterprise messaging integration

**Time required:** ~45 minutes  
**Prerequisites:** Completed Parts 1 and 2 (âœ…)

**âž¡ï¸ Continue to [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)**

---

### Alternative Next Steps

**Want to explore more first?**

**Option 1: Experiment with your RAG system**
- Upload more documents
- Try different question types
- Test performance limits
- Customize chunk sizes (advanced)

**Option 2: Learn system internals**
- Review [LESSONS_LEARNED_Troubleshooting_Chronicle.md](LESSONS_LEARNED_Troubleshooting_Chronicle.md)
- Understand ChromaDB v1 API details
- Study n8n workflow architecture
- Read about vector embeddings

**Option 3: Prepare demonstration materials**
- Create sample documents for demos
- Prepare compelling questions
- Practice explanations
- Plan presentation flow

**You can always return to Part 3 when ready!**

---

## Document Information

**Version:** 1.0.0  
**Last Updated:** December 18, 2024  
**Part of:** Local RAG System Implementation Guides  
**Previous Guide:** [GUIDE_1_ENVIRONMENT_SETUP.md](GUIDE_1_ENVIRONMENT_SETUP.md)  
**Next Guide:** [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)

**This guide covered:**
- RAG system concepts and architecture
- Python script implementation
- Document loading and chunking
- Vector embeddings and search
- n8n workflow automation
- Interactive chat interface
- Complete system testing

**Feedback:** Encountered issues not covered here? Document them for future updates.

---

**âž¡ï¸ Ready for enterprise integration? Proceed to [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)**
