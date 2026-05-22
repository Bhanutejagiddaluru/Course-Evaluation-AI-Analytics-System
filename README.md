# Course-Evaluation-AI-Analytics-System
ERP / SIS / LMS / Survey Systems → Integration &amp; Search Layer → Mage AI Orchestration → Raw Ingestion → Bronze/Silver/Gold Pipeline → FastAPI + AI Access Layer → Streamlit Dashboard


This project converts structured student feedback data from CSV/Excel format into a privacy-safe, validated, analytics-ready system for course evaluation insights.

This submitted version contains a reduced codebase. Non-essential experimental files were removed. The important implementation files, SQL views, API, dashboard, and AI agent are included. Additional setup and architecture notes are available in the `docs/`, `infra/`, `database/`, and `pipelines/` folders.

---

## 1. Project Goal

The goal of this project is to take a large flat student-feedback file and turn it into trusted academic insights.

The system supports:

- Course performance analysis
- Instructor alias performance analysis
- Department-level summaries
- Course alert/risk detection
- Text-to-SQL question answering
- Streamlit dashboard access
- FastAPI backend access
- SQL views for validated analytics

The system does not let AI reason from raw CSV files directly. The data is cleaned, privacy-protected, validated, and exposed through controlled SQL views and API endpoints.

---

## 2. Main Architecture

```text
Raw CSV / Excel
    ↓
Bronze Cleaned Data
    ↓
SQL Analytics Views
    ↓
FastAPI Backend
    ↓
Streamlit Dashboard / Ask AI
```

Current MVP uses:

- PostgreSQL for database storage
- SQL views for analytics
- FastAPI for backend access
- Streamlit for dashboard
- Gemini/Text-to-SQL agent for natural language querying

---

## 3. Important Project Folders

```text
BSA_Student_Feedback/
│
├── api/
│   └── main.py
│
├── dashboard/
│   └── streamlit_app.py
│
├── src/course_eval_ai/agents/
│   └── text_to_sql_agent.py
│
├── database/
│   ├── views/
│   ├── schemas/
│   └── migrations/
│
├── pipelines/
│   └── scripts/
│       ├── run_sql_views.py
│       └── test_sql_views.py
│
├── data/
│   └── bronze/
│       └── cleaned/
│
├── docs/
├── infra/
├── requirements.txt
├── pyproject.toml
└── README.md
```

---

## 4. Important Files

### FastAPI backend

```text
api/main.py
```

This file runs the backend API and exposes course evaluation endpoints.

### Streamlit dashboard

```text
dashboard/streamlit_app.py
```

This file runs the dashboard UI.

### Text-to-SQL agent

```text
src/course_eval_ai/agents/text_to_sql_agent.py
```

This file contains the AI agent logic for converting natural language questions into safe SQL queries.

Before running the Ask AI feature, replace the API key placeholder with your own key or configure it through environment variables.

Do not expose production API keys in public repositories.

---

## 5. Requirements

Install dependencies from the project root:

```bash
pip install -r requirements.txt
```

Install the project in editable mode:

```bash
pip install -e .
```

Expected successful install:

```text
Successfully installed course_eval_ai-0.0.0
```

---

## 6. Database Setup

This project expects PostgreSQL to be running locally.

Default database used:

```text
course_eval_ai
```

Default connection used in the project:

```text
postgresql://postgres:admin@localhost:5432/course_eval_ai
```

If your PostgreSQL username, password, host, port, or database name is different, update the database URL inside:

```text
api/main.py
dashboard/streamlit_app.py
src/course_eval_ai/agents/text_to_sql_agent.py
```

---

## 7. Create SQL Views

If `psql` is not recognized on Windows, use the provided Python script instead of the `psql` command.

From project root, run:

```bash
python pipelines/scripts/run_sql_views.py
```

Expected output:

```text
Running: vw_bronze_row_scores.sql
Running: vw_course_performance.sql
Running: vw_instructor_performance.sql
Running: vw_department_performance.sql
SKIPPED: vw_college_performance.sql is empty
Running: vw_course_alerts.sql
All SQL views created successfully.
```

Then test the views:

```bash
python pipelines/scripts/test_sql_views.py
```

---

## 8. Run FastAPI

From the project root:

```bash
python -m uvicorn api.main:app --reload
```

FastAPI URLs:

```text
Root:              http://127.0.0.1:8000
Docs:              http://127.0.0.1:8000/docs
Health:            http://127.0.0.1:8000/api/v1/health
Courses:           http://127.0.0.1:8000/api/v1/courses/performance
Instructors:       http://127.0.0.1:8000/api/v1/instructors/performance
Instructors test:  http://127.0.0.1:8000/api/v1/instructors/performance?min_response_count=1
```

---

## 9. Run Streamlit Dashboard

From the project root:

```bash
python -m streamlit run dashboard/streamlit_app.py
```

Streamlit URL:

```text
http://localhost:8501
```

---

## 10. Test Queries in pgAdmin

Run these SQL queries in pgAdmin after creating the views.

### Row-level scoring view

```sql
SELECT *
FROM vw_bronze_row_scores
LIMIT 10;
```

### Course performance

```sql
SELECT *
FROM vw_course_performance
ORDER BY avg_course_rating ASC
LIMIT 20;
```

### Instructor performance

```sql
SELECT *
FROM vw_instructor_performance
ORDER BY avg_instructor_rating ASC
LIMIT 20;
```

### Department performance

```sql
SELECT *
FROM vw_department_performance
ORDER BY avg_course_rating ASC;
```

### Course alerts

```sql
SELECT *
FROM vw_course_alerts;
```

---

## 11. Available API Endpoints

### Health check

```text
GET /api/v1/health
```

Returns API and database status.

### Course performance

```text
GET /api/v1/courses/performance
```

Returns course-level performance metrics.

### Instructor performance

```text
GET /api/v1/instructors/performance
```

Returns instructor alias performance metrics.

Optional query:

```text
GET /api/v1/instructors/performance?min_response_count=1
```

### Department summary

```text
GET /api/v1/departments/summary
```

Returns department-level course and instructor summaries.

### Course alerts

```text
GET /api/v1/alerts/courses
```

Returns course records flagged for review or low-confidence conditions.

---

## 12. Dashboard Features

The Streamlit dashboard includes:

- Overview metrics
- Course performance charts
- Instructor alias performance views
- Department summaries
- Course/instructor rating gap analysis
- Risk and alert tables
- Ask AI / Text-to-SQL question interface

The dashboard is a proof-of-access interface for the analytics layer.

---

## 13. Ask AI / Text-to-SQL

The Ask AI feature uses:

```text
src/course_eval_ai/agents/text_to_sql_agent.py
```

The agent is restricted to safe SQL behavior:

- SELECT-only queries
- No INSERT, UPDATE, DELETE, DROP, ALTER, or TRUNCATE
- Restricted tables are blocked
- Only approved course evaluation tables/views should be queried
- Generated SQL is shown before the result

Example questions:

```text
Which courses have the lowest course ratings?
Which instructor aliases have the lowest instructor ratings?
Which courses have the biggest gap between instructor rating and course rating?
Show average course rating by department.
Show average professor rating by college.
Which course has the most evaluations?
```

---

## 14. Current MVP Notes

This submission keeps the important working pieces only.

Included:

- Cleaned Bronze table flow
- SQL analytics views
- FastAPI backend
- Streamlit dashboard
- Text-to-SQL agent
- Database test scripts
- Project structure and architecture notes

Removed or reduced:

- Non-essential experiments
- Extra prototype files
- Unused exploratory code

The README and supporting docs describe the intended full architecture.

---

## 15. Troubleshooting

### `streamlit` is not recognized

Use:

```bash
python -m streamlit run dashboard/streamlit_app.py
```

instead of:

```bash
streamlit run dashboard/streamlit_app.py
```

### `uvicorn` is not recognized

Use:

```bash
python -m uvicorn api.main:app --reload
```

instead of:

```bash
uvicorn api.main:app --reload
```

### `psql` is not recognized

Use:

```bash
python pipelines/scripts/run_sql_views.py
```

instead of running SQL files through `psql`.

### Database connection error

Check that PostgreSQL is running and that this database exists:

```text
course_eval_ai
```

Also verify the database URL in:

```text
api/main.py
dashboard/streamlit_app.py
src/course_eval_ai/agents/text_to_sql_agent.py
```

---

## 16. Quick Start

Run these commands from the project root:

```bash
pip install -r requirements.txt
pip install -e .
python pipelines/scripts/run_sql_views.py
python pipelines/scripts/test_sql_views.py
python -m uvicorn api.main:app --reload
python -m streamlit run dashboard/streamlit_app.py
```

Open:

```text
FastAPI docs:   http://127.0.0.1:8000/docs
FastAPI health: http://127.0.0.1:8000/api/v1/health
Streamlit app:  http://localhost:8501
```

---

## 17. Final Note

This project demonstrates a working MVP path from structured student feedback data to privacy-safe analytics and AI-assisted querying.

The main code paths are:

```text
database/views/
api/main.py
dashboard/streamlit_app.py
src/course_eval_ai/agents/text_to_sql_agent.py
pipelines/scripts/run_sql_views.py
pipelines/scripts/test_sql_views.py
```
