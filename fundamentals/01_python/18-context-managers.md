# Context Managers in Python for AI Engineering

## Why does an AI Engineer need this?

AI applications frequently work with resources that must be properly acquired and released.

Examples include:

* Files
* Database connections
* HTTP connections
* Locks
* Temporary directories
* Network sessions
* Model resources
* Tracing spans
* Transactions
* Streaming connections
* Cloud resources

A common pattern is:

```text id="a6k3pw"
Acquire resource
      ↓
Use resource
      ↓
Something goes wrong?
      ↓
Release resource
```

Python's **context managers** provide a clean and reliable way to manage this lifecycle.

You have already seen the most common example:

```python id="1o6j5n"
with open("document.txt") as file:
    content = file.read()
```

The context manager ensures that the file is properly closed when the block finishes, including when an exception occurs.

For AI engineering, context managers are especially useful for making resource handling **safe, predictable, and explicit**.

---

# 1. What is a Context Manager?

A context manager controls what happens when entering and leaving a block of code.

The syntax is:

```python id="5w8m3j"
with resource:
    # use resource
```

Conceptually:

```text id="n2q7ya"
Enter context
     ↓
Execute block
     ↓
Exit context
     ↓
Cleanup
```

The cleanup happens even if an exception occurs inside the block.

---

# 2. The `with` Statement

Basic example:

```python id="g7d4h2"
with open("document.txt", "r") as file:
    content = file.read()
```

When the block exits:

```text id="2h8m5f"
open file
   ↓
read file
   ↓
block finishes
   ↓
file closes
```

You don't need to explicitly write:

```python id="e9c1rz"
file.close()
```

---

# 3. Why Context Managers Matter

Without a context manager:

```python id="b2v6hk"
file = open("document.txt")

try:
    content = file.read()
finally:
    file.close()
```

With a context manager:

```python id="m9x4pa"
with open("document.txt") as file:
    content = file.read()
```

The second version is shorter and makes the resource lifecycle obvious.

More importantly, it protects cleanup when exceptions occur.

---

# 4. Exception Safety

Consider:

```python id="j6s8vk"
with open("document.txt") as file:
    process_document(file)
```

If:

```python id="k2z1qm"
process_document(file)
```

raises an exception, Python still exits the context manager.

Conceptually:

```text id="p8s4dz"
Enter
  ↓
Process document
  ↓
Exception
  ↓
Exit context
  ↓
Close file
  ↓
Exception propagates
```

This is one of the most important benefits of context managers.

---

# 5. The Context Manager Protocol

A class can act as a context manager by implementing:

```python id="r3w7xn"
__enter__()
__exit__()
```

Example:

```python id="x9m2qb"
class Resource:

    def __enter__(self):
        print("Resource acquired")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Resource released")
```

Use it:

```python id="h5v8kd"
with Resource() as resource:
    print("Using resource")
```

Output:

```text id="q1s7az"
Resource acquired
Using resource
Resource released
```

---

# 6. Understanding `__enter__`

`__enter__()` runs when entering the `with` block.

```python id="4b9z5w"
def __enter__(self):
    ...
```

Its return value is assigned to the variable after `as`.

```python id="m7k1fd"
with Resource() as resource:
    ...
```

is conceptually related to:

```python id="s8c4qp"
resource = resource_manager.__enter__()
```

The returned object does not have to be the same object as the context manager itself.

---

# 7. Understanding `__exit__`

The `__exit__()` method runs when leaving the context.

It receives:

```python id="9x6t2v"
exc_type
exc_value
traceback
```

Conceptually:

```python id="h2n8kj"
def __exit__(self, exc_type, exc_value, traceback):
    ...
```

If no exception occurred:

```text id="v6x0qr"
exc_type  = None
exc_value = None
traceback = None
```

If an exception occurred, these contain information about it.

---

# 8. Suppressing Exceptions

A context manager can suppress an exception by returning:

```python id="g5j3sw"
True
```

from `__exit__`.

Example:

```python id="k9d2pf"
class IgnoreErrors:

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        return True
```

Then:

```python id="n4m8xq"
with IgnoreErrors():
    raise ValueError("Something failed")

print("Continues")
```

This suppresses the exception.

### Be careful

Silently suppressing exceptions can hide serious failures.

In production AI systems, exceptions should generally be suppressed only intentionally and for well-understood cases.

---

# 9. Context Managers and Files

File handling is the most common example.

```python id="c7q2ma"
with open(
    "documents/article.md",
    "r",
    encoding="utf-8"
) as file:
    text = file.read()
```

This is especially important in AI pipelines that process many files.

Instead of leaving file handles open:

```text id="q0m3kr"
Document 1 → open
Document 2 → open
Document 3 → open
...
```

each context manages its own lifecycle.

---

# 10. Context Managers and Database Connections

A database operation may conceptually look like:

```python id="g8v2ld"
with database_connection() as connection:
    result = connection.execute(query)
```

The context manager can handle:

```text id="k5r9xa"
Acquire connection
      ↓
Execute query
      ↓
Commit / rollback
      ↓
Release connection
```

The exact behavior depends on the database library.

Always follow the driver's transaction and connection-management semantics.

---

# 11. Transactions

Context managers are useful for transaction boundaries.

Conceptually:

```python id="w3s7kp"
with database.transaction():
    create_document()
    create_chunks()
    update_metadata()
```

If everything succeeds:

```text id="t1k4bm"
Commit
```

If something fails:

```text id="f7d2qa"
Rollback
```

This is an important pattern when multiple database operations must succeed or fail together.

The actual transaction behavior depends on the database library.

---

# 12. Context Managers and Locks

Python locks can be used as context managers.

```python id="n8q5yc"
import threading

lock = threading.Lock()

with lock:
    update_shared_state()
```

This is safer and cleaner than manually writing:

```python id="2k9p6s"
lock.acquire()

try:
    update_shared_state()
finally:
    lock.release()
```

The context manager guarantees that the lock is released when the block exits.

---

# 13. Context Managers for AI Pipelines

Suppose an AI pipeline needs to create a temporary workspace.

```text id="m2v8fk"
Start ingestion
     ↓
Create temporary directory
     ↓
Extract files
     ↓
Process documents
     ↓
Create embeddings
     ↓
Cleanup temporary files
```

A context manager can manage the temporary workspace lifecycle.

Python provides:

```python id="v9h4st"
tempfile.TemporaryDirectory
```

Example:

```python id="x2j6qa"
from tempfile import TemporaryDirectory


with TemporaryDirectory() as directory:
    process_documents(directory)
```

When the block exits, the temporary directory is cleaned up.

---

# 14. `contextlib`

Python provides the:

```python id="p6d8xy"
contextlib
```

module for creating and working with context managers.

Useful utilities include:

```text id="k3f7nm"
contextmanager
asynccontextmanager
nullcontext
suppress
closing
ExitStack
AsyncExitStack
```

These are especially useful for production Python applications.

---

# 15. `@contextmanager`

Instead of creating a class with `__enter__` and `__exit__`, you can use:

```python id="w8m2rq"
contextlib.contextmanager
```

Example:

```python id="u7p5dz"
from contextlib import contextmanager


@contextmanager
def resource():
    print("Acquire")

    try:
        yield "resource"
    finally:
        print("Release")
```

Use it:

```python id="s2x8kv"
with resource() as value:
    print(value)
```

Output:

```text id="f5m1ra"
Acquire
resource
Release
```

---

# 16. Why `try/finally` Matters

The important structure is:

```python id="n5k2cw"
@contextmanager
def resource():

    acquire()

    try:
        yield
    finally:
        release()
```

The `finally` block ensures cleanup happens even if the code inside the `with` block raises an exception.

This is the central pattern:

```text id="m4q8sd"
Acquire
  ↓
try
  ↓
yield
  ↓
finally
  ↓
Release
```

---

# 17. Example: Timing Context Manager

A context manager can measure an entire block of code.

```python id="c8n4yb"
import time
from contextlib import contextmanager


@contextmanager
def timer(name):

    start = time.perf_counter()

    try:
        yield

    finally:
        duration = time.perf_counter() - start

        print(
            f"{name} took {duration:.3f}s"
        )
```

Use it:

```python id="r7m3vq"
with timer("RAG retrieval"):
    retrieve_documents(query)
```

This is useful when you want to measure a **block** rather than a single function.

---

# 18. Decorator vs Context Manager

You learned decorators in the previous chapter.

They can solve related problems.

### Decorator

Wraps a function:

```python id="j3c9ws"
@measure_time
def retrieve():
    ...
```

### Context manager

Wraps a block:

```python id="x5n7qa"
with timer("retrieval"):
    retrieve()
    rerank()
```

Use a decorator when:

```text id="8f2j4k"
Behavior belongs to a function
```

Use a context manager when:

```text id="4v9n1p"
Behavior belongs to a block/resource lifecycle
```

---

# 19. Context Managers for Tracing

Observability systems often represent operations as spans.

Conceptually:

```python id="m7c5x2"
with tracer.start_as_current_span("retrieval"):
    retrieve_documents(query)
```

Lifecycle:

```text id="z6q3bw"
Start span
   ↓
Retrieval
   ↓
Finish span
```

This is a natural context-manager use case because the span has a clear start and end.

---

# 20. Context Managers for LLM Operations

You might want to track a complete LLM operation:

```python id="b8k1qs"
with track_llm_call("answer_generation"):
    response = llm.generate(prompt)
```

The context manager could record:

* Start time
* End time
* Success/failure
* Request ID
* Token usage
* Model name

Be careful about logging sensitive prompts and responses.

---

# 21. Async Context Managers

Modern AI applications frequently use asynchronous APIs.

Python supports asynchronous context managers:

```python id="q9v4kd"
async with resource:
    ...
```

A class can implement:

```python id="n3h7xc"
__aenter__()
__aexit__()
```

Example:

```python id="v6m2ps"
class AsyncResource:

    async def __aenter__(self):
        print("Acquire")
        return self

    async def __aexit__(
        self,
        exc_type,
        exc_value,
        traceback
    ):
        print("Release")
```

Use it:

```python id="w5k8zr"
async with AsyncResource():
    await perform_operation()
```

---

# 22. `@asynccontextmanager`

`contextlib` also provides:

```python id="c1r7xm"
asynccontextmanager
```

Example:

```python id="h8m3vq"
from contextlib import asynccontextmanager


@asynccontextmanager
async def resource():

    await acquire_resource()

    try:
        yield
    finally:
        await release_resource()
```

Use:

```python id="q4s9jb"
async with resource():
    await process()
```

This is useful for asynchronous resource lifecycle management.

---

# 23. FastAPI Lifespan

FastAPI provides application lifecycle management using an async context manager pattern.

Conceptually:

```python id="u7c2nx"
@asynccontextmanager
async def lifespan(app):
    await initialize_resources()

    yield

    await cleanup_resources()
```

The lifecycle is:

```text id="b6p4zd"
Application startup
       ↓
Initialize resources
       ↓
yield
       ↓
Application running
       ↓
Shutdown
       ↓
Cleanup
```

This can be useful for application-level resources such as:

* Database pools
* HTTP clients
* Model clients
* Application caches
* Other shared infrastructure

Use the framework's current lifecycle API when implementing this in a real application.

---

# 24. `nullcontext`

Sometimes a function should use a context manager only when a condition is enabled.

Python provides:

```python id="y2m5kc"
contextlib.nullcontext
```

Example:

```python id="n8x4pv"
from contextlib import nullcontext


context = timer("RAG") if enable_timing else nullcontext()

with context:
    run_rag()
```

This avoids writing two separate code paths.

---

# 25. `suppress`

Python also provides:

```python id="d4m8sz"
contextlib.suppress
```

Example:

```python id="q7p1xv"
from contextlib import suppress


with suppress(FileNotFoundError):
    delete_cache_file()
```

This is useful when a specific exception is intentionally ignorable.

However, broad suppression should be avoided.

Prefer:

```python id="g9s2mc"
with suppress(FileNotFoundError):
```

over:

```python id="j5k8rq"
with suppress(Exception):
```

---

# 26. `ExitStack`

Sometimes the number of resources is dynamic.

For example:

```text id="v4n7pa"
Document 1 → resource
Document 2 → resource
Document 3 → resource
...
```

`ExitStack` allows multiple context managers to be managed dynamically.

Example:

```python id="x6q2mb"
from contextlib import ExitStack


with ExitStack() as stack:

    files = [
        stack.enter_context(
            open(path, "r", encoding="utf-8")
        )
        for path in paths
    ]

    process_files(files)
```

When the stack exits, the managed resources are cleaned up.

This is useful when the number of resources is not known ahead of time.

---

# 27. `AsyncExitStack`

For asynchronous resources:

```python id="m8f3qk"
from contextlib import AsyncExitStack
```

Conceptually:

```python id="j6w9sb"
async with AsyncExitStack() as stack:
    resource = await stack.enter_async_context(
        create_resource()
    )

    await process(resource)
```

This becomes useful in applications managing multiple async resources dynamically.

---

# 28. Context Managers and HTTP Clients

Modern AI services frequently communicate with:

* LLM providers
* Embedding providers
* Search services
* Internal APIs
* Vector databases

An HTTP client may expose a context-manager interface:

```python id="v2k7mx"
with create_http_client() as client:
    response = client.get(...)
```

For async clients:

```python id="q8m4zr"
async with create_async_http_client() as client:
    response = await client.get(...)
```

This allows the client to cleanly manage connection pools and network resources.

The exact API depends on the HTTP library.

---

# 29. Context Managers and Temporary Files

AI pipelines often need temporary artifacts.

Examples:

* Extracted PDFs
* Converted images
* Intermediate audio
* Uploaded files
* Generated datasets

Use:

```python id="b7x3nj"
from tempfile import TemporaryDirectory


with TemporaryDirectory() as directory:
    run_pipeline(directory)
```

This provides automatic cleanup.

It is preferable to manually tracking temporary files in many situations.

---

# 30. Context Managers and Resource Limits

Context managers can represent temporary ownership of a resource.

For example:

```text id="n3p8cx"
Acquire GPU slot
      ↓
Run inference
      ↓
Release GPU slot
```

or:

```text id="f6q2wv"
Acquire semaphore
      ↓
Call external API
      ↓
Release semaphore
```

Python's synchronization primitives can often be used with `with`.

Example:

```python id="h4s9mv"
with lock:
    update_shared_state()
```

The same lifecycle principle applies:

```text id="j1c7zk"
Acquire
   ↓
Use
   ↓
Release
```

---

# 31. Common Mistakes

## Mistake 1: Forgetting cleanup

Without a context manager, resources may remain open if an exception occurs.

---

## Mistake 2: Suppressing exceptions accidentally

Returning `True` from `__exit__` suppresses the exception.

Only do this intentionally.

---

## Mistake 3: Putting too much business logic into a context manager

A context manager should usually focus on lifecycle management.

Avoid turning:

```python id="n7x2mc"
with resource():
```

into a hidden application workflow.

---

## Mistake 4: Using synchronous context managers in async code incorrectly

Async resources should generally use:

```python id="c5q8yw"
async with
```

when the resource requires asynchronous acquisition or cleanup.

---

## Mistake 5: Forgetting that `yield` divides lifecycle phases

With:

```python id="f8m3kv"
@contextmanager
def resource():
    acquire()

    try:
        yield
    finally:
        release()
```

everything before `yield` is setup.

Everything after `yield` in `finally` is cleanup.

---

## Mistake 6: Holding resources longer than necessary

Avoid:

```python id="p6w9sd"
with database_connection():
    perform_unrelated_long_operation()
```

Keep resource lifetimes as short as practical.

---

## Mistake 7: Creating unnecessary custom context managers

Python already provides many useful ones.

Before creating a custom abstraction, check:

```text id="r4n8jq"
contextlib
tempfile
threading
database library
HTTP library
framework
```

---

# Production Insight

A context manager makes resource ownership explicit.

Consider a RAG ingestion service:

```text id="q1c6vb"
Upload
  ↓
Temporary workspace
  ↓
Extract
  ↓
Chunk
  ↓
Embed
  ↓
Store
  ↓
Cleanup
```

A context manager can make the temporary resource lifecycle explicit:

```python id="u8m2zr"
with TemporaryDirectory() as directory:
    documents = extract_documents(directory)
    chunks = chunk_documents(documents)
    embeddings = generate_embeddings(chunks)
    store_embeddings(embeddings)
```

Even if embedding or extraction fails:

```text id="f3k7sx"
Exception
   ↓
Exit context
   ↓
Cleanup
   ↓
Exception propagates
```

This is much safer than relying on every execution path to remember cleanup manually.

The production principle is:

> If a resource has a clear acquire/use/release lifecycle, consider representing that lifecycle with a context manager.

---

# How It's Used in AI Frameworks

Context-manager patterns appear throughout AI engineering.

Common examples include:

```text id="x5m8qr"
Files
    → open()

Database transactions
    → transaction()

HTTP clients
    → client sessions

Temporary files
    → TemporaryDirectory()

Locks
    → with lock

Tracing
    → span lifecycle

Application startup/shutdown
    → async lifespan

Streaming resources
    → async context managers
```

Framework APIs change over time, so check the current documentation when implementing framework-specific lifecycle behavior.

The transferable concept is:

```text id="h9v3kc"
Acquire
   ↓
Use
   ↓
Cleanup
```

---

# Practical Exercise

Build a context manager for an AI pipeline timer.

Create:

```python id="j6q1rx"
from contextlib import contextmanager
import time
```

Implement:

```python id="w2m7vb"
@contextmanager
def measure_operation(name):
    ...
```

It should:

1. Record the start time.
2. Execute the code inside the `with` block.
3. Record the end time.
4. Print the elapsed time.
5. Still record the time if an exception occurs.

Use it:

```python id="c4n9pk"
with measure_operation("RAG pipeline"):
    load_documents()
    retrieve_documents()
    generate_answer()
```

Then extend it to report:

```text id="z7v3qm"
Operation: RAG pipeline
Duration: 1.84s
Status: success
```

and:

```text id="k5x8sd"
Operation: RAG pipeline
Duration: 0.92s
Status: failed
```

---

# Interview Questions

## Beginner

1. What is a context manager?
2. What does the `with` statement do?
3. Why is `with open(...)` preferred over manually calling `close()`?
4. What are `__enter__` and `__exit__`?
5. What does `__exit__` receive?
6. How can a context manager suppress an exception?
7. What is `contextlib`?
8. What does `@contextmanager` do?
9. Why is `try/finally` commonly used with context managers?
10. What is `yield` doing inside a `@contextmanager` function?

## Intermediate

11. What happens if an exception occurs inside a `with` block?
12. What is the difference between a decorator and a context manager?
13. What is the difference between a context manager and a context variable?
14. What is `ExitStack`?
15. What is `nullcontext`?
16. What is `contextlib.suppress`?
17. How do you create an async context manager?
18. What are `__aenter__` and `__aexit__`?
19. When should you use `async with`?
20. How would you manage multiple resources with `ExitStack`?

## AI Engineering

21. How can context managers help with RAG pipelines?
22. How would you manage temporary files during document ingestion?
23. How would you use a context manager for LLM tracing?
24. How would you manage database transactions in an AI service?
25. How would you safely use an HTTP client in an AI application?
26. How would you manage async resources in FastAPI?
27. Why should LLM clients and network resources have clearly defined lifecycles?
28. How would you build a context manager for measuring AI pipeline latency?
29. How would you combine context managers with observability?
30. Design a context-managed document ingestion pipeline that guarantees cleanup when processing fails.

---

# Key Takeaways

* Context managers manage resource lifecycles.
* The `with` statement provides a clean setup/cleanup boundary.
* `__enter__` handles entering the context.
* `__exit__` handles exiting the context.
* `@contextmanager` simplifies custom context managers.
* `try/finally` is central to reliable cleanup.
* Context managers can intentionally suppress exceptions.
* `async with` is used for asynchronous context managers.
* `ExitStack` manages dynamically created resources.
* `AsyncExitStack` handles dynamic async resources.
* Context managers are useful for files, databases, locks, temporary files, HTTP clients, tracing, and application lifecycle management.
* Decorators wrap functions; context managers wrap execution blocks or resource lifecycles.
* Resource lifetimes should generally be kept as short as practical.
* Cleanup logic should be reliable even when exceptions occur.

---

# Summary

Context managers provide a clean abstraction for managing:

```text id="f8x2zn"
Acquire resource
      ↓
Use resource
      ↓
Cleanup resource
```

For AI engineering, this pattern appears in many places:

```text id="m7q4kc"
File processing
      ↓
Temporary files
      ↓
Database transactions
      ↓
HTTP clients
      ↓
LLM tracing
      ↓
Async resources
```

The most important pattern to remember is:

```python id="w3n6rp"
with resource():
    use_resource()
```

and for custom synchronous lifecycle management:

```python id="d9k2vm"
@contextmanager
def resource():
    acquire()

    try:
        yield
    finally:
        release()
```

For asynchronous resources:

```python id="q6v8bx"
@asynccontextmanager
async def resource():
    await acquire()

    try:
        yield
    finally:
        await release()
```

Context managers are a small Python feature with significant production value because they make **resource ownership, cleanup, and failure handling explicit**.
