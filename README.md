# 🛒 Coles Retail Multi-Agent Data Assistant

An enterprise multi-agent data engineering and analytics platform powered by **LangGraph**, **PostgreSQL**, and **Streamlit**.

The platform automates everyday retail data engineering and business intelligence workflows while enforcing a strict **Human-in-the-Loop (HITL)** approval architecture for database schema evolutions and data modifications.

---

## 📽️ Demo

![Platform Demo](assets/demo.gif)

---

## 🌟 Key Features

- **Multi-Agent Orchestration (LangGraph):**
  - **Supervisor Router:** Interprets natural language requests and accurately routes between analytical SQL queries and file-level ETL workflows.
  - **SQL Analyst Agent:** Inspects PostgreSQL schema dynamically, handles fuzzy matching (`ILIKE`) across products, generates visualizations, and drafts structured mutation proposals.
  - **ETL Python Analyst Agent:** Handles flat-file processing, POS basket reconciliation pipelines, and file ingestion using Pandas.

- **Human-in-the-Loop (HITL) Database Governance:**
  - **Autonomous Read Sandbox:** Automated agent execution is restricted to safe read queries (`SELECT`, `WITH`, `EXPLAIN`).
  - **Structured JSON Proposals:** Any detected schema alteration (`ALTER TABLE`, SCD Type 2 tracking) or record update (`UPDATE`, `INSERT`, `DELETE`) is intercepted as a migration proposal.
  - **Interactive Streamlit Review Card:** Displays target tables, planned impact, and generated SQL, executing changes only after explicit human approval.

- **Automated Visual Analytics:**
  - Exports query results to CSV and generates instant interactive charts and data tables when charting keywords are present in the request.

- **Fully Containerized:**
  - Complete multi-container environment orchestrated with Docker Compose (PostgreSQL warehouse + Streamlit application).

---

## 🏗️ Architecture & Safety Pipeline

```text
                       User Prompt (Streamlit UI)
                                   │
                                   ▼
                       ┌────────────────────────┐
                       │   Supervisor Router    │
                       └───────────┬────────────┘
                                   │
                 ┌─────────────────┴─────────────────┐
                 ▼                                   ▼
      ┌──────────────────────┐           ┌──────────────────────┐
      │  ETL Python Analyst  │           │     SQL Analyst      │
      └──────────┬───────────┘           └──────────┬───────────┘
                 │ (Pandas / Reconcile)             │
                 ▼                                  ▼
         File Ingestion &                    Query Classifier
         Discrepancy Report                         │
                                         ┌──────────┴──────────┐
                                         ▼                     ▼
                                   [SELECT Query]      [DDL / DML Mutation]
                                         │                     │
                                         ▼                     ▼
                                  LLM Judge Check       JSON Proposal Block
                                         │                     │
                                         ▼                     ▼
                                  Read-Only Exec        HITL Review Card
                                  & Auto-Chart           (Approve / Reject)
                                                               │
                                                               ▼
                                                        execute_approved_
                                                            mutation
---
```
## 📁 Repository Structure

```text
├── agents/
│   ├── data_agent.py          # Supervisor routing agent and LangGraph entrypoint
│   ├── sql_analyst.py         # SQL Analyst agent with guardrails and proposal logic
│   └── etl_analyst.py         # Python ETL agent for Pandas file transformations
├── models/
│   └── schema.py              # Pydantic schemas (RetailAgentState, JudgeSchema, RouterSchema)
├── utils/
│   ├── database.py            # Database utility with separated read vs. mutation runners
│   ├── etl_tools.py           # Flat-file reconciliation and processing tools
│   └── llm_pick.py            # Model selector and provider configuration
├── data/                      # Staged datasets, CSV query outputs, and anomalies
├── assets/                    # Project demo GIF and media assets
├── app.py                     # Streamlit web application and HITL interface
├── docker-compose.yml         # Container configuration (PostgreSQL + App)
├── Dockerfile                 # Application container build definition
├── requirements.txt           # Python project dependencies
└── README.md
```
🚀 Getting Started
1. Prerequisites
Docker Desktop installed and running.

An OpenAI or compatible LLM API Key.

2. Environment Setup
Create a .env file in the root directory:

```
OPENAI_API_KEY=your_api_key_here
POSTGRES_DB=coles_retail
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
```
3. Build & Run
Run the application using Docker Compose:
```
docker compose up --build -d
```
Access the Streamlit web dashboard at:
```
http://localhost:8501
```
---
## 💡 Example Prompts

| Intent | Example Prompt | Behavior |
| :--- | :--- | :--- |
| **Autonomous Analytics** | *"Show me the total sales amount grouped by customer loyalty tier as a chart."* | Generates a grouped `SELECT` query, saves CSV, and renders a native bar chart. |
| **Schema Evolution (DDL)** | *"Need to update dim_customers_loyalty by adding effective start date, end date, and current status columns."* | Bypasses execution, generates an `ALTER TABLE` proposal, and prompts for human review. |
| **Data Modification (DML)** | *"Update the loyalty tier to 'Gold' for customer Shopper_1 (flybuys_id 'FB-10001') in dim_customers_loyalty."* | Proposes an `UPDATE` statement with impact summary; commits only on user approval. |
| **ETL File Pipeline** | *"Run the POS basket reconciliation pipeline on the latest CSV files."* | Routes to ETL agent, identifies order discrepancies via Pandas, and outputs an anomaly report. |

---

## 🛡️ Security & Guardrails

- **Strict Read Isolation:** Autonomous query execution enforces an allowlist check permitting only `SELECT`, `WITH`, and `EXPLAIN` statements.
- **Human-Gated Mutations:** Destructive or mutating operations (`ALTER`, `UPDATE`, `INSERT`, `DELETE`) require explicit user authorization through the interactive UI.
- **State Validation:** Typed Pydantic schemas validate agent state transitions across the graph to prevent unexpected execution paths.
