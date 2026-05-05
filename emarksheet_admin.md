Module Name: emarksheet admin in python

Commercial / Functional Changes:

Academic/session rollover to S26: Configuration and task logic updated for Summer 2026 / ACA_YEAR “2026 - 27” and EXAM_YEAR_VENDER 202601, with table constants moved/added for s26_* datasets (NTH + TH).
Admin workflow improvements:
Role-based landing: admin users redirected to a new Admin Dashboard; non-admin to vendor report list.
Alert dashboard added/streamlined for operational monitoring (counts + “run core reports” JSON payload).
PDF verification workflow (NTH): shifted to a CSV extraction + load-only pipeline (no comparison stage inside the task), including server1/server2 path modes, improved filename parsing (supports old + new session naming), and operational controls for large runs.
Marks string conversion workflow: enhanced to generate/import marks-string data into MySQL with post-import verification (row counts, precision checks) and mismatch reporting.
Immudb audit workflow: enhanced robustness and added region-wise mode support (final-lock + multiple region tables) with better pagination and typing behavior.
Reporting changes:
Multiple SQL query updates to align with new session/table naming for PDF verification data and lock tables.
Report templates updated for clarity (e.g., “Sheet No” display, improved empty-state layouts, status badges).
Positive Impact:
PDF verification scalability controls (explicitly tuned for 50K–60K PDFs):
Added windowing + cooldown + micro-pauses and capped concurrency (default PDF_MAX_WORKERS=3, PDF_WINDOW_SIZE=20000, cooldown 20s) to reduce sustained CPU/DB pressure during large extraction runs.
Reduced DB write churn by updating progress less frequently (e.g., every 50 files).
Dashboard latency reduction (wall-clock):
KPI COUNT queries run in parallel via ThreadPoolExecutor (4 workers), reducing wall-clock time from approximately sum(query times) to approximately max(query time) for the 4 KPI counters.
Audit performance on large tables:
Immudb audit moved to keyset pagination (avoids OFFSET degradation), improving throughput predictability on large datasets.
Stability improvements:
Added “safe save” pattern in PDF tasks (retries + connection refresh) to reduce task failures from transient DB connectivity issues.
Prevented immediate post-login idle logout by clearing stale inactivity timestamp.
Technical Changes / Upgrades:
Previous technology (if replaced):
Offset-based pagination patterns and heavier sequential dashboard querying (replaced by keyset pagination + parallelism in key areas).
Some direct/raw connection patterns replaced with more consistent connection management (SQLAlchemy engine usage + lazy DB connection wrapper in views).
New technology used:
SQLAlchemy 2.0.35 used broadly for query execution / table DDL + DML in tasks.
ThreadPoolExecutor used for dashboard KPI parallelization and PDF extraction concurrency.
Framework / library upgrades (as reflected in current deps):
Core stack includes Django, Celery 5.3.4, Redis, Channels 4, PyMuPDF, pandas 2+, mysql-connector-python 9+, PyMySQL, djangorestframework, whitenoise, etc.
Database changes:
S26 schema alignment: extensive table/database constant updates (e.g., EMS_DB, EMSLOCK_DB, EMS_DB_TH, s26_* tables).
Read-only/select DB config added: a dedicated msbte_select DB connection for controlled access.
Query refactors in reporting views (e.g., region-wise institute stats) to reduce heavy scans and simplify transformations.
API changes:
Added/maintained JSON endpoints for task status retrieval and combined dashboard report payloads (e.g., recent tasks/status endpoints, “run dashboard reports” returning structured JSON).
Code improvements:
Consolidation/movement of duplicated functions (e.g., marks conversion/PDF verification logic centralized under data_activity_link).
More robust parsing/validation logic (e.g., w1/w2 detection from folder path segments; filename/session parsing).
Architecture Improvements:
Modularization: functionality increasingly split into app modules (notably emarksheet/data_activity_link/*) instead of keeping all workflows in the root emarksheet app.
Scalability improvements:
Concurrency controls (PDF extraction) + parallel query execution (dashboard) + keyset pagination (audit) enable safer operation on large datasets.
Maintainability improvements:
Centralized S26 constants in msbte_config.py reduced hardcoded table-name drift across tasks/views/templates.
Security Enhancements:
CSRF hardening: CSRF_TRUSTED_ORIGINS explicitly populated (e.g., cloudreportems.msbte.co.in and known admin portals).
JWT secret handling improvement: Node attachment service jwt_secret sourced from environment variables (instead of being fixed in code).
Session/timeout alignment:
Server-side SESSION_COOKIE_AGE aligned to inactivity timeout (35 minutes) to reduce inconsistent session expiry behavior.
Note (factual risk present in current code): some credentials/keys are still defined in settings/config files rather than exclusively via environment variables.
Challenges Faced:
High-volume PDF processing (tens of thousands of PDFs) causing CPU spikes, long runtimes, and increased DB write pressure.
DB connection instability during long-running Celery tasks (saving progress/results reliably).
Annual session/schema churn (S25 → S26) requiring consistent updates across configs, SQL queries, tasks, and templates.
Large-table auditing where OFFSET pagination becomes slow/unreliable at scale.
Solutions Implemented:
Throttled, windowed PDF extraction with explicit limits:
Worker cap, window sizing, cooldown pauses, chunked CSV insert (10,000-row chunks), and reduced progress-write frequency.
Resilient task persistence:
Retry-based ProcessLog save with connection refresh strategies in long tasks.
Schema abstraction via config constants:
Centralized s26_* table/database names to reduce scattered SQL edits.
Keyset pagination + type detection in immudb audits:
Numeric-vs-string literal handling and keyset conditions to keep pagination fast and correct.
Before vs After (Short Summary):
Before: S25-centric configs/queries, more sequential dashboard querying, less controlled large-batch PDF extraction, and more duplication across apps.
After: S26-aligned configs and SQL, parallel KPI querying, windowed/throttled PDF extraction tuned for 50K–60K PDFs, keyset-paginated immudb audits, and clearer admin UX via role-based landing + dashboards.
Metrics (if available):
PDF extraction throughput controls: supports 50K–60K PDFs with defaults PDF_MAX_WORKERS=3, PDF_WINDOW_SIZE=20000, PDF_BATCH_COOLDOWN_SECONDS=20, chunked DB insert size 10,000.
Dashboard KPI counts: wall-clock reduced from ~T1+T2+T3+T4 to ~max(T1..T4) by running 4 COUNT queries in parallel.
Audit pagination: switched to keyset pagination (removes OFFSET growth cost); exact query-time deltas not stored in-repo.
API response times / query timings / error rate: not tracked in codebase (no historical baselines committed).