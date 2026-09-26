# Exception Handling

> Exception handling is the process of detecting, managing, and recovering from errors that occur during program execution. Robust exception handling is essential for building reliable AI applications because failures such as API timeouts, invalid inputs, network issues, and rate limits are inevitable in production systems.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand what exceptions are.
- Handle runtime errors gracefully.
- Use `try`, `except`, `else`, and `finally`.
- Raise custom exceptions.
- Create custom exception classes.
- Follow production-grade error handling practices.
- Build fault-tolerant AI applications.

---

# Why Exception Handling Matters in AI Engineering

AI applications depend on multiple external systems:

- LLM APIs
- Vector databases
- Cloud storage
- Databases
- Third-party APIs
- File systems

Every external dependency can fail.

Without proper exception handling, a single failure can crash your application.

Example:

```text
User
   │
   ▼
Backend
   │
   ▼
OpenAI API ❌
```

Instead of crashing, your application should detect the failure and respond appropriately.

---

# What is an Exception?

An exception is an error that interrupts the normal flow of a program.

Example:

```python
number = 10 / 0
```

Output:

```text
ZeroDivisionError: division by zero
```

Python stops execution because the exception is not handled.

---

# Handling Exceptions

Use `try` and `except`.

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero.")
```

Output:

```text
Cannot divide by zero.
```

The program continues running.

---

# Handling Multiple Exceptions

Different errors should be handled differently.

```python
try:
    number = int(input())
    result = 10 / number

except ValueError:
    print("Invalid number.")

except ZeroDivisionError:
    print("Cannot divide by zero.")
```

---

# Catching Multiple Exceptions

```python
try:
    ...
except (TypeError, ValueError):
    print("Invalid input.")
```

---

# Generic Exception Handling

```python
try:
    ...
except Exception as e:
    print(e)
```

Use this only when you cannot predict every possible exception.

Avoid hiding unexpected bugs.

---

# The `else` Block

Executed only if no exception occurs.

```python
try:
    number = int("10")

except ValueError:
    print("Invalid")

else:
    print("Conversion successful")
```

Output:

```text
Conversion successful
```

---

# The `finally` Block

Always executes.

Useful for releasing resources.

```python
file = open("data.txt")

try:
    data = file.read()

finally:
    file.close()
```

Even if an exception occurs, the file is closed.

---

# Raising Exceptions

Sometimes your code should generate an exception intentionally.

```python
age = -5

if age < 0:
    raise ValueError("Age cannot be negative.")
```

---

# Custom Exceptions

Production applications often define their own exceptions.

```python
class InvalidDocumentError(Exception):
    pass
```

Usage:

```python
raise InvalidDocumentError("Unsupported document format.")
```

---

# Exception Chaining

Preserve the original error while providing additional context.

```python
try:
    ...
except ValueError as e:
    raise RuntimeError("Failed to process input.") from e
```

This makes debugging much easier.

---

# Production Example

Calling an LLM API.

```python
try:
    response = client.responses.create(
        model="gpt-4.1",
        input="Explain RAG."
    )

except TimeoutError:
    print("Request timed out.")

except ConnectionError:
    print("Network error.")

except Exception as e:
    print(f"Unexpected error: {e}")
```

Never assume an external API will always succeed.

---

# Retrying Failed Operations

Some failures are temporary.

Example:

```python
import time

for attempt in range(3):

    try:
        result = api_call()
        break

    except TimeoutError:
        time.sleep(2)
```

Retrying improves reliability for transient failures.

---

# Logging Exceptions

Never silently ignore errors.

Bad:

```python
try:
    ...
except:
    pass
```

Good:

```python
import logging

logging.exception("Embedding generation failed.")
```

Logs include the full traceback for debugging.

---

# AI Engineering Example

Suppose you're ingesting PDF documents.

```python
try:
    text = extract_text(file)

except FileNotFoundError:
    print("Document not found.")

except InvalidDocumentError:
    print("Unsupported format.")

except Exception:
    print("Unexpected ingestion failure.")
```

The pipeline continues processing other documents instead of stopping entirely.

---

# Common Built-in Exceptions

| Exception | Description |
|-----------|-------------|
| ValueError | Invalid value |
| TypeError | Wrong data type |
| IndexError | Invalid list index |
| KeyError | Missing dictionary key |
| FileNotFoundError | File does not exist |
| ZeroDivisionError | Division by zero |
| ImportError | Import failure |
| TimeoutError | Operation timed out |
| ConnectionError | Network failure |
| PermissionError | Permission denied |

---

# Best Practices

- Catch specific exceptions whenever possible.
- Keep `try` blocks small.
- Log exceptions before handling them.
- Raise meaningful custom exceptions.
- Never suppress exceptions silently.
- Add context when re-raising exceptions.
- Clean up resources using `finally` or context managers.

---

# Common Mistakes

## Catching every exception

Bad:

```python
except Exception:
    pass
```

This hides bugs and makes debugging difficult.

---

## Large try blocks

Bad:

```python
try:
    ...
    ...
    ...
    ...
```

Keep the protected code as small as possible.

---

## Ignoring errors

Bad:

```python
except:
    pass
```

Always log or handle the error appropriately.

---

## Using exceptions for normal control flow

Avoid:

```python
try:
    value = dictionary[key]
except KeyError:
    ...
```

Prefer:

```python
value = dictionary.get(key)
```

when missing keys are expected.

---

# Production Checklist

Before deploying an AI application, ensure:

- API calls are wrapped in exception handling.
- Timeouts are handled.
- Retries are implemented for transient failures.
- Logs capture complete stack traces.
- Sensitive information (API keys, user data) is never logged.
- Custom exceptions clearly describe domain-specific failures.

---

# Interview Questions

- What is an exception?
- What is the difference between a syntax error and an exception?
- Explain `try`, `except`, `else`, and `finally`.
- Why should you catch specific exceptions?
- What is exception chaining?
- How do you create a custom exception?
- Why is `except: pass` considered bad practice?
- How would you handle API failures in an AI application?
- When should you retry a failed request?
- What should be logged when an exception occurs?

---

# Summary

In this chapter, you learned:

- What exceptions are.
- How to handle runtime errors.
- Using `try`, `except`, `else`, and `finally`.
- Raising and creating custom exceptions.
- Logging and retrying failed operations.
- Best practices for production AI systems.

Exception handling is not just about preventing crashes—it is about building resilient, observable, and maintainable AI applications that continue to function even when external services fail.