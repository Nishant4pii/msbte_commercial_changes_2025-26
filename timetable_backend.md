Module Name: time table module

Commercial / Functional Changes:

Timetable result ordering rules updated across APIs to match a new master-code priority sequence: 'K','F','I','J','G','E','C','S','B','Y','A' (replaced older sequences like 'G','E','C','D','B','A','S','R','M','N','O','T' / 'K','I','G','E',...).
Exam-center-wise timetable response grouping refined:
1 subject_name per paper_code/day/session is now selected deterministically using SUBSTRING_INDEX(GROUP_CONCAT(DISTINCT subject_name ...), 1) with a stable separator.
scheme list now uses GROUP_CONCAT(DISTINCT Course_Year_Master_Code ...) with internal ordering by scheme priority.
Ordering for exam-center timetable explicitly aligned to business presentation:
ORDER BY exam_dayw ASC, daysession DESC, <scheme-priority>, paper_code ASC.
Environment naming convention change from W25 → S26 for timetable/RAC/EMS_TH schema/table references (ops-driven seasonal dataset switch).
Positive Impact:
Deterministic API output ordering for daywise/institutewise/papercodewise/examcenterwise endpoints, reducing UI-side sorting and “same request → different order” variance to 0 (ordering is now fully specified in SQL).
Exam-center aggregation correctness improved:
Eliminated ambiguous GROUP_CONCAT comma-splitting by using a custom separator ('|||') and DISTINCT, reducing subject/scheme duplication in grouped rows (qualitatively: from “multi-value concatenation artifacts possible” → “single chosen subject_name + distinct scheme list”).
Deployment reliability improved by pinning GitHub Action version (see #4), reducing supply-chain drift risk and unpredictable changes from a moving @master.
Technical Changes / Upgrades:
Previous technology (if replaced):
GitHub Actions used appleboy/ssh-action@master (floating).
EMS_TH table reference used w25_th_input and env DB names referenced W25 datasets.
New technology used:
GitHub Actions pinned to appleboy/ssh-action@v1.0.3 in both auto_deploy_dev.yaml and auto_deploy_live.yaml.
Framework / library upgrades:
package-lock.json updated (notably dependency patch/minor updates such as body-parser 1.20.3 → 1.20.4, qs 6.13.0 → 6.14.2, raw-body 2.5.2 → 2.5.3, minimatch 3.1.2 → 3.1.5, object-inspect 1.13.1 → 1.13.4, and related dependency graph changes).
Database changes:
Dataset/schema targeting updated to S26:
TIMETABLE_ALL_COURSES: w25_timetable_subjects_course_all → s26_timetable_subjects_course_all
PAPER_CODES: paper_code_w25 → paper_code_s26
RAC_DC_DETAILS: w25_rac_dc_details → s26_rac_dc_details
EMS_TH.TH_INPUT: w25_th_input → s26_th_input
SQL query-level changes focused on GROUP BY, ORDER BY, and stable priority ordering using MySQL FIELD(...) and SUBSTR(...).
API changes:
No new endpoints observed in 2026 commits; changes are behavioral (sorting, grouping, data selection) within existing controller queries:
controllers/coursewiseController.js
controllers/daywiseController.js
controllers/institutewiseController.js
controllers/papercodewiseController.js
controllers/examcenterwiseController.js
Code improvements:
database/database.js enhanced for operational stability/observability:
Pool lifecycle management (singleton pool + graceful shutdown hooks)
Health check utility (SELECT 1 as health_check)
Query retry wrapper with exponential backoff (safeQuery, default 3 attempts, waits (2^attempt * 1000ms) between retries)
Extra logging of SQL and params (increases traceability; may increase log volume).
Architecture Improvements:
Maintainability: centralized seasonal table naming via database/table_name.js (moving W25 → S26 via config) reduced hardcoded table drift across controllers.
Operational resiliency: DB access layer moved toward a more managed pool lifecycle with shutdown handling + health check, improving service behavior during deploy/restart cycles.
No monolith/microservice split detected; architecture remains single Node.js backend.
Security Enhancements:
Supply-chain hardening: Actions dependency pinned from a moving branch (@master) to a fixed release tag (@v1.0.3).
Dependency patch upgrades in package-lock.json (e.g., body-parser, qs, raw-body) reduce exposure to known issues addressed in patch releases.
No explicit auth/authorization flow changes detected in 2026 diffs; primary security-related work was dependency + CI pinning.
Challenges Faced:
Inconsistent/undesired ordering of timetable rows across multiple “wise” endpoints, causing mismatched presentation between API consumers.
Ambiguous aggregation for exam-center-wise grouping where concatenated fields could yield non-deterministic or hard-to-parse results.
Seasonal dataset rollover (W25 → S26) requiring coordinated updates across env files and table references without breaking existing queries.
CI/CD drift risk due to using a floating GitHub Action reference.
Solutions Implemented:
Standardized ordering using MySQL:
ORDER BY ... FIELD(master_code, ...) / FIELD(SUBSTR(...), ...) applied consistently.
Added secondary/tertiary sorts (paper_code ASC, exam_dayw ASC, daysession DESC) to fully specify output order.
Fixed aggregation determinism in exam-center query:
GROUP_CONCAT(DISTINCT ...) + stable separator + SUBSTRING_INDEX(..., 1) to choose a single prioritized subject_name.
GROUP_CONCAT(DISTINCT Course_Year_Master_Code ORDER BY <priority>) for stable scheme output.
Rolled dataset references forward by updating:
.env.dev, .env.development, .env.production
database/table_name.js
Pinned GitHub Actions dependency to a version tag.
Before vs After (Short Summary):
Before:
Timetable APIs could return rows in inconsistent priority order (older FIELD(master_code, ...) lists varied by endpoint).
Exam-center-wise grouping used simpler GROUP_CONCAT patterns that could produce ambiguous concatenated values.
Env/table references targeted W25 datasets in multiple places; deployment used a floating action reference.
After:
All key timetable “wise” endpoints enforce a consistent master-code priority and stable multi-column ordering.
Exam-center-wise endpoint returns deterministic grouped rows with a single prioritized subject_name and distinct, ordered scheme list.
Dataset references updated to S26; GitHub Actions deployment uses pinned appleboy/ssh-action@v1.0.3.
Metrics (if available):
API response time reduced from ___ to ___: Not measured in repo changes (no benchmarks/telemetry committed).
Query execution time improved by ___%: Not measured; changes were primarily ordering/grouping logic (could increase or decrease depending on indexes/data).
Manual effort reduced by ___%: Not directly measured; deployment drift risk reduced by pinning action version (qualitative).
Error rate reduced by ___%: Not measured; deterministic ordering and more robust DB pool handling likely reduces intermittent “ordering/aggregation anomalies,” but no logged metric in codebase.