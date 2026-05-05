Module Name: time table frontend

Commercial / Functional Changes:

Center-wise workflow: Added Center-wise Form enablement in Home controlled by an environment flag (incl. prod enablement); fixed flag evaluation to numeric to avoid misbehavior when env values are strings.
User guidance / access messaging: Replaced/updated the access restricted UI and added an instructional section for “upcoming timetable release” and schedule exploration guidance.
Exam session operations updates: Updated environment-driven session config across environments for Summer 2026, including exam type change (staging TENTATIVE → FINAL), session dates, and submission deadline correction to 16/03/2026.
Timetable display improvements: Added a reusable date rendering utility for timetable components and updated its formatting + styling for clearer date display.
UX copy/state messaging: Improved loading/error messages in timetable-related components.
Positive Impact:
Reduced client-side regressions from config: Environment flags standardized to numeric values, removing ambiguity in feature gating (center-wise form).
Better maintainability + consistency of data fetching: Centralized data fetching via RTK Query hooks across multiple components; reduced duplicated request logic (measurable churn: a single refactor was +459/-298 LOC).
Improved user guidance during restricted/preview periods: Replaced access-restricted card with clearer instructional content (UI change of +99/-19 LOC in one feature commit).
Measurable delivery activity (2026 YTD): 33 non-merge commits, 48 total commits since 2026-01-01; +1356/-734 LOC net +622 (non-merge).
Technical Changes / Upgrades:
Previous technology (if replaced): Direct axios calls scattered across components/forms.
New technology used: Redux Toolkit + RTK Query for state + API data fetching.
Framework / library upgrades: None explicitly evidenced in 2026 commit history (no version-bump commits observed).
Database changes: None (frontend module only; no DB artifacts in 2026 commits).
API changes: No new backend endpoints recorded; frontend shifted consumption to RTK Query hooks (query/mutation usage patterns refactored).
Code improvements: Modularized timetable date display via a shared renderDate utility (initial add touched 6 files, +45/-5), then iterative formatting/styling refinements.
Architecture Improvements:
State/data layer modularization: Introduced a more centralized architecture using Redux + RTK Query instead of per-component request logic, improving maintainability and enabling consistent caching/loading/error handling.
Feature gating via environment: Center-wise form behavior became config-driven, supporting safer rollout across dev/staging/prod.
Security Enhancements:
IP restriction controls: Updated production environment toggles to enable/disable IP restriction and updated allowed IP lists (multiple ops/config commits).
Client-side hardening for CDN behavior: Added a custom mitigation to prevent Cloudflare Rocket Loader from rewriting/breaking module scripts and preload links (plugin added: +23/-3 LOC; plus index.html protection).
Challenges Faced:
CDN/script mutation issue: Cloudflare Rocket Loader interfering with module scripts/preloads in the frontend.
Environment type mismatch: Feature flag values behaving inconsistently due to string vs numeric evaluation.
Release toggling churn: Center-wise form section was repeatedly commented/uncommented around rollout timing.
Solutions Implemented:
Rocket Loader mitigation: Implemented a targeted plugin + HTML adjustments to stop Rocket Loader from rewriting module script/preload tags.
Normalized env flags: Standardized environment configs to numeric and fixed conditional checks accordingly in Home.
Config-driven rollout: Shifted center-wise UI enablement to an explicit environment switch (incl. production enablement).
Before vs After (Short Summary):
Before: Axios-based, component-level fetching; center-wise form not reliably controlled; access/release messaging less guided; Cloudflare Rocket Loader could break module behavior.
After: RTK Query-based fetching with centralized patterns; center-wise form gated via numeric env config (prod-ready); clearer instructional UI for restricted/release states; Rocket Loader mitigations applied.
Metrics (if available):
Delivery volume (2026 YTD): 33 non-merge commits (48 total incl. merges).
Code churn (2026 YTD, non-merge): +1356 insertions / -734 deletions (net +622).
Largest refactor footprint: Axios → RTK Query refactor commit: +459 / -298 LOC.
Runtime metrics (API/UX timing/error rate): Not present in git history; no measured response time/error rate deltas were recorded in commits.