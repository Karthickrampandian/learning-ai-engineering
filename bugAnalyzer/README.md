# Bug Analyzer

An AI-powered bug analysis tool that fetches bugs from Jira, detects duplicates, identifies relevant source files dynamically, and provides structured code fixes using Claude AI.

## What it does

Takes a bug report → finds similar past bugs → analyses source code → shows exact bug location with before/after fix.

## Pipeline

**1. Bug Ingestion**
- Connects to Jira via JQL to fetch bugs (currently hardcoded for demo)
- Returns bug ID and summary as key-value pairs

**2. Duplicate Detection**
- Queries ChromaDB with bug summary to find top 2 similar past bugs
- Builds context from similar bugs if found
- Passes summary + context (or summary alone) to Claude

**3. Bug Analysis (Claude Call 1)**
- Claude returns: severity, priority, component, suggestion, title
- Output mapped to structured JSON

**4. Dynamic File Identification (Claude Call 2)**
- Scans local repository for all source files (`.jsx`, `.js`, `.ts`, `.py`)
- Passes file list + bug summary to Claude
- Claude identifies top 3 most relevant files — no hardcoding

**5. Code Analysis (Claude Call 3)**
- Reads content of identified files
- Claude analyses code and returns structured JSON:
  - Bug location (function + line number)
  - Buggy code (complete function)
  - Fixed code (complete fixed function)
  - What changed (before/after diff)
  - One-line explanation

**6. Caching**
- Upserts each bug into ChromaDB after analysis
- Used for duplicate detection in future runs

**7. Streamlit UI**
- Displays all bugs as expandable cards
- Shows similar past bugs, severity, priority, component
- Side-by-side buggy vs fixed code with syntax highlighting
- "What Changed" section showing exact diff
- CSV export of full bug report

## Tech Stack

- **Python** — core language
- **Claude API (Opus)** — bug analysis, file identification, code fix
- **ChromaDB** — vector store for duplicate detection and caching
- **Streamlit** — UI
- **Jira REST API** — bug source (hardcoded for demo)

## Key Design Decisions

- **Dynamic file identification** — Claude decides which files to analyse based on bug description. No hardcoded file maps.
- **3-call pipeline per bug** — analysis → file ID → code fix. Each call has a focused prompt.
- **Duplicate detection before LLM call** — saves API cost by surfacing known bugs first.
- **Structured JSON output** — all Claude responses parsed to JSON for consistent UI rendering.

## Branches

| Branch | Approach |
|---|---|
| `main` | Full pipeline with code analysis |
| `feature/basic` | Simple Jira → LLM → Streamlit |
| `feature/tool-use` | Claude tool use implementation |
| `feature/langgraph-agent` | LangGraph agent with conditional routing |
| `feature/rag-chromadb` | RAG-based similar bug retrieval |
| `feature/github-issues` | Dynamic file identification (current) |

## Run Locally

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=your_key_here
streamlit run Bug_APP.py
```

## Limitations

- Jira personal tier API token has scope restrictions — bugs hardcoded for demo
- Repo path currently points to local SauceDemo clone
- CrossEncoder runs on CPU — slow on free tier deployment
