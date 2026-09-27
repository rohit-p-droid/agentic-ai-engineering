# Pydantic

Pydantic is a Python library for defining and validating structured data using Python type annotations.

It is especially important for AI engineers because modern AI applications constantly handle data coming from external or unreliable sources:

* API requests
* LLM outputs
* tool calls
* configuration files
* databases
* RAG pipelines
* agent state
* microservices

A normal Python type hint tells developers what a value is expected to be.

Pydantic can additionally **validate data at runtime**.

For example:

```python
from pydantic import BaseModel


class ChatRequest(BaseModel):
    query: str
    temperature: float
```

Now `ChatRequest` defines an explicit data contract for the application.

---

# Why does an AI Engineer need this?

Consider an AI API:

```text
Client
  ↓
HTTP Request
  ↓
FastAPI
  ↓
Validation
  ↓
AI Service
  ↓
Agent
  ↓
LLM
```

The incoming request cannot be trusted to have the correct structure.

It might contain:

```json
{
  "query": "Explain RAG",
  "temperature": 0.7
}
```

But it could also contain:

```json
{
  "query": 123,
  "temperature": "hello"
}
```

Pydantic allows the application to define what valid data looks like and reject or process invalid data according to the model's validation rules.

This makes it extremely useful at system boundaries.

---

# 1. Creating a Pydantic Model

Install Pydantic if necessary:

```bash
python -m pip install pydantic
```

Create a model:

```python
from pydantic import BaseModel


class ChatRequest(BaseModel):
    query: str
    temperature: float
```

Create an object:

```python
request = ChatRequest(
    query="What is RAG?",
    temperature=0.7,
)
```

Access values:

```python
print(request.query)
print(request.temperature)
```

---

# 2. Validation

Pydantic validates data when creating a model.

For example:

```python
from pydantic import BaseModel, ValidationError


class ChatRequest(BaseModel):
    query: str
    temperature: float


try:
    request = ChatRequest(
        query="What is RAG?",
        temperature="invalid",
    )
except ValidationError as error:
    print(error)
```

The invalid input produces a validation error instead of silently entering the application as arbitrary data.

---

# 3. Type Coercion

Pydantic may convert compatible input types depending on the field and model configuration.

For example:

```python
class SearchRequest(BaseModel):
    top_k: int
```

Input such as:

```python
request = SearchRequest(top_k="5")
```

may be converted to:

```python
request.top_k == 5
```

However, don't depend on implicit conversion blindly.

For APIs and critical boundaries, explicitly define the behavior you want and use strict types when necessary.

---

# 4. Strict Types

Pydantic provides strict types when coercion is undesirable.

```python
from pydantic import BaseModel, StrictInt


class SearchRequest(BaseModel):
    top_k: StrictInt
```

Now values that merely look like integers are not automatically accepted as integers.

Strict validation can be useful when the distinction between types matters.

---

# 5. Optional Fields

A field can allow `None`:

```python
class ChatRequest(BaseModel):
    query: str
    system_prompt: str | None = None
```

Now:

```python
request = ChatRequest(
    query="Explain embeddings",
)
```

is valid.

The default value is:

```python
None
```

---

# 6. Default Values

You can define defaults:

```python
class SearchRequest(BaseModel):
    query: str
    top_k: int = 5
    threshold: float = 0.7
```

Now:

```python
request = SearchRequest(
    query="What is hybrid search?"
)
```

automatically gets:

```text
top_k = 5
threshold = 0.7
```

---

# 7. Field Constraints

Pydantic can express constraints on fields.

```python
from pydantic import BaseModel, Field


class SearchRequest(BaseModel):
    query: str = Field(min_length=1)
    top_k: int = Field(gt=0, le=100)
```

This means:

```text
query
→ must not be empty

top_k
→ greater than 0
→ less than or equal to 100
```

This is useful for API validation.

---

# 8. AI API Example

Suppose an application exposes:

```text
POST /chat
```

The request could be:

```python
class ChatRequest(BaseModel):
    query: str = Field(min_length=1)
    temperature: float = Field(ge=0.0, le=2.0)
```

This prevents invalid values such as:

```text
empty query
temperature < 0
temperature > 2
```

before the request reaches the LLM service.

---

# 9. Nested Models

Pydantic models can contain other Pydantic models.

```python
class User(BaseModel):
    id: str
    name: str


class ChatRequest(BaseModel):
    user: User
    query: str
```

Input:

```python
request = ChatRequest(
    user={
        "id": "user-123",
        "name": "Rohit",
    },
    query="Explain RAG",
)
```

Pydantic validates the nested structure.

This is useful for complex AI APIs.

---

# 10. Lists of Models

You can define structured lists:

```python
class Document(BaseModel):
    id: str
    content: str
    score: float


class SearchResponse(BaseModel):
    results: list[Document]
```

Example:

```python
response = SearchResponse(
    results=[
        {
            "id": "doc-1",
            "content": "RAG combines retrieval with generation.",
            "score": 0.94,
        },
        {
            "id": "doc-2",
            "content": "Embeddings represent semantic meaning.",
            "score": 0.89,
        },
    ]
)
```

This gives the retrieval response a defined structure.

---

# 11. Pydantic for RAG

A RAG pipeline can define models such as:

```python
class DocumentChunk(BaseModel):
    id: str
    content: str
    source: str
    metadata: dict[str, str]
```

Retrieved chunks:

```python
class RetrievedChunk(BaseModel):
    chunk: DocumentChunk
    score: float
```

And the final response:

```python
class RAGResponse(BaseModel):
    answer: str
    sources: list[str]
```

The pipeline now has explicit contracts:

```text
DocumentChunk
      ↓
RetrievedChunk
      ↓
RAGResponse
```

---

# 12. Pydantic for Agent Tools

Agents frequently call tools with structured arguments.

For example:

```python
class SearchToolInput(BaseModel):
    query: str = Field(min_length=1)
    limit: int = Field(default=5, gt=0, le=20)
```

A tool can then receive validated arguments.

Conceptually:

```text
LLM
 ↓
Tool call
 ↓
Pydantic validation
 ↓
Tool execution
```

This is an important pattern because LLM-generated tool arguments should not automatically be trusted.

---

# 13. Pydantic for Structured LLM Output

One of the most useful AI-engineering applications of Pydantic is structured output.

Suppose an LLM should classify a support request:

```python
class Classification(BaseModel):
    category: str
    priority: str
    explanation: str
```

Instead of asking the model to return arbitrary text, the application can request a structured response conforming to a schema.

Conceptually:

```text
LLM
 ↓
Structured output
 ↓
Pydantic model
 ↓
Application logic
```

This is much easier to consume programmatically than free-form text.

The exact API for structured outputs depends on the LLM provider or framework being used.

---

# 14. Validation Is Different from Prompting

Consider this prompt:

```text
Return the priority as either "low", "medium", or "high".
```

The model may still produce unexpected output.

A Pydantic model provides an application-level contract:

```python
class Ticket(BaseModel):
    priority: str
```

Better yet, constrain the value using an enum.

```python
from enum import Enum


class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"


class Ticket(BaseModel):
    priority: Priority
```

Now the application has a machine-readable definition of valid values.

---

# 15. Enums

Enums are useful when a field can only have a fixed set of values.

```python
from enum import Enum


class AgentStatus(str, Enum):
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"
```

Then:

```python
class AgentResult(BaseModel):
    status: AgentStatus
    output: str
```

This is much safer than passing arbitrary strings around.

---

# 16. Custom Validators

Sometimes basic field constraints are not enough.

Pydantic supports validators.

For example:

```python
from pydantic import BaseModel, field_validator


class SearchRequest(BaseModel):
    query: str

    @field_validator("query")
    @classmethod
    def validate_query(cls, value: str) -> str:
        value = value.strip()

        if not value:
            raise ValueError("Query cannot be empty")

        return value
```

The validator executes when the model is created.

This is useful when validation requires custom business logic.

---

# 17. Model Validators

Sometimes validation depends on multiple fields.

For example:

```python
from pydantic import BaseModel, model_validator


class SearchRequest(BaseModel):
    query: str
    top_k: int
    threshold: float

    @model_validator(mode="after")
    def validate_request(self):
        if self.top_k < 5 and self.threshold > 0.95:
            raise ValueError(
                "High threshold with very small top_k may be too restrictive"
            )

        return self
```

Use model-level validation when the validity of one field depends on another.

---

# 18. Serialization

Pydantic models can be converted into dictionaries:

```python
request.model_dump()
```

Example:

```python
request = ChatRequest(
    query="Explain RAG",
    temperature=0.7,
)

data = request.model_dump()
```

Result:

```python
{
    "query": "Explain RAG",
    "temperature": 0.7,
}
```

For JSON:

```python
request.model_dump_json()
```

These operations are useful when communicating with APIs, queues, databases, or other services.

---

# 19. Deserialization

Pydantic can also construct models from external data.

For example:

```python
data = {
    "query": "Explain embeddings",
    "temperature": 0.5,
}

request = ChatRequest.model_validate(data)
```

The data is parsed and validated against the model.

---

# 20. Configuration with Pydantic

AI applications often require environment variables:

```text
OPENAI_API_KEY
QDRANT_URL
QDRANT_API_KEY
DATABASE_URL
MODEL_NAME
```

Pydantic Settings can be used to represent application configuration.

For example:

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    openai_api_key: str
    qdrant_url: str
    model_name: str = "default-model"
```

Then the application can load configuration from the environment.

This keeps configuration separate from application logic.

---

# 21. Pydantic and FastAPI

Pydantic is deeply integrated into FastAPI.

Example:

```python
from fastapi import FastAPI
from pydantic import BaseModel


app = FastAPI()


class ChatRequest(BaseModel):
    query: str
    temperature: float = 0.7


class ChatResponse(BaseModel):
    answer: str


@app.post("/chat", response_model=ChatResponse)
def chat(request: ChatRequest) -> ChatResponse:
    return ChatResponse(
        answer=f"You asked: {request.query}"
    )
```

The flow becomes:

```text
HTTP Request
     ↓
Pydantic validation
     ↓
Python function
     ↓
Pydantic response model
     ↓
HTTP Response
```

FastAPI can also use these models to generate API schemas and documentation.

---

# 22. Request vs Response Models

Do not automatically use the same model for both input and output.

For example:

```python
class ChatRequest(BaseModel):
    query: str
```

and:

```python
class ChatResponse(BaseModel):
    answer: str
    sources: list[str]
```

This makes the API contract explicit.

For a production AI API:

```text
Client
 ↓
ChatRequest
 ↓
Agent / RAG
 ↓
ChatResponse
 ↓
Client
```

---

# 23. Pydantic and Microservices

Suppose your system has:

```text
API Service
      ↓
Agent Service
      ↓
RAG Service
      ↓
Vector Service
```

Each service needs to agree on data structures.

Pydantic models can define those contracts.

For example:

```python
class AgentRequest(BaseModel):
    user_id: str
    query: str
    session_id: str
```

And:

```python
class AgentResponse(BaseModel):
    answer: str
    session_id: str
    sources: list[str]
```

This makes service boundaries explicit.

The same concept applies when services communicate over HTTP, queues, or RPC systems.

---

# 24. Pydantic and gRPC

In a system where Python and another service communicate through gRPC, the primary wire contract is defined by Protocol Buffers.

For example:

```text
Python Service
      ↓
gRPC / Protobuf
      ↓
NestJS Service
```

Pydantic can still be useful **inside the Python service** for validating and transforming application-level data before it reaches the gRPC boundary.

Conceptually:

```text
HTTP / internal data
       ↓
Pydantic
       ↓
Application logic
       ↓
Protobuf message
       ↓
gRPC
```

Pydantic and Protobuf solve different problems.

---

# 25. Pydantic vs Dataclasses

This distinction is important.

| Feature                   | Dataclass     | Pydantic         |
| ------------------------- | ------------- | ---------------- |
| Type annotations          | Yes           | Yes              |
| Boilerplate reduction     | Yes           | Yes              |
| Runtime validation        | No by default | Yes              |
| Serialization             | Basic/manual  | Built-in helpers |
| External input validation | Limited       | Strong           |
| API models                | Possible      | Excellent fit    |
| Internal data structures  | Excellent fit | Also suitable    |
| LLM structured outputs    | Possible      | Excellent fit    |

A practical mental model:

```text
Dataclass
→ Internal Python data structure

Pydantic
→ Validated data model
```

This is a guideline, not a hard rule.

---

# 26. Pydantic vs Type Hints

Type hints:

```python
def search(query: str) -> list[str]:
    ...
```

describe what the function expects.

Pydantic:

```python
class SearchRequest(BaseModel):
    query: str
```

can validate actual runtime data.

Think:

```text
Type hints
    ↓
Static description

Pydantic
    ↓
Runtime validation + structured data
```

They complement each other rather than replace each other.

---

# 27. Common Mistakes

## Mistake 1: Treating Pydantic as only an API library

Pydantic is useful beyond FastAPI.

It can be used for:

* configuration
* tool inputs
* LLM outputs
* RAG results
* internal boundaries
* microservice contracts

---

## Mistake 2: Trusting LLM output without validation

This is risky:

```python
result = llm.invoke(prompt)

priority = result["priority"]
```

The model's output may not always have the expected structure.

A schema-based approach is safer:

```text
LLM
 ↓
Structured output
 ↓
Validation
 ↓
Application logic
```

---

## Mistake 3: Using one giant model

Avoid creating a model containing every field used throughout the application.

Prefer focused models:

```text
ChatRequest
ChatResponse
SearchRequest
SearchResult
ToolInput
ToolResult
```

Each model should represent a meaningful contract.

---

## Mistake 4: Overusing custom validators

Not every field needs custom validation.

Prefer built-in constraints where possible:

```python
Field(gt=0, le=100)
```

Use custom validators when actual business logic requires them.

---

## Mistake 5: Confusing validation with security

Pydantic validates data structure and declared constraints.

It does not automatically make an application secure.

You still need:

* authentication
* authorization
* input sanitization where appropriate
* rate limiting
* secret management
* access control
* safe tool execution

---

# Production Insight

Pydantic becomes particularly valuable at **trust boundaries**.

Think about where data enters your system:

```text
User
 ↓
HTTP API
 ↓
Pydantic
 ↓
Application
```

Or:

```text
LLM
 ↓
Structured output
 ↓
Pydantic
 ↓
Application
```

Or:

```text
Configuration
 ↓
Pydantic Settings
 ↓
Application
```

The general pattern is:

```text
Untrusted / external data
        ↓
Validation
        ↓
Trusted application representation
        ↓
Business logic
```

This is one of the most important patterns for production AI engineering.

---

# How It's Used in AI Frameworks

Pydantic is widely used across the Python AI ecosystem for structured data.

Common use cases include:

### FastAPI

Request and response validation.

```python
class ChatRequest(BaseModel):
    query: str
```

### LangChain

Pydantic models can be used for structured tool inputs and structured outputs, depending on the integration.

### LangGraph

Typed state and structured data models can be used when defining application state and workflow boundaries.

### OpenAI and other LLM providers

Structured-output integrations can use schemas to constrain model responses. The exact APIs vary by provider and SDK version.

### Configuration

`pydantic-settings` can be used to load and validate application configuration.

---

# Interview Questions

### Beginner

**1. What is Pydantic?**

A Python library for defining structured data models and validating data using Python type annotations.

**2. What is `BaseModel`?**

The base class commonly used to create Pydantic models.

**3. Does Pydantic perform runtime validation?**

Yes. Pydantic validates data when models are created or explicitly validated.

---

### Intermediate

**4. What is the difference between type hints and Pydantic?**

Type hints describe expected types and support static analysis. Pydantic uses type annotations to validate and parse actual runtime data.

**5. What is `Field()` used for?**

It can define metadata, defaults, and validation constraints for model fields.

**6. What is a Pydantic validator?**

A function that applies custom validation or transformation logic to model data.

**7. What is serialization?**

Converting a structured object into a representation such as a dictionary or JSON.

---

### AI Engineering

**8. Why is Pydantic useful for LLM applications?**

LLMs produce probabilistic outputs. Pydantic can provide a structured application-level contract for consuming those outputs.

**9. How can Pydantic be used with agent tools?**

A Pydantic model can define and validate the arguments expected by a tool before the tool executes.

**10. Why is Pydantic useful in RAG systems?**

It can define structured models for documents, chunks, retrieval results, queries, and final responses.

**11. How does Pydantic improve microservice architecture?**

It makes service inputs and outputs explicit and validates data at application boundaries.

**12. Pydantic vs dataclass?**

Dataclasses are lightweight structured Python objects. Pydantic adds runtime validation, parsing, and serialization capabilities.

---

# Practical Exercise

Build models for a small RAG API.

Create:

```python
class SearchRequest(BaseModel):
    query: str
    top_k: int
```

Then:

```python
class SearchResult(BaseModel):
    id: str
    content: str
    score: float
```

Finally:

```python
class SearchResponse(BaseModel):
    results: list[SearchResult]
```

Add validation so that:

```text
query
→ cannot be empty

top_k
→ must be between 1 and 20

score
→ must be between 0 and 1
```

Then test both valid and invalid inputs.

---

# Summary

Remember:

```text
BaseModel
    ↓
Create structured models

Field
    ↓
Define constraints

Validation
    ↓
Protect application boundaries

Nested models
    ↓
Represent complex data

Validators
    ↓
Custom validation logic

Enums
    ↓
Restrict values

model_dump()
    ↓
Convert model → dictionary

model_validate()
    ↓
Validate external data

Pydantic Settings
    ↓
Validate configuration
```

For AI engineering, one of the most important patterns is:

```text
External Data
     ↓
Pydantic Validation
     ↓
Application Logic
     ↓
AI / Agent / RAG
     ↓
Structured Pydantic Response
     ↓
External System
```

The key idea is:

> **Type hints describe what your data should look like. Pydantic lets your application enforce that structure at runtime.**
