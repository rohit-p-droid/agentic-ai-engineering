# Python for AI Engineering — Interview Questions

This guide covers Python concepts that are especially relevant to **AI Engineers, GenAI Engineers, Agentic AI Engineers, and Forward Deployed Engineers**.

The focus is not memorizing Python syntax.

The goal is to understand how Python behaves when building:

* LLM applications
* RAG systems
* AI agents
* Multi-agent systems
* FastAPI services
* Background workers
* Data pipelines
* AI evaluation systems
* Production AI applications

---

# 1. Python Fundamentals

## 1.1 What are Python's main characteristics?

Python is:

* Dynamically typed
* High-level
* Interpreted/bytecode-executed
* Object-oriented
* Multi-paradigm
* Garbage-collected
* Rich in libraries and tooling

For AI engineering, Python is particularly useful because it provides a large ecosystem around:

```text
LLMs
Machine Learning
Data Processing
APIs
RAG
Agents
Vector Databases
Evaluation
Infrastructure
```

---

## 1.2 What is the difference between a list, tuple, set, and dictionary?

| Type    | Ordered                | Mutable | Duplicates  | Typical AI use      |
| ------- | ---------------------- | ------- | ----------- | ------------------- |
| `list`  | Yes                    | Yes     | Yes         | Documents/chunks    |
| `tuple` | Yes                    | No      | Yes         | Fixed configuration |
| `set`   | No guaranteed indexing | Yes     | No          | Deduplication       |
| `dict`  | Yes                    | Yes     | Keys unique | Structured records  |

Example:

```python
documents = ["doc1", "doc2", "doc3"]

config = ("gpt-model", 0.2)

unique_sources = {"docs", "web"}

metadata = {
    "document_id": "123",
    "source": "internal_docs",
}
```

---

## 1.3 What is mutable vs immutable?

Mutable objects can be changed after creation.

Examples:

```python
list
dict
set
```

Immutable objects include:

```python
int
float
str
tuple
frozenset
```

This matters when passing objects between functions, storing state, or sharing data across concurrent operations.

---

## 1.4 What is the difference between `==` and `is`?

`==` compares values.

```python
a == b
```

`is` checks object identity.

```python
a is b
```

Use `is` for identity checks such as:

```python
if value is None:
    ...
```

Do not use `is` when you mean value equality.

---

## 1.5 What are `*args` and `**kwargs`?

They allow functions to accept variable numbers of arguments.

```python
def tool(*args, **kwargs):
    ...
```

`*args` collects positional arguments.

`**kwargs` collects keyword arguments.

They are particularly useful when building:

* Decorators
* Generic wrappers
* Tool systems
* Framework integrations

---

# 2. Functions

## 2.1 Are functions first-class objects in Python?

Yes.

Functions can be:

```text
Stored in variables
Passed to functions
Returned from functions
Stored in data structures
```

Example:

```python
def retrieve(query):
    return ["document"]

processor = retrieve

results = processor("AI agents")
```

This concept is important for decorators, callbacks, middleware, and functional pipelines.

---

## 2.2 What is a higher-order function?

A function that accepts another function or returns a function.

Example:

```python
def with_logging(func):

    def wrapper(*args, **kwargs):
        print("Running")
        return func(*args, **kwargs)

    return wrapper
```

This is the foundation of decorators.

---

## 2.3 What is a lambda?

A lambda is a small anonymous function.

```python
sort_key = lambda result: result.score
```

Useful for short transformations, sorting, filtering, and mapping.

Avoid using lambdas when the logic becomes complex.

---

# 3. OOP

## 3.1 What are the four common OOP concepts?

```text
Encapsulation
Inheritance
Polymorphism
Abstraction
```

AI applications commonly use OOP for:

```text
LLM clients
Retrievers
Agents
Tools
Memory
Database repositories
Services
```

---

## 3.2 Why use composition instead of deep inheritance?

Consider:

```python
class Agent:
    def __init__(self, llm, memory, tools):
        self.llm = llm
        self.memory = memory
        self.tools = tools
```

The agent is composed of independent components.

This is often easier to modify than:

```text
BaseAgent
    ↓
RAGAgent
    ↓
ToolAgent
    ↓
MemoryToolAgent
    ↓
ProductionMemoryToolAgent
```

Deep inheritance hierarchies can become difficult to understand and maintain.

---

## 3.3 What is polymorphism?

Different objects can expose the same interface while implementing behavior differently.

Example:

```python
class Retriever:
    def search(self, query):
        raise NotImplementedError
```

Implementations:

```python
class VectorRetriever(Retriever):
    def search(self, query):
        ...


class KeywordRetriever(Retriever):
    def search(self, query):
        ...
```

Application code can work with:

```python
retriever.search(query)
```

without needing to know the implementation.

---

# 4. Dataclasses and Pydantic

## 4.1 When would you use a dataclass?

Dataclasses are useful for structured internal Python data.

Examples:

```text
Document
Chunk
RetrievalResult
AgentState
EvaluationResult
ToolResult
```

Example:

```python
from dataclasses import dataclass


@dataclass
class RetrievalResult:
    document_id: str
    score: float
    content: str
```

---

## 4.2 When would you use Pydantic?

Pydantic is particularly useful at boundaries where data needs validation.

Examples:

```text
API requests
API responses
Configuration
Tool arguments
Structured LLM outputs
Service boundaries
```

Example:

```python
from pydantic import BaseModel


class SearchRequest(BaseModel):
    query: str
    limit: int = 5
```

---

## 4.3 Dataclass vs Pydantic

A useful rule:

```text
Internal trusted data
        ↓
Dataclass

External/untrusted data
        ↓
Pydantic
```

This is not an absolute rule, but it is a useful architectural guideline.

---

# 5. Exceptions

## 5.1 Why shouldn't you use bare `except`?

Avoid:

```python
try:
    result = call_llm()
except:
    return None
```

This can hide:

```text
Programming errors
Network failures
Authentication failures
Rate limits
Unexpected failures
```

Prefer specific exceptions and meaningful recovery behavior.

---

## 5.2 When should you retry an LLM API call?

Retrying can make sense for transient failures such as:

```text
Temporary network failure
Rate limiting
Transient service errors
```

You should not blindly retry:

```text
Invalid API key
Invalid request
Malformed input
Permission errors
Programming bugs
```

Production retries should generally consider:

```text
Exponential backoff
Jitter
Maximum attempts
Timeouts
Idempotency
Provider rate limits
```

---

## 5.3 Why create custom exceptions?

Custom exceptions make failures easier to classify.

Example:

```python
class RetrievalError(Exception):
    pass


class LLMServiceError(Exception):
    pass


class ToolExecutionError(Exception):
    pass
```

This allows higher layers to handle different failure categories appropriately.

---

# 6. Context Managers

## 6.1 What is a context manager?

A context manager manages setup and cleanup around a block of code.

Example:

```python
with open("document.txt") as file:
    content = file.read()
```

The resource is cleaned up automatically.

---

## 6.2 Where are context managers useful in AI applications?

Examples:

```text
Files
Database connections
Transactions
Locks
Temporary resources
Tracing spans
Model sessions
Network resources
```

They are especially useful when resources must reliably be released even if an exception occurs.

---

# 7. Iterators and Generators

## 7.1 What is a generator?

A generator produces values lazily using `yield`.

```python
def documents():
    for path in paths:
        yield load_document(path)
```

Instead of loading everything into memory:

```text
100,000 documents
        ↓
Generate one at a time
```

This is useful for large AI data pipelines.

---

## 7.2 Where would you use generators in RAG?

A pipeline can be:

```text
Load
 ↓
Clean
 ↓
Chunk
 ↓
Embed
 ↓
Store
```

Generators can allow these stages to process data incrementally.

---

## 7.3 Generator vs list

List:

```python
chunks = [create_chunk(doc) for doc in documents]
```

Everything is created immediately.

Generator:

```python
chunks = (create_chunk(doc) for doc in documents)
```

Values are produced lazily.

This can significantly reduce memory usage for large datasets.

---

# 8. Decorators

## 8.1 What is a decorator?

A decorator modifies or wraps a function or class.

Example:

```python
@log_execution
def generate_answer(query):
    ...
```

Conceptually:

```python
generate_answer = log_execution(generate_answer)
```

---

## 8.2 Where are decorators useful in AI applications?

Common examples:

```text
Logging
Metrics
Tracing
Retries
Caching
Authentication
Authorization
Validation
Tool registration
```

---

## 8.3 What is `functools.wraps` and why use it?

When writing decorators:

```python
from functools import wraps
```

`wraps` preserves metadata such as:

```text
Function name
Documentation
Module
```

Without it, debugging and introspection become harder.

---

# 9. Virtual Environments and Dependencies

## 9.1 Why use virtual environments?

AI projects often have conflicting dependencies.

For example:

```text
Project A
Python 3.11
Pydantic 1.x

Project B
Python 3.12
Pydantic 2.x
```

A virtual environment isolates dependencies.

---

## 9.2 Why use `python -m pip`?

Instead of:

```bash
pip install fastapi
```

you can use:

```bash
python -m pip install fastapi
```

This makes it clearer which Python interpreter's package manager is being used.

---

## 9.3 What is the difference between `.env` and `.venv`?

```text
.env
→ Environment configuration/secrets

.venv
→ Python virtual environment
```

They serve completely different purposes.

---

# 10. Asyncio

## 10.1 What problem does asyncio solve?

Asyncio helps efficiently handle many I/O-bound operations.

Examples:

```text
LLM API calls
HTTP requests
Database queries
Vector database calls
External tools
```

The key idea:

```text
Don't block while waiting for I/O.
```

---

## 10.2 What does `await` do?

`await` allows an async function to pause while waiting for an awaitable operation.

For example:

```python
result = await llm.generate(prompt)
```

While the operation is waiting, the event loop can execute other work.

---

## 10.3 Why is async useful for agents?

An agent may need to call:

```text
Search API
Database
Vector DB
LLM
Another service
```

Some operations can happen concurrently.

Conceptually:

```text
             ┌── Search
Agent ────────┼── Database
             └── Retrieval
```

Then results can be combined.

---

## 10.4 What happens if you run blocking code inside async code?

Example:

```python
async def handler():
    requests.get(url)
```

The synchronous network call can block the event loop.

For blocking operations that cannot be replaced with async APIs, techniques such as:

```python
await asyncio.to_thread(blocking_function)
```

can move the work to a thread.

---

# 11. Threading

## 11.1 When should you use threads?

Threads are useful primarily for I/O-bound work, especially when the libraries involved are synchronous.

Examples:

```text
HTTP APIs
File operations
Database operations
SDK calls
```

---

## 11.2 Does Python threading provide CPU parallelism?

In standard CPython, the Global Interpreter Lock means multiple threads generally do not execute Python bytecode in parallel for CPU-bound work.

For CPU-heavy Python workloads, multiprocessing or specialized native/GPU libraries may be more appropriate.

---

## 11.3 When would you use `ThreadPoolExecutor`?

Example:

```python
from concurrent.futures import ThreadPoolExecutor


with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch_document, documents))
```

Useful when you have multiple independent blocking operations.

---

# 12. Multiprocessing

## 12.1 When would you use multiprocessing?

Primarily for CPU-bound workloads.

AI examples:

```text
Document preprocessing
Image processing
Audio processing
CPU-heavy evaluation
Large transformations
Parsing
```

---

## 12.2 Threading vs multiprocessing

| Workload         | Typical choice        |
| ---------------- | --------------------- |
| Async HTTP       | asyncio               |
| Sync HTTP        | Threads               |
| File I/O         | asyncio/threads       |
| CPU-heavy Python | Multiprocessing       |
| GPU workloads    | GPU/framework runtime |
| Distributed jobs | Workers/queues        |

The correct choice depends on the actual workload and libraries involved.

---

# 13. Concurrency Scenario

### Interview Question

You need to call three independent services:

```text
LLM API: 800 ms
Search API: 600 ms
Database: 500 ms
```

Should you call them sequentially?

If the operations are independent and the APIs support concurrency, parallel execution can reduce total waiting time toward the slowest operation rather than the sum of all three.

But production systems must also account for:

```text
Rate limits
Connection limits
Timeouts
Retries
Failure handling
Resource consumption
```

---

# 14. File Handling

## 14.1 Why is `with open()` preferred?

```python
with open("document.txt") as file:
    content = file.read()
```

The context manager ensures the file is closed correctly.

---

## 14.2 Why use `pathlib`?

Instead of manually constructing paths:

```python
path = "documents/" + filename
```

use:

```python
from pathlib import Path

path = Path("documents") / filename
```

It provides a clearer cross-platform path API.

---

# 15. Project Architecture

## 15.1 How would you structure a production AI application?

A reasonable starting point:

```text
src/
└── app/
    ├── api/
    ├── agents/
    ├── rag/
    ├── llms/
    ├── tools/
    ├── services/
    ├── database/
    ├── config/
    ├── observability/
    └── main.py

tests/
├── unit/
├── integration/
├── e2e/
└── evals/
```

The exact structure should evolve with the application.

---

## 15.2 Why separate API and business logic?

Because the same business logic may eventually be called by:

```text
HTTP API
CLI
Background worker
Scheduled job
Another service
```

Keeping business logic independent from HTTP makes reuse and testing easier.

---

## 15.3 Why avoid a giant `utils.py`?

Because unrelated responsibilities eventually accumulate there.

Instead of:

```text
utils.py
```

prefer domain-specific modules such as:

```text
documents/parsing.py
llms/client.py
evaluation/scoring.py
auth/validation.py
```

---

# 16. Testing

## 16.1 What is the difference between unit and integration testing?

### Unit test

Tests a small component in isolation.

```text
Chunking function
Prompt builder
Parser
Tool validation
```

### Integration test

Tests multiple components working together.

```text
Application
 ↓
Retriever
 ↓
Vector database
```

---

## 16.2 What should you mock in an LLM application?

Potential candidates:

```text
LLM APIs
External APIs
Email services
Payment services
Vector databases
Databases
```

But don't mock everything.

The purpose of integration tests is to verify that real components actually work together.

---

## 16.3 Why are LLM applications harder to test?

Traditional software often has deterministic outputs.

LLM systems may have:

```text
Non-deterministic responses
Model updates
Prompt sensitivity
Retrieval variability
External service failures
Token limits
```

Therefore AI systems need multiple forms of testing:

```text
Unit tests
Integration tests
E2E tests
AI evaluations
Regression tests
```

---

# 17. AI Evaluation

## 17.1 What is the difference between a test and an evaluation?

A traditional test might ask:

```text
Did the function return the expected value?
```

An AI evaluation might ask:

```text
Did the generated answer correctly answer the question?
```

Possible evaluation dimensions:

```text
Correctness
Relevance
Faithfulness
Retrieval quality
Tool selection
Safety
Latency
Cost
```

---

## 17.2 How would you evaluate a RAG system?

Break the system into stages.

```text
Question
   ↓
Retrieval
   ↓
Context
   ↓
Generation
```

Evaluate separately:

### Retrieval

```text
Did we retrieve the right documents?
```

### Context

```text
Did the final context contain useful evidence?
```

### Generation

```text
Did the answer correctly use the retrieved evidence?
```

This makes debugging much easier.

---

# 18. LLM Engineering Questions

## 18.1 What happens when an LLM API fails?

A production system should consider:

```text
Timeout
Retry
Backoff
Rate limits
Fallback
Logging
Tracing
User-facing error
```

The correct behavior depends on the failure type.

---

## 18.2 How would you reduce LLM latency?

Potential techniques include:

```text
Smaller model where appropriate
Streaming
Parallel independent calls
Prompt reduction
Caching
Reducing unnecessary tool calls
Reducing retrieved context
Connection reuse
```

Do not optimize blindly.

Measure first.

---

## 18.3 How would you reduce LLM cost?

Potential techniques:

```text
Smaller models
Prompt optimization
Caching
Batching
Reducing unnecessary calls
Limiting context
Better routing
Avoiding redundant agent loops
```

Cost and quality should be evaluated together.

---

# 19. RAG Interview Questions

## 19.1 What is RAG?

Retrieval-Augmented Generation combines retrieval with generation.

Conceptually:

```text
User Query
    ↓
Retriever
    ↓
Relevant Context
    ↓
LLM
    ↓
Answer
```

---

## 19.2 Why not simply put the entire document into the prompt?

Because large documents can cause:

```text
Context limitations
Higher cost
Higher latency
More irrelevant information
Potentially worse retrieval precision
```

Chunking and retrieval allow the system to provide more targeted context.

---

## 19.3 What factors affect chunking?

Consider:

```text
Document structure
Semantic boundaries
Chunk size
Overlap
Metadata
Retrieval strategy
Embedding model
Query type
```

There is no universal chunk size that works for every dataset.

---

## 19.4 What is hybrid retrieval?

Hybrid retrieval combines multiple retrieval signals.

For example:

```text
Vector similarity
+
Keyword matching
```

This can help when exact terms matter while semantic similarity is also useful.

---

## 19.5 What is reranking?

Initial retrieval may return:

```text
Top 50 candidates
```

A reranker can reorder those candidates:

```text
50 candidates
      ↓
Reranker
      ↓
Top 5
```

This can improve the relevance of the final context.

---

# 20. Agent Engineering

## 20.1 What is an AI agent?

A useful engineering definition is:

> An AI system that can use a model to decide and execute actions toward a goal, typically through tools and state.

A simplified loop:

```text
Goal
 ↓
Reason/Decide
 ↓
Select Tool
 ↓
Execute
 ↓
Observe Result
 ↓
Continue or Finish
```

---

## 20.2 Agent vs workflow

A workflow generally has predefined control flow:

```text
Step A
 ↓
Step B
 ↓
Step C
```

An agent can dynamically decide:

```text
Which tool?
Which step?
Whether another action is necessary?
```

Many production systems combine both:

```text
Deterministic workflow
+
Agentic decision points
```

---

## 20.3 Why should agents have limited tools?

Giving an agent dozens of tools can increase:

```text
Decision complexity
Latency
Token usage
Failure opportunities
Unexpected behavior
```

Use the smallest useful tool set.

---

## 20.4 How would you prevent an agent from running forever?

Use explicit limits:

```text
Maximum iterations
Maximum tool calls
Timeout
Token budget
Cost budget
State validation
Termination conditions
```

Example:

```python
MAX_ITERATIONS = 10
```

A production system should also expose observability around these limits.

---

# 21. Multi-Agent Systems

## 21.1 When would you use multiple agents?

Potential reasons include:

```text
Different responsibilities
Different tools
Different context
Parallel work
Specialized workflows
Organizational boundaries
```

Example:

```text
Supervisor
 ├── Researcher
 ├── Analyst
 └── Writer
```

But multiple agents also introduce:

```text
More latency
More tokens
More state management
More failure modes
More debugging complexity
```

Use them when the separation provides a real benefit.

---

## 21.2 How should agents communicate?

Possible approaches:

```text
Shared state
Structured messages
Events
Tool results
Service APIs
```

Prefer structured contracts over passing arbitrary text whenever possible.

---

# 22. Tool Calling

## 22.1 What makes a good AI tool?

A good tool should have:

```text
Clear name
Clear description
Structured inputs
Structured outputs
Validation
Authorization
Timeouts
Error handling
Observability
```

For example:

```python
def search_customer(
    customer_id: str
) -> Customer:
    ...
```

---

## 22.2 Should every function become an agent tool?

No.

A function should become a tool when the agent genuinely needs to decide when and how to invoke it.

Internal implementation functions do not necessarily need to be exposed to the model.

---

# 23. MCP Interview Questions

## 23.1 What problem does MCP address?

Model Context Protocol provides a standardized way for AI applications to interact with external tools and context providers.

Conceptually:

```text
AI Application
      ↓
MCP
      ↓
Tools / Resources / External Systems
```

The key engineering idea is standardizing the interface between an AI application and external capabilities.

---

## 23.2 Why can MCP be useful for agent systems?

It can reduce the need for custom integrations between every AI client and every external tool provider.

Instead of:

```text
Client A → Custom Integration → Tool
Client B → Custom Integration → Tool
Client C → Custom Integration → Tool
```

a standardized protocol can provide a common interface.

---

# 24. FastAPI Questions

## 24.1 Why is FastAPI commonly used for AI services?

It provides:

```text
Async support
Request validation
Type hints
Automatic API documentation
Dependency injection
High-performance ASGI architecture
```

It works well for services that expose:

```text
LLM endpoints
RAG APIs
Agent APIs
Embedding services
Inference orchestration
```

---

## 24.2 What is the difference between `def` and `async def` routes?

```python
@app.get("/")
def endpoint():
    ...
```

uses synchronous execution.

```python
@app.get("/")
async def endpoint():
    ...
```

allows asynchronous operations using `await`.

Using `async def` does not automatically make blocking code asynchronous.

---

# 25. Production Scenarios

## Scenario 1: LLM latency increased

Your API normally responds in 2 seconds but suddenly takes 8 seconds.

What would you investigate?

```text
LLM latency
Retrieval latency
Database latency
Tool calls
Network latency
Queue delays
Retries
Prompt size
Token generation
Concurrency
```

Use tracing to identify where time is actually being spent.

---

## Scenario 2: RAG answers became worse

Investigate:

```text
Document ingestion
Chunking
Embedding generation
Index freshness
Query transformation
Retrieval
Reranking
Context construction
Prompt
LLM behavior
```

Do not immediately change the model.

First determine which stage degraded.

---

## Scenario 3: Agent uses the wrong tool

Investigate:

```text
Tool descriptions
Tool schemas
Prompt
Available tools
Conversation context
Routing logic
Model behavior
Evaluation dataset
```

Then add targeted tests around tool selection.

---

## Scenario 4: API server becomes slow

Investigate whether the server is:

```text
Blocking the event loop
Waiting on LLM calls
Waiting on database calls
Running CPU-heavy work
Creating too many threads
Running too many concurrent tasks
```

Measure before changing the concurrency model.

---

## Scenario 5: Document ingestion consumes too much memory

Potential causes:

```text
Loading all files into memory
Building huge lists
Duplicating document content
Creating too many chunks at once
Unbounded concurrency
```

Possible solutions:

```text
Generators
Streaming
Batching
Bounded concurrency
Incremental processing
```

---

# 26. System Design Questions

Be prepared to design:

### 1. RAG System

```text
Documents
 ↓
Ingestion
 ↓
Chunking
 ↓
Embeddings
 ↓
Vector DB
 ↓
Retrieval
 ↓
Reranking
 ↓
LLM
```

Discuss:

```text
Scalability
Latency
Cost
Evaluation
Caching
Security
Observability
Failure recovery
```

---

### 2. Multi-Agent System

Design:

```text
User
 ↓
Supervisor
 ├── Research Agent
 ├── Retrieval Agent
 ├── Action Agent
 └── Reviewer
```

Discuss:

```text
State
Tool permissions
Termination
Retries
Timeouts
Observability
Human approval
```

---

### 3. Document Processing Platform

Design:

```text
Upload
 ↓
Object Storage
 ↓
Queue
 ↓
Worker
 ↓
Parsing
 ↓
Chunking
 ↓
Embedding
 ↓
Vector DB
```

Discuss:

```text
Large files
Retries
Idempotency
Parallel processing
Dead-letter queues
Monitoring
```

---

# 27. FDE-Style Python Questions

For Forward Deployed Engineer interviews, expect practical questions such as:

### "A customer's API is slow. How would you debug it?"

A strong engineering approach:

```text
Reproduce
 ↓
Measure
 ↓
Trace
 ↓
Identify bottleneck
 ↓
Form hypothesis
 ↓
Change one thing
 ↓
Measure again
```

---

### "The customer's documents contain inconsistent formats. What would you do?"

Consider:

```text
Input validation
Document classification
Format-specific parsers
Fallback parsing
Normalization
Metadata extraction
Error handling
Evaluation
```

---

### "The customer's RAG system retrieves irrelevant results."

Investigate:

```text
Chunking
Metadata
Embedding model
Query transformation
Keyword retrieval
Hybrid search
Reranking
Filters
Evaluation dataset
```

---

### "The customer's agent occasionally performs an unsafe action."

Consider:

```text
Tool authorization
Input validation
Output validation
Human approval
Permission boundaries
Tool scopes
Audit logs
Maximum action limits
```

---

# 28. Coding Exercises

You should be able to implement these without relying heavily on frameworks.

## Exercise 1 — Retry Decorator

Implement:

```python
@retry(max_attempts=3)
def call_service():
    ...
```

Requirements:

* Retry transient failures.
* Stop after the maximum attempts.
* Preserve the original function metadata.
* Add a delay between retries.

---

## Exercise 2 — Concurrent API Calls

Given:

```python
urls = [...]
```

fetch them concurrently using:

```text
asyncio
```

or:

```text
ThreadPoolExecutor
```

depending on the API client.

---

## Exercise 3 — Streaming Document Processor

Build:

```text
File
 ↓
Generator
 ↓
Clean
 ↓
Chunk
 ↓
Output
```

without loading every document into memory.

---

## Exercise 4 — RAG Retriever

Implement:

```python
retrieve(
    query,
    documents,
    top_k=5
)
```

and return:

```python
[
    {
        "document_id": "...",
        "score": 0.91,
        "content": "..."
    }
]
```

---

## Exercise 5 — Tool Registry

Implement:

```python
registry.register(tool)
registry.get("search")
registry.list_tools()
```

Then use the registry to execute a tool by name.

---

## Exercise 6 — Agent Loop

Implement a simplified:

```text
Observe
 ↓
Decide
 ↓
Tool
 ↓
Observe
 ↓
Finish
```

with:

```text
Maximum iterations
Timeout
Tool errors
Termination condition
```

---

# 29. Questions You Should Be Able to Answer Without Memorizing

Before moving beyond Python fundamentals, you should be able to explain:

* What happens when a Python function is called?
* What is an object?
* What is the difference between mutable and immutable objects?
* How does exception handling work?
* What does a context manager do?
* What is a generator?
* What is a decorator?
* What does asyncio solve?
* When would you use threads?
* When would you use multiprocessing?
* How do Python packages work?
* How do virtual environments work?
* How should a production Python project be structured?
* How would you test an AI application?
* How would you handle an unreliable external API?


---

# 30. AI Engineering Questions You Should Practice

You should also be able to reason through:
* How would you build a RAG system?
* How would you improve retrieval quality?
* How would you reduce RAG latency?
* How would you reduce LLM cost?
* How would you evaluate an LLM application?
* How would you prevent an agent from looping?
* How would you design tool permissions?
* How would you debug an unreliable agent?
* How would you structure a multi-agent system?
* How would you connect Python AI services to another backend?
* How would you design an AI system for production?
* How would you handle failures in an LLM API?
* How would you observe an agent's execution?
* How would you safely deploy prompt changes?
* How would you handle millions of documents?
* How would you handle concurrent users?

---

# 31. Final Python Interview Checklist

Before considering the Python fundamentals section complete, make sure you understand:

### Python

* [ ] Variables and data types
* [ ] Lists, tuples, sets, dictionaries
* [ ] Mutability
* [ ] Functions
* [ ] Scope
* [ ] `*args` and `**kwargs`
* [ ] Lambda functions
* [ ] Comprehensions
* [ ] OOP
* [ ] Composition
* [ ] Inheritance
* [ ] Polymorphism
* [ ] Abstract classes
* [ ] Dataclasses
* [ ] Pydantic
* [ ] Exceptions
* [ ] Context managers
* [ ] Iterators
* [ ] Generators
* [ ] Decorators
* [ ] Modules
* [ ] Packages
* [ ] Virtual environments
* [ ] Dependency management

### Concurrency

* [ ] Asyncio
* [ ] Coroutines
* [ ] Tasks
* [ ] `asyncio.gather`
* [ ] Timeouts
* [ ] Cancellation
* [ ] Threading
* [ ] Thread pools
* [ ] Locks
* [ ] Multiprocessing
* [ ] Process pools
* [ ] CPU-bound vs I/O-bound work

### Engineering

* [ ] Project structure
* [ ] Dependency injection
* [ ] Configuration
* [ ] Logging
* [ ] Testing
* [ ] Integration testing
* [ ] E2E testing
* [ ] AI evaluations
* [ ] Error handling
* [ ] Observability
* [ ] API design

### AI Engineering

* [ ] LLM integration
* [ ] RAG
* [ ] Embeddings
* [ ] Vector databases
* [ ] Hybrid retrieval
* [ ] Reranking
* [ ] Agents
* [ ] Tool calling
* [ ] Multi-agent systems
* [ ] MCP
* [ ] Agent evaluation
* [ ] Agent observability
* [ ] Production AI architecture

---

# Key Takeaways

Python interviews for AI Engineering are not only about syntax.

A strong AI Engineer should be able to connect Python fundamentals to production problems:

```text
Python
  ↓
Software Engineering
  ↓
Concurrency
  ↓
APIs & Services
  ↓
RAG
  ↓
Agents
  ↓
Production AI Systems
```

The most important skill is not memorizing answers.

It is being able to explain:

> **Why would you choose this approach, what trade-offs does it have, and how would you make it reliable in production?**

That mindset will become increasingly important as you move from Python fundamentals into **LLMs, RAG, agents, system design, and production AI engineering**.
