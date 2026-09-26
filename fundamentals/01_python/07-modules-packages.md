# Modules and Packages

> Modules and packages are the foundation of writing organized, reusable, and scalable Python applications. Every production AI project—from a simple RAG application to a multi-agent system—uses modules and packages to separate responsibilities and improve maintainability.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand the difference between modules and packages.
- Import Python modules correctly.
- Create your own modules and packages.
- Organize AI projects using a clean folder structure.
- Avoid common import-related issues.
- Follow production best practices.

---

# Why Modules and Packages Matter in AI Engineering

As AI applications grow, keeping everything in a single file quickly becomes unmanageable.

Instead of:

```text
main.py
(3000+ lines)
```

A production application is divided into logical components.

```text
app/
│
├── agents/
├── llms/
├── prompts/
├── memory/
├── tools/
├── retrievers/
├── vectorstores/
├── config/
├── api/
└── main.py
```

This makes the project easier to understand, test, and extend.

---

# What is a Module?

A module is simply a Python file (`.py`) containing code.

Example:

```text
math_utils.py
```

```python
def add(a, b):
    return a + b
```

Import it:

```python
import math_utils

print(math_utils.add(2, 3))
```

---

# What is a Package?

A package is a directory that groups related modules.

Example:

```text
utils/
│
├── math_utils.py
├── string_utils.py
└── file_utils.py
```

Usage:

```python
from utils.math_utils import add
```

Packages help organize larger applications.

---

# The `__init__.py` File

Traditionally, a package contains an `__init__.py` file.

```text
utils/
│
├── __init__.py
├── math_utils.py
└── file_utils.py
```

Its purposes include:

- Marking a directory as a package.
- Initializing package-level code.
- Controlling what is exposed to users.

Although modern Python supports namespace packages without it, including `__init__.py` is still recommended for clarity and compatibility.

---

# Importing Modules

## Import the Entire Module

```python
import math

print(math.sqrt(25))
```

---

## Import Specific Objects

```python
from math import sqrt

print(sqrt(25))
```

---

## Import with an Alias

```python
import numpy as np

array = np.array([1, 2, 3])
```

Aliases improve readability and reduce typing.

---

# Relative Imports

Within a package:

```python
from .loader import load_document
```

Or:

```python
from ..utils.logger import setup_logger
```

Relative imports are useful for internal package organization but should generally be avoided in entry-point scripts.

---

# Absolute Imports

Preferred in production projects.

```python
from app.utils.logger import setup_logger
```

Absolute imports are easier to understand and refactor.

---

# Creating Your Own Package

Example:

```text
llms/
│
├── __init__.py
├── openai_client.py
├── anthropic_client.py
└── gemini_client.py
```

Import:

```python
from llms.openai_client import OpenAIClient
```

---

# Organizing an AI Project

Example structure:

```text
chatbot/
│
├── app/
│   ├── api/
│   ├── agents/
│   ├── memory/
│   ├── prompts/
│   ├── retrievers/
│   ├── vectorstores/
│   ├── llms/
│   ├── tools/
│   ├── config/
│   └── utils/
│
├── tests/
│
├── pyproject.toml
│
└── README.md
```

Each package has a single responsibility.

---

# Avoid Circular Imports

Bad example:

```text
agent.py

imports memory.py

↓

memory.py

imports agent.py
```

This creates a circular dependency and causes import errors.

Instead, move shared functionality into a separate module.

---

# The `__name__` Variable

Every Python module has a built-in variable called `__name__`.

```python
print(__name__)
```

When a file is run directly:

```text
__main__
```

When imported:

```text
module_name
```

---

# Using `if __name__ == "__main__"`

```python
def main():
    print("Starting application")

if __name__ == "__main__":
    main()
```

This prevents code from executing when the module is imported elsewhere.

---

# Exposing Public APIs

Inside `__init__.py`:

```python
from .openai_client import OpenAIClient
from .gemini_client import GeminiClient

__all__ = [
    "OpenAIClient",
    "GeminiClient",
]
```

Users can now write:

```python
from llms import OpenAIClient
```

---

# Production Example

Imagine you're building a RAG application.

```text
rag_app/
│
├── loaders/
│   ├── pdf_loader.py
│   ├── markdown_loader.py
│   └── html_loader.py
│
├── chunking/
│
├── embeddings/
│
├── retrievers/
│
├── vectorstores/
│
├── prompts/
│
├── llms/
│
├── agents/
│
└── main.py
```

Every feature lives in its own package.

This keeps the project modular and scalable.

---

# Common Standard Library Modules

| Module | Purpose |
|---------|----------|
| os | Operating system interactions |
| pathlib | File paths |
| json | JSON processing |
| logging | Logging |
| asyncio | Asynchronous programming |
| csv | CSV handling |
| typing | Type hints |
| dataclasses | Data classes |
| collections | Specialized data structures |
| functools | Functional programming utilities |

---

# Best Practices

- Keep modules focused on one responsibility.
- Prefer absolute imports.
- Use meaningful package names.
- Avoid circular imports.
- Group related functionality.
- Keep entry-point files (`main.py`) small.
- Export only the public API.

---

# Common Mistakes

## Everything in One File

Bad:

```text
main.py
(4000+ lines)
```

Split functionality into packages.

---

## Circular Imports

Bad:

```text
agent.py

↓

memory.py

↓

agent.py
```

Refactor shared logic into another module.

---

## Wildcard Imports

Avoid:

```python
from utils import *
```

Use explicit imports instead.

---

## Poor Package Organization

Bad:

```text
utils/
```

containing unrelated files such as:

- AI models
- API routes
- Database logic
- File handling

Organize packages by responsibility rather than using a generic `utils` folder for everything.

---

# How It's Used in AI Frameworks

## LangChain

Organizes components into packages such as:

- chains
- prompts
- tools
- agents
- callbacks

---

## LangGraph

Uses separate modules for:

- Graphs
- Nodes
- State
- Checkpointing

---

## OpenAI SDK

Provides organized modules for:

- Chat completions
- Responses API
- Embeddings
- Files
- Fine-tuning

---

## FastAPI

Typically separates:

- API routes
- Services
- Models
- Schemas
- Dependencies

---

# Interview Questions

- What is a module?
- What is a package?
- What is the purpose of `__init__.py`?
- What is the difference between absolute and relative imports?
- What is a circular import?
- What is the purpose of `__name__`?
- Why do we use `if __name__ == "__main__"`?
- How would you organize a production AI project?
- Why are wildcard imports discouraged?

---

# Summary

In this chapter, you learned:

- What modules and packages are.
- How to import code effectively.
- How to create reusable packages.
- The role of `__init__.py`.
- Absolute vs relative imports.
- Avoiding circular imports.
- Organizing production AI applications.

Well-structured modules and packages make AI projects easier to develop, test, and maintain. As your applications grow—from simple chatbots to complex multi-agent systems—a clear project structure becomes just as important as writing good code.