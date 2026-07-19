---
name: one-fact-one-source
description: Use when a coding task may create, change, audit, or consolidate multiple implementations of the same business fact across functions, constants, statuses, permissions, amounts, schemas, contracts, derived caches, or repositories. Finds the canonical source and name, distinguishes semantic duplicates from similar code, migrates consumers without unapproved behavior or contract changes, and verifies obsolete definitions and drift are gone.
---

# One Fact, One Source

## Core Rule

Keep exactly one canonical function, constant, type, schema, contract, or authoritative output for each business fact.

Allow consumers to present or transport the result differently. Do not let them reimplement the underlying status, amount, permission, classification, or business decision.

Do not merge code merely because it looks similar. Merge only when the business meaning, inputs, boundaries, and ownership are the same. Preserve and document intentional differences.

## Scope of Effort

Match the process weight to the task:

- **Audit only**: run Discover and produce the Before Editing record. Report definitions, consumers, conflicts, and the recommended canonical source when the evidence is sufficient. Do not modify files or continue into consolidation.
- **Small change** (touches one existing business fact, no consolidation intended): run a targeted semantic search across business terms, names and aliases, status literals, magic values, equivalent conditions, schemas, contracts, and call sites. Reuse the canonical source without adding a parallel implementation. Skip repo-wide counting, clone scans, and the full consolidation report unless the search reveals conflicting or duplicated definitions.
- **Consolidation or refactor** (merging duplicates, extracting a canonical source, cross-repo work): follow the full Discover -> Before Editing -> While Editing -> Validate workflow.
- If a small change reveals duplicated definitions of the fact, report the duplication instead of silently expanding scope. Consolidate only when the task authorizes it.

## Example

Illustrative example: three consumers independently decide whether an order qualifies for free shipping.

```js
// cart.js, checkout.js, and api/order.js
const freeShipping = subtotal >= 1000
```

Consolidate the rule into one canonical function:

```js
// domain/shipping.js
const FREE_SHIPPING_MINIMUM = 1000
export const qualifiesForFreeShipping = (subtotal) =>
  subtotal >= FREE_SHIPPING_MINIMUM
```

All consumers call `qualifiesForFreeShipping()`. UI labels, fee presentation, and API response shapes may differ, but consumers must not re-derive the eligibility rule.

Do not merge two `date + N days` functions merely because their code is similar. A payment due date and a contract end date are different business facts when their meanings, ownership, or boundaries differ.

## Discover

Use layered discovery:

1. Use `rg` to find names, synonyms, constants, status values, magic values, inline conditions, types, schemas, and cross-repo copies.
2. Search consumer boundaries, including imports, exports, re-exports, direct calls, string dispatch, configuration, rules, endpoint names, generated code, and cross-language contracts. Record the methods used, scope covered, and known blind spots.
3. When available and supported for the project language, use `jscpd` to find identical or near-identical code and `ast-grep` to find structurally equivalent code. Fall back to `rg` and manual inspection when they are unavailable or unsupported.
4. Inspect every candidate semantically. Tool matches are evidence of duplication, not proof that two implementations represent the same business fact.

## Before Editing

For code involving business facts:

1. Record the business fact, canonical name and file, duplicate definitions, consumers found through the recorded search coverage, known blind spots, implementation differences, expected behavior, test baseline, planned removals, public or persisted identifiers, required compatibility names, and authorized behavior or identifier changes.
2. Count the definitions, consumers, old names, and detected clone clusters before editing so the result can be compared after consolidation.
3. If existing implementations disagree, stop and report the differences. Do not guess which behavior is correct.
4. If no canonical source exists, place it in the lowest stable shared layer that owns the business meaning.
5. Separate business meaning from presentation, transport, caching, and platform adapters.
6. Identify every known mirror repository or generated artifact and verify that it is readable. Record inaccessible mirrors before editing.

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

## Allowed Differences

Allow different UI labels, colors, layouts, platform adapters, and transport wrappers when they only present or transport the canonical result.

Exclude pure layout helpers and helpers without business meaning from this rule.

## Completion Gate

Do not claim completion merely because a shared helper exists.

Require all of the following:

- one canonical implementation;
- consumers found through the recorded search coverage migrated, with known blind spots disclosed;
- obsolete definitions removed and retained compatibility aliases delegating to the canonical source;
- intentional differences preserved and documented;
- available tests and builds passing, with missing test infrastructure reported;
- old-name and drift checks executed for accessible scope, with inaccessible mirrors reported; guards extended where infrastructure exists, or recommended in the report.
