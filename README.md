# VersionRAG – Prototype Implementation

This repository contains a prototype implementation of **VersionRAG (Versioned Document Retrieval-Augmented Generation)** — a retrieval-augmented question answering system designed to handle **versioned document collections**. The project was developed for academic purposes as part of a bachelor's thesis.

> ⚠️ **Prototype Notice**  
> This project is a **research prototype** developed for academic evaluation purposes only.  
> It is not optimized for production use, and the implementation prioritizes clarity and experimentation over code quality or performance.

## Overview

**VersionRAG** is a retrieval-augmented generation (RAG) pipeline developed specifically for working with **versioned document collections**. It supports version-aware retrieval and reasoning by explicitly modeling:

- document and version metadata,
- changes across document states (e.g., additions, deprecations),
- and a version-sensitive graph structure.

The core idea is to build a **version graph** during indexing, capturing relationships between document versions and tracking their evolution. During retrieval, VersionRAG uses a hybrid strategy: depending on the query type, it either leverages the graph structure directly (e.g., for change reasoning), or performs vector-based content retrieval with version constraints applied via the graph.

In contrast to standard RAG or GraphRAG, VersionRAG is designed to **maintain temporal alignment and version specificity**, making it suitable for scenarios that require high precision across evolving documentation.

For benchmarking purposes, the repository also includes independent implementations of:

- **Naive RAG** (baseline with standard dense retrieval),
- **GraphRAG** (retrieval via knowledge graph).

These implementations are included solely for performance comparison and are not part of the VersionRAG system.


![VersionRAG Framework Overview](data/img/framework_versionrag.png)

*Figure: High-level architecture of the VersionRAG system. The indexing phase builds a version-aware graph; the retrieval phase selects the appropriate retrieval strategy based on the question context.*

![VersionRAG Graph Structure](data/img/framework_graph_structure.png)

*Figure: Conceptual structure of the version-aware document graph. Nodes represent categories, documents, versions, content, and tracked changes.*

## Features

## ✅ VersionRAG Features

- **Multi-document support**  
  Handles multiple versioned documents across different categories, automatically distinguishing between standard documents and changelogs. Indexing is fully automated with no manual intervention required.

- **Version-aware graph construction**  
  Builds a structured graph representing documents, versions, and occured changes between subsequent versions.

- **Change tracking**  
  Detects and links additions, removals, and deprecations across document versions.

- **Version metadata indexing**  
  Extracts and stores version-specific metadata for targeted retrieval.

- **Targeted retrieval capabilities**  
  - Retrieve content for a specific document version  
  - List available document versions  
  - Identify tracked changes between two versions

- **LLM integration**  
  Supports dynamic configuration of the underlying language model used for generation.

- **Baseline implementations included**  
  Includes reference implementations of standard (naive) RAG and GraphRAG for performance comparison.

## Getting Started

### Installation

Install dependencies listed in `src/requirements.txt`:

```bash
python -m venv .venv

# Windows
.\.venv\Scripts\python -m pip install --upgrade pip
.\.venv\Scripts\python -m pip install -r src/requirements.txt

# macOS/Linux
./.venv/bin/python -m pip install --upgrade pip
./.venv/bin/python -m pip install -r src/requirements.txt
```

### Neo4j (Local / Free)

If you want to run everything locally (free), use **Neo4j Desktop** (recommended on Windows):

- Create a local DBMS (Neo4j **5.x** recommended) and start it
- Default local endpoints are:
  - Browser UI: `http://localhost:7474`
  - Bolt URI: `bolt://localhost:7687`

Then create `src/.env` (copy from `src/env.template`) and set:

- `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD`
- `NEO4J_URI_AURA`, `NEO4J_USERNAME_AURA`, `NEO4J_PASSWORD_AURA`

To verify Neo4j connectivity (without printing secrets):

```bash
# Windows
.\.venv\Scripts\python src\util\check_neo4j.py

# macOS/Linux
./.venv/bin/python src/util/check_neo4j.py
```

### Milvus (Vector DB)

This project uses `pymilvus`.

- **Linux/macOS**: you can use a local Milvus Lite database file.
- **Windows**: **Milvus Lite is not supported** by `pymilvus`, so you must run a Milvus server (e.g. via Docker) and connect using `MILVUS_URI`.

#### Windows (Docker) quick start

1) Start Milvus Standalone:

```bash
docker run -d --name milvus-standalone ^
  -p 19530:19530 -p 9091:9091 ^
  milvusdb/milvus:v2.4.9 ^
  milvus run standalone
```

2) In `src/.env` set:

- `MILVUS_URI="http://localhost:19530"`

#### Start fresh (reset Milvus)

- **Full reset (Docker, deletes data volumes)**:

```bash
docker compose -f milvus-standalone-docker-compose.yml down -v
docker compose -f milvus-standalone-docker-compose.yml up -d
```

- **Soft reset (drop only this repo's collections)**:

```bash
# Windows
.\.venv\Scripts\python src\util\reset_milvus.py

# macOS/Linux
./.venv/bin/python src/util/reset_milvus.py
```

### Using Groq (no OpenAI key) — **recommended free setup**

Groq can be used for **LLM generation**, but it does **not** provide embeddings. To run fully without OpenAI, set:

- `LLM_MODE="groq"`
- `GROQ_API_KEY="..."`
- `EMBEDDING_PROVIDER="local"`

Copy `src/env.template` → `src/.env` and adjust values.

### Running the Program

Switch into the src/ directory and run the main program:
```bash
cd src
# Windows
..\.\.venv\Scripts\python main.py

# macOS/Linux
../.venv/bin/python main.py
```

## 🔐 Configuration

To run the system, a `.env` file must be placed in the `src/` directory to provide necessary credentials for external services.

### Required environment variables:

#### 🔑 LLM API keys
- `OPENAI_API_KEY` – for OpenAI-based language models
- `GROQ_API_KEY` – for Groq-based language models

> ⚙️ The selection of which LLMs are used is configured in `src/util/constants.py`.

#### 🛠️ Local vector database (Neo4j)
- `NEO4J_URI`  
- `NEO4J_USER`  
- `NEO4J_PASSWORD`  

#### ☁️ Cloud-based vector database (Neo4j Aura)
- `NEO4J_URI_AURA`  
- `NEO4J_USERNAME_AURA`  
- `NEO4J_PASSWORD_AURA`  
- `AURA_INSTANCEID`  

> ⚠️ Make sure to never commit your `.env` file to version control.