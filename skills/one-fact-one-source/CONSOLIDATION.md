# Consolidation Steps

Read this only for consolidation or refactor work. Audit-only and small-change tasks stop after Discover and the Before Editing record in `SKILL.md`.

## While Editing

1. Before making consolidation edits, verify the working tree is clean, record the current baseline commit, and run existing tests and builds where available. If the tree is not clean, stop and report instead of mixing unrelated changes. Create commits only when the task explicitly authorizes git writes; when authorized, keep the consolidation in a separate scoped commit so it can be reverted independently.
2. Keep one canonical implementation and export it from the owning module.
3. Change every known consumer to import, call, or read the canonical result.
4. Remove obsolete parallel functions, constants, types, state sets, and inline decisions. Keep a compatibility alias only when required by the task or by compatibility constraints recorded in Before Editing, make it delegate to the canonical source, and document why it remains.
5. Keep adapters, controllers, UI wrappers, and platform wrappers limited to input/output conversion.
6. Treat caches, read models, and derived fields as projections, not sources of truth. Document their source and invalidation boundary.
7. Use a shared package, generated artifact, or drift check for cross-repo mirrors. Do not rely on manual copying alone.
8. Treat consolidation as moving without behavior change unless the task explicitly authorizes a behavior change.
9. Treat renames of API fields or routes, database fields or values, event names, configuration keys, serialized names, and cross-repo contracts as potentially breaking identifier changes. Do not make them without explicit authorization.
10. List every authorized behavior or identifier change separately in the final report.

## Validate

Before declaring completion:

1. When the project has an existing test framework, add or update canonical tests for normal, boundary, fallback, and invalid cases. Do not introduce a new test framework without authorization; if none exists, record the available manual or build verification and recommend tests.
2. Run affected full test suites, type checks, linters, and builds that exist in the project.
3. Use `rg` to prove obsolete names, copied conditions, duplicated state sets, and magic values are gone. List any documented compatibility names that intentionally remain and prove they delegate to the canonical source.
4. Re-run any `jscpd` or `ast-grep` checks used during discovery and record the before/after clone counts.
5. If the project already has lint rules, CI checks, or contract tests for this purpose, extend them to forbid old names from returning. Otherwise, recommend adding such a guard in the final report instead of adding CI or lint infrastructure uninvited.
6. Run an existing deterministic drift check when mirrors exist. If none exists, compare every accessible mirror using an appropriate reproducible content, contract, generated-artifact, or hash comparison. If a mirror is inaccessible, report it as not checked and do not claim cross-repo completion. If drift remains, fail closed and list the files.
7. Report the baseline commit, pre-edit working-tree state, git-write authorization, consolidation commit when created, canonical fact/name/file, definition count before/after, search methods and coverage, known blind spots, migrated consumer count, removed old names, retained compatibility names, intentional differences, authorized behavior or identifier changes, clone scan before/after, test/check/build outputs, inaccessible mirrors, unresolved drift, conflicts, and risks.

Return to `SKILL.md` for the Completion Gate.
