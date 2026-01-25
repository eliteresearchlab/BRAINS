# BRAINS Platform — Multi-Agent Medical Diagnostic System

![Status](https://img.shields.io/badge/status-active-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Python](https://img.shields.io/badge/python-3.11+-blue)
![Node.js](https://img.shields.io/badge/node-18+-green)

**BRAINS** is a production-ready, multi-agent medical diagnostic platform powered by **Retrieval-Augmented Generation (RAG)** and **LLM inference**. It combines a Next.js frontend, NestJS backend, and Python ML service to provide domain-specific medical diagnosis with semantic search and conversational chat capabilities.

# Explore pages - [BRAINS](https://abirbokhtiar.github.io/BRAINS/)


## Table of Contents

- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Core Components](#core-components)
- [API Documentation](#api-documentation)
- [Configuration](#configuration)
- [Docker & Deployment](#docker--deployment)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Quick Start

### Prerequisites

- **Python** 3.11 or 3.12 (Windows, Mac, Linux)
- **Node.js** 18+ (for NestJS and Next.js)
- **Docker** (optional, for containerized deployment)
- **OpenAI API Key** (for LLM integration)

### 1. Clone & Setup ML Service

```bash
cd ml-service-python

# Create virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1  # Windows PowerShell
# or: source .venv/bin/activate  # Mac/Linux

# Install dependencies
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt

# Build knowledge base (generates FAISS index)
python build_kb.py

# Start ML service
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Expected output:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
[STARTUP] ML Service Ready. FAISS + Multi-Agent System Loaded.
```

### 2. Setup NestJS Backend

```bash
cd backend-nestjs/backend-nestjs

# Install dependencies
npm install

# Create .env (if needed)
# ML_SERVICE_URL=http://localhost:8000

# Start development server
npm run start:dev
```

Expected output:
```
[Nest] 12345 - 11/27/2025, 2:30:45 PM   [NestFactory] Nest application successfully started +12ms
```

### 3. Setup Next.js Frontend

```bash
cd frontend-nextjs

# Install dependencies
npm install

# Create .env.local
# NEXT_PUBLIC_API_BASE=http://localhost:4000

# Start dev server
npm run dev
```

Expected output:
```
▲ Next.js 14.x.x
  ▲ Local:        http://localhost:3000
```

### 4. Test the System

Open browser to `http://localhost:3000` and:
- Submit a diagnosis query in the form
- View real-time results with evidence and confidence scores
- Use chat interface for general medical questions

---

## Project Structure

```
brains-project/
├── frontend-nextjs/                    # Next.js UI application
│   ├── src/app/
│   │   ├── page.tsx                    # Landing page
│   │   ├── chat/                       # Chat interface
│   │   ├── diagnose/                   # Diagnosis form
│   │   ├── components/                 # Reusable UI components
│   │   ├── lib/                        # API client helpers
│   │   └── api/                        # Next.js API routes (optional proxy)
│   ├── package.json
│   └── tsconfig.json
│
├── backend-nestjs/
│   └── backend-nestjs/
│       ├── src/
│       │   ├── main.ts                 # App bootstrap
│       │   ├── app.module.ts           # Root module
│       │   ├── app.controller.ts       # Health & general routes
│       │   ├── agents/                 # Agents domain
│       │   │   ├── agents.controller.ts
│       │   │   ├── agents.service.ts
│       │   │   └── agents.module.ts
│       │   ├── ml/                     # ML client service
│       │   │   ├── ml.service.ts
│       │   │   └── ml.module.ts
│       │   ├── patient/                # Patient domain
│       │   └── auth/                   # Auth module (if present)
│       ├── package.json
│       ├── tsconfig.json
│       └── Dockerfile
│
├── ml-service-python/                  # Python FastAPI ML service
│   ├── build_kb.py                     # Knowledge base builder
│   ├── requirements.txt                # Python dependencies
│   ├── Dockerfile
│   ├── data/
│   │   ├── faiss.index                 # FAISS vector index (binary)
│   │   ├── kb_texts.jsonl              # Knowledge base (JSONL format)
│   │   └── kb_meta.json                # Metadata (optional)
│   └── app/
│       ├── main.py                     # FastAPI server & endpoints
│       ├── encoder.py                  # SentenceTransformer wrapper
│       ├── indexer.py                  # FAISS index management
│       ├── schemas.py                  # Pydantic models
│       ├── agents_controller.py        # Agent orchestration utilities
│       ├── ml_core/                    # Core ML utilities
│       │   ├── __init__.py
│       │   ├── brains_retrieval.py     # FAISS retrieval wrapper
│       │   ├── prompt_builder.py       # LLM prompt construction
│       │   └── llm_connector.py        # LLM integration (OpenAI)
│       ├── agents/                     # Multi-agent orchestrator
│       │   ├── __init__.py
│       │   ├── domains/                # Domain-level routers
│       │   │   ├── __init__.py
│       │   │   ├── neurology.py        # Neurology agents
│       │   │   ├── cardiology.py       # Cardiology agents
│       │   │   ├── pulmonology.py      # Pulmonology agents
│       │   │   └── general.py          # General medical assistant
│       │   └── orchestrator/           # Orchestrator agent
│       │       ├── __init__.py
│       │       └── orchestrator.py     # Main orchestrator logic
│       └── diseases/                   # Disease-specific agents
│           ├── __init__.py
│           ├── alzheimers.py           # Alzheimer's agent
│           ├── brain_tumor.py          # Brain tumor agent
│           ├── stroke.py               # Stroke agent
│           ├── heart_failure.py        # Heart failure agent
│           └── copd.py                 # COPD agent
│
├── infra/
│   └── docker-compose.yml              # Orchestration for all services
│
└── README.md                           # This file
```

---

## Architecture

### System Diagram

```
┌─────────────────┐
│   Next.js UI    │ (3000)
│   • Chat        │
│   • Diagnose    │
└────────┬────────┘
         │ HTTP REST
         ▼
┌─────────────────┐
│  NestJS API     │ (4000)
│   • Routes      │
│   • Auth        │
│   • ML Gateway  │
└────────┬────────┘
         │ HTTP REST
         ▼
┌─────────────────────────────┐
│  Python ML Service (8000)   │
│  ┌─────────────────────┐    │
│  │ Orchestrator Agent  │    │
│  │ • Route to domain   │    │
│  │ • Select agent      │    │
│  └──────────┬──────────┘    │
│             │               │
│  ┌──────────┴─────────────┐ │
│  ▼                        ▼ │
│ General Domain      Specific Domains
│ (Chat Agent)       (Neurology, Cardiology, etc.)
│  │                      │
│  │                      ▼
│  │              Disease Agents
│  │              (Alzheimer's, Stroke, etc.)
│  │                      │
│  └──────┬───────────────┘
│         ▼
│  ┌─────────────────┐
│  │  RAG Pipeline   │
│  │ 1. Embed query  │
│  │ 2. FAISS search │
│  │ 3. Build prompt │
│  │ 4. LLM call     │
│  └────────┬────────┘
│           │
│  ┌────────┴───────┬───────────┐
│  ▼                ▼           ▼
│ FAISS Index    Prompt      OpenAI
│ (data/)        Builder     LLM API
└─────────────────────────────────┘
```

### Message Flow (Diagnosis Request)

```
User Input
    │
    ▼
Next.js Form
    │
    ├─ POST /api/proxy/diagnose
    │
    ▼
NestJS Backend (4000)
    │
    ├─ POST /api/ml/diagnose
    │
    ▼
Python ML Service (8000)
    │
    ├─ orchestrator.choose_domain(patient)
    │
    ├─ domain.select_agent(patient)
    │
    ├─ agent.diagnose(patient):
    │   ├─ encoder.encode(query)
    │   ├─ indexer.search(vector, k=5)  [FAISS]
    │   ├─ prompt_builder.build_prompt(patient, docs)
    │   ├─ llm_connector.generate(prompt)  [OpenAI]
    │   └─ Return structured diagnosis
    │
    ▼
Response JSON
{
  "domain": "neurology",
  "agent": "alzheimers",
  "diagnosis": "Probable Alzheimer's Disease...",
  "confidence": 85,
  "rationale": "Based on MMSE score and memory loss...",
  "retrieved": [...]
}
    │
    ▼
NestJS → Next.js
    │
    ▼
Display Results
```

---

## Core Components

### 1. Orchestrator Agent

**File:** `ml-service-python/app/agents/orchestrator/orchestrator.py`

Routes patient data to the appropriate domain and disease agent.

**Input:**
```python
{
  "age": 72,
  "symptoms": ["memory loss", "confusion"],
  "mmse": 22,
  "_requested_domain": "neurology"  # optional override
}
```

**Output:**
```python
{
  "domain": "neurology",
  "agent": "alzheimers",
  "diagnosis": {...}
}
```

**Logic:**
1. Extract text from patient dict
2. Score domains by keyword matching (memory → neurology, cough → pulmonology)
3. Call `domain.select_agent(patient)` to pick specific disease agent
4. Call `agent.diagnose(patient)` to perform RAG + LLM inference

---

### 2. Domain Modules

**Location:** `ml-service-python/app/agents/domains/`

Each domain (neurology, cardiology, pulmonology, general) exposes:
- `select_agent(patient)` → returns agent name
- `get_agent(name)` → returns agent instance

**Example: Neurology Domain**
```python
# Selects between: alzheimers, stroke, brain_tumor
def select_agent(patient):
    text = str(patient).lower()
    if "memory" in text:
        return "alzheimers"
    if "stroke" in text:
        return "stroke"
    return "alzheimers"  # default
```

---

### 3. Disease Agents (RAG Pipeline)

**Location:** `ml-service-python/app/diseases/`

Each disease agent implements `.diagnose(patient)`:

```python
class AlzheimersAgent:
    def diagnose(self, patient):
        # 1. Build query
        query = format_patient_text(patient)
        
        # 2. Retrieve documents (FAISS)
        evidence = retrieve_for_patient_text(query, k=5)
        
        # 3. Build prompt
        prompt = prompt_builder.build_prompt(
            patient, evidence, disease="Alzheimer's disease"
        )
        
        # 4. Call LLM
        response = llm_connector.generate(prompt)
        
        # 5. Parse & return
        return {
            "summary": diagnosis_text,
            "confidence": 85,
            "rationale": explanation,
            "evidence": evidence
        }
```

---

### 4. Retrieval Agent (FAISS)

**File:** `ml-service-python/app/indexer.py` + `ml_core/brains_retrieval.py`

Semantic search over vector index:

```python
def search(query_embedding, top_k=5):
    # FAISS inner product search
    scores, indices = index.search(query_emb, top_k)
    results = [documents[i] for i in indices[0] if i < len(documents)]
    return results
```

---

### 5. LLM Connector

**File:** `ml-service-python/app/ml_core/llm_connector.py`

Integrates with OpenAI API:

```python
def generate(prompt, temperature=0.2, max_tokens=512):
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ],
        temperature=temperature,
        max_tokens=max_tokens
    )
    return response.choices[0].message.content
```

---

## API Documentation

### ML Service Endpoints

#### 1. Health Check
```http
GET /health
```

**Response:**
```json
{
  "status": "ok",
  "service": "ml-service-python-multi-agent"
}
```

---

#### 2. Retrieve Documents
```http
POST /api/retrieve
Content-Type: application/json

{
  "text": "memory loss and confusion",
  "k": 5
}
```

**Response:**
```json
{
  "docs": [
    {
      "id": 105,
      "domain": "neurology",
      "disease": "alzheimers",
      "text": "Progressive short-term memory loss, disorientation...",
      "score": 0.92
    },
    ...
  ]
}
```

---

#### 3. Diagnose (Auto-Route)
```http
POST /api/ml/diagnose
Content-Type: application/json

{
  "age": 72,
  "mmse": 22,
  "cdr": 1,
  "symptoms": ["memory loss", "confusion"]
}
```

**Response:**
```json
{
  "domain": "neurology",
  "agent": "alzheimers",
  "diagnosis": {
    "summary": "Probable Alzheimer's Disease based on MMSE score...",
    "confidence": 85,
    "rationale": "Cognitive decline with memory impairment...",
    "evidence": [...]
  }
}
```

---

#### 4. Diagnose (Explicit Domain)
```http
POST /api/ml/diagnose/neurology
Content-Type: application/json

{
  "age": 72,
  "symptoms": ["memory loss"]
}
```

---

### NestJS Backend Endpoints

#### Diagnose (via NestJS)
```http
POST /api/ml/diagnose
Content-Type: application/json

{
  "age": 72,
  "symptoms": ["memory loss"]
}
```

Proxies to ML service and returns same response.

---

### Next.js Frontend Routes

| Route | Description |
|-------|-------------|
| `/` | Landing / chat interface |
| `/chat` | Chat with medical assistant |
| `/diagnose` | Patient intake form |
| `/results` | Diagnosis results display |
| `/api/proxy/diagnose` | Server-side proxy to ML service |

---

## Configuration

### Environment Variables

**ML Service** (`ml-service-python/.env`)
```bash
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
OPENAI_API_BASE=https://api.openai.com/v1
SBERT_MODEL=all-MiniLM-L6-v2
```

**NestJS** (`backend-nestjs/.env`)
```bash
ML_SERVICE_URL=http://localhost:8000
DATABASE_URL=postgres://user:pass@localhost:5432/brains
NODE_ENV=development
```

**Next.js** (`frontend-nextjs/.env.local`)
```bash
NEXT_PUBLIC_API_BASE=http://localhost:4000
```

### Knowledge Base Configuration

**Building KB from scratch:**
```bash
cd ml-service-python
python build_kb.py
```

This script:
1. Reads medical data from `build_kb.py` (hardcoded examples)
2. Generates embeddings via SentenceTransformer
3. Builds FAISS index
4. Saves to `data/faiss.index` and `data/kb_texts.jsonl`

**Custom KB Format:**
Each line in `kb_texts.jsonl` must be valid JSON:
```json
{"id": 101, "domain": "neurology", "disease": "alzheimers", "text": "Clinical case...", "source": "pubmed"}
```

---

## Docker & Deployment

### Local Docker Compose

```bash
cd infra
docker-compose up --build
```

**Services:**
- `frontend`: Next.js (3000)
- `backend`: NestJS (4000)
- `ml-service`: Python (8000)
- `db`: PostgreSQL (5432, optional)

### Production Deployment

**Frontend (Vercel):**
```bash
cd frontend-nextjs
vercel deploy --prod
```

**Backend (Render / AWS):**
```bash
# Push to Render
git push origin main
# Auto-deploys via webhook
```

**ML Service (AWS ECS Fargate / Render):**
```bash
# Build & push image
docker build -t brains-ml:latest ml-service-python/
docker tag brains-ml:latest YOUR_ECR_REPO/brains-ml:latest
docker push YOUR_ECR_REPO/brains-ml:latest

# Deploy to ECS/Render
```

---

## Security

### Best Practices

1. **PII Protection**
   - Never log full patient data; hash/mask identifiers
   - Encrypt data at rest and in transit (TLS)
   - HIPAA compliance: minimal PII storage, audit trails

2. **API Security**
   - Use JWT for authentication
   - Rate limiting per user/IP
   - CORS configured for frontend domain only

3. **LLM Safety**
   - Content moderation on outputs (OpenAI Moderation API)
   - Disclaimer: "I am an AI, not a clinician"
   - Human-in-the-loop for high-confidence diagnoses

4. **Secrets Management**
   - Never commit `.env` files
   - Use environment variables or vaults (AWS Secrets Manager, HashiCorp Vault)
   - Rotate API keys regularly

---

## Troubleshooting

### Issue: FAISS Index Not Found
```
FileNotFoundError: data/faiss.index not found
```

**Solution:**
```bash
cd ml-service-python
python build_kb.py
```

---

### Issue: Invalid JSON in KB
```
json.decoder.JSONDecodeError: Expecting value: line 23...
```

**Solution:**
Clean `kb_texts.jsonl`:
```bash
# Remove invalid lines
python -c "
import json
with open('data/kb_texts.jsonl', 'r') as f:
    lines = f.readlines()
valid = [l for l in lines if l.strip()]
with open('data/kb_texts.jsonl', 'w') as f:
    f.writelines(valid)
"
```

---

### Issue: Python Dependencies Build Failure

**Error:**
```
AttributeError: module 'pkgutil' has no attribute 'ImpImporter'
```

**Solution:**
Use Python 3.11 or 3.12, not 3.13:
```bash
python --version  # Check version
# If 3.13, switch to 3.11 or 3.12
python -m venv .venv --python=python3.11
```

---

### Issue: ML Service Not Reachable from NestJS

**Error:**
```
ENOTFOUND ml-service
```

**Solution:**
Update `backend-nestjs/.env`:
```bash
ML_SERVICE_URL=http://localhost:8000  # for local dev
```

---

### Issue: OpenAI API Key Invalid

**Error:**
```
AuthenticationError: API key not valid
```

**Solution:**
1. Check `.env` has valid `OPENAI_API_KEY`
2. Verify key in OpenAI dashboard (usage, expiry)
3. Restart services after updating key

---

## Contributing

### Workflow

1. Create feature branch: `git checkout -b feature/your-feature`
2. Make changes and test locally
3. Commit with clear messages: `git commit -m "Add multi-agent support"`
4. Push and open PR: `git push origin feature/your-feature`
5. Await review and merge

### Testing

```bash
# NestJS unit tests
cd backend-nestjs
npm test

# Python unit tests
cd ml-service-python
pytest app/

# Next.js linting
cd frontend-nextjs
npm run lint
```

---

## License

MIT License — see LICENSE file for details.

---

## Support & Contact

For issues, questions, or feature requests:
- **Email:** abirbokhtiar107@gmail.com
- **Issues:** GitHub Issues
- **Documentation:** [Full Wiki](https://abirbokhtiar.github.io/BRAINS/docs/)

---

## Roadmap

- [ ] Add WebSocket streaming for chat responses
- [ ] Integrate with FHIR EHRs
- [ ] Fine-tune embedding model on clinical text
- [ ] Implement clinician dashboard with audit trails
- [ ] Deploy to Kubernetes cluster
- [ ] Add multi-language support

---

**Last Updated:** November 27, 2025

Made by @Abir
