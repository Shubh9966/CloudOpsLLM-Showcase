<div align="center">

# 🛡️ CloudOpsLLM — Self-RAG Incident Response Copilot

<p align="center">
  <img src="https://img.shields.io/badge/PYTHON-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/FASTAPI-0.100%2B-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI" />
  <img src="https://img.shields.io/badge/LANGGRAPH-ORCHESTRATION-1C3041?style=for-the-badge&logo=langchain&logoColor=white" alt="LangGraph" />
  <img src="https://img.shields.io/badge/GROQ-AI%20INFERENCE-F55036?style=for-the-badge&logo=groq&logoColor=white" alt="Groq" />
  <img src="https://img.shields.io/badge/PINECONE-VECTOR%20DB-000000?style=for-the-badge&logo=pinecone&logoColor=white" alt="Pinecone" />
  <img src="https://img.shields.io/badge/TAVILY-WEB%20SEARCH-4A5568?style=for-the-badge&logo=tavily&logoColor=white" alt="Tavily" />
  <img src="https://img.shields.io/badge/SQLITE-MEMORY-003B57?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLite" />
  <img src="https://img.shields.io/badge/LANGSMITH-TRACING-333333?style=for-the-badge&logo=langchain&logoColor=white" alt="LangSmith" />
  <img src="https://img.shields.io/badge/Repository-Private-critical?style=for-the-badge&logo=github" alt="Repository Private" />
</p>

**Ground every production troubleshooting decision in private operational runbooks, or verified real-time internet context when internal docs fall short.**

</div>

---

## 🔒 Source Code Access Notice

> **Note:** The core proprietary production implementation, adaptive Self-RAG routing nodes, and custom operational chunking workflows are maintained inside an authenticated private repository.  
> If you are a recruiter, collaborator, or engineering team looking to audit the full codebase or evaluate architecture integration, please contact: **[shubham.shivare9112@gmail.com](mailto:shubham.shivare9112@gmail.com)**.



## 🏗️ Architecture & Self-RAG Graph

```mermaid
flowchart TD
    classDef entry fill:#1E293B,stroke:#0F172A,stroke-width:2px,color:#F8FAFC,font-weight:bold
    classDef memory fill:#0284C7,stroke:#0369A1,stroke-width:2px,color:#FFFFFF
    classDef retrieve fill:#0D9488,stroke:#0F766E,stroke-width:2px,color:#FFFFFF
    classDef grade fill:#D97706,stroke:#B45309,stroke-width:2px,color:#FFFFFF
    classDef rewrite fill:#7C3AED,stroke:#6D28D9,stroke-width:2px,color:#FFFFFF
    classDef generate fill:#2563EB,stroke:#1D4ED8,stroke-width:2px,color:#FFFFFF
    classDef verify fill:#059669,stroke:#047857,stroke-width:2px,color:#FFFFFF
    classDef fallback fill:#E11D48,stroke:#BE123C,stroke-width:2px,color:#FFFFFF
    classDef limit fill:#475569,stroke:#334155,stroke-width:2px,color:#FFFFFF
    classDef guard fill:#DB2777,stroke:#BE185D,stroke-width:2px,color:#FFFFFF

    START((START)):::entry --> RL{"IP Rate Limit - /api/chat<br/>Max 2 requests per IP per hour?"}:::limit
    RL -->|"No - Quota Exceeded"| BLK["HTTP 429 Response<br/>Rate-limit notice with contact email<br/>No LLM calls triggered"]:::limit
    RL -->|"Yes - Within Limit"| C["1. contextualize<br/>SQLite Thread Memory"]:::memory
    C --> D{"2. decide_retrieval<br/>Needs Evidence?"}:::grade

    D -->|"No - Direct Inquiries"| DIR{"3a. direct<br/>Cloud-related? Strict Guardrail"}:::guard
    D -->|"Yes - Diagnostic Queries"| RET["3b. retrieve<br/>Pinecone Top-K Runbooks"]:::retrieve

    RET --> GR{"4. grade - IsREL<br/>Evidence Relevant?"}:::grade

    GR -->|"Relevant > 0"| GEN["8. generate<br/>Draft Answer from Context"]:::generate
    GR -->|"0 Relevant, Retries < 2"| RWI["5. rewrite_internal<br/>Optimize Vector Query"]:::rewrite
    GR -->|"Internal Rewrites Exhausted"| RWW["6. rewrite_web<br/>Build Web Query"]:::rewrite
    GR -->|"All Fallbacks Exhausted"| NO["12. no_answer<br/>Safe Operational Fallback"]:::fallback

    RWI --> RET
    RWW --> WEB["7. web_search<br/>Tavily Search API"]:::retrieve
    WEB --> GR

    GEN --> SUP{"9. support - IsSUP<br/>Hallucination Check"}:::verify
    SUP -->|"Partial/No Support, Retries < 2"| REV["10. revise<br/>Remove Ungrounded Claims"]:::rewrite
    REV --> SUP

    SUP -->|"Fully Supported"| USE{"11. usefulness - IsUSE<br/>Solves Incident?"}:::verify
    USE -->|"Not Useful - Internal"| RWI
    USE -->|"Not Useful - Web"| RWW
    USE -->|"Useful"| MEM["13. commit_memory<br/>Save Checkpoint & Audit Record"]:::memory

    DIR -->|"Yes - Valid Cloud Query"| MEM
    DIR -->|"No - Non-Cloud Topic"| REF["Guardrail Refusal<br/>Ask strictly cloud-related questions only<br/>No retrieval or external search"]:::guard
    REF --> MEM
    NO --> MEM
    MEM --> END((END)):::entry
```

**Graph Color Legend:**  
🟦 **Blue (`Memory & Generation`)** · 🟩 **Teal (`Pinecone / Tavily Retrieval`)** · 🟧 **Amber (`LLM Grading & Routing`)** · 🟪 **Purple (`Self-Correction & Query Rewrite`)** · 🟢 **Emerald (`IsSUP & IsUSE Verification`)** · 🟥 **Crimson (`Safe No-Answer Fallback`)** · ⬛ **Slate (`IP Rate Limiter`)** · 🩷 **Pink (`Cloud-Only Guardrail & Refusal`)**

---
---

## ✨ Core Highlights

- 🧠 **13-Node LangGraph Self-RAG**: Dynamically routes incident queries, grades evidence relevance (`IsREL`), automatically rewrites sub-optimal queries, verifies context support/hallucinations (`IsSUP`), and validates operational usefulness (`IsUSE`).
- 📚 **Private Runbooks First**: Prioritizes internal SOP documents (`.pdf`, `.md`, `.txt`, `.docx`) utilizing deterministic SHA-256 chunking (`_stable_chunk_id`) to ensure repeated ingestions never produce duplicate vector entries.
- 🌐 **Smart Web Fallback**: Triggers targeted **Tavily** web search only after internal operational runbook query rewrites have been exhausted without sufficient relevance score.
- 🚧 **Cloud-Only Guardrail**: Non-retrieval conversational questions route through a strict cloud classifier. Queries unrelated to cloud, DevOps, or site-reliability engineering are halted with a safe, direct guardrail refusal without invoking downstream tools.
- ⏱️ **IP Rate Limiting**: Enforces a sliding-window rate limit (default: 2 requests per IP per hour) on `/api/chat` with `X-Forwarded-For` proxy awareness to safeguard LLM tokens and downstream APIs.
- 💾 **Stateful Checkpointing & Audit Trails**: Preserves continuous thread state across multi-turn diagnostic sessions using SQLite checkpointing (`langgraph_memory.sqlite`) and logs every routing step and query trace to an audit store (`audit.db`).

---
---

## 🔌 API Endpoints Reference

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/chat` | Runs the full 13-node Self-RAG graph (`{"question": "...", "thread_id": "..."}`) and returns response, active route, `IsSUP` score, `IsUSE` assessment, and workflow trace. Rate-limited to 2 requests/IP/hr. |
| `POST` | `/api/upload` | Uploads and indexes new operational runbooks (`.pdf`, `.md`, `.txt`, `.docx`) into Pinecone with deduplication hashes. |
| `GET` | `/api/audits` | Retrieves chronological audit event records from SQLite (`?limit=20`). |
| `GET` | `/api/health` | Health and connectivity check across database and LLM inference providers. |

---

## 🔍 Observability & Auditing

1. **Integrated UI Trace Drawer & SQLite Audit**:
   - Each state transition is captured within `state["trace"]`, exposed in the console's **"Inspect Self-RAG workflow trace"** panel, and permanently logged to `data/audit.db`.
2. **Native LangSmith Telemetry**:
   - Built on `langchain-core>=0.3.0` and `langgraph>=0.2.0`, configuring the following environment variables enables automatic end-to-end tracing:
     ```env
     LANGCHAIN_TRACING_V2=true
     LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
     LANGCHAIN_API_KEY=lsv2_pt_your_langsmith_key
     LANGCHAIN_PROJECT=CloudOpsLLM
     ```

---

## 📬 Contact & Inquiries

For technical audits, integrations, or access requests:
- **Email:** [shubham.shivare9112@gmail.com](mailto:shubham.shivare9112@gmail.com)
- **GitHub:** [@Shubh9966](https://github.com/Shubh9966)
