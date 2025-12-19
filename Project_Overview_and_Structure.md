# Local AI Document Assistant for Enterprise IT

**Build your own ChatGPT-style assistant that runs entirely on your network—no external AI APIs, no data leaving your infrastructure.**

## 🎯 What This Project Does

This project shows you how to build a complete AI-powered document analysis system that:
- Answers questions about your technical documents (network assessments, policies, reports)
- Responds via Webex Teams like a colleague (@mention or DM)
- Processes everything locally on your Mac (no cloud AI services)
- Costs nothing to run after initial setup
- Takes 2-4 hours to build (no coding experience required)

**Real-World Example**: Upload a Cisco network assessment report, then ask:
- "What's the proposed budget for the datacenter upgrade?"
- "Who should I contact about the security concerns?"
- "Summarize the top 3 infrastructure risks"

Your AI assistant responds in seconds, citing the specific sections from your document.

## 🔒 Why Build This Instead of Using ChatGPT?

| Your Local System | Cloud AI Services |
|-------------------|-------------------|
| ✅ Documents stay on your network | ❌ Data sent to external servers |
| ✅ Free to run (after setup) | ❌ API costs per query |
| ✅ HIPAA/SOC2/compliance friendly | ❌ May violate data policies |
| ✅ Works offline | ❌ Requires internet |
| ✅ Customize for your org | ❌ Generic responses |

**Perfect for**: IT engineers handling sensitive network assessments, financial data, customer information, or any documents that can't leave your infrastructure.

## 👥 Who This Is For

**You don't need to be a programmer!** This guide is written for:
- IT engineers and network administrators
- Technical project managers
- Anyone comfortable following step-by-step instructions
- People who've never worked with AI or containers before

**Prerequisites**:
- A Mac with Apple Silicon (M1/M2/M3/M4) and 16GB+ RAM
- Ability to use Terminal and copy/paste commands
- 2-4 hours of focused time
- Curiosity and patience

## 🏗️ What You'll Build

Three interconnected systems that work together:

### 1️⃣ **Local AI Foundation** (Environment Setup)
- **Podman**: Think of this as a "virtual computer" that runs isolated software packages
- **Ollama**: The AI "brain" that reads and understands text (runs on your Mac)
- **ChromaDB**: The "smart filing cabinet" that stores document chunks with context
- **n8n**: Visual workflow builder (no code required—drag and drop)

### 2️⃣ **Document Intelligence System** (RAG Implementation)
- Upload documents through a simple web form
- AI breaks documents into searchable chunks
- Ask questions, get accurate answers with source citations
- Chat interface you can test immediately

### 3️⃣ **Enterprise Integration** (Webex Bot)
- AI assistant responds in Webex Teams
- @mention the bot or send it a DM
- 5-8 second response time
- Works like chatting with a knowledgeable colleague

## 📁 Repository Structure
```
local-rag-webex-bot/
│
├── README.md                          # ← You are here
├── LICENSE                            # MIT License
│
├── 📘 docs/
│   ├── 01_Environment_Setup.md        # Install Podman, Ollama, ChromaDB, n8n
│   ├── 02_RAG_System_Guide.md         # Build document upload + chat system
│   ├── 03_Webex_Integration.md        # Connect your AI to Webex Teams
│   ├── 04_Comprehensive_Troubleshooting.md  # Common issues + solutions
│   └── 05_Whats_Next.md               # Advanced features + customization
│
├── 🔧 workflows/
│   ├── Document_Upload_DOC_Support.json      # n8n workflow: Upload docs
│   ├── RAG_Query_Chat.json                   # n8n workflow: Chat interface
│   ├── Webex_RAG_Bot.json                    # n8n workflow: Webex bot
│   └── README_Workflows.md                   # How to import these files
│
├── 📚 resources/
│   ├── n8n_Quick_Reference.md         # Common n8n operations cheat sheet
│   ├── n8n_Navigation_Guide.md        # Visual guide to n8n interface
│   ├── n8n_Visual_UI_Reference.md     # Screenshots + annotations
│   └── sample_documents/              # Test documents to try
│       ├── sample_network_assessment.txt
│       └── sample_policy_doc.txt
│
├── 🐛 troubleshooting/
│   ├── Troubleshooting_Chronicle.md   # Detailed debugging stories
│   ├── Common_Errors.md               # Quick error → solution guide
│   └── Diagnostic_Scripts.md          # Test scripts to verify setup
│
├── 📖 learning/
│   ├── RAG_Concepts_Explained.md      # What is RAG? (beginner-friendly)
│   ├── Prerequisites_and_Learning.md  # Background knowledge needed
│   └── Use_Case_Examples.md           # Real-world applications
│
└── 🎓 extras/
    ├── Python_CLI_Scripts/            # Alternative: Python-based access
    │   ├── test_rag_system.py
    │   ├── interactive_chat.py
    │   └── README_CLI.md
    └── Advanced_Customization.md      # Tune performance, add features
```

## 🚀 Quick Start (Choose Your Path)

### ⏱️ Fast Track (2 hours)
**Goal**: Get the AI assistant working in Webex Teams

1. **[Environment Setup](docs/01_Environment_Setup.md)** (60 min)
   - Install all required software
   - Verify everything works
   
2. **[RAG System](docs/02_RAG_System_Guide.md)** (30 min)
   - Upload your first document
   - Test the chat interface
   
3. **[Webex Integration](docs/03_Webex_Integration.md)** (30 min)
   - Create bot account
   - Connect to Webex Teams

### 🎓 Learning Path (4 hours)
**Goal**: Understand what you're building and why

1. **[Prerequisites & Concepts](learning/Prerequisites_and_Learning.md)** (30 min)
   - What is RAG?
   - How does local AI work?
   
2. **[Environment Setup](docs/01_Environment_Setup.md)** (60 min)
   
3. **[RAG System Deep Dive](docs/02_RAG_System_Guide.md)** (60 min)
   - Build step-by-step
   - Learn what each component does
   
4. **[Webex Integration](docs/03_Webex_Integration.md)** (45 min)
   
5. **[Customization](extras/Advanced_Customization.md)** (45 min)
   - Tune for your documents
   - Add features

### 🆘 Something Broke?
- **[Troubleshooting Chronicle](troubleshooting/Troubleshooting_Chronicle.md)**: Real debugging sessions with solutions
- **[Common Errors](troubleshooting/Common_Errors.md)**: Quick error → fix lookup
- **[Diagnostic Scripts](troubleshooting/Diagnostic_Scripts.md)**: Test what's working

## 📚 Documentation Guide

### Start Here Documents
1. **[Environment Setup](docs/01_Environment_Setup.md)**: Install everything (START HERE)
2. **[RAG System Guide](docs/02_RAG_System_Guide.md)**: Build the AI assistant
3. **[Webex Integration](docs/03_Webex_Integration.md)**: Connect to Teams

### Supporting Documents
- **[Comprehensive Troubleshooting](docs/04_Comprehensive_Troubleshooting.md)**: All known issues + solutions
- **[What's Next](docs/05_Whats_Next.md)**: Advanced features, optimizations, ideas

### Resources for Reference
- **[n8n Quick Reference](resources/n8n_Quick_Reference.md)**: Common operations cheat sheet
- **[n8n Navigation Guide](resources/n8n_Navigation_Guide.md)**: Visual interface guide
- **[RAG Concepts](learning/RAG_Concepts_Explained.md)**: Understanding the technology

### Workflow Files
- **[workflows/](workflows/)**: Import these JSON files into n8n
- **[workflows/README_Workflows.md](workflows/README_Workflows.md)**: How to import + configure

## 🎯 Success Criteria

You'll know you're done when you can:
1. ✅ Upload a document through a web form
2. ✅ Ask questions and get accurate answers with citations
3. ✅ @mention your bot in Webex Teams and get a response
4. ✅ See response times of 5-10 seconds

## 💡 Common Use Cases

**Network Assessments**: "What VLANs need reconfiguration?"  
**Policy Documents**: "What's our data retention policy for customer emails?"  
**Incident Reports**: "What caused last month's outage?"  
**Budget Proposals**: "What's the total cost for the datacenter refresh?"  
**Meeting Notes**: "What action items were assigned to the network team?"

## ⚙️ Technical Specifications

- **AI Model**: Llama 3.2 (3B parameters) - runs on Mac M4 Pro
- **Embedding Model**: nomic-embed-text - converts text to searchable vectors
- **Vector Database**: ChromaDB 0.4.24 - stores document chunks
- **Workflow Engine**: n8n - visual automation (no coding)
- **Containerization**: Podman - isolated environments
- **Response Time**: 5-10 seconds per query
- **RAM Requirements**: 16GB minimum, 24GB recommended
- **Storage**: ~5GB for models + documents

## 🤝 Contributing

Found a bug? Have an improvement? Create an issue or submit a pull request!

## 📄 License

MIT License - Use this however you want for your organization.

## 🙏 Acknowledgments

Built for Cisco Live 2026 demonstration. Designed to show IT engineers that local AI is practical, private, and powerful.

---

**Ready to build?** Start with **[Environment Setup](docs/01_Environment_Setup.md)** →