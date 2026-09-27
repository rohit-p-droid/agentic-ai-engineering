# Agentic AI Engineering

> A practical, engineering-first roadmap for becoming a **GenAI / Agentic AI Engineer** by learning the fundamentals, building real systems, understanding production architecture, and solving increasingly difficult engineering challenges.

---

## What Is This?

**Agentic AI Engineering** is a hands-on learning repository for developers who want to move beyond simply calling LLM APIs and learn how to build **real-world AI systems**.

This repository covers the journey from:

```text
Python & Software Engineering
            ↓
Machine Learning
            ↓
Deep Learning
            ↓
Transformers & LLMs
            ↓
Embeddings & Vector Search
            ↓
RAG
            ↓
AI Agents
            ↓
Multi-Agent Systems
            ↓
MCP / Agent Communication
            ↓
AI System Design
            ↓
Production AI Engineering
```

The focus is **engineering**, not just theory.

You will learn concepts, implement them from scratch where useful, use modern frameworks, build projects, study production architectures, and solve increasingly difficult challenges.

---

# Why This Repository Exists

The AI ecosystem moves extremely fast.

It is easy to learn:

```python
response = llm.invoke("Hello")
```

and think you understand AI engineering.

Production systems are much more complicated.

A real AI application may involve:

```text
                    User
                      │
                      ▼
                  API Layer
                      │
                      ▼
                Application Logic
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        RAG          Agent       Tools
          │           │           │
          ▼           ▼           ▼
     Vector DB     LLMs       External APIs
          │           │           │
          └───────────┼───────────┘
                      ▼
                 Evaluation
                      │
                      ▼
               Observability
                      │
                      ▼
                Production
```

Building systems like this requires much more than prompt engineering.

You need to understand:

- Software engineering
- APIs
- Databases
- Distributed systems
- Concurrency
- Machine learning
- Deep learning
- LLMs
- Retrieval
- Agents
- Evaluation
- Observability
- System design
- Production architecture

This repository is designed to connect all of those areas.

---

# Who Is This For?

This roadmap is primarily for:

- Software developers moving into AI Engineering
- Backend developers learning GenAI
- Full-stack developers moving into AI
- ML engineers wanting stronger software engineering skills
- Developers interested in agentic systems
- Engineers targeting GenAI Engineer roles
- Engineers targeting AI Engineer roles
- Engineers interested in Forward Deployed AI Engineering
- Developers who want to build production-grade AI applications

You do **not** need to become a research scientist to follow this roadmap.

The goal is to become an engineer who can take an AI problem from:

```text
Problem
   ↓
Architecture
   ↓
Prototype
   ↓
Implementation
   ↓
Evaluation
   ↓
Production
```

---

# What You Will Learn

## 1. Software Engineering Foundations

Before building complex AI systems, you need strong engineering fundamentals.

You will learn:

```text
Python
Git
APIs
Databases
Networking
Concurrency
Testing
Debugging
System Design
Linux
Docker
```

The goal is not to learn these topics independently.

You will learn how they apply to AI systems.

---

# 2. Machine Learning Fundamentals

You will understand the core ideas behind machine learning:

```text
Datasets
Features
Labels
Training
Validation
Testing
Loss Functions
Optimization
Overfitting
Evaluation
```

You will also implement practical ML examples.

The goal is to understand what happens underneath modern AI systems.

---

# 3. Deep Learning

You will learn the foundations of neural networks:

```text
Neurons
Weights
Biases
Activation Functions
Forward Pass
Backpropagation
Gradient Descent
Optimizers
Regularization
```

Then progress toward:

```text
CNNs
RNNs
Attention
Transformers
```

---

# 4. Transformers

Transformers are fundamental to modern language models.

You will understand:

```text
Attention
Self-Attention
Query
Key
Value
Multi-Head Attention
Positional Information
Transformer Blocks
Encoder
Decoder
```

The goal is to understand transformers well enough to reason about LLM behavior and architecture.

---

# 5. Large Language Models

You will learn how modern LLMs work at a practical engineering level.

Topics include:

```text
Language Modeling
Pretraining
Inference
Context Windows
Sampling
Temperature
Instruction Tuning
Fine-Tuning
Reasoning
Model Selection
Inference Cost
Latency
```

You will also learn how to integrate LLMs into applications using APIs and SDKs.

---

# 6. Tokenization

You will understand how text becomes model input:

```text
Text
 ↓
Tokenizer
 ↓
Tokens
 ↓
Token IDs
 ↓
Model
```

This matters for:

- Context limits
- Prompt design
- Cost
- Latency
- Chunking
- RAG

---

# 7. Embeddings

You will learn how text and other information can be represented as vectors.

```text
Document
   ↓
Embedding Model
   ↓
Vector
```

Then use embeddings for:

- Semantic search
- Similarity
- Retrieval
- Clustering
- Recommendation
- RAG

---

# 8. Vector Databases

You will learn how systems store and retrieve embeddings at scale.

Topics include:

```text
Vector Search
ANN
Similarity Metrics
Metadata
Filtering
Indexing
Hybrid Search
Scaling
```

You will work with technologies such as:

- Qdrant
- pgvector
- Pinecone
- Weaviate
- Chroma

The objective is to understand the underlying concepts rather than memorize a specific database API.

---

# 9. RAG

RAG is one of the most important application patterns in GenAI.

You will build RAG systems from the ground up.

```text
Documents
    ↓
Parsing
    ↓
Cleaning
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector Database
    ↓
Retrieval
    ↓
Reranking
    ↓
Context
    ↓
LLM
    ↓
Answer
```

You will go beyond basic vector search and study:

- Query transformation
- Query expansion
- Hybrid retrieval
- Metadata filtering
- Reranking
- Context compression
- Retrieval evaluation
- Generation evaluation
- RAG observability
- Production RAG architecture

---

# 10. AI Agents

You will learn how to build systems that can reason about tasks and use tools.

A simplified agent loop:

```text
Goal
 ↓
Decide
 ↓
Select Tool
 ↓
Execute
 ↓
Observe
 ↓
Decide Again
 ↓
Finish
```

Topics include:

- Tool calling
- Agent state
- Planning
- Memory
- Reflection
- Routing
- Structured outputs
- Human-in-the-loop
- Agent evaluation
- Agent observability
- Failure handling

---

# 11. Multi-Agent Systems

You will learn when and how multiple agents can work together.

Example:

```text
                Supervisor
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   Researcher    Analyst      Writer
        │           │           │
        └───────────┼───────────┘
                    ▼
                 Reviewer
```

You will explore:

- Agent specialization
- Shared state
- Agent communication
- Parallel execution
- Supervisor patterns
- Hierarchical systems
- Reflection
- Failure recovery
- Multi-agent evaluation

You will also learn when **not** to use multi-agent architecture.

---

# 12. MCP and Agent Communication

Modern AI systems increasingly need standardized ways to connect models with external capabilities.

You will learn concepts around:

```text
MCP
Tool Servers
Resources
Structured Interfaces
Agent Communication
A2A
Service Boundaries
```

The goal is to understand how AI systems interact with external systems at scale.

---

# 13. AI Frameworks

You will learn how to use frameworks without becoming dependent on abstractions you don't understand.

Frameworks covered may include:

```text
LangChain
LangGraph
LlamaIndex
OpenAI SDK
Pydantic AI
CrewAI
Agno
```

For each framework, the focus is:

```text
Concept
 ↓
Simple Example
 ↓
Architecture
 ↓
Production Usage
 ↓
Trade-offs
```

The repository will emphasize understanding the underlying architecture first.

---

# 14. AI System Design

Eventually, you need to move from:

> "How do I implement this?"

to:

> "How should this system be designed?"

You will learn to design systems involving:

```text
LLM APIs
RAG
Agents
Vector Databases
Queues
Workers
Caching
Databases
Observability
Authentication
Rate Limiting
```

You will practice designing systems such as:

- Enterprise RAG platforms
- AI customer-support systems
- Document intelligence platforms
- Multi-agent research systems
- AI coding assistants
- Voice AI systems
- AI workflow automation platforms

---

# 15. Production AI Engineering

A prototype is not a production system.

You will learn how to handle:

```text
Latency
Cost
Reliability
Retries
Timeouts
Rate Limits
Caching
Observability
Security
Evaluation
Scaling
Data Quality
Model Failures
```

You will learn to think about:

```text
                    Quality
                       │
                       ▼
Cost ─────────── Production ─────────── Reliability
                       │
                       ▼
                    Latency
```

AI engineering is often about balancing these competing constraints.

---

# 16. Evaluation

One of the biggest differences between traditional software and AI systems is that AI outputs are often not deterministic.

Therefore:

```text
"It works on my example"
```

is not enough.

You will learn:

```text
Evaluation Datasets
Golden Sets
LLM-as-a-Judge
Retrieval Metrics
Generation Metrics
Regression Testing
Agent Evaluation
Tool Selection Evaluation
Quality / Cost / Latency Analysis
```

The goal is to make AI systems measurable.

---

# 17. Observability

When an AI system fails, you need to know **where** it failed.

A production trace might look like:

```text
Request
  │
  ├── Query Rewrite        120ms
  │
  ├── Retrieval             80ms
  │
  ├── Reranking             90ms
  │
  ├── LLM Call             850ms
  │
  └── Tool Call             60ms
```

You will learn to monitor:

- Latency
- Token usage
- Cost
- Errors
- Retrieval quality
- Agent decisions
- Tool calls
- Model responses
- System throughput

---

# 18. Projects

This repository is project-driven.

Projects will progress from:

```text
Beginner
   ↓
Intermediate
   ↓
Advanced
   ↓
Production
```

Examples include:

### Beginner

- LLM API client
- Document processor
- Semantic search
- AI chatbot

### Intermediate

- RAG application
- AI research assistant
- Tool-calling agent
- Document intelligence system

### Advanced

- Multi-agent research platform
- Agentic RAG
- AI workflow engine
- Production evaluation platform

### Production

Complete systems combining:

```text
API
+
Database
+
RAG
+
Agents
+
Tools
+
Evaluation
+
Observability
+
Authentication
+
Deployment
```

---

# 19. Challenges

Reading is not enough.

This repository will contain increasingly difficult challenges.

Examples:

```text
Build a tokenizer
Build an embedding search engine
Implement attention
Build RAG from scratch
Implement hybrid retrieval
Build an agent loop
Build a tool registry
Build multi-agent orchestration
Design a scalable RAG system
Debug a broken AI pipeline
Optimize an expensive agent
```

The difficulty should increase as you progress.

---

# 20. Interview Preparation

The repository also includes interview preparation for:

```text
Python
Machine Learning
Deep Learning
LLMs
RAG
Agents
System Design
Coding
Production AI
```

The focus is not memorizing interview answers.

You should be able to explain:

```text
Why?
How?
Trade-offs?
Failure modes?
Scaling?
Cost?
Latency?
Reliability?
```

---

# Repository Structure

```text
agentic-ai-engineering/
│
├── README.md
├── CONTRIBUTING.md
├── LICENSE
├── CHANGELOG.md
│
├── assets/
│
├── roadmap/
│
├── fundamentals/
│   ├── 01-python-for-ai/
│   ├── 02-machine-learning/
│   ├── 03-deep-learning/
│   ├── 04-transformers/
│   ├── 05-llms/
│   ├── 06-tokenization/
│   ├── 07-embeddings/
│   ├── 08-vector-databases/
│   └── 09-prompt-engineering/
│
├── rag/
│
├── agents/
│
├── frameworks/
│
├── projects/
│
├── system-design/
│
├── interviews/
│
├── papers/
│
├── tools/
│
├── cheatsheets/
│
├── blogs/
│
├── resources/
│
└── examples/
```

Each section is designed to build on the previous ones.

---

# How To Use This Repository

Do not treat this repository like a checklist where you read every Markdown file and move on.

Use this cycle:

```text
             ┌──────────────┐
             │    Learn     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │    Build     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │  Experiment  │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │    Break     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │    Debug     │
             └──────┬───────┘
                    ↓
             ┌──────────────┐
             │   Improve    │
             └──────┬───────┘
                    │
                    └──────────────→
```

For every major topic:

1. Understand the theory.
2. Implement a small example.
3. Build something practical.
4. Intentionally break it.
5. Debug it.
6. Measure it.
7. Improve it.
8. Apply it to a larger AI system.

---

# The Engineering Mindset

This roadmap is built around a simple principle:

> **Don't just learn how to make AI work. Learn how to make AI systems reliable.**

When learning a technology, ask:

### 1. What problem does it solve?

### 2. How does it work internally?

### 3. When should I use it?

### 4. When should I avoid it?

### 5. What are its trade-offs?

### 6. How does it fail?

### 7. How do I test it?

### 8. How do I observe it?

### 9. How does it scale?

### 10. What does it cost?

This mindset is more important than memorizing frameworks.

---

# Build From First Principles

Frameworks are useful.

But you should understand what happens underneath them.

For example, before relying entirely on a RAG framework, understand:

```text
Document
 ↓
Chunking
 ↓
Embedding
 ↓
Vector Storage
 ↓
Similarity Search
 ↓
Ranking
 ↓
Context
 ↓
LLM
```

Before using an agent framework, understand:

```text
Model
 ↓
Decision
 ↓
Tool Selection
 ↓
Tool Execution
 ↓
Observation
 ↓
State Update
 ↓
Next Decision
```

Frameworks should accelerate your engineering, not replace your understanding.

---

# Production-First Thinking

A demo might look like:

```python
response = llm.invoke(prompt)
```

A production system needs to answer:

```text
What if the model times out?

What if the provider rate-limits us?

What if the response is malformed?

What if retrieval returns irrelevant documents?

What if the agent loops?

What if a tool fails?

What if the request takes 30 seconds?

What if 10,000 users arrive simultaneously?

What if the model becomes more expensive?

How do we evaluate quality?

How do we trace the failure?

How do we roll back a prompt change?
```

These questions are at the heart of this roadmap.

---

# Recommended Progression

The overall journey is:

```text
PHASE 1
Software Engineering
        ↓
Python + Git + APIs + Databases
        ↓

PHASE 2
AI Fundamentals
        ↓
ML + Deep Learning + Transformers
        ↓

PHASE 3
LLM Engineering
        ↓
LLMs + Tokenization + Embeddings
        ↓

PHASE 4
RAG Engineering
        ↓
Retrieval + Reranking + Evaluation
        ↓

PHASE 5
Agent Engineering
        ↓
Tools + Memory + Planning + Workflows
        ↓

PHASE 6
Multi-Agent Engineering
        ↓
Communication + Coordination + MCP
        ↓

PHASE 7
Production AI
        ↓
System Design + Evaluation + Observability
        ↓

PHASE 8
Advanced Projects
        ↓
Real-world AI Systems
```

---

# What "Finished" Means

You are not finished when you have read every file.

You are approaching the goal when you can independently:

- Understand an AI problem.
- Break it into engineering components.
- Choose appropriate technologies.
- Build a working prototype.
- Evaluate its quality.
- Identify failure modes.
- Improve latency and cost.
- Add observability.
- Handle production failures.
- Scale the architecture.
- Explain your technical decisions.

Ultimately:

```text
Problem
   ↓
Requirements
   ↓
Architecture
   ↓
Implementation
   ↓
Evaluation
   ↓
Deployment
   ↓
Monitoring
   ↓
Iteration
```

That is the real skill this repository is trying to develop.

---

# A Note on Frameworks

Frameworks will change.

Models will change.

APIs will change.

Libraries will change.

The fundamentals will remain much more stable.

Therefore, prioritize:

```text
Concepts
   >
Architecture
   >
Engineering Principles
   >
Frameworks
```

Learn frameworks deeply enough to use them effectively, but do not build your entire understanding around a single framework.

---

# Contribution & Learning in Public

This repository can also serve as a public engineering learning log.

As you learn:

- Add implementations.
- Document experiments.
- Record failures.
- Add diagrams.
- Add benchmarks.
- Document architectural decisions.
- Add interview questions.
- Improve explanations.
- Build increasingly difficult projects.

The goal is not to make the repository look perfect.

The goal is to show **how engineering understanding develops over time**.

---

# Final Goal

By the end of this roadmap, you should be able to look at a problem such as:

> "Build an AI system that can understand a company's internal knowledge, retrieve relevant information, reason over it, use external tools, collaborate across specialized agents, and operate reliably in production."

and break it down into:

```text
                    User
                      │
                      ▼
                  API Layer
                      │
                      ▼
               Orchestration
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
         RAG        Agents       Tools
          │           │           │
          ▼           ▼           ▼
     Vector DB     LLMs      External APIs
          │           │           │
          └───────────┼───────────┘
                      ▼
                  Evaluation
                      │
                      ▼
                Observability
                      │
                      ▼
                 Production
```

and understand **why every component exists, how the components interact, how the system can fail, and how to improve it**.

That is the target.

---

# Start Here

If you are new to the roadmap:

```text
roadmap/
    ↓
fundamentals/
    ↓
fundamentals/01-python-for-ai/
    ↓
fundamentals/02-machine-learning/
```

If you already have strong software engineering fundamentals, move faster through the introductory material and spend more time on:

```text
RAG
Agents
System Design
Evaluation
Production Engineering
Projects
```

---

## The Principle

> **Learn deeply. Build constantly. Break things deliberately. Measure everything. Understand the system underneath the abstraction.**

Welcome to **Agentic AI Engineering**.

