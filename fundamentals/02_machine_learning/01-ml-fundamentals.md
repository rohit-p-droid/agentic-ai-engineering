# Machine Learning Fundamentals

Machine Learning (ML) is a branch of artificial intelligence where systems learn patterns from data and use those patterns to make predictions or decisions without being explicitly programmed for every case.

For an Agentic AI Engineer, you don't need to become an ML researcher. You need to understand how ML systems work well enough to build, integrate, evaluate, debug, and deploy AI-powered applications.

---

## 1. What Is Machine Learning?

Traditional programming:

```text
Rules + Data → Program → Output
```

Machine learning:

```text
Data + Expected Output → Learning Algorithm → Model
```

Once trained:

```text
New Data → Model → Prediction
```

### Example

Traditional approach:

```python
if age > 18:
    category = "adult"
else:
    category = "minor"
```

ML approach:

```text
Historical user data
        ↓
     Training
        ↓
      Model
        ↓
New user's data
        ↓
    Prediction
```

The model learns relationships from examples instead of relying entirely on manually written rules.

---

# 2. Why Machine Learning?

ML is useful when explicitly writing rules becomes difficult.

For example, consider spam detection.

It is difficult to write rules for every possible spam message:

```text
"Congratulations! You won..."
"Claim your reward..."
"FREE MONEY..."
...
```

Instead, we provide many examples:

```text
Message                         Label
---------------------------------------
"Meeting at 10 AM"              Not Spam
"Win $1,000 now!"               Spam
"Your order has shipped"        Not Spam
"Claim your free prize"         Spam
```

The model learns patterns associated with spam.

---

# 3. AI vs ML vs Deep Learning

These terms are related but not identical.

```text
Artificial Intelligence
│
├── Machine Learning
│   │
│   ├── Classical ML
│   │   └── Decision Trees, SVM, KNN, etc.
│   │
│   └── Deep Learning
│       └── Neural Networks
│
└── Other AI approaches
```

### Artificial Intelligence

The broad field of building systems capable of performing tasks associated with intelligence.

### Machine Learning

A subset of AI where systems learn patterns from data.

### Deep Learning

A subset of ML based primarily on neural networks with multiple layers.

### Generative AI

Systems that generate new content such as:

* Text
* Images
* Audio
* Video
* Code

Modern LLMs are generally based on deep learning and transformer architectures.

---

# 4. Types of Machine Learning

The three major categories are:

```text
Machine Learning
│
├── Supervised Learning
├── Unsupervised Learning
└── Reinforcement Learning
```

There are also approaches such as:

* Semi-supervised learning
* Self-supervised learning

---

## 4.1 Supervised Learning

The model learns from labeled data.

```text
Input → Known Output
```

Example:

```text
House Features → House Price

1200 sq ft → ₹70 lakh
1800 sq ft → ₹95 lakh
2500 sq ft → ₹1.4 crore
```

The model learns the relationship between inputs and outputs.

Common supervised learning tasks:

### Classification

Predict a category.

```text
Email → Spam / Not Spam
Image → Cat / Dog
Transaction → Fraud / Normal
```

### Regression

Predict a continuous numerical value.

```text
House features → Price
Experience → Salary
Temperature → Electricity consumption
```

---

# 5. Unsupervised Learning

The data does not contain explicit labels.

```text
Input Data → Model → Discovered Patterns
```

Example:

```text
Customer Data
     ↓
Clustering
     ↓
Customer Groups
```

The model may discover groups such as:

```text
Group 1 → High-value customers
Group 2 → Occasional customers
Group 3 → New customers
```

Common unsupervised techniques include:

* Clustering
* Dimensionality reduction
* Anomaly detection
* Representation learning

---

# 6. Reinforcement Learning

In reinforcement learning, an agent interacts with an environment.

```text
       Action
Agent ─────────→ Environment
  ↑                  │
  │                  ↓
  └──── Reward ──────┘
```

The agent learns which actions produce better rewards.

Example:

```text
State → Action → Reward → New State
```

Reinforcement learning is important conceptually for understanding systems such as RLHF and RL-based optimization, although most Agentic AI application development does not require implementing RL algorithms from scratch.

---

# 7. Dataset

A dataset is a collection of examples used to develop or evaluate a model.

Example:

| Age | Income | Purchased |
| --: | -----: | --------- |
|  22 |  30000 | No        |
|  35 |  70000 | Yes       |
|  42 |  90000 | Yes       |
|  25 |  40000 | No        |

Each row is an example.

Each column represents a variable.

---

# 8. Features and Labels

### Feature

An input variable used by the model.

```text
Age
Income
Experience
Location
```

### Label / Target

The value the model is trying to predict.

```text
Purchased
Salary
House Price
Spam Status
```

Example:

```text
Features:

Age
Income
Location

        ↓

      Model

        ↓

Label:

Purchased
```

Usually we represent these as:

```text
X = Features
y = Target
```

Example:

```python
X = df[["age", "income"]]
y = df["purchased"]
```

---

# 9. Training Data and Test Data

We normally don't train and evaluate a model on exactly the same data.

A common split is:

```text
Dataset
   │
   ├── Training Data
   │
   └── Test Data
```

For example:

```text
80% → Training
20% → Testing
```

The training data is used to learn patterns.

The test data is used to measure how well the model performs on unseen examples.

---

# 10. Training

Training is the process of learning model parameters from data.

Conceptually:

```text
Training Data
     ↓
Learning Algorithm
     ↓
Model Parameters
     ↓
Trained Model
```

For example, a linear regression model might learn:

```text
y = w₁x₁ + w₂x₂ + b
```

The training process attempts to find suitable values for:

```text
w₁
w₂
b
```

---

# 11. Inference

Inference means using a trained model to make predictions on new data.

```text
New Input
   ↓
Trained Model
   ↓
Prediction
```

Example:

```python
prediction = model.predict(new_data)
```

Training and inference are different workloads.

```text
Training:
Data → Optimization → Model

Inference:
Input → Model → Output
```

This distinction becomes very important when designing production AI systems.

---

# 12. Parameters

Parameters are values learned by the model during training.

For a simple linear model:

```text
y = wx + b
```

`w` and `b` are parameters.

The training algorithm learns their values from the data.

Different ML algorithms have different types and numbers of parameters.

---

# 13. Hyperparameters

Hyperparameters are configuration values chosen before or around the training process.

Examples:

```text
Learning rate
Number of trees
Maximum tree depth
Number of clusters
Batch size
Number of epochs
```

Example:

```python
RandomForestClassifier(
    n_estimators=100,
    max_depth=10
)
```

Here:

```text
n_estimators
max_depth
```

are hyperparameters.

A useful distinction:

```text
Parameters
    ↓
Learned from data

Hyperparameters
    ↓
Configured by the developer / training process
```

---

# 14. Model

A model is the learned representation of patterns in the training data.

Conceptually:

```text
Training Data
      ↓
Learning Algorithm
      ↓
     Model
```

A model can then be used for inference:

```text
New Input
    ↓
  Model
    ↓
Prediction
```

Examples:

```text
Linear Regression
Decision Tree
Random Forest
Neural Network
Transformer
```

---

# 15. Loss Function

A loss function measures how incorrect a model's prediction is.

```text
Prediction
    +
Actual Value
    ↓
Loss Function
    ↓
Error
```

Example:

```text
Actual Price      = ₹50 lakh
Predicted Price   = ₹45 lakh

Error             = ₹5 lakh
```

Different problems use different loss functions.

Examples:

### Mean Squared Error

Commonly used for regression.

```text
MSE = average((actual - predicted)²)
```

### Cross-Entropy Loss

Commonly used for classification.

The training process attempts to minimize the loss.

---

# 16. Optimization

Training generally involves finding parameters that minimize the loss.

Conceptually:

```text
Parameters
    ↓
Model Prediction
    ↓
Loss
    ↓
Optimization
    ↓
Updated Parameters
    ↓
Repeat
```

A common optimization algorithm is:

```text
Gradient Descent
```

The basic idea:

```text
Move parameters in a direction that reduces the loss.
```

---

# 17. Generalization

A good ML model should perform well on data it has not seen before.

This ability is called **generalization**.

```text
Training Data
     ↓
Learn Patterns
     ↓
Model
     ↓
Unseen Data
     ↓
Good Predictions
```

The goal isn't to memorize the training data.

The goal is to learn useful patterns that transfer to new data.

---

# 18. Overfitting

Overfitting happens when a model learns the training data too closely, including noise or irrelevant patterns.

```text
Training Performance → Very Good
Test Performance     → Poor
```

Example:

```text
Model memorizes:
"These exact examples were labeled spam."

Instead of learning:
"These characteristics are associated with spam."
```

Common ways to reduce overfitting:

* More training data
* Regularization
* Simpler models
* Feature selection
* Cross-validation
* Early stopping
* Data augmentation

---

# 19. Underfitting

Underfitting occurs when a model is too simple to capture important patterns.

```text
Training Performance → Poor
Test Performance     → Poor
```

Conceptually:

```text
Too Simple
    ↓
Underfitting

Good Complexity
    ↓
Generalization

Too Complex
    ↓
Overfitting
```

---

# 20. Bias and Variance

Two important sources of model error are bias and variance.

### High Bias

The model is too simplistic.

```text
High Bias → Underfitting
```

### High Variance

The model is too sensitive to training data.

```text
High Variance → Overfitting
```

The goal is to find an appropriate balance.

```text
Bias ←──── Balance ────→ Variance
```

---

# 21. Data Leakage

Data leakage occurs when information that should not be available during training accidentally enters the training process.

Example:

Suppose we want to predict whether a customer will default on a loan.

If the training data contains:

```text
Loan Default Date
```

while trying to predict default before it happens, the model has access to future information.

This can produce:

```text
Excellent validation score
        ↓
Poor real-world performance
```

Data leakage is one of the most important problems to understand when building production ML systems.

---

# 22. Distribution Shift

A model assumes that the data it sees during deployment is reasonably related to the data it learned from.

But real-world data can change.

```text
Training Distribution
        ↓
      Model
        ↓
Production Distribution
        ↓
Performance Changes
```

Examples:

* User behavior changes
* New products appear
* Language changes
* Sensor characteristics change
* Fraud patterns evolve

This is particularly important for production AI systems.

---

# 23. ML Development Lifecycle

A typical ML workflow looks like:

```text
Problem Definition
       ↓
Data Collection
       ↓
Data Cleaning
       ↓
Feature Engineering
       ↓
Train / Validation / Test Split
       ↓
Model Training
       ↓
Evaluation
       ↓
Hyperparameter Tuning
       ↓
Final Model
       ↓
Deployment
       ↓
Monitoring
       ↓
Retraining
```

This is an iterative process rather than a strictly linear one.

---

# 24. ML vs Rule-Based Systems

Rule-based:

```text
Input
 ↓
Rules
 ↓
Output
```

ML:

```text
Historical Data
 ↓
Training
 ↓
Model
 ↓
New Input
 ↓
Prediction
```

Rule-based systems are often preferable when:

* Rules are deterministic
* Rules are easy to define
* Explainability is critical
* There is insufficient training data

ML can be useful when:

* Patterns are difficult to manually encode
* Large amounts of data are available
* Predictions need to adapt to learned patterns

In production systems, both approaches are often combined.

---

# 25. Classical ML vs LLMs

Classical ML models generally solve focused prediction or pattern-recognition problems.

Examples:

```text
Fraud Detection
Classification
Forecasting
Recommendation
Clustering
Anomaly Detection
```

LLMs are general-purpose foundation models capable of tasks such as:

```text
Text Generation
Reasoning
Summarization
Extraction
Code Generation
Tool Calling
```

An Agentic AI system can combine both.

Example:

```text
User Request
     ↓
LLM Agent
     ↓
┌───────────────┐
│ Tools         │
│ Retrieval     │
│ ML Models     │
│ APIs          │
│ Databases     │
└───────────────┘
     ↓
Final Response
```

An agent doesn't necessarily need to use an ML model for every operation.

---

# 26. Why ML Fundamentals Matter for Agentic AI

As an Agentic AI Engineer, you will encounter concepts such as:

```text
Embeddings
Vector Similarity
Retrieval
Reranking
Classification
Evaluation
Anomaly Detection
Model Serving
Fine-tuning
Inference
```

These concepts become easier to understand once the ML fundamentals are clear.

For example:

```text
Document
   ↓
Embedding Model
   ↓
Vector
   ↓
Similarity Search
   ↓
Retrieved Documents
   ↓
LLM
   ↓
Answer
```

The embedding model is itself a machine learning model.

Understanding ML helps you reason about:

* What the model learned
* What its inputs represent
* What its outputs mean
* How to evaluate it
* Where it can fail
* How distribution changes affect it
* How to deploy it efficiently

---

# 27. Important Terminology

| Term                | Meaning                                                         |
| ------------------- | --------------------------------------------------------------- |
| Dataset             | Collection of examples                                          |
| Feature             | Input variable                                                  |
| Label               | Expected output                                                 |
| Model               | Learned representation/pattern                                  |
| Parameter           | Value learned during training                                   |
| Hyperparameter      | Configuration controlling training/model behavior               |
| Training            | Learning from data                                              |
| Inference           | Generating predictions from a trained model                     |
| Loss                | Measure of prediction error                                     |
| Optimization        | Process of minimizing loss                                      |
| Generalization      | Performance on unseen data                                      |
| Overfitting         | Learning training data too closely                              |
| Underfitting        | Model is too simple                                             |
| Epoch               | One complete pass through training data                         |
| Batch               | Subset of training examples processed together                  |
| Learning Rate       | Step size used by an optimizer                                  |
| Feature Engineering | Creating useful input representations                           |
| Data Leakage        | Unintended access to information unavailable at prediction time |
| Distribution Shift  | Change between training and production data                     |

---

# 28. Minimal Python Example

Using scikit-learn:

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

X, y = load_iris(return_X_y=True)

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

model = LogisticRegression(max_iter=200)

model.fit(X_train, y_train)

predictions = model.predict(X_test)

accuracy = accuracy_score(y_test, predictions)

print(f"Accuracy: {accuracy:.2f}")
```

The workflow is:

```text
Load Data
   ↓
Split Data
   ↓
Create Model
   ↓
Train
   ↓
Predict
   ↓
Evaluate
```

You will explore each stage in much more detail in the following sections.

---

# 29. What You Should Understand Before Moving On

You should be able to explain:

* What machine learning is
* AI vs ML vs Deep Learning
* Supervised vs unsupervised learning
* Classification vs regression
* Features vs labels
* Training vs inference
* Parameters vs hyperparameters
* Loss functions
* Optimization
* Generalization
* Overfitting vs underfitting
* Bias vs variance
* Data leakage
* Distribution shift
* Basic ML lifecycle
* Why ML fundamentals matter for embeddings and LLM systems

---

# 30. Agentic AI Engineer Perspective

You don't need to memorize every ML algorithm.

Your goal is to develop enough ML intuition to answer questions like:

```text
Should I use an ML model here?

What type of problem is this?

What data does the model need?

What should the target be?

How should I evaluate it?

Could the evaluation be misleading?

Could data leakage exist?

Will the model generalize?

How will it behave in production?

What happens when the data distribution changes?
```

That level of understanding provides the foundation for the later parts of this roadmap:

```text
Machine Learning
      ↓
Embeddings
      ↓
Vector Search
      ↓
RAG
      ↓
LLM Applications
      ↓
Agents
      ↓
Agentic AI Systems
```
