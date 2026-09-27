# Type Hints

Type hints allow you to explicitly describe the expected types of variables, function parameters, return values, and other parts of Python code.

Python is dynamically typed, so this is valid:

```python
name = "Rohit"
name = 42
```

Python will not prevent the variable from changing types.

Type hints allow us to communicate and validate the intended structure of our code.

For AI engineers, this becomes particularly useful because AI applications often move structured data between:

* LLMs
* agents
* tools
* APIs
* databases
* RAG pipelines
* background jobs
* microservices

---

# Why does an AI Engineer need this?

AI applications often contain complex data flows.

For example:

```text
User Query
    ↓
Query Processor
    ↓
Retriever
    ↓
Retrieved Documents
    ↓
LLM
    ↓
Structured Response
    ↓
API
```

Without type hints, it can become difficult to understand what each component expects and returns.

With type hints:

```python
def retrieve_documents(query: str) -> list[str]:
    ...
```

we immediately know:

```text
Input  → string
Output → list of strings
```

This makes large AI systems easier to:

* understand
* maintain
* refactor
* test
* debug
* review

---

# 1. Basic Type Hints

A variable can have a type annotation:

```python
name: str = "Rohit"
age: int = 25
temperature: float = 0.7
is_streaming: bool = True
```

The syntax is:

```python
variable: type = value
```

Python does not automatically enforce these annotations at runtime.

They primarily provide information to:

* developers
* IDEs
* static type checkers
* libraries and frameworks

---

# 2. Function Parameter Types

Type hints become especially useful for functions.

```python
def create_prompt(query: str) -> str:
    return f"Answer this question: {query}"
```

The function expects:

```text
query → str
```

and returns:

```text
str
```

Another example:

```python
def calculate_similarity(score1: float, score2: float) -> float:
    return (score1 + score2) / 2
```

---

# 3. Return Types

You can specify what a function returns:

```python
def get_user_name() -> str:
    return "Rohit"
```

List:

```python
def get_documents() -> list[str]:
    return [
        "Document 1",
        "Document 2",
    ]
```

Dictionary:

```python
def get_metadata() -> dict[str, str]:
    return {
        "source": "documentation",
        "type": "markdown",
    }
```

---

# 4. Common Built-in Types

Some common Python type hints are:

```python
str
int
float
bool
list
dict
tuple
set
```

Examples:

```python
documents: list[str]

scores: list[float]

metadata: dict[str, str]

coordinates: tuple[float, float]

unique_ids: set[str]
```

---

# 5. Optional Values

Sometimes a function may return a value or `None`.

For example:

```python
def find_document(document_id: str) -> str | None:
    ...
```

This means:

```text
Return:
    str
    OR
    None
```

Example:

```python
def find_document(document_id: str) -> str | None:
    if document_id == "doc-1":
        return "AI Engineering Guide"

    return None
```

Modern Python allows:

```python
str | None
```

instead of the older:

```python
Optional[str]
```

---

# 6. Union Types

A value may have multiple possible types.

```python
document_id: int | str
```

This means:

```text
document_id can be:
    int
    OR
    str
```

Example:

```python
def get_document(document_id: int | str) -> str:
    ...
```

Use union types when multiple input types are genuinely supported.

Do not use them simply because the code is unclear.

---

# 7. Any

Python provides:

```python
from typing import Any
```

Example:

```python
def process_data(data: Any):
    ...
```

`Any` essentially tells the type checker:

> Do not make assumptions about this value's type.

This can be useful when dealing with genuinely dynamic data.

However, overusing `Any` weakens the benefits of type hints.

For example, this:

```python
data: Any
```

provides much less information than:

```python
data: dict[str, str]
```

Prefer specific types whenever practical.

---

# 8. Type Aliases

You can create reusable names for complex types.

```python
DocumentMetadata = dict[str, str]
```

Then:

```python
def get_metadata() -> DocumentMetadata:
    ...
```

Another example:

```python
Embedding = list[float]
```

Now:

```python
def generate_embedding(text: str) -> Embedding:
    ...
```

Type aliases can make AI pipelines easier to read.

---

# 9. Typed Dictionaries

Sometimes a dictionary has a known structure.

For example:

```python
from typing import TypedDict


class Document(TypedDict):
    id: str
    content: str
    source: str
```

Now:

```python
document: Document = {
    "id": "doc-123",
    "content": "Python is useful for AI engineering.",
    "source": "documentation",
}
```

This is more descriptive than:

```python
document: dict
```

Typed dictionaries are useful when working with structured dictionary-based data.

---

# 10. Type Hints for RAG

Consider a retrieval function:

```python
def retrieve_documents(
    query: str,
    top_k: int,
) -> list[str]:
    ...
```

This immediately communicates the contract:

```text
query → string
top_k → integer
result → list of strings
```

A more structured example:

```python
class RetrievedDocument(TypedDict):
    id: str
    content: str
    score: float
```

Then:

```python
def retrieve_documents(
    query: str,
    top_k: int,
) -> list[RetrievedDocument]:
    ...
```

Now the retrieval pipeline has a clearly defined output.

---

# 11. Type Hints for Agents

An agent may receive a task and return a result.

```python
def run_agent(task: str) -> str:
    ...
```

A more structured system could define:

```python
class AgentResult(TypedDict):
    answer: str
    tool_used: str
    success: bool
```

Then:

```python
def run_agent(task: str) -> AgentResult:
    ...
```

This makes agent-to-agent communication easier to understand.

---

# 12. Type Hints for Tools

Consider an agent tool:

```python
def search_documents(
    query: str,
    limit: int = 5,
) -> list[str]:
    ...
```

The signature itself describes the tool contract.

This becomes especially valuable when an agent needs to interact with multiple tools.

---

# 13. Callable

Sometimes a function receives another function.

For example:

```python
from collections.abc import Callable
```

You can define:

```python
def process_text(
    text: str,
    processor: Callable[[str], str],
) -> str:
    return processor(text)
```

This means:

```text
processor:
    accepts str
    returns str
```

Example:

```python
def clean_text(text: str) -> str:
    return text.strip().lower()


result = process_text(
    " Hello AI ",
    clean_text,
)
```

This pattern is useful for configurable AI pipelines.

---

# 14. Generics

Generics allow reusable components to work with different types.

For example:

```python
from typing import TypeVar

T = TypeVar("T")


def first_item(items: list[T]) -> T:
    return items[0]
```

The function can work with:

```python
numbers = [1, 2, 3]

first_number = first_item(numbers)
```

or:

```python
documents = ["doc1", "doc2"]

first_document = first_item(documents)
```

The type remains consistent.

Generics become useful when building reusable infrastructure and framework-level components.

---

# 15. Protocol

`Protocol` can describe behavior rather than requiring a specific class hierarchy.

Example:

```python
from typing import Protocol


class Retriever(Protocol):
    def retrieve(self, query: str) -> list[str]:
        ...
```

Any class implementing the expected method can satisfy the protocol.

For example:

```python
class QdrantRetriever:
    def retrieve(self, query: str) -> list[str]:
        return []
```

This allows components to depend on an interface rather than a concrete implementation.

This is useful for AI systems because you may want to switch between:

```text
Qdrant
Pinecone
Weaviate
Elasticsearch
custom retriever
```

without changing the rest of the application.

---

# 16. Type Hints and Dependency Injection

Consider a service:

```python
class RAGService:

    def __init__(self, retriever: Retriever):
        self.retriever = retriever
```

The service depends on the behavior:

```text
Retriever
```

rather than:

```text
QdrantRetriever
```

This makes testing easier.

For example:

```python
class FakeRetriever:
    def retrieve(self, query: str) -> list[str]:
        return ["test document"]
```

You can inject the fake implementation during tests.

This is a powerful software-engineering pattern for AI applications.

---

# 17. Type Hints and Pydantic

Type hints become particularly powerful when combined with Pydantic.

Example:

```python
from pydantic import BaseModel


class ChatRequest(BaseModel):
    query: str
    temperature: float = 0.7
```

Now the model describes structured input:

```text
ChatRequest
├── query: str
└── temperature: float
```

This is heavily used in API development and AI applications.

For example, FastAPI can use Pydantic models to define API request and response structures.

---

# 18. Type Hints for LLM Responses

Suppose an LLM should return:

```text
answer
confidence
sources
```

Instead of passing around an unstructured dictionary:

```python
response = {
    "answer": "...",
    "confidence": 0.92,
    "sources": ["doc-1", "doc-2"],
}
```

you can define a model:

```python
class Answer:
    answer: str
    confidence: float
    sources: list[str]
```

For production applications, a Pydantic model is often more appropriate when runtime validation is required:

```python
from pydantic import BaseModel


class Answer(BaseModel):
    answer: str
    confidence: float
    sources: list[str]
```

This combines:

```text
Type information
+
Runtime validation
+
Structured data
```

---

# 19. Type Hints Do Not Provide Runtime Validation

This is an important distinction.

Consider:

```python
def add(a: int, b: int) -> int:
    return a + b
```

Python does not automatically reject:

```python
add("10", "20")
```

Type hints are primarily used by static analysis tools.

If runtime validation is required, use mechanisms such as:

* Pydantic
* explicit validation
* custom validation logic

Think of it as:

```text
Type hints
    ↓
Developer + static analysis

Pydantic
    ↓
Runtime validation
```

---

# 20. Static Type Checkers

Tools can analyze your code without running it.

Common options include:

* mypy
* Pyright
* basedpyright

Example:

```python
def get_score() -> float:
    return "0.95"
```

A type checker can detect that the function is declared to return:

```text
float
```

but actually returns:

```text
str
```

This allows many problems to be found before production.

---

# 21. Type Hints in IDEs

Modern IDEs use type hints to provide:

* autocomplete
* navigation
* warnings
* documentation
* refactoring support
* type checking

For a large AI codebase, this can significantly improve development speed.

For example:

```python
retriever.retrieve(...)
```

An IDE can understand what methods are available when `retriever` has a defined type.

---

# 22. Production Example

Imagine a RAG service:

```python
class RetrievedChunk:
    id: str
    content: str
    score: float


class Retriever:
    def search(
        self,
        query: str,
        top_k: int,
    ) -> list[RetrievedChunk]:
        ...


class AnswerGenerator:
    def generate(
        self,
        query: str,
        chunks: list[RetrievedChunk],
    ) -> str:
        ...
```

The data flow becomes explicit:

```text
query: str
    ↓
Retriever.search()
    ↓
list[RetrievedChunk]
    ↓
AnswerGenerator.generate()
    ↓
str
```

This is much easier to reason about than passing unstructured dictionaries everywhere.

---

# 23. Common Mistakes

## Mistake 1: Using `Any` everywhere

Avoid:

```python
def process(data: Any) -> Any:
    ...
```

when the actual structure is known.

Prefer:

```python
def process(data: dict[str, str]) -> list[str]:
    ...
```

---

## Mistake 2: Thinking type hints enforce types

This:

```python
age: int = "twenty"
```

does not automatically produce a runtime exception just because of the annotation.

Use static type checking or runtime validation where appropriate.

---

## Mistake 3: Making types unnecessarily complicated

Avoid creating extremely complex type expressions when a simple model would be clearer.

Instead of:

```python
dict[str, list[dict[str, str | float | None]]]
```

consider creating a structured model.

For example:

```python
class SearchResult:
    ...
```

Readable types are better types.

---

## Mistake 4: Ignoring return types

This:

```python
def retrieve(query: str):
    ...
```

provides less information than:

```python
def retrieve(query: str) -> list[RetrievedChunk]:
    ...
```

For production code, return types are especially valuable.

---

# Production Insight

Type hints become increasingly valuable as an AI application grows.

A small prototype may contain:

```text
100 lines
```

and type hints may seem optional.

A production AI system may contain:

```text
API
 ↓
Agent
 ↓
Tools
 ↓
RAG
 ↓
Retriever
 ↓
Vector DB
 ↓
LLM
 ↓
Evaluation
 ↓
Observability
```

When dozens of components communicate with each other, explicit contracts become extremely useful.

A good rule is:

> **Use types to make boundaries between components explicit.**

Especially at boundaries such as:

```text
API → Service
Service → Agent
Agent → Tool
Retriever → RAG pipeline
Microservice → Microservice
LLM → Application
```

---

# How It's Used in AI Frameworks

Type hints are common throughout modern Python AI tooling.

### FastAPI

FastAPI uses Python type annotations to define API parameters and integrate with validation and schema generation.

```python
@app.post("/chat")
def chat(request: ChatRequest) -> ChatResponse:
    ...
```

### Pydantic

Pydantic uses Python type annotations to define structured data models and perform runtime validation.

```python
class ChatRequest(BaseModel):
    query: str
```

### LangChain / LangGraph

Typed Python code can be used to describe:

* tool inputs
* state
* configuration
* structured outputs
* application components

For complex agent systems, explicitly typed state and data structures can make workflows easier to understand and maintain.

---

# Interview Questions

### Beginner

**1. What are type hints in Python?**

They are annotations that describe the expected types of variables, function parameters, and return values.

**2. Are Python type hints enforced at runtime?**

Not by Python itself. Static type checkers can analyze them, while libraries such as Pydantic can provide runtime validation.

**3. What does this mean?**

```python
def search(query: str) -> list[str]:
```

The function expects a string and is expected to return a list of strings.

---

### Intermediate

**4. What is the difference between `Any` and a specific type?**

`Any` disables most static type checking for that value, while a specific type provides information that tools can use to detect errors.

**5. What is `Optional` / `T | None`?**

It indicates that a value can either contain the specified type or `None`.

```python
str | None
```

**6. What is a `TypedDict`?**

It allows you to describe the expected keys and value types of a dictionary.

**7. What is a `Protocol`?**

It allows you to define an interface based on behavior rather than requiring classes to inherit from a particular base class.

---

### AI Engineering

**8. Why are type hints useful in RAG systems?**

They make data contracts explicit between components such as document loaders, retrievers, rerankers, and answer generators.

**9. Why are type hints useful in agent systems?**

Agents interact with tools, state, models, and other components. Explicit types make those interfaces easier to understand, test, and maintain.

**10. Type hints vs Pydantic — what's the difference?**

Type hints primarily describe expected types and support static analysis.

Pydantic models can use those annotations to perform runtime validation and structured data handling.

**11. How can type hints improve microservice development?**

They make service interfaces clearer and reduce ambiguity when data is passed between APIs or components.

---

# Practical Exercise

Build a small typed RAG interface.

Define:

```python
class RetrievedChunk:
    ...
```

with:

```text
id
content
score
```

Then define:

```python
class Retriever:
    def search(
        self,
        query: str,
        top_k: int,
    ) -> list[RetrievedChunk]:
        ...
```

Finally create:

```python
def generate_answer(
    query: str,
    chunks: list[RetrievedChunk],
) -> str:
    ...
```

The goal is not to build a real RAG system yet.

The goal is to practice designing **clear contracts between AI components**.

---

# Summary

Remember these concepts:

```text
Type hints
    ↓
Describe expected types

Function annotations
    ↓
Inputs + outputs

TypedDict
    ↓
Typed dictionary structures

Generics
    ↓
Reusable typed components

Protocol
    ↓
Behavior-based interfaces

Pydantic
    ↓
Runtime validation + structured data

Static type checker
    ↓
Find type-related problems before runtime
```

For AI engineering, type hints are especially valuable at component boundaries.

The goal is not to add types everywhere just for the sake of it.

The goal is to make the **data flowing through your AI system explicit, understandable, and maintainable**.
