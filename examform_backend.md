Module Name:
examform module

Commercial / Functional Changes:

Elective completeness rule added: regular form now persists selected electives with flag='Y' and also persists remaining/unselected electives as optional with flag='O' in exam_subject_s (via insertOptionalElectiveSubjects), preventing “missing elective” commercial gaps.
Idempotent elective backfill: remaining electives insert uses NOT IN (existing exam_subject_s elective subject_abbr) to avoid duplicates on rerun/reprocess.
Exemption handling tightened: main subjects can be updated to flag='E' for exempted papers (updateExemptionFlag), aligning exemption business rule with stored subject rows.
Feature-window enforcement: examform apply/confirm/cancel are gated by centralized config flags like START_EXAMFORM_W_* (hard close/open control for operations).
Role-based workflows: separate flows exposed for Candidate / Institute / RBTE with distinct routes and access control.
Positive Impact:
Error reduction: eliminated cases where electives were not present in exam_subject_s after selection (now guaranteed: selected Y + remaining O).
Reprocessing safety: elective backfill is duplicate-proof (idempotent insert), reducing manual DB cleanup during retries.
Operational control: config-based gating reduces “link open/close” manual interventions to 0-code operational toggles.
DB efficiency (measurable by query shape): subjects are inserted using set-based INSERT ... SELECT (single statement per category) instead of per-subject inserts for Regular flow.
Technical Changes / Upgrades:
Previous technology (if replaced): N/A (not verifiable from current repo snapshot).
New technology used: Node.js backend with Express routing, JWT auth, Joi validation, MySQL queries via shared service/query layer.
Framework / library upgrades: N/A (no version delta measured here).
Database changes:
Subject inserts standardized as INSERT ... SELECT from course_sub_marks / course_sub_marks_lateral.
paper_code mapped consistently from paper_code_eq in marks master during inserts.
Added copy-table and BK-table write paths (e.g., exam_subject_s_copy, exam_subject_w_bk, block-specific copies) controlled via config EXAM_FORM_W_BK.
API changes:
Introduced/standardized endpoints for apply flow: /beforeApply, /apply, /applyNext and X-form variants, with shared controller dispatch.
applyNext implements copy-then-delete on re-apply: move existing rows to *_copy table, then delete from live table before reinserting.
Code improvements:
Centralized SQL in query maps (shared/query.js, utils/functions/examform/query.js).
Dynamic subject table resolution by form type + block (exam_subject_s vs exam_subject_x_s vs exam_subject_a_g_w).
Added in-module caching (Map) for repeated lookups (paper-code block and exam-center resolution) to reduce repeated DB hits.
Architecture Improvements:
Modularization: examform split by domain roles (resources/candidate|institutes|rbte/examform/*) with shared core (resources/shared/*).
Maintainability: query strings and business logic separated (controller → utils/service → query map), reducing duplication across Candidate/Institute/RBTE flows.
Scalability (practical): Regular subject insertion is set-based; caching added for repeated validations; copy/BK paths reduce risk during mass operations.
Security Enhancements:
Authentication/authorization: enforced JWT-based role checks (candidateAuth, instituteAuth, rbteAuth) and per-route access control.
Data validation: strict Joi schemas for apply payloads (enrollment format, course/year/master codes, appear code, exam_no length, block validation).
Operational security: feature flag gating (requireConfig) prevents unauthorized usage outside the configured window (hard 403 + explicit disabled flag list).
Challenges Faced:
Elective inconsistency: only selected electives were being stored, leaving unselected electives absent in exam_subject_s, causing downstream commercial/reporting mismatches.
Safe retries: reprocessing could create duplicates or require manual cleanup without guarded insert strategy.
Multi-form complexity: Regular vs X vs Block-G needed different persistence rules and table targets.
Solutions Implemented:
Optional elective persistence: implemented insertOptionalElectiveSubjects to insert remaining electives with flag='O' and a NOT IN guard on existing electives.
Re-apply strategy: implemented applyNext flow to *copy existing rows to _copy and delete before reinsert, preserving audit while keeping live table clean.
Dynamic routing + table selection: encapsulated subject-table choice based on r_x + block, and used block-specific insert/replace for X Block-G.
Config + validation: enforced config windows and Joi validation at route level to reduce invalid/late requests.
Before vs After (Short Summary):
Before: elective dataset in exam_subject_s could be incomplete (missing non-selected electives); reruns risked duplicates/manual cleanup; flow controls were more manual.
After: electives are complete (Y + O), reruns are safe via idempotent insert + copy/delete reapply, and operations are controlled via config flags + strict validation.
Metrics (if available):
API response time reduced from ___ to ___: N/A (no runtime benchmarks provided).
Query execution time improved by ___%: N/A (no profiling numbers provided).
Manual effort reduced by ___%: measurable directionally—reprocessing no longer needs manual duplicate cleanup due to idempotent inserts; exact % N/A.
Error rate reduced by ___%: N/A (no production error stats provided).