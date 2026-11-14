# VectorDB — HNSW + KD-Tree + Brute Force + RAG

A self-contained C++ vector database with a web UI. Supports three ANN algorithms (HNSW, KD-Tree, Brute Force), three distance metrics, and a full RAG pipeline via local Ollama.

---

## Local Development

### Prerequisites
- GCC / G++ with C++17 support
- On Windows: [MSYS2 / UCRT64](https://www.msys2.org/) is the easiest way to get a working GCC

### Build

```powershell
# Add MSYS2 to PATH if needed (one-time per shell session)
$env:PATH = "C:\msys64\ucrt64\bin;" + $env:PATH

# Compile
g++ -std=c++17 -O2 main.cpp -o db.exe -lws2_32
```

### Run

```powershell
.\db.exe
```

Expected output:
```
=== VectorDB Engine ===
Port: 8080
20 demo vectors | 16 dims | HNSW+KD-Tree+BruteForce
Ollama: OFFLINE (install from ollama.com)
Starting HTTP server on port 8080 ...
```

Open **http://localhost:8080** in your browser.

---

## RAG / AI Features (optional)

Requires [Ollama](https://ollama.com) running locally:

```bash
ollama pull nomic-embed-text   # embedding model (768D)
ollama pull llama3.2           # generation model
ollama serve                   # start the Ollama daemon
```

Once Ollama is running, the **Documents** and **Ask AI** tabs in the UI become fully functional.

---

## Docker

```bash
docker build -t vectordb .
docker run -p 8080:8080 vectordb
```

> Note: Ollama RAG features won't work inside Docker unless you also run Ollama and network the containers together.

---

## Production Deployment (Railway)

The binary auto-detects the `PORT` environment variable injected by Railway:

```cpp
const char* envPort = std::getenv("PORT");
if (envPort) port = std::stoi(envPort);
```

The frontend auto-detects the environment too — on production it uses relative URLs (same origin), on `localhost` it falls back to `http://localhost:8080`.

### Deploy steps
1. Push this repo to GitHub.
2. Create a new Railway project → **Deploy from GitHub repo**.
3. Railway detects the `Dockerfile` automatically and builds it.
4. No environment variables need to be set manually — `PORT` is injected by Railway.

> Ollama RAG features are **not available** on Railway (Ollama requires a local GPU/CPU process). The demo vector search, HNSW graph, benchmarks, and visualizer all work fine in production.

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Serves the web UI |
| GET | `/items` | All demo vectors |
| GET | `/search?v=…&k=5&metric=cosine&algo=hnsw` | KNN search |
| POST | `/insert` | Insert a 16D demo vector |
| DELETE | `/delete/:id` | Delete a demo vector |
| GET | `/benchmark?v=…&k=5&metric=cosine` | Compare all three algorithms |
| GET | `/hnsw-info` | HNSW graph layer stats |
| GET | `/status` | Ollama availability + counts |
| GET | `/stats` | DB stats |
| POST | `/doc/insert` | Embed & insert a document (requires Ollama) |
| GET | `/doc/list` | List all document chunks |
| DELETE | `/doc/delete/:id` | Delete a document chunk |
| POST | `/doc/search` | Semantic chunk retrieval |
| POST | `/doc/ask` | Full RAG: retrieve + generate |
