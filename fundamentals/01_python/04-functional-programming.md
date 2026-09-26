# Functional Programming in Python

> Functional Programming (FP) is a programming paradigm that treats computation as the evaluation of functions while minimizing mutable state and side effects. Although Python is not a purely functional language, many AI frameworks use functional programming concepts to build clean, reusable, and composable pipelines.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand the core principles of Functional Programming.
- Write pure and reusable functions.
- Avoid unnecessary side effects.
- Use higher-order functions effectively.
- Apply functional programming concepts in AI applications.
- Know when Functional Programming is beneficial and when OOP is a better choice.

---

# Why Functional Programming Matters in AI Engineering

Modern AI systems are essentially pipelines.

A typical workflow might look like this:

```text
User Query
    │
    ▼
Query Cleaning
    │
    ▼
Query Expansion
    │
    ▼
Embedding Generation
    │
    ▼
Document Retrieval
    │
    ▼
Reranking
    │
    ▼
LLM
    │
    ▼
Final Response
```

Each stage performs one task and passes the result to the next.

This is exactly the kind of problem Functional Programming solves well.

---

# What is Functional Programming?

Instead of storing behavior inside objects, Functional Programming focuses on writing small, independent functions that receive inputs and return outputs.

Example:

```python
def square(x):
    return x * x
```

The function performs one task and returns a result without changing anything outside itself.

---

# Pure Functions

A pure function has two properties:

- The same input always produces the same output.
- It does not modify external state.

Example:

```python
def add(a, b):
    return a + b
```

Good:

```python
print(add(5, 3))
```

Always returns:

```text
8
```

---

# Impure Functions

An impure function depends on or modifies external state.

```python
count = 0

def increment():
    global count
    count += 1
```

Its behavior depends on previous executions.

Impure functions are harder to test and debug.

---

# Why Pure Functions are Useful

Pure functions are:

- Predictable
- Easier to test
- Easier to reuse
- Easier to parallelize
- Less likely to introduce hidden bugs

---

# First-Class Functions

In Python, functions are objects.

You can:

- Store them in variables
- Pass them as arguments
- Return them from other functions

```python
def greet():
    return "Hello"

message = greet

print(message())
```

---

# Higher-Order Functions

A higher-order function accepts another function or returns one.

```python
def process(text, formatter):
    return formatter(text)
```

```python
def uppercase(text):
    return text.upper()

print(process("hello", uppercase))
```

Output:

```text
HELLO
```

This makes code flexible and reusable.

---

# Lambda Functions

Lambda functions are anonymous functions written in a single expression.

```python
square = lambda x: x * x

print(square(5))
```

Output:

```text
25
```

Use them for short, simple operations.

Avoid complex lambda expressions.

---

# map()

Applies a function to every element.

```python
numbers = [1, 2, 3, 4]

result = map(lambda x: x * 2, numbers)

print(list(result))
```

Output:

```python
[2, 4, 6, 8]
```

AI Example:

```python
documents = [
    "Hello",
    "Python",
    "OpenAI"
]

lowercase = list(map(str.lower, documents))
```

---

# filter()

Filters elements using a condition.

```python
numbers = [1, 2, 3, 4, 5]

result = filter(lambda x: x % 2 == 0, numbers)

print(list(result))
```

Output:

```python
[2, 4]
```

AI Example:

```python
documents = [
    "",
    "AI",
    "",
    "Python"
]

valid_docs = list(filter(bool, documents))
```

---

# reduce()

Combines multiple values into one.

```python
from functools import reduce

numbers = [1, 2, 3, 4]

total = reduce(lambda x, y: x + y, numbers)

print(total)
```

Output:

```text
10
```

---

# List Comprehensions

Python usually prefers comprehensions over `map()`.

```python
numbers = [1,2,3,4]

squares = [x*x for x in numbers]
```

Cleaner and easier to read.

---

# Immutability

Functional Programming prefers immutable data.

Instead of modifying:

```python
numbers.append(5)
```

Create a new object.

```python
new_numbers = numbers + [5]
```

This avoids unexpected side effects.

---

# Function Composition

Small functions can be combined.

```python
def clean(text):
    return text.strip()

def lowercase(text):
    return text.lower()

query = lowercase(clean(" Hello "))
```

Output:

```text
hello
```

This approach is common in AI preprocessing pipelines.

---

# Real AI Example

Document preprocessing pipeline.

```python
def clean(text):
    return text.strip()

def lowercase(text):
    return text.lower()

def remove_newlines(text):
    return text.replace("\n", " ")

document = "  Hello\nWorld  "

document = clean(document)
document = lowercase(document)
document = remove_newlines(document)
```

Each function performs exactly one responsibility.

---

# Functional vs OOP

| Functional Programming | Object-Oriented Programming |
|------------------------|-----------------------------|
| Focuses on functions | Focuses on objects |
| Stateless | Stateful |
| Easy to test | Great for complex systems |
| Composable | Highly extensible |
| Excellent for pipelines | Excellent for architecture |

Modern AI applications typically use both paradigms together.

---

# Where Functional Programming is Used

Functional Programming is commonly used for:

- Data preprocessing
- Prompt transformation
- Document cleaning
- Query rewriting
- Embedding pipelines
- Output parsing
- Evaluation pipelines
- Data validation

---

# Best Practices

- Prefer pure functions where possible.
- Keep functions small and focused.
- Avoid modifying global variables.
- Write reusable utility functions.
- Use comprehensions for readability.
- Avoid overly complex lambda functions.
- Compose small functions into larger workflows.

---

# Common Mistakes

## Functions doing multiple tasks

Bad:

```python
def process_document():
    ...
```

Better:

```python
clean()

chunk()

embed()

store()
```

Each function should have one responsibility.

---

## Modifying global state

Avoid:

```python
documents = []

def add(doc):
    documents.append(doc)
```

Prefer returning new values instead.

---

## Overusing lambda

Bad:

```python
lambda x: complicated_expression...
```

If the logic spans more than one line, use a normal function.

---

# Production Example

A document ingestion pipeline.

```python
def clean(text):
    return text.strip()

def chunk(text):
    return text.split(".")

def remove_empty(chunks):
    return [c for c in chunks if c]

document = "AI is amazing. Python is powerful."

cleaned = clean(document)

chunks = chunk(cleaned)

final_chunks = remove_empty(chunks)
```

Each function is reusable and independently testable.

---

# Interview Questions

- What is Functional Programming?
- What is a pure function?
- What are side effects?
- What are higher-order functions?
- What are first-class functions?
- What is function composition?
- What is immutability?
- What is the difference between Functional Programming and OOP?
- When should you use Functional Programming in AI applications?
- Why are pure functions easier to test?

---

# Summary

In this chapter, you learned:

- Functional Programming principles
- Pure and impure functions
- First-class functions
- Higher-order functions
- Lambda expressions
- `map()`, `filter()`, and `reduce()`
- Function composition
- Immutability
- Real-world AI use cases

Functional Programming complements Object-Oriented Programming. While OOP is ideal for modeling AI systems and their components, Functional Programming excels at building clean, reusable data-processing pipelines. Mastering both paradigms will help you write more maintainable and scalable AI applications.