# Testing in Python for AI Engineering

## Why does an AI Engineer need this?

AI systems are software systems.

Even when an application uses LLMs, agents, RAG, embeddings, and probabilistic outputs, the surrounding software still needs reliable tests.

Consider a production AI system:

```text
User
 ↓
API
 ↓
Authentication
 ↓
Agent
 ↓
RAG
 ├── Query transformation
 ├── Retrieval
 ├── Reranking
 └── Context construction
 ↓
LLM
 ↓
Tools
 ↓
Database / External APIs
 ↓
Response
```

A failure anywhere in this pipeline can affect the user.

Testing helps answer:

* Does this function behave correctly?
* Does the API validate input correctly?
* Does the retriever return the expected documents?
* Does the agent transition between states correctly?
* Does a tool handle failures correctly?
* Does the application recover from external API failures?
* Did a code change break existing behavior?

For AI engineers, testing is especially important because AI applications combine:

```text
Deterministic software
+
External services
+
Probabilistic model outputs
+
Distributed systems
```

The testing strategy therefore needs to distinguish between what can be tested exactly and what needs evaluation.

---

# 1. What is Software Testing?

Testing means checking whether software behaves as expected.

A simple example:

```python
def add(a, b):
    return a + b
```

A test might verify:

```python
assert add(2, 3) == 5
```

The test defines expected behavior.

If a future change causes:

```python
add(2, 3)
```

to return something other than `5`, the test can detect the regression.

---

# 2. Why Testing AI Applications Is Different

Traditional software often has deterministic outputs.

For example:

```python
calculate_total(100, 20)
```

should consistently return:

```text
120
```

LLM output may not be deterministic.

For example:

```text
User:
Explain this document.
```

The model may produce multiple valid answers.

Therefore, you generally should not test:

```python
assert response == "The exact expected sentence..."
```

for every LLM response.

Instead, test different layers separately.

---

# 3. Testing Pyramid

A useful model is:

```text
                /\
               /  \
              / E2E\
             /------ \
            /Integration\
           /------------\
          /     Unit     \
         /----------------\
```

The lower levels usually contain more tests because they are:

* Faster
* Cheaper
* More deterministic
* Easier to debug

Typical distribution:

```text
Many unit tests
      ↓
Some integration tests
      ↓
Fewer end-to-end tests
```

AI applications should follow the same principle while adding AI-specific evaluation.

---

# 4. Python Testing Tools

Python provides:

```text
unittest
```

as part of the standard library.

The Python ecosystem also commonly uses:

```text
pytest
```

`pytest` is widely used because it provides a concise testing style and a large ecosystem.

This chapter uses `pytest` examples.

---

# 5. Installing Pytest

Inside your virtual environment:

```bash
pip install pytest
```

Then run:

```bash
pytest
```

A common project structure is:

```text
project/
├── app/
│   ├── agents/
│   ├── rag/
│   ├── services/
│   └── tools/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── pyproject.toml
└── README.md
```

---

# 6. Your First Test

Suppose:

```python
# app/math_utils.py

def add(a, b):
    return a + b
```

Test:

```python
# tests/test_math_utils.py

from app.math_utils import add


def test_add():
    assert add(2, 3) == 5
```

Run:

```bash
pytest
```

If the assertion succeeds, the test passes.

---

# 7. Test Naming

Use descriptive test names.

Prefer:

```python
def test_retriever_returns_matching_documents():
    ...
```

over:

```python
def test_retriever():
    ...
```

Good test names communicate expected behavior.

A useful pattern is:

```text
test_<component>_<expected_behavior>
```

For example:

```python
def test_query_refiner_preserves_user_intent():
    ...
```

---

# 8. Arrange, Act, Assert

A common testing pattern is:

```text
Arrange
   ↓
Act
   ↓
Assert
```

Example:

```python
def test_chunk_document():

    # Arrange
    text = "A" * 100

    # Act
    chunks = chunk_document(text, size=50)

    # Assert
    assert len(chunks) == 2
```

This makes tests easier to understand.

---

# 9. Assertions

Pytest uses normal Python assertions.

Examples:

```python
assert result == expected
```

```python
assert result is not None
```

```python
assert len(results) == 5
```

```python
assert "document_id" in result
```

```python
assert response.status_code == 200
```

---

# 10. Testing Exceptions

Suppose:

```python
def divide(a, b):

    if b == 0:
        raise ValueError("Cannot divide by zero")

    return a / b
```

Test:

```python
import pytest


def test_divide_by_zero():

    with pytest.raises(ValueError):
        divide(10, 0)
```

You can also check the message:

```python
def test_divide_by_zero():

    with pytest.raises(
        ValueError,
        match="Cannot divide by zero"
    ):
        divide(10, 0)
```

---

# 11. Unit Tests

A unit test focuses on one small component.

Examples:

```text
Chunking function
Query transformation
Prompt builder
Metadata parser
Validation function
Token counter
Document cleaner
Tool input validator
```

Example:

```python
def test_clean_document_removes_extra_whitespace():
    text = "Hello    world"

    result = clean_document(text)

    assert result == "Hello world"
```

Unit tests should generally be:

* Fast
* Deterministic
* Independent
* Easy to debug

---

# 12. Testing RAG Components

Do not treat the entire RAG system as one test.

Break it into components.

```text
RAG
├── Document loading
├── Parsing
├── Cleaning
├── Chunking
├── Metadata
├── Embedding
├── Retrieval
├── Reranking
├── Context construction
└── Generation
```

For example:

```python
def test_chunking_preserves_document_metadata():
    ...
```

and:

```python
def test_retriever_filters_by_document_type():
    ...
```

This makes failures easier to isolate.

---

# 13. Testing Chunking

Suppose:

```python
def chunk_text(text, size):
    ...
```

You should test cases such as:

```text
Normal document
Empty document
Very short document
Very large document
Exact chunk boundary
Unicode text
Overlapping chunks
```

Example:

```python
def test_chunk_text_empty_input():
    assert chunk_text("", 100) == []
```

Another:

```python
def test_chunk_text_does_not_exceed_size():
    chunks = chunk_text(
        "some document text",
        100
    )

    for chunk in chunks:
        assert len(chunk) <= 100
```

The exact assertion depends on the chunking algorithm.

---

# 14. Testing Query Transformation

Suppose:

```python
def refine_query(query, context):
    ...
```

You can test deterministic behavior around the transformation layer.

Example:

```python
def test_query_refinement_preserves_original_intent():

    result = refine_query(
        "How does authentication work?",
        context="The application uses OAuth."
    )

    assert "authentication" in result.lower()
```

Do not assume a single exact wording if an LLM is involved.

Test the contract rather than the exact generated sentence.

---

# 15. Testing Retrieval

Retrieval can often be tested using a controlled test dataset.

Example:

```text
Document A → Python
Document B → Kubernetes
Document C → PostgreSQL
```

Query:

```text
Python async programming
```

Expected:

```text
Document A
```

Example:

```python
def test_retriever_returns_relevant_document():

    results = retriever.search(
        "Python async programming"
    )

    document_ids = [
        result.document_id
        for result in results
    ]

    assert "python-guide" in document_ids
```

This tests retrieval behavior without depending on a live production database.

---

# 16. Testing Hybrid Retrieval

If your system combines:

```text
Vector similarity
+
Keyword matching
```

test both signals.

For example:

```text
Query:
"Qdrant filtering"
```

Expected documents should contain relevant concepts even if exact keyword matching or semantic similarity contributes differently.

You can test:

```text
Vector-only retrieval
Keyword-only retrieval
Hybrid retrieval
```

and compare expected behavior on a fixed evaluation dataset.

---

# 17. Testing Reranking

Suppose initial retrieval returns:

```text
Document A
Document B
Document C
```

The reranker should change the ordering when relevance warrants it.

A deterministic reranking function can be tested exactly:

```python
def test_reranker_places_most_relevant_document_first():
    ...
```

If the reranker itself is model-based, prefer evaluation datasets and relevance judgments rather than requiring one exact output for every case.

---

# 18. Testing Embeddings

Embeddings are usually generated by an external model.

Do not write tests that depend on an external embedding API for every unit test.

Instead, separate:

```text
Embedding generation
```

from:

```text
Application logic
```

For example:

```python
embedding = embedding_client.embed(text)
```

can be mocked in unit tests.

Then test your application behavior using controlled vectors.

---

# 19. Testing LLM Calls

Avoid calling a real LLM provider in every unit test.

Problems include:

```text
Cost
Latency
Rate limits
Network failures
Non-deterministic output
Provider availability
```

Instead, mock the LLM boundary.

For example:

```python
class FakeLLM:

    def generate(self, prompt):
        return "Test response"
```

Then:

```python
def test_answer_service_returns_llm_response():

    llm = FakeLLM()

    service = AnswerService(llm)

    result = service.answer("question")

    assert result == "Test response"
```

---

# 20. Mocking

Mocking means replacing a real dependency with a controlled test double.

For example:

```text
Production:
Application → OpenAI API

Test:
Application → Fake LLM
```

This allows the test to focus on application behavior.

Common dependencies to mock:

```text
LLM APIs
Embedding APIs
Vector databases
External HTTP APIs
Email services
Payment APIs
Cloud services
Message queues
```

---

# 21. `unittest.mock`

Python provides mocking utilities:

```python
from unittest.mock import Mock
```

Example:

```python
llm = Mock()

llm.generate.return_value = "Test response"
```

Then:

```python
result = service.answer("question")

assert result == "Test response"
```

You can also verify that a method was called:

```python
llm.generate.assert_called_once()
```

---

# 22. Mocking Tool Calls

Suppose an agent uses:

```text
search_tool
database_tool
email_tool
```

During a unit test, you can replace them with mocks.

```python
search_tool = Mock()

search_tool.run.return_value = [
    "document 1",
    "document 2"
]
```

Then test the agent logic without calling a real search service.

This makes tests faster and deterministic.

---

# 23. Fixtures

Pytest fixtures provide reusable test setup.

Example:

```python
import pytest


@pytest.fixture
def sample_document():
    return {
        "id": "doc-1",
        "text": "Python is useful for AI engineering."
    }
```

Use it:

```python
def test_document_has_id(sample_document):

    assert sample_document["id"] == "doc-1"
```

Fixtures are useful for:

```text
Test data
Mock clients
Database setup
Application configuration
Temporary files
Test services
```

---

# 24. AI-Specific Fixtures

You might create fixtures for:

```text
Fake LLM
Fake embedding model
Test documents
Test chunks
Fake vector store
Agent state
Tool registry
```

Example:

```python
@pytest.fixture
def fake_llm():
    return FakeLLM()
```

Then many tests can reuse the same controlled dependency.

---

# 25. Parameterized Tests

The same behavior often needs testing with multiple inputs.

Instead of:

```python
def test_query_empty():
    ...

def test_query_whitespace():
    ...

def test_query_normal():
    ...
```

you can use parameterization.

```python
import pytest


@pytest.mark.parametrize(
    "query",
    [
        "",
        " ",
        "Python",
        "How does RAG work?"
    ]
)
def test_query_is_string(query):
    assert isinstance(query, str)
```

This reduces repetitive test code.

---

# 26. Testing Configuration

Configuration should also be tested.

For example:

```text
Missing API URL
Invalid timeout
Invalid environment
Missing required setting
```

If using Pydantic settings, test that invalid configurations fail early.

This is especially useful for production deployments.

---

# 27. Testing APIs

AI applications commonly expose REST APIs.

For example:

```text
POST /chat
POST /documents
GET /documents/{id}
POST /search
```

Tests should verify:

```text
Valid request
Invalid request
Authentication
Authorization
Validation
Status codes
Response structure
Error handling
```

For FastAPI applications, a test client can be used to test endpoints without manually running a separate server process.

---

# 28. API Test Example

Conceptually:

```python
def test_chat_endpoint(client):

    response = client.post(
        "/chat",
        json={
            "message": "What is RAG?"
        }
    )

    assert response.status_code == 200

    data = response.json()

    assert "answer" in data
```

The LLM itself can be replaced with a fake dependency.

---

# 29. Testing Authentication

For an AI API:

```text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Agent
```

Test:

```text
No token
Invalid token
Expired token
Valid token
Insufficient permissions
```

Example:

```python
def test_chat_requires_authentication(client):

    response = client.post(
        "/chat",
        json={"message": "Hello"}
    )

    assert response.status_code in {
        401,
        403
    }
```

The exact status depends on the authentication design.

---

# 30. Integration Tests

Integration tests verify that multiple real components work together.

Example:

```text
Application
 ↓
Retriever
 ↓
Vector database
```

or:

```text
API
 ↓
Agent
 ↓
Database
```

Unlike unit tests, integration tests may use real infrastructure.

For example:

```text
Application
 ↓
Test Qdrant instance
```

rather than:

```text
Application
 ↓
Mock Qdrant
```

---

# 31. When to Use Real Infrastructure

Use real dependencies when the interaction itself is what you're testing.

Examples:

```text
Database schema
Vector database filters
Serialization
Network behavior
Transaction handling
Authentication integration
```

But avoid turning every test into an integration test.

A useful strategy is:

```text
Unit tests
    ↓
Fast feedback

Integration tests
    ↓
Validate boundaries

E2E tests
    ↓
Validate critical user flows
```

---

# 32. Testing Vector Databases

A RAG system may depend on:

```text
Qdrant
Pinecone
Weaviate
Chroma
```

Integration tests can verify:

```text
Insert
Retrieve
Filter
Update
Delete
Metadata
Collection configuration
```

For example:

```python
def test_vector_search_returns_expected_document():
    ...
```

Use an isolated test collection or test instance.

Do not let tests modify production data.

---

# 33. End-to-End Tests

End-to-end tests validate the complete application flow.

Example:

```text
User
 ↓
API
 ↓
Authentication
 ↓
Agent
 ↓
Retriever
 ↓
Vector DB
 ↓
LLM
 ↓
Response
```

An E2E test might verify:

```python
def test_user_can_ask_question():
    ...
```

E2E tests are powerful but usually:

* Slower
* More expensive
* More fragile
* Harder to debug

Therefore, keep them focused on critical flows.

---

# 34. Testing Agents

Agent systems require multiple layers of testing.

Consider:

```text
Agent
├── Planning
├── State
├── Tool selection
├── Tool execution
├── Routing
└── Final response
```

Test deterministic logic directly.

For example:

```python
def test_agent_routes_database_question_to_database_tool():
    ...
```

For model-based routing, use controlled model outputs or evaluation datasets.

---

# 35. Testing LangGraph-Style Workflows

A graph-based agent may look like:

```text
START
  ↓
Planner
  ↓
Router
 ├── Search
 ├── Database
 └── Calculator
  ↓
Reviewer
  ↓
END
```

Test:

```text
Node behavior
State updates
Conditional routing
Error paths
Retries
Termination
```

For example:

```python
def test_router_selects_search_node():
    ...
```

and:

```python
def test_failed_tool_routes_to_error_handler():
    ...
```

The exact testing APIs depend on the framework version.

---

# 36. Testing Agent State

Suppose:

```python
state = {
    "query": "...",
    "documents": [],
    "answer": None
}
```

A node might modify:

```python
state["documents"]
```

Test the state transition:

```python
def test_retrieval_node_updates_documents():

    state = {
        "query": "What is RAG?",
        "documents": []
    }

    result = retrieval_node(state)

    assert len(result["documents"]) > 0
```

Testing state transitions is particularly important for complex agent graphs.

---

# 37. Testing Tool Failure

Agents need to handle tool failures.

Test:

```text
Tool succeeds
Tool times out
Tool returns invalid data
Tool raises exception
Tool returns empty result
```

Example:

```python
def test_agent_handles_tool_failure():

    tool = Mock()

    tool.run.side_effect = TimeoutError()

    result = agent.run(
        "Search for information"
    )

    assert result.status == "failed"
```

The expected behavior depends on your system design.

---

# 38. Testing Retries

Suppose an external service fails twice and succeeds on the third attempt.

A test can simulate:

```python
mock.call.side_effect = [
    TimeoutError(),
    TimeoutError(),
    "success"
]
```

Then verify:

```python
result = service.call()

assert result == "success"
assert mock.call_count == 3
```

This tests retry behavior without making real network requests.

---

# 39. Testing Timeouts

External services should have timeout behavior.

A mock can simulate:

```python
mock.call.side_effect = TimeoutError()
```

Then verify that your application:

```text
Does not hang indefinitely
Records the failure
Retries when appropriate
Returns a controlled error
```

---

# 40. Testing Async Code

AI applications frequently use:

```python
async def
```

and:

```python
await
```

Pytest can test asynchronous functions using appropriate async test support.

Conceptually:

```python
@pytest.mark.asyncio
async def test_async_retrieval():
    result = await retrieve_documents()
    assert result
```

The exact plugin/configuration depends on your test setup.

The important principle is to test async behavior without converting everything into synchronous wrappers.

---

# 41. Testing Concurrency

Concurrency introduces additional failure modes:

```text
Race conditions
Deadlocks
Resource exhaustion
Ordering issues
Rate limits
```

Tests should cover important concurrency behavior.

For example:

```text
100 concurrent retrieval requests
```

may reveal problems that:

```text
1 request
```

does not.

Do not confuse unit tests with load tests.

Concurrency correctness and performance/load testing are related but separate concerns.

---

# 42. Testing Background Jobs

Suppose:

```text
Upload
 ↓
Queue
 ↓
Worker
 ↓
Document processing
```

Test:

```text
Job created
Job consumed
Successful processing
Failure handling
Retry
Idempotency
Dead-letter behavior
```

A useful property is **idempotency**.

If the same job is processed twice, it should not accidentally create duplicate production data.

---

# 43. Testing Idempotency

Suppose:

```python
process_document("doc-123")
```

is called twice.

A good ingestion system might ensure:

```text
First call  → creates data
Second call → safely detects existing state
```

Test this behavior explicitly.

This is especially important in distributed AI pipelines where retries can happen.

---

# 44. Testing Prompt Construction

Prompt construction is often deterministic even when the LLM output is not.

Suppose:

```python
def build_prompt(question, context):
    ...
```

Test:

```python
def test_prompt_contains_context():

    prompt = build_prompt(
        question="What is RAG?",
        context="RAG combines retrieval and generation."
    )

    assert "RAG combines retrieval" in prompt
    assert "What is RAG?" in prompt
```

This is more reliable than testing the final LLM response.

---

# 45. Testing Structured LLM Output

Suppose your application expects:

```json
{
    "answer": "...",
    "confidence": 0.8
}
```

The important application contract is:

```text
Valid schema
Required fields
Correct types
Allowed ranges
Error handling
```

Test validation separately from the model.

For example:

```python
def test_llm_response_schema():

    result = AnswerResponse(
        answer="RAG retrieves context.",
        confidence=0.8
    )

    assert result.confidence <= 1
```

This is particularly useful when using Pydantic.

---

# 46. LLM Output Testing

For LLM-generated content, use multiple levels of testing.

### Level 1: Contract tests

Verify:

```text
Schema
Required fields
Types
```

### Level 2: Deterministic properties

Verify:

```text
Required topic is mentioned
Forbidden information is absent
Citations have expected structure
```

### Level 3: Evaluation

Measure:

```text
Correctness
Relevance
Groundedness
Faithfulness
Safety
```

Do not rely on exact string equality for open-ended generation.

---

# 47. RAG Evaluation vs Unit Testing

These are different.

Unit test:

```text
Did the retriever call the vector store correctly?
```

RAG evaluation:

```text
Did the retrieved context actually support the answer?
```

For example:

```text
Unit test
→ expected function behavior

RAG evaluation
→ expected AI system quality
```

Both are necessary.

---

# 48. Testing Hallucination-Related Behavior

You generally cannot guarantee that an LLM will never hallucinate using a simple unit test.

Instead, build an evaluation dataset:

```text
Question
Expected evidence
Expected answer characteristics
```

Then evaluate:

```text
Was the answer supported?
Did it introduce unsupported claims?
Did it cite relevant sources?
```

This becomes an evaluation problem rather than a traditional unit-testing problem.

---

# 49. Regression Testing for AI

AI systems can regress even when traditional tests pass.

For example:

```text
Prompt changed
 ↓
Retrieval unchanged
 ↓
LLM behavior changed
 ↓
Answer quality decreased
```

Therefore maintain evaluation datasets.

Example:

```text
evals/
├── rag_questions.jsonl
├── expected_sources.jsonl
└── agent_tasks.jsonl
```

Run them when making important changes to:

```text
Prompts
Models
Retrieval
Chunking
Reranking
Agent workflows
Tool definitions
```

---

# 50. Golden Datasets

A golden dataset contains representative cases with expected behavior.

Example:

```json
{
  "question": "How does authentication work?",
  "expected_document": "authentication.md"
}
```

A larger dataset might contain:

```text
Question
Expected documents
Expected answer properties
Difficulty
Category
```

Golden datasets are extremely useful for regression testing AI systems.

---

# 51. Evaluation Metrics

Depending on the system, useful metrics include:

```text
Retrieval recall
Precision
MRR
NDCG
Answer relevance
Faithfulness
Groundedness
Tool success rate
Task completion rate
Latency
Cost
```

The correct metrics depend on the product and task.

Do not optimize a metric without understanding what user behavior it represents.

---

# 52. Testing Safety and Security Boundaries

AI applications should also test security-sensitive behavior.

Examples:

```text
Prompt injection
Unauthorized tool access
Data leakage
Cross-user data access
Unsafe tool parameters
Permission bypass
```

For example:

```text
User A's documents
        ↓
User B's query
        ↓
Should NOT retrieve User A's documents
```

This is an application-level security requirement.

Test it explicitly.

---

# 53. Testing Multi-Tenant RAG

For a multi-tenant system:

```text
Tenant A
 ├── Document A1
 └── Document A2

Tenant B
 ├── Document B1
 └── Document B2
```

A query from Tenant A should only retrieve authorized data.

An integration test should verify:

```python
def test_tenant_cannot_retrieve_other_tenant_documents():
    ...
```

This is much more important than merely testing whether retrieval returns any documents.

---

# 54. Test Isolation

Tests should not depend on each other's execution order.

Bad:

```text
Test A creates database record
Test B assumes it exists
```

Better:

```text
Test A → creates its own data
Test B → creates its own data
```

Each test should be isolated.

This makes the suite more reliable.

---

# 55. Test Data

Use controlled test data.

For example:

```text
tests/
└── fixtures/
    ├── documents/
    ├── queries/
    ├── responses/
    └── datasets/
```

Avoid using real customer data.

Use:

```text
Synthetic data
Anonymized data
Minimal representative datasets
```

This reduces privacy and security risks.

---

# 56. Test Environment

Keep test environments separate from production.

For example:

```text
Development
     ↓
Test
     ↓
Staging
     ↓
Production
```

Never allow automated tests to accidentally:

```text
Delete production data
Send production emails
Modify production documents
Consume production API quotas
```

Use separate credentials and resources.

---

# 57. Test Coverage

Coverage measures how much code is exercised by tests.

A coverage tool can identify:

```text
Covered code
Uncovered code
```

But:

> High code coverage does not automatically mean high-quality tests.

For example:

```python
def test_function_runs():
    function()
```

may execute the code without verifying meaningful behavior.

Test **behavior**, not just execution.

---

# 58. What Should Be Tested Most?

Prioritize:

```text
Critical business logic
Security boundaries
Data transformations
Agent state transitions
External service failures
Database operations
RAG retrieval behavior
API contracts
Error handling
```

Less critical:

```text
Simple wrappers
Framework boilerplate
Trivial getters/setters
```

Testing effort should reflect risk.

---

# 59. Common Mistakes

## Mistake 1: Testing only happy paths

Test failures too.

```text
Success
Failure
Timeout
Invalid input
Empty result
Partial result
```

---

## Mistake 2: Calling real LLM APIs in every test

This creates:

```text
Cost
Latency
Flakiness
Rate limits
```

Mock external services for unit tests.

---

## Mistake 3: Exact-match testing LLM output

This is usually too brittle for open-ended generation.

Test contracts and properties instead.

---

## Mistake 4: No integration tests

Mocks cannot catch every integration problem.

Test important real boundaries.

---

## Mistake 5: No evaluation dataset

Traditional software tests cannot fully measure RAG or agent quality.

Maintain representative AI evaluation cases.

---

## Mistake 6: Testing production data

Never use production resources casually for automated tests.

---

## Mistake 7: Ignoring security tests

For AI systems, test:

```text
Authorization
Tenant isolation
Prompt injection defenses
Tool permissions
Data boundaries
```

---

## Mistake 8: Over-mocking

If everything is mocked, you may end up testing the mocks instead of the system.

Use mocks for unit boundaries and real infrastructure for important integration tests.

---

# Production Insight

A production AI testing strategy should look more like:

```text
                    AI Testing
                        │
       ┌────────────────┼─────────────────┐
       ↓                ↓                 ↓
Traditional Tests    AI Evaluation     Security Tests
       │                │                 │
       ↓                ↓                 ↓
Unit               RAG quality        Authorization
Integration        Agent quality      Data isolation
API                LLM behavior       Prompt injection
E2E                Regression         Tool permissions
```

For example, when changing a RAG system:

```text
Code tests
    ↓
Does the pipeline still work?

Integration tests
    ↓
Does Qdrant integration still work?

Evaluation dataset
    ↓
Did retrieval quality change?

LLM evaluation
    ↓
Did answer quality change?

Security tests
    ↓
Did tenant isolation remain intact?
```

This layered approach is much more reliable than trying to test the entire AI system with one giant test.

---

# Practical Project

Build a test suite for a small RAG application.

Use:

```text
Python
Pytest
Fake LLM
Fake embedding model
Test vector store
```

Structure:

```text
rag-project/
├── app/
│   ├── chunking.py
│   ├── embeddings.py
│   ├── retriever.py
│   ├── generator.py
│   └── pipeline.py
│
├── tests/
│   ├── unit/
│   │   ├── test_chunking.py
│   │   ├── test_embeddings.py
│   │   └── test_generator.py
│   │
│   ├── integration/
│   │   └── test_retriever.py
│   │
│   └── evals/
│       └── test_rag_quality.py
│
└── pyproject.toml
```

Implement tests for:

### Chunking

```text
Empty input
Short input
Large input
Chunk boundaries
Metadata preservation
```

### Retrieval

```text
Relevant document
No results
Metadata filtering
Tenant isolation
```

### LLM

```text
Successful response
Invalid structured response
Provider failure
Timeout
```

### Pipeline

```text
Successful request
Retrieval failure
LLM failure
Invalid input
```

### Evaluation

Create at least:

```text
10 representative questions
```

For each question, define:

```text
Expected source
Expected answer characteristics
```

Then run the evaluation whenever you modify:

```text
Chunking
Retrieval
Prompts
Models
Reranking
Agent workflow
```

---

# Interview Questions

## Beginner

1. What is software testing?
2. What is a unit test?
3. What is an integration test?
4. What is an end-to-end test?
5. What is pytest?
6. What is an assertion?
7. What is a fixture?
8. What is mocking?
9. What does `pytest.raises()` do?
10. What is test isolation?

## Intermediate

11. Explain the Arrange-Act-Assert pattern.
12. When should you use a mock?
13. When should you use a real dependency?
14. What is parameterized testing?
15. How do you test exceptions?
16. How do you test asynchronous Python code?
17. How do you test API endpoints?
18. What is test coverage?
19. Why doesn't high code coverage guarantee good tests?
20. How do you test retry behavior?

## AI Engineering

21. Why is testing an LLM application different from testing normal software?
22. Why should you avoid calling real LLM APIs in every unit test?
23. How would you test an LLM-based application?
24. How would you test a RAG pipeline?
25. How would you test retrieval quality?
26. What is the difference between RAG unit testing and RAG evaluation?
27. How would you test an agent's tool selection?
28. How would you test a multi-agent workflow?
29. How would you test agent state transitions?
30. How would you test tool failures?
31. How would you test LLM retries?
32. How would you test tenant isolation in a RAG system?
33. How would you test prompt construction?
34. Why is exact string matching usually inappropriate for open-ended LLM responses?
35. What is a golden dataset?
36. What metrics can be used to evaluate RAG?
37. How would you perform regression testing after changing an LLM?
38. How would you test prompt-injection defenses?
39. How would you test an MCP/tool-based agent?
40. Design a complete testing strategy for a production multi-agent RAG system.

---

# Key Takeaways

* Testing is essential for production AI engineering.
* Use unit tests for deterministic application logic.
* Use integration tests for important system boundaries.
* Use E2E tests for critical user workflows.
* Mock expensive or external dependencies in unit tests.
* Do not call real LLM APIs in every unit test.
* Test RAG components independently.
* Test retrieval with controlled datasets.
* Test agent state transitions and tool failures.
* Test API contracts and security boundaries.
* Test asynchronous and concurrent behavior where relevant.
* Use evaluation datasets for LLM, RAG, and agent quality.
* Do not rely on exact string matching for open-ended LLM output.
* Maintain regression datasets when changing prompts, models, retrieval, or agent workflows.
* Test tenant isolation and authorization explicitly.
* Code coverage is useful, but meaningful behavioral tests matter more.
* AI testing combines traditional software testing with AI evaluation.

---

# Summary

AI engineering requires two different but complementary approaches:

```text
Traditional Software Testing
            +
AI Evaluation
```

Traditional tests answer:

> **Does the software behave according to its contract?**

AI evaluation answers:

> **Does the AI system produce useful, correct, grounded, and safe behavior for representative tasks?**

A production AI system should therefore have layers such as:

```text
Unit Tests
     ↓
Integration Tests
     ↓
API Tests
     ↓
E2E Tests
     ↓
AI Evaluation
     ↓
Security Testing
```

The most important principle is:

> **Test deterministic behavior with assertions, and evaluate probabilistic AI behavior with representative datasets and measurable criteria.**

This distinction becomes increasingly important as you move from simple LLM applications toward RAG systems, tool-using agents, and multi-agent production systems.
