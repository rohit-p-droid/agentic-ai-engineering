# Logging in Python for AI Engineering

## Why does an AI Engineer need this?

An AI application can fail in many places:

```text
User request
    ↓
API
    ↓
Agent
    ↓
LLM
    ↓
Tool
    ↓
Database / Vector DB / External API
    ↓
Response
```

When something goes wrong, you need to answer questions such as:

* Which request failed?
* Which agent was running?
* Which tool failed?
* Which LLM call timed out?
* Which document was being processed?
* How long did retrieval take?
* How many retries happened?
* What exception occurred?
* Which service generated the error?

`print()` statements are useful while learning, but production systems need a proper logging system.

Python provides the built-in:

```python
logging
```

module for this purpose.

For AI engineers, logging is an important part of **production debugging, observability, incident investigation, and system reliability**.

---

# 1. What is Logging?

Logging means recording information about what an application is doing.

Example:

```text
2026-09-27 18:30:21 INFO Starting document ingestion
2026-09-27 18:30:22 INFO Generated embeddings
2026-09-27 18:30:23 INFO Stored 128 chunks
```

Logs provide a historical record of application behavior.

---

# 2. `print()` vs Logging

You can debug code with:

```python
print("Starting retrieval")
```

But production applications should generally use:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Starting retrieval")
```

Logging provides features that `print()` does not:

* Log levels
* Timestamps
* Logger names
* Exception information
* Filtering
* Formatting
* Output handlers
* File logging
* Structured logging integrations
* Environment-specific configuration

---

# 3. Creating a Logger

Basic example:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Application started")
```

A common production pattern is:

```python
logger = logging.getLogger(__name__)
```

inside each module.

For example:

```text
app/
├── agents/
│   └── agent.py
├── rag/
│   └── retriever.py
├── services/
│   └── llm.py
└── api/
    └── routes.py
```

Each module can create its own logger.

This gives you useful logger names such as:

```text
app.agents.agent
app.rag.retriever
app.services.llm
app.api.routes
```

---

# 4. Log Levels

Python's logging module provides standard log levels.

From lower to higher severity:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

---

# 5. DEBUG

`DEBUG` is used for detailed diagnostic information.

Example:

```python
logger.debug(
    "Retrieved %d chunks for query",
    len(chunks)
)
```

Useful during development and troubleshooting.

Examples:

```text
Query embedding generated
Retrieved 10 candidates
Reranking started
Agent selected search_tool
```

Debug logs are often disabled or reduced in production.

---

# 6. INFO

`INFO` describes normal application behavior.

Example:

```python
logger.info(
    "Document ingestion started: %s",
    document_id
)
```

Examples:

```text
Application started
Document uploaded
Embedding generation completed
Agent workflow completed
Database connection established
```

This is usually the main level for normal operational events.

---

# 7. WARNING

`WARNING` indicates something unexpected or potentially problematic, but the application can continue.

Example:

```python
if len(chunks) < 3:
    logger.warning(
        "Only %d chunks retrieved",
        len(chunks)
    )
```

Other examples:

```text
Retrying API request
Cache miss
Fallback model selected
Low retrieval score
Approaching rate limit
Configuration value missing but default used
```

---

# 8. ERROR

`ERROR` indicates that an operation failed.

Example:

```python
try:
    response = llm.generate(prompt)
except Exception:
    logger.exception("LLM generation failed")
```

Examples:

```text
Vector database request failed
Document processing failed
Tool execution failed
External API request failed
Database operation failed
```

---

# 9. CRITICAL

`CRITICAL` represents a severe failure that may prevent the application from operating correctly.

Example:

```python
logger.critical(
    "Required configuration is missing"
)
```

Examples might include:

```text
Application cannot initialize
Required database unavailable
Critical infrastructure failure
```

Do not use `CRITICAL` for ordinary errors.

---

# 10. Basic Logging Example

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(__name__)

logger.debug("Debug message")
logger.info("Application started")
logger.warning("Low retrieval score")
logger.error("Tool execution failed")
logger.critical("Application cannot start")
```

With an `INFO` level, debug messages may not be emitted.

---

# 11. Logging Variables

Avoid building strings manually.

Instead of:

```python
logger.info(
    f"Processing document {document_id}"
)
```

prefer:

```python
logger.info(
    "Processing document %s",
    document_id
)
```

The logging module can defer formatting until the message actually needs to be emitted.

For more complex structured logging, use appropriate structured logging tools or adapters.

---

# 12. Logging Exceptions

One of the most useful logging features is:

```python
logger.exception(...)
```

Example:

```python
try:
    result = retrieve_documents(query)
except Exception:
    logger.exception(
        "Document retrieval failed"
    )
```

This records the exception and traceback.

A traceback is extremely valuable when debugging production failures.

---

# 13. `logger.error()` vs `logger.exception()`

Consider:

```python
try:
    result = process()
except Exception as error:
    logger.error(
        "Processing failed: %s",
        error
    )
```

This records the error message.

But:

```python
try:
    result = process()
except Exception:
    logger.exception(
        "Processing failed"
    )
```

also records the traceback.

For failures where the traceback matters, `logger.exception()` is often more useful.

`logger.exception()` should normally be called while handling an active exception.

---

# 14. Logging Configuration

For a small application:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format=(
        "%(asctime)s "
        "%(levelname)s "
        "%(name)s "
        "%(message)s"
    )
)
```

Example output:

```text
2026-09-27 18:30:21 INFO app.rag.retriever Retrieval started
```

The format can include:

```text
%(asctime)s
%(levelname)s
%(name)s
%(message)s
%(filename)s
%(lineno)d
```

---

# 15. Useful Log Fields

For AI systems, useful fields can include:

```text
timestamp
level
service
environment
request_id
trace_id
user_id
document_id
agent_name
tool_name
model
duration_ms
status
error_type
```

Be selective.

More fields do not automatically mean better observability.

---

# 16. Logging in a RAG Pipeline

Consider:

```text
User query
   ↓
Query transformation
   ↓
Embedding
   ↓
Vector search
   ↓
Keyword search
   ↓
Reranking
   ↓
LLM
   ↓
Answer
```

Useful logs could be:

```python
logger.info(
    "RAG request started: request_id=%s",
    request_id
)

logger.debug(
    "Query transformation completed"
)

logger.info(
    "Retrieved %d candidates",
    len(results)
)

logger.info(
    "Reranking completed in %.2f ms",
    duration_ms
)

logger.info(
    "Answer generation completed"
)
```

This allows you to understand where time and failures occur.

---

# 17. Logging an Agent Workflow

Suppose an agent executes:

```text
User
 ↓
Planner
 ↓
Search Tool
 ↓
Database Tool
 ↓
Summarizer
 ↓
Final Response
```

Useful events include:

```python
logger.info(
    "Agent workflow started: request_id=%s",
    request_id
)

logger.info(
    "Agent selected tool: %s",
    tool_name
)

logger.info(
    "Tool execution completed: tool=%s",
    tool_name
)

logger.info(
    "Agent workflow completed"
)
```

Avoid logging every internal detail by default.

---

# 18. Logging Tool Calls

Agentic systems often execute external tools.

Example:

```python
def execute_tool(tool_name, arguments):

    logger.info(
        "Executing tool: %s",
        tool_name
    )

    try:
        result = tool_registry[
            tool_name
        ](**arguments)

        logger.info(
            "Tool completed: %s",
            tool_name
        )

        return result

    except Exception:
        logger.exception(
            "Tool failed: %s",
            tool_name
        )
        raise
```

This gives you a useful lifecycle:

```text
Tool started
     ↓
Tool completed
```

or:

```text
Tool started
     ↓
Tool failed
```

---

# 19. Request IDs

A request ID helps connect logs belonging to one request.

Example:

```text
request_id=abc123
```

Then logs might look like:

```text
INFO request_id=abc123 API request received
INFO request_id=abc123 Retrieval started
INFO request_id=abc123 8 chunks retrieved
INFO request_id=abc123 LLM generation started
INFO request_id=abc123 Request completed
```

This is extremely useful in distributed systems.

---

# 20. Correlation IDs

In a multi-service architecture:

```text
API
 ↓
Agent Service
 ↓
RAG Service
 ↓
Vector DB
 ↓
LLM Service
```

A request may travel through several services.

A correlation or trace ID allows you to associate related operations.

Conceptually:

```text
trace_id=7f31...

API
 └── Agent
      ├── Retrieval
      ├── Tool
      └── LLM
```

This becomes especially important when debugging distributed agentic systems.

---

# 21. Logging Latency

Latency is critical for AI applications.

Example:

```python
import time

start = time.perf_counter()

result = retrieve_documents(query)

duration = (
    time.perf_counter() - start
) * 1000

logger.info(
    "Retrieval completed in %.2f ms",
    duration
)
```

Track latency for important stages:

```text
Embedding
Retrieval
Reranking
LLM generation
Tool execution
Database queries
External APIs
```

---

# 22. A Reusable Timing Decorator

You learned decorators earlier.

Logging can be combined with them.

```python
import logging
import time
from functools import wraps

logger = logging.getLogger(__name__)


def log_execution_time(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        start = time.perf_counter()

        try:
            return func(*args, **kwargs)

        finally:
            duration = (
                time.perf_counter() - start
            )

            logger.info(
                "%s completed in %.3fs",
                func.__name__,
                duration
            )

    return wrapper
```

Use:

```python
@log_execution_time
def retrieve_documents(query):
    ...
```

This is useful for development and basic instrumentation.

For production observability, dedicated metrics/tracing systems are often more appropriate than relying only on logs.

---

# 23. Structured Logging

Traditional logs look like:

```text
Retrieval completed in 120ms
```

Structured logs represent fields separately.

Conceptually:

```json
{
  "level": "INFO",
  "event": "retrieval_completed",
  "duration_ms": 120,
  "result_count": 8,
  "request_id": "abc123"
}
```

Structured logging is valuable because machines can query fields directly.

You can search for:

```text
duration_ms > 1000
```

or:

```text
event = "tool_failed"
```

instead of parsing arbitrary strings.

---

# 24. Why Structured Logs Matter for AI

AI systems generate a large amount of dynamic information.

For example:

```text
model
tool
agent
retrieval_strategy
document_count
chunk_count
latency
token_usage
status
```

Structured logs make these fields easier to aggregate and analyze.

For example:

```text
agent=research_agent
tool=web_search
duration_ms=840
status=success
```

This is more useful operationally than:

```text
Research agent web search finished successfully in 840 milliseconds
```

---

# 25. Logging Token Usage

LLM applications may track token usage.

A log event could contain:

```text
model
input_tokens
output_tokens
total_tokens
duration_ms
```

For example:

```python
logger.info(
    "LLM call completed: model=%s input_tokens=%d output_tokens=%d",
    model_name,
    input_tokens,
    output_tokens
)
```

Only log usage information that is actually available from the provider/client.

Do not invent token counts.

---

# 26. Logging Model Information

In multi-model systems, record which model handled a request.

Example:

```python
logger.info(
    "LLM request: model=%s",
    model_name
)
```

This can help investigate:

```text
Which model handled this request?
Did latency change after a model change?
Did a particular model produce more errors?
```

---

# 27. Logging Retries

External AI services can fail temporarily.

Suppose an API call is retried.

Log the retry:

```python
logger.warning(
    "Retrying LLM request: attempt=%d",
    attempt
)
```

Useful fields:

```text
attempt
max_attempts
error_type
backoff
service
model
```

Avoid logging sensitive request content just to diagnose a retry.

---

# 28. Logging Rate Limits

AI APIs may impose rate limits.

A useful warning could be:

```python
logger.warning(
    "LLM rate limit encountered; retrying"
)
```

If the provider exposes useful rate-limit metadata, it can be recorded where appropriate.

This can help distinguish:

```text
Application bug
```

from:

```text
Provider rate limiting
```

---

# 29. Logging Errors in Background Jobs

AI applications often process documents asynchronously.

Example:

```text
Upload
 ↓
Queue
 ↓
Worker
 ↓
Extraction
 ↓
Chunking
 ↓
Embedding
 ↓
Vector DB
```

If a worker fails, logs should identify the job.

Example:

```python
logger.exception(
    "Document processing failed: document_id=%s",
    document_id
)
```

Useful identifiers:

```text
job_id
document_id
tenant_id
attempt
worker
pipeline_stage
```

Only include identifiers that are safe and appropriate to log.

---

# 30. Logging Sensitive Data

This is one of the most important topics for AI applications.

Avoid blindly logging:

```text
Passwords
API keys
Access tokens
Authorization headers
Credit card information
Personal information
Private documents
Sensitive prompts
Confidential model responses
Secrets
```

Bad:

```python
logger.info(
    "OpenAI key: %s",
    api_key
)
```

Never do this.

Also avoid:

```python
logger.info(
    "User prompt: %s",
    prompt
)
```

unless you have explicitly designed for the privacy, security, retention, and access implications.

---

# 31. Redaction

If some sensitive value must be logged for debugging, redact it.

Example:

```python
def redact_token(token: str) -> str:
    if len(token) <= 8:
        return "***"

    return (
        token[:4]
        + "..."
        + token[-4:]
    )
```

Then:

```python
logger.debug(
    "Token: %s",
    redact_token(token)
)
```

Even better: avoid logging secrets entirely.

---

# 32. Prompt and Response Logging

LLM applications often tempt developers to log:

```text
Prompt
↓
LLM
↓
Response
```

This can be useful for debugging, but creates privacy and security risks.

Before logging prompts or responses, consider:

```text
Is the data sensitive?
Who can access logs?
How long are logs retained?
Is the data customer-owned?
Do policies permit logging?
Can the information be redacted?
```

A safer default is to log metadata rather than the full content:

```text
request_id
model
latency
token usage
status
retrieval count
```

---

# 33. Logging Document Processing

For a RAG ingestion service:

```python
logger.info(
    "Document ingestion started: document_id=%s",
    document_id
)
```

Then:

```python
logger.info(
    "Document extracted: document_id=%s pages=%d",
    document_id,
    page_count
)
```

Then:

```python
logger.info(
    "Document chunked: document_id=%s chunks=%d",
    document_id,
    chunk_count
)
```

Then:

```python
logger.info(
    "Embeddings generated: document_id=%s chunks=%d",
    document_id,
    chunk_count
)
```

This gives you a clear pipeline history without logging the document itself.

---

# 34. Logging Configuration with Environment Variables

Different environments may use different log levels.

For example:

```text
Development → DEBUG
Staging     → INFO
Production  → INFO/WARNING
```

Conceptually:

```python
import os
import logging

level = os.getenv(
    "LOG_LEVEL",
    "INFO"
).upper()

logging.basicConfig(
    level=getattr(
        logging,
        level,
        logging.INFO
    )
)
```

This allows deployment configuration to control verbosity.

---

# 35. Logging to Files

You can configure a file handler:

```python
import logging

logging.basicConfig(
    filename="app.log",
    level=logging.INFO
)
```

However, production deployments commonly send logs to centralized logging infrastructure rather than relying on local application files.

For containerized applications, writing logs to standard output/error and letting the platform collect them is often a common architecture.

The exact approach depends on the deployment environment.

---

# 36. Logging to Multiple Destinations

Python logging supports handlers.

Conceptually:

```text
Logger
  ├── Console Handler
  ├── File Handler
  └── External Handler
```

Different handlers can have different levels and formatting.

For example:

```text
Console → INFO
File    → DEBUG
Alerting → ERROR
```

This gives you more control than a single output destination.

---

# 37. Logger Hierarchy

Python loggers are hierarchical.

For example:

```text
app
├── api
├── agents
├── rag
└── services
```

You can have:

```python
logging.getLogger("app")
```

and:

```python
logging.getLogger("app.rag")
```

The hierarchy allows logging configuration to be organized by application area.

---

# 38. Avoid Configuring Logging in Every Module

Avoid doing this everywhere:

```python
logging.basicConfig(...)
```

Instead, configure logging at the application entry point.

For example:

```text
app/
├── main.py
├── api/
├── agents/
├── rag/
└── services/
```

`main.py` can configure the logging system.

Other modules should generally do:

```python
logger = logging.getLogger(__name__)
```

This keeps configuration centralized.

---

# 39. Logging in FastAPI

A request lifecycle might look like:

```text
Request
 ↓
Authentication
 ↓
API handler
 ↓
Agent
 ↓
RAG
 ↓
LLM
 ↓
Response
```

Useful API logs include:

```text
request received
request completed
request failed
status code
duration
request ID
```

Avoid logging sensitive request bodies by default.

Framework/server logs and application logs can complement each other.

---

# 40. Logging in Multi-Agent Systems

Multi-agent systems create an additional observability challenge.

Consider:

```text
Supervisor
   ↓
Research Agent
   ↓
Search Tool
   ↓
Research Agent
   ↓
Writer Agent
   ↓
Reviewer Agent
```

Logs should make the workflow traceable.

Useful metadata:

```text
trace_id
request_id
agent_name
node_name
tool_name
step
status
duration
```

For example:

```text
agent=researcher step=search status=started
agent=researcher step=search status=completed duration_ms=720
agent=writer step=generation status=started
```

This makes agent execution much easier to debug.

---

# 41. Logging Agent State

Be careful when logging agent state.

Agent state may contain:

* User information
* Retrieved documents
* Tool arguments
* Conversation history
* Internal instructions
* Credentials
* Sensitive business information

Instead of logging the entire state:

```python
logger.debug(
    "Agent state: %s",
    state
)
```

prefer selected metadata:

```python
logger.debug(
    "Agent state updated: step=%s tool_count=%d",
    step,
    len(tool_calls)
)
```

Log what you need, not everything available.

---

# 42. Logging MCP Tool Calls

MCP-based systems can involve:

```text
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tool
```

Useful metadata could include:

```text
server
tool
request_id
duration
status
error_type
```

For example:

```python
logger.info(
    "MCP tool call completed: server=%s tool=%s",
    server_name,
    tool_name
)
```

Avoid logging sensitive tool arguments unless there is a deliberate and secure reason to do so.

---

# 43. Logs vs Metrics vs Traces

Logging is only one part of observability.

### Logs

Answer:

> What happened?

Example:

```text
Tool execution failed
```

### Metrics

Answer:

> How often/how much?

Example:

```text
LLM error rate = 2.4%
```

### Traces

Answer:

> How did this request move through the system?

Example:

```text
API
 └── Agent
      ├── Retrieval
      ├── Tool
      └── LLM
```

Production AI systems often need all three.

---

# 44. Logging and Observability

A mature AI system might have:

```text
                 Observability
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Logs         Metrics        Traces
        │             │             │
     Events        Counts         Requests
     Errors        Latency        Spans
     Metadata      Rates          Dependencies
```

Logs provide detailed event information.

Metrics provide aggregate system behavior.

Traces provide request-level execution flow.

---

# 45. Common Logging Mistakes

## Mistake 1: Using `print()` everywhere

This makes production debugging difficult.

---

## Mistake 2: Logging everything

More logs can make systems harder to understand.

Good logs should answer useful operational questions.

---

## Mistake 3: Logging secrets

Never log:

```text
API keys
Passwords
Tokens
Private credentials
```

---

## Mistake 4: Logging complete prompts and responses by default

This can expose sensitive data.

---

## Mistake 5: No request ID

Distributed systems become much harder to debug without correlation identifiers.

---

## Mistake 6: No latency information

An AI application can be functionally correct but operationally slow.

Measure important stages.

---

## Mistake 7: Catching exceptions and only logging them

Avoid:

```python
try:
    process()
except Exception:
    logger.exception("Failed")
```

if you then silently continue when the operation should fail.

Logging does not replace proper error handling.

---

## Mistake 8: Logging sensitive identifiers carelessly

Even identifiers can become sensitive depending on the system.

Use the minimum information required.

---

## Mistake 9: Inconsistent log messages

Prefer consistent event names and fields.

For example:

```text
retrieval_started
retrieval_completed
retrieval_failed
```

rather than many unrelated message formats.

---

# Production Insight

Logs should be designed around the questions you will ask during an incident.

Imagine a production RAG request takes 12 seconds.

You should be able to investigate:

```text
Request
  ↓
Query transformation       200 ms
  ↓
Embedding                   300 ms
  ↓
Vector retrieval            80 ms
  ↓
Reranking                   450 ms
  ↓
Tool call                  900 ms
  ↓
LLM generation            9,800 ms
```

Without useful logs and telemetry, you may only see:

```text
Request failed
```

With useful observability, you can identify where the time or failure occurred.

For agentic AI systems, this becomes even more important because execution paths may be dynamic.

A good production logging strategy should therefore prioritize:

```text
Context
+
Lifecycle events
+
Failures
+
Latency
+
Correlation
+
Security
```

rather than simply generating more log lines.

---

# Practical Exercise

Build a logging system for a small RAG pipeline.

Create:

```text
rag/
├── ingestion.py
├── retrieval.py
├── generation.py
└── main.py
```

Configure logging in `main.py`.

Each module should create its own logger:

```python
logger = logging.getLogger(__name__)
```

Log these events:

### Ingestion

```text
document_ingestion_started
document_ingestion_completed
document_ingestion_failed
```

### Retrieval

```text
retrieval_started
retrieval_completed
retrieval_failed
```

### Generation

```text
llm_generation_started
llm_generation_completed
llm_generation_failed
```

Include:

```text
request_id
document_id
duration
status
```

Do not log:

```text
API keys
full user prompts
full documents
full LLM responses
```

Then intentionally introduce an error and verify that the traceback is captured.

---

# Interview Questions

## Beginner

1. What is logging?
2. Why is logging preferred over `print()` in production?
3. What are the standard Python log levels?
4. What is the difference between `INFO` and `DEBUG`?
5. What is the difference between `WARNING` and `ERROR`?
6. How do you create a logger?
7. What does `logging.getLogger(__name__)` do?
8. What does `logging.basicConfig()` do?
9. How do you log an exception?
10. What is the difference between `logger.error()` and `logger.exception()`?

## Intermediate

11. What is a logging handler?
12. What is a logging formatter?
13. How does logger hierarchy work?
14. Why should logging configuration usually be centralized?
15. What is structured logging?
16. Why are request IDs useful?
17. What is a correlation ID?
18. How would you log a background job?
19. How would you measure latency through logs?
20. How would you configure different log levels for development and production?

## AI Engineering

21. What should you log for an LLM request?
22. What should you log for a RAG pipeline?
23. How would you debug a slow RAG request?
24. How would you log a multi-agent workflow?
25. What information should be recorded for tool execution?
26. How would you log retries from an LLM provider?
27. Why should full prompts and responses not automatically be logged?
28. How would you correlate logs across multiple AI microservices?
29. What is the difference between logs, metrics, and traces?
30. Design a logging strategy for a production multi-agent system.

---

# Key Takeaways

* Use Python's `logging` module instead of relying on `print()` in production applications.
* Use `logging.getLogger(__name__)` in individual modules.
* Understand `DEBUG`, `INFO`, `WARNING`, `ERROR`, and `CRITICAL`.
* Use `logger.exception()` when you need the traceback while handling an exception.
* Log important lifecycle events, failures, and latency.
* Request IDs and correlation IDs are essential for distributed AI systems.
* Structured logs make AI application telemetry easier to search and analyze.
* RAG systems should log pipeline stages without exposing sensitive document contents.
* Agentic systems should record useful execution metadata such as agent, tool, step, status, and duration.
* Never log API keys, passwords, tokens, or other secrets.
* Be careful with prompts, responses, documents, and agent state because they may contain sensitive information.
* Logs are only one part of observability; metrics and traces provide complementary information.
* Good logging is not about logging everything. It is about recording the information needed to understand system behavior.

---

# Summary

Production AI systems are distributed, asynchronous, and often nondeterministic.

A useful logging strategy makes their behavior observable.

The core pattern is:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Operation started")

try:
    result = perform_operation()

except Exception:
    logger.exception("Operation failed")
    raise

logger.info("Operation completed")
```

For AI engineering, extend this with useful metadata:

```text
request_id
trace_id
agent
tool
model
pipeline_stage
duration
status
error_type
```

while protecting:

```text
secrets
credentials
private documents
sensitive prompts
sensitive responses
```

The goal of logging is not to produce more output.

The goal is to make it possible to answer:

> **What happened, where did it happen, why did it happen, and how long did it take?**
