# File Handling

> File handling is the process of reading, writing, updating, and managing files stored on disk. In AI engineering, file handling is a fundamental skill because nearly every AI application interacts with documents, datasets, configuration files, logs, prompts, and model outputs.

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand how Python works with files.
- Read and write text files.
- Work with JSON, CSV, and binary files.
- Manage file paths correctly.
- Handle large files efficiently.
- Follow production best practices for AI applications.

---

# Why File Handling Matters in AI Engineering

Almost every AI application processes files.

Examples include:

- PDF documents
- CSV datasets
- JSON configurations
- Prompt templates
- Markdown documentation
- Images
- Audio files
- Log files

Example RAG ingestion pipeline:

```text
PDF Files
     │
     ▼
Read File
     │
     ▼
Extract Text
     │
     ▼
Chunk Document
     │
     ▼
Generate Embeddings
     │
     ▼
Store in Vector Database
```

Without file handling, none of these steps are possible.

---

# Opening a File

Use Python's built-in `open()` function.

```python
file = open("notes.txt", "r")
```

Parameters:

- File path
- Mode

Common modes:

| Mode | Description |
|------|-------------|
| `r` | Read |
| `w` | Write (overwrite) |
| `a` | Append |
| `x` | Create new file |
| `rb` | Read binary |
| `wb` | Write binary |

---

# Reading a File

```python
file = open("notes.txt", "r")

content = file.read()

file.close()

print(content)
```

`read()` loads the entire file into memory.

Suitable for small files.

---

# Reading Line by Line

Better for large files.

```python
with open("notes.txt") as file:

    for line in file:
        print(line.strip())
```

This is memory efficient.

---

# Reading a Fixed Number of Characters

```python
with open("notes.txt") as file:

    print(file.read(100))
```

Reads only the first 100 characters.

---

# Writing to a File

```python
with open("output.txt", "w") as file:

    file.write("Hello AI")
```

If the file exists, its previous contents are replaced.

---

# Appending to a File

```python
with open("logs.txt", "a") as file:

    file.write("Pipeline completed\n")
```

Existing data remains intact.

---

# Why Use `with`?

Always prefer the context manager.

```python
with open("notes.txt") as file:

    content = file.read()
```

When execution leaves the block:

- File is automatically closed.
- Resources are released.
- Exceptions are handled safely.

Avoid manually calling `close()` unless absolutely necessary.

---

# Working with File Paths

Avoid hardcoding paths.

Bad:

```python
"C:\\Users\\Documents\\data.txt"
```

Use `pathlib`.

```python
from pathlib import Path

path = Path("documents") / "data.txt"
```

Benefits:

- Cross-platform
- Cleaner syntax
- Easier maintenance

---

# Checking if a File Exists

```python
from pathlib import Path

path = Path("data.txt")

if path.exists():
    print("Found")
```

---

# Creating Directories

```python
from pathlib import Path

Path("logs").mkdir(exist_ok=True)
```

Useful for storing outputs.

---

# Reading JSON Files

JSON is widely used in AI applications.

Example:

```json
{
  "model": "gpt-4.1",
  "temperature": 0.3
}
```

Python:

```python
import json

with open("config.json") as file:

    config = json.load(file)

print(config["model"])
```

---

# Writing JSON

```python
import json

config = {
    "model": "gpt-4.1",
    "temperature": 0.2
}

with open("config.json", "w") as file:

    json.dump(config, file, indent=4)
```

---

# Reading CSV Files

Useful for datasets.

```python
import csv

with open("employees.csv") as file:

    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

---

# Working with Binary Files

Binary mode is used for:

- Images
- Audio
- PDFs
- Models

```python
with open("image.png", "rb") as file:

    image = file.read()
```

---

# File Encoding

Always specify UTF-8 for text files.

```python
with open(
    "notes.txt",
    encoding="utf-8"
) as file:

    print(file.read())
```

This avoids encoding-related issues.

---

# Production Example

Suppose you're building a document ingestion pipeline.

```python
from pathlib import Path

documents = Path("documents")

for file in documents.glob("*.md"):

    with file.open(
        encoding="utf-8"
    ) as f:

        text = f.read()

        print(text)
```

This approach is clean, portable, and scalable.

---

# Processing Large Files

Avoid:

```python
content = file.read()
```

Instead:

```python
for line in file:

    process(line)
```

Large language model datasets can be several gigabytes.

Reading them entirely into memory is inefficient.

---

# Common File Formats in AI

| Format | Purpose |
|---------|----------|
| TXT | Prompts |
| Markdown | Documentation |
| JSON | Configuration |
| CSV | Structured datasets |
| PDF | Knowledge base |
| HTML | Web pages |
| DOCX | Reports |
| Images | Vision models |
| Audio | Speech models |

---

# Best Practices

- Always use `with`.
- Prefer `pathlib` over string paths.
- Specify UTF-8 encoding.
- Validate file existence.
- Handle exceptions.
- Stream large files.
- Avoid hardcoded paths.

---

# Common Mistakes

## Forgetting to Close Files

Bad:

```python
file = open("data.txt")
```

Good:

```python
with open("data.txt") as file:
    ...
```

---

## Hardcoded Paths

Bad:

```python
"C:\\Users\\Downloads\\file.txt"
```

Use `pathlib`.

---

## Reading Huge Files into Memory

Avoid:

```python
file.read()
```

Prefer streaming.

---

## Assuming Files Always Exist

Always check or handle:

```python
FileNotFoundError
```

---

# How It's Used in AI Frameworks

## LangChain

- Reads PDFs
- Reads Markdown
- Loads text files
- Loads JSON documents

---

## LangGraph

- Stores checkpoints
- Reads configuration files
- Loads prompt templates

---

## OpenAI SDK

- Uploads files
- Reads prompts
- Stores generated outputs

---

## Vector Databases

- Import document collections
- Export embeddings
- Store metadata

---

# Interview Questions

- What is the difference between `r`, `w`, `a`, and `rb`?
- Why is `with open()` preferred?
- What is a context manager?
- Why should you use `pathlib`?
- How do you read JSON files?
- How do you process large files efficiently?
- When should binary mode be used?
- Why is UTF-8 important?
- How would you build a document ingestion pipeline?

---

# Summary

In this chapter, you learned:

- Reading and writing files.
- Context managers.
- Working with JSON and CSV.
- Binary file handling.
- Using `pathlib`.
- Streaming large files.
- Production best practices.

File handling is one of the core building blocks of AI engineering. Whether you're creating a RAG system, building an AI agent, or processing datasets, efficient and reliable file handling is a skill you'll use in nearly every project.