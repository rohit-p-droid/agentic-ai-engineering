# 02 — Machine Learning

This section builds the **machine learning foundation required for an Agentic AI Engineer**.

The goal is not to become an ML researcher. The goal is to understand how machine learning systems work, how to train and evaluate models, and how ML concepts connect to modern AI systems such as embeddings, RAG, LLMs, and agents.

---

## What You'll Learn

```text
Machine Learning Fundamentals
        ↓
Data & Features
        ↓
Supervised / Unsupervised Learning
        ↓
Model Training
        ↓
Evaluation & Generalization
        ↓
Classical ML Algorithms
        ↓
Neural Networks
        ↓
PyTorch
        ↓
Training & Optimization
        ↓
Model Serving
        ↓
ML Pipelines
        ↓
Embeddings & Similarity Search
        ↓
ML for LLM Systems
        ↓
ML System Design
```

---

## Topics

### Fundamentals

* What is Machine Learning?
* AI vs ML vs Deep Learning
* Types of Machine Learning
* Supervised Learning
* Unsupervised Learning
* Reinforcement Learning
* Datasets
* Features and Labels
* Training and Inference
* Parameters and Hyperparameters
* Loss Functions
* Optimization
* Generalization
* Overfitting and Underfitting
* Bias and Variance
* Data Leakage
* Distribution Shift

### Data & Features

* Data preprocessing
* Missing values
* Encoding categorical data
* Feature scaling
* Feature engineering
* Feature selection

### Classical Machine Learning

* Classification
* Regression
* Clustering
* Decision Trees
* Ensemble Learning
* Gradient Boosting
* K-Nearest Neighbors
* Naive Bayes
* Support Vector Machines
* Dimensionality Reduction

### Model Development

* Model evaluation
* Cross-validation
* Hyperparameter tuning
* Training workflows
* Model selection
* Model serving
* ML pipelines

### Deep Learning

* Neural network fundamentals
* Tensors
* Forward propagation
* Backpropagation
* Activation functions
* Optimizers
* PyTorch basics
* Training neural networks

### Agentic AI-Relevant ML

* Embeddings
* Vector representations
* Similarity search
* Representation learning
* ML evaluation
* ML models inside LLM systems
* Model inference
* ML system design

---

## Projects

The section includes practical projects that progressively move from classical ML toward production-oriented AI systems.

```text
projects/
│
├── 01-classification-project.md
├── 02-regression-project.md
├── 03-clustering-project.md
├── 04-embedding-search-project.md
└── 05-production-ml-service.md
```

The final projects focus on concepts that are directly useful when building AI and agentic systems.

---

## Tools & Libraries

Primary tools covered in this section:

```text
Python
NumPy
Pandas
Matplotlib
Scikit-learn
PyTorch
```

Later sections of the roadmap will build on these foundations with tools such as:

```text
LangChain
LangGraph
Qdrant
LLM APIs
MCP
FastAPI
Docker
```

---

## Prerequisites

Before starting this section, you should be comfortable with:

* Python
* Functions
* Classes
* Data structures
* File handling
* Modules and packages
* Virtual environments
* Type hints
* Async programming
* Testing
* Basic mathematical reasoning

These concepts are covered in the previous **01 — Python** section.

---

## Learning Approach

For every topic, focus on four levels:

### 1. Concept

Understand what the technique does and why it exists.

### 2. Mathematics

Understand the important mathematical intuition without unnecessarily diving into research-level mathematics.

### 3. Implementation

Implement the concept using Python and relevant libraries.

### 4. Engineering

Understand how the technique behaves in a real production system.

The goal is:

```text
Understand
    ↓
Implement
    ↓
Evaluate
    ↓
Deploy
    ↓
Integrate
```

---

## Agentic AI Connection

Machine learning is one of the foundations underneath modern AI systems.

For example, a typical RAG system involves:

```text
Documents
    ↓
Embedding Model
    ↓
Vector Representations
    ↓
Vector Database
    ↓
Similarity Search
    ↓
Retrieved Context
    ↓
LLM
    ↓
Agent
    ↓
Final Response
```

Understanding ML helps you reason about what happens inside these components instead of treating them as black boxes.

---

## Recommended Order

Follow the files in numerical order:

```text
01 → Fundamentals
02 → Data Preprocessing
03 → Feature Engineering
04 → Supervised Learning
05 → Unsupervised Learning
06 → Model Evaluation
07 → Cross Validation
08 → Overfitting / Underfitting
09 → Regularization
10 → Feature Selection
11 → Classification
12 → Regression
13 → Clustering
14 → Dimensionality Reduction
15 → Decision Trees
16 → Ensemble Learning
17 → Gradient Boosting
18 → KNN
19 → Naive Bayes
20 → SVM
21 → Neural Networks
22 → PyTorch
23 → Training Workflow
24 → Hyperparameter Tuning
25 → Model Serving
26 → ML Pipelines
27 → Embeddings
28 → Similarity Search
29 → ML for LLM Systems
30 → ML System Design
```

Then complete the projects, interview questions, and resources.

---

## End Goal

By the end of this section, you should be able to:

* Understand common ML algorithms
* Prepare datasets for ML
* Train and evaluate models
* Identify overfitting and data leakage
* Understand model generalization
* Build basic ML systems with Python
* Use PyTorch for basic neural-network workflows
* Understand embeddings and similarity search
* Understand how ML components fit into LLM applications
* Design basic production ML workflows
* Explain ML concepts clearly in technical interviews

Most importantly, you should be able to treat ML as an **engineering component of larger AI systems**, rather than learning algorithms in isolation.
