# ERPilot AI — Product & Architecture Showcase 🤖📊

**English** | [Türkçe](README.tr.md)

**A natural language financial assistant and anomaly detection platform for enterprise data. Built for the BTK Academy Hackathon 2026 (Finance Category).**

Ask questions in plain language ("Which department spent the most in January?"); the system queries verified financial data, generates interactive charts, provides concise AI commentary, and highlights statistical outliers.

> ℹ️ **About This Showcase:**  
> This repository is a **public product showcase and technical portfolio**. The full application source code (FastAPI backend, query engine, and prompt guardrail layers) is kept in a private repository. This repository contains the system architecture, sequence diagrams, and UI screenshots.

---

## 📸 Screenshots & UI Walkthrough

| Overview Dashboard (Revenue, Expenses, Profit & Natural Language Chat) | Budget vs Actual Spend |
|---|---|
| ![Overview dashboard](docs/img/genel-en.png) | ![Budget vs actual](docs/img/butce-en.png) |

| Outlier / Anomaly Detection (IQR Method) | Ask the Data (Function-Calling NL Querying) |
|---|---|
| ![Anomalies](docs/img/anomali-en.png) | ![Querying data](docs/img/sor-en.png) |

| Dark Theme | Turkish Localization |
|---|---|
| ![Dark theme](docs/img/genel-dark.png) | ![Turkish dashboard](docs/img/genel-tr.png) |

---

## 🎯 Key Engineering Highlights

- **Ask the Data (Natural Language Querying):** Google Gemini converts unstructured queries into a structured `QuerySpec`. Numbers are calculated from underlying data, preventing hallucination.
- **Secure Architecture (Zero SQL/Code Injection):** The model never writes raw SQL or executable code. It only populates strict Pydantic `Literal` allowlists. Irrelevant or adversarial prompts are rejected at the architectural layer.
- **Statistical Anomaly Detection:** Outlier expenses exceeding normal thresholds are flagged automatically using the Interquartile Range (IQR) method.
- **AI Commentary & Persistent Caching:** Generates concise explanations for each chart. When an API key is absent, an offline deterministic engine generates insights from the same calculations.
- **Bilingual & Dual Theme:** Turkish and English language support with seamless dark/light theme switching.

---

## 🏗️ System Architecture

```mermaid
flowchart TB
    subgraph FE[Frontend - Single Page App]
        UI["6 Dashboard Views (Chart.js)<br/>Dark/Light Theme + TR/EN Support"]
    end
    subgraph BE[FastAPI Backend Layer]
        API["REST API Endpoints<br/>/api/chat, /api/dashboard, /api/anomalies"]
        RL["Rate Limiter & TTL Cache"]
        GC["Gemini Client (Function Calling)"]
        QE["Query Engine (Allowlist Validation)"]
        AN["Analytics & IQR Anomaly Engine"]
        DS["DataSource Abstraction Layer"]
    end
    subgraph DATA[Data Sources]
        CSV[("Normalized Demo Data")]
        SAP[("SAP S/4HANA (OData REST / BAPI)")]
    end

    UI -->|HTTPS REST| API
    API --> RL --> GC
    API --> AN
    GC --> QE
    QE --> DS
    AN --> DS
    DS --> CSV
    DS -.Future Expansion.-> SAP
```

### "Ask the Data" Execution Flow (Function Calling + Guardrail Security)

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as FastAPI
    participant G as Gemini AI
    participant Q as Query Engine
    participant D as Data Layer (pandas)

    U->>F: Natural language financial question
    F->>B: POST /api/chat (message, lang)
    B->>G: User query + query_finance_data schema
    alt Irrelevant / Out of domain prompt
        G-->>B: Function NOT called
        B-->>F: Pre-defined safe rejection message
    else Valid financial query
        G-->>B: query_finance_data(QuerySpec)
        B->>Q: Validate QuerySpec against strict allowlist
        Q->>D: Safe pandas filtering & aggregation
        D-->>Q: Verified numerical results
        Q-->>B: Computation output
        B->>G: Summarize output (raw user input excluded)
        G-->>B: Formatted natural language response
        B-->>F: Verified response & chart update
    end
```

---

## 💻 Tech Stack

| Layer | Technology |
|---|---|
| **Backend** | Python, FastAPI, Pydantic, pandas, NumPy, SlowAPI |
| **Artificial Intelligence** | Google Gemini (Function Calling / Structured Outputs), Offline Rule Engine |
| **Frontend** | Vanilla JavaScript, Chart.js, HTML5, CSS3, Modern Responsive Layout |
| **Data Architecture** | Normalized Schemas, CSV (SAP S/4HANA OData Ready) |
| **Documentation** | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) · [docs/DEMO.md](docs/DEMO.md) · [docs/pitch.md](docs/pitch.md) |

---

## 🔒 Source Code Notice

This repository does not contain application source code, database files, or API credentials. It is published purely as an architectural case study and product showcase.
