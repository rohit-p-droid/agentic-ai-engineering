# Multiprocessing in Python for AI Engineering

## Why does an AI Engineer need this?

AI engineering is not only about calling LLM APIs.

Real AI systems also perform CPU-intensive work such as:

* Document parsing
* Image preprocessing
* Audio preprocessing
* Text normalization
* Large-scale data transformation
* Tokenization
* Feature extraction
* CPU-heavy evaluation
* Batch processing
* Data preparation
* File conversion
* Computationally expensive algorithms

For I/O-bound operations, threading and `asyncio` are often useful.

For CPU-bound Python workloads, **multiprocessing** can provide true parallel execution by running work in separate processes.

A simplified mental model is:

```text
Threading

Process
├── Thread 1
├── Thread 2
└── Thread 3

        ↓
Same Python interpreter
        ↓
GIL applies


Multiprocessing

Process 1 → Python interpreter
Process 2 → Python interpreter
Process 3 → Python interpreter

        ↓
Separate processes
        ↓
Can execute CPU-bound Python code in parallel
```

---

# 1. What is Multiprocessing?

Multiprocessing means running multiple independent processes concurrently.

Python provides the built-in:

```python
multiprocessing
```

module.

Each process has its own:

* Python interpreter
* Memory space
* Execution state
* Resources

Conceptually:

```text
Main Process
│
├── Worker Process 1
├── Worker Process 2
├── Worker Process 3
└── Worker Process 4
```

Unlike threads, these workers are not simply separate execution paths inside the same Python interpreter.

---

# 2. Why Multiprocessing Helps With the GIL

The GIL limits simultaneous execution of Python bytecode by threads within a standard CPython interpreter.

Consider:

```python
def expensive_computation():
    ...
```

With threads:

```text
Thread 1 ── CPU work
Thread 2 ── CPU work
Thread 3 ── CPU work

        ↓

GIL limits Python bytecode execution
```

With processes:

```text
Process 1 ── CPU core
Process 2 ── CPU core
Process 3 ── CPU core
```

Each process has its own Python interpreter and therefore its own GIL.

This allows CPU-bound Python workloads to execute in parallel across processes.

---

# 3. CPU-Bound Work

Multiprocessing is primarily useful for workloads where the CPU is the bottleneck.

Examples:

```text
Large document processing
        ↓
CPU computation
        ↓
CPU computation
        ↓
CPU computation
```

Examples in AI engineering include:

* Large-scale text preprocessing
* CPU-heavy parsing
* Image transformations
* Audio processing
* Data preprocessing
* Numerical computations
* CPU-heavy evaluation
* Batch transformations

However, always profile the actual workload before introducing multiprocessing.

---

# 4. Creating a Process

Basic example:

```python
from multiprocessing import Process


def process_document():
    print("Processing document")


process = Process(target=process_document)

process.start()
process.join()
```

`start()` starts the process.

`join()` waits for it to finish.

---

# 5. Passing Arguments

Arguments can be passed using `args`.

```python
from multiprocessing import Process


def process_document(document_id):
    print(f"Processing {document_id}")


process = Process(
    target=process_document,
    args=("doc-123",)
)

process.start()
process.join()
```

Multiple arguments:

```python
process = Process(
    target=process_document,
    args=("doc-123", "customer-456")
)
```

---

# 6. Multiple Processes

You can create several processes manually:

```python
from multiprocessing import Process


def process_document(document_id):
    print(f"Processing {document_id}")


processes = []

for document_id in range(4):
    process = Process(
        target=process_document,
        args=(document_id,)
    )

    process.start()
    processes.append(process)


for process in processes:
    process.join()
```

Conceptually:

```text
Main Process
     │
     ├── Process 1
     ├── Process 2
     ├── Process 3
     └── Process 4
```

However, manually creating many processes is usually not the best approach.

For batch workloads, use a process pool.

---

# 7. ProcessPoolExecutor

Python's:

```python
concurrent.futures.ProcessPoolExecutor
```

provides a convenient high-level interface.

Example:

```python
from concurrent.futures import ProcessPoolExecutor


def calculate(value):
    return value * value


values = [1, 2, 3, 4, 5]


with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(
        executor.map(calculate, values)
    )

print(results)
```

The executor manages the worker processes.

---

# 8. ThreadPoolExecutor vs ProcessPoolExecutor

The APIs look similar.

```python
ThreadPoolExecutor
```

uses threads.

```python
ProcessPoolExecutor
```

uses processes.

| Feature       | ThreadPoolExecutor      | ProcessPoolExecutor      |
| ------------- | ----------------------- | ------------------------ |
| Worker type   | Threads                 | Processes                |
| Memory        | Shared                  | Separate                 |
| GIL           | Shared                  | Separate interpreter/GIL |
| I/O-bound     | Excellent use case      | Usually unnecessary      |
| CPU-bound     | Limited for Python code | Strong use case          |
| Startup cost  | Lower                   | Higher                   |
| Data transfer | Easier                  | Serialization required   |
| Isolation     | Lower                   | Higher                   |

A useful rule:

```text
I/O-bound
    ↓
asyncio / threads

CPU-bound Python
    ↓
multiprocessing
```

This is a guideline, not an absolute rule.

---

# 9. Example: CPU-Heavy Document Processing

Imagine an ingestion system processing thousands of documents.

```text
Documents
    ↓
Text extraction
    ↓
Normalization
    ↓
CPU-heavy processing
    ↓
Chunks
    ↓
Embeddings
    ↓
Vector DB
```

If the normalization stage is CPU-heavy, multiprocessing can parallelize that stage.

```python
from concurrent.futures import ProcessPoolExecutor


def normalize_document(text):
    # CPU-heavy processing
    return text.lower().replace("\n", " ")


documents = load_documents()


with ProcessPoolExecutor(max_workers=4) as executor:
    normalized = list(
        executor.map(normalize_document, documents)
    )
```

The CPU-heavy work can be distributed across processes.

---

# 10. Multiprocessing in a RAG Pipeline

A RAG ingestion pipeline might look like:

```text
Documents
    ↓
Extraction
    ↓
Cleaning
    ↓
Chunking
    ↓
Embedding
    ↓
Vector Database
```

Not every stage should use multiprocessing.

For example:

```text
Cleaning
   ↓
CPU-bound
   ↓
Multiprocessing
```

while:

```text
Embedding API
   ↓
Network I/O
   ↓
Async / Threads
```

And:

```text
Vector DB
   ↓
Network I/O
   ↓
Async / Threads
```

This leads to an important architectural principle:

> Different stages of an AI pipeline may require different concurrency models.

---

# 11. Multiprocessing Does Not Automatically Make Everything Faster

Consider:

```python
ProcessPoolExecutor(max_workers=32)
```

This does not guarantee better performance.

Processes have overhead.

They require:

* Process creation
* Memory allocation
* Data serialization
* Inter-process communication
* Scheduling
* Context switching

If the task is tiny:

```text
Task takes 1 ms
Process overhead takes 5 ms
```

multiprocessing may actually make the application slower.

The workload needs enough computation to justify the overhead.

---

# 12. Process Pools

A process pool maintains a fixed number of worker processes.

```text
ProcessPoolExecutor
│
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4
```

Tasks are submitted to the pool.

```python
with ProcessPoolExecutor(max_workers=4) as executor:
    results = executor.map(process_document, documents)
```

This avoids creating a new process for every task.

---

# 13. Choosing `max_workers`

A common mistake is:

```python
max_workers=100
```

without considering the machine.

For CPU-heavy workloads, the number of workers is usually related to the available CPU cores.

Python provides:

```python
import os

print(os.cpu_count())
```

For example:

```text
CPU cores = 8
```

A reasonable starting point might be around the available CPU capacity, followed by benchmarking.

The exact number depends on:

* CPU architecture
* Memory
* Task characteristics
* Other processes
* Container limits
* Deployment environment

Do not assume:

```text
workers = CPU cores
```

is always optimal.

Measure it.

---

# 14. Data Between Processes

One major difference from threads is memory.

Threads share memory:

```text
Process
│
├── Thread A ──┐
├── Thread B ──┼── Shared memory
└── Thread C ──┘
```

Processes generally do not:

```text
Process A → Memory A

Process B → Memory B

Process C → Memory C
```

Therefore, data often needs to be transferred between processes.

Python commonly uses serialization mechanisms such as `pickle`.

Conceptually:

```text
Process A
   ↓
Serialize
   ↓
IPC
   ↓
Deserialize
   ↓
Process B
```

This has a cost.

---

# 15. Pickling

`ProcessPoolExecutor` generally needs function arguments and return values to be serializable between processes.

For example:

```python
def calculate(value):
    return value * 2
```

Works well.

But passing complex objects can cause problems.

Avoid designing process workers around:

* Open network connections
* Database connections
* File handles
* Active SDK clients
* Thread locks
* Non-serializable objects

Instead, pass simple data where possible.

```python
def process_document(text):
    ...
```

rather than:

```python
def process_document(database_client):
    ...
```

---

# 16. The `__main__` Guard

Multiprocessing requires special care, especially on platforms that use the `spawn` start method.

Use:

```python
if __name__ == "__main__":
    main()
```

Example:

```python
from multiprocessing import Process


def worker():
    print("Worker running")


def main():
    process = Process(target=worker)
    process.start()
    process.join()


if __name__ == "__main__":
    main()
```

This prevents unintended recursive process creation.

It is an important Python multiprocessing pattern.

---

# 17. Multiprocessing Start Methods

Python can create child processes using different start methods.

Common methods include:

```text
spawn
fork
forkserver
```

Their availability and behavior depend on the operating system and Python environment.

### `spawn`

Starts a fresh Python interpreter.

This is commonly used on Windows and has important implications for imports and initialization.

### `fork`

Creates a child process based on the parent process's memory state.

It is commonly associated with Unix-like systems.

However, forking applications that contain complex threaded runtimes, network clients, or other initialized resources can introduce problems.

### `forkserver`

Uses a server process to create child processes.

It can provide different isolation characteristics from direct forking.

The important engineering lesson is:

> Do not assume process creation behaves identically across operating systems.

---

# 18. Multiprocessing and AI SDK Clients

A common mistake is creating an LLM or database client globally and assuming it can safely be shared across processes.

For example:

```python
client = SomeLLMClient(...)

with ProcessPoolExecutor() as executor:
    ...
```

The client may contain:

* Network connections
* Connection pools
* Locks
* Threads
* Native resources

These resources should not automatically be assumed to be safe to share across process boundaries.

A safer pattern is often to initialize process-local resources inside the worker.

Conceptually:

```python
def worker(text):
    client = create_client()
    return client.process(text)
```

Whether this is appropriate depends on the SDK and workload.

---

# 19. CPU Processing + LLM API Calls

Consider this pipeline:

```text
Document
   ↓
CPU-heavy parsing
   ↓
Multiprocessing
   ↓
Chunks
   ↓
Embedding API
   ↓
Async / Threads
   ↓
Vector DB
```

This is a realistic hybrid architecture.

The key is to classify each stage.

```text
CPU-heavy
    → processes

Network-bound
    → async / threads

GPU-heavy
    → GPU framework/runtime
```

One concurrency model does not need to control the entire application.

---

# 20. Inter-Process Communication

Processes sometimes need to communicate.

Python provides several mechanisms.

Common options include:

* `Queue`
* `Pipe`
* `Value`
* `Array`
* `Manager`
* Shared memory

For example:

```python
from multiprocessing import Queue
```

Conceptually:

```text
Process A
   ↓
 Queue
   ↓
Process B
```

---

# 21. Multiprocessing Queue

A queue can transfer data between processes.

```python
from multiprocessing import Process, Queue


def worker(queue):
    queue.put("processing complete")


if __name__ == "__main__":
    queue = Queue()

    process = Process(
        target=worker,
        args=(queue,)
    )

    process.start()

    result = queue.get()

    process.join()

    print(result)
```

This is useful for producer-consumer architectures.

---

# 22. Shared State

Python also provides mechanisms for sharing state between processes.

For example:

```python
from multiprocessing import Value
```

However, shared state between processes introduces synchronization complexity.

Whenever possible, prefer:

```text
Input
  ↓
Worker
  ↓
Output
```

over heavily shared mutable state.

Stateless worker functions are easier to reason about and scale.

---

# 23. Multiprocessing and Web Applications

Be careful when using multiprocessing directly inside a web request.

For example:

```text
HTTP Request
     ↓
Create processes
     ↓
Perform CPU work
     ↓
Return response
```

This can be acceptable for small controlled workloads, but long-running processing is often better moved to a background worker architecture.

For production systems, consider:

```text
API
 ↓
Job Queue
 ↓
Worker Processes
 ↓
Result Store
```

This separates request handling from heavy computation.

---

# 24. Background Job Architecture

A scalable AI processing system might look like:

```text
             ┌───────────────┐
User ───────→│ API Service   │
             └───────┬───────┘
                     │
                     ↓
                Job Queue
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Worker 1   Worker 2   Worker 3
          │          │          │
          └──────────┼──────────┘
                     ↓
                Result Store
```

Possible technologies include:

* Celery
* RQ
* Dramatiq
* Task queues
* Kubernetes Jobs
* Cloud-managed queues

The exact choice depends on the application.

Multiprocessing is often a building block inside a worker rather than the entire production job architecture.

---

# 25. Multiprocessing vs Distributed Workers

These are different concepts.

### Multiprocessing

Runs multiple processes on the same machine.

```text
One machine
├── Process 1
├── Process 2
├── Process 3
└── Process 4
```

### Distributed workers

Run workers across multiple machines or containers.

```text
Machine A → Worker
Machine B → Worker
Machine C → Worker
```

For very large AI workloads, distributed workers may be more appropriate than relying on multiprocessing on one machine.

---

# 26. Multiprocessing and GPU Workloads

Do not assume that multiprocessing is the correct way to parallelize GPU workloads.

Modern AI workloads often use specialized frameworks and runtimes for GPU execution.

Examples include:

* PyTorch
* TensorFlow
* CUDA-based systems
* Inference servers
* Distributed GPU frameworks

The architecture may instead look like:

```text
Application
    ↓
Inference Service
    ↓
GPU
```

or:

```text
Workers
   ↓
GPU runtime
   ↓
Model
```

GPU resource management has different constraints from CPU multiprocessing.

---

# 27. Common Mistakes

## Mistake 1: Using multiprocessing for network I/O

If the workload is:

```text
HTTP request
    ↓
WAIT
    ↓
HTTP response
```

multiprocessing may introduce unnecessary overhead.

Consider `asyncio` or threads.

---

## Mistake 2: Passing huge objects between processes

Large objects may need to be serialized and copied.

For example:

```python
executor.map(process_document, huge_documents)
```

may have substantial memory and serialization overhead.

Consider:

* Smaller batches
* Streaming
* Shared memory where appropriate
* External storage
* Passing identifiers instead of huge objects

---

## Mistake 3: Sharing database connections

A database connection created in the parent process should not automatically be reused by child processes.

Create process-local connections according to the database client's recommended model.

---

## Mistake 4: Sharing LLM clients blindly

SDK clients may contain network state or other resources that are not designed for cross-process reuse.

Follow the SDK's process-safety guidance.

---

## Mistake 5: Creating too many processes

Processes consume significantly more resources than lightweight tasks.

Bound concurrency.

---

## Mistake 6: Ignoring memory usage

Suppose a process loads:

```text
500 MB model/data
```

and you run:

```text
8 processes
```

Memory requirements can become substantial.

Process isolation has a resource cost.

---

## Mistake 7: Forgetting the `__main__` guard

Especially on platforms using `spawn`, incorrect process initialization can cause recursive process creation or other failures.

---

# Production Insight

The biggest mistake is thinking:

> "Multiprocessing = faster."

The correct question is:

> "Is my workload CPU-bound enough that process-level parallelism justifies its overhead?"

For example:

```text
Task A: 5 seconds CPU work
Task B: 5 seconds CPU work
Task C: 5 seconds CPU work
Task D: 5 seconds CPU work
```

Sequential:

```text
≈ 20 seconds
```

With suitable parallelism on a machine with enough CPU capacity:

```text
≈ 5+ seconds
```

The actual result depends on:

* Number of CPU cores
* Process overhead
* Serialization
* Memory bandwidth
* Operating system
* Task size
* Other workloads

Benchmark the real workload.

---

# How It's Used in AI Engineering

Multiprocessing is especially useful around AI workloads rather than necessarily inside the model itself.

Common examples:

```text
Large dataset
     ↓
CPU preprocessing
     ↓
Multiprocessing
     ↓
Prepared data
     ↓
Model / embedding service
```

Another example:

```text
Documents
     ↓
CPU-heavy extraction
     ↓
Process pool
     ↓
Clean text
     ↓
Async embedding calls
     ↓
Vector DB
```

This hybrid architecture is often more appropriate than trying to solve every concurrency problem with one technology.

---

# Practical Exercise

Build a CPU-intensive document processor.

Create:

```text
documents/
├── doc1.txt
├── doc2.txt
├── doc3.txt
└── doc4.txt
```

For every document:

1. Read the document.
2. Normalize the text.
3. Perform a CPU-intensive transformation.
4. Calculate statistics.
5. Return a structured result.

First implement it sequentially:

```python
results = [
    process_document(document)
    for document in documents
]
```

Then implement it using:

```python
ProcessPoolExecutor
```

Measure:

```text
Sequential execution time
Multiprocessing execution time
```

Then experiment with:

```text
max_workers = 1
max_workers = 2
max_workers = 4
max_workers = 8
```

Compare the results.

The purpose is not simply to find the largest worker count.

The goal is to understand how:

```text
CPU cores
+
task size
+
process overhead
+
memory
```

affect performance.

---

# Interview Questions

## Beginner

1. What is multiprocessing?
2. What is the difference between a process and a thread?
3. Why does multiprocessing help with CPU-bound Python workloads?
4. What is the GIL?
5. What is `Process`?
6. What is `ProcessPoolExecutor`?
7. What does `join()` do?
8. Why is the `__main__` guard important?
9. What is a process pool?
10. What is inter-process communication?

## Intermediate

11. Why is multiprocessing more expensive than threading?
12. What is serialization/pickling?
13. Why can't you freely share Python objects between processes?
14. What is a race condition between processes?
15. How can processes communicate?
16. What are multiprocessing queues?
17. What are multiprocessing start methods?
18. What is the difference between `spawn` and `fork`?
19. Why can large objects make multiprocessing inefficient?
20. How do you choose the number of worker processes?

## AI Engineering

21. When would you use multiprocessing in a RAG pipeline?
22. Which stages of a RAG pipeline are likely to be CPU-bound?
23. Why would you use multiprocessing for preprocessing but `asyncio` for embeddings?
24. How would you design a CPU-heavy document processing service?
25. How would you handle an LLM client inside multiprocessing workers?
26. Why should database connections generally be process-local?
27. How does multiprocessing differ from distributed workers?
28. When would a task queue be preferable to creating processes inside an API request?
29. How would you process millions of documents using CPU workers and an embedding API?
30. How would you benchmark whether multiprocessing actually improves an AI pipeline?

---

# Key Takeaways

* Multiprocessing runs work in separate processes.
* Each process has its own Python interpreter and memory space.
* It can provide true parallelism for CPU-bound Python workloads.
* `ProcessPoolExecutor` is a convenient way to manage process workers.
* Processes have more overhead than threads.
* Data passed between processes generally needs serialization.
* Large objects can make multiprocessing expensive.
* Use bounded worker counts.
* The `__main__` guard is important for safe multiprocessing programs.
* Be careful with database connections, SDK clients, files, and other process-sensitive resources.
* Multiprocessing is different from distributed processing.
* CPU-heavy pipeline stages may benefit from multiprocessing.
* Network-bound stages are often better handled with `asyncio` or threading.
* GPU workloads generally require specialized GPU execution strategies.
* Production AI systems often combine multiple concurrency models.

---

# Summary

Multiprocessing is a powerful tool for **CPU-bound Python workloads**.

The key distinction is:

```text
I/O-bound
    ↓
asyncio / threading


CPU-bound Python
    ↓
multiprocessing


GPU-heavy AI
    ↓
GPU frameworks / inference runtimes


Large-scale distributed workloads
    ↓
Distributed workers / job systems
```

For an AI engineer, multiprocessing becomes especially useful when building data and document-processing pipelines.

A production-oriented pipeline might look like:

```text
Documents
    ↓
CPU-heavy preprocessing
    ↓
Process Pool
    ↓
Clean / structured data
    ↓
Async embedding requests
    ↓
Vector Database
```

The important skill is not simply knowing how to create processes.

It is knowing **where process-level parallelism belongs in an AI system, when its overhead is justified, and how to combine it with other concurrency models safely.**
