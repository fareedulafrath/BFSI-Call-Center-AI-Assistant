# BFSI Call Center AI Assistant
A lightweight, compliant AI assistant for Banking, Financial Services and Insurance (BFSI) call center queries. It provides fast, accurate and standardized responses using a three-tier pipeline: **dataset similarity** → **fine-tuned small language model** → **RAG** (for complex financial/policy queries).

## Features

- **Tier 1**: Curated Alpaca-format BFSI dataset (150+ samples); strong similarity returns stored response (no generation).
- **Tier 2**: Local small language model (SLM), optionally fine-tuned on the same dataset.
- **Tier 3**: RAG over structured knowledge (rates, EMI, penalties, policy) for complex queries.
- **Guardrails**: Out-of-domain rejection, PII detection, no guessing of financial numbers, disclaimer.

## Requirements

- Python 3.10+
- Optional: GPU for faster SLM inference and fine-tuning

## Setup

1. **Clone and enter the project**
   ```bash
   cd "BFSI Call Center AI Assistant"
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```
   (Optional: use a virtual environment: `python -m venv venv` then `venv\Scripts\activate` on Windows.)

3. **One-time setup (download embedding model, build indices)**
   ```bash
   python scripts/setup.py
   ```
   Or manually: `python scripts/build_index.py` then `python scripts/ingest_rag.py`.

4. **Verify**
   ```bash
   python scripts/test_pipeline.py
   ```
   This tests Tier 1 and guardrails without loading the SLM.

5. **Configuration**
   - Copy `.env.example` to `.env` if you need overrides.
   - Edit `config.yaml` for similarity threshold, model paths, RAG top-k, etc.
   - On Windows, `use_4bit` is false by default (bitsandbytes not supported); the SLM runs in full precision.

6. **Optional: Fine-tune the SLM (Tier 2)**
   ```bash
   python scripts/finetune.py
   ```
   Downloads the base model, trains LoRA adapters, and saves them under `models/adapters/v1.0`. Requires sufficient RAM/GPU.

**First run (Streamlit/CLI):** Queries that match the dataset (e.g. "How is EMI calculated?") are answered immediately from Tier 1. The first query that does *not* match will trigger a one-time download of the base SLM (TinyLlama, ~600MB) from Hugging Face; subsequent responses will use the cached model. If the SLM fails to load (e.g. no GPU, low memory), you will get a fallback message asking the user to rephrase or contact customer care.

## Running the demo

- **CLI**
  ```bash
  python demo/cli.py
  ```

- **Streamlit UI**
  ```bash
  streamlit run demo/app_streamlit.py
  ```

- **FastAPI**
  ```bash
  uvicorn demo.api:app --reload
  ```
  Then `POST /query` with `{"query": "How is EMI calculated?"}`.

## 📁 Project Structure

BFSI-Call-Center-AI-Assistant/
│
├── data/
│   ├── alpaca_bfsi.json
│   ├── dataset_index/
│   └── chroma_db/
│
├── models/
│   ├── base_slm/
│   └── finetuned_adapters/
│
├── src/
│   ├── similarity/
│   ├── slm/
│   ├── rag/
│   ├── orchestrator/
│   ├── guardrails/
│   ├── config/
│   └── logging_config/
│
├── scripts/
│   ├── build_dataset.py
│   ├── validate_dataset.py
│   ├── build_index.py
│   ├── ingest_rag.py
│   └── finetune.py
│
├── knowledge/
│   ├── rates.md
│   ├── penalties.md
│   └── product_overview.md
│
├── demo/
│   ├── cli_app.py
│   ├── streamlit_app.py
│   └── fastapi_app.py
│
├── docs/
│   ├── architecture.md
│   └── runbook.md
│
├── requirements.txt
└── README.md

## Documentation

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for architecture, data flow, tier logic, guardrails, and how to update the dataset, model, and knowledge base.

## Compliance and safety

- Responses are prioritized from the curated dataset; the SLM is used only when there is no strong match.
- Complex financial/policy queries use RAG so answers are grounded in knowledge documents.
- No guessing of rates or policy numbers; no exposure of sensitive customer data; out-of-domain queries are rejected.
=======
