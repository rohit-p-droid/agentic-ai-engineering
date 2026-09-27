# Generators in Python for AI Engineering

## Why does an AI Engineer need this?

AI applications frequently process large amounts of data:

* Documents
* PDF pages
* Text chunks
* Database rows
* API responses
* Logs
* Search results
* Tokens
* Embeddings
* Evaluation records
* Streaming LLM responses

A common problem is loading everything into memory at once.

For example:

```python
documents = load_all_documents()
```

If there are thousands or millions of documents, this can consume significant memory.

Generators provide a way to process data **lazily**.

Instead of producing all results immediately, a generator produces one result at a time when requested.

```text
Normal function

Input
  ↓
Process everything
  ↓
Return everything
  ↓
Memory: potentially large


Generator

Input
  ↓
Process one item
  ↓
yield
  ↓
Process next item
  ↓
yield
  ↓
...
```

Generators are particularly useful for building **memory-efficient AI data pipelines**.

---

# 1. What is a Generator?

A generator is a special type of iterator that produces values lazily.

Instead of:

```python
return
```

a generator function uses:

```python
yield
```

Example:

```python
def numbers():
    yield 1
    yield 2
    yield 3
```

You can consume it with:

```python
for number in numbers():
    print(number)
```

Output:

```text
1
2
3
```

The generator does not create all values at once.

---

# 2. `yield` vs `return`

Consider:

```python
def get_numbers():
    return [1, 2, 3, 4, 5]
```

The function creates the complete list before returning it.

With a generator:

```python
def get_numbers():
    yield 1
    yield 2
    yield 3
    yield 4
    yield 5
```

Values are produced one at a time.

```text
yield 1
   ↓
consumer receives 1

yield 2
   ↓
consumer receives 2

yield 3
   ↓
consumer receives 3
```

---

# 3. Generator Functions

Any function containing `yield` becomes a generator function.

```python
def document_ids():
    for i in range(5):
        yield f"doc-{i}"
```

Calling it:

```python
documents = document_ids()
```

does not execute the entire function immediately.

```python
print(documents)
```

returns a generator object.

To consume it:

```python
for document in documents:
    print(document)
```

---

# 4. Lazy Evaluation

Generators use **lazy evaluation**.

Consider:

```python
def generate_documents():
    for i in range(1_000_000):
        yield f"document-{i}"
```

This does not create one million strings immediately.

Instead:

```text
Request next item
      ↓
Generate one item
      ↓
Return it
      ↓
Wait
      ↓
Request next item
      ↓
Generate next item
```

This can dramatically reduce memory usage.

---

# 5. Memory Efficiency

Compare a list:

```python
def get_chunks():
    return [
        chunk
        for chunk in generate_chunks()
    ]
```

with a generator:

```python
def get_chunks():
    for chunk in generate_chunks():
        yield chunk
```

The list stores all results.

The generator keeps only the current execution state and the current value in memory, plus whatever objects your pipeline itself retains.

Conceptually:

```text
List

[chunk1, chunk2, chunk3, ..., chunk100000]
              ↓
       Large memory usage


Generator

chunk1 → process → discard
chunk2 → process → discard
chunk3 → process → discard
```

---

# 6. Generators for Large Documents

Suppose you have a large collection of documents.

Instead of:

```python
def load_documents(path):
    return list(path.glob("*.md"))
```

you can write:

```python
from pathlib import Path


def load_documents(path):
    for file_path in Path(path).glob("*.md"):
        yield file_path
```

Then:

```python
for document in load_documents("documents"):
    print(document)
```

Only the next file path needs to be produced at a time.

---

# 7. Generators in RAG Pipelines

Generators are especially useful in RAG ingestion.

Consider:

```text
Documents
    ↓
Extract
    ↓
Clean
    ↓
Chunk
    ↓
Embed
    ↓
Store
```

Instead of loading everything:

```text
10,000 documents
      ↓
10,000 documents in memory
```

you can create a streaming pipeline:

```text
Document 1
    ↓
Chunks
    ↓
Embedding
    ↓
Vector DB

Document 2
    ↓
Chunks
    ↓
Embedding
    ↓
Vector DB

...
```

Example:

```python
def load_documents(paths):
    for path in paths:
        yield read_document(path)


def create_chunks(documents):
    for document in documents:
        for chunk in chunk_document(document):
            yield chunk


documents = load_documents(paths)
chunks = create_chunks(documents)

for chunk in chunks:
    embedding = create_embedding(chunk)
    store_embedding(embedding)
```

The stages form a lazy pipeline.

---

# 8. Generator Pipelines

One of the most useful patterns is chaining generators.

```python
def load_documents(paths):
    for path in paths:
        yield read_document(path)


def clean_documents(documents):
    for document in documents:
        yield clean(document)


def chunk_documents(documents):
    for document in documents:
        yield from chunk(document)
```

Then:

```python
pipeline = chunk_documents(
    clean_documents(
        load_documents(paths)
    )
)
```

Finally:

```python
for chunk in pipeline:
    process_chunk(chunk)
```

Conceptually:

```text
load
  ↓
clean
  ↓
chunk
  ↓
process
```

Each stage produces data only when the next stage requests it.

---

# 9. `yield from`

`yield from` allows one generator to yield values from another iterable.

Without `yield from`:

```python
def chunks():
    for chunk in create_chunks():
        yield chunk
```

With `yield from`:

```python
def chunks():
    yield from create_chunks()
```

Both can produce the same values.

It is especially useful when composing generators.

---

# 10. Generator Expressions

Python also provides generator expressions.

List comprehension:

```python
squares = [
    x * x
    for x in range(1_000_000)
]
```

Generator expression:

```python
squares = (
    x * x
    for x in range(1_000_000)
)
```

The list creates all values immediately.

The generator expression produces them lazily.

```python
for square in squares:
    process(square)
```

---

# 11. Generator vs List

| Feature        | List              | Generator                 |
| -------------- | ----------------- | ------------------------- |
| Evaluation     | Eager             | Lazy                      |
| Memory         | Stores all values | Produces values on demand |
| Reusable       | Usually yes       | Usually one-pass          |
| Indexing       | Yes               | No direct indexing        |
| Length         | `len()` available | Not generally available   |
| Large datasets | Can be expensive  | Often efficient           |
| Streaming      | No                | Excellent                 |

---

# 12. Generators Are Usually One-Pass

Consider:

```python
numbers = (x for x in range(5))
```

Consume it:

```python
for number in numbers:
    print(number)
```

If you try again:

```python
for number in numbers:
    print(number)
```

there may be nothing left.

The generator has already been exhausted.

```text
Generator
   ↓
1
2
3
4
5
   ↓
Exhausted
```

If you need to iterate multiple times, consider whether a list or another reusable data structure is more appropriate.

---

# 13. `next()`

You can manually request the next value from a generator.

```python
def numbers():
    yield 10
    yield 20
    yield 30


generator = numbers()

print(next(generator))
print(next(generator))
print(next(generator))
```

Output:

```text
10
20
30
```

Calling `next()` after exhaustion raises:

```python
StopIteration
```

A `for` loop handles this automatically.

---

# 14. How Generators Preserve State

A generator function pauses at `yield`.

Example:

```python
def numbers():
    print("Start")
    yield 1

    print("Continue")
    yield 2

    print("Finish")
```

When you call:

```python
generator = numbers()
```

the function does not execute fully.

Then:

```python
next(generator)
```

executes until the first `yield`.

The generator remembers where it stopped.

```text
Start
  ↓
yield 1
  ↓
pause
  ↓
next()
  ↓
Continue
  ↓
yield 2
```

This preserved execution state is a key property of generators.

---

# 15. Generator Pipelines and Backpressure

Generators can naturally support a form of pull-based processing.

Consider:

```text
Consumer
   ↑
Chunk processor
   ↑
Chunk generator
   ↑
Document loader
```

The consumer asks for the next item.

That request propagates backward through the pipeline.

```text
Need next chunk
      ↓
Create chunk
      ↓
Load/process document
      ↓
Return chunk
```

This means upstream stages do not necessarily need to produce data faster than downstream stages consume it.

This is useful when processing large datasets.

---

# 16. Streaming File Processing

Generators are useful for large files.

Instead of:

```python
with open("large.log") as file:
    content = file.read()
```

which loads the entire file into memory, you can process lines incrementally:

```python
def read_lines(path):
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            yield line
```

Then:

```python
for line in read_lines("large.log"):
    process(line)
```

This is useful for:

* Logs
* CSV files
* JSON Lines
* Large text files
* Dataset preprocessing

---

# 17. Generators for JSON Lines

JSONL files are common in AI datasets.

Example:

```text
{"text": "RAG retrieves relevant context."}
{"text": "Agents can call tools."}
{"text": "Embeddings represent semantic information."}
```

You can process them lazily:

```python
import json


def read_jsonl(path):
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            yield json.loads(line)
```

Then:

```python
for record in read_jsonl("dataset.jsonl"):
    process_record(record)
```

This avoids loading the entire dataset into memory.

---

# 18. Generators for Database Results

For large datasets, applications often need to avoid loading every row into memory.

A conceptual pattern is:

```python
def fetch_records(cursor):
    for row in cursor:
        yield row
```

Then:

```python
for row in fetch_records(cursor):
    process(row)
```

Whether the underlying database driver actually streams efficiently depends on the driver and query configuration.

The generator itself does not magically make a database query memory-efficient.

This distinction is important.

---

# 19. Generator-Based Document Chunking

A chunking function is a good generator use case.

```python
def chunk_text(text, chunk_size=500):
    for start in range(0, len(text), chunk_size):
        yield text[start:start + chunk_size]
```

Usage:

```python
for chunk in chunk_text(document):
    create_embedding(chunk)
```

Instead of:

```python
chunks = chunk_text(document)
```

where all chunks might be stored, the generator can produce one chunk at a time.

---

# 20. Generator-Based Embedding Pipeline

Consider:

```python
def generate_embeddings(chunks):
    for chunk in chunks:
        embedding = embedding_client.embed(chunk)
        yield {
            "text": chunk,
            "embedding": embedding,
        }
```

Then:

```python
for item in generate_embeddings(chunks):
    vector_db.insert(item)
```

This creates a streaming pipeline:

```text
Chunk
 ↓
Embedding
 ↓
Vector DB
 ↓
Next Chunk
 ↓
Embedding
 ↓
Vector DB
```

For large ingestion jobs, this can significantly reduce the amount of intermediate data held in memory.

---

# 21. Generator-Based Evaluation

Generators are also useful for evaluation pipelines.

```python
def evaluate_documents(documents):
    for document in documents:
        prediction = generate_answer(document)
        score = evaluate(prediction)

        yield {
            "document": document,
            "score": score,
        }
```

Then:

```python
for result in evaluate_documents(documents):
    save_result(result)
```

This allows long-running evaluations to process records incrementally.

---

# 22. Generators and Streaming LLM Responses

Many LLM systems support streaming responses.

Conceptually:

```text
LLM
 ↓
Token/chunk
 ↓
Token/chunk
 ↓
Token/chunk
 ↓
Final response
```

Python generators can represent this kind of incremental consumption:

```python
def stream_response():
    for chunk in llm_stream():
        yield chunk
```

Consumer:

```python
for chunk in stream_response():
    print(chunk, end="")
```

However, actual LLM SDKs may expose streaming through generators, async generators, callbacks, iterators, or provider-specific abstractions.

Always check the SDK's current API.

---

# 23. Generators vs Async Generators

A normal generator:

```python
def stream():
    yield value
```

An async generator:

```python
async def stream():
    yield value
```

Consume a normal generator with:

```python
for item in stream():
    ...
```

Consume an async generator with:

```python
async for item in stream():
    ...
```

Async generators are particularly useful when each yielded item may involve asynchronous I/O.

Example:

```python
async def stream_embeddings(chunks):
    for chunk in chunks:
        embedding = await create_embedding(chunk)
        yield embedding
```

---

# 24. Generator vs Async Generator

| Feature     | Generator            | Async Generator      |
| ----------- | -------------------- | -------------------- |
| Definition  | `def`                | `async def`          |
| Yield       | `yield`              | `yield`              |
| Consumption | `for`                | `async for`          |
| I/O         | Blocking/synchronous | Async                |
| Common use  | Streaming data       | Streaming async data |
| AI example  | Sync LLM streaming   | Async LLM streaming  |

---

# 25. Generators and Memory Efficiency

A common misconception is:

> "Generators always make programs faster."

That is not necessarily true.

Generators primarily help with:

* Memory efficiency
* Lazy evaluation
* Streaming
* Pipeline composition

They can sometimes be slower than list operations because values are generated incrementally.

The main benefit is often:

```text
Lower peak memory
+
Incremental processing
+
Better streaming behavior
```

rather than raw CPU speed.

---

# 26. When Not to Use Generators

Do not automatically use generators everywhere.

A list may be better when:

* You need random access.
* You need `len()`.
* You need to iterate multiple times.
* The dataset is small.
* You need to sort the complete dataset.
* You need to retain all results.
* Recomputing values would be expensive.

Example:

```python
documents = load_documents()
```

may be perfectly reasonable when there are only 20 documents.

Use the simplest appropriate data structure.

---

# 27. Common Mistakes

## Mistake 1: Thinking generators store all values

They do not.

Generators produce values lazily.

---

## Mistake 2: Trying to index a generator

This does not work:

```python
generator[0]
```

If you need indexing, use a list or another indexed structure.

---

## Mistake 3: Forgetting that generators are one-pass

After exhaustion:

```python
for item in generator:
    ...
```

a second iteration may produce nothing.

---

## Mistake 4: Converting the generator to a list unnecessarily

This:

```python
items = list(generator)
```

removes much of the memory benefit.

Only do it when you actually need all values at once.

---

## Mistake 5: Assuming a generator makes database queries streaming

The generator controls Python-side consumption.

The database driver and query configuration determine how much database data is actually buffered.

---

## Mistake 6: Performing expensive work before yielding

For example:

```python
def process():
    results = expensive_process_everything()

    for result in results:
        yield result
```

This may still perform the entire expensive operation before the first value is yielded.

A genuinely streaming pipeline should perform work incrementally.

---

# Production Insight

Generators are especially valuable at **data boundaries**.

Instead of designing:

```text
Load everything
     ↓
Process everything
     ↓
Store everything
```

you can design:

```text
Load one
   ↓
Process one
   ↓
Store one
   ↓
Load next
```

For example:

```text
Large Document Dataset
        ↓
Generator
        ↓
Document
        ↓
Chunk Generator
        ↓
Chunk
        ↓
Embedding
        ↓
Vector DB
```

This creates a pipeline where memory usage does not necessarily grow with the total dataset size.

However, production systems still need to consider:

* Batching
* API limits
* Database throughput
* Backpressure
* Retries
* Checkpointing
* Failure recovery
* Parallelism

Generators solve the **lazy data production** problem; they do not solve the entire distributed processing problem.

---

# How It's Used in AI Frameworks

Generators appear naturally in AI engineering abstractions such as:

* Streaming responses
* Dataset iteration
* Document loaders
* Batch processing
* Evaluation pipelines
* Token streams
* Retrieval results

Frameworks may expose these concepts through:

```text
Iterator
Generator
AsyncIterator
AsyncGenerator
StreamingResponse
Callback
```

Do not assume every framework uses Python generators directly.

The important concept is **incremental consumption of results**.

---

# Practical Exercise

Build a memory-efficient RAG ingestion pipeline using generators.

Create:

```text
documents/
├── doc1.md
├── doc2.md
├── doc3.md
└── doc4.md
```

Implement:

### Step 1 — Document loader

```python
def load_documents(paths):
    ...
```

Yield one document at a time.

### Step 2 — Cleaner

```python
def clean_documents(documents):
    ...
```

Yield cleaned documents.

### Step 3 — Chunker

```python
def chunk_documents(documents):
    ...
```

Yield one chunk at a time.

### Step 4 — Embedding stage

```python
def generate_embeddings(chunks):
    ...
```

Yield embedding records.

### Step 5 — Storage

```python
for item in generate_embeddings(
    chunk_documents(
        clean_documents(
            load_documents(paths)
        )
    )
):
    store(item)
```

Then compare this architecture with:

```python
documents = load_all()
cleaned = clean_all(documents)
chunks = chunk_all(cleaned)
embeddings = embed_all(chunks)
store_all(embeddings)
```

Think about:

* Peak memory usage
* Streaming
* Failure handling
* Retry behavior
* API rate limits
* Batch sizes

---

# Interview Questions

## Beginner

1. What is a generator?
2. What is the difference between `yield` and `return`?
3. What is lazy evaluation?
4. How do you create a generator?
5. How do you consume a generator?
6. What does `next()` do?
7. What is `StopIteration`?
8. What is a generator expression?
9. What is `yield from`?
10. Why are generators memory efficient?

## Intermediate

11. How does a generator preserve execution state?
12. Why are generators usually one-pass?
13. What happens when a generator is exhausted?
14. What is the difference between an iterator and a generator?
15. When would you use a list instead of a generator?
16. Can a generator be indexed?
17. What happens when you convert a generator to a list?
18. How can generators be chained together?
19. What is an async generator?
20. What is the difference between a generator and an async generator?

## AI Engineering

21. How would you use generators in a RAG ingestion pipeline?
22. Why are generators useful for large document datasets?
23. How can generators reduce memory usage during embedding generation?
24. How can generators be used for streaming LLM responses?
25. How would you build a generator-based document loader?
26. How would you combine generators with batching?
27. What is the difference between lazy Python processing and true database streaming?
28. How would you handle failures in a long-running generator pipeline?
29. When would you use an async generator instead of a normal generator?
30. Design a memory-efficient pipeline for processing millions of documents.

---

# Key Takeaways

* Generators produce values lazily.
* `yield` turns a function into a generator function.
* Generators can significantly reduce peak memory usage.
* They are useful for large datasets and streaming pipelines.
* Generators are generally consumed one value at a time.
* Most generators are one-pass and become exhausted.
* `yield from` makes generator composition easier.
* Generator expressions provide lazy alternatives to list comprehensions.
* Generators are useful for RAG ingestion, document processing, evaluation, and streaming.
* Async generators combine lazy production with asynchronous operations.
* Generators do not automatically make database operations streaming.
* Generators do not automatically make code faster.
* They are most valuable when data can be processed incrementally.
* Production pipelines may combine generators with batching, concurrency, queues, and checkpoints.

---

# Summary

Generators are a fundamental Python tool for building **memory-efficient and streaming data pipelines**.

The core idea is simple:

```text
Don't produce everything at once.

Produce:
    one item
    ↓
    process
    ↓
    next item
    ↓
    process
    ↓
    ...
```

For AI engineering, this becomes particularly useful when dealing with:

```text
Large documents
      ↓
Document loader
      ↓
Chunk generator
      ↓
Embedding generator
      ↓
Vector DB
```

The important distinction is:

```text
List
    → eager
    → stores results


Generator
    → lazy
    → produces results on demand


Async Generator
    → lazy
    → supports asynchronous operations
```

Learning generators gives you an important building block for designing AI systems that process large datasets and streaming outputs without unnecessarily keeping everything in memory.
