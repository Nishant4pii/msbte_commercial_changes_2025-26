1. Module Name
MSBTE Admin Dashboard — module-wise admin for operators (developer/COE-style configuration) and client roles (RBTE / institute-facing workflows)
Concrete “module-wise” surfaces include the Module Control Panel (per-module link visibility, e.g. E-Marksheet, Exam Form summer/winter, COE) and JSON-backed update_module_config under dashboard URLs.

2. Commercial / Functional Changes
Exam-cycle operations (Summer 2026 / S26): Repeated updates to msbte_config.py for exam numbers, timetable/marksheet/emarksheet/RAC/confirmation table names, staging DB names (e.g. mdb_s26), and alignment with non-AICTE vs AICTE naming.
New or expanded functional areas (from commits): Seating chart (institute/RBTE flows, NIL, corrections, confirmations removal), ABC ID reports and statistics, photo correction and bulk/single photo download, E-marksheet RBTE reports, EC–DC–CI module and distribution reports, affiliation admin (new institute, remove affiliation, history/detail, course history), enrollment (candidate search, date-wise confirmations), DTE data preparation, hall ticket prep (later commits also remove/adjust hall ticket URLs per log), bulk email and template updates (Summer 2026), bulk candidate registration (runtime table names, logging), RTS data push, circular management (EC/DC/SC options, active toggle), RBTE reporting (fees, COB, inventory, detention, malpractices, remuneration revisions), online exam center report, RAC capacity reporting, institute details coursewise, 2FA for admin group.
Client vs internal: RBTE/institute reports, exports (Excel), print layouts; internal data preparation, validation reports (exam_validations in INSTALLED_APPS), Celery task adjustments, and CI/deploy workflow changes.
Explicit “client/compliance/ops” requests: Not documented in git beyond commit intent (e.g. “Summer 2026”, “non-AICTE”, email template updates); treat as exam-board operational cycles, not third-party contracts in-repo.
3. Positive Impact
Measured in repo: ~752 commits from 2026-01-03 through 2026-05-02 on the analyzed branch (velocity / scope indicator only).
Session policy (quantified in commits): Auto-logout inactivity 15 → 30 → 50 minutes (documented in commit subjects).
Operational: Centralized msbte_config-driven table names reduce hard-coded season drift errors when DB objects rename per exam.
UX (qualitative from commits): Print/export/navigation improvements across reports; superuser SQL visibility on some reports for support.
No stored figures for API latency, query ms, error rate, or manual hours saved.
4. Technical Changes / Upgrades
Previous → New: Not a full framework migration in the log; evolution is incremental on Django.
Stack (from requirements.txt): Django ≥ 3.2.25, Celery 5.3.4, Redis ≥ 5.0.1, django-celery-beat/results 2.5.x, mysql-connector-python ≥ 9, PyMySQL 1.1.0, DRF ≥ 3.14, PyJWT ≥ 2.8, pandas ≥ 2.1, SQLAlchemy ≥ 2, openpyxl, Gunicorn, WhiteNoise, django-otp + qrcode (TOTP 2FA), LangChain / LangGraph / ChromaDB / sentence-transformers (chatbot/RAG path), GitHub Actions + appleboy/ssh-action (version pins in commits).
Database: Heavy use of MySQL via connector/PyMySQL; commits describe query refactors, dynamic table names, strict SQL mode toggling during bulk affiliation enrollment prep, and migrations for exam form data prep tasks.
API / integration: Node API for exam form configuration and affiliation views; JWT-protected report views; cache-control headers and exam form API handling (commits).
Code: New Django apps under dashboard/ and data_prepration/, decorators (jwt_report_authorized), modular URLs/templates per subdomain (examform, seating_chart, enrollment, etc.).
5. Architecture Improvements
Still a Django monolith with many domain apps (clearer modularization by exam/affiliation/seating/enrollment packages rather than microservices).
Scalability: Celery + Redis for async work; pandas for report/data prep paths.
Maintainability: msbte_config constants for DB object names; JSON module configs for control-panel-driven feature flags; repeated refactor commits on seating chart and fees/SQL.
6. Security Enhancements
AuthZ: @jwt_report_authorized(allowed_roles=[...]) on large parts of RBTE/eligibility/report code (replacing blanket @login_required in multiple commits).
AuthN / session: TOTP 2FA (django-otp stack in requirements); JWT decorator enhancements.
HTTP hardening (commits): Origin / Content-Type headers on password-related views; cache-control on exam form API paths.
Vulnerability fixes: No CVE IDs in commit messages; fixes are targeted (e.g. SQL LIKE %% correction in abc_id views).
7. Challenges Faced
Seasonal schema churn (S25 → S26, timetable/marks/confirmation/RAC table renames).
MySQL strict mode vs bulk inserts in affiliation enrollment preparation.
Data edge cases: scheme change COB loop risk (commits mention guarding skipped enrollments); deleted subjects / NIL / correction flows in seating chart.
Deploy/ops: Docker restart permissions (sudo removal), runner name changes (msbte-admin vs dashboard), syncing main/dev/live branches.
8. Solutions Implemented
Central config (msbte_config.py) + view refactors to read constants instead of literals.
Temporary strict mode disable around controlled inserts (per commit).
Task logic updates to avoid infinite processing on scheme-change COB.
Explicit SQL escaping and query simplification where bugs were found.
Workflow pinning for SSH deploy action versions and clearer deploy scripts.
9. Before vs After (Short Summary)
Before (start of git history, 2026-01-03): Initial import + baseline auto-deploy; minimal product surface in history.
After (current branch state to 2026-05-02): Broad MSBTE admin coverage—module control panel, many RBTE/institute reports, seating chart suite, ABC ID, photo correction, E-marksheet, affiliation admin, bulk mail/registration, 2FA, JWT-scoped reports, and Summer 2026 configuration alignment.
10. Metrics (if available)
API response time: Not recorded in the repository (no benchmark artifacts in git messages reviewed).
Query execution time improved by ___%: Not available from commits alone.
Manual effort reduced by ___%: Not available.
Error rate reduced by ___%: Not available.
Proxy “metrics” from git: 0 commits dated 2025; 752 commits with --since=2025-01-01 (all 2026); session timeout changes 15 → 30 → 50 minutes per commit history.