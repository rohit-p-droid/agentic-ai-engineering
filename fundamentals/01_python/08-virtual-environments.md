# Virtual Environments

> A virtual environment creates an isolated Python environment for a project, allowing the project to have its own Python packages and dependencies without affecting other projects on the same machine.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand why virtual environments are necessary.
- Create and activate a virtual environment.
- Install and manage dependencies.
- Understand environment isolation.
- Use `.gitignore` correctly.
- Work with virtual environments in AI projects.
- Understand the difference between virtual environments and containers.

---

# Why Virtual Environments Matter

Imagine you have two AI projects.

```text
Project A
LangChain 0.x
Pydantic 1.x
Python 3.11

Project B
LangChain 1.x
Pydantic 2.x
Python 3.12
```

If both projects use the same global Python environment, their dependencies can conflict.

```text
Global Python Environment

├── langchain 0.x
├── langchain 1.x       ❌
├── pydantic 1.x
├── pydantic 2.x        ❌
└── ...
```

A virtual environment isolates the dependencies.

```text
Project A
└── .venv
    ├── Python
    ├── LangChain
    └── Pydantic

Project B
└── .venv
    ├── Python
    ├── LangChain
    └── Pydantic
```

Each project can manage its dependencies independently.

---

# What is a Virtual Environment?

A virtual environment is an isolated Python environment containing:

- Python interpreter
- Installed packages
- Package metadata
- Scripts and executables

It is usually created inside or associated with a project.

A common convention is:

```text
my-ai-project/
│
├── .venv/
├── src/
├── tests/
└── pyproject.toml
```

---

# Creating a Virtual Environment

Python provides the built-in `venv` module.

```bash
python -m venv .venv
```

This creates:

```text
.venv/
```

---

# Activating the Environment

## Windows

Command Prompt:

```bash
.venv\Scripts\activate
```

PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

## macOS / Linux

```bash
source .venv/bin/activate
```

After activation, your terminal usually displays:

```text
(.venv)
```

---

# Deactivating

When finished:

```bash
deactivate
```

This returns you to the system Python environment.

---

# Verifying the Environment

Check which Python executable is being used.

### Windows

```bash
where python
```

### macOS / Linux

```bash
which python
```

You should see a path pointing to your project's virtual environment.

You can also run:

```bash
python --version
```

---

# Installing Packages

After activating the environment:

```bash
python -m pip install requests
```

For an AI project:

```bash
python -m pip install openai
```

Or:

```bash
python -m pip install langchain langgraph
```

Using:

```bash
python -m pip
```

helps ensure that `pip` belongs to the Python interpreter you're currently using.

---

# Checking Installed Packages

```bash
python -m pip list
```

For detailed information about a package:

```bash
python -m pip show openai
```

---

# Requirements Files

A traditional way to record dependencies is:

```text
requirements.txt
```

Example:

```text
openai
fastapi
qdrant-client
pydantic
```

Install everything with:

```bash
python -m pip install -r requirements.txt
```

---

# Pinning Versions

For reproducible environments, versions can be specified.

```text
openai==1.99.0
fastapi==0.116.0
pydantic==2.11.0
```

This ensures that the same versions are installed.

However, blindly pinning every dependency can make upgrades harder. Modern dependency managers can resolve and lock compatible dependency versions for you.

---

# Generating a Requirements File

You can export the current environment:

```bash
python -m pip freeze > requirements.txt
```

This records installed packages and their versions.

Example:

```text
fastapi==...
openai==...
pydantic==...
```

Be careful with this approach: `pip freeze` records everything installed in the environment, including packages you may not directly depend on.

---

# Virtual Environments and Git

Never commit your virtual environment to Git.

Bad:

```text
git add .venv/
```

A virtual environment can be recreated from dependency definitions.

Add this to `.gitignore`:

```gitignore
.venv/
venv/
env/
```

A typical AI project might have:

```text
my-ai-project/
│
├── .venv/              # Not committed
├── .env                # Not committed
├── .gitignore
├── requirements.txt
├── src/
└── README.md
```

---

# Virtual Environment vs `.env`

These are completely different concepts.

## `.venv`

Contains the isolated Python environment and installed packages.

```text
.venv/
```

## `.env`

Usually contains environment variables.

```text
OPENAI_API_KEY=...
DATABASE_URL=...
```

Never commit secrets from `.env` to Git.

---

# Virtual Environment vs Docker

A virtual environment isolates Python dependencies.

Docker isolates the application environment more broadly.

```text
Virtual Environment

Project
└── Python dependencies
```

Docker:

```text
Container
├── Application
├── Python
├── System libraries
└── Dependencies
```

They solve related but different problems.

A production AI application may use both:

```text
Docker Container
│
└── Python Virtual Environment
    │
    ├── FastAPI
    ├── LangGraph
    ├── OpenAI SDK
    └── Qdrant Client
```

In containerized applications, however, many teams install dependencies directly into the container's Python environment rather than creating an additional `venv`. The right choice depends on the deployment workflow.

---

# Using a Virtual Environment with VS Code

After creating `.venv`, configure your editor to use its Python interpreter.

In VS Code:

```text
Command Palette
    ↓
Python: Select Interpreter
    ↓
Select .venv
```

The integrated terminal should then use the project's environment.

---

# AI Project Example

Suppose you're building a RAG application.

```bash
mkdir rag-project
cd rag-project

python -m venv .venv
```

Activate it.

Then install dependencies:

```bash
python -m pip install \
    openai \
    qdrant-client \
    langchain \
    langgraph
```

Project structure:

```text
rag-project/
│
├── .venv/
├── src/
│   ├── ingestion/
│   ├── retrieval/
│   ├── generation/
│   └── main.py
│
├── tests/
├── .gitignore
├── requirements.txt
└── README.md
```

The virtual environment keeps the project's Python dependencies isolated.

---

# Recreating an Environment

If another developer clones the project:

```bash
git clone <repository>
cd rag-project
```

Create a new environment:

```bash
python -m venv .venv
```

Activate it and install dependencies:

```bash
python -m pip install -r requirements.txt
```

The environment is recreated without committing `.venv` itself.

---

# Common Problems

## `python` command not found

Your Python installation may not be available on the system PATH.

Verify:

```bash
python --version
```

or:

```bash
py --version
```

on Windows.

---

## Package installed but import fails

You may have installed the package into a different Python environment.

Check:

```bash
python -m pip show package-name
```

Then verify:

```bash
python -c "import package_name"
```

---

## Wrong Python interpreter

Your IDE may be using a different interpreter from your terminal.

Check:

```bash
where python
```

or:

```bash
which python
```

Then select the correct interpreter in your IDE.

---

# Best Practices

- Create one isolated environment per project.
- Keep `.venv` outside version control.
- Use `python -m pip` to avoid interpreter mismatches.
- Record project dependencies.
- Keep secrets outside dependency files.
- Use a consistent Python version for the project.
- Recreate environments rather than copying them between machines.
- Keep dependencies intentionally scoped to the project.

---

# Common Mistakes

## Installing everything globally

Avoid:

```bash
pip install langchain
pip install openai
pip install fastapi
```

without an isolated project environment.

---

## Committing `.venv`

Never commit:

```text
.venv/
```

The directory can be very large and is platform-specific.

---

## Confusing `.env` with `.venv`

Remember:

```text
.venv → Python environment

.env  → Environment variables
```

---

## Using the wrong Python

This is one of the most common problems.

You may think:

```bash
pip install openai
```

installed the package for your project while `pip` actually belongs to another Python installation.

Prefer:

```bash
python -m pip install openai
```

---

# How This Applies to AI Engineering

AI projects often have rapidly changing dependencies.

For example:

```text
AI Application
│
├── Python
├── FastAPI
├── OpenAI SDK
├── LangGraph
├── Pydantic
├── Qdrant Client
├── NumPy
└── Other dependencies
```

A dependency update can introduce compatibility problems.

Virtual environments provide the isolation needed to experiment safely without breaking unrelated projects.

---

# Interview Questions

- What is a virtual environment?
- Why should you use one?
- How do you create a virtual environment in Python?
- How do you activate and deactivate it?
- Why shouldn't `.venv` be committed to Git?
- What is the difference between `.env` and `.venv`?
- What is the difference between a virtual environment and Docker?
- Why might `pip install` and `python` refer to different environments?
- What is `requirements.txt`?
- What does `pip freeze` do?

---

# Summary

In this chapter, you learned:

- Why dependency isolation matters.
- How to create a virtual environment.
- How to activate and deactivate it.
- How to install and inspect packages.
- How to record dependencies.
- How to use `.gitignore`.
- The difference between `.venv`, `.env`, and Docker.
- How virtual environments fit into AI projects.

A clean development environment is the foundation of a reproducible AI project. As projects become more complex, dependency management becomes increasingly important, especially when working with rapidly evolving AI frameworks.