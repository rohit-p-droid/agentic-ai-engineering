# pip, Poetry and uv

Package management is one of the most important parts of Python development.

For AI engineers, this becomes even more important because GenAI projects often depend on many rapidly changing libraries:

* OpenAI SDKs
* LangChain
* LangGraph
* Pydantic
* FastAPI
* Qdrant clients
* Transformers
* PyTorch
* NumPy
* Evaluation and observability tools

A project can work perfectly on one machine and fail on another because of dependency versions.

This chapter explains three important Python package-management tools:

* `pip`
* `Poetry`
* `uv`

---

## Why does an AI Engineer need this?

AI projects usually have many dependencies.

For example:

```text
AI Application
│
├── FastAPI
├── Pydantic
├── OpenAI
├── LangChain
├── LangGraph
├── Qdrant Client
├── NumPy
└── PyTorch
```

These packages themselves have dependencies.

For example:

```text
LangGraph
   ↓
LangChain
   ↓
Pydantic
   ↓
other dependencies
```

Installing and maintaining these dependencies manually becomes difficult.

A package manager helps you:

* install dependencies
* remove dependencies
* upgrade dependencies
* resolve dependency conflicts
* reproduce environments
* maintain project configuration
* share dependencies with other developers

---

# 1. pip

`pip` is Python's standard package installer.

It is the most commonly encountered Python package-management tool.

## Installing a package

```bash
python -m pip install openai
```

Install multiple packages:

```bash
python -m pip install fastapi uvicorn pydantic
```

Using:

```bash
python -m pip
```

instead of simply:

```bash
pip
```

helps ensure that `pip` belongs to the Python interpreter you are currently using.

---

## Check installed packages

```bash
python -m pip list
```

Example:

```text
Package      Version
------------ -------
fastapi      ...
openai       ...
pydantic     ...
```

Check a specific package:

```bash
python -m pip show openai
```

---

# 2. Installing Specific Versions

AI libraries change quickly.

You may want to install a specific version:

```bash
python -m pip install pydantic==2.10.0
```

You can also specify a minimum version:

```bash
python -m pip install "pydantic>=2.0"
```

Or a compatible version range:

```bash
python -m pip install "pydantic>=2.0,<3.0"
```

Version constraints become important when building production AI systems.

---

# 3. requirements.txt

A common way of describing project dependencies is:

```text
requirements.txt
```

Example:

```text
fastapi==0.115.0
uvicorn==0.30.6
openai==1.50.0
pydantic==2.9.2
qdrant-client==1.12.0
```

Another developer can install them using:

```bash
python -m pip install -r requirements.txt
```

This makes the environment reproducible.

---

# 4. pip freeze

You can generate a list of installed packages:

```bash
python -m pip freeze
```

Example:

```text
fastapi==...
openai==...
pydantic==...
qdrant-client==...
```

Save it:

```bash
python -m pip freeze > requirements.txt
```

However, there is an important production consideration.

`pip freeze` records **everything installed in the environment**, including transitive dependencies.

For example:

```text
Your application
      │
      ├── langchain
      │      ├── dependency A
      │      └── dependency B
      │
      └── fastapi
             └── dependency C
```

A frozen environment can therefore contain many packages that your application never directly uses.

For small projects this may be acceptable.

For larger projects, more structured dependency management is often preferable.

---

# 5. Dependency Conflict

Suppose your project requires:

```text
Package A → Pydantic 1.x
Package B → Pydantic 2.x
```

You may encounter dependency conflicts.

This is particularly common in AI projects because frameworks evolve quickly.

A dependency resolver attempts to find versions that satisfy all requirements.

If no compatible combination exists, installation may fail.

This is one reason dependency management should be treated as part of engineering rather than an afterthought.

---

# 6. Poetry

Poetry is a Python dependency and project management tool.

It manages:

* dependencies
* project metadata
* dependency resolution
* virtual environments
* package configuration
* lock files

A Poetry project commonly uses:

```text
pyproject.toml
```

and:

```text
poetry.lock
```

---

## Creating a Poetry project

```bash
poetry new ai-assistant
```

A typical structure:

```text
ai-assistant/
├── pyproject.toml
├── README.md
├── tests/
└── src/
    └── ai_assistant/
        └── __init__.py
```

---

# 7. Adding Dependencies with Poetry

Add a dependency:

```bash
poetry add openai
```

Add several:

```bash
poetry add fastapi uvicorn qdrant-client
```

Add a development dependency:

```bash
poetry add --group dev pytest
```

Poetry updates the project configuration and lock file accordingly.

---

# 8. pyproject.toml

A simplified example:

```toml
[project]
name = "ai-assistant"
version = "0.1.0"
description = "An AI assistant application"

dependencies = [
    "fastapi",
    "openai",
    "qdrant-client",
]
```

Modern Python projects increasingly use `pyproject.toml` as the central project configuration file.

It can contain information about:

* project metadata
* dependencies
* build configuration
* development tools
* formatting
* linting
* testing

The exact configuration depends on the tooling used.

---

# 9. poetry.lock

The lock file records the resolved dependency versions.

Conceptually:

```text
pyproject.toml
      ↓
Dependency requirements
      ↓
Dependency resolver
      ↓
poetry.lock
      ↓
Exact resolved dependency graph
```

This improves reproducibility across machines and environments.

For application repositories, the lock file is generally something you want to commit to version control.

---

# 10. Installing from a Poetry Project

After cloning a project:

```bash
poetry install
```

Poetry uses the project configuration and lock file to recreate the environment.

This is useful for:

```text
Developer A
     ↓
git push
     ↓
Git repository
     ↓
Developer B
     ↓
poetry install
```

Both developers can work with the same resolved dependency set.

---

# 11. uv

`uv` is a modern Python package and project management tool developed by Astral.

It provides functionality for:

* package installation
* dependency resolution
* virtual environments
* project management
* lock files
* Python version management

It is designed with a strong focus on speed.

For AI engineering workflows, this can be particularly useful because AI projects often have large dependency trees.

---

# 12. Creating a uv Project

Create a project:

```bash
uv init ai-assistant
```

Move into it:

```bash
cd ai-assistant
```

Create the environment:

```bash
uv venv
```

Install a dependency:

```bash
uv add openai
```

Add more dependencies:

```bash
uv add fastapi uvicorn qdrant-client
```

---

# 13. Running Commands with uv

Instead of manually activating the virtual environment, you can use:

```bash
uv run python main.py
```

For example:

```bash
uv run uvicorn app.main:app --reload
```

This allows `uv` to execute commands using the project's managed environment.

---

# 14. uv and pyproject.toml

A modern `uv` project can use:

```text
pyproject.toml
uv.lock
```

Conceptually:

```text
pyproject.toml
      │
      │ dependency requirements
      ↓
    uv
      │
      │ resolution
      ↓
   uv.lock
```

The lock file records the resolved dependency information.

For reproducible application environments, committing the lock file is generally appropriate.

---

# 15. pip vs Poetry vs uv

| Feature                   | pip             | Poetry   | uv              |
| ------------------------- | --------------- | -------- | --------------- |
| Package installation      | Yes             | Yes      | Yes             |
| Dependency resolution     | Yes             | Yes      | Yes             |
| Lock file workflow        | Limited/basic   | Yes      | Yes             |
| Project management        | Limited         | Yes      | Yes             |
| Virtual environments      | External/`venv` | Yes      | Yes             |
| `pyproject.toml` workflow | Possible        | Yes      | Yes             |
| Speed                     | Standard        | Moderate | Very fast       |
| Simplicity                | Very high       | Medium   | High            |
| Common legacy usage       | Very high       | High     | Growing rapidly |

The important point is not to memorize which tool is "best".

Understand the underlying problem:

> **Your application needs a reproducible dependency environment.**

---

# 16. Which One Should an AI Engineer Learn?

Learn all three at a basic level.

You will encounter existing projects using different tools.

A practical learning order is:

```text
pip
 ↓
requirements.txt
 ↓
pyproject.toml
 ↓
Poetry
 ↓
uv
```

You should be comfortable reading a project and understanding:

```text
What dependencies does this project have?
What versions are required?
How is the environment created?
How are dependencies installed?
How are versions locked?
```

---

# 17. AI Project Example

Suppose we are building a RAG API.

Dependencies:

```text
FastAPI
OpenAI
LangChain
LangGraph
Qdrant
Pydantic
```

With pip:

```bash
python -m pip install fastapi openai langchain langgraph qdrant-client pydantic
```

With Poetry:

```bash
poetry add fastapi openai langchain langgraph qdrant-client pydantic
```

With uv:

```bash
uv add fastapi openai langchain langgraph qdrant-client pydantic
```

The commands differ, but the engineering goal is the same:

```text
Application
    ↓
Dependencies
    ↓
Resolved versions
    ↓
Reproducible environment
```

---

# 18. Development vs Production Dependencies

Not every package is required to run your application.

For example:

```text
Runtime
├── fastapi
├── openai
├── qdrant-client
└── pydantic

Development
├── pytest
├── ruff
└── mypy
```

Separating these dependencies keeps environments cleaner.

For example, Poetry supports dependency groups:

```bash
poetry add --group dev pytest ruff
```

The exact mechanism differs between package-management tools, but the concept is important.

---

# 19. Dependency Pinning

There are different levels of version constraints.

### Exact version

```text
openai==1.50.0
```

### Minimum version

```text
openai>=1.50.0
```

### Version range

```text
openai>=1.50.0,<2.0
```

Exact pinning provides stronger reproducibility.

Flexible constraints can make upgrades easier.

In production, use a lock/resolution mechanism and deliberately test dependency upgrades rather than blindly upgrading everything.

---

# 20. AI Frameworks Make This More Important

AI libraries evolve rapidly.

For example:

```text
Your application
      ↓
LangGraph
      ↓
LangChain
      ↓
Pydantic
      ↓
Other dependencies
```

A framework upgrade can introduce:

* API changes
* deprecated methods
* changed defaults
* different model interfaces
* changed dependency requirements

Therefore:

```text
"latest version"
```

does not automatically mean:

```text
"safe version for my application"
```

Always test upgrades.

---

# 21. Common Mistakes

### Mistake 1: Installing globally

```bash
pip install langchain
```

outside a project environment can pollute the global Python installation.

Prefer:

```bash
python -m venv .venv
```

and then install dependencies inside it.

---

### Mistake 2: Not committing dependency configuration

A repository should clearly communicate how its environment is reproduced.

Depending on the project, this could include:

```text
requirements.txt
```

or:

```text
pyproject.toml
poetry.lock
```

or:

```text
pyproject.toml
uv.lock
```

---

### Mistake 3: Blindly running upgrades

Avoid treating:

```bash
pip install --upgrade everything
```

as a deployment strategy.

AI frameworks can introduce breaking changes.

---

### Mistake 4: Using `pip freeze` without understanding it

A frozen environment can contain many transitive dependencies.

Know which dependencies your application actually uses.

---

### Mistake 5: Ignoring Python version compatibility

A dependency may support:

```text
Python 3.10+
```

while another dependency requires:

```text
Python 3.11+
```

Your Python version is part of the environment.

---

# 22. Production Insight

Dependency management is part of **software supply-chain management**.

In production AI systems, you should think about:

```text
Python version
      ↓
Direct dependencies
      ↓
Transitive dependencies
      ↓
Resolved versions
      ↓
Lock file
      ↓
CI tests
      ↓
Deployment
```

A production deployment should not depend on whatever happens to be the latest package version at deployment time.

A safer workflow is:

```text
Update dependency
        ↓
Resolve versions
        ↓
Run tests
        ↓
Run integration tests
        ↓
Validate AI behavior
        ↓
Deploy
```

This is especially important for LLM applications because a dependency upgrade can affect not only application code but also:

* prompts
* structured output
* tool calling
* agent execution
* RAG pipelines
* serialization
* API clients

---

# 23. How It's Used in AI Projects

A production AI repository might look like:

```text
ai-platform/
│
├── app/
│   ├── agents/
│   ├── rag/
│   ├── llms/
│   ├── tools/
│   ├── api/
│   └── config/
│
├── tests/
│
├── pyproject.toml
├── uv.lock
├── .python-version
├── .env.example
├── Dockerfile
└── README.md
```

The dependency-management files are not separate from the application architecture.

They are part of making the application reproducible.

---

# 24. Useful Commands Cheat Sheet

## pip

```bash
python -m pip install package
python -m pip uninstall package
python -m pip list
python -m pip show package
python -m pip freeze
python -m pip install -r requirements.txt
```

## Poetry

```bash
poetry new project
poetry install
poetry add package
poetry remove package
poetry update
poetry run python main.py
```

## uv

```bash
uv init
uv venv
uv add package
uv remove package
uv sync
uv run python main.py
```

---

# 25. Interview Questions

### Beginner

**1. What is pip?**

Python's package installer used to install and manage Python packages.

**2. What is `requirements.txt`?**

A file describing Python dependencies and their version constraints.

**3. What does `pip freeze` do?**

It outputs the packages and versions installed in the current environment.

**4. Why should you use a virtual environment?**

To isolate project dependencies from other Python projects and the global environment.

---

### Intermediate

**5. What is dependency resolution?**

The process of finding package versions that satisfy the dependency requirements of the project and its dependencies.

**6. What is a lock file?**

A file containing resolved dependency information used to make installations more reproducible.

**7. What is the difference between a direct and transitive dependency?**

A direct dependency is explicitly required by your project.

A transitive dependency is installed because one of your dependencies requires it.

Example:

```text
Your application
      ↓
LangChain        ← direct dependency
      ↓
Dependency X     ← transitive dependency
```

---

### AI Engineering

**8. Why is dependency management particularly important in GenAI applications?**

Because AI frameworks and SDKs evolve quickly and often have large dependency graphs. Version changes can affect agent execution, RAG pipelines, model clients, structured outputs, and tool integrations.

**9. Why might an AI application work locally but fail in production?**

Possible causes include:

* different Python versions
* different package versions
* missing dependencies
* incompatible dependency combinations
* environment-variable differences
* operating-system differences

**10. Why should dependency upgrades be tested?**

Because upgrades can introduce API changes, behavioral changes, or dependency conflicts that affect application functionality.

---

# Summary

The important concepts are:

```text
pip
 ↓
Package installation

requirements.txt
 ↓
Dependency declaration

Poetry / uv
 ↓
Project + dependency management

Lock file
 ↓
Reproducible dependency resolution

Virtual environment
 ↓
Environment isolation
```

For an AI Engineer, dependency management is not just about installing packages.

It is about making sure that:

> **The AI application you built today can be reliably reproduced, tested, and deployed tomorrow.**
