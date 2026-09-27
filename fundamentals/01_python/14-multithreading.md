# Multithreading in Python for AI Engineering

## Why does an AI Engineer need this?

AI applications frequently spend a significant amount of time **waiting** rather than computing.

For example, an AI application may need to:

* Call an LLM API
* Download documents
* Call external APIs
* Read files
* Query databases
* Query vector databases
* Send notifications
* Execute multiple independent tools
* Process multiple user requests simultaneously

If these operations are performed one after another, the application may spend most of its time waiting.

**Multithreading** allows Python to execute multiple tasks concurrently, particularly when those tasks spend significant time waiting for I/O.

A useful mental model is:

```text
Single Thread

Task A ──────── wait ────────
                         Task B ──────── wait ────────
                                                  Task C ────────


Multiple Threads

Task A ──────── wait ────────
Task B ─── wait ─────────────
Task C ─────── wait ─────────

        ↓
Less idle waiting
```

Multithreading is especially useful when working with **blocking libraries** that do not provide an asynchronous API.

---

# 1. What is a Thread?

A thread is a unit of execution inside a process.

A Python process can contain multiple threads:

```text
Process
│
├── Thread 1
├── Thread 2
├── Thread 3
└── Thread 4
```

Threads within the same process share memory.

This makes communication between threads relatively convenient, but it also means that multiple threads can access the same data.

That creates potential synchronization problems.

---

# 2. Process vs Thread

A process is an independent running program.

A thread is an execution path inside a process.

```text
Process
│
├── Thread
├── Thread
└── Thread
```

Processes generally have separate memory spaces.

Threads inside the same process share memory.

| Feature           | Process                     | Thread                    |
| ----------------- | --------------------------- | ------------------------- |
| Memory            | Separate                    | Shared                    |
| Creation cost     | Higher                      | Lower                     |
| Communication     | More expensive              | Easier                    |
| Isolation         | Higher                      | Lower                     |
| Best for          | CPU-heavy work              | I/O-heavy work            |
| Python GIL impact | Process has own interpreter | Threads share interpreter |

---

# 3. The Python GIL

One of the most important concepts when learning Python multithreading is the **Global Interpreter Lock (GIL)**.

In standard CPython implementations, the GIL prevents multiple threads from executing Python bytecode simultaneously within the same interpreter.

This means:

```text
4 CPU-heavy Python threads
        ↓
Do NOT automatically mean
        ↓
4 CPU cores running Python code simultaneously
```

Therefore, multithreading is generally **not the solution for CPU-bound Python code**.

However, this does not mean threads are useless.

Threads are very useful for **I/O-bound workloads**.

For example:

```text
Thread 1 → waiting for LLM API
Thread 2 → waiting for database
Thread 3 → waiting for HTTP API
Thread 4 → waiting for file
```

While one thread is waiting, another thread can make progress.

---

# 4. I/O-Bound vs CPU-Bound

Understanding this distinction is critical.

## I/O-Bound

The application spends most of its time waiting for something external.

Examples:

* HTTP requests
* LLM API calls
* Database queries
* Vector database requests
* File operations
* Network requests
* Cloud storage

```text
CPU → request → WAIT → response
```

Multithreading can be useful here.

---

## CPU-Bound

The application spends most of its time performing computation.

Examples:

* Large numerical calculations
* Image processing
* CPU-heavy parsing
* Compression
* Complex data processing
* Some ML preprocessing tasks

```text
CPU → computation → computation → computation
```

For CPU-heavy Python code, multiprocessing or native libraries that release the GIL may be more appropriate.

---

# 5. Creating a Thread

Python provides the `threading` module.

```python
import threading


def process_document():
    print("Processing document")


thread = threading.Thread(target=process_document)

thread.start()
thread.join()
```

`start()` begins execution.

`join()` waits for the thread to finish.

```text
Main Thread
    │
    ├── starts Thread
    │
    ├───────────────
    │              Thread executes
    │───────────────
    │
    └── join()
```

---

# 6. Passing Arguments to Threads

You can pass arguments using `args`.

```python
import threading


def process_document(document_id):
    print(f"Processing {document_id}")


thread = threading.Thread(
    target=process_document,
    args=("doc-123",)
)

thread.start()
thread.join()
```

For keyword arguments:

```python
thread = threading.Thread(
    target=process_document,
    kwargs={"document_id": "doc-123"}
)
```

---

# 7. Running Multiple Threads

Suppose an application needs to fetch information from three independent services.

Sequential execution:

```python
fetch_users()
fetch_documents()
fetch_metadata()
```

The operations happen one after another.

With threads:

```python
import threading


threads = [
    threading.Thread(target=fetch_users),
    threading.Thread(target=fetch_documents),
    threading.Thread(target=fetch_metadata),
]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()
```

The operations can overlap while they are waiting.

---

# 8. ThreadPoolExecutor

Creating threads manually is useful for learning, but production applications often use:

```python
concurrent.futures.ThreadPoolExecutor
```

Example:

```python
from concurrent.futures import ThreadPoolExecutor


def fetch_document(document_id):
    return f"Document {document_id}"


document_ids = ["doc-1", "doc-2", "doc-3", "doc-4"]


with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(
        executor.map(fetch_document, document_ids)
    )

print(results)
```

The executor manages the worker threads for you.

---

# 9. Why ThreadPoolExecutor is Useful for AI Applications

Consider a RAG system processing multiple documents.

```text
Document 1 → embedding API
Document 2 → embedding API
Document 3 → embedding API
Document 4 → embedding API
```

If the embedding client is synchronous, making the calls sequentially can waste time waiting for network responses.

A thread pool can execute several blocking requests concurrently.

```text
Thread 1 → embedding(doc1)
Thread 2 → embedding(doc2)
Thread 3 → embedding(doc3)
Thread 4 → embedding(doc4)
```

This pattern can also be useful for:

* External API calls
* Document downloads
* Metadata retrieval
* Synchronous vector DB clients
* Synchronous LLM SDK calls
* Multiple independent tools

---

# 10. ThreadPoolExecutor with Results

```python
from concurrent.futures import ThreadPoolExecutor


def generate_embedding(text):
    # Call embedding API
    return f"embedding-for-{text}"


texts = [
    "Introduction to RAG",
    "Vector databases",
    "Agent memory",
    "Tool calling",
]


with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [
        executor.submit(generate_embedding, text)
        for text in texts
    ]

    results = [
        future.result()
        for future in futures
    ]

print(results)
```

`submit()` returns a `Future`.

A `Future` represents work that is running or will run.

```text
submit()
   ↓
Future
   ↓
result()
   ↓
Actual result
```

---

# 11. Futures

A future allows you to interact with work that has been submitted to an executor.

```python
future = executor.submit(generate_embedding, text)

print(future.done())

result = future.result()
```

Useful methods include:

```python
future.done()
future.result()
future.exception()
future.cancel()
```

Be careful with:

```python
future.result()
```

Calling it before the operation finishes can block the current thread.

---

# 12. Handling Exceptions

Exceptions raised inside worker threads need to be handled properly.

```python
from concurrent.futures import ThreadPoolExecutor


def call_api():
    raise RuntimeError("API failed")


with ThreadPoolExecutor(max_workers=2) as executor:
    future = executor.submit(call_api)

    try:
        result = future.result()
    except RuntimeError as error:
        print(f"Request failed: {error}")
```

For AI applications, failures can come from:

* LLM API errors
* Timeouts
* Rate limits
* Network failures
* Authentication errors
* Database failures

Threading does not remove the need for proper error handling.

---

# 13. Shared State

Threads share memory.

This is convenient:

```python
results = []

def process():
    results.append("done")
```

But shared mutable state can introduce race conditions.

For example:

```python
counter = 0
```

Multiple threads modifying the same variable can produce unexpected behavior.

---

# 14. Race Conditions

A race condition occurs when the result depends on the timing of multiple threads accessing shared state.

Conceptually:

```text
Thread A → read counter = 10
Thread B → read counter = 10

Thread A → write 11
Thread B → write 11

Expected: 12
Actual:   11
```

This is one reason shared mutable state should be minimized.

---

# 15. Locks

Python provides `threading.Lock`.

```python
import threading


counter = 0
lock = threading.Lock()


def increment():
    global counter

    with lock:
        counter += 1
```

Only one thread can execute the protected section at a time.

```text
Thread A
   ↓
[ LOCK ]
   ↓
modify shared state
   ↓
[ UNLOCK ]

Thread B
   ↓
waits
```

---

# 16. When to Use Locks

Use a lock when multiple threads need to safely modify shared mutable state.

Examples:

* Shared counters
* In-memory caches
* Shared queues
* Mutable registries
* Resource managers

However, don't use locks everywhere.

Excessive locking can make code:

* Harder to reason about
* Slower
* More prone to deadlocks
* Less concurrent

A better design is often to reduce shared mutable state.

---

# 17. Thread-Safe Queues

Python provides:

```python
queue.Queue
```

This is useful for producer-consumer systems.

```python
from queue import Queue
import threading


queue = Queue()


def producer():
    queue.put("document-1")
    queue.put("document-2")


def consumer():
    document = queue.get()
    print(f"Processing {document}")
    queue.task_done()
```

Conceptually:

```text
Producer
   │
   ↓
 Queue
   │
   ↓
Consumer
```

This pattern can be useful in document-processing pipelines.

---

# 18. Threading for RAG Pipelines

Consider document ingestion:

```text
Documents
    ↓
Extract text
    ↓
Chunk
    ↓
Generate embeddings
    ↓
Store in vector DB
```

Some operations may be independent.

For example:

```text
Document 1 ──→ embedding
Document 2 ──→ embedding
Document 3 ──→ embedding
Document 4 ──→ embedding
```

If the embedding client is synchronous, a thread pool can improve throughput.

Example:

```python
from concurrent.futures import ThreadPoolExecutor


def embed_chunk(chunk):
    return embedding_client.embed(chunk)


chunks = load_chunks()


with ThreadPoolExecutor(max_workers=8) as executor:
    embeddings = list(
        executor.map(embed_chunk, chunks)
    )
```

In production, the correct worker count depends on:

* API rate limits
* Provider limits
* Network latency
* Batch support
* Database capacity
* Memory usage
* Cost
* Application workload

More threads do **not** automatically mean better performance.

---

# 19. Threading for Agent Tools

Suppose an agent needs information from several independent systems:

```text
                 ┌── CRM
                 │
User → Agent ────┼── Database
                 │
                 ├── Search API
                 │
                 └── Internal API
```

If these tools use blocking clients, threads can execute independent calls concurrently.

```python
from concurrent.futures import ThreadPoolExecutor


def get_crm_data():
    ...


def get_customer_data():
    ...


def search_documents():
    ...


with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [
        executor.submit(get_crm_data),
        executor.submit(get_customer_data),
        executor.submit(search_documents),
    ]

    results = [future.result() for future in futures]
```

This can reduce total waiting time when the operations are independent.

---

# 20. Threading vs asyncio

Both can be useful for I/O-bound work, but they work differently.

| Feature            | Threading                   | asyncio                       |
| ------------------ | --------------------------- | ----------------------------- |
| Concurrency model  | Threads                     | Event loop                    |
| Execution          | OS-managed threads          | Cooperative                   |
| Memory             | Shared between threads      | Usually shared within process |
| Blocking libraries | Works well                  | Must be isolated/offloaded    |
| Async libraries    | Not required                | Designed for them             |
| Overhead           | Higher                      | Generally lower               |
| Complexity         | Often simpler for sync code | Requires async programming    |
| Best use           | Blocking I/O                | Async I/O                     |

Example:

```text
Synchronous HTTP client
        ↓
ThreadPoolExecutor
```

versus:

```text
Async HTTP client
        ↓
asyncio
```

---

# 21. Running Blocking Code from Asyncio

Sometimes an AI application is already asynchronous but needs to call a blocking library.

For example:

```python
result = synchronous_llm_client.call(...)
```

Calling it directly inside an async function can block the event loop.

Instead:

```python
import asyncio


def blocking_call():
    return synchronous_llm_client.call(...)


async def main():
    result = await asyncio.to_thread(blocking_call)
```

`asyncio.to_thread()` allows blocking work to execute in a separate thread.

This is an important pattern when integrating older or synchronous libraries into modern async applications.

---

# 22. Threading vs Multiprocessing

A simplified rule:

```text
I/O-bound
   ├── asyncio
   └── threading

CPU-bound
   └── multiprocessing
```

For example:

| Workload                      | Common approach                |
| ----------------------------- | ------------------------------ |
| LLM API calls                 | Async / Threads                |
| HTTP requests                 | Async / Threads                |
| Database calls                | Async / Threads                |
| Vector DB calls               | Async / Threads                |
| File I/O                      | Threads / Async                |
| CPU-heavy Python processing   | Multiprocessing                |
| Parallel numerical operations | Depends on library             |
| GPU workloads                 | Usually specialized frameworks |

This is a guideline, not an absolute rule.

The actual choice depends on the library and workload.

---

# 23. Threading in FastAPI

FastAPI applications commonly use asynchronous endpoints:

```python
@app.get("/documents")
async def get_documents():
    ...
```

If a dependency is synchronous or blocking, it should not unnecessarily block the event loop.

A synchronous function may instead be executed in a worker thread depending on the framework's execution model.

The important engineering principle is:

> Don't allow long-running blocking operations to unnecessarily block the main async event loop.

---

# 24. Thread Safety of AI Libraries

Not every library or client should automatically be assumed to be thread-safe.

Before sharing a client instance across threads, check the library's documentation.

Potentially shared resources include:

* HTTP clients
* Database clients
* SDK clients
* Model objects
* Caches
* Connection pools

Possible strategies include:

```text
One shared thread-safe client
```

or:

```text
One client per thread
```

or:

```text
Use a connection/client pool
```

The correct choice depends on the library.

---

# 25. Avoid Thread Explosion

This is dangerous:

```python
for document in documents:
    threading.Thread(
        target=process_document,
        args=(document,)
    ).start()
```

If there are thousands of documents, you could create thousands of threads.

Instead, use a bounded executor:

```python
with ThreadPoolExecutor(max_workers=10) as executor:
    executor.map(process_document, documents)
```

This provides controlled concurrency.

---

# 26. Rate Limits Matter

Suppose an LLM provider allows only a limited number of requests per minute.

Increasing:

```python
max_workers=100
```

does not mean your application should send 100 requests simultaneously.

You may encounter:

```text
Too many requests
       ↓
Rate limit
       ↓
Retries
       ↓
More requests
       ↓
More rate limits
```

This can create a feedback loop.

Production systems should consider:

* Concurrency limits
* Rate limits
* Retries
* Exponential backoff
* Timeouts
* Circuit breakers
* Request batching

---

# 27. Example: Controlled Concurrent API Calls

```python
from concurrent.futures import ThreadPoolExecutor
import requests


def fetch(url):
    response = requests.get(
        url,
        timeout=10
    )

    response.raise_for_status()

    return response.json()


urls = [
    "https://example.com/api/1",
    "https://example.com/api/2",
    "https://example.com/api/3",
]


with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(fetch, urls))
```

Important production characteristics:

* Timeout is configured.
* Worker count is bounded.
* HTTP errors are not silently ignored.
* Results are collected explicitly.

---

# 28. Threading and Timeouts

External AI systems should not be allowed to wait forever.

Use timeouts at the client level whenever supported.

```python
requests.get(
    url,
    timeout=10
)
```

For futures:

```python
future.result(timeout=10)
```

Timeouts are particularly important for:

* LLM APIs
* External APIs
* Database calls
* Vector databases
* Tool calls

---

# 29. Common Mistakes

## Mistake 1: Using threads for CPU-heavy Python code

```python
ThreadPoolExecutor(max_workers=8)
```

does not automatically make CPU-bound Python code eight times faster.

Use multiprocessing or specialized native/GPU libraries where appropriate.

---

## Mistake 2: Creating unlimited threads

Thousands of threads can consume significant resources.

Use bounded pools.

---

## Mistake 3: Ignoring rate limits

Concurrent LLM requests can quickly hit provider limits.

Concurrency must be designed together with rate limiting.

---

## Mistake 4: Sharing unsafe mutable state

Multiple threads modifying shared data can introduce race conditions.

Prefer immutable data or thread-safe structures.

---

## Mistake 5: Holding locks too long

Avoid:

```python
with lock:
    call_external_api()
    process_large_document()
```

A slow external call should generally not hold a shared lock.

Keep critical sections small.

---

## Mistake 6: Assuming every SDK is thread-safe

Check the specific SDK/client documentation.

---

## Mistake 7: Mixing async and blocking code incorrectly

This can block an entire async application.

Use async-native libraries or move blocking operations to threads when appropriate.

---

# Production Insight

Multithreading is not primarily about "making Python faster."

For AI engineers, its major value is **overlapping waiting time**.

Consider an agent that calls three independent services:

```text
CRM API        → 800 ms
Search API     → 600 ms
Customer DB    → 500 ms
```

Sequential execution could approach:

```text
800 + 600 + 500 = 1900 ms
```

If the calls can safely execute concurrently, total wall-clock time can approach the duration of the slowest call:

```text
max(800, 600, 500) ≈ 800 ms
```

Real systems will have additional overhead and constraints, but this illustrates the core benefit.

The goal is not:

> "Use more threads."

The goal is:

> "Identify independent blocking operations and introduce controlled concurrency where it improves the system."

---

# How It's Used in AI Frameworks

## LLM SDKs

Many LLM providers offer both synchronous and asynchronous clients.

If using a synchronous client:

```text
Sync client
    ↓
ThreadPoolExecutor
```

can be useful for concurrent independent requests.

If an async client is available:

```text
Async client
    ↓
asyncio
```

is often a more natural design.

---

## LangChain

AI workflows may execute multiple independent operations such as:

```text
Retriever
Tool A
Tool B
Tool C
```

Depending on the specific runnable and integration, frameworks can provide synchronous and asynchronous execution models.

When building production systems, understand whether the underlying operation is:

* Sync
* Async
* CPU-bound
* I/O-bound

rather than assuming the framework automatically makes everything concurrent.

---

## LangGraph

Agent graphs can contain multiple independent operations.

Conceptually:

```text
             ┌── Retriever
Agent ───────┼── CRM Tool
             └── Searc
```
