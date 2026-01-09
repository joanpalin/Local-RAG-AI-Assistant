# Local RAG System with Enterprise Messaging Integration

> **Privacy-first AI document analysis with Webex Teams integration. No external APIs, full control, runs on your local machine.**

---

## ✨ What This Is

A complete, pilot-ready system that lets you:
- 📄 **Upload technical documents** (FAQs, reports, policies)
- 💬 **Ask questions in Webex Teams** (or Python/web interface)
- 🤖 **Get AI-powered answers** based on YOUR documents
- 🔒 **Keep everything local** - no data sent to OpenAI, Anthropic, etc.
- ⚡ **Respond in 5-10 seconds** with cited sources

**Perfect for:** IT teams, network engineers, technical documentation management, compliance-sensitive environments.

---

## 🎯 Quick Start (3 Options)

Choose your path:

### 👶 Complete Beginner (Never used terminal?)
**Time:** 8-10 hours  
**Path:** [Prerequisites Guide](PREREQUISITES_AND_LEARNING.md) → [Environment Setup](GUIDE_1_ENVIRONMENT_SETUP.md) → [RAG System](GUIDE_2_RAG_SYSTEM.md)

Start here if you're new to command-line tools or containers.

### 🛠️ Some IT Experience (Used terminal before)
**Time:** 4-6 hours  
**Path:** [Environment Setup](GUIDE_1_ENVIRONMENT_SETUP.md) → [RAG System](GUIDE_2_RAG_SYSTEM.md) → [Webex Integration](GUIDE_3_WEBEX_INTEGRATION.md)

Start here if you're comfortable with basic Unix commands.

### 🚀 Jump Right In (Experienced with Docker/containers)
**Time:** 1-2 hours  
**Quick commands:**
```bash
# Install base tools
brew install podman ollama

# Start services
podman machine init && podman machine start
ollama serve &

# Download AI models
ollama pull llama3.2:3b
ollama pull nomic-embed-text

# Start ChromaDB
podman run -d -p 8000:8000 \
  -v $HOME/chromadb-data:/chroma/chroma \
  --name chromadb \
  chromadb/chromadb:0.4.24

# Start n8n
podman run -d -p 5678:5678 \
  -v $HOME/.n8n:/home/node/.n8n \
  --name n8n \
  docker.n8n.io/n8nio/n8n

# Clone and set up
git clone <this-repo>
cd <repo>
python3 load_document.py documents/sample.md
python3 query_rag.py "What is the network uptime?"
```

Then add [Webex Integration](GUIDE_3_WEBEX_INTEGRATION.md) for messaging.

---

## 📖 Documentation

### Core Guides (Follow in Order)

| Guide | Purpose | Time | Difficulty |
|-------|---------|------|-----------|
| [Environment Setup](GUIDE_1_ENVIRONMENT_SETUP.md) | Install Podman, Ollama, ChromaDB, n8n | 45-60 min | ⭐⭐ |
| [RAG System](GUIDE_2_RAG_SYSTEM.md) | Set up document loading and querying | 60-90 min | ⭐⭐ |
| [Webex Integration](GUIDE_3_WEBEX_INTEGRATION.md) | Add enterprise messaging bot | 30-45 min | ⭐⭐⭐ |

### Supporting Documentation

| Document | Use When |
|----------|----------|
| [Prerequisites](PREREQUISITES_AND_LEARNING.md) | New to terminal/containers/Python |
| [Troubleshooting](TROUBLESHOOTING.md) | Something isn't working |
| [What's Next](WHATS_NEXT.md) | System working, want to improve it |
| [Documentation Standards](DOCUMENTATION_STANDARDS.md) | Contributing to docs |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────┐
│          Your Local Machine          │
│                                      │
│  ┌─────────────┐  ┌──────────────┐   │
│  │   Ollama    │  │   Podman     │   │
│  │  (Native)   │  │  (Containers)│   │
│  │             │  │              │   │
│  │ • LLM       │  │ • ChromaDB   │   │
│  │ • Embeddings│  │ • n8n        │   │
│  └─────────────┘  └──────────────┘   │
│         ↕                  ↕         │
│    ┌────────────────────────────┐    │
│    │   Your Documents (Local)   │    │
│    │   Network Assessments      │    │
│    │   Technical Reports        │    │
│    │   Policies & Procedures    │    │
│    └────────────────────────────┘    │
└──────────────────────────────────────┘
              ↕ (via tunnel)
┌──────────────────────────────────────┐
│  Webex Teams Cloud                   │
│  • Users ask questions               │
│  • Bot responds with AI answers      │
└──────────────────────────────────────┘
```

**Key Design Principles:**
- ✅ **Everything local** (except Webex messaging)
- ✅ **No external AI APIs** (Ollama runs locally)
- ✅ **Your data stays yours** (never leaves your control)
- ✅ **Open source components** (no vendor lock-in)

---

## 🎓 What You'll Learn

Even if you're a beginner, by completing this project you'll understand:

**Technical Skills:**
- Container orchestration (Podman)
- Vector databases (ChromaDB)
- Local LLM deployment (Ollama)
- Workflow automation (n8n)
- RAG architecture
- Webhook integration
- REST APIs

**Practical Knowledge:**
- How AI document analysis works
- Privacy-preserving AI deployment
- Enterprise integration patterns
- System troubleshooting
- Production deployment

**Career Skills:**
- Modern AI/ML deployment
- Infrastructure as code
- DevOps practices
- Technical documentation

---

## 💡 Use Cases

**Network Engineering:**
- Query Cisco network assessments
- Find equipment needing replacement
- Identify security risks
- Budget planning analysis

**IT Documentation:**
- Search technical runbooks
- Find configuration procedures
- Retrieve troubleshooting steps
- Onboard new team members

**Compliance & Policy:**
- Query company policies
- Find compliance requirements
- Reference procedures
- Audit documentation access

**General Knowledge Base:**
- Company wiki alternative
- Technical documentation search
- Team knowledge sharing
- Historical project reference

---

## 🚦 System Requirements

### Hardware
- **Used in the lab:** Apple Silicon Mac M4 with 16GB RAM 
- **Recommended:** 24GB+ RAM for better performance
- **Storage:** 50GB free disk space
- **OS** This guide is optimized for macOS. Linux adaptation is straightforward; Windows requires WSL2. You can use any AI assistant and provide this current guide and ask for the equivalent in Windows

### Software
- **Used in the lab:** Sonoma (14.0) 
- **Internet:** Required for initial setup and Webex integration
- **Optional:** Homebrew (package manager) - highly recommended

### Skills
- **Minimum:** Basic computer literacy, willingness to learn
- **Helpful:** Command line experience, basic programming concepts
- **Not required:** Coding expertise, AI/ML background, DevOps experience

---

## ⚡ Performance Benchmarks

On Mac M4 Pro (16GB RAM):

| Operation | Time | Notes |
|-----------|------|-------|
| Document upload (5 pages) | 30-60 sec | One-time per document |
| First query after startup | 10-15 sec | Model loading (cold start) |
| Subsequent queries | 5-8 sec |  Target performance |
| Webex bot response | 5-10 sec | End-to-end with webhook |

**Optimization tips:**
- Keep system running between queries (avoid cold starts)
- Use SSD storage (faster database access)

---

## 🔥 Why This Approach?

### Privacy & Compliance
Your documents never leave your infrastructure. Perfect for:
- Healthcare (HIPAA compliance)
- Finance (SOC2/PCI requirements)
- Government (data sovereignty)
- Any sensitive corporate information

### Cost Efficiency
- **Initial setup:** $0 (using open source tools and local machine)
- **Ongoing costs:** $0 (runs on existing hardware)

### Full Control
- Choose your own AI models
- Customize response behavior
- Keep data indefinitely
- No service dependencies
- Works offline

### Learning Value
Understanding RAG systems gives you:
- Competitive advantage in AI/ML projects
- Ability to build custom AI solutions
- Knowledge of modern DevOps practices
- Hands-on experience with enterprise AI

---

## 📦 What's Included

### Documentation (50,000+ words)
- ✅ Step-by-step implementation guides
- ✅ Prerequisites for beginners
- ✅ Advanced features guide
- ✅ Visual references and diagrams
- ✅ Real troubleshooting chronicles

### Sample Files
- ✅ Working n8n workflow JSONs
- ✅ Python scripts (load, query)
- ✅ Configuration templates
- ✅ Quick setup scripts

### Tested & Verified
- ✅ Production-tested on macOS M4 
- ✅ All commands verified working
- ✅ Common errors documented
- ✅ Performance benchmarked
- ✅ Webex integration validated

---

## 🛠️ Tech Stack

### Core Components
- **Ollama** - Local LLM hosting (llama3.2:3b model)
- **ChromaDB** - Vector database (v0.4.24)
- **Podman** - Container runtime (Docker alternative)
- **n8n** - Workflow automation (visual programming)
- **Python** - Scripting and automation

### Models
- **llama3.2:3b** - Fast, efficient LLM (2GB RAM footprint)
- **nomic-embed-text** - Text embeddings (274MB)

### Integrations
- **Webex Teams** - Enterprise messaging platform
- **localhost.run** - Secure tunneling (free tier)

### Why These Choices?
- **Ollama:** Easiest local LLM deployment, no Python dependencies
- **llama3.2:3b:** Best speed/quality balance for consumer hardware
- **ChromaDB:** Simple, reliable vector DB with HTTP API
- **Podman:** Docker-compatible, better security model for macOS
- **n8n:** Visual workflow design, beginner-friendly, no code required

---

## 🎯 Project Goals

### Primary Goals (Achieved ✅)
- ✅ **Privacy-first:** No external AI API dependencies
- ✅ **Beginner-accessible:** Complete guides for IT engineers with no AI experience
- ✅ **Production-ready:** Reliable, tested, documented
- ✅ **Enterprise integration:** Webex Teams bot
- ✅ **Fast responses:** Sub-10 second query times

### Design Philosophy
- **Simplicity:** Use the simplest tool that works
- **Documentation:** Explain every step, assume no prior knowledge
- **Pragmatism:** Real solutions from real troubleshooting
- **Teachable:** Users learn while building

### What Makes This Different
Unlike typical "AI tutorials" that assume coding knowledge:
- Written for IT engineers (network/infrastructure background)
- Uses analogies to familiar concepts (routers, switches, VLANs)
- Every command explained before execution
- Comprehensive troubleshooting from real deployment experience
- pilot-ready, not just installation guide

---

## 📈 Success Stories

### Time Savings
- **Before:** Manual document searching: 15-30 minutes per query
- **After:** AI-powered answers: 5-10 seconds
- **ROI:** 100x+ time savings on repeated queries

### Capabilities Gained
- **Before:** Static documents, manual search
- **After:** 
  - Natural language queries
  - Instant answers from any document
  - Team-wide access via Webex
  - Mobile access (Webex mobile app)
  - Multiple query interfaces (CLI, web, messaging)

### Knowledge Accessibility
- **Before:** Knowledge locked in PDFs and Word documents
- **After:** 
  - Searchable knowledge base
  - Always available (24/7 with cloud deployment)
  - No learning curve for end users
  - Scales to entire team
  - Historical queries preserved

---

## 🔄 Development Status

### Current Version: 1.0 (Pilot Ready)

**Completed Features:**
- ✅ Core RAG system with vector storage
- ✅ Document loading (txt, doc, docx formats)
- ✅ Python query interface
- ✅ n8n visual workflows (3 complete workflows)
- ✅ Webex bot integration with @mention support
- ✅ Form-based document upload
- ✅ Comprehensive documentation (8 guides, 50K+ words)

**Known Limitations:**
- ⚠️ Free tunnel URLs change on restart (use paid ngrok/localhost.run for production)
- ⚠️ macOS-specific instructions (Linux/Windows need adaptation)
- ⚠️ Single collection support (multi-collection is future feature)
- ⚠️ No conversation history (stateless queries, can be added)
- ⚠️ Document format limited to text-based (PDF requires additional setup)

**Roadmap:** See [What's Next](WHATS_NEXT.md)

---

## 🤝 Contributing

### How to Contribute

**Documentation:**
- Improve clarity or fix unclear sections
- Add screenshots/diagrams
- Fix typos or formatting issues
- Share real-world use cases
- Translate to other languages

**Code:**
- Bug fixes and improvements
- Performance optimizations
- New features (see roadmap)
- Platform adaptations (Linux/Windows)
- Additional document format support

**Testing:**
- Test on different hardware configurations
- Report issues with detailed logs
- Validate installation guides
- Share feedback on documentation clarity

### Guidelines
1. Read [DOCUMENTATION_STANDARDS.md](DOCUMENTATION_STANDARDS.md) first
2. Test all changes thoroughly on clean system
3. Update relevant documentation
4. Follow existing code/documentation style
5. Provide clear commit messages describing changes

### Reporting Issues
When opening issues, please include:
- Operating system and version
- Hardware specs (CPU model, RAM)
- Which guide/step you're following
- Complete error messages (not screenshots of text)
- What you've already tried
- Output of diagnostic commands

---

## 📝 License

MIT License

**What you can do:**
- ✅ Use commercially in your organization
- ✅ Modify for your specific needs
- ✅ Distribute modified versions
- ✅ Private use within your company
- ✅ Use in consulting/training

**What you must do:**
- 📋 Include original license in distributions
- 📋 Include copyright notice
- 📋 State significant changes made

**What you cannot do:**
- ❌ Hold authors liable for any damages
- ❌ Use trademarks without permission
- ❌ Claim original authorship

See [LICENSE](LICENSE) file for full legal text.

---

## 🙏 Acknowledgments

### Technologies Used
- **Ollama** - Making local LLM deployment accessible ([ollama.ai](https://ollama.ai))
- **ChromaDB** - Simple, scalable vector database ([trychroma.com](https://trychroma.com))
- **llama3.2** - Meta's efficient language model
- **n8n** - Visual workflow automation ([n8n.io](https://n8n.io))
- **Podman** - Secure container runtime ([podman.io](https://podman.io))

### Inspiration
This project was built to demonstrate that:
- Privacy-preserving AI is practical for enterprise use
- Local deployment is viable on consumer hardware
- You don't need expensive cloud services for AI pilots
- Open source AI is production-ready
- IT engineers can build AI systems without coding backgrounds

### Built For
**Cisco Live 2026** - Demonstrating practical AI for network engineers

### Community
Thanks to everyone who:
- Tested early versions and reported issues
- Provided feedback on documentation clarity
- Shared use cases and deployment stories
- Contributed improvements and fixes

Special thanks to the open source communities behind Ollama, ChromaDB, n8n, and Podman for making this possible.

---

## 📞 Support

### Documentation Resources
- 📖 [Installation Guides](GUIDE_1_ENVIRONMENT_SETUP.md) - Complete setup instructions
- 🔧 [Troubleshooting](TROUBLESHOOTING.md) - Common issues and solutions
- 💡 [What's Next](WHATS_NEXT.md) - Advanced features and optimization
- 🎓 [Prerequisites](PREREQUISITES_AND_LEARNING.md) - Background knowledge

### Getting Help

**Before asking for help:**
1. ✅ Check the [Troubleshooting Guide](TROUBLESHOOTING.md) - Most common issues are documented
2. ✅ Search [existing GitHub issues](../../issues) - Someone may have solved your problem
3. ✅ Verify prerequisites - Did you complete all prior steps?
4. ✅ Review error messages - They often tell you exactly what's wrong

**When you need to ask:**
1. Open a [new issue](../../issues/new) with details
2. Use the issue template (helps us help you faster)
3. Include system information and complete error messages
4. Describe what you expected vs. what happened

### Community Support
- 💬 [GitHub Discussions](../../discussions) - Ask questions, share ideas
- 🐛 [Report Bugs](../../issues/new?template=bug_report.md) - Found a problem?
- ✨ [Request Features](../../issues/new?template=feature_request.md) - Ideas for improvements?

---

## ⭐ Star This Repo

If you find this project useful:
- ⭐ **Star the repository** to show support and help others discover it
- 🍴 **Fork for your own use** and customize for your organization
- 📢 **Share with colleagues** who work with sensitive documents
- 💬 **Provide feedback** to help improve the documentation
- 🤝 **Contribute back** improvements you make

**Let's make privacy-preserving AI accessible to everyone!**

---

## 📜 Version History

### v1.0.0 (December 2025) - Initial Release
**🎉 First lab-ready release**

**Features:**
- ✅ Complete documentation suite (8 comprehensive guides)
- ✅ Working RAG system with vector storage
- ✅ Webex Teams integration with bot
- ✅ Three production-ready n8n workflows
- ✅ Python CLI tools for document management
- ✅ Comprehensive troubleshooting guide

**Tested & Verified:**
- ✅ macOS Sonoma (M4 Pro, 24GB RAM)
- ✅ All installation steps validated
- ✅ Performance benchmarked
- ✅ Common errors documented and resolved

**Documentation:**
- ✅ 50,000+ words of beginner-friendly documentation
- ✅ IT analogies throughout (routers, switches, VLANs)
- ✅ Every command explained with expected output
- ✅ Real troubleshooting experiences documented

**Known Issues:**
- Free tunnel URLs require manual update after restart
- macOS-specific (Linux/Windows adaptations needed)
- Single collection limitation (multi-collection in v2.0)

---

**Built with ❤️ for IT engineers who value privacy and control**

---

## 🔐 Security & Privacy

### Data Privacy
- ✅ **Documents never leave your infrastructure**
- ✅ **No telemetry or tracking**
- ✅ **No external AI API calls**
- ✅ **Complete offline operation** (after initial setup)
- ✅ **Audit trail optional** (you control logging)

### Security Considerations
- Container isolation prevents cross-contamination
- Local execution limits attack surface
- No cloud credentials required (except optional Webex)
- Regular updates recommended for underlying components

See [Security Guide](docs/Security.md) for hardening recommendations.

---

**Ready to build your own private AI assistant?**

**👉 Start here: [Environment Setup Guide](GUIDE_1_ENVIRONMENT_SETUP.md)**

---

*Last Updated: January 2026*  
*Documentation Version: 1.0.0*  
*Project Status: Production Ready*
