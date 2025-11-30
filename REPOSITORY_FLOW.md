# TheGradientPath - Repository Flow Documentation

## 📋 Overview

**TheGradientPath** is a comprehensive educational repository covering modern Machine Learning, Deep Learning, and AI from fundamentals to production systems. This document outlines the complete flow and architecture of all projects within this repository.

---

## 🏗️ Repository Architecture

```
TheGradientPath/
├── Root Level Scripts          # Quick-start tutorials
├── AiAgents/                   # AI Agent frameworks & benchmarks
├── Keras/                      # Deep Learning with TensorFlow/Keras
├── Pytotch/                    # PyTorch implementations
├── LLMFineTuning/              # LLM fine-tuning techniques
├── Rag/                        # RAG systems & retrieval methods
├── MCPFromScratch/             # Model Context Protocol implementation
├── Text2SQL/                   # Natural language to SQL
└── RealWorldProjects/          # Production-ready deployments
```

---

## 🔄 Project Flows by Category

### 1. 🤖 AI Agents (`AiAgents/`)

#### **Agent Framework Benchmark Flow**

**Purpose**: Compare 7 major AI agent frameworks using identical multi-agent systems

**Architecture Flow**:
```
User Query
    ↓
Orchestrator Agent (Routing Decision)
    ↓
    ├── Legal Expert Agent → Legal domain questions
    ├── Operational Agent → Programming & general queries
    └── Tool Integration
            ↓
        ├── Weather API
        ├── Calculator
        ├── Web Search
        └── MCP Server Integration
```

**Frameworks Evaluated**:
1. **LangChain/LangGraph** - State machine architecture, maximum flexibility
2. **OpenAI Agents** - Native MCP support, minimal code
3. **CrewAI** - Simple delegation, rapid prototyping
4. **LlamaIndex** - Balanced workflow architecture
5. **AutoGen** - Enterprise async infrastructure
6. **Semantic Kernel** - Microsoft ecosystem integration
7. **Vanilla Python** - Zero framework baseline

**Key Components**:
- Agent orchestration & routing
- Tool integration (custom + built-in)
- State management across agents
- Memory management (conversation history)
- MCP server integration
- Token tracking & guardrails

**Evaluation Metrics**:
- Code complexity & readability
- Developer experience
- Documentation quality
- Feature completeness
- Setup complexity
- Flexibility & abstraction level

---

### 2. 🧠 Deep Learning with Keras (`Keras/`)

#### **Image Classification with MLP Flow**

**Dataset**: MNIST handwritten digits

**Pipeline Flow**:
```
Raw MNIST Images (28x28 grayscale)
    ↓
Normalization & Preprocessing
    ↓
Multi-Layer Perceptron
    ├── Dense layers
    ├── Dropout regularization
    └── BatchNormalization
    ↓
Softmax Classification (10 classes)
    ↓
TensorBoard Logging & Visualization
```

**Key Features**:
- Functional API architecture
- Visualkeras diagram generation
- Comprehensive metrics tracking

---

#### **Transformer-Based Text Generation Flow**

**Path**: `Keras/transformers/text_generation/`

**Architecture Flow**:
```
Text Input
    ↓
Tokenization & Preprocessing
    ↓
Embedding Layer + Positional Encoding
    ↓
Multi-Head Self-Attention
    ├── Query, Key, Value projections
    ├── Attention scores computation
    └── Multi-head concatenation
    ↓
Feed-Forward Network
    ├── Dense(hidden_dim)
    ├── ReLU activation
    └── Dense(embed_dim)
    ↓
Layer Normalization + Residual Connections
    ↓
Output Projection Layer
    ↓
Text Generation (token sampling)
```

**Components**:
- Complete Transformer from scratch
- Custom training loop
- Text preprocessing pipeline
- Generation sampling strategies

---

#### **Text Generation with KV Cache Flow**

**Path**: `Keras/transformers/kv_cache_for text_gen/`

**Optimization Flow**:
```
Generation Loop (Token-by-token)
    ↓
Standard Transformer          KV Cache Transformer
    ├── Recompute all K,V    →    ├── Reuse cached K,V
    ├── O(n²) complexity     →    ├── O(n) complexity
    └── Slow inference       →    └── Fast inference
    ↓
Cached Key-Value States
    ├── Store previous computations
    ├── Append new K,V pairs
    └── Dramatically reduced computation
```

**Performance Benefits**:
- Reduced inference latency
- Lower memory overhead during generation
- Production-ready LLM optimization

---

#### **Time Series Forecasting Flow**

**Path**: `Keras/transformers/time_series_forecast/`

**Pipeline Flow**:
```
Synthetic Stock Price Data
    ↓
MinMax Scaling (0-1 normalization)
    ↓
Sequence Creation (sliding window)
    ↓
Transformer Architecture
    ├── Temporal embeddings
    ├── Multi-head attention (temporal dependencies)
    └── Feed-forward network
    ↓
Price Prediction (future values)
    ↓
Inverse scaling & Visualization
```

---

### 3. 🔥 PyTorch Projects (`Pytotch/`)

#### **CNN Image Classification Flow**

**Dataset**: Fashion-MNIST (70,000 images, 10 clothing categories)

**Training Pipeline**:
```
Fashion-MNIST Dataset (28x28 → resize to 16x16)
    ↓
Data Augmentation (optional)
    ↓
CNN Architecture
    ├── Conv2d(1→16) + BatchNorm + ReLU + MaxPool
    ├── Conv2d(16→32) + BatchNorm + ReLU + MaxPool
    └── Fully Connected(512→10) + BatchNorm
    ↓
Cross-Entropy Loss + SGD Optimizer
    ↓
Training Loop (with validation)
    ↓
Model Checkpoint (fashion_mnist_cnn.pth)
    ↓
Inference & Evaluation
```

**Key Features**:
- Batch normalization for stability
- Proper train/validation split
- Model persistence & loading
- Real-time training metrics

---

### 4. 🎯 LLM Fine-Tuning (`LLMFineTuning/`)

#### **All PEFT Techniques from Scratch**

**Path**: `LLMFineTuning/all_peft_tecniques_from_scratch/`

**Implementation Flow**:
```
Base Transformer Model
    ↓
PEFT Technique Selection
    ├── LoRA (Low-Rank Adaptation)
    ├── Prefix Tuning
    ├── P-Tuning
    ├── Adapter Layers
    └── BitFit
    ↓
Parameter-Efficient Training
    ├── Freeze base model weights
    ├── Train only adapter parameters
    └── Minimal memory footprint
    ↓
Fine-tuned Model (small delta weights)
    ↓
Inference with merged weights
```

**PEFT Techniques Covered**:
- **LoRA**: Low-rank matrices for weight updates
- **Prefix Tuning**: Learnable prefix tokens
- **Adapter Layers**: Small bottleneck layers
- **BitFit**: Bias-only fine-tuning

---

#### **GRPO Reasoning with Unsloth**

**Path**: `LLMFineTuning/GRPO_REASONING_UNSLOTH/`

**Training Flow**:
```
Reasoning Dataset
    ↓
Unsloth Optimization
    ├── 2x faster training
    ├── 50% memory reduction
    └── Flash Attention 2
    ↓
GRPO (Group Relative Policy Optimization)
    ├── Reward modeling
    ├── Policy gradient optimization
    └── Reasoning capability enhancement
    ↓
Fine-tuned Reasoning Model
```

**Key Optimizations**:
- Unsloth fast inference engine
- GRPO for improved reasoning
- Memory-efficient training

---

#### **SFT with HuggingFace Tool Choice**

**Path**: `LLMFineTuning/SFT_HF_TOOL_CHOICE/`

**Workflow**:
```
Tool-Calling Dataset Generation
    ↓
Dataset Format (gen_dataset.py)
    ├── User queries
    ├── Tool definitions
    └── Expected tool selections
    ↓
Supervised Fine-Tuning (SFT)
    ├── HuggingFace Trainer
    ├── Tool choice classification
    └── Parameter-efficient training
    ↓
Tool-Aware Language Model
    ↓
Inference: Auto-select appropriate tools
```

**Root Level Scripts**:
- `llm_class_tutorial.py`: Classification with SmolLM2-135M
- `llm_sft_lora.py`: LoRA fine-tuning implementation

---

### 5. 📖 RAG Systems (`Rag/`)

#### **Dartboard RAG Flow**

**Path**: `Rag/dartboard/`

**Retrieval Pipeline**:
```
User Query
    ↓
Query Embedding
    ↓
Vector Database Search
    ↓
Dartboard Algorithm
    ├── Relevance Scoring
    ├── Diversity Scoring
    └── Combined Optimization
        ↓
        Select top-k documents
        ├── Balance relevance vs diversity
        ├── Avoid redundant information
        └── Maximize information gain
    ↓
Retrieved Documents (non-redundant)
    ↓
LLM Context Injection
    ↓
Generated Answer
```

**Key Components**:
- `ingestion.py`: Document embedding & vector store creation
- `retrieval.py`: Dartboard algorithm implementation
- `main.py`: End-to-end pipeline orchestration

**Algorithm**:
- Optimize: `Score = α * Relevance + β * Diversity`
- Prevents redundant document retrieval
- Configurable relevance/diversity weights

---

#### **Hybrid Multivector Knowledge Graph RAG**

**Path**: `Rag/hybrid_multivector_knowledge_graph_rag/`

**Advanced Retrieval Flow**:
```
Document Ingestion
    ↓
    ├── Text Chunking
    ├── Entity Extraction (LLM-powered)
    ├── Relationship Extraction
    └── Multi-Vector Embeddings
        ↓
Knowledge Graph Construction (Neo4j)
    ├── Entity nodes
    ├── Relationship edges
    └── Document chunks with embeddings
        ↓
Query Processing
    ├── Entity recognition in query
    ├── Graph traversal algorithm selection
    └── LLM-powered Cypher generation
        ↓
Hybrid Retrieval
    ├── Vector Similarity Search
    ├── Graph Traversal (BFS, DFS, A*, Beam Search)
    ├── Causal Chain Discovery
    └── Multi-hop Reasoning
        ↓
Context Aggregation
    ↓
LLM Generation with Graph Context
```

**11+ Graph Traversal Algorithms**:
1. **BFS** (Breadth-First Search)
2. **DFS** (Depth-First Search)
3. **A*** (Heuristic pathfinding)
4. **Beam Search** (Top-k expansion)
5. **Context-to-Cypher** (LLM query generation)
6. **Causal Chain Traversal**
7. **Similarity-based expansion**
8. **Multi-hop reasoning**
9. **Entity-centric retrieval**
10. **Relationship filtering**
11. **Hybrid scoring**

**Components**:
- `ingestion.py`: Graph construction & embedding
- `query.py`: Hybrid retrieval implementation
- `traversal/`: Graph algorithm implementations
- `prompts.py`: LLM prompt templates

---

#### **Vision RAG**

**Path**: `Rag/vision_rag/`

**Multimodal RAG Flow**:
```
PDF Documents with Images
    ↓
PDF Processing (pdf_processor.py)
    ├── Text extraction
    ├── Image extraction
    └── Layout analysis
    ↓
Multimodal Embeddings
    ├── Text: OpenAI text embeddings
    └── Images: CLIP/vision embeddings
    ↓
PostgreSQL + pgvector Storage
    ├── Document metadata
    ├── Text embeddings
    └── Image embeddings
    ↓
Query Processing
    ├── Text query embedding
    ├── Image query embedding (if applicable)
    └── Hybrid similarity search
    ↓
Retrieved Context (text + images)
    ↓
Vision-Language Model (VLM)
    ↓
Multimodal Answer Generation
```

**Infrastructure**:
- Docker Compose setup (PostgreSQL + pgvector)
- `init_db.py`: Database initialization
- `ingestion.py`: Document processing pipeline
- `query.py`: Multimodal retrieval

---

### 6. 🔌 MCP From Scratch (`MCPFromScratch/`)

**Model Context Protocol Implementation**

**System Architecture Flow**:
```
Client Application
    ↓
WebSocket Connection
    ↓
MCP Protocol Handshake
    ├── Capabilities negotiation
    ├── Tool discovery
    └── Resource listing
    ↓
MCP Server (FastAPI)
    ├── Tool Endpoints
    │   ├── Calculator
    │   ├── Database queries
    │   └── Text-to-SQL
    ├── Prompt Templates
    └── Resource Access
        ↓
Session Management (SQLite)
    ├── API key validation
    ├── Usage tracking
    └── Quota enforcement
        ↓
Tool Execution
    ↓
Response to Client
    ↓
LLM Integration (Client-side)
    ├── Tool selection
    ├── Parameter extraction
    └── Natural language response
```

**Components**:

**Server** (`server/`):
- `server.py`: FastAPI WebSocket server
- `server_session.py`: Session management
- `base_session.py`: Protocol base classes
- `protocol_types.py`: Pydantic models
- `initdb.py`: Database initialization

**Client** (`client/`):
- `main_agent.py`: LLM-powered agent
- `main_basic.py`: Simple client example
- `protocol_types.py`: Client-side models

**Key Concepts**:
- Custom protocol design
- WebSocket persistent connections
- Async/await patterns
- API authentication & quotas
- LLM tool integration

---

### 7. 🗃️ Text2SQL (`Text2SQL/`)

#### **Semantic Text2SQL Flow**

**Path**: `Text2SQL/SemanticText2SQL/`

**Query Processing Pipeline**:
```
Natural Language Query
    ↓
Query Embedding
    ↓
Schema Embedding Database
    ├── Table embeddings
    ├── Column embeddings
    └── Relationship embeddings
    ↓
Semantic Similarity Search
    ├── Retrieve relevant tables
    ├── Retrieve relevant columns
    └── Retrieve join relationships
    ↓
Schema Context Construction
    ↓
LLM SQL Generation
    ├── Schema-aware prompt
    ├── Few-shot examples
    └── SQL syntax validation
    ↓
Generated SQL Query
    ↓
Database Execution (PostgreSQL)
    ↓
Result Formatting & Response
```

**Components**:
- `gen_embeddings.py`: Schema embedding generation
- `db_schema.txt`: Database schema definition
- Docker Compose: PostgreSQL setup
- `init-db.sql`: Database initialization

**Key Features**:
- Schema-aware embedding search
- Context-based SQL generation
- Support for complex joins
- Semantic understanding of queries

---

### 8. 🌐 Real-World Production Projects (`RealWorldProjects/`)

#### **Cyber Attack Prediction System**

**Path**: `RealWorldProjects/CyberAttackPrediction/`

**End-to-End Production Flow**:
```
Network Traffic Capture (Scapy)
    ↓
monitor-app (Next.js + Network Agent)
    ├── Packet capture
    ├── Feature extraction
    │   ├── Protocol type
    │   ├── Packet size
    │   ├── Flow duration
    │   └── Port numbers
    └── HTTP POST to ml-service
        ↓
ml-service (Flask API)
    ├── Feature preprocessing
    ├── AutoEncoder (Anomaly Detection)
    │   ├── Normal traffic reconstruction
    │   └── High error = anomaly
    └── SGD Classifier (Attack Type)
        ├── DDoS
        ├── Port Scan
        ├── Intrusion
        └── Benign
        ↓
Prediction Response
    ↓
Web Dashboard (Next.js UI)
    ├── Real-time metrics
    ├── Attack visualization
    └── Alert management
```

**AWS Infrastructure (CloudFormation)**:
```
GitHub Repository
    ↓
CodePipeline (CI/CD)
    ├── Source Stage (GitHub)
    ├── Build Stage (CodeBuild)
    └── Deploy Stage (CodeDeploy)
        ↓
Application Load Balancer
    ├── HTTPS listener (port 443)
    └── HTTP listener (port 80)
        ↓
Auto Scaling Groups
    ├── monitor-app instances
    └── ml-service instances
        ↓
Target Groups (Health Checks)
    ↓
Production Traffic Handling
```

**Infrastructure Components**:
- **CF_NETWORK_ATTACK_PREDICTION.yml**: Complete CloudFormation template
- **Auto Scaling**: Dynamic capacity based on traffic
- **Load Balancing**: High availability
- **CI/CD**: Automated deployments
- **Security Groups**: Network isolation
- **IAM Roles**: Secure permissions

**Deployment Flow**:
1. Push code to GitHub
2. CodePipeline triggers automatically
3. CodeBuild compiles and tests
4. CodeDeploy deploys to EC2 instances
5. Health checks ensure successful deployment
6. Load balancer routes traffic to new instances

---

## 🔗 Cross-Project Integration Patterns

### **Common Architectural Patterns**

1. **Embedding-Based Retrieval**
   - Used in: RAG systems, Text2SQL, Vision RAG
   - Flow: Text/Image → Embedding → Vector DB → Similarity Search

2. **LLM Agent Orchestration**
   - Used in: AI Agents, MCP, Tool-calling LLMs
   - Flow: User Query → Agent Routing → Tool Selection → Execution

3. **Multi-Stage Pipelines**
   - Used in: All projects
   - Flow: Input → Preprocessing → Model Inference → Post-processing → Output

4. **Production Deployment**
   - Used in: Cyber Attack Prediction
   - Flow: GitHub → CI/CD → Build → Test → Deploy → Monitor

---

## 🛠️ Technology Stack Overview

### **Frameworks & Libraries**
- **Deep Learning**: TensorFlow/Keras, PyTorch
- **LLM**: HuggingFace Transformers, Unsloth, OpenAI API
- **Agent Frameworks**: LangChain, CrewAI, AutoGen, LlamaIndex, Semantic Kernel
- **Vector DBs**: ChromaDB, pgvector, Neo4j
- **Web**: FastAPI, Next.js, Flask
- **Deployment**: AWS (CloudFormation, CodePipeline, EC2, ALB)

### **Key Techniques**
- **Fine-tuning**: LoRA, PEFT, SFT, GRPO
- **RAG**: Vector search, Graph traversal, Dartboard algorithm
- **Attention**: Multi-head self-attention, KV cache
- **Optimization**: Batch normalization, Dropout, Early stopping
- **ML**: CNNs, Transformers, AutoEncoders, Classification

---

## 📊 Learning Path Recommendation

### **Beginner Path**
1. `Pytotch/CnnImageClassification/` - Basic CNN concepts
2. `Keras/ImageClassificationWithMLP/` - Neural network fundamentals
3. `llm_class_tutorial.py` - LLM classification basics

### **Intermediate Path**
1. `Keras/transformers/text_generation/` - Transformer architecture
2. `Rag/dartboard/` - RAG fundamentals
3. `LLMFineTuning/SFT_HF_TOOL_CHOICE/` - Fine-tuning techniques
4. `MCPFromScratch/` - Client-server architecture

### **Advanced Path**
1. `AiAgents/AgentFrameworkBenchmark/` - Multi-agent systems
2. `Rag/hybrid_multivector_knowledge_graph_rag/` - Advanced retrieval
3. `LLMFineTuning/GRPO_REASONING_UNSLOTH/` - Advanced fine-tuning
4. `RealWorldProjects/CyberAttackPrediction/` - Production deployment

---

## 🎯 Use This Document For

- **Understanding project architecture** before diving into code
- **Planning learning paths** based on skill level
- **Identifying relevant components** for specific use cases
- **Prompting LLMs** with complete context of repository structure
- **Onboarding new contributors** to the repository
- **Designing similar systems** using proven patterns

---

## 📝 Quick Reference

### **Project Selection Guide**

| Goal | Recommended Project |
|------|-------------------|
| Learn Transformers from scratch | `Keras/transformers/text_generation/` |
| Build production RAG | `Rag/hybrid_multivector_knowledge_graph_rag/` |
| Compare agent frameworks | `AiAgents/AgentFrameworkBenchmark/` |
| Deploy ML to AWS | `RealWorldProjects/CyberAttackPrediction/` |
| Understand PEFT | `LLMFineTuning/all_peft_tecniques_from_scratch/` |
| Build custom protocol | `MCPFromScratch/` |
| Optimize LLM inference | `Keras/transformers/kv_cache_for text_gen/` |
| Learn graph-based retrieval | `Rag/hybrid_multivector_knowledge_graph_rag/` |

---

## 🔄 Continuous Updates

This repository is actively maintained with:
- **Video tutorials** for major projects (YouTube links in READMEs)
- **Jupyter notebooks** for interactive learning
- **Production-ready code** with comprehensive documentation
- **Regular updates** with latest techniques and frameworks

---

**Last Updated**: November 2025
**Maintained By**: TheGradientPath Community
