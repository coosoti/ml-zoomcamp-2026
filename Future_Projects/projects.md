# Nordwind Energie AI Platform: Portfolio Plan

A six-project portfolio built around one fictional German energy and infrastructure utility. The projects share code, data, and infrastructure so they read as one coherent AI platform rather than six unrelated demos.

**Target skills:** production AI/ML, RAG, agentic AI, Azure, Databricks, APIs and integration, MLOps, observability, Copilot ecosystem, stakeholder work, DE/EN documentation.

---

## 1. Repo structure

```
nordwind-ai-platform/
├── README.md                     # Platform overview + architecture diagram (DE/EN)
├── .github/
│   ├── workflows/ci.yml          # lint, unit tests, eval smoke test
│   └── copilot-instructions.md   # coding standards for GitHub Copilot
├── infra/                        # Bicep or Terraform (Azure resources, budgets)
├── common/                       # shared Python package
│   ├── config.py                 # env + Key Vault settings
│   ├── logging.py                # structured logging
│   ├── llm_client.py             # one wrapper: retries, timeouts, cost + token tracking
│   └── tracing.py                # OpenTelemetry / MLflow tracing helpers
├── data/
│   ├── generators/               # synthetic manuals, invoices, tickets, sensor data
│   └── README.md                 # note: all data is synthetic, no real personal data
├── p1_rag/                       # Enterprise document Q&A
├── p2_agents/                    # Support and procurement agent
├── p3_predictive_maintenance/    # Databricks lifecycle project
├── p4_extraction/                # Document extraction pipeline
├── p5_observability/             # Eval + monitoring layer
├── p6_copilot/                   # Copilot Studio + GitHub Copilot docs
├── docs/
│   ├── adr/                      # architecture decision records (short, dated)
│   ├── de/                       # German documentation
│   └── en/                       # English documentation
└── Makefile                      # make setup | test | eval | deploy | teardown
```

Every project folder follows the same layout: `README.md` (DE/EN), `src/`, `tests/`, `eval/`, `Dockerfile` (where applicable), `docs/architecture.md`.

---

## 2. Phase 0: Shared foundation (about 2 days)

- [ ] Create the monorepo, `pyproject.toml`, pre-commit (ruff, black, mypy)
- [ ] Set up `common/llm_client.py` with retry, timeout, token and cost logging
- [ ] Write IaC for: resource group, Storage Account, Key Vault, Azure OpenAI, Azure AI Search, Container Apps environment, Log Analytics
- [ ] **Set Azure budget alerts on day one** (for example 20, 50, 100 EUR)
- [ ] Add a `make teardown` target that destroys everything
- [ ] GitHub Actions: lint, unit tests, and an eval smoke test on every PR
- [ ] Write synthetic data generators (Faker plus LLM):
  - [ ] 30-50 maintenance manual and policy documents (PDF/DOCX, German and English)
  - [ ] 100+ invoices and contracts with known ground truth
  - [ ] 300+ support tickets (some with injected prompt attacks)
  - [ ] 6-12 months of sensor time series with injected failure patterns
- [ ] Document in `data/README.md` that all data is synthetic and why

**Definition of done:** a fresh clone can run `make setup && make test` and `make deploy` provisions the base infrastructure.

---

## 3. Project 3: Predictive maintenance on Databricks (weeks 1-3)

**Scenario:** predict pump or turbine failures 7 days ahead from sensor data.

**Architecture:** raw files → Bronze Delta → Silver (cleaned, joined with maintenance logs) → Gold feature tables → training job → MLflow registry → batch scoring job → monitoring table → dashboard.

### Backlog
- [ ] **3.1** Generate sensor data with injected degradation patterns and maintenance logs
- [ ] **3.2** Bronze ingestion notebook (Auto Loader if available, otherwise plain notebook)
- [ ] **3.3** Silver layer: dedupe, unit normalisation, join with maintenance events
- [ ] **3.4** Gold feature pipeline: rolling mean, variance, slope, time since last maintenance
- [ ] **3.5** Baseline model (logistic regression), then gradient boosting (LightGBM/XGBoost)
- [ ] **3.6** Log all runs to MLflow; register the best model with a `champion` alias
- [ ] **3.7** Time-based train/validation split (no random split, to avoid leakage) and document why
- [ ] **3.8** Choose the metric with a business argument (recall over precision: a missed failure costs more than a false alarm)
- [ ] **3.9** Scheduled Databricks Workflow for daily batch scoring into a Delta table
- [ ] **3.10** Drift monitoring: PSI or KS test per feature, logged daily
- [ ] **3.11** Simulate drift, show the alert firing and a retraining run
- [ ] **3.12** Unit tests for feature functions; a data-quality check step

### Deliverables
- Lineage diagram, MLflow experiment screenshot, drift chart
- Short note: what changes if you need real-time serving (Model Serving design, even if not deployable on your tier)

**Caveat:** check the compute and feature limits of Databricks Free Edition. If Model Serving is unavailable, batch scoring plus a documented serving design is still solid.

---

## 4. Project 1: Enterprise document Q&A with RAG (weeks 3-5)

**Scenario:** technicians and HR staff ask questions over manuals, policies, and GDPR documents and receive cited answers.

**Architecture:** Blob Storage → parse and chunk → embeddings → Azure AI Search (hybrid + semantic ranker) → retrieval → Azure OpenAI generation with citations → FastAPI → Streamlit/Gradio UI.

### Backlog
- [ ] **1.1** Ingestion: parse PDF/DOCX, keep headings and page numbers as metadata
- [ ] **1.2** Implement three chunking strategies: fixed size, heading-aware, parent-child
- [ ] **1.3** Index schema with fields for text, vector, source, page, section, language, allowed_roles
- [ ] **1.4** Vector-only, keyword-only, and hybrid retrieval, plus semantic reranking
- [ ] **1.5** Generation prompt: must cite sources; must answer "I don't know" if retrieval is weak
- [ ] **1.6** Build the evaluation set: 50-100 questions with reference answers and source chunks (include unanswerable questions and German/English mixes)
- [ ] **1.7** Metrics: hit rate@k, MRR, faithfulness, answer relevance
- [ ] **1.8** Experiment table comparing chunking and retrieval variants
- [ ] **1.9** Security trimming: filter by user role in the search query
- [ ] **1.10** FastAPI endpoints: `/ask`, `/health`, `/ingest`; input validation and rate limiting
- [ ] **1.11** Dockerfile and deploy to Azure Container Apps with managed identity (no keys in code)
- [ ] **1.12** Simple UI with citations shown next to answers

### Deliverables
- Results table (variant vs. metrics). This is the most convincing artifact.
- ADRs: why this chunking strategy, why this embedding model, why hybrid search

---

## 5. Project 4: Document extraction pipeline (weeks 5-6)

**Scenario:** incoming invoices and contracts become validated structured data in a downstream system.

**Architecture:** Blob upload → Event Grid / Azure Function → Document Intelligence (OCR/layout) → LLM extraction with Pydantic schema → validation rules → confidence routing → mock ERP API (or human review queue).

### Backlog
- [ ] **4.1** Pydantic schemas: invoice number, supplier, dates, line items, VAT, total, IBAN
- [ ] **4.2** Extract with schema-constrained output; auto-retry with the validation error fed back
- [ ] **4.3** Business rules: line items sum to total, VAT rate plausible (19% / 7% in Germany), IBAN checksum, date sanity
- [ ] **4.4** Confidence scoring: rule pass/fail, field agreement across two passes, model-reported uncertainty
- [ ] **4.5** Routing: auto-approve, or send to a review queue with reasons
- [ ] **4.6** Mock ERP (FastAPI) with idempotency keys
- [ ] **4.7** Retry with backoff plus a dead-letter queue
- [ ] **4.8** Evaluate field-level accuracy against ground truth
- [ ] **4.9** Compare configurations: small model + rules vs. larger model; report cost per document and auto-approval rate
- [ ] **4.10** Handle messy inputs: rotated scans, low-quality PDFs, mixed languages

### Deliverables
- Accuracy vs. cost table
- Sequence diagram of the flow, including failure paths

---

## 6. Project 2: Agentic support and procurement assistant (weeks 6-8)

**Scenario:** an agent handles internal tickets: classifies, looks up orders, checks stock, drafts replies, and escalates when unsure.

**Architecture:** ticket → orchestrator (LangGraph or Azure AI Foundry Agent Service) → tools (order API, inventory API, Project 1 RAG as knowledge tool) → guardrails → draft reply → human approval for risky actions.

### Backlog
- [ ] **2.1** Mock tool APIs: `get_order_status`, `check_stock`, `create_return`, `search_knowledge` (calls P1)
- [ ] **2.2** Explicit graph: classify → plan → tool call → verify → respond (avoid one giant prompt)
- [ ] **2.3** Typed tool schemas with argument validation
- [ ] **2.4** Guardrails: input filtering, PII redaction, allowlist of actions, hard limits on steps, tokens, and cost per ticket
- [ ] **2.5** Human-in-the-loop: refunds and cancellations pause the graph until approved
- [ ] **2.6** Prompt-injection defence: treat ticket text and tool output as untrusted data; never let it change tool permissions
- [ ] **2.7** Scenario test suite of 30-50 cases: happy paths, missing data, conflicting tool results, injection attempts, out-of-scope requests
- [ ] **2.8** Metrics: task success, tool-call correctness, escalation precision, cost per ticket
- [ ] **2.9** Expose the tools via an MCP server (optional but valuable)
- [ ] **2.10** Failure analysis document: what broke, why, how you fixed it

### Deliverables
- Trace screenshot of a full run
- Failure analysis and an "agent vs. plain workflow" note: where a deterministic workflow would have been better

---

## 7. Project 5: LLM observability and evaluation (weeks 7-9)

**Scenario:** a cross-cutting layer that makes Projects 1, 2, and 4 measurable and maintainable.

### Backlog
- [ ] **5.1** Instrument every LLM call: model, tokens, latency, cost, prompt version, trace ID
- [ ] **5.2** Prompt registry: prompts stored as versioned files; each trace records the version
- [ ] **5.3** Nightly evaluation job over the existing test sets (deterministic checks plus LLM-as-judge)
- [ ] **5.4** Calibrate the judge: label 20-30 samples yourself and report agreement
- [ ] **5.5** CI gate: PR fails if faithfulness or task success drops beyond a threshold
- [ ] **5.6** Dashboard: p50/p95 latency, cost per request, error rate, quality over time
- [ ] **5.7** Alerting on error spikes, cost anomalies, and quality drops
- [ ] **5.8** Response caching and a model-routing experiment (cheap model first, escalate on low confidence)
- [ ] **5.9** Quantify savings and quality impact

### Deliverables
- Before/after chart of cost vs. quality
- One-page "how we know it is degrading" runbook

---

## 8. Project 6: Copilot ecosystem (weeks 9-10)

**Scenario:** business users get a low-code agent; you document how you use AI tools in engineering work.

### Backlog
- [ ] **6.1** Copilot Studio agent grounded on SharePoint documents (check what your Microsoft account and region allow)
- [ ] **6.2** Custom connector to the Project 2 ticket API
- [ ] **6.3** Trade-off document: Copilot Studio vs. custom RAG/agent stack (control, cost, governance, evaluation, data residency)
- [ ] **6.4** `.github/copilot-instructions.md` with coding standards
- [ ] **6.5** "GitHub Copilot workflow" doc: test generation, refactoring, PR review, plus examples where you rejected suggestions and why

### Deliverables
- The trade-off document. Expect the interview question "when would you not use it?"

---

## 9. Timeline

| Weeks | Focus |
|---|---|
| 0 | Foundation, synthetic data, IaC |
| 1-3 | P3 Databricks lifecycle |
| 3-5 | P1 RAG |
| 5-6 | P4 Extraction |
| 6-8 | P2 Agents |
| 7-9 | P5 Observability (parallel to P2) |
| 9-10 | P6 Copilot |
| 11-12 | Polish: diagrams, DE/EN READMEs, demo video, platform overview |

**If time runs short, cut in this order:** P6, then P4. The strongest trio for this role is P1, P3, and P5.

---

## 10. Free learning resources per project

Catalogs change, so verify what is currently offered.

| Project | Resources |
|---|---|
| P3 | Databricks Academy (free self-paced ML and Data Engineering), Databricks Free Edition, MLflow docs |
| P1 | DeepLearning.AI RAG short courses, Microsoft Learn (Azure AI Search, generative AI apps), Microsoft "Generative AI for Beginners" |
| P4 | Microsoft Learn (Document Intelligence), Pydantic docs |
| P2 | DeepLearning.AI (LangGraph, agents), Hugging Face Agents Course, Microsoft "AI Agents for Beginners", Anthropic Academy |
| P5 | MLflow tracing and evaluation docs, OpenTelemetry docs, DataTalks.Club LLM Zoomcamp |
| P6 | Microsoft Learn (Copilot Studio), GitHub Copilot docs |
| MLOps general | DataTalks.Club MLOps Zoomcamp, Made With ML, Full Stack Deep Learning |

---

## 11. Quality checklist (apply to every project)

- [ ] README in German and English: problem, architecture diagram, how to run, results, limitations
- [ ] At least 3 ADRs (decision, alternatives, trade-offs)
- [ ] Unit tests and an evaluation set with a reported metric
- [ ] No secrets in the repo (Key Vault, managed identity, `.env.example`)
- [ ] GDPR notes: data residency (EU regions), PII handling, retention, logging of prompts
- [ ] Cost per request or per run stated
- [ ] "What I would change at 100x scale" section
- [ ] "Known limitations" section (honest limits build trust)

## 12. Interview prep per project

Prepare a 2-minute answer for each:
1. What business problem did it solve, and for whom?
2. What would you change at 100x scale?
3. How would you know it was degrading in production?
4. What does it cost to run per month?
5. What failed, and what did you learn?
6. When would you not use this approach?