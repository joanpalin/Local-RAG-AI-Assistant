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

**You should see all green checkmarks (✓):**
- Podman containers running (chromadb, n8n)
- Ollama models present (llama3.2:3b, nomic-embed-text)
- All services accessible
- cisco_docs collection exists

**If anything shows ✗:** Return to Part 1 and fix issues before continuing.

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
  ↓
Split into Chunks (small pieces, ~800 characters each)
  ↓
Convert to Vectors (numbers that represent meaning)
  ↓
Store in ChromaDB (searchable database)


Step 2: ASK A QUESTION (Every Query)
──────────────────────────────────────
Your Question: "What's the network uptime?"
  ↓
Convert to Vector (same process as documents)
  ↓
Search ChromaDB for Similar Vectors
  ↓
Find 3 Most Relevant Chunks


Step 3: BUILD CONTEXT (Behind the Scenes)
───────────────────────────────────────────
Retrieved Chunks + Your Question
  ↓
Combine into a Single Prompt
  ↓
Send to AI Model


Step 4: GENERATE ANSWER (The Magic)
──────────────────────────────────────
AI Reads the Context
  ↓
Writes Human-Friendly Answer
  ↓
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
Match: ✗ NO - doesn't contain word "uptime"
```

**Vector search (semantic matching):**
```
Question: "What's our uptime?"
  → Vector: [0.23, -0.15, 0.87, ...]

Document chunk: "The network availability was 99.94%"
  → Vector: [0.25, -0.14, 0.85, ...]

Distance calculation: CLOSE! (similar meaning)
Match: ✓ YES - semantically similar
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
| **Privacy** | ✓ Docs never leave your machine | ✗ Uploaded to external servers |
| **Cost** | ✓ Free after setup | ✗ $0.002-0.06 per 1K tokens |
| **Compliance** | ✓ HIPAA/SOC2 friendly | ✗ May violate policies |
| **Customization** | ✓ Your docs, your rules | ✗ Generic knowledge |
| **Offline** | ✓ Works without internet | ✗ Requires connection |
| **Speed** | ✓ 5-10 seconds | ✓ 2-5 seconds (faster) |
| **Accuracy** | ✓ Cites your documents | ⚠️ May hallucinate |
| **Knowledge** | ⚠️ Only your documents | ✓ Knows everything |

**Best use cases for local RAG:**
- Confidential company documents
- Medical records (HIPAA)
- Financial data (SOC2)
- Legal documents (attorney-client)
- Proprietary research
- Offline demonstrations

---

### Visual RAG Architecture

```
┌─────────────────────────────────────────────────────────────-┐
│                    YOUR RAG SYSTEM                           │
├─────────────────────────────────────────────────────────────-┤
│                                                              │
│  1. DOCUMENT INGESTION (One-time setup)                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │
│  │  Document    │───>│  Chunking    │───>│  Embedding   │    │
│  │  Upload      │    │  (800 chars) │    │  (Ollama)    │    │
│  └──────────────┘    └──────────────┘    └──────┬───────┘    │
│                                                   │          │
│                                                   ↓          │
│                                         ┌──────────────────┐ │
│                                         │  ChromaDB        │ │
│                                         │  Vector Storage  │ │
│                                         └──────────────────┘ │
│                                                              │
│  2. QUERY PROCESSING (Every question)                        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │
│  │  Your        │───>│  Embedding   │───>│  Vector      │    │
│  │  Question    │    │  (Ollama)    │    │  Search      │    │
│  └──────────────┘    └──────────────┘    └──────┬───────┘    │
│                                                   │          │
│                                                   ↓          │
│                                         ┌──────────────────┐ │
│                                         │  Top 3 Chunks    │ │
│                                         │  Retrieved       │ │
│                                         └─────┬────────────┘ │
│                                               │              │
│                                               ↓              │
│  3. ANSWER GENERATION                                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Question + Retrieved Chunks → Ollama (llama3.2)     │    │
│  │                                  ↓                   │    │
│  │                            Generated Answer          │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────-┘
```

---

## Part 2B: Python Script Setup

**Time:** 10 minutes  
**What you'll do:** Create Python scripts for document loading and querying

### Why Python Scripts?

**Python scripts are useful for:**
- Quick testing and debugging
- Automation and batch processing
- Understanding how the system works
- Creating custom tools

**Don't worry if you're not a Python expert** - we provide complete, working scripts that you can copy/paste.

---

### Script 1: Document Loader

**Action: Create the document loading script**

```bash
cat > ~/cisco-rag-demo/scripts/load_document.py << 'EOF'
#!/usr/bin/env python3
"""
Document Loader for RAG System
Loads documents into ChromaDB with proper chunking and embedding
"""
import requests
import sys
import uuid
from pathlib import Path

# Configuration
OLLAMA_URL = "http://localhost:11434"
CHROMA_URL = "http://localhost:8000/api/v1"
COLLECTION_NAME = "cisco_docs"
CHUNK_SIZE = 800
CHUNK_OVERLAP = 100

def get_collection_id():
    """Get the UUID of the cisco_docs collection"""
    try:
        response = requests.get(f"{CHROMA_URL}/collections")
        collections = response.json()
        
        for collection in collections:
            if collection['name'] == COLLECTION_NAME:
                return collection['id']
        
        print(f"Error: Collection '{COLLECTION_NAME}' not found")
        print("Create it first with:")
        print(f'curl -X POST {CHROMA_URL}/collections -H "Content-Type: application/json" -d \'{{"name":"{COLLECTION_NAME}"}}\'')
        return None
    except Exception as e:
        print(f"Error getting collection: {e}")
        return None

def chunk_text(text, chunk_size=CHUNK_SIZE, overlap=CHUNK_OVERLAP):
    """
    Split text into overlapping chunks
    
    Args:
        text: Input text to chunk
        chunk_size: Size of each chunk
        overlap: Number of overlapping characters between chunks
    
    Returns:
        List of text chunks
    """
    chunks = []
    start = 0
    
    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end]
        chunks.append(chunk)
        start = end - overlap  # Move back by overlap amount
    
    return chunks

def generate_embedding(text):
    """
    Generate embedding vector for text using Ollama
    
    Args:
        text: Text to embed
    
    Returns:
        List of floats (embedding vector)
    """
    try:
        response = requests.post(
            f"{OLLAMA_URL}/api/embeddings",
            json={
                "model": "nomic-embed-text",
                "prompt": text
            },
            timeout=30
        )
        
        if response.status_code == 200:
            return response.json()['embedding']
        else:
            print(f"Error generating embedding: {response.status_code}")
            return None
    except Exception as e:
        print(f"Error connecting to Ollama: {e}")
        return None

def load_document(filepath, source_name=None):
    """
    Load a document into ChromaDB
    
    Args:
        filepath: Path to document file
        source_name: Optional custom source name
    """
    # Get collection ID
    collection_id = get_collection_id()
    if not collection_id:
        return False
    
    # Read file
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            text = f.read()
    except Exception as e:
        print(f"Error reading file: {e}")
        return False
    
    # Use filename as source if not provided
    if source_name is None:
        source_name = Path(filepath).name
    
    print(f"\nLoading document: {source_name}")
    print(f"Total characters: {len(text)}")
    
    # Chunk the text
    chunks = chunk_text(text)
    print(f"Created {len(chunks)} chunks")
    
    # Process each chunk
    successful = 0
    for i, chunk in enumerate(chunks, 1):
        print(f"Processing chunk {i}/{len(chunks)}...", end=' ')
        
        # Generate embedding
        embedding = generate_embedding(chunk)
        if embedding is None:
            print("FAILED")
            continue
        
        # Generate unique ID for this chunk
        chunk_id = str(uuid.uuid4())
        
        # Add to ChromaDB
        try:
            response = requests.post(
                f"{CHROMA_URL}/collections/{collection_id}/add",
                json={
                    "ids": [chunk_id],
                    "embeddings": [embedding],
                    "documents": [chunk],
                    "metadatas": [{
                        "source": source_name,
                        "chunk_index": i,
                        "total_chunks": len(chunks)
                    }]
                }
            )
            
            if response.status_code == 201:
                print("✓")
                successful += 1
            else:
                print(f"FAILED ({response.status_code})")
        except Exception as e:
            print(f"ERROR: {e}")
    
    print(f"\nCompleted: {successful}/{len(chunks)} chunks loaded successfully")
    return successful == len(chunks)

def main():
    if len(sys.argv) < 2:
        print("Usage: python3 load_document.py <filepath> [source_name]")
        print("\nExample:")
        print("  python3 load_document.py ~/Documents/network_assessment.txt")
        print("  python3 load_document.py report.txt 'Q3 Network Report'")
        return 1
    
    filepath = sys.argv[1]
    source_name = sys.argv[2] if len(sys.argv) > 2 else None
    
    # Verify file exists
    if not Path(filepath).exists():
        print(f"Error: File not found: {filepath}")
        return 1
    
    # Load the document
    success = load_document(filepath, source_name)
    return 0 if success else 1

if __name__ == "__main__":
    sys.exit(main())
EOF

chmod +x ~/cisco-rag-demo/scripts/load_document.py
```

**What this script does:**
1. Reads your document file
2. Splits it into 800-character chunks with 100-character overlap
3. Generates embeddings for each chunk using Ollama
4. Stores everything in ChromaDB with metadata

---

### Script 2: Query System

**Action: Create the query script**

```bash
cat > ~/cisco-rag-demo/scripts/query_rag.py << 'EOF'
#!/usr/bin/env python3
"""
RAG Query System
Ask questions and get answers from your documents
"""
import requests
import sys
import json

# Configuration
OLLAMA_URL = "http://localhost:11434"
CHROMA_URL = "http://localhost:8000/api/v1"
COLLECTION_NAME = "cisco_docs"
NUM_RESULTS = 3

def get_collection_id():
    """Get the UUID of the cisco_docs collection"""
    try:
        response = requests.get(f"{CHROMA_URL}/collections")
        collections = response.json()
        
        for collection in collections:
            if collection['name'] == COLLECTION_NAME:
                return collection['id']
        
        print(f"Error: Collection '{COLLECTION_NAME}' not found")
        return None
    except Exception as e:
        print(f"Error getting collection: {e}")
        return None

def generate_embedding(text):
    """Generate embedding for query"""
    try:
        response = requests.post(
            f"{OLLAMA_URL}/api/embeddings",
            json={
                "model": "nomic-embed-text",
                "prompt": text
            },
            timeout=30
        )
        
        if response.status_code == 200:
            return response.json()['embedding']
        else:
            print(f"Error generating embedding: {response.status_code}")
            return None
    except Exception as e:
        print(f"Error connecting to Ollama: {e}")
        return None

def search_documents(query, collection_id):
    """Search for relevant document chunks"""
    print(f"Searching for: {query}")
    
    # Generate query embedding
    query_embedding = generate_embedding(query)
    if query_embedding is None:
        return None
    
    # Search ChromaDB
    try:
        response = requests.post(
            f"{CHROMA_URL}/collections/{collection_id}/query",
            json={
                "query_embeddings": [query_embedding],
                "n_results": NUM_RESULTS
            }
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error searching: {response.status_code}")
            return None
    except Exception as e:
        print(f"Error querying ChromaDB: {e}")
        return None

def generate_answer(query, context_chunks):
    """Generate answer using Ollama with retrieved context"""
    # Build context from chunks
    context = "\n\n---\n\n".join(context_chunks)
    
    # Create prompt
    prompt = f"""Based on the following context from documents, answer the question. 
If the answer is not in the context, say "I don't have enough information to answer that."

Context:
{context}

Question: {query}

Answer:"""
    
    print("\nGenerating answer...")
    
    try:
        response = requests.post(
            f"{OLLAMA_URL}/api/generate",
            json={
                "model": "llama3.2:3b",
                "prompt": prompt,
                "stream": False
            },
            timeout=60
        )
        
        if response.status_code == 200:
            return response.json()['response']
        else:
            print(f"Error generating answer: {response.status_code}")
            return None
    except Exception as e:
        print(f"Error connecting to Ollama: {e}")
        return None

def query_rag(query):
    """Complete RAG query process"""
    # Get collection
    collection_id = get_collection_id()
    if not collection_id:
        return False
    
    # Search for relevant chunks
    results = search_documents(query, collection_id)
    if not results:
        return False
    
    # Extract documents and metadata
    documents = results['documents'][0]
    metadatas = results['metadatas'][0]
    distances = results['distances'][0]
    
    print(f"\nFound {len(documents)} relevant chunks:")
    for i, (doc, meta, dist) in enumerate(zip(documents, metadatas, distances), 1):
        print(f"\n{i}. Source: {meta['source']}, Chunk: {meta['chunk_index']}/{meta['total_chunks']}")
        print(f"   Similarity: {1 - dist:.3f}")
        print(f"   Preview: {doc[:100]}...")
    
    # Generate answer
    answer = generate_answer(query, documents)
    if answer:
        print("\n" + "="*60)
        print("ANSWER:")
        print("="*60)
        print(answer)
        print("="*60)
        return True
    
    return False

def main():
    if len(sys.argv) < 2:
        print("Usage: python3 query_rag.py '<your question>'")
        print("\nExample:")
        print("  python3 query_rag.py 'What is the network uptime?'")
        print("  python3 query_rag.py 'What switches are installed?'")
        return 1
    
    query = ' '.join(sys.argv[1:])
    success = query_rag(query)
    return 0 if success else 1

if __name__ == "__main__":
    sys.exit(main())
EOF

chmod +x ~/cisco-rag-demo/scripts/query_rag.py
```

**What this script does:**
1. Takes your question as input
2. Converts it to an embedding
3. Searches ChromaDB for similar chunks
4. Sends chunks + question to Ollama
5. Returns AI-generated answer with sources

---

### Success Checkpoint

Verify scripts were created:

```bash
ls -la ~/cisco-rag-demo/scripts/
```

**Expected output:**
```
-rwxr-xr-x  load_document.py
-rwxr-xr-x  query_rag.py
-rwxr-xr-x  test_connection.py
```

All three scripts should be executable (x permission).

---

## Part 2C: Load Your First Document

**Time:** 5 minutes  
**What you'll do:** Create a test document and load it into the system

### Create Test Document

**Action: Create a sample document**

```bash
cat > ~/cisco-rag-demo/test_document.txt << 'EOF'
NETWORK INFRASTRUCTURE ASSESSMENT REPORT
Boston Office - Q3 2024

EXECUTIVE SUMMARY
The Boston office network infrastructure was assessed during Q3 2024. The network currently operates 24 Cisco Catalyst switches across three distribution layers supporting 450 end users.

NETWORK UPTIME AND RELIABILITY
Network uptime for Q3 2024 was 99.94% with a mean time to repair (MTTR) of 2.4 hours. The incident log shows five outages during the quarter: two planned maintenance windows (totaling 3 hours) and three unplanned events caused by power fluctuations in Building A.

EQUIPMENT INVENTORY
Current network equipment includes:
- 4x Cisco Catalyst 9300 core switches (installed 2022)
- 12x Cisco Catalyst 2960X distribution switches (installed 2019)
- 8x Cisco Catalyst 2960 access switches (installed 2018)
- 2x Cisco ASA 5516-X firewalls (installed 2021)

BANDWIDTH UTILIZATION
Average bandwidth utilization across distribution layer: 45%
Peak bandwidth utilization: 78% (during business hours 9-11 AM)
Inter-VLAN routing traffic: 23% of total traffic

RECOMMENDATIONS
1. Replace 8 aging Catalyst 2960 access switches (end of support: December 2024)
2. Implement redundant power supplies in Building A to prevent future outages
3. Upgrade core switch firmware to version 17.6.3 for security patches
4. Budget allocation: $125,000 for equipment refresh, $62,500 for facility upgrades

DATACENTER CONNECTIVITY
The Boston office connects to the primary datacenter via dual 10Gbps fiber links with automatic failover. Average latency: 2.3ms. No packet loss recorded during the assessment period.

SECURITY POSTURE
All network devices are configured with 802.1X port authentication. Firewall rules reviewed and updated. No unauthorized access attempts detected. Annual penetration testing scheduled for Q4 2024.

FUTURE GROWTH
Current infrastructure can support 30% user growth (up to 585 users) without major upgrades. Beyond that threshold, core switch capacity should be evaluated.
EOF
```

**What's in this document:**
- Network uptime statistics
- Equipment inventory
- Budget recommendations
- Security information
- Multiple topics for testing queries

---

### Load the Document

**Action: Run the loading script**

```bash
python3 ~/cisco-rag-demo/scripts/load_document.py \
  ~/cisco-rag-demo/test_document.txt \
  "Boston Network Assessment Q3 2024"
```

**Expected output:**
```
Loading document: Boston Network Assessment Q3 2024
Total characters: 1847
Created 3 chunks
Processing chunk 1/3... ✓
Processing chunk 2/3... ✓
Processing chunk 3/3... ✓

Completed: 3/3 chunks loaded successfully
```

**What just happened:**
1. Script read the document (1847 characters)
2. Split it into 3 chunks of ~800 chars each
3. Generated embeddings for each chunk
4. Stored all chunks in ChromaDB

**Time:** ~15-30 seconds (depending on your system)

---

### Verify Document Loaded

**Check ChromaDB:**

```bash
# Get collection UUID
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")

# Check document count
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"
```

**Expected output:**
```json
3
```

**What this means:** Your collection now contains 3 chunks from your document.

---

## Part 2D: Test with Python Queries

**Time:** 5 minutes  
**What you'll do:** Ask questions and get AI-generated answers

### Query 1: Specific Fact

**Question about uptime:**

```bash
python3 ~/cisco-rag-demo/scripts/query_rag.py "What was the network uptime in Q3?"
```

**Expected flow:**
1. Converting question to embedding (2 sec)
2. Searching ChromaDB (< 1 sec)
3. Finding 3 relevant chunks
4. Generating answer with Ollama (5-10 sec)

**Expected output:**
```
Searching for: What was the network uptime in Q3?

Found 3 relevant chunks:

1. Source: Boston Network Assessment Q3 2024, Chunk: 1/3
   Similarity: 0.892
   Preview: NETWORK INFRASTRUCTURE ASSESSMENT REPORT...

2. Source: Boston Network Assessment Q3 2024, Chunk: 2/3
   Similarity: 0.856
   Preview: Network uptime for Q3 2024 was 99.94%...

3. Source: Boston Network Assessment Q3 2024, Chunk: 3/3
   Similarity: 0.743
   Preview: DATACENTER CONNECTIVITY...

Generating answer...

============================================================
ANSWER:
============================================================
According to the Boston Network Assessment Q3 2024 report, 
the network uptime for Q3 2024 was 99.94% with a mean time 
to repair (MTTR) of 2.4 hours.
============================================================
```

**What happened:**
- ✓ Found correct chunk with uptime information
- ✓ AI cited the specific document
- ✓ Answered with exact statistics
- ✓ Included relevant context (MTTR)

---

### Query 2: Equipment List

**Question about hardware:**

```bash
python3 ~/cisco-rag-demo/scripts/query_rag.py "What is hte peak bandwitdth?"
```

**Expected answer should include:**
- 4x Catalyst 9300 core switches
- 12x Catalyst 2960X distribution switches
- 8x Catalyst 2960 access switches
- Installation years

---

### Query 3: Budget Information

**Question about costs:**

```bash
python3 ~/cisco-rag-demo/scripts/query_rag.py "What is the equipment budget?"
```

**Expected answer should include:**
- $125,000 for equipment refresh
- $62,500 for facility upgrades
- Total budget allocation

---

### Query 4: Testing Boundaries

**Question not in document:**

```bash
python3 ~/cisco-rag-demo/scripts/query_rag.py "What is the wifi password?"
```

**Expected answer:**
```
I don't have enough information to answer that question.
```

**What this proves:** The AI doesn't hallucinate - if information isn't in your documents, it says so.

---

### Success Checkpoint

✓ Your Python RAG system works when:
- Documents load without errors
- Queries find relevant chunks
- AI generates accurate answers
- System handles unknown questions gracefully

**If queries are slow (>20 seconds):**
- First query is always slower (loading model)
- Subsequent queries should be faster (5-10 sec)
- Check RAM usage if all queries are slow

---

## Part 2E: n8n Document Upload Workflow

**Time:** 10 minutes  
**What you'll do:** Create form-based workflow for document uploads

### Understanding Upload Workflow

**Visual workflow diagram:**

```
┌─────────────────────────────────────────────────────┐
│      DOCUMENT UPLOAD WORKFLOW (FORM-BASED)          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Form]  →  [Extract]  →  [Parse]  →  [Get UUID]    │
│     ↓           ↓            ↓            ↓         │
│  Upload    Validate      Extract      Find          │
│  file      file type     text       collection      │
│                                                     │
│  [Process]  →  [Embed]  →  [Prepare]  →  [Store]    │
│      ↓            ↓           ↓            ↓        │
│  Chunk       Generate     Build       Save to       │
│  text        vectors      payload    ChromaDB       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Key features:**
- Popup form interface (click "Test workflow" button)
- Accepts `.txt`, `.doc`, `.docx` files
- Handles file parsing automatically
- User-friendly error messages

---

### Prepare Workflows Directory

```bash
# Create workflows directory if it doesn't exist
mkdir -p ~/cisco-rag-demo/workflows

# Verify it was created
ls -la ~/cisco-rag-demo/ | grep workflows
```

**Expected output:**
```
drwxr-xr-x  ... workflows
```

✓ **Checkpoint:** Workflows directory exists

---

### Create Upload Workflow

**Action: Create workflow JSON file**

```bash
cat > ~/cisco-rag-demo/workflows/document_upload_form.json << 'EOF'
{
  "name": "RAG - Document Upload (Form)",
  "nodes": [
    {
      "parameters": {
        "path": "form-upload-rag-doc",
        "formTitle": "Upload Document to RAG",
        "formDescription": "Upload .txt, .doc, or .docx files",
        "formFields": {
          "values": [
            {
              "fieldLabel": "Select Document File",
              "fieldType": "file",
              "acceptFileTypes": ".txt,.doc,.docx",
              "requiredField": true
            },
            {
              "fieldLabel": "Document Name (optional)",
              "placeholder": "Leave blank to use filename"
            },
            {
              "fieldLabel": "Collection",
              "fieldType": "dropdown",
              "fieldOptions": {
                "values": [{"option": "cisco_docs"}]
              },
              "requiredField": true
            }
          ]
        }
      },
      "name": "Form Trigger",
      "type": "n8n-nodes-base.formTrigger",
      "typeVersion": 2,
      "position": [240, 300],
      "webhookId": "form-upload-rag-doc"
    },
    {
      "parameters": {
        "jsCode": "const item = $input.first();\nconst formData = item.json;\nconst binaryData = item.binary;\n\nconst userProvidedName = formData['Document Name (optional)'] || '';\nconst collectionName = formData['Collection'] || 'cisco_docs';\n\nlet fileObj = null;\nconst possibleFileFields = ['Select Document File', 'file', 'document'];\n\nif (binaryData) {\n  for (const fieldName of possibleFileFields) {\n    if (binaryData[fieldName]) {\n      fileObj = binaryData[fieldName];\n      break;\n    }\n  }\n  if (!fileObj) {\n    const binaryKeys = Object.keys(binaryData);\n    if (binaryKeys.length > 0) fileObj = binaryData[binaryKeys[0]];\n  }\n}\n\nif (!fileObj) throw new Error('No file uploaded');\n\nconst fileData = fileObj.data;\nconst filename = fileObj.fileName || 'uploaded_file';\nconst extension = filename.split('.').pop().toLowerCase();\n\nif (!['txt', 'doc', 'docx'].includes(extension)) {\n  throw new Error(`Unsupported: .${extension}. Use .txt/.doc/.docx`);\n}\n\nif (fileData.length > 10 * 1024 * 1024) {\n  throw new Error('File too large (max 10MB)');\n}\n\nconst documentName = userProvidedName || filename.replace(/\\.[^/.]+$/, '').replace(/_/g, ' ');\n\nreturn [{\n  json: { filename, extension, size: fileData.length, documentName, collectionName },\n  binary: { fileData: fileObj }\n}];"
      },
      "name": "Extract File Data",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [460, 300]
    },
    {
      "parameters": {
        "jsCode": "const fileInfo = $input.first().json;\nconst binaryData = $input.first().binary.fileData;\n\nif (!binaryData?.data) throw new Error('No file data');\n\nconst fileBuffer = Buffer.from(binaryData.data, 'base64');\nconst extension = fileInfo.extension;\nlet documentText = '';\n\nif (extension === 'txt') {\n  documentText = fileBuffer.toString('utf-8');\n} else if (extension === 'docx') {\n  const AdmZip = require('adm-zip');\n  const zip = new AdmZip(fileBuffer);\n  let documentXml = null;\n  \n  for (const entry of zip.getEntries()) {\n    if (entry.entryName === 'word/document.xml') {\n      documentXml = entry.getData().toString('utf8');\n      break;\n    }\n  }\n  \n  if (!documentXml) throw new Error('Invalid .docx file');\n  \n  const textMatches = documentXml.match(/<w:t[^>]*>([^<]+)<\\/w:t>/g);\n  if (textMatches?.length > 0) {\n    documentText = textMatches.map(m => m.replace(/<[^>]+>/g, '')).join(' ');\n  } else {\n    documentText = documentXml.replace(/<[^>]+>/g, ' ').replace(/\\s+/g, ' ').trim();\n  }\n} else if (extension === 'doc') {\n  const fullText = fileBuffer.toString('binary');\n  const matches = fullText.match(/[\\x20-\\x7E]{4,}/g) || [];\n  \n  documentText = matches\n    .filter(text => /[a-zA-Z]/.test(text) && !/^[^a-zA-Z0-9\\s]{10,}$/.test(text))\n    .join(' ')\n    .replace(/\\s+/g, ' ')\n    .trim();\n  \n  if (documentText.length < 100) {\n    throw new Error('Cannot extract .doc text. Convert to .docx or .txt first.');\n  }\n}\n\ndocumentText = documentText\n  .replace(/\\r\\n/g, '\\n')\n  .replace(/\\n{3,}/g, '\\n\\n')\n  .replace(/[\\x00-\\x08\\x0B-\\x0C\\x0E-\\x1F]/g, '')\n  .replace(/\\s+/g, ' ')\n  .trim();\n\nreturn [{\n  json: {\n    document: documentText,\n    filename: fileInfo.filename,\n    documentName: fileInfo.documentName,\n    collectionName: fileInfo.collectionName,\n    textLength: documentText.length,\n    fileType: fileInfo.extension\n  }\n}];"
      },
      "name": "Parse Document Content",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [680, 300]
    },
    {
      "parameters": {
        "url": "http://chromadb:8000/api/v1/collections",
        "options": {"timeout": 5000}
      },
      "name": "Get Collection UUID",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [900, 300]
    },
    {
      "parameters": {
        "jsCode": "const collections = $input.all().map(item => item.json);\nconst documentInput = $('Parse Document Content').first().json;\n\nconst collectionName = documentInput.collectionName;\nconst document = documentInput.document;\nconst documentName = documentInput.documentName;\n\nlet collectionUuid = null;\nfor (const coll of collections) {\n  if (coll.name === collectionName) {\n    collectionUuid = coll.id;\n    break;\n  }\n}\n\nif (!collectionUuid) throw new Error(`Collection '${collectionName}' not found`);\n\nconst timestamp = Date.now();\nconst safeDocName = documentName.replace(/[^a-zA-Z0-9_-]/g, '_').substring(0, 50);\nconst documentId = `${safeDocName}_${timestamp}`;\n\nconst chunkSize = 800;\nconst overlap = 100;\nconst chunks = [];\nlet start = 0;\n\nwhile (start < document.length) {\n  const end = start + chunkSize;\n  const chunk = document.substring(start, end).trim();\n  \n  if (chunk.length > 0) {\n    chunks.push({\n      chunk,\n      index: chunks.length,\n      document_id: documentId,\n      document_name: documentName,\n      collection_uuid: collectionUuid,\n      collection_name: collectionName,\n      embedding_request: {\n        model: 'nomic-embed-text',\n        prompt: chunk\n      }\n    });\n  }\n  start = end - overlap;\n}\n\nif (chunks.length === 0) throw new Error('No chunks created');\n\nreturn chunks.map(c => ({ json: c }));"
      },
      "name": "Process Document",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1120, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "http://host.containers.internal:11434/api/embeddings",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json.embedding_request) }}",
        "options": {"timeout": 30000}
      },
      "name": "Generate Embeddings",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [1340, 300]
    },
    {
      "parameters": {
        "jsCode": "const chunk = $input.first().json.chunk;\nconst embedding = $input.first().json.embedding;\nconst documentId = $input.first().json.document_id;\nconst documentName = $input.first().json.document_name;\nconst chunkIndex = $input.first().json.index;\nconst collectionUuid = $input.first().json.collection_uuid;\n\nconst chunkId = `${documentId}_chunk_${chunkIndex}_${Date.now()}`;\n\nconst payload = {\n  ids: [chunkId],\n  embeddings: [embedding],\n  documents: [chunk],\n  metadatas: [{\n    source: documentName,\n    chunk_index: chunkIndex,\n    document_id: documentId\n  }]\n};\n\nreturn [{\n  json: {\n    collection_uuid: collectionUuid,\n    payload: payload,\n    chunk_id: chunkId\n  }\n}];"
      },
      "name": "Prepare Storage",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1560, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=http://chromadb:8000/api/v1/collections/{{ $('Get Collection UUID').item.json.id }}/add",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json.payload) }}"
      },
      "name": "Store in ChromaDB",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [1780, 300]
    }
  ],
  "connections": {
    "Form Trigger": {"main": [[{"node": "Extract File Data", "type": "main", "index": 0}]]},
    "Extract File Data": {"main": [[{"node": "Parse Document Content", "type": "main", "index": 0}]]},
    "Parse Document Content": {"main": [[{"node": "Get Collection UUID", "type": "main", "index": 0}]]},
    "Get Collection UUID": {"main": [[{"node": "Process Document", "type": "main", "index": 0}]]},
    "Process Document": {"main": [[{"node": "Generate Embeddings", "type": "main", "index": 0}]]},
    "Generate Embeddings": {"main": [[{"node": "Prepare Storage", "type": "main", "index": 0}]]},
    "Prepare Storage": {"main": [[{"node": "Store in ChromaDB", "type": "main", "index": 0}]]}
  },
  "settings": {"executionOrder": "v1"},
  "active": false
}
EOF
```

---

### Import Upload Workflow

**Action:**

1. In n8n, click **"Add workflow"**
2. Click **"⋮"** menu → **"Import from file"**
3. Select: `~/cisco-rag-demo/workflows/document_upload_form.json`
4. Click **"Import"**

**You should see:**
- 8 nodes in sequence
- Form Trigger → Extract → Parse → Get UUID → Process → Embed → Prepare → Store
- All nodes connected properly

---

### Activate Upload Workflow

**Action:**
1. Click **"Active"** toggle
2. Status changes to "Active"
3. **Important:** A "Test workflow" button appears

---

### Test Form Upload

**Action: Click "Test workflow" button**

1. Popup form appears: "Upload Document to RAG"
2. Click **"Select Document File"**
3. **Option A:** Upload Boston Network Assessment if you have it
4. **Option B:** Create a quick test file:

```bash
cat > ~/cisco-rag-demo/test_upload.txt << 'EOF'
NETWORK MONITORING TEST

Network uptime should be monitored continuously.
Alert thresholds: 95% minimum uptime required.
Response time: Under 100ms for critical systems.
Packet loss: Less than 0.1% acceptable.

The monitoring system sends immediate alerts when thresholds are exceeded.
EOF
```

5. Upload the file:
   - **Select Document File:** Choose file
   - **Document Name:** Leave blank
   - **Collection:** Select "cisco_docs"
6. Click **"Submit"**

**Expected result:**
- Form shows success message
- File upload completes
- Form closes automatically

**In n8n:** All 8 nodes show green checkmarks

---

### Verify Upload

**Query uploaded document:**

```bash
python3 ~/cisco-rag-demo/scripts/query_rag.py "What should the monitoring system do?"
```

**Expected:** Answer mentions sending immediate alerts

---

## Part 2F: n8n Chat Interface

**Time:** 10 minutes  
**What you'll do:** Create interactive chat for asking questions

### Understanding Chat Workflow

**Chat workflow diagram:**

```
┌──────────────────────────────────────────────────┐
│            CHAT INTERFACE WORKFLOW               │
├──────────────────────────────────────────────────┤
│                                                  │
│  [Chat Trigger]  ──>  [Process Question]         │
│       ↓                      ↓                   │
│  Receive user           Extract question         │
│  message                                         │
│                                                  │
│  [Embed]  ──>  [Search]  ──>  [Retrieve]         │
│     ↓              ↓              ↓              │
│  Convert to    Find similar   Get top 3          │
│  vector        chunks         chunks             │
│                                                  │
│  [Generate]  ──>  [Format]  ──>  [Reply]         │
│     ↓              ↓              ↓              │
│  Create AI     Clean up        Send to           │
│  answer        response        user              │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

### Prepare Workflows Directory

Before creating the workflow file, ensure the workflows directory exists:

```bash
# Create workflows directory if it doesn't exist
mkdir -p ~/cisco-rag-demo/workflows

# Verify it was created
ls -la ~/cisco-rag-demo/ | grep workflows
```

**Expected output:**
```
drwxr-xr-x  ... workflows
```

✓ **Checkpoint:** Workflows directory exists

---

### Create Chat Workflow

**Action: Create chat workflow JSON**

```bash
cat > ~/cisco-rag-demo/workflows/rag_chat.json << 'EOF'
{
  "name": "RAG Chat Interface",
  "nodes": [
    {
      "parameters": {
        "options": {}
      },
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1,
      "position": [
        48,
        32
      ],
      "webhookId": "rag-chat",
      "id": "5d8ff83c-7b4a-4341-82ad-77ffbe61325b"
    },
    {
      "parameters": {
        "jsCode": "// Build LLM prompt with retrieved context\nconst searchResults = $input.first().json;\nconst question = $('Prepare Search').first().json.question;\n\nconst contexts = searchResults.documents[0];\n\nif (!contexts || contexts.length === 0) {\n  throw new Error('No documents found in ChromaDB. Load documents first.');\n}\n\nconst contextText = contexts.map((ctx, i) => \n  `[Source ${i + 1}]\\n${ctx}`\n).join('\\n\\n---\\n\\n');\n\nconst systemPrompt = `You are a helpful AI assistant analyzing Cisco network assessment documents. \nProvide accurate, concise answers based on the context provided. \nIf the context doesn't contain enough information to answer fully, say so.`;\n\nconst prompt = `${systemPrompt}\\n\\nCONTEXT FROM DOCUMENTS:\\n${contextText}\\n\\nUSER QUESTION: ${question}\\n\\nYour answer (be specific and cite relevant details from the context):`;\n\nconsole.log(`Retrieved ${contexts.length} chunks for question: ${question}`);\n\nreturn [{\n  json: {\n    llm_request: {\n      model: 'llama3.2:3b',\n      prompt: prompt,\n      stream: false,\n      options: {\n        temperature: 0.7,\n        top_p: 0.9\n      }\n    }\n  }\n}];"
      },
      "name": "Build Prompt",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [
        1248,
        32
      ],
      "id": "d8fa7712-5d19-465d-a02c-6dc29f30b5be"
    },
    {
      "parameters": {
        "jsCode": "// Get UUID and prepare query embedding request\nconst collections = $input.all().map(item => item.json);\nconst question = $('Chat Trigger').first().json.chatInput;\n\n// Flatten nested arrays (handle both [[{...}]] and [{...}] formats)\nlet flatCollections = collections.flat();\n\n// Find cisco_docs collection\nlet collectionUuid = null;\nfor (const coll of flatCollections) {\n  if (coll.name === 'cisco_docs') {\n    collectionUuid = coll.id;\n    break;\n  }\n}\n\nif (!collectionUuid) {\n  throw new Error(\"Collection 'cisco_docs' not found! Load documents first.\");\n}\n\nreturn [{\n  json: {\n    collection_uuid: collectionUuid,\n    question: question,\n    embedding_request: {\n      model: 'nomic-embed-text',\n      prompt: question\n    }\n  }\n}];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        416,
        32
      ],
      "id": "c2ecbc29-bce4-4987-945e-0875be07d390",
      "name": "Prepare Query"
    },
    {
      "parameters": {
        "jsCode": "// Prepare ChromaDB search request\nconst embedding = $input.first().json.embedding;\nconst queryData = $('Prepare Query').first().json;\n\nreturn [{\n  json: {\n    collection_uuid: queryData.collection_uuid,\n    question: queryData.question,\n    search_request: {\n      query_embeddings: [embedding],\n      n_results: 3\n    }\n  }\n}];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        816,
        32
      ],
      "id": "0d7959ad-2521-49fd-a7ad-b11ff60a169f",
      "name": "Prepare Search"
    },
    {
      "parameters": {
        "url": "http://chromadb:8000/api/v1/collections",
        "options": {
          "timeout": 5000
        }
      },
      "name": "Get Collection UUID",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [
        224,
        32
      ],
      "id": "f132df78-0a06-4934-a1ad-731ce8e1684c"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=http://chromadb:8000/api/v1/collections/{{ $json.collection_uuid }}/query",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify($json.search_request) }}",
        "options": {
          "timeout": 5000
        }
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        1024,
        32
      ],
      "id": "220eae9b-21fe-4a39-864e-90679de6331d",
      "name": "Search ChromaDB"
    },
    {
      "parameters": {
        "jsCode": "// Format response for chat interface\nconst llmData = $input.first().json;\nconst answer = llmData.response;\n\nconst cleanAnswer = answer.trim();\n\nconsole.log('Generated ' + cleanAnswer.length + ' character response');\n\n// Return in format expected by Chat Trigger\nreturn [{\n  json: {\n    output: cleanAnswer\n  }\n}];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1680,
        32
      ],
      "id": "b6e315d3-ec94-4c70-8e70-93ef93c16294",
      "name": "Format Response"
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
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        1472,
        32
      ],
      "id": "b7da2483-2ddf-4f8d-a57d-0b1074732475",
      "name": "Generate Answer"
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
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        624,
        32
      ],
      "id": "ef7967a3-5fa8-4cfd-bc5a-81b0a47f700c",
      "name": "Generate Query Embedding"
    }
  ],
  "pinData": {},
  "connections": {
    "Chat Trigger": {
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
    "Prepare Query": {
      "main": [
        [
          {
            "node": "Generate Query Embedding",
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
    "Prepare Search": {
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
    "Generate Answer": {
      "main": [
        [
          {
            "node": "Format Response",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Generate Query Embedding": {
      "main": [
        [
          {
            "node": "Prepare Search",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1",
    "availableInMCP": false
  },
  "versionId": "736bf97e-7521-4ab0-ab53-48faebddf382",
  "meta": {
    "instanceId": "6fa23a8d1b651a4bc39c80909c6ffb56117e9e4a8c53ae3f221584b59b6c390b"
  },
  "id": "q4J-v-MCyKQM-1tW5xZee",
  "tags": []
}
EOF
```

---

### Import Chat Workflow

**Action:**

1. In n8n, click **"Add workflow"**
2. Click **"⋮"** menu → **"Import from file"**
3. Select: `~/cisco-rag-demo/workflows/rag_chat.json`
4. Click **"Import"**

**You should see:**
- 9 nodes in sequence
- Chat Trigger node (special type)
- All nodes connected properly

---

### Activate Chat Workflow

**Action:**
1. Click **"Active"** toggle
2. Status changes to "Active"
3. **Important:** A "Chat" button appears in the bottom-right of n8n interface

---

### Test Chat Interface

**Action: Click the Chat button**

1. Chat window opens
2. Type: **"What was the network uptime?"**
3. Press Enter

**Expected behavior:**
1. Message sent
2. Processing... (5-10 seconds)
3. AI response appears with answer from your documents

**Try more questions:**
- "What switches are installed?"
- "What is the budget for equipment?"
- "Tell me about the datacenter connectivity"
- "What is the wifi password?" (should say insufficient information)

---

## Part 2G: System Verification

**Time:** 5 minutes  
**What you'll do:** Comprehensive system test

### Complete System Check

**Test all components:**

```bash
# 1. Environment still working
~/cisco-rag-demo/system-check.sh

# 2. Documents in database
UUID=$(curl -s http://localhost:8000/api/v1/collections | \
       python3 -c "import sys, json; print([c['id'] for c in json.load(sys.stdin) if c['name']=='cisco_docs'][0])")
curl -s "http://localhost:8000/api/v1/collections/$UUID/count"

# 3. Python query works
python3 ~/cisco-rag-demo/scripts/query_rag.py "What is network uptime?"

# 4. Chat interface - test manually in browser
```

---

### Final Checklist

**Environment (from Part 1):**
- [ ] Podman machine running
- [ ] Ollama service running
- [ ] ChromaDB container running
- [ ] n8n container running
- [ ] All models downloaded

**Python Scripts (this guide):**
- [ ] load_document.py works
- [ ] query_rag.py returns accurate answers
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

**✓✓✓ If all boxes checked: Your RAG system is PILOT READY!**

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
ollama serve > /dev/null 2>&1 &

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
# On macOS: Open Activity Monitor
# Look at "Memory Pressure"

# On Linux:
free -h
```

**If pressure is high (yellow/red):**
- Close other applications
- Restart system to free memory

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

🎉 **Congratulations!** You've built a complete RAG system!

**What you've accomplished:**
- ✓ Understand RAG concepts deeply
- ✓ Built document upload system (2 methods)
- ✓ Built query system (2 methods)
- ✓ Created visual workflows in n8n
- ✓ Everything runs locally and privately

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
**Prerequisites:** Completed Parts 1 and 2 (✓)

**⚡ Continue to [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)**

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


**You can always return to Part 3 when ready!**


**Feedback:** Encountered issues not covered here? Document them for future updates.

---

**⚡ Ready for enterprise integration? Proceed to [GUIDE_3_WEBEX_INTEGRATION.md](GUIDE_3_WEBEX_INTEGRATION.md)**
