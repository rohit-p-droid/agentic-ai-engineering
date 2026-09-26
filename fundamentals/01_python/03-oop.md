# Object-Oriented Programming (OOP)

> Object-Oriented Programming (OOP) is a programming paradigm that organizes code into reusable, modular, and maintainable objects. Nearly every modern AI framework leverages OOP to model components such as language models, agents, tools, retrievers, vector stores, and memory systems.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand why OOP is essential for AI engineering.
- Create and use classes and objects.
- Apply encapsulation, inheritance, polymorphism, and abstraction.
- Build reusable AI components.
- Design maintainable Python applications.

---

# Why OOP Matters in AI Engineering

Modern AI applications are made up of many interacting components:

- LLMs
- Agents
- Tools
- Memory
- Vector Databases
- Retrievers
- Prompt Templates
- Evaluators

Representing each of these as objects makes applications easier to build, extend, and test.

Instead of writing one large script, OOP encourages splitting responsibilities into reusable classes.

---

# What is a Class?

A class is a blueprint for creating objects.

It defines:

- Attributes (data)
- Methods (behavior)

```python
class Chatbot:
    pass
```

A class itself doesn't perform work until an object is created.

---

# What is an Object?

An object is an instance of a class.

```python
class Chatbot:
    pass

bot = Chatbot()
```

Here, `bot` is an object created from the `Chatbot` class.

---

# Constructors (`__init__`)

The constructor initializes an object when it is created.

```python
class Chatbot:

    def __init__(self, model_name):
        self.model_name = model_name

bot = Chatbot("gpt-4.1")

print(bot.model_name)
```

Output

```text
gpt-4.1
```

---

# Instance Attributes

Attributes store information about an object.

```python
class LLM:

    def __init__(self, model):
        self.model = model
        self.temperature = 0.3
```

Each object maintains its own state.

---

# Instance Methods

Methods define behavior.

```python
class Chatbot:

    def greet(self):
        return "Hello!"
```

```python
bot = Chatbot()

print(bot.greet())
```

---

# The `self` Keyword

`self` refers to the current object.

```python
class User:

    def __init__(self, name):
        self.name = name
```

Without `self`, Python wouldn't know which object's data to access.

---

# Encapsulation

Encapsulation means keeping data and behavior together while hiding internal implementation details.

```python
class VectorStore:

    def __init__(self):
        self._documents = []

    def add(self, doc):
        self._documents.append(doc)
```

Users interact through methods instead of modifying internal state directly.

---

# Private Attributes

Python uses naming conventions.

```python
class APIKey:

    def __init__(self):
        self.__secret = "abc123"
```

Double underscores trigger name mangling, helping prevent accidental access.

---

# Inheritance

Inheritance allows one class to reuse another.

```python
class LLM:

    def generate(self, prompt):
        pass


class OpenAIModel(LLM):

    def generate(self, prompt):
        return "Response"
```

Benefits:

- Reusability
- Cleaner architecture
- Less duplication

---

# Method Overriding

Child classes can redefine parent methods.

```python
class Retriever:

    def search(self, query):
        pass


class HybridRetriever(Retriever):

    def search(self, query):
        return "Hybrid Search"
```

The interface remains the same while behavior changes.

---

# Polymorphism

Different classes can provide different implementations of the same method.

```python
models = [
    OpenAIModel(),
    AnthropicModel(),
    GeminiModel()
]

for model in models:
    print(model.generate("Hello"))
```

Each object responds differently to the same method call.

---

# Abstraction

Abstraction hides unnecessary implementation details.

```python
from abc import ABC, abstractmethod

class LLM(ABC):

    @abstractmethod
    def generate(self, prompt):
        pass
```

Every subclass must implement `generate()`.

This guarantees a consistent interface.

---

# Composition

Composition builds objects using other objects.

```python
class Agent:

    def __init__(self, llm, memory, retriever):
        self.llm = llm
        self.memory = memory
        self.retriever = retriever
```

Composition is generally preferred over deep inheritance hierarchies because it offers greater flexibility and easier testing.

---

# Class Variables

Shared across all instances.

```python
class Config:

    API_VERSION = "v1"
```

---

# Static Methods

Utility functions related to a class.

```python
class PromptFormatter:

    @staticmethod
    def format(question):
        return question.strip()
```

No object creation is required.

---

# Class Methods

Operate on the class itself.

```python
class Model:

    default_model = "gpt-4.1"

    @classmethod
    def create_default(cls):
        return cls(cls.default_model)

    def __init__(self, model):
        self.model = model
```

---

# Special Methods

Python supports many "magic methods."

```python
__init__()

__str__()

__repr__()

__len__()

__call__()

__iter__()

__enter__()

__exit__()
```

Many AI libraries implement these to create intuitive APIs.

---

# Real-World AI Architecture

```text
                 Agent
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
    Memory      Retriever      LLM
      │            │            │
      ▼            ▼            ▼
Conversation   Vector DB     OpenAI API
```

Each component is represented as an independent object with a clear responsibility.

---

# Best Practices

- Keep each class focused on a single responsibility.
- Prefer composition over inheritance.
- Hide implementation details behind methods.
- Use abstract base classes for shared interfaces.
- Avoid deeply nested inheritance hierarchies.
- Design small, reusable components.

---

# Common Mistakes

## One class doing everything

```python
class AIApplication:
    ...
```

Split responsibilities into dedicated classes.

---

## Excessive inheritance

```text
A

↓

B

↓

C

↓

D

↓

E
```

Deep inheritance trees become difficult to understand and maintain.

---

## Public mutable state

Avoid exposing internal attributes directly.

Provide methods for controlled access.

---

# Interview Questions

- What is OOP?
- What is the difference between a class and an object?
- What is encapsulation?
- What is inheritance?
- What is polymorphism?
- What is abstraction?
- Why is composition preferred over inheritance?
- What is method overriding?
- What is the difference between `@staticmethod` and `@classmethod`?
- What is `self`?
- How does Python implement private variables?
- How is OOP used in AI frameworks like LangChain?

---

# Summary

In this chapter, you learned:

- Classes and objects
- Constructors
- Attributes and methods
- Encapsulation
- Inheritance
- Polymorphism
- Abstraction
- Composition
- Static methods
- Class methods
- Special methods
- OOP best practices for AI engineering

These concepts provide the foundation for understanding the architecture of modern AI frameworks and building scalable, maintainable AI applications.