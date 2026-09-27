# Dataclasses

Dataclasses provide a convenient way to create Python classes that primarily store data.

Before dataclasses, a simple data-holding class often required a lot of boilerplate:

```python
class Document:
    def __init__(self, id, content, source, score):
        self.id = id
        self.content = content
        self.source = source
        self.score = score
```

With a dataclass:

```python
from dataclasses import dataclass


@dataclass
class Document:
    id: str
    content: str
    source: str
    score: float
```

Python automatically generates useful methods such as `__init__` and `__repr__`.

Dataclasses are particularly useful in AI engineering for representing internal application data such as:

* retrieved documents
* chunks
* search results
* agent state
* tool results
* configuration
* evaluation results
* pipeline metadata

---

# Why does an AI Engineer need this?

AI systems constantly move structured data between components.

For example:

```text
Document Loader
      ↓
Document
      ↓
Chunker
      ↓
DocumentChunk
      ↓
Embedding
      ↓
Vector Store
      ↓
RetrievedChunk
      ↓
LLM
```

Instead of passing loosely structured dictionaries everywhere:

```python
{
    "id": "chunk-123",
    "content": "...",
    "source": "docs.md",
    "score": 0.91,
}
```

you can represent the data explicitly:

```python
@dataclass
class RetrievedChunk:
    id: str
    content: str
    source: str
    score: float
```

Now the application's internal data model is clear.

---

# 1. Creating a Dataclass

Import `dataclass`:

```python
from dataclasses import dataclass
```

Define the class:

```python
@dataclass
class Document:
    id: str
    content: str
    source: str
```

Create an object:

```python
document = Document(
    id="doc-123",
    content="Python is useful for AI engineering.",
    source="python.md",
)
```

Access fields normally:

```python
print(document.id)
print(document.content)
print(document.source)
```

---

# 2. What Does `@dataclass` Do?

The decorator:

```python
@dataclass
```

automatically generates several useful methods.

For example:

```python
@dataclass
class Document:
    id: str
    content: str
```

is conceptually similar to writing:

```python
class Document:
    def __init__(self, id: str, content: str):
        self.id = id
        self.content = content
```

Dataclasses can also automatically generate methods such as:

```text
__init__
__repr__
__eq__
```

depending on the configuration.

This significantly reduces boilerplate.

---

# 3. Default Values

Fields can have default values.

```python
@dataclass
class SearchConfig:
    top_k: int = 5
    threshold: float = 0.7
```

Now:

```python
config = SearchConfig()
```

produces:

```text
top_k = 5
threshold = 0.7
```

You can override them:

```python
config = SearchConfig(
    top_k=10,
    threshold=0.8,
)
```

---

# 4. Required and Optional Fields

Fields without defaults are required:

```python
@dataclass
class AgentTask:
    task_id: str
    description: str
    priority: int = 1
```

This works:

```python
task = AgentTask(
    task_id="task-1",
    description="Search documentation",
)
```

But:

```python
AgentTask()
```

would fail because `task_id` and `description` are required.

---

# 5. Field Order

Required fields should generally appear before fields with defaults.

Correct:

```python
@dataclass
class Document:
    id: str
    content: str
    score: float = 0.0
```

Avoid:

```python
@dataclass
class Document:
    score: float = 0.0
    content: str
```

This causes an error because a non-default field follows a default field.

---

# 6. `field()`

The `field()` function provides additional control over dataclass fields.

```python
from dataclasses import dataclass, field
```

Example:

```python
@dataclass
class AgentState:
    messages: list[str] = field(default_factory=list)
```

This is important for mutable values such as lists and dictionaries.

---

# 7. Why `default_factory` Matters

Avoid this pattern:

```python
@dataclass
class AgentState:
    messages: list[str] = []
```

A mutable default value can lead to unexpected behavior.

Prefer:

```python
@dataclass
class AgentState:
    messages: list[str] = field(default_factory=list)
```

Now each object gets its own list.

Example:

```python
state1 = AgentState()
state2 = AgentState()

state1.messages.append("Hello")
```

`state2.messages` remains empty.

---

# 8. Dataclasses and AI Agent State

Agent workflows often maintain state.

For example:

```python
@dataclass
class AgentState:
    user_query: str
    messages: list[str] = field(default_factory=list)
    tool_calls: list[str] = field(default_factory=list)
    final_answer: str | None = None
```

This gives the agent workflow a clearly defined state structure.

Conceptually:

```text
AgentState
├── user_query
├── messages
├── tool_calls
└── final_answer
```

This becomes useful when building custom agent orchestration systems.

---

# 9. `__post_init__`

Sometimes you need additional initialization after the dataclass-generated `__init__`.

Use:

```python
__post_init__
```

Example:

```python
@dataclass
class DocumentChunk:
    content: str
    token_count: int = 0

    def __post_init__(self):
        self.token_count = len(self.content.split())
```

Now:

```python
chunk = DocumentChunk(
    content="Python is useful for AI engineering."
)

print(chunk.token_count)
```

The additional logic runs after initialization.

---

# 10. Dataclasses for RAG

A RAG pipeline can use several data structures.

For example:

```python
@dataclass
class Document:
    id: str
    content: str
    source: str
```

After chunking:

```python
@dataclass
class DocumentChunk:
    id: str
    document_id: str
    content: str
    index: int
```

After retrieval:

```python
@dataclass
class RetrievedChunk:
    chunk: DocumentChunk
    score: float
```

The pipeline becomes:

```text
Document
   ↓
DocumentChunk
   ↓
RetrievedChunk
```

Each stage has an explicit data representation.

---

# 11. Dataclasses for Tool Results

Suppose an agent calls a search tool.

Instead of returning a dictionary:

```python
{
    "success": True,
    "results": ["doc1", "doc2"],
    "count": 2,
}
```

you could define:

```python
@dataclass
class SearchResult:
    success: bool
    results: list[str]
    count: int
```

Then:

```python
def search(query: str) -> SearchResult:
    ...
```

This makes the tool interface easier to understand.

---

# 12. Dataclasses for Evaluation

AI systems often need evaluation results.

For example:

```python
@dataclass
class EvaluationResult:
    query: str
    answer: str
    relevance_score: float
    grounded: bool
```

Then:

```python
result = EvaluationResult(
    query="What is RAG?",
    answer="RAG retrieves relevant context before generating an answer.",
    relevance_score=0.92,
    grounded=True,
)
```

This is useful when building evaluation pipelines.

---

# 13. Frozen Dataclasses

By default, dataclass instances can be modified:

```python
@dataclass
class ModelConfig:
    model: str
    temperature: float
```

You can create an immutable version:

```python
@dataclass(frozen=True)
class ModelConfig:
    model: str
    temperature: float
```

Now:

```python
config = ModelConfig(
    model="example-model",
    temperature=0.2,
)
```

Attempting:

```python
config.temperature = 0.8
```

raises an exception.

Frozen dataclasses can be useful for configuration or value objects that should not change after creation.

---

# 14. Comparing Dataclass Objects

Dataclasses provide equality comparison by default.

```python
@dataclass
class Document:
    id: str
    content: str
```

Then:

```python
doc1 = Document("1", "Hello")
doc2 = Document("1", "Hello")

print(doc1 == doc2)
```

The result is:

```text
True
```

This is useful in testing and data processing.

---

# 15. Ordering

Dataclasses can optionally generate ordering methods.

For example:

```python
@dataclass(order=True)
class SearchResult:
    score: float
    document_id: str
```

This can allow comparisons based on field ordering.

However, don't enable ordering unless it represents a meaningful domain concept.

For retrieval systems, explicitly sorting by score is often clearer:

```python
results.sort(
    key=lambda result: result.score,
    reverse=True,
)
```

---

# 16. Dataclasses vs Dictionaries

Dictionary:

```python
document = {
    "id": "doc-1",
    "content": "...",
    "score": 0.92,
}
```

Dataclass:

```python
@dataclass
class Document:
    id: str
    content: str
    score: float
```

### Dictionary

Advantages:

* flexible
* easy to serialize
* useful for dynamic data

Disadvantages:

* no clear schema
* typo-prone
* less IDE support
* harder to refactor safely

### Dataclass

Advantages:

* explicit structure
* type hints
* IDE support
* readable
* useful for internal application models

Disadvantages:

* less flexible for highly dynamic data
* does not automatically provide runtime validation

---

# 17. Dataclasses vs Pydantic

This distinction is extremely important for AI engineers.

### Dataclass

```python
@dataclass
class ChatRequest:
    query: str
    temperature: float
```

Primarily provides:

```text
Structure
+
Convenient class generation
+
Type annotations
```

### Pydantic

```python
from pydantic import BaseModel


class ChatRequest(BaseModel):
    query: str
    temperature: float
```

Provides:

```text
Structure
+
Type annotations
+
Runtime validation
+
Serialization
+
Parsing
```

A practical rule:

```text
Internal application data
        ↓
Dataclass can be a great choice

External/untrusted data
        ↓
Pydantic is often more appropriate
```

For example:

```text
HTTP request
    ↓
Pydantic model
    ↓
Application logic
    ↓
Dataclasses
```

This is not a strict rule, but it is a useful architectural distinction.

---

# 18. Dataclasses and JSON

Dataclasses don't automatically behave like JSON objects.

You can convert them using `asdict()`:

```python
from dataclasses import asdict


document_dict = asdict(document)
```

Example:

```python
@dataclass
class Document:
    id: str
    content: str


document = Document(
    id="doc-1",
    content="Python for AI",
)

data = asdict(document)
```

Result:

```python
{
    "id": "doc-1",
    "content": "Python for AI",
}
```

This can then be passed to serialization code.

---

# 19. Nested Dataclasses

Dataclasses can contain other dataclasses.

```python
@dataclass
class Document:
    id: str
    content: str


@dataclass
class SearchResult:
    document: Document
    score: float
```

Now:

```python
result = SearchResult(
    document=Document(
        id="doc-1",
        content="RAG retrieves relevant context.",
    ),
    score=0.94,
)
```

This is useful for representing structured AI pipeline results.

---

# 20. Dataclasses and Dependency Injection

Dataclasses can also represent configuration.

```python
@dataclass(frozen=True)
class RAGConfig:
    top_k: int = 5
    similarity_threshold: float = 0.7
    collection_name: str = "documents"
```

A service can receive this configuration:

```python
class Retriever:
    def __init__(self, config: RAGConfig):
        self.config = config
```

This keeps configuration separate from business logic.

---

# 21. Production Example

Consider a multi-agent system.

You might define:

```python
@dataclass
class AgentMessage:
    sender: str
    receiver: str
    content: str


@dataclass
class ToolCall:
    name: str
    arguments: dict


@dataclass
class AgentResult:
    success: bool
    output: str
    tool_calls: list[ToolCall] = field(default_factory=list)
```

The system can then pass explicit objects between components:

```text
Agent
  ↓
AgentMessage
  ↓
Tool Executor
  ↓
ToolCall
  ↓
AgentResult
  ↓
Next Agent
```

This is much easier to reason about than passing arbitrary dictionaries throughout the system.

---

# Common Mistakes

## Mistake 1: Mutable defaults

Avoid:

```python
@dataclass
class State:
    messages: list[str] = []
```

Prefer:

```python
@dataclass
class State:
    messages: list[str] = field(default_factory=list)
```

---

## Mistake 2: Using dataclasses for untrusted external input

Dataclasses don't automatically validate incoming data.

For example, an HTTP request containing:

```json
{
    "temperature": "hello"
}
```

is not automatically validated simply because the target class has:

```python
temperature: float
```

Use runtime validation where required.

---

## Mistake 3: Creating giant dataclasses

Avoid creating one class containing everything:

```text
Agent
RAG
Database
API
Configuration
Evaluation
Logging
```

A dataclass should represent a coherent piece of data.

---

## Mistake 4: Confusing dataclasses with Pydantic

They solve overlapping but different problems.

Think:

```text
Dataclass
→ convenient Python data structure

Pydantic
→ validated data model
```

---

# Production Insight

Dataclasses are particularly useful at **internal boundaries**.

For example:

```text
External API
      ↓
Pydantic
      ↓
Application service
      ↓
Dataclass
      ↓
Retriever
      ↓
Dataclass
      ↓
LLM
      ↓
Pydantic / structured output
      ↓
API response
```

This gives each boundary an appropriate representation.

The important engineering principle is:

> **Choose a data representation based on where the data comes from and how it is used.**

Don't automatically use dictionaries everywhere just because Python makes them convenient.

---

# How It's Used in AI Frameworks

Dataclass-based patterns appear frequently in Python AI applications for:

* configuration
* state
* tool results
* retrieval results
* evaluation records
* internal domain objects

Frameworks may also use their own models for specific purposes.

For example:

```text
FastAPI
    ↓
Pydantic models

Custom AI pipeline
    ↓
Dataclasses

LLM structured output
    ↓
Pydantic / schema-based models
```

The exact choice depends on the framework and the application's requirements.

---

# Interview Questions

### Beginner

**1. What is a dataclass?**

A Python class designed to reduce boilerplate when creating classes primarily used to store data.

**2. What does `@dataclass` do?**

It automatically generates common methods such as `__init__`, `__repr__`, and `__eq__` based on the declared fields and configuration.

**3. Why use `default_factory`?**

It creates a new default value for each instance, which is important for mutable objects such as lists and dictionaries.

---

### Intermediate

**4. What is `__post_init__`?**

A method that runs after the generated dataclass `__init__` method and can be used for additional initialization or derived values.

**5. What does `frozen=True` do?**

It makes dataclass instances effectively immutable by preventing normal field reassignment.

**6. What is the difference between a dataclass and a dictionary?**

A dataclass provides a defined structure, attributes, type annotations, and generated methods, while a dictionary is more flexible and dynamic.

---

### AI Engineering

**7. When would you use a dataclass instead of Pydantic?**

Dataclasses are often useful for internal application data where runtime validation is not required. Pydantic is generally more appropriate when parsing or validating external or untrusted data.

**8. How can dataclasses help in RAG systems?**

They can represent structured objects such as documents, chunks, retrieval results, and evaluation records.

**9. How can dataclasses help in agent systems?**

They can represent agent state, messages, tool calls, tool results, and other internal workflow data.

**10. Why shouldn't every AI response be represented as a dictionary?**

Explicit data structures make contracts clearer, improve IDE support, reduce key-related mistakes, and make refactoring and testing easier.

---

# Practical Exercise

Create three dataclasses:

```python
Document
DocumentChunk
RetrievedChunk
```

Use the following relationships:

```text
Document
├── id
├── content
└── source

DocumentChunk
├── id
├── document_id
├── content
└── index

RetrievedChunk
├── chunk
└── score
```

Then create a small pipeline:

```text
Document
    ↓
DocumentChunk
    ↓
RetrievedChunk
```

Finally, use `asdict()` to convert the result into a dictionary.

---

# Summary

Remember:

```text
@dataclass
    ↓
Reduce class boilerplate

field()
    ↓
Customize fields

default_factory
    ↓
Safe mutable defaults

__post_init__
    ↓
Post-initialization logic

frozen=True
    ↓
Immutable-style objects

asdict()
    ↓
Convert dataclass → dictionary
```

For AI engineering, dataclasses are particularly useful for **internal structured data**.

A useful mental model is:

```text
Dictionary
→ flexible but loosely structured

Dataclass
→ structured internal Python data

Pydantic
→ structured + runtime validation
```

Use the simplest representation that gives your system the guarantees it actually needs.
