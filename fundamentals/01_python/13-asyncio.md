# Asyncio

`asyncio` is Python's standard library for writing concurrent code using `async` and `await`.

It is particularly useful for AI applications because many AI workloads spend significant time **waiting** for external operations:

* LLM API responses
* embedding APIs
* vector database queries
* HTTP requests
* database queries
* tool calls
* microservice communication
* file or network operations

Instead of blocking the program while waiting for one operation to finish, asynchronous code can allow other work to run.

---

# Why does an AI Engineer need this?

Consider an agent that needs to call three independent services:

```text
Agent
 ├── Search API
 ├── Vector Database
 └── External API
```

A sequential implementation might do:

```text
Search API
    ↓ wait
Vector DB
    ↓ wait
External API
    ↓ wait
Continue
```

If the operations are independent, they can potentially execute concurrently:

```text
Search API ────────┐
Vector DB ─────────┼──→ Continue
External API ──────┘
```

This can reduce overall waiting time.

The important concept is:

> **Asyncio is primarily about efficiently handling waiting, not making CPU-heavy code faster.**

---

# 1. What Is Concurrency?

Suppose we have:

```python
def task_a():
    ...
    
def task_b():
    ...
```

A sequential program might execute:

```text
Task A
  ↓
Task B
```

Concurrency allows progress on multiple tasks during periods when one task is waiting.

For I/O-bound applications:

```text
Task A → waiting
             ↓
Task B → running
             ↓
Task A → resumes
```

This is where `asyncio` becomes useful.

---

# 2. Synchronous Code

Consider:

```python
import time


def call_llm():
    time.sleep(2)
    return "LLM response"


def call_search():
    time.sleep(2)
    return "Search result"


call_llm()
call_search()
```

The approximate execution time is:

```text
2 seconds + 2 seconds = 4 seconds
```

The second operation does not begin until the first one finishes.

---

# 3. Async Functions

An asynchronous function is defined using:

```python
async def
```

Example:

```python
import asyncio


async def call_llm():
    await asyncio.sleep(2)
    return "LLM response"
```

The function now returns a coroutine when called:

```python
result = call_llm()
```

It doesn't immediately execute the function body to completion.

You need to run it using an event loop.

---

# 4. `await`

`await` tells Python that the current coroutine is waiting for an asynchronous operation.

Example:

```python
async def call_llm():
    await asyncio.sleep(2)
    return "LLM response"
```

Conceptually:

```text
Coroutine
   ↓
await operation
   ↓
I/O waiting
   ↓
Other async work can run
   ↓
Operation completes
   ↓
Coroutine resumes
```

---

# 5. Running Async Code

The common entry point is:

```python
import asyncio


async def main():
    print("Hello")
    

asyncio.run(main())
```

`asyncio.run()` creates and manages the event loop for the program's top-level coroutine.

---

# 6. Sequential Async Code

Consider:

```python
import asyncio


async def call_service(name: str):
    await asyncio.sleep(2)
    return f"{name} completed"


async def main():
    result1 = await call_service("LLM")
    result2 = await call_service("Search")

    print(result1)
    print(result2)


asyncio.run(main())
```

Although the functions are asynchronous, this is still sequential:

```text
LLM
 ↓ 2 sec
Search
 ↓ 2 sec
Done
```

Approximate time:

```text
4 seconds
```

Simply adding `async` does **not** automatically make operations concurrent.

---

# 7. `asyncio.gather()`

When operations are independent, `asyncio.gather()` can run them concurrently.

```python
import asyncio


async def call_service(name: str):
    await asyncio.sleep(2)
    return f"{name} completed"


async def main():
    results = await asyncio.gather(
        call_service("LLM"),
        call_service("Search"),
    )

    print(results)


asyncio.run(main())
```

Conceptually:

```text
LLM ──────────┐
              ├──→ results
Search ───────┘
```

Approximate time:

```text
2 seconds
```

rather than:

```text
4 seconds
```

assuming both operations take approximately two seconds and can safely run concurrently.

---

# 8. AI Example: Parallel Retrieval

Suppose an AI system searches multiple sources:

```python
async def search_vector_db(query: str):
    ...


async def search_web(query: str):
    ...


async def search_documents(query: str):
    ...
```

These searches may be independent.

You could orchestrate them with:

```python
results = await asyncio.gather(
    search_vector_db(query),
    search_web(query),
    search_documents(query),
)
```

Conceptually:

```text
                   ┌── Vector DB
                   │
Query ─────────────┼── Web Search
                   │
                   └── Documents
                         ↓
                    Combine results
```

This pattern is useful for multi-source retrieval.

---

# 9. Coroutine

Calling an async function produces a coroutine object.

```python
async def fetch_data():
    return "data"


result = fetch_data()
```

`result` is not the returned string yet.

It represents work that can be awaited.

You normally do:

```python
result = await fetch_data()
```

inside another async function.

---

# 10. Tasks

A task schedules a coroutine to run on the event loop.

```python
task = asyncio.create_task(
    call_service("LLM")
)
```

You can then continue doing other work.

Later:

```python
result = await task
```

This is useful when you want to start asynchronous work before immediately waiting for its result.

---

# 11. Tasks vs `gather()`

Consider:

```python
results = await asyncio.gather(
    task_a(),
    task_b(),
    task_c(),
)
```

This is convenient when you want to start several operations and wait for their results.

With tasks:

```python
task_a = asyncio.create_task(operation_a())
task_b = asyncio.create_task(operation_b())

# Do other async work here

result_a = await task_a
result_b = await task_b
```

Tasks give you more control over scheduling and lifecycle.

---

# 12. Async HTTP Requests

AI applications frequently call APIs.

Using an asynchronous HTTP client such as `httpx`:

```python
import httpx


async def fetch_data(url: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        response.raise_for_status()
        return response.json()
```

The network request is asynchronous.

This allows the event loop to work on other tasks while waiting for the response.

---

# 13. Async LLM Calls

Many modern AI SDKs provide asynchronous APIs.

The exact method names depend on the SDK and version, but the general pattern is:

```python
response = await client.some_async_method(...)
```

For example, an application might conceptually do:

```python
async def generate_answer(prompt: str) -> str:
    response = await llm_client.generate_async(prompt)
    return response.text
```

The important concept is not the specific SDK method.

It is:

```text
Application
    ↓
Async LLM request
    ↓
Await network response
    ↓
Continue execution
```

Always check the current SDK documentation for the exact asynchronous API.

---

# 14. Async Database Operations

AI applications often combine LLMs with databases.

For example:

```text
Request
  ↓
PostgreSQL
  ↓
Vector DB
  ↓
LLM
```

If the database client supports asynchronous operations, you can use:

```python
result = await database.fetch(...)
```

This prevents the event loop from being blocked while waiting for the database.

---

# 15. Async Vector Database Operations

Suppose a RAG application performs a vector search.

Conceptually:

```python
async def retrieve(query: str):
    embedding = await create_embedding(query)

    results = await vector_db.search(
        embedding=embedding,
        limit=5,
    )

    return results
```

If the embedding API and vector database client both support async operations, the complete retrieval pipeline can remain asynchronous.

---

# 16. Async Agent Tools

An agent may have tools such as:

```text
Search documentation
Query database
Call CRM API
Send notification
Call another service
```

Some tools are naturally I/O-bound.

For example:

```python
async def search_crm(customer_id: str):
    response = await client.get(...)
    return response.json()
```

The agent can await the tool without blocking the entire event loop.

---

# 17. Async Context Managers

You will often see:

```python
async with
```

For example:

```python
async with httpx.AsyncClient() as client:
    response = await client.get(url)
```

This allows asynchronous setup and cleanup of resources.

The same concept as:

```python
with
```

but for asynchronous resources.

---

# 18. Async Iterators

Some AI APIs stream responses.

For example:

```text
Token 1
Token 2
Token 3
Token 4
...
```

Asynchronous iteration can be used:

```python
async for chunk in stream:
    print(chunk)
```

This is useful for:

* streaming LLM responses
* processing events
* consuming asynchronous data sources
* agent event streams

---

# 19. Streaming LLM Responses

A conversational application might stream:

```text
Hello
Hello, how
Hello, how can
Hello, how can I
...
```

Instead of waiting for the complete response.

Conceptually:

```python
async for chunk in llm_stream:
    yield chunk
```

This allows the application to forward chunks to the client as they arrive.

---

# 20. Async Generators

An async generator combines:

```text
async
+
yield
```

Example:

```python
async def generate_events():
    for i in range(3):
        await asyncio.sleep(1)
        yield f"event-{i}"
```

Consume it:

```python
async for event in generate_events():
    print(event)
```

This is useful for streaming AI events.

---

# 21. Timeouts

External services can take too long.

An AI application should not wait indefinitely.

Modern Python provides timeout mechanisms through `asyncio`.

For example:

```python
async def call_model():
    ...


async def main():
    try:
        result = await asyncio.wait_for(
            call_model(),
            timeout=10,
        )
    except asyncio.TimeoutError:
        print("Request timed out")
```

The exact timeout API can vary depending on the Python version and library.

The principle is:

> **Every external operation should have a sensible timeout.**

---

# 22. Cancellation

Tasks can be cancelled.

```python
task = asyncio.create_task(call_service())

task.cancel()
```

The coroutine can receive cancellation through `asyncio.CancelledError`.

Cancellation matters for:

* user disconnects
* request timeouts
* shutdown
* cancelled agent runs
* abandoned workflows

Production systems should clean up resources correctly when cancellation occurs.

---

# 23. Semaphores

Launching hundreds of requests concurrently can overload:

* your application
* an LLM provider
* a vector database
* an external API

A semaphore can limit concurrency.

```python
semaphore = asyncio.Semaphore(5)
```

Then:

```python
async def limited_call():
    async with semaphore:
        return await call_service()
```

This means only a limited number of calls can execute inside the protected section at once.

---

# 24. Rate Limits

AI APIs commonly impose rate limits.

Suppose you have:

```text
100 tasks
```

and each task calls an LLM.

Running all 100 simultaneously may produce:

```text
RateLimitError
RateLimitError
RateLimitError
...
```

A production implementation may combine:

```text
Concurrency limit
+
Rate limiting
+
Retries
+
Backoff
```

Asyncio provides concurrency primitives, but it does not automatically solve provider rate limits.

---

# 25. Asyncio Is Not Multithreading

This distinction is important.

### Asyncio

Primarily uses cooperative concurrency within an event loop.

Useful for:

```text
I/O-bound work
```

Examples:

* HTTP requests
* database operations
* LLM calls
* network communication

### Threads

Useful when working with blocking operations that cannot easily be made asynchronous.

### Multiprocessing

Useful for CPU-heavy work that benefits from separate processes.

---

# 26. Asyncio vs Threads vs Multiprocessing

| Problem                      | Typical approach                        |
| ---------------------------- | --------------------------------------- |
| LLM API waiting              | Asyncio                                 |
| HTTP requests                | Asyncio                                 |
| Database I/O                 | Asyncio                                 |
| Vector DB network calls      | Asyncio                                 |
| Blocking third-party library | Thread / appropriate executor           |
| CPU-heavy computation        | Multiprocessing / optimized native code |
| GPU computation              | GPU framework                           |

The actual choice depends on the library and workload.

---

# 27. Blocking Code Inside Async Functions

One of the most common mistakes is putting blocking code inside an async function.

Bad:

```python
import time


async def process():
    time.sleep(10)
```

`time.sleep()` blocks the thread running the event loop.

Prefer:

```python
async def process():
    await asyncio.sleep(10)
```

for an actual asynchronous delay.

More generally, avoid synchronous blocking operations inside an async path when an async alternative exists.

---

# 28. Running Blocking Code

Sometimes you have a synchronous library that cannot be replaced.

Python provides ways to run blocking work outside the event loop.

For example:

```python
result = await asyncio.to_thread(
    blocking_function
)
```

Conceptually:

```text
Async event loop
      │
      ├── Async work
      │
      └── Blocking function → worker thread
```

This prevents the blocking function from freezing the event loop.

---

# 29. Async FastAPI

FastAPI supports asynchronous route handlers.

Example:

```python
from fastapi import FastAPI


app = FastAPI()


@app.get("/chat")
async def chat():
    result = await generate_answer()
    return {"answer": result}
```

This is particularly useful when the endpoint performs asynchronous I/O.

However, simply changing:

```python
def
```

to:

```python
async def
```

does not make blocking code asynchronous.

The operations inside the function must also be compatible with async execution.

---

# 30. Async RAG Architecture

Consider:

```text
User
 ↓
FastAPI
 ↓
Query Processing
 ↓
Embedding API
 ↓
Vector DB
 ↓
Reranker
 ↓
LLM
 ↓
Response
```

A fully asynchronous implementation can look conceptually like:

```text
Request
   ↓
async query processing
   ↓
await embedding
   ↓
await vector search
   ↓
await reranking
   ↓
await LLM
   ↓
Response
```

If multiple retrieval operations are independent:

```text
              ┌── Vector DB
              │
Query ────────┼── Keyword Search
              │
              └── External Search
                    ↓
                Merge Results
                    ↓
                   LLM
```

they may be executed concurrently.

---

# 31. Async Multi-Agent Systems

Asyncio becomes particularly interesting in multi-agent architectures.

Suppose:

```text
Supervisor Agent
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
Research  SQL  API
Agent     Agent Agent
 └────┼────┘
      ↓
Supervisor
```

If the sub-agents can work independently, they may execute concurrently.

Conceptually:

```python
results = await asyncio.gather(
    research_agent.run(task),
    sql_agent.run(task),
    api_agent.run(task),
)
```

The supervisor can then combine the results.

Whether parallel execution is appropriate depends on dependencies between the agents.

---

# 32. Asyncio and Agent State

If multiple asynchronous tasks modify shared state, you need to think carefully about coordination.

For example:

```python
state = {
    "messages": []
}
```

Multiple tasks modifying the same structure can produce difficult-to-debug behavior.

Use appropriate synchronization primitives or design tasks to return results rather than directly modifying shared state.

A safer pattern is often:

```text
Task A → Result A
Task B → Result B
Task C → Result C
             ↓
         Coordinator
```

rather than having every task mutate global state.

---

# 33. Locks

Asyncio provides synchronization primitives such as:

```python
asyncio.Lock()
```

Example:

```python
lock = asyncio.Lock()


async def update_state():
    async with lock:
        # modify shared state
        ...
```

The lock ensures that only one coroutine at a time enters the protected section.

Use locks carefully. Excessive locking can reduce concurrency and create complicated code.

---

# 34. Queues

`asyncio.Queue` can be useful for producer-consumer systems.

Conceptually:

```text
Producer
   ↓
asyncio.Queue
   ↓
Consumer
```

Example applications:

* background AI jobs
* document ingestion
* embedding pipelines
* event processing
* agent task queues

Example:

```python
queue = asyncio.Queue()
```

Producer:

```python
await queue.put(document)
```

Consumer:

```python
document = await queue.get()
```

---

# 35. Production Insight

Asyncio is most valuable when your application spends a lot of time waiting.

For example:

```text
100 users
   ↓
FastAPI
   ↓
LLM API
   ↓
Vector DB
   ↓
Database
```

If every request blocks a worker while waiting for external services, your server may handle fewer concurrent requests than necessary.

Async I/O allows one worker to make progress on other requests while waiting.

The important architecture is:

```text
Many requests
      ↓
Async application
      ↓
External I/O
      ↓
Efficient concurrency
```

But async is not automatically faster.

If your workload is CPU-bound, async alone will not solve the problem.

---

# Common Mistakes

## Mistake 1: Thinking `async` automatically means parallel

This:

```python
async def task():
    ...
```

does not automatically execute tasks concurrently.

You need appropriate scheduling, such as:

```python
asyncio.gather()
```

or tasks.

---

## Mistake 2: Forgetting `await`

Incorrect:

```python
result = call_api()
```

when `call_api()` is asynchronous.

Usually:

```python
result = await call_api()
```

is required.

---

## Mistake 3: Blocking the event loop

Avoid:

```python
time.sleep(5)
```

inside async code.

Prefer:

```python
await asyncio.sleep(5)
```

when you genuinely need an asynchronous delay.

---

## Mistake 4: Starting unlimited tasks

This can be dangerous:

```python
tasks = [
    asyncio.create_task(call_api(item))
    for item in items
]
```

when `items` contains thousands of elements.

Use controlled concurrency with techniques such as:

```text
Semaphore
Queue
Batching
Rate limiting
```

---

## Mistake 5: Using async everywhere

Not every function needs to be asynchronous.

This:

```python
def calculate_score(a: float, b: float) -> float:
    return a * b
```

does not become better just because it is changed to:

```python
async def calculate_score(...):
```

Use async where asynchronous I/O or concurrency provides a meaningful benefit.

---

# Interview Questions

### Beginner

**1. What is asyncio?**

Python's standard library for writing asynchronous concurrent code using an event loop, coroutines, tasks, and `async`/`await`.

**2. What is a coroutine?**

An asynchronous function's execution object that can be awaited and scheduled by the event loop.

**3. What does `await` do?**

It suspends the current coroutine until the awaited asynchronous operation completes, allowing other async work to run.

**4. What does `asyncio.run()` do?**

It runs the top-level coroutine and manages the event loop for that execution.

---

### Intermediate

**5. What is the difference between sequential async code and concurrent async code?**

Calling:

```python
await task_a()
await task_b()
```

waits for A before starting B.

Using:

```python
await asyncio.gather(
    task_a(),
    task_b(),
)
```

allows independent operations to execute concurrently.

**6. What is an event loop?**

The mechanism that schedules and runs asynchronous tasks and resumes them when their awaited operations are ready.

**7. What is `asyncio.create_task()`?**

It schedules a coroutine as an asyncio task so it can execute concurrently with other async work.

**8. What is the difference between asyncio and multithreading?**

Asyncio uses cooperative concurrency around an event loop, while threads provide separate execution contexts. Asyncio is particularly effective for I/O-bound operations that have asynchronous APIs.

---

### AI Engineering

**9. Why is asyncio useful for LLM applications?**

LLM calls are network I/O operations and often involve significant waiting. Async execution allows the application to handle other work while waiting for responses.

**10. How can asyncio improve a RAG pipeline?**

Independent retrieval operations, embedding calls, or external API requests can potentially execute concurrently.

**11. How can asyncio be used in multi-agent systems?**

Independent agents can be scheduled concurrently and their results can later be combined by a coordinator.

**12. How do you prevent an AI application from overwhelming an LLM provider?**

Use controlled concurrency, rate limiting, batching, retries with backoff, and appropriate provider limits.

**13. What happens if you call blocking code inside an async function?**

It can block the event loop and prevent other coroutines from making progress.

---

# Practical Exercise

Build a small asynchronous retrieval simulation.

Create:

```python
async def search_vector_db(query: str):
    ...


async def search_documents(query: str):
    ...


async def search_web(query: str):
    ...
```

Make each function simulate network latency using:

```python
await asyncio.sleep(...)
```

Then execute them concurrently:

```python
results = await asyncio.gather(
    search_vector_db(query),
    search_documents(query),
    search_web(query),
)
```

Measure the difference between:

```text
Sequential execution
```

and:

```text
Concurrent execution
```

Then extend the exercise by adding:

```text
Semaphore
Timeout
Error handling
```

This will give you a practical understanding of the patterns used in production AI services.

---

# Summary

Remember:

```text
async def
    ↓
Define coroutine

await
    ↓
Wait without blocking the event loop

asyncio.run()
    ↓
Run top-level async application

create_task()
    ↓
Schedule concurrent work

gather()
    ↓
Wait for multiple operations

Semaphore
    ↓
Limit concurrency

Queue
    ↓
Producer / consumer workflows

Lock
    ↓
Coordinate shared state

async for
    ↓
Consume asynchronous streams
```

For AI engineering, the most important mental model is:

```text
LLM / API / DB / Vector DB
          ↓
       I/O wait
          ↓
       asyncio
          ↓
   Other work can run
```

And remember:

> **Asyncio is primarily a tool for efficiently handling I/O-bound concurrency. It does not make CPU-heavy Python code automatically faster.**
