1) Module Name: Affiliation
2) Commercial / Functional Changes
Institute-side affiliation status / payments / uploads / reports and RBTE-side confirmation, institute lists, certificates, add-courses, reporting (scoped to src/**/affiliation/** + affiliation nav configs).
3) Positive Impact
Operational reliability: loading/error handling improvements on affiliation status flows reduce failed interactions during peak submission windows.
Print correctness: conditional rendering refinements for MSBTE block/closure reduce incorrect print outputs (fewer reprints).
4) Technical Changes / Upgrades
Stack: React + MUI + RTK Query patterns inside institute/RBTE sections; report links continue to use VITE_REPORT_URL dashboards.
5) Architecture Improvements
Clear split between src/sections/institute/affiliation vs src/sections/rbte/affiliation, with dedicated nav config files for each portal.
6) Security Enhancements
No affiliation-specific auth mechanism changes isolated in the scoped diff set; relies on existing portal authentication.
7) Challenges Faced
Complex print layouts and status-dependent UI branches (block/closure) without breaking existing institutes’ workflows.
8) Solutions Implemented
Targeted UI refactors + conditional rendering simplification while preserving required fields for finance/affiliation checks.
9) Before vs After (Short Summary)
Before: more fragile affiliation status UX and less consistent print branching.
After: more defensive UI (loading/error) and clearer conditional print behavior.
10) Metrics (path-scoped since 2025-01-01)
Commits: 80
Files touched: 35
Line churn: +24,428 / −10,811 (net +13,617)
1) Module Name: Certificate of eligibility (COE)
2) Commercial / Functional Changes
Institute + RBTE COE workflows (admission eligibility + institute eligibility + manual eligibility UI) and the narrow COE Redux slices under src/redux/slices/COE/**/COE/** (explicitly excludes unrelated COE/... folders like ec-dc, enrollment, scholarship, etc., to keep metrics COE-pure).
3) Positive Impact
Eligibility correctness: stronger subject-selection / eligibility validation reduces invalid COE applications before RBTE/institute confirmation steps.
Reporting: expanded/structured COE report navigation improves traceability for applied vs confirmed cohorts.
4) Technical Changes / Upgrades
React Hook Form + Joi style validation improvements (e.g., eligibility subject selection flows).
MUI DataGrid layout refinements (e.g., auto row height for dense tables).
5) Architecture Improvements
COE UI remains modularized by admission vs institute vs manual routes under src/pages/**/COE/** mirroring paths.institute.coe / paths.rbte.coe.
6) Security Enhancements
Input validation tightening on eligibility-related fields within scoped COE components (no new auth system introduced in COE-only paths).
7) Challenges Faced
Course/scheme-driven UI branching (checkbox visibility, semester labeling) across many scheme/course combinations.
8) Solutions Implemented
Refactored selection logic + dynamic labels + clearer validation gates tied to course code / exam history rules.
9) Before vs After (Short Summary)
Before: more manual/fragile eligibility subject handling.
After: more automated selection constraints + improved reporting entry points.
10) Metrics (COE-narrow path scope)
Commits: 153
Files touched: 93
Line churn: +21,068 / −5,470 (net +15,598)
1) Module Name: Exam form
2) Commercial / Functional Changes
Institute Winter exam form (Exam-Form-for-WINTER), RBTE winter exam form (exam-form-winter), candidate exam form pages, and examform Redux scoped to:
src/redux/slices/examform/index.js
src/redux/slices/examform/regular-examform/**
src/redux/slices/examform/x-examform/**
Explicitly excludes src/redux/slices/examform/copycase/** and src/redux/slices/examform/stationery/** to avoid cross-module inflation.
3) Positive Impact
Attendance & seating operations: measurable churn added multi-print attendance flows, stronger loading/error handling, and seating-chart correction safeguards (scoped lines land primarily in exam-form paths).
Fewer bad submissions: correction page improvements reduce duplicate/incorrect subject corrections.
4) Technical Changes / Upgrades
Joi schema adjustments (e.g., digit-length flexibility for enrollment/seat fields tied to exam-form screens).
Print UX: dedicated print components for attendance / hall-ticket related outputs within exam-form modules.
5) Architecture Improvements
Clear separation between Regular vs X exam form Redux slices (regular-examform vs x-examform) instead of a single monolithic API file for all exam-domain features.
6) Security Enhancements
Validation hardening on exam-form inputs; operational fixes to user_type values used in attendance submission flows (role correctness).
7) Challenges Faced
Large, parallel workflows (institute vs RBTE vs candidate) sharing similar screens but different rules.
8) Solutions Implemented
Route/menu alignment + component refactors to isolate RBTE vs institute behaviors while reusing shared examform APIs where safe.
9) Before vs After (Short Summary)
Before: fewer attendance/correction affordances; more brittle validation on high-variance identifiers.
After: expanded attendance tooling + safer correction submission behavior.
10) Metrics (exam-form-pure path scope)
Commits: 289
Files touched: 136
Line churn: +39,603 / −7,478 (net +32,125)
1) Module Name: Copy case
2) Commercial / Functional Changes
Institute Exam Center + SC center malpractice/copy-case flows (add/edit/confirm/print) and RBTE copy-case malpractices admin flows, plus dedicated copycase Redux endpoints.
3) Positive Impact
Data integrity: normalization fixes in edit/confirm tables reduce incorrect candidate display.
Operator feedback: toast-based success/error handling on confirmations reduces silent failures.
4) Technical Changes / Upgrades
Continued use of RTK Query mutations under src/redux/slices/examform/copycase/** with UI in src/sections/**/copyCase/**.
5) Architecture Improvements
Copy case remains isolated as its own UI subtree + slice, reducing coupling with regular exam-form slices.
6) Security Enhancements
Stronger client-side validation around seat/paper/sr identifiers for malpractice capture (scoped UI).
7) Challenges Faced
High-stakes workflows (malpractice) with print + tabular editing and strict seat/paper joins.
8) Solutions Implemented
Table mapping fixes + print layout refinements + clearer confirmation UX.
9) Before vs After (Short Summary)
Before: more error-prone mapping/display in edit/confirm grids.
After: improved mapping, print clarity, and user-visible outcomes on confirm.
10) Metrics
Commits: 76
Files touched: 43
Line churn: +9,108 / −2,289 (net +6,819)
1) Module Name: Exemption
2) Commercial / Functional Changes
Institute exemption apply/confirm/print/cancel + report section, RBTE exemption confirm/cancel/print/report flows, plus RBTE exemption Redux (src/redux/slices/COE/rbte/exemption) and the institute typo-folder slice src/redux/slices/COE/institute/examption (as present in repo).
3) Positive Impact
RBTE throughput: table/layout refactors (auto row height) improve usability on wide confirm tables.
Reporting: report accordion refactors reduce duplicated UI code across exemption report lists.
4) Technical Changes / Upgrades
MUI DataGrid / table presentation updates; shared AccordionComponent reuse for report lists.
5) Architecture Improvements
Keeps exemption UI + API slice adjacent under RBTE/institute namespaces, aligned to paths.institute.exemption / paths.rbte.exemption.
6) Security Enhancements
No standalone auth overhaul in scoped exemption paths; relies on portal auth.
7) Challenges Faced
Seasonal rename churn (e.g., Winter 2025 → Summer 2026 references) across many instructional strings.
8) Solutions Implemented
Systematic text/reference updates + reusable report presentation components.
9) Before vs After (Short Summary)
Before: more duplicated report UI + less responsive dense tables.
After: more reusable reporting UI + improved dense-table readability.
10) Metrics
Commits: 45
Files touched: 27
Line churn: +5,438 / −1,799 (net +3,639)
1) Module Name: Scholarship
2) Commercial / Functional Changes
Institute scholarship UI under src/sections/institute/scholership (repo spelling), RBTE scholarship pages under src/sections/rbte/scholership + src/pages/rbte/scholership, and Redux src/redux/slices/COE/rbte/scholarship + src/redux/slices/COE/institute/Scholership.
3) Positive Impact
Autonomous flows: added/expanded document verification + edit flows with clearer row selection and status display.
Reporting toggles: explicit enable/disable adjustments for “Scholarship Allotted Report with Bank Details” reduce accidental publication of incomplete report links.
4) Technical Changes / Upgrades
API mutations extended with parameters like is_autonomous where required for document retrieval paths.
5) Architecture Improvements
RBTE scholarship verification/edit flows separated into dedicated route pages under src/pages/rbte/scholership/**.
6) Security Enhancements
Stronger document verification UX (status/selection), reducing mis-clicks that send wrong documents downstream (UI-layer risk reduction).
7) Challenges Faced
Autonomous vs non-autonomous branching and bank-details reporting sensitivity.
8) Solutions Implemented
Conditional API parameters + more explicit UI states + controlled report list changes.
9) Before vs After (Short Summary)
Before: fewer autonomous scholarship controls and less explicit document state handling.
After: richer autonomous edit/verify behavior and safer report list management.
10) Metrics
Commits: 103
Files touched: 46
Line churn: +22,566 / −7,379 (net +15,187)
1) Module Name: Stationery (Institute stationary/* + RBTE stationery/*)
2) Commercial / Functional Changes
Institute stationery management (src/sections/institute/stationary/**), RBTE stationery module (src/sections/rbte/common/stationery/** + pages), and src/redux/slices/examform/stationery/**.
3) Positive Impact
Operational accuracy: EC vs DC labeling fixes reduce wrong dispatch decisions (measurable as fewer user-reported “wrong center type” issues—tracked qualitatively; not instrumented in-repo).
Throughput: loading states + validation on Summer 2026 stationery flows reduce incomplete submissions.
4) Technical Changes / Upgrades
Schema validation fixes (e.g., remarks → remark) and DataGrid column/data swaps to correct rendering.
5) Architecture Improvements
Stationery Redux isolated under examform/stationery rather than mixing into unrelated slices.
6) Security Enhancements
Validation/schema tightening reduces bad payloads sent to stationery APIs (client-side).
7) Challenges Faced
Complex stationery matrices (32-page variants, barcode/QR/drawing) with strict totals.
8) Solutions Implemented
Serial number range editing with validation hooks; clearer EC/DC list retrieval consistency fixes.
9) Before vs After (Short Summary)
Before: more error-prone grids/labels during Summer 2026 rollout prep.
After: more validated, labeled, and loading-stable stationery capture.
10) Metrics
Commits: 51
Files touched: 19
Line churn: +6,907 / −2,913 (net +3,994)
1) Module Name: Enrollment
2) Commercial / Functional Changes
Parallel institute + RBTE enrollment UI (src/sections/**/enrollment/**, src/pages/**/enrollment/**) plus enrollment Redux slices under src/redux/slices/COE/**/enrollment and src/redux/slices/institute/enrollment.
3) Positive Impact
Print completeness: added child institute details and improved print outputs for enrollment confirmations.
Verification: expanded document verification options and submission logic clarity.
4) Technical Changes / Upgrades
MUI Grid sizing adjustments for responsive application forms; routing path corrections for non-AICTE application entry.
5) Architecture Improvements
Enrollment remains split by AICTE vs NON-AICTE subtrees, preserving operational separation.
6) Security Enhancements
Stronger validation around institute identifiers in enrollment-related forms (scoped changes).
7) Challenges Faced
Terminology consistency (“Candidate” vs “Student”) across large forms.
8) Solutions Implemented
Systematic copy updates + routing fixes + print enrichment.
9) Before vs After (Short Summary)
Before: less complete enrollment print/verification coverage.
After: richer verification + clearer routing + improved print payload coverage.
10) Metrics
Commits: 53
Files touched: 51
Line churn: +16,947 / −4,044 (net +12,903)
1) Module Name: ABCID
2) Commercial / Functional Changes
Candidate ABC ID creation (src/sections/candidate/create-abc-id/**, src/pages/candidate/create-abc-id/**) + institute dashboards/reports (abc-id-card, abc-id-data-report) + abcidApiSlice + create-abc-id slice + validation helper.
3) Positive Impact
OTP-based onboarding reduces manual institute helpdesk interventions for identity creation (qualitative; not measured in-repo).
4) Technical Changes / Upgrades
Dedicated apiSliceABCID integration and iterative OTP verification endpoint adjustments.
5) Architecture Improvements
Clear separation: candidate flow vs institute reporting vs shared API slice.
6) Security Enhancements
OTP verification flow changes are directly in-scope for reducing fraudulent ABC ID creation (client + API contract alignment).
7) Challenges Faced
Endpoint drift during rollout (OTP verify URL updates).
8) Solutions Implemented
Targeted endpoint updates + UI hardening on OTP steps.
9) Before vs After (Short Summary)
Before: smaller/less complete ABC ID UX and more brittle OTP wiring.
After: fuller create flow + institute reporting surfaces + updated OTP integration.
10) Metrics
Commits: 14
Files touched: 13
Line churn: +2,232 / −494 (net +1,738)
1) Module Name: Search institute
2) Commercial / Functional Changes
Public/auth-surface institute search + table + print (src/sections/search-institute/** only; does not count paths.js / route-config / .env edits unless you expand scope).
3) Positive Impact
Broader ID acceptance + improved print layout reduces institute-support rework for “cannot find institute” cases (qualitative).
4) Technical Changes / Upgrades
MUI DataGrid enhancements for totals + search params; print formatting improvements; migration to useSecureSearchParams for encrypted query handling.
5) Architecture Improvements
Clear 3-surface split: search, table, print components under one module folder.
6) Security Enhancements
Encrypted query parameters (useSecureSearchParams) reduce sensitive query leakage in URLs for this module.
7) Challenges Faced
Evolving institute ID formats + print fidelity vs screen tables.
8) Solutions Implemented
Validation/mapping normalization + print-first layout adjustments + secure params.
9) Before vs After (Short Summary)
Before: narrower ID support + less print-ready output.
After: broader ID support + stronger print output + safer URL query handling.
10) Metrics (UI folder scope only)
Commits: 48
Files touched: 3
Line churn: +3,532 / −1,508 (net +2,024)
1) Module Name: Digital evaluation
2) Commercial / Functional Changes
Institute-only digital evaluation activity suite: DC controller receipts, SC↔DC, SC→SC team lead flows + print pages (src/sections/institute/digital-evaluation-activity/**, src/pages/institute/digital-evaluation-activity/**).
3) Positive Impact
Receipt reconciliation: added totals on print pages reduces manual summation during DC/SC audits.
Operator clarity: renamed headers/notes and improved empty-state handling reduce misreads.
4) Technical Changes / Upgrades
Print CSS improvements (auto page sizing), signature line standardization, toast feedback for empty lists.
5) Architecture Improvements
Feature isolated under a dedicated folder namespace (digital-evaluation-activity) rather than being embedded in generic “institute common”.
6) Security Enhancements
No new crypto/auth subsystem in scoped paths; benefits from portal auth.
7) Challenges Faced
Multi-role print variants (DC vs SC vs team lead) with consistent totals.
8) Solutions Implemented
Separate print routes/components + totals blocks + standardized signature/time blocks.
9) Before vs After (Short Summary)
Before: less complete receipts/prints for digital evaluation activity tracking.
After: totals + clearer print layouts + better empty/error feedback.
10) Metrics
Commits: 26
Files touched: 11
Line churn: +2,450 / −517 (net +1,933)
1) Module Name: DC EC CI (RBTE dc-ec + ec-dc Redux)
2) Commercial / Functional Changes
Full RBTE distribution center / DEC capacity / scanning center mapping surface area: src/sections/rbte/dc-ec/**, src/pages/rbte/dc-ec/**, src/redux/slices/COE/rbte/ec-dc/**.
3) Positive Impact
Capacity management throughput: second/final year DEC capacity screens gained loading states and clearer “connected institute” semantics (rac_code-driven navigation appears in this tree).
4) Technical Changes / Upgrades
Additional RTK mutations for DEC capacity present/save/final-year edit flows; UI refactors on DEC capacity pages.
5) Architecture Improvements
Centralized ECDCCI API surface in ec-dc slice consumed by multiple DEC pages.
6) Security Enhancements
Payload clarity improvements (e.g., rac_code → API field mapping) reduce accidental wrong-institute submissions (data-integrity/security adjacent).
7) Challenges Faced
Overlapping concepts (DEC vs DC vs EC lists) causing UI mislabeling without strict naming discipline.
8) Solutions Implemented
Header renaming (DC→EC where appropriate), list retrieval hook corrections, save-button loading guards.
9) Before vs After (Short Summary)
Before: more ambiguous DEC/EC/DC presentation; fewer guardrails on saves.
After: clearer naming, stronger loading/save behavior, expanded DEC capacity management coverage.
10) Metrics
Commits: 66
Files touched: 57
Line churn: +10,606 / −2,826 (net +7,780)
1) Module Name: RAC (DEC capacity-by-rac_code UI slice)
2) Commercial / Functional Changes
RAC-scoped DEC capacity editing/navigation UI:
src/sections/rbte/dc-ec/dec/second-edit-DEC-capacity-details/**
src/sections/rbte/dc-ec/dec/final-year-edit-dec-capacity-details/**
related pages under src/pages/rbte/dc-ec/dec/** (second-year edit / filled-not-edit / final-year edit)
3) Positive Impact
Navigation precision: rac_code query-driven flows reduce wrong institute context when editing connected institute capacity.
4) Technical Changes / Upgrades
Explicit query param handling (rac_code) and payload mapping (rac, rac_inst_arr) aligned to backend expectations.
5) Architecture Improvements
Keeps RAC-sensitive edit flows localized to dedicated DEC edit subtrees (even though DC EC CI contains them too).
6) Security Enhancements
Data integrity: clearer payload keys reduce accidental mis-posting of institute identifiers in DEC edits.
7) Challenges Faced
RAC is both a reporting concept (rac-capacity-reports URL exists in paths.rbte.dcec) and an edit-flow key—easy to confuse in UI.
8) Solutions Implemented
Dedicated edit pages + grid linkage to secondYearDecCapacityEdit?rac_code=... style navigation.
9) Before vs After (Short Summary)
Before: less explicit RAC-aware navigation/payload structure in edit flows.
After: RAC-aware edit routing and clearer connected-institute presentation.
10) Metrics (RAC-only subtree; subset of DC EC CI tree)
Commits: 13
Files touched: 8
Line churn: +1,752 / −399 (net +1,353)
1) Module Name: Emarksheet
2) Commercial / Functional Changes
RBTE emarksheet password reset (non-theory) + reports UI and emarksheet API slice wiring.
3) Positive Impact
Operational reporting: added/updated report entries (e.g., sheet statistics NTH) expands RBTE reporting coverage without new manual report builds in separate tools.
4) Technical Changes / Upgrades
Centralized RTK base query refactor touched emarksheet slice as part of broader API consistency (scoped lines remain in emarksheet paths).
5) Architecture Improvements
Emarksheet remains isolated under src/sections/rbte/emarksheet + dedicated Redux folder.
6) Security Enhancements
Password reset flows are inherently security-sensitive; changes stayed within RBTE emarksheet module boundaries in scoped paths.
7) Challenges Faced
Report URL drift across seasons (Winter 2025 vs Summer 2026 naming).
8) Solutions Implemented
Report list adjustments + environment URL corrections in emarksheet module files.
9) Before vs After (Short Summary)
Before: smaller emarksheet report surface + some outdated entries.
After: expanded/adjusted report set aligned to current exam season operations.
10) Metrics
Commits: 9
Files touched: 8
Line churn: +567 / −291 (net +276)