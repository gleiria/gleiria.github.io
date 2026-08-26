+++
title = 'Pydantic 2'
date = 2026-08-22T07:15:06+01:00
draft = true
+++



Assumption: customer acquisition accelerates over the six-month period, with the signup rate reaching approximately 4× the January rate by the end of June.

Wrote customer.csv (10000 rows)
Wrote subscription.csv (10000 rows)
Wrote daily_usage.csv (699051 rows)
Wrote product_incident.csv (1 rows)

A Django app where an LLM (Google Gemini, via pydantic-ai) answers natural-language questions about a synthetic SaaS business's data. The core design idea: the LLM never touches the database or generates SQL — it can only call a fixed set of deterministic Python functions ("tools") that query via the Django ORM. This keeps every number in an answer traceable back to real code, not a guess.


LLMs should reason over data, not perform deterministic computation that can be delegated to the database or application code.


Components and how they talk to each other

CSV files (ai_agent_data/)
        │  python manage.py load_data --data-dir ...
        ▼
┌─────────────────────┐
│  SQLite (db.sqlite3) │  ← Django ORM models: Customer, Subscription, DailyUsage, ProductIncident
└─────────────────────┘
        ▲
        │  ORM queries (Count/Sum aggregations)
        │
┌───────────────────┐        instructions + tool         ┌──────────────────┐
│   agent.py         │ ───── registry (pydantic-ai) ────► │  Gemini (LLM)     │
│  Agent + 5 @tools   │ ◄──── tool calls / results ─────── │  google:gemini-*  │
└───────────────────┘                                     └──────────────────┘
        ▲
        │ agent.run_sync(question)
        │
┌───────────────────┐
│  views.py           │
│  ask_agent(request) │──► visualisations.get_chart(tool_name, tool_output)
└───────────────────┘         │
        │                     ▼ builds plotly.graph_objects.Figure, returns JSON-safe dict
        │ JsonResponse({answer, chart})
        ▼
┌───────────────────┐
│ dashboard.html      │  fetch('/ask/') → renders chat bubble + Plotly.newPlot(figure)
│ (Bootstrap + JS)    │
└───────────────────┘
1. Data layer — models.py + load_data
Four models, intentionally simple relationships:

Customer — segment, signup date (primary key: customer_id)
Subscription — one-to-one with Customer (plan, price, status)
DailyUsage — many-to-one with Customer, one row per customer per day, unique-constrained on (customer, date)
ProductIncident — standalone, date + severity
load_data (a Django management command) reads CSVs from a directory you point it at and bulk_creates them with update_conflicts=True, so re-running it is idempotent — it upserts rather than duplicating.

2. Agent layer — agent.py
Defines a single pydantic-ai Agent with:

Instructions: told to act as a data analyst and only state numbers a tool returned — never estimate.
Tools (@agent.tool_plain): get_customer_count, get_customer_distribution_by_segment, get_revenue_by_segment, get_daily_usage_totals, get_incidents. Each is a plain Python function doing a Django ORM aggregation (Count, Sum, .values().annotate()), returning plain dicts/lists/ints.
pydantic-ai handles the protocol with Gemini: it sends the question + tool schemas, Gemini decides which tool(s) to call, pydantic-ai executes them against your actual database, feeds results back to Gemini, and returns a final natural-language answer plus the full message history (including each tool call and its return value).

3. Visualization layer — visualisations.py
A lookup table (CHART_TOOLS) mapping tool name → chart metadata (bar or line, title, target HTML element id). get_chart(tool_name, data) checks whether the tool that just ran is chartable, and if so builds a plotly.graph_objects.Figure — assigning Plotly's default qualitative color per bar (or a single color for lines) — then serializes it to a plain JSON-safe dict via fig.to_plotly_json(). This is deliberately decoupled from the agent: it doesn't know anything about the LLM, it just knows "if this tool ran, here's how to plot its output."

4. Web layer — views.py + urls.py
GET / → dashboard: renders the static page shell.
POST /ask/ → ask_agent: parses the question from the request body, calls agent.run_sync(question), then walks result.all_messages() looking for ToolReturnParts — i.e., it inspects which tools the agent actually called during this conversation and asks visualisations.get_chart() if the last one is chartable. Returns {"answer": ..., "chart": ...} as JSON.
This is the one place where the agent layer and visualization layer meet — neither knows about the other directly; views.py wires them together.

5. Frontend — dashboard.html
Server-rendered Bootstrap 5 page. A <form> submits via fetch() (JSON + CSRF token) to /ask/; JS appends chat bubbles for the question/answer, and if a chart came back, calls Plotly.newPlot(chartDiv, chart.figure.data, chart.figure.layout, ...) — Plotly.js (loaded from a CDN <script> tag) does the actual drawing. All chart-shape decisions were made server-side; the JS is just a dumb renderer now.

Design principles at play
Determinism over generation: the LLM can only call fixed tools, never write raw SQL — every figure in an answer is traceable to a specific ORM query.
Thin, single-purpose files: agent.py only knows tools/LLM, visualisations.py only knows chart-building, views.py only wires HTTP ↔ agent ↔ chart. No shared abstractions beyond what's needed.
SQLite for now, Postgres planned: settings.py currently points straight at db.sqlite3 — simplest possible setup for local development; the README notes Postgres is intended for eventual deployment, not built yet.
No build step for the frontend: Bootstrap + Plotly via CDN <script> tags, vanilla JS — matches the "no unnecessary infrastructure" instruction in this project's CLAUDE.md.
Testing
agent_app/tests/test_agent.py and test_load_data.py — pytest-django, testing the ORM-backed tool functions and the load command's idempotency directly (no LLM calls in tests; GOOGLE_API_KEY is a dummy value just to satisfy the Agent's construction-time check). CI runs this on every branch push via GitHub Actions.

