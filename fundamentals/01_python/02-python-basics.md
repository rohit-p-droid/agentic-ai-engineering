# Variables and Data Types

Variables are one of the most fundamental concepts in Python. They allow you to store and manipulate data, making your programs dynamic and reusable.

Unlike languages such as Java or C++, Python is **dynamically typed**, meaning you don't need to declare the data type of a variable explicitly. Python determines the type automatically at runtime.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand how variables work in Python.
- Use Python's built-in data types.
- Choose the appropriate data structure for different scenarios.
- Write clean and readable variable names.
- Avoid common mistakes related to variable assignment.

---

# What is a Variable?

A variable is a name that refers to an object stored in memory.

Think of it as a label attached to a value.

```python
name = "Rohit"
age = 24
salary = 850000
```

Here:

- `name` references a string object.
- `age` references an integer object.
- `salary` references another integer object.

---

# How Python Stores Variables

Unlike some programming languages, variables do not directly contain values.

Instead, they reference objects.

```text
name
 │
 ▼
"Rohit"

age
 │
 ▼
24
```

When assigning a new value:

```python
age = 24
age = 25
```

The variable simply points to a different object.

---

# Dynamic Typing

Python allows variables to change types during execution.

```python
value = 10
value = "Hello"
value = True
```

Although valid, changing variable types unnecessarily can make code harder to understand. In production code, prefer keeping a variable's type consistent.

---

# Naming Variables

Choose descriptive names.

Good:

```python
user_name = "Alice"
total_price = 199.99
document_chunks = []
embedding_model = "text-embedding-3-large"
```

Avoid:

```python
a = 10
x = "abc"
temp = []
```

Except in very small scopes, descriptive names improve readability and maintainability.

---

# Python Naming Conventions

Follow the PEP 8 style guide.

| Convention | Example | Usage |
|------------|---------|-------|
| snake_case | user_name | Variables, functions |
| PascalCase | DocumentProcessor | Classes |
| UPPER_CASE | MAX_RETRIES | Constants |

---

# Built-in Data Types

## Integer

Whole numbers.

```python
age = 24
documents = 1500
```

---

## Float

Numbers with decimal points.

```python
accuracy = 0.94
price = 99.99
```

---

## String

Represents text.

```python
model_name = "gpt-4.1"
query = "Explain transformers."
```

Strings are immutable.

---

## Boolean

Represents logical values.

```python
is_active = True
is_cached = False
```

Frequently used in conditions.

---

## None

Represents the absence of a value.

```python
response = None
```

Commonly used when a value will be assigned later or to indicate "no result."

---

# Checking Data Types

Use the `type()` function.

```python
age = 24

print(type(age))
```

Output:

```python
<class 'int'>
```

---

# Type Conversion

Convert between types explicitly.

```python
age = "24"

age = int(age)

price = float("99.9")

count = str(100)
```

Always validate user input before conversion to avoid runtime errors.

---

# Multiple Assignment

Python supports assigning multiple variables in a single statement.

```python
x, y, z = 1, 2, 3
```

Swapping values is also simple.

```python
x, y = y, x
```

No temporary variable is required.

---

# Constants

Python does not enforce constants, but the convention is to use uppercase names.

```python
MAX_RETRIES = 3

API_TIMEOUT = 30
```

Treat these values as read-only.

---

# Production Example

```python
MODEL_NAME = "gpt-4.1"

EMBEDDING_MODEL = "text-embedding-3-large"

MAX_CONTEXT_LENGTH = 8192

temperature = 0.3

user_query = "Explain Retrieval-Augmented Generation."

documents = []

response = None
```

This demonstrates clear naming, consistent types, and common patterns used in AI applications.

---

# Best Practices

- Use meaningful variable names.
- Keep variable types consistent.
- Avoid single-letter variable names except in small loops.
- Prefer explicit type conversion.
- Follow PEP 8 naming conventions.
- Use uppercase for constants.
- Initialize variables close to where they are first used.

---

# Common Mistakes

### Reusing variables with different types

```python
data = []

data = "Hello"
```

This makes code harder to understand and may introduce bugs.

---

### Using unclear names

```python
a = 100
b = []
```

Prefer descriptive alternatives.

---

### Forgetting type conversion

```python
age = input()

result = age + 5
```

This raises a `TypeError` because `input()` returns a string.

Correct approach:

```python
age = int(input())

result = age + 5
```

---

# Interview Questions

### Why is Python called a dynamically typed language?

### What is the difference between mutable and immutable objects?

### What is the purpose of `None`?

### What happens internally when assigning a variable?

### What is the difference between `==` and `is`?

### Why should constants use uppercase names?

---

# Summary

In this chapter, you learned:

- What variables are.
- How Python stores references.
- Dynamic typing.
- Built-in data types.
- Naming conventions.
- Type conversion.
- Production best practices.

These concepts form the foundation for writing clean and maintainable Python code.