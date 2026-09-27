# Decorators in Python for AI Engineering

## Why does an AI Engineer need this?

Decorators are a Python feature that allows you to **modify or extend the behavior of a function or class without changing its core implementation**.

They are heavily used in modern Python applications and AI frameworks.

You will encounter decorators when working with:

* FastAPI
* Flask
* Django
* LangChain integrations
* Testing frameworks
* Retry systems
* Logging
* Caching
* Authentication
* Authorization
* Monitoring
* Metrics
* Background jobs
* Validation

For an AI engineer, decorators are especially useful for implementing **cross-cutting concerns**.

For example:

```text id="q0q6wb"
LLM Function
     │
     ├── Logging
     ├── Timing
     ├── Retry
     ├── Metrics
     └── Error handling
```

Instead of putting all of that logic inside the LLM function, decorators can keep the actual business logic focused.

---

# 1. What is a Decorator?

A decorator is a function that takes another function and returns a modified function.

Conceptually:

```text id="x6v8n8"
Original Function
       ↓
    Decorator
       ↓
Modified Function
```

Example:

```python id="4p6e2k"
def decorator(function):
    def wrapper():
        print("Before")
        function()
        print("After")

    return wrapper
```

Apply it:

```python id="8f8h7c"
@decorator
def greet():
    print("Hello")
```

Calling:

```python id="0g6k0x"
greet()
```

produces:

```text id="v9d9fi"
Before
Hello
After
```

---

# 2. The `@` Syntax

This:

```python id="c5m2p0"
@decorator
def greet():
    print("Hello")
```

is essentially equivalent to:

```python id="n5s7yo"
def greet():
    print("Hello")


greet = decorator(greet)
```

The `@` syntax is simply a convenient way to apply a decorator.

---

# 3. Basic Decorator Structure

A typical decorator looks like:

```python id="7ibp6p"
def decorator(function):

    def wrapper(*args, **kwargs):
        # Before
        result = function(*args, **kwargs)
        # After

        return result

    return wrapper
```

There are three important parts:

```text id="1m0i8x"
decorator(function)
       ↓
wrapper(*args, **kwargs)
       ↓
function(*args, **kwargs)
```

`*args` and `**kwargs` allow the decorator to work with functions having different arguments.

---

# 4. Why `*args` and `**kwargs` Matter

Suppose the decorated function accepts arguments:

```python id="qpmk7w"
@decorator
def generate_answer(query, context):
    ...
```

The wrapper needs to forward them:

```python id="f5v8bq"
def wrapper(*args, **kwargs):
    return function(*args, **kwargs)
```

This allows:

```python id="v1i7hh"
generate_answer(
    query="What is RAG?",
    context="RAG retrieves external information."
)
```

to work correctly.

---

# 5. A Logging Decorator

Logging is one of the most practical decorator use cases.

```python id="x8b9r6"
def log_call(function):

    def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")

        result = function(*args, **kwargs)

        print(f"Finished {function.__name__}")

        return result

    return wrapper
```

Use it:

```python id="h5e9fu"
@log_call
def generate_answer(query):
    return f"Answer for {query}"
```

Now:

```python id="y0v5r3"
generate_answer("What is RAG?")
```

produces logging around the function.

In production, use the `logging` module rather than `print()`.

---

# 6. Preserving Function Metadata

A decorator can accidentally change metadata such as:

```python id="qf8z2b"
__name__
__doc__
```

For example:

```python id="3x8y4n"
print(generate_answer.__name__)
```

Without proper handling, you may see:

```text id="c6xk2v"
wrapper
```

instead of:

```text id="bq1w9s"
generate_answer
```

Python provides:

```python id="z9gq1n"
functools.wraps
```

Use it:

```python id="j8qv7s"
from functools import wraps


def log_call(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")

        result = function(*args, **kwargs)

        print(f"Finished {function.__name__}")

        return result

    return wrapper
```

This is a best practice.

---

# 7. Timing Decorator

AI operations can be expensive and slow.

For example:

* LLM calls
* Embedding generation
* Retrieval
* Document processing
* Database queries

A timing decorator can measure execution time.

```python id="m1o8ku"
import time
from functools import wraps


def measure_time(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()

        result = function(*args, **kwargs)

        duration = time.perf_counter() - start

        print(
            f"{function.__name__} took {duration:.3f}s"
        )

        return result

    return wrapper
```

Use:

```python id="c9a8px"
@measure_time
def retrieve_documents(query):
    ...
```

This can help identify latency bottlenecks.

---

# 8. Timing an AI Pipeline

You could decorate different stages:

```python id="q4k5nm"
@measure_time
def retrieve_documents(query):
    ...


@measure_time
def generate_answer(context):
    ...


@measure_time
def store_result(result):
    ...
```

This can reveal:

```text id="8z3qsl"
retrieve_documents → 120 ms
generate_answer    → 1800 ms
store_result       → 80 ms
```

The application can then focus optimization efforts where they matter.

---

# 9. Retry Decorator

External services can fail temporarily.

For example:

```text id="glf4b2"
LLM API
   ↓
Temporary network error
   ↓
Retry
   ↓
Success
```

A simple retry decorator:

```python id="krz1h4"
import time
from functools import wraps


def retry(attempts=3, delay=1):

    def decorator(function):

        @wraps(function)
        def wrapper(*args, **kwargs):

            for attempt in range(attempts):
                try:
                    return function(*args, **kwargs)

                except Exception:
                    if attempt == attempts - 1:
                        raise

                    time.sleep(delay)

        return wrapper

    return decorator
```

Usage:

```python id="l1f7c4"
@retry(attempts=3, delay=2)
def call_llm():
    ...
```

---

# 10. Why Blind Retries Are Dangerous

This is important in AI systems.

Not every error should be retried.

For example:

```text id="u4f4g6"
Authentication Error
    ↓
Retrying won't fix credentials


Invalid Request
    ↓
Retrying won't fix invalid input


Rate Limit
    ↓
Retry may help if backoff is appropriate


Temporary Network Error
    ↓
Retry may help
```

Production retry systems should consider:

* Error type
* Maximum attempts
* Exponential backoff
* Jitter
* Timeout
* Rate limits
* Idempotency

Avoid:

```python id="2vax44"
except Exception:
    retry_forever()
```

---

# 11. Exponential Backoff

Instead of retrying immediately:

```text id="8j6x5g"
Attempt 1 → wait 1s
Attempt 2 → wait 2s
Attempt 3 → wait 4s
Attempt 4 → wait 8s
```

A simple formula:

```python id="o7f1tv"
delay = base_delay * (2 ** attempt)
```

Production systems often add **jitter** to avoid many clients retrying at exactly the same time.

---

# 12. Decorators with Parameters

The retry example uses a decorator that accepts configuration:

```python id="n7s7ea"
@retry(attempts=3, delay=2)
def call_llm():
    ...
```

This requires an additional function layer.

Conceptually:

```text id="smn2ib"
retry(attempts=3)
       ↓
returns decorator
       ↓
decorator(function)
       ↓
wrapper
```

Structure:

```python id="2g7m7b"
def retry(attempts):

    def decorator(function):

        @wraps(function)
        def wrapper(*args, **kwargs):
            ...

        return wrapper

    return decorator
```

This pattern is important to understand.

---

# 13. Authentication Decorator

Decorators can enforce authorization before executing a function.

Conceptually:

```python id="n4q2aw"
def require_auth(function):

    @wraps(function)
    def wrapper(user, *args, **kwargs):

        if not user.is_authenticated:
            raise PermissionError("Authentication required")

        return function(user, *args, **kwargs)

    return wrapper
```

Then:

```python id="t6gk9f"
@require_auth
def get_private_documents(user):
    ...
```

The authentication concern stays separate from the business logic.

---

# 14. AI Tool Authorization

This concept is particularly useful for agent systems.

Suppose an agent has tools:

```text id="prx9i8"
Agent
 ├── search_documents
 ├── send_email
 ├── update_customer
 └── delete_record
```

Not every user should be allowed to call every tool.

A decorator can enforce permissions:

```python id="3w9f0a"
@require_permission("customer:update")
def update_customer(customer_id, data):
    ...
```

The actual authorization implementation should live in a proper security layer.

A decorator can provide a convenient enforcement point, but it should not replace a complete authorization architecture.

---

# 15. Caching Decorator

Python provides:

```python id="x1q2bm"
functools.lru_cache
```

Example:

```python id="8b2p0m"
from functools import lru_cache


@lru_cache(maxsize=100)
def expensive_calculation(value):
    return value * value
```

The result is cached.

This can be useful for deterministic expensive operations.

---

# 16. Be Careful Caching LLM Calls

Caching LLM calls is more complicated.

Consider:

```python id="y7z8j3"
@cache
def generate_answer(query):
    ...
```

Potential problems:

* Model output can depend on context.
* Prompts may change.
* User-specific information may be involved.
* Retrieved context may change.
* Model configuration may change.
* Responses may become stale.
* Sensitive data could accidentally be retained.

If caching LLM results, design the cache key and data lifecycle carefully.

For example:

```text id="6x1kqm"
cache key
=
model
+
prompt version
+
input
+
relevant context/version
```

The exact key depends on the application.

---

# 17. Decorator Stacking

Multiple decorators can be applied:

```python id="74fj4a"
@log_call
@measure_time
@retry(attempts=3)
def call_llm():
    ...
```

The order matters.

Conceptually:

```text id="jv4q7s"
call_llm
   ↓
retry
   ↓
measure_time
   ↓
log_call
```

Python applies decorators from the bottom upward.

Equivalent conceptually to:

```python id="6r8d4f"
call_llm = log_call(
    measure_time(
        retry(attempts=3)(call_llm)
    )
)
```

Understanding decorator order is important.

---

# 18. Decorator Order Can Change Behavior

Consider:

```python id="y1c7fb"
@measure_time
@retry(attempts=3)
def call_llm():
    ...
```

The timing decorator may measure the entire retry process.

But:

```python id="g0k6yp"
@retry(attempts=3)
@measure_time
def call_llm():
    ...
```

can measure individual attempts depending on implementation.

Therefore:

> Decorator order is part of program behavior.

---

# 19. Async Decorators

AI applications frequently use asynchronous functions.

For example:

```python id="8xw7hf"
async def call_llm():
    ...
```

A normal synchronous wrapper is not always appropriate.

Use an async wrapper:

```python id="p9u4hz"
from functools import wraps


def log_async(function):

    @wraps(function)
    async def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")

        result = await function(*args, **kwargs)

        print(f"Finished {function.__name__}")

        return result

    return wrapper
```

Usage:

```python id="h0s5yk"
@log_async
async def call_llm():
    ...
```

The wrapper itself must await the coroutine.

---

# 20. Sync and Async Decorators

A decorator designed for synchronous functions should not automatically be applied to asynchronous functions.

Synchronous:

```python id="qz5z5g"
def function():
    ...
```

Async:

```python id="k2q3fa"
async def function():
    ...
```

The wrapper must match the execution model.

For production libraries, decorators may explicitly support both forms.

---

# 21. FastAPI Decorators

FastAPI uses decorators heavily.

For example:

```python id="5v5r4n"
@app.get("/documents")
async def get_documents():
    ...
```

The decorator registers the function as a route handler.

Conceptually:

```text id="wmq2o7"
@app.get("/documents")
        ↓
Register function
        ↓
HTTP GET /documents
        ↓
Function executes
```

This demonstrates that decorators are not only for logging or retries.

They can also be used to **register behavior with a framework**.

---

# 22. Testing Decorators

Testing frameworks also use decorators.

For example:

```python id="y5o7xk"
@pytest.mark.parametrize(
    "query",
    ["RAG", "agents", "embeddings"]
)
def test_search(query):
    ...
```

The decorator provides metadata to the testing framework.

This pattern is common across Python ecosystems.

---

# 23. Class Decorators

Decorators can also modify classes.

Example:

```python id="9q8y4e"
def register_tool(cls):
    TOOL_REGISTRY.append(cls)
    return cls
```

Usage:

```python id="x3r7q1"
@register_tool
class SearchTool:
    ...
```

This can be useful for plugin architectures.

Conceptually:

```text id="i9xq4n"
Tool Class
   ↓
Decorator
   ↓
Tool Registry
```

AI frameworks can use similar patterns to register:

* Tools
* Plugins
* Components
* Handlers
* Routes

---

# 24. Decorators for Tool Registration

A simple example:

```python id="9d8x1e"
TOOLS = {}


def tool(name):

    def decorator(function):
        TOOLS[name] = function
        return function

    return decorator
```

Usage:

```python id="k6r4s8"
@tool("search_documents")
def search_documents(query):
    return search(query)
```

Now:

```python id="j0w7ra"
TOOLS["search_documents"]("What is RAG?")
```

can execute the registered function.

This illustrates how decorators can help build tool registries.

---

# 25. Decorators for Metrics

AI applications need observability.

A decorator can collect metrics:

```python id="3x1k0e"
import time
from functools import wraps


def track_metrics(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()

        try:
            result = function(*args, **kwargs)
            return result

        finally:
            duration = time.perf_counter() - start

            record_metric(
                function.__name__,
                duration
            )

    return wrapper
```

You could track:

```text id="pxv9tg"
tool calls
LLM calls
retrieval calls
errors
latency
```

In production, use a proper observability system rather than building an ad-hoc metrics system around `print()`.

---

# 26. Decorators for AI Observability

A production AI application may want to track:

```text id="p2k8xc"
LLM Call
├── latency
├── success/failure
├── model
├── token usage
├── request ID
└── trace ID
```

A decorator can provide a convenient instrumentation boundary:

```python id="9o5n5f"
@trace_llm_call
def generate_answer(prompt):
    ...
```

However, observability should be designed carefully around:

* Sensitive data
* Prompt logging
* User data
* PII
* Token costs
* Trace correlation

Do not automatically log complete prompts or responses in production.

---

# 27. Decorators for Validation

A decorator can validate function inputs.

Example:

```python id="3qj7p2"
def require_query(function):

    @wraps(function)
    def wrapper(query, *args, **kwargs):
        if not query.strip():
            raise ValueError("Query cannot be empty")

        return function(query, *args, **kwargs)

    return wrapper
```

Usage:

```python id="j5d1v8"
@require_query
def search_documents(query):
    ...
```

For complex validation, however, dedicated validation libraries such as Pydantic are usually more maintainable.

---

# 28. Decorators vs Middleware

Decorators and middleware can both implement cross-cutting behavior, but they operate at different levels.

### Decorator

Usually wraps a specific function.

```text id="n9xk5q"
Request
   ↓
Function decorator
   ↓
Function
```

### Middleware

Usually wraps an entire application or request/response pipeline.

```text id="n1m4sp"
Request
   ↓
Middleware
   ↓
Router
   ↓
Handler
   ↓
Response
```

Examples of middleware concerns:

* CORS
* Authentication
* Request logging
* Request IDs
* Global error handling

Use the abstraction appropriate to the scope of the concern.

---

# 29. Decorators vs Context Managers

Both can implement cross-cutting behavior.

A decorator is useful around a function:

```python id="7ph2y1"
@measure_time
def generate_answer():
    ...
```

A context manager is useful around a block:

```python id="w9p6x4"
with timer():
    generate_answer()
    retrieve_documents()
```

Use decorators when the behavior naturally belongs to a function or method.

Use context managers when the behavior belongs to a controlled block of execution or resource lifecycle.

---

# 30. Decorators and Dependency Injection

Decorators can participate in dependency systems, but they should not become a replacement for explicit dependency management.

For example:

```text id="2l4qk8"
Endpoint
   ↓
Authentication
   ↓
Service
   ↓
Repository
```

A decorator might enforce authentication.

Dependency injection can provide:

* Database clients
* Configuration
* Services
* Repositories
* LLM clients

Keep responsibilities clear.

---

# 31. Common Mistakes

## Mistake 1: Forgetting `functools.wraps`

Without:

```python id="q8j5n0"
@wraps(function)
```

function metadata can be lost.

Use it in most custom decorators.

---

## Mistake 2: Forgetting to return the result

Incorrect:

```python id="q6n4st"
def wrapper(*args, **kwargs):
    function(*args, **kwargs)
```

Correct:

```python id="6e5m2a"
def wrapper(*args, **kwargs):
    return function(*args, **kwargs)
```

---

## Mistake 3: Breaking async functions

A synchronous wrapper around an async function can cause incorrect behavior.

Use:

```python id="g6x8j9"
async def wrapper(...):
    return await function(...)
```

when appropriate.

---

## Mistake 4: Catching every exception

Avoid:

```python id="qv2j7k"
except Exception:
    retry()
```

without understanding which failures are retryable.

---

## Mistake 5: Hiding too much behavior

A function with five decorators can become difficult to understand.

```python id="x7y3sa"
@a
@b
@c
@d
@e
def function():
    ...
```

Keep decorators purposeful and understandable.

---

## Mistake 6: Logging sensitive AI data

Do not automatically log:

* User prompts
* Personal information
* API keys
* Authentication tokens
* Retrieved confidential documents
* Full LLM responses

Instrumentation must respect security and privacy requirements.

---

## Mistake 7: Putting business logic inside generic decorators

A decorator should usually handle a cross-cutting concern.

Avoid turning it into a hidden business-logic layer.

---

# Production Insight

Decorators are most valuable when they separate **business logic from cross-cutting concerns**.

Instead of:

```python id="e3j7km"
def generate_answer(query):

    log_request()

    validate_request()

    start_timer()

    try:
        result = call_llm(query)
        return result

    except TemporaryError:
        retry()

    finally:
        record_metrics()
```

you can conceptually separate the concerns:

```python id="q1v4y9"
@validate_request
@measure_time
@retry(attempts=3)
@track_metrics
def generate_answer(query):
    return call_llm(query)
```

The core function becomes easier to read.

However, decorators should not become a hidden maze of behavior.

A production engineer should be able to understand:

```text
What runs?
When does it run?
In what order?
What happens on failure?
What data is captured?
```

---

# How It's Used in AI Frameworks

Decorators appear throughout the Python AI ecosystem.

Common patterns include:

```text id="7s4r0c"
FastAPI
    → route registration

Testing frameworks
    → test metadata

Tool systems
    → tool registration

Caching
    → result reuse

Observability
    → tracing and metrics

Retry libraries
    → transient failure handling
```

Framework-specific decorator APIs change over time, so when using a particular library, check its current documentation.

The transferable skill is understanding how decorators transform or register functions.

---

# Practical Exercise

Build a small AI service with custom decorators.

Create:

### 1. Timing decorator

```python id="c0k2nv"
@measure_time
def retrieve_documents(query):
    ...
```

It should record execution time.

### 2. Logging decorator

```python id="r4x7p3"
@log_call
def retrieve_documents(query):
    ...
```

Log the function name and success/failure.

### 3. Retry decorator

```python id="b5m8qd"
@retry(attempts=3)
def call_llm(prompt):
    ...
```

Retry only selected transient failures.

### 4. Tool decorator

```python id="x3v9pk"
@tool("search_documents")
def search_documents(query):
    ...
```

Register the function in a tool registry.

Finally combine them:

```python id="m8k2zr"
@log_call
@measure_time
@retry(attempts=3)
@tool("search_documents")
def search_documents(query):
    ...
```

Then document the exact execution order.

---

# Interview Questions

## Beginner

1. What is a decorator?
2. What does the `@` syntax mean?
3. How do you create a decorator?
4. What are `*args` and `**kwargs` used for in decorators?
5. What does `functools.wraps` do?
6. Why should a wrapper return the original function's result?
7. Can classes be decorated?
8. Can decorators accept arguments?
9. What is a decorator factory?
10. Can multiple decorators be applied to one function?

## Intermediate

11. In what order are stacked decorators applied?
12. How do you write a parameterized decorator?
13. How do decorators work internally?
14. How do you preserve function metadata?
15. How do you write a decorator for an async function?
16. What problems can decorators introduce?
17. What is the difference between a decorator and middleware?
18. What is the difference between a decorator and a context manager?
19. How can decorators be used for caching?
20. How can decorators be used for retries?

## AI Engineering

21. How would you create a decorator for LLM request logging?
22. How would you measure LLM latency using a decorator?
23. How would you implement retry logic for transient LLM failures?
24. Which LLM errors should generally not be retried?
25. How would you register agent tools using decorators?
26. How would you instrument agent tool execution?
27. How would you prevent sensitive prompts from being logged?
28. How would you write a decorator that works with async LLM calls?
29. How would you combine authentication and authorization with AI tools?
30. When should decorator-based behavior instead be implemented as middleware or a service layer?

---

# Key Takeaways

* Decorators modify or extend functions without changing their core implementation.
* The `@decorator` syntax is shorthand for function transformation.
* `*args` and `**kwargs` make decorators reusable across different function signatures.
* `functools.wraps` preserves function metadata.
* Decorators are useful for cross-cutting concerns.
* Common AI engineering uses include logging, timing, retries, caching, authorization, metrics, tracing, and tool registration.
* Decorator order matters.
* Async functions require async-aware wrappers.
* Blind retries can make failures worse.
* Sensitive AI data should not automatically be logged.
* Decorators should remain small and focused.
* Middleware is generally better for application-wide request behavior.
* Context managers are better for scoped resource or execution-block behavior.
* Pydantic or dedicated validation systems are often better for complex validation.
* Decorators are a mechanism, not an architecture by themselves.

---

# Summary

Decorators allow you to separate **what a function does** from **what should happen around that function**.

For AI engineering, this is particularly useful for:

```text id="k3h8zq"
LLM calls
   ↓
logging
timing
retry
metrics
tracing
authorization
```

The core function can remain focused:

```python id="a4f7n2"
def generate_answer(query):
    return llm.generate(query)
```

while surrounding behavior is composed separately:

```python id="m9r2vc"
@measure_time
@retry(attempts=3)
@track_metrics
def generate_answer(query):
    return llm.generate(query)
```

Understanding decorators gives you a practical foundation for reading and building Python frameworks, APIs, agent tools, and production AI infrastructure.
