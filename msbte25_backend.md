1. Module Name
MSBTE 25 Node backend (msbte25_backend) — Express API covering admin, institute, RBTE, candidate, and auth (login / session-related) flows (plus integrations such as MahaIT / RTS routes under the same service).

2. Commercial / Functional Changes
Core COB workflows: Admission-level COB (DSD selection, application id/details, submit/cancel/confirm/print-style flows per commit history), institute-level COB (apply + detail submit), and related validation/query orchestration.
Auth & identity: Login, password change/reset patterns, OTP/mobile update flows, generate-default-password (admin-gated route in auth/login), JWT-based session handling (commits reference JWT expiry tuning).
RBTE / EDDCCI – DEC (Diploma Engineering Course): Endpoints for 1st / 2nd / final year DEC list/save/delete, capacity present reads with Joi validation, and RBTE access checks wired into DEC controller methods.
Admin / operations: Circulars logic extensions (RAC malpractice retrieval, access rules for specific circular IDs, authority letter data), QP Stat endpoints, config routes.
Institute operations: Digital evaluation activity endpoints; ABC ID flows (Aadhaar validation/existence checks); online exam center data paths; affiliation status handling (including HTTP status correction for blocked affiliation).
SC / DC distribution reporting: New/updated endpoints and QueryMap SQL for DC center list, formatted dispatch answer book variants (SC lead, SC→SC lead, controller→DC), ordering/sorting fixes, sc_code vs sc_center corrections, empty-list responses for DC/EC.
Integrations: MahaIT URL updates, RTS dashboard API behavior changes, application tracking messaging and date handling; env-driven MAHAIT / DB name consistency updates (DB_MAIN, DB_EMARKSHEET_TH, PURE_NON_AICTE_BLOCK, etc., per commits).
Client / ops signals (only what Git states): Postman collection/resources were added/updated under postman/; GitHub Actions deploy workflow SSH action version bumped — no ticket IDs or compliance docs are present in the analyzed metadata.
3. Positive Impact
Breadth of automation: Large surface area (964 commits) across admission, institute, RBTE DEC, circulars, and hall-ticket/exemption/certificate-style route families reduces reliance on ad-hoc scripts for the same operations (qualitative; no hours-saved field in repo).
Performance (documented in commits, not benchmarked): Explicit optimizations called out in history — e.g. CTE-based SC center query, parallel processing / streaming and pre-computed table path for online exam center retrieval, query deduplication / ordering fixes for dispatch answer book exports — intended to cut DB work and redundant rows.
UX / correctness: Clearer application tracking status text; HTTP 429 with retryAfter on login rate limit; 403 CORS JSON for disallowed origins; empty list JSON responses instead of ambiguous errors for DC/EC center lists.
Stability: JSON parser exclusions for encrypted / non-JSON MahaIT-style payloads reduces body-parser failures; trust proxy + rate-limit validation flags align behavior behind reverse proxies.
Measurable metrics in repo: Not available (no benchmarks/, APM exports, or before/after timings committed).
4. Technical Changes / Upgrades
Previous technology (replaced in-repo): N/A — first Git commit 2025-06-16; this repo is a greenfield Node backend from that point, not a migration from another codebase recorded here.
New / current stack (from package.json): Node.js + Express ^5.1.0, mysql2 ^3.15.3, mongoose ^9.0.0 (Mongo adapter present in dbConfig), axios ^1.13.2, Joi ^18.0.2, jsonwebtoken ^9.0.2, helmet ^8.1.0, express-rate-limit ^8.2.1, cors ^2.8.5, express-fileupload ^1.5.2, dotenv ^17.2.3, cross-env ^10.1.0, module-alias for @controllers / @services / @routes / etc.
Framework / library upgrades (evidence): Dependency churn including Express 5-era stack; lockfile fix commit removing peer dependency flag for Express (2026-05-05).
Database: Primary pattern — centralized SQL in service queryMap layers + MySQL adapter (no runtime @prisma/client usage found under src/; Prisma exists as npm run prisma:generate tooling for src/dbConfig/prisma/main.prisma). Commits mention table renames (RAC / online exam / copy-case tables), AES_DECRYPT for mobile fields, and numerous JOIN / GROUP BY / filter fixes.
API changes: New route families per history — e.g. DEC CRUD/read APIs, QP Stat, digital evaluation activity, dispatch answer book variants, generate-default-password, RTS / MahaIT touchpoints; multiple merge dev → live events indicate promotion of API batches to production branch.
Code improvements: Controllers refactored for readability, Joi validation modules, middleware composition (closeLink on EDDCCI routes per commits), maintenance window middleware, env validation patterns in MahaIT controller commits.
5. Architecture Improvements
Style: Modular monolith — Express app in src/index.js, configuration in src/appConfig, role-based route trees under src/routes/{admin,institute,rbte,candidate,auth,common,mahaIT}, controllers + services + query maps.
Scalability: Rate limiting (global + login-specific), 50mb upload/body limits where needed, conditional JSON parsing to support large/encrypted payloads without breaking the whole server.
Maintainability: Unified query maps (explicitly mentioned in early admission-level refactor commits), repeated QueryMap / CircularManager centralization for SQL-heavy domains.
6. Security Enhancements
Authentication / authorization: JWT usage; RBTE auth on DEC routes; admin auth on sensitive auth utilities; login rate limiter (loginLimiter on login routes) with 429 + retryAfter.
Transport / headers: Helmet frameguard: deny; trust proxy: true for correct client IP behind proxies.
CORS: Origin allowlist from allowDomains; dev/staging relaxed patterns (localhost / dev tunnels); exceptions for specific MahaIT/RTS paths (applicationTracking, pustRTSDashboardData) documented in appConfig.
Data handling: AES_DECRYPT for mobile-related SQL (commits); sanitized filename handling for DEC manuals file download (per importantLinks controller grep).
Vulnerability fixes: No CVE-linked commit messages observed; security work is defense-in-depth (rate limit, CORS, helmet, crypto usage) rather than a disclosed patch list.
7. Challenges Faced
Integration complexity: MahaIT-style routes needing raw / text bodies and many excluded paths from express.json — high risk of subtle parsing bugs.
Data model drift: Repeated commits renaming RAC / copy-case / online exam tables and fixing wrong table references — indicates schema or environment mismatch pressure.
Multi-role authorization: Wiring RBTE access checks into DEC while keeping institute/RBTE/candidate boundaries consistent.
SQL correctness at scale: Dispatch answer book and center-list queries required multiple iterations for joins, ordering, and empty-result behavior.
Branch / release friction: Frequent Merge branch 'main' / dev / live commits suggest parallel workstreams and hotfix-style integration.
8. Solutions Implemented
Parser strategy: express.json with type callback returning false for paths that must remain unparsed; added express.text for encrypted text/plain payloads.
Query centralization + iteration: QueryMap updates, CTE for SC center details, parallel/streaming retrieval path for online exam centers.
Access control: Circular ID allow/deny lists and related SQL for restricted circular classes; affiliation response status alignment.
Operational guardrails: Global + login rate limits; maintenance middleware; env knobs for DB names and feature flags (PURE_NON_AICTE_BLOCK, etc.).
Release process: PR merges (e.g. PR #5 live) and deploy workflow updates.
9. Before vs After (Short Summary)
Before (2025-06-16, first commit): No application code history in Git prior to initial scaffold; backend not yet present in this repository timeline.
After (2026-05-05, HEAD): 964+ commits delivering a multi-role COB backend on Express 5 / mysql2 / Joi / JWT, with DEC, circular/QPStat, digital evaluation, MahaIT/RTS integration paths, hardened HTTP layer (CORS, helmet, rate limits), and extensive SQL/query-map coverage for institute/RBTE/admin flows.
10. Metrics (if available)
Metric	Value	Source
Development velocity (proxy)	~964 commits from 2025-06-16 to 2026-05-05	git log --since=2025-01-01
API P95 / P99 latency	Not in repository	N/A
Query time improvement %	Not in repository (commits claim optimization intent only)	N/A
Manual effort / error rate %	Not in repository	N/A
