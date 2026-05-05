1. Module Name
Theory e-marksheet frontend — MSBTE EMS theory institute UI (ems_th_frontend_app, @msbte/vite-js v6.1.0).

2. Commercial / Functional Changes
Institute-centric product: Role/navigation refactored from “examiner” to “institute”; examiner pages moved under institute directory structure and routes aligned to institute workflows.
Theory e-marksheet & related flows: Automated e-marksheets, manual examiner marksheet creation/listing, paper code and receipt of answerbook (digital evaluation), sheet allocation / confirmation reports, reports for sheets filled but not confirmed with special codes 401, 403, 409 (pagination/loading).
Marks / special-code rules (business logic): Iterative updates to allowed special marks codes (e.g. 401–408, subsets 404/406/407, then 401, 403, 404, 406, 407, etc.), mark inclusion for codes 401–408, handling of empty/previous special marks, toast messaging for 401/403 and 404/406/407, and marks input later set read-only/disabled for tighter form control.
Operational / academic calendar: Examination season/year labels updated (e.g. Summer 2025 → Winter 2025; 2026 env: subfolders, report years, seasons; vite base path now /MES012026_TH).
Config-driven UI: Feature visibility via env (e.g. paper code list, receipt of answerbook, digital evaluation status; VITE_ALLOW_ALL_CANCEL_SUBJECTS for cancellation rules; VITE_ROWS_PER_PAGE_IN_DASHBOARD).
Receipt / reporting presentation: Institute code on reports, rows per page from env, ReceiptTable totals/special-code counts/answer-book totals, Summer/Winter season text, district center code sourcing fix, username → EC code from localStorage, total calculations with refactored number parsing.
Sign-in / account: Numeric-only institute username step, password-reset redirect on username verification errors, “Change Password” route commented in nav.
Home / filters: “Select ALL courses” in DateFilterForm; home “latest news” copy updates; accreditation text consistency.
App shell (2026-01): New main App wiring providers, routing, maintenance mode.
3. Positive Impact
UX: Loading bars for paper date/day fetch; loading toast on marks submit with dismiss on success/error; clearer validation toasts; scrollbar/nav label tweaks; receipt layout bold labels / tables; breadcrumbs/report copy cleanup.
Data correctness: Fixes for special-code indexing in ReceiptTable, normalization of special-code arrays to flat rows, array checks to avoid double-wrapping in report data paths.
Operations: CI/CD (GitHub Actions) for dev / stage / live with server/path updates and yarn deploy-live (etc.) scripts; ESLint auto-fix in deploy workflow to reduce failed builds from lint noise.
Measurable in-repo: ~117 commits since 2025-01-01; default dashboard pagination moved to 50 rows and made env-configurable; Vite dependency bump recorded in history (6.3.6 → 6.4.1 on 2025-11-04 — current package.json pins ^6.3.5). No checked-in APM numbers for latency/error-rate improvements.
4. Technical Changes / Upgrades
Previous → new (from history): Template/boilerplate pared down toward MSBTE-specific institute app; examiner naming/structure → institute structure.
Stack (current package.json): React 18.3.1, Vite 6.x (^6.3.5), @vitejs/plugin-react-swc, Node 20.x, Yarn 1.22.22, MUI 5, Redux Toolkit / RTK Query 2.5.1, React Router 6.26, Axios 1.7, Auth0 React 2.2, AWS Amplify 6.5, Socket.IO client 4.8, Zod, React Hook Form, ExcelJS, @react-pdf/renderer, etc.
Framework/library upgrades: Dependency refresh (2025-10-01 commits); explicit Vite minor bump in Nov 2025 per log.
Database: None in this frontend repo (no migrations/indexing here). Backend DB is out of scope for this codebase.
API / client: RTK Query apiSlice: new/changed marksheet mutations and queries, getFileByName, cleanup of unused endpoints, transformResponse toggled/commented in places, refetchOnMountOrArgChange / refetchOnFocus / refetchOnReconnect, keepUnusedDataFor: 0. Auth paths evolved: /auth2/checkUser, /auth2/login, etc., while IP query uses /auth/getIp (history shows auth ↔ auth2 corrections on IP and other calls).
Code quality: Large refactors of ReceiptTable, report pages, Marksheet / ManualMarksheet, apiSlice formatting; ESLint config broadened (extensions/resolver). localStorage key standardized: token → authToken.
5. Architecture Improvements
Modularization by domain: Institute dashboards for exam-center activity, digital evaluation reports, automated e-marksheets, reports split into many page + section components.
Scalability / maintainability: Environment-driven feature flags and academic-year/path configuration (VITE_*, import.meta.env), centralized API layer via RTK Query, route consolidation for institute activity. Still a single SPA (not microservices).
6. Security Enhancements
Auth: Continued use of Bearer tokens in prepareHeaders; migration toward /auth2/* for primary login/user flows.
Input hardening: SignIn changes for sanitized mobile/name fields, OTP handling, password visibility controls; numeric-only username input on step 1.
Operational hygiene: Error logging on username verification; clearer client-side validation for marks/special codes (reduces bad submissions; not a substitute for server-side validation).
7. Challenges Faced
Evolving business rules for special marks codes (multiple commit iterations show alignment with changing policy).
Data-shape drift between API and UI (ReceiptTable / reports / seat normalization).
Auth service split (auth vs auth2) causing endpoint corrections (including getIp).
Build/deploy friction addressed via lint autofix and workflow hardening.
8. Solutions Implemented
Iterative validation + toasts + read-only marks when appropriate.
Defensive normalization and indexing fixes in ReceiptTable and report pipelines.
Endpoint updates and apiSlice cleanup to match backend.
CI/CD pipelines and env documentation for dev/stage/live.
9. Before vs After (Short Summary)
Before: Early Sept 2025 initial import; examiner-oriented naming/paths; thinner reporting/marksheet surface.
After: Institute-first theory EMS frontend with broader digital-evaluation and marksheet/report coverage, stricter/configurable UI rules, 2026 deployment base path /MES012026_TH, and automated deployment to multiple environments.
10. Metrics (if available)
API response time (before → after): Not recorded in this repository.
Query execution time improvement %: N/A (no DB in this app).
Manual effort reduced %: Not quantified in-repo (qualitatively: deploy scripts + CI + env-driven toggles reduce repeated manual config).
Error rate reduced %: Not quantified; commits explicitly target indexing, normalization, and validation bugs (effect unmeasured here).