# Python Project Structure for AI Engineering

## Why does an AI Engineer need this?

An AI prototype can start with:

```text id="1m4p8x"
app.py
```

But production AI applications quickly grow.

A simple RAG application may eventually contain:

```text id="q7z3km"
API
Agents
LLM clients
Prompts
RAG pipeline
Retrievers
Vector databases
Tools
Background jobs
Database models
Authentication
Configuration
Logging
Testing
Evaluation
Observability
```

Without a clear structure, the project can become:

```text id="j8v2qa"
main.py
utils.py
utils2.py
helper.py
helper_final.py
agent.py
agent_new.py
```

Good project structure is not about creating many folders.

It is about making **responsibilities, dependencies, and boundaries clear**.

For AI engineers, this becomes especially important because AI applications combine:

```text id="6w3p9r"
Application code
+
AI workflows
+
External services
+
Data pipelines
+
Infrastructure
```

---

# 1. Start Simple

A small AI application does not need a huge architecture.

For example:

```text id="n8k4vc"
my-ai-app/
├── app.py
├── requirements.txt
└── README.md
```

This can be perfectly reasonable for a prototype.

The problem begins when everything gets placed into `app.py`.

---

# 2. A Growing AI Application

As the application grows:

```text id="w2q7ms"
my-ai-app/
├── app.py
├── llm.py
├── rag.py
├── agents.py
├── tools.py
├── database.py
├── config.py
├── prompts.py
└── utils.py
```

This is better, but responsibilities can still become mixed.

For example:

```text id="3g8n6f"
rag.py
```

might contain:

```text id="4h2x9v"
Document loading
Chunking
Embedding
Retrieval
Prompt construction
LLM calls
Response formatting
```

That becomes difficult to maintain.

---

# 3. Separate Responsibilities

A better approach is to organize code around responsibilities.

For example:

```text id="0b8p3s"
app/
├── api/
├── agents/
├── rag/
├── llms/
├── tools/
├── database/
├── config/
└── utils/
```

Now each area has a clearer purpose.

---

# 4. Recommended Production Structure

A practical structure for an AI application:

```text id="u4c9kz"
project/
│
├── src/
│   └── app/
│       ├── api/
│       ├── agents/
│       ├── rag/
│       ├── llms/
│       ├── tools/
│       ├── database/
│       ├── models/
│       ├── services/
│       ├── prompts/
│       ├── config/
│       └── main.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── evals/
│
├── scripts/
├── docs/
├── .env.example
├── .gitignore
├── pyproject.toml
├── Dockerfile
├── README.md
└── LICENSE
```

This is a starting point, not a mandatory template.

The correct structure depends on application size and architecture.

---

# 5. Why Use `src/`?

The `src/` layout separates application source code from project-level files.

Example:

```text id="g2v8dm"
project/
├── src/
│   └── app/
├── tests/
├── scripts/
├── pyproject.toml
└── README.md
```

This helps prevent accidental imports from the repository root and makes package installation/testing behavior more representative of a real installed package.

For larger Python projects, `src/` is a useful convention.

---

# 6. The `api/` Directory

The API layer handles communication with clients.

Example:

```text id="x8m4qb"
api/
├── routes/
│   ├── chat.py
│   ├── documents.py
│   └── search.py
├── dependencies.py
└── schemas.py
```

Responsibilities:

```text id="2s5x7p"
Request validation
Authentication
Authorization
HTTP responses
Dependency injection
API-specific schemas
```

The API layer should generally not contain the entire AI workflow.

Avoid:

```python id="v6q2jm"
@app.post("/chat")
def chat(request):

    # 300 lines of:
    # retrieval
    # prompt construction
    # LLM calls
    # database operations
    # agent logic
```

Instead:

```python id="n7w3kc"
@app.post("/chat")
def chat(request):
    return chat_service.execute(request)
```

Keep transport concerns separate from business logic.

---

# 7. The `agents/` Directory

Agent logic belongs here.

Example:

```text id="x5p9nd"
agents/
├── supervisor.py
├── researcher.py
├── writer.py
├── reviewer.py
├── state.py
└── graph.py
```

Depending on the application, this may contain:

```text id="r7c3mz"
Agent definitions
Agent state
Graph definitions
Routing logic
Agent prompts
Tool selection
Workflow configuration
```

Do not put every function containing an LLM call into `agents/`.

A normal LLM service and an autonomous workflow are different concepts.

---

# 8. The `rag/` Directory

RAG-specific functionality can be organized as:

```text id="j3w8sx"
rag/
├── ingestion/
│   ├── loaders.py
│   ├── parser.py
│   ├── chunking.py
│   └── pipeline.py
│
├── retrieval/
│   ├── vector.py
│   ├── keyword.py
│   ├── hybrid.py
│   └── reranker.py
│
├── embeddings.py
├── context.py
└── pipeline.py
```

This makes the pipeline easier to reason about:

```text id="v5r1xq"
Ingestion
   ↓
Chunking
   ↓
Embedding
   ↓
Storage
   ↓
Retrieval
   ↓
Reranking
   ↓
Context construction
```

---

# 9. The `llms/` Directory

The LLM layer can isolate provider-specific code.

Example:

```text id="p9n2cx"
llms/
├── client.py
├── openai.py
├── embeddings.py
└── factory.py
```

For example:

```python id="a4m8zk"
class LLMClient:
    ...
```

Then application code can depend on an abstraction rather than directly scattering provider-specific calls throughout the codebase.

Conceptually:

```text id="s6v3dy"
Agent
  ↓
LLM interface
  ↓
Provider implementation
  ↓
LLM API
```

This can make testing and provider changes easier.

Do not create abstractions merely for theoretical future flexibility. Add them where they provide a real boundary.

---

# 10. The `tools/` Directory

Agent tools should have clear boundaries.

Example:

```text id="z8k4mp"
tools/
├── search.py
├── database.py
├── email.py
├── calculator.py
└── registry.py
```

A tool should ideally have:

```text id="q2m7nc"
Clear input
Clear output
Explicit permissions
Error handling
Observable execution
```

For example:

```python id="w4x8ps"
def search_documents(query: str):
    ...
```

Agent logic should not need to know every implementation detail.

---

# 11. The `database/` Directory

Database-related code can be separated:

```text id="c7n2vq"
database/
├── connection.py
├── models.py
├── repositories/
│   ├── documents.py
│   └── users.py
└── migrations/
```

Responsibilities might include:

```text id="3x9pfa"
Connection management
Transactions
ORM models
Queries
Repositories
Migrations
```

Do not mix database access directly into every agent node.

Instead:

```text id="h8v2jd"
Agent
  ↓
Service
  ↓
Repository
  ↓
Database
```

This creates clearer boundaries.

---

# 12. Models vs Schemas

The word "model" can mean different things in AI applications.

You may have:

```text id="r5k1ws"
Database models
Pydantic schemas
LLM output models
AI model clients
```

Avoid ambiguous naming where possible.

For example:

```text id="m7c2zx"
database/
└── models.py

api/
└── schemas.py

llms/
└── clients.py
```

Clear naming prevents confusion.

---

# 13. The `services/` Directory

Services can coordinate business operations.

Example:

```text id="t4q9vn"
services/
├── chat.py
├── document.py
├── search.py
└── user.py
```

A service might coordinate:

```text id="e8p3mc"
API
 ↓
Service
 ├── Agent
 ├── RAG
 ├── Database
 └── External API
```

For example:

```python id="g5w1rs"
class ChatService:

    def __init__(
        self,
        agent,
        conversation_repository
    ):
        self.agent = agent
        self.repository = conversation_repository

    def execute(self, request):
        ...
```

Services should represent meaningful application operations rather than becoming a generic dumping ground.

---

# 14. The `prompts/` Directory

Large AI applications often contain many prompts.

Instead of:

```python id="n4q8yz"
prompt = """
very large prompt...
"""
```

inside business logic, consider organizing prompts separately.

Example:

```text id="w7m3kp"
prompts/
├── agent/
│   ├── planner.txt
│   ├── researcher.txt
│   └── reviewer.txt
├── rag/
│   └── answer.txt
└── system/
    └── assistant.txt
```

This makes prompts easier to:

```text id="6x2qvn"
Review
Version
Test
Update
Compare
```

The exact format can be `.txt`, `.md`, Python constants, or another approach depending on the project.

---

# 15. Prompt Versioning

Prompts are part of application behavior.

Changing:

```text id="j1q6sd"
system prompt v1
```

to:

```text id="p7z4mk"
system prompt v2
```

can change application behavior even if no Python code changed.

Therefore, important prompts should be treated like code.

For critical applications:

```text id="c9x2tw"
Prompt
 ↓
Evaluation dataset
 ↓
Regression test
```

This makes prompt changes safer.

---

# 16. The `config/` Directory

Configuration should be centralized.

Example:

```text id="s5v8ny"
config/
├── settings.py
└── logging.py
```

Configuration might include:

```text id="w4n1fz"
Database URL
LLM provider
Model name
Timeout
Vector database URL
Environment
Feature flags
```

Avoid scattering:

```python id="z9r3vk"
os.getenv("...")
```

throughout the application.

Instead, centralize configuration access.

---

# 17. Environment Variables

Secrets and environment-specific configuration should not be hardcoded.

Bad:

```python id="x3j7qp"
API_KEY = "my-secret-key"
```

Better:

```python id="v8m2rs"
API_KEY = os.getenv("API_KEY")
```

Or use a configuration library such as Pydantic Settings.

A repository should generally contain:

```text id="q5c9wx"
.env.example
```

rather than:

```text id="b6k2md"
.env
```

The actual `.env` file containing secrets should normally be excluded from Git.

---

# 18. Example `.env.example`

```env id="z8y4nt"
APP_ENV=development

DATABASE_URL=
LLM_API_KEY=
LLM_MODEL=

VECTOR_DB_URL=
VECTOR_DB_API_KEY=
```

This documents which environment variables are required without exposing their values.

---

# 19. The `tests/` Directory

Keep tests separate from application code.

Example:

```text id="x7q3mb"
tests/
├── unit/
├── integration/
├── e2e/
└── evals/
```

### Unit

Fast, isolated tests.

### Integration

Tests real component boundaries.

### E2E

Tests complete user flows.

### Evals

Measures AI behavior and quality.

This separation helps you understand what failed.

---

# 20. The `evals/` Directory

AI applications benefit from explicit evaluation datasets.

For example:

```text id="m3w7xp"
evals/
├── datasets/
│   ├── rag.jsonl
│   └── agents.jsonl
├── evaluators/
│   ├── retrieval.py
│   └── answer.py
└── runners/
    └── run.py
```

This separates:

```text id="n4x8sq"
Test correctness
```

from:

```text id="d7p2cm"
Measure AI quality
```

---

# 21. The `scripts/` Directory

Scripts are useful for operational tasks.

Examples:

```text id="y6v3qk"
scripts/
├── ingest_documents.py
├── seed_database.py
├── run_evals.py
├── migrate.py
└── benchmark_retrieval.py
```

A script should have a clear purpose.

Avoid putting core application logic only inside scripts.

Reusable logic should live inside the application package.

---

# 22. The `docs/` Directory

Documentation can include:

```text id="c4p9zm"
Architecture
API documentation
Deployment
Runbooks
Design decisions
Evaluation methodology
```

Example:

```text id="u7m2wx"
docs/
├── architecture.md
├── deployment.md
├── observability.md
└── decisions/
```

This becomes especially useful when the application is worked on by multiple engineers.

---

# 23. Architecture Decision Records

For important architectural decisions, maintain short records.

Example:

```text id="g3q8vy"
docs/decisions/
├── 001-vector-database.md
├── 002-hybrid-retrieval.md
└── 003-agent-orchestration.md
```

A decision record can explain:

```text id="k5w1pn"
Problem
Options considered
Decision
Trade-offs
Consequences
```

This helps future engineers understand why the system looks the way it does.

---

# 24. `pyproject.toml`

Modern Python projects commonly use:

```text id="r6x2qm"
pyproject.toml
```

for project metadata and tool configuration.

It can contain configuration for:

```text id="4n8zsv"
Package metadata
Dependencies
Build system
pytest
Ruff
Mypy
Other developer tools
```

A simplified example:

```toml id="v1q9cx"
[project]
name = "ai-assistant"
version = "0.1.0"
description = "Production-oriented AI assistant"

dependencies = [
    "fastapi",
    "pydantic",
]

[dependency-groups]
dev = [
    "pytest",
]
```

The exact dependency-management format depends on the package manager and Python tooling you choose.

---

# 25. Dependency Management

Common approaches include:

```text id="k8w4pv"
pip + requirements.txt
uv
Poetry
PDM
```

For a modern project, choose one clear dependency-management strategy.

Do not mix multiple package managers without a reason.

The important goals are:

```text id="n2m7qz"
Reproducibility
Locking/version control
Easy setup
Clear dependency boundaries
```

---

# 26. `__init__.py`

A package may contain:

```text id="b7r3xf"
agents/
├── __init__.py
├── researcher.py
└── writer.py
```

`__init__.py` can mark a directory as a regular Python package and can also expose selected package-level APIs.

Do not automatically put lots of application logic into it.

Keep it minimal unless there is a clear reason otherwise.

---

# 27. Dependency Injection

AI applications often have many dependencies:

```text id="z5q2km"
LLM client
Database
Vector store
Embedding model
Tool registry
Configuration
```

Instead of constructing everything deep inside business logic:

```python id="y8c4vs"
def answer(query):

    llm = OpenAIClient(...)
    db = QdrantClient(...)
```

inject dependencies:

```python id="p1w7nd"
class AnswerService:

    def __init__(self, llm, retriever):
        self.llm = llm
        self.retriever = retriever
```

This makes testing easier:

```text id="v9x3cq"
Production
→ Real LLM

Test
→ Fake LLM
```

Dependency injection is particularly valuable in AI applications because external services are expensive and non-deterministic.

---

# 28. Avoid Global Mutable State

Avoid patterns such as:

```python id="m4x8pk"
global_agent = Agent(...)
```

with many modules mutating it.

Global state can make:

```text id="k2v9zs"
Testing
Concurrency
Debugging
Configuration
```

more difficult.

Prefer explicit dependencies where practical.

---

# 29. Separate Interfaces from Implementations

Consider a retrieval system.

You might define an interface:

```python id="f6q1wx"
class Retriever:

    def search(self, query):
        raise NotImplementedError
```

and implementations:

```text id="g9m3vk"
VectorRetriever
KeywordRetriever
HybridRetriever
```

Then:

```text id="x4p7zs"
Application
    ↓
Retriever interface
    ↓
Implementation
```

This can make switching implementations and testing easier.

Again, use abstractions where they solve an actual problem rather than creating layers purely for architecture's sake.

---

# 30. Keep External Services at the Boundaries

A useful architectural principle is:

```text id="z7n2mc"
Core application logic
        ↓
Interfaces
        ↓
External systems
```

External systems might include:

```text id="p3x8qb"
LLM providers
Vector databases
PostgreSQL
Redis
Search APIs
MCP servers
Cloud storage
Email providers
```

Avoid scattering provider-specific code throughout the application.

---

# 31. Example Dependency Flow

A clean AI application might look like:

```text id="j5m8rv"
API
 ↓
Service
 ↓
Agent
 ├── Retriever
 ├── LLM
 └── Tools
      ↓
External systems
```

The direction of dependencies should remain understandable.

For example:

```text id="v8q2ks"
API → Service → Domain/AI logic → Infrastructure
```

rather than:

```text id="x6p4mb"
Database → API → Agent → Database → Utility → API
```

Circular dependencies make systems harder to maintain.

---

# 32. Avoid the `utils.py` Dumping Ground

A common problem:

```text id="y3n7qp"
utils.py
```

eventually contains:

```text id="f4m8kx"
format_date()
parse_pdf()
call_llm()
generate_embedding()
validate_user()
send_email()
calculate_score()
```

These functions have unrelated responsibilities.

Instead, put code where its responsibility belongs.

For example:

```text id="t2w9pc"
documents/parsing.py
llms/client.py
users/validation.py
email/service.py
evaluation/scoring.py
```

A small utility module is fine.

A giant `utils.py` usually signals weak boundaries.

---

# 33. Example: Small RAG Application

A practical structure:

```text id="p8w3mq"
rag-app/
│
├── src/
│   └── rag_app/
│       ├── api/
│       │   ├── routes.py
│       │   └── schemas.py
│       │
│       ├── config/
│       │   └── settings.py
│       │
│       ├── rag/
│       │   ├── ingestion.py
│       │   ├── chunking.py
│       │   ├── embeddings.py
│       │   ├── retrieval.py
│       │   └── generation.py
│       │
│       ├── llms/
│       │   └── client.py
│       │
│       ├── database/
│       │   └── connection.py
│       │
│       └── main.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── evals/
│
├── .env.example
├── .gitignore
├── pyproject.toml
├── Dockerfile
└── README.md
```

This is enough structure for many small-to-medium applications.

---

# 34. Example: Production Multi-Agent System

A larger application might look like:

```text id="s4k9xn"
agent-platform/
│
├── src/
│   └── agent_platform/
│
│       ├── api/
│       │   ├── routes/
│       │   │   ├── chat.py
│       │   │   ├── documents.py
│       │   │   └── tasks.py
│       │   ├── dependencies.py
│       │   └── schemas.py
│       │
│       ├── agents/
│       │   ├── supervisor.py
│       │   ├── researcher.py
│       │   ├── writer.py
│       │   ├── reviewer.py
│       │   ├── state.py
│       │   └── graph.py
│       │
│       ├── rag/
│       │   ├── ingestion/
│       │   ├── retrieval/
│       │   ├── reranking/
│       │   └── context.py
│       │
│       ├── llms/
│       │   ├── client.py
│       │   ├── providers/
│       │   └── embeddings.py
│       │
│       ├── tools/
│       │   ├── search.py
│       │   ├── database.py
│       │   ├── email.py
│       │   └── registry.py
│       │
│       ├── database/
│       │   ├── connection.py
│       │   ├── models.py
│       │   └── repositories/
│       │
│       ├── services/
│       │   ├── chat.py
│       │   ├── documents.py
│       │   └── tasks.py
│       │
│       ├── prompts/
│       │   ├── agents/
│       │   └── rag/
│       │
│       ├── observability/
│       │   ├── logging.py
│       │   ├── metrics.py
│       │   └── tracing.py
│       │
│       ├── config/
│       │   └── settings.py
│       │
│       └── main.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── evals/
│
├── scripts/
├── docs/
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

This is an example of how the architecture can evolve when the system becomes significantly larger.

Do not start every project with this many directories.

---

# 35. Monolith vs Microservices

Project structure is not the same thing as service architecture.

You can have a well-structured monolith:

```text id="s6p4vc"
One application
├── API
├── Agents
├── RAG
├── Database
└── Tools
```

Later, a component may become a separate service:

```text id="n8x2qa"
API Service
      ↓
Agent Service
      ↓
RAG Service
      ↓
Vector DB
```

Do not split services simply because the folder structure has grown.

Service boundaries should be driven by actual requirements such as:

```text id="w5k9mz"
Independent scaling
Deployment independence
Team ownership
Failure isolation
Resource requirements
Security boundaries
```

---

# 36. Python Service + Node.js Service

AI systems may combine technologies.

For example:

```text id="x3v7kp"
React / Next.js
       ↓
NestJS API
       ↓
gRPC
       ↓
Python AI Service
       ↓
LLM / RAG / Agents
```

The repositories might be:

```text id="b9m2ws"
frontend/
backend/
ai-service/
```

or maintained independently.

The important principle is to define a clear contract between services.

Possible communication mechanisms include:

```text id="q8c4nv"
REST
gRPC
Message queues
Events
```

---

# 37. gRPC Boundary

If a Python AI service communicates with a NestJS service:

```text id="m3x7qb"
NestJS
   ↓
gRPC
   ↓
Python
```

keep the service boundary explicit.

For example:

```text id="d4n8kp"
proto/
└── ai_service.proto
```

The protobuf contract defines:

```text id="z1w5mc"
Requests
Responses
Methods
Data types
```

Application-level validation can still be performed inside each service.

---

# 38. Background Workers

AI systems frequently need asynchronous processing.

For example:

```text id="c6m9rx"
API
 ↓
Queue
 ↓
Worker
 ↓
Document ingestion
 ↓
Embeddings
 ↓
Vector DB
```

Keep worker entry points separate from core application logic.

Example:

```text id="j2v7sn"
workers/
├── document_ingestion.py
├── embedding.py
└── evaluation.py
```

The worker should call reusable application services rather than duplicate business logic.

---

# 39. Configuration vs Business Logic

Avoid:

```python id="q4z8nx"
if os.getenv("ENV") == "production":
    # 100 lines of business logic
```

Configuration should influence behavior through explicit settings.

For example:

```python id="p7x2mc"
settings.environment
settings.llm_model
settings.max_retries
```

Then business logic can remain readable.

---

# 40. Logging and Observability Structure

A production project may contain:

```text id="m5q9vc"
observability/
├── logging.py
├── metrics.py
└── tracing.py
```

The rest of the application can use these capabilities without knowing how telemetry is exported.

For example:

```text id="v8c3pk"
Agent
 ↓
Observability layer
 ↓
Logs / Metrics / Traces
```

This keeps observability concerns separate from business logic.

---

# 41. Testing Structure

A practical test layout:

```text id="j7m2wx"
tests/
├── unit/
│   ├── test_chunking.py
│   ├── test_prompt_builder.py
│   └── test_tools.py
│
├── integration/
│   ├── test_vector_store.py
│   └── test_database.py
│
├── e2e/
│   └── test_chat.py
│
└── evals/
    ├── test_retrieval.py
    └── test_answers.py
```

This corresponds to the testing strategy from the previous chapter.

---

# 42. A Useful Rule for New Files

Before creating a new file, ask:

> What single responsibility does this file own?

For example:

```text id="g8x4vn"
retrieval.py
```

should primarily deal with retrieval.

If it starts containing:

```text id="m5k2qs"
Database migrations
Email sending
Authentication
LLM prompts
```

it is probably doing too much.

---

# 43. A Useful Rule for New Folders

Do not create folders just to make the tree look sophisticated.

Bad:

```text id="p4m8yx"
helpers/
common/
misc/
core/
base/
utils/
shared/
```

with unclear responsibilities.

Good:

```text id="x7q2kn"
agents/
rag/
tools/
database/
api/
```

where each folder communicates a real domain or responsibility.

---

# 44. Dependency Direction

A useful mental model:

```text id="n9v3mx"
                API
                 ↓
              Services
                 ↓
          AI / Domain Logic
           ↙           ↘
        RAG            Agents
         ↓               ↓
     Interfaces        Tools
         ↓               ↓
      Infrastructure / External Services
```

The exact architecture may differ, but dependency direction should be intentional.

If everything imports everything else, the project becomes difficult to evolve.

---

# 45. Circular Dependencies

A common problem:

```text id="t3q8vz"
A imports B
B imports A
```

For example:

```text id="m4w7cx"
agent.py → service.py
service.py → agent.py
```

This can cause import errors and tightly coupled architecture.

Possible solutions:

```text id="7k1pms"
Move shared types
Introduce a smaller interface
Invert the dependency
Refactor responsibilities
```

Do not solve every circular import with random local imports.

Fix the underlying dependency design.

---

# 46. Keep Domain Logic Independent

Where practical, core logic should not depend directly on HTTP.

Instead of:

```python id="y5q9bn"
def process_request(http_request):
    ...
```

prefer:

```python id="r3k7vx"
def process(query):
    ...
```

Then:

```text id="k8w2pc"
HTTP API
   ↓
process(query)
```

This makes the logic easier to:

```text id="v2m9qs"
Test
Reuse
Run from workers
Run from CLI
Expose through another API
```

---

# 47. CLI Entry Points

Some AI applications need command-line operations:

```text id="q6w4zn"
Ingest documents
Run evaluations
Rebuild index
Run benchmarks
Create admin user
```

These should be separate entry points rather than hidden inside the web server.

For example:

```text id="c8p2mx"
python -m app.cli ingest
python -m app.cli evaluate
```

The CLI should call the same application services used elsewhere.

---

# 48. Docker and Project Structure

A production Python application may include:

```text id="b7m3qk"
Dockerfile
.dockerignore
```

A simplified Docker flow:

```text id="x4v9ps"
Source
 ↓
Install dependencies
 ↓
Build image
 ↓
Run application
```

Do not copy secrets into the image.

Use runtime configuration for environment-specific values.

---

# 49. README Structure

Every serious project should explain how to use it.

A useful README might contain:

```text id="f6k2rw"
# Project Name

## What it does

## Architecture

## Features

## Tech Stack

## Project Structure

## Setup

## Environment Variables

## Running Locally

## Running Tests

## Running Evaluations

## API

## Deployment

## Troubleshooting
```

The README is part of the engineering quality of the project.

---

# 50. The Project Structure Should Evolve

There is no perfect project structure.

A useful progression is:

```text id="h5n8mq"
Prototype
   ↓
Small application
   ↓
Production application
   ↓
Large system
   ↓
Multiple services
```

For example:

### Prototype

```text id="r3v7zx"
app.py
```

### Small application

```text id="k8w2mc"
app/
├── api/
├── rag/
├── llms/
└── main.py
```

### Production application

```text id="q4x9pn"
src/
├── api/
├── agents/
├── rag/
├── services/
├── tools/
├── database/
├── observability/
└── config/
```

### Distributed system

```text id="z7m3ks"
api-service/
agent-service/
rag-service/
worker-service/
```

Start simple and introduce structure as complexity demands it.

---

# Common Mistakes

## Mistake 1: Overengineering too early

Do not create 50 directories for a 200-line prototype.

---

## Mistake 2: One giant `main.py`

As the application grows, move responsibilities into focused modules.

---

## Mistake 3: Giant `utils.py`

Put functionality into the domain where it belongs.

---

## Mistake 4: Mixing API and business logic

HTTP handling and AI workflow logic should generally have separate responsibilities.

---

## Mistake 5: Scattering provider-specific code

Avoid direct LLM/vector database calls everywhere.

Create meaningful boundaries where useful.

---

## Mistake 6: Hardcoding secrets

Use environment configuration or a proper secret-management mechanism.

---

## Mistake 7: Global mutable state

This can create testing and concurrency problems.

---

## Mistake 8: Creating abstractions for everything

An abstraction should solve a real problem.

---

## Mistake 9: Splitting into microservices too early

A well-structured monolith is often easier to develop and operate than unnecessary distributed services.

---

## Mistake 10: Mixing test and production resources

Tests should use isolated databases, vector stores, credentials, and external resources.

---

# Production Insight

A good project structure should make it easy to answer:

```text id="s5q9kw"
Where is the API?
Where is the business logic?
Where is the agent?
Where is RAG?
Where are tools?
Where are database operations?
Where are prompts?
Where is configuration?
Where are tests?
Where are evaluations?
Where is observability?
```

If an engineer needs to search the entire repository to answer these questions, the structure probably needs improvement.

For AI applications, the most useful boundaries are usually around:

```text id="m8x3qv"
API
Agents
RAG
LLM providers
Tools
Data
Infrastructure
Observability
Evaluation
```

These boundaries also make it easier to:

```text id="n4k7pc"
Test components
Replace providers
Scale workloads
Debug failures
Add features
Onboard engineers
```

---

# Practical Exercise

Create a production-oriented structure for a small AI assistant.

Start with:

```text id="j2v6mn"
ai-assistant/
```

Build:

```text id="x9q3rw"
ai-assistant/
│
├── src/
│   └── ai_assistant/
│       ├── api/
│       ├── agents/
│       ├── rag/
│       ├── llms/
│       ├── tools/
│       ├── services/
│       ├── config/
│       ├── observability/
│       └── main.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── evals/
│
├── scripts/
├── docs/
├── .env.example
├── .gitignore
├── pyproject.toml
├── Dockerfile
└── README.md
```

Then implement:

1. A FastAPI API.
2. A configuration module.
3. An LLM client interface.
4. A simple RAG service.
5. A fake LLM for unit tests.
6. One agent workflow.
7. One tool.
8. Structured logging.
9. Unit tests.
10. One small evaluation dataset.

The goal is not to build a huge application.

The goal is to practice **separation of responsibilities**.

---

# Interview Questions

## Beginner

1. How should you structure a Python project?
2. Why might you use a `src/` layout?
3. What belongs in `api/`?
4. What belongs in `services/`?
5. What belongs in `config/`?
6. Why should tests be separated from application code?
7. What is `pyproject.toml`?
8. What is the purpose of `.env.example`?
9. Why should secrets not be committed to Git?
10. What is the purpose of `__init__.py`?

## Intermediate

11. How would you structure a FastAPI application?
12. How would you structure a RAG application?
13. How would you structure a multi-agent application?
14. How would you separate API logic from business logic?
15. What is dependency injection?
16. Why is dependency injection useful for testing LLM applications?
17. How would you avoid circular dependencies?
18. Why is a large `utils.py` usually problematic?
19. How would you organize prompts?
20. How would you organize background workers?

## AI Engineering

21. Design a production project structure for a RAG application.
22. Where would you put an LLM provider integration?
23. Where would you put agent state?
24. Where would you put tool implementations?
25. How would you structure a multi-agent LangGraph application?
26. How would you isolate provider-specific code?
27. How would you structure AI evaluations?
28. Where would observability code live?
29. How would you structure a Python AI service communicating with a NestJS service through gRPC?
30. When would you split a monolith into multiple AI services?
31. How would you structure a document-ingestion worker?
32. How would you prevent application code from depending directly on infrastructure?
33. How would you design dependency injection for an LLM, vector store, and database?
34. How would you structure prompts so they can be versioned and evaluated?
35. Design the folder structure for a production multi-agent RAG platform.

---

# Key Takeaways

* Project structure should make responsibilities and dependencies clear.
* Start simple and introduce structure as complexity grows.
* `src/` is a useful layout for larger Python projects.
* Keep API, business logic, AI workflows, and infrastructure separated.
* Organize RAG functionality around meaningful pipeline responsibilities.
* Isolate LLM provider integrations where useful.
* Give agent workflows and tools clear boundaries.
* Centralize configuration.
* Never commit secrets.
* Keep tests separate from application code.
* Separate unit, integration, E2E, and AI evaluation concerns.
* Treat important prompts as versioned application behavior.
* Use dependency injection to make external services replaceable and testable.
* Avoid giant `utils.py` and `main.py` files.
* Avoid unnecessary abstractions and premature microservices.
* Keep dependency direction intentional.
* Use clear service boundaries when integrating Python with systems such as NestJS through gRPC.
* Project structure should evolve with the application's actual complexity.

---

# Summary

A production AI application is still a software system.

The AI-specific components:

```text id="k8m3qp"
LLMs
RAG
Agents
Tools
Embeddings
Vector databases
```

should live inside a sound software architecture.

A useful mental model is:

```text id="r4x7ns"
                 API
                  ↓
               Services
                  ↓
          ┌───────┴────────┐
          ↓                ↓
         RAG             Agents
          ↓                ↓
      Retrieval          Tools
          │                │
          └───────┬────────┘
                  ↓
            Infrastructure
                  ↓
       DB / LLM / Vector DB / APIs
```

Around this core:

```text id="v6q2mw"
Config
Observability
Tests
Evaluations
Scripts
Documentation
```

The goal of project structure is not to create the most sophisticated folder tree.

The goal is to make the system:

**understandable, testable, maintainable, and capable of evolving as the AI application grows.**
