1. Module Name
Non-Theory Emarksheet frontend — MSBTE EMS Non-Theory e-marksheet SPA (ems_nth_frontend_app; UI labels reference “Non Theory Emarksheet” / “Non-Theory Emarksheet”; config-global.js names the app Emarksheet Non-Thoery).

2. Commercial / Functional Changes
Role / feature gating via env flags (examples from commits): VITE_SHOW_* for HOD, examiner, principal, institute, automated/manual e-mark sheets, reports, message system, confirmation flows, maintenance bypass, devtools, institute update, office-in-charge update — toggled across development / staging / production .env* files for phased rollout and exam windows.
Exam / product naming: Copy and metadata moved across Winter 2025 Non-AICTE Non-Theory → Summer 2026 / Special Term strings for principal, examiner, HOD reports and titles (82285b4, 6596f17, 3bfa3c2, 78b6e0a, cf70f8b, cd1049e, etc.).
Separate Non-AICTE Non-Theory entry: VITE_SUBFOLDER_NTH_NONAICTE added to envs; main nav “Non Theory Emarksheet Non AICTE Special term Summer 2026” links to /${subFolder_NTH}/ with target="_blank" and rel="noopener" (aadf920, f4b5534, 0624b6e, src/layouts/config-nav-main.jsx).
Marks business rules (iterative): Validation updated multiple times — e.g. 0–100 + special 401 → 401–408 → 0–200 → 0–1000 with 401-only in latest relevant fix (454a95b, 6a52c39, 98e0493, b9b8b0c, dd974aa); 401 allowed for specific I-schemes (c82c6a3, da09534); invalid marks cleared and inputs refocused (fe062ff).
External examiner / HOD flows: “Other” for designation, qualification, institution; Aadhar paste/limit instructions; seat no wording vs sheet no (f3390c6); seat number exactly 6 digits; Aadhar 12 digits; external course/institute code input rules (0e1ae28, 19709d1, e1390c2, 0d4a201, a7089cd).
Detention: Submit disabled when no detained candidates (4811985).
Undertaking / examiner APIs: New undertaking-related API usage called out in history (16e6cce).
Reports: HOD/examiner report tables — subject_thpr and subject_subthpr in one cell; pagination/print row fixes (0ef1ce8, e9157d0, 63d7a19).
Maintenance: Time-windowed MaintenanceMode wrapper and env-driven force/disable (1fe177f, f78825a, a78aef9, 4c68f8e, fd23f50, etc.).
Institute login: Points to theory subfolder (VITE_SUBFOLDER_TH) while Non-Theory stays on main subfolder (908786e, config-nav-main.jsx).
3. Positive Impact
Operational control: Feature flags reduce need for code deploys to open/close logins, e-mark sheet modes, reports, and maintenance (02a75cb, b1b989c, 94b3af4, etc.).
Data freshness: RTK Query refetchOnFocus / refetchOnMountOrArgChange on detention/marksheet-related sections (dbef77d).
Less chatty polling: Institute mobile forms refactored to drop unnecessary polling (a8c1045).
UX / error clarity: Toasts and disabled submit states on dialogs (a70c1e9, 75c704b); centered sign-in and create-examiner errors more specific (e93686b, 24c5a4e); non-array statusReport guarded in reports (a03cc40).
Print / reporting: Report table normalization and print path fixes reduce missing rows when printing (63d7a19).
Measurable from repo only: Username max length 10 → 11 (8482dd1); marks upper bound moved 100 → 1000 in schema/dialogs at various commits (790d255, a2a450e, 6a52c39); final business rule in history tightened to 0–1000 or 401 only (b9b8b0c).
4. Technical Changes / Upgrades
Stack (current package.json): React 18.3.x, Vite 6.3.x, @vitejs/plugin-react-swc, Node 20.x (engines), Material UI 5 / MUI X 7, Redux Toolkit 2.5 + React Redux 9, Axios 1.7, React Router 6.26, Zod 3.23, React Hook Form 7.52, @react-pdf/renderer 3.4, ESLint 8 + vite-plugin-checker.
Build / CDN: modulePreload: false in Vite build; custom cloudflareRocketLoaderFix helper in vite.config.js (Rocket Loader / data-cfasync pattern; commit 62e1eec — note: the helper exists in file; the plugins array currently lists only react() and checker(), so verify in your branch whether the plugin is also registered).
Client config: Axios instance updated to prefer VITE_SERVER_URL (8756ffb).
CI: ESLint auto-fix in CI workflows (0266cbd).
API surface (frontend): RTK Query apiSlice extended (undertaking, marksheet fill/confirm paths per commits); exact endpoint list belongs in src/utils/apiSlice (not fully enumerated here).
Code quality: Refactors in FillUpdateFormSection, AddExternalExaminerFormSection, AutomatedEMarkSheetsExaminer (localStorage reactivity), ReportsTable, InstituteMobileForm, CancelSheetNumberRequestSection (role vs username), ExaminerMarksheetDialog (confirm/submit state), etc.
5. Architecture Improvements
Remains a single Vite + React SPA with lazy-loaded dashboard routes (src/routes/sections/dashboard.jsx pattern in codebase).
Scalability / maintainability: Environment-driven navigation and features centralize rollout; MaintenanceMode wraps the app tree (src/main.jsx); clearer separation of automated vs manual e-mark sheet route trees for HOD/examiner. No evidence in history of monolith → microservices migration (frontend-only repo).
6. Security Enhancements
Navigation hardening: External Non-Theory app link uses rel="noopener" with target="_blank" (0624b6e).
Client-side validation tightening: Numeric seat and Aadhar length rules; alphanumeric constraints on external codes (0e1ae28, 19709d1, 8e019c8).
Production hardening toggles: Devtools guard import enable/disable in main.jsx and related logout/block behavior adjusted over several commits (7ba121b, dd34944, 172159b, 7e01ca4).
Auth UX: Logout redirect loop mitigation when token cleared vs in-memory auth (src/pages/home.jsx pattern in codebase).
No committed evidence of JWT algorithm changes, new IdP, or server-side RBAC (this repo is frontend-only).
7. Challenges Faced
Evolving marks rules: Multiple successive commits changing allowed ranges and special codes indicate changing examination/registrar rules or clarification cycles (454a95b, d0ee2c9, 98e0493, 6a52c39, b9b8b0c).
CDN + SPA: Cloudflare Rocket Loader vs Vite modulepreload / script loading (62e1eec, modulePreload: false in vite.config.js).
Coordinated env drift: Staging VITE_SERVER_URL ordering/comment fixes (3a5b11c, 8cf3b04, c348734).
Feature-flag matrix complexity: Many combinations of examiner/HOD/institute/manual/automation/maintenance across three environments.
8. Solutions Implemented
Feature flags in .env.* for controlled exposure.
MaintenanceMode time windows + force flags for controlled downtime messaging.
Vite build adjustments (modulePreload: false, Rocket Loader–related HTML transforms in repo).
Schema/UI alignment for marks and external examiner fields with user-visible error text and button loading/disabled states.
RTK Query refetch policies and reduced polling where data could go stale.
9. Before vs After (Short Summary)
Before (early 2025): New frontend baseline (2025-02-22 first commit); simpler env set; later waves added flags, maintenance, and richer examiner/HOD flows.
After (through latest inspected commits, e.g. 2026-05-03): Non-AICTE Non-Theory Summer 2026 / Special Term naming; dedicated Non-Theory subfolder URL + env; nav link opens in new tab with noopener; marks validation 0–1000 or code 401 with clearer copy; broader RTK refetch and CI lint automation.
10. Metrics (if available)
Metric	Value in repo / git
API response time (before → after)	Not recorded in this repository
Query execution time improvement %	N/A (no DB in this frontend repo)
Manual effort reduction %	Not quantified in commits
Error rate reduction %	Not quantified in commits
Commit volume (proxy for change rate)	674 commits with git log --since=2025-01-01
Concrete numeric deltas from commits	Username max 10 → 11; marks ceiling 100 → 1000 (intermediate steps 200, 401–408 in history)
