<!--
Sync Impact Report
Version change: template -> 1.0.0
Modified principles:
- Principle 1 placeholder -> I. Local-First Data Ownership
- Principle 2 placeholder -> II. Explicit Remote Sync Contracts
- Principle 3 placeholder -> III. Deterministic Conflict Resolution
- Principle 4 placeholder -> IV. Compatibility and Migration Discipline
- Principle 5 placeholder -> V. Small, Testable Unity Integrations
Added sections:
- Platform & Packaging Constraints
- Delivery Workflow & Quality Gates
Removed sections:
- None
Templates requiring updates:
- ✅ .specify/templates/plan-template.md
- ✅ .specify/templates/spec-template.md
- ✅ .specify/templates/tasks-template.md
- ✅ .opencode/commands/speckit.constitution.md
Follow-up TODOs:
- None
-->

# SQLite-net for Unity Constitution

## Core Principles

### I. Local-First Data Ownership
Features that persist application state MUST treat the local SQLite database as the
primary source of truth. Reads and writes MUST succeed without network connectivity,
and any remote system MUST synchronize from recorded local state instead of bypassing
it. Rationale: package consumers need deterministic on-device behavior across desktop,
mobile, and WebGL targets with intermittent connectivity.

### II. Explicit Remote Sync Contracts
Any remote sync capability MUST define sync triggers, payload semantics, idempotency,
retry behavior, and partial-failure recovery before implementation begins. Sync
operations MUST preserve local writes until the remote side acknowledges them and MUST
store enough metadata to resume interrupted exchanges. Rationale: implicit or
fire-and-forget sync paths create data loss and unreproducible defects.

### III. Deterministic Conflict Resolution
When local and remote changes diverge, the system MUST apply a documented conflict
strategy selected per record type, such as version vectors, monotonic revisions, or a
user-mediated merge flow. Conflict outcomes MUST be reproducible from stored metadata
and MUST never silently discard unmerged user data. Rationale: conflict handling is a
correctness requirement, not a polish task.

### IV. Compatibility and Migration Discipline
Changes to schema, generated sqlite-net mirrors, or native bindings MUST preserve an
upgrade path across supported Unity targets or ship an explicit migration plan.
Features that add sync metadata MUST version their stored format and define behavior
for older records. Rationale: this package spans generated managed wrappers and
committed native libraries, so uncontrolled format changes are expensive and risky.

### V. Small, Testable Unity Integrations
Implementations MUST prefer the smallest change that fits the existing package
structure, keep runtime logic in `Runtime/`, and add editor/import/build behavior only
when platform handling requires it. Each feature MUST define automated verification for
serialization, sync state transitions, and conflict resolution at the narrowest
available test surface. Rationale: the repo has limited in-repo automation and
generated or vendored code, so tight scope and explicit verification are required.

## Platform & Packaging Constraints

- Generated sqlite-net mirror files in `Runtime/sqlite-net/` MUST be changed through
  upstream sources in `Plugins/sqlite-net~/src/*` and/or
  `Plugins/tools~/fix-library-path.sed` whenever regeneration would overwrite manual
  edits.
- Native library changes MUST use the narrowest relevant `make -C Plugins ...` target
  and document affected platforms in the feature plan.
- Local-first and sync features MUST preserve existing `SQLiteAsset` read-only
  semantics and MUST account for Android and WebGL streaming-assets limitations.

## Delivery Workflow & Quality Gates

1. Specs and plans MUST document offline behavior, sync boundaries, conflict rules,
   migration impact, and supported target platforms before implementation begins.
2. The Constitution Check in each implementation plan MUST fail if it omits
   local-first persistence, resumable sync behavior, deterministic conflict handling,
   compatibility or migration review, or concrete verification steps.
3. Before merge, contributors MUST run the narrowest relevant verification:
   `make -C Plugins source` for generated mirror changes, targeted native build
   commands for native updates, and repo-local editor tests or documented manual
   validation for runtime behavior.
4. Code review MUST confirm that touched files respect generation boundaries and that
   data-loss scenarios, retry behavior, and conflict cases are covered.

## Governance

- This constitution overrides conflicting process guidance in `.specify/`, feature
  plans, and task lists.
- Amendments MUST update this document and every impacted template or command guide in
  the same change, and the Sync Impact Report at the top of this file MUST summarize
  the propagation.
- Versioning follows semantic versioning for governance: MAJOR for incompatible
  principle or governance changes, MINOR for new principles or materially expanded
  obligations, and PATCH for clarifications that do not change required behavior.
- Every plan, task list, and review that touches persistence, sync, migrations, or
  conflict handling MUST include an explicit compliance check against these principles.
- The repo guidance in `AGENTS.md` and `README.md` MAY add implementation detail, but
  they MUST NOT relax the requirements defined here.

**Version**: 1.0.0 | **Ratified**: 2026-06-23 | **Last Amended**: 2026-06-23
