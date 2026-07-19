---
name: one-fact-one-source
description: Use when creating, changing, reviewing, or refactoring code involving business rules, statuses, amounts, permissions, classifications, shared constants, types, schemas, contracts, duplicated logic, inline business decisions, derived caches used as truth, or cross-repo mirrors. Enforces one business fact = one canonical source = one name, with discovery, migration boundaries, tests, forbidden-old-name guards, and drift checks.
---

# One Fact, One Source

## Core Rule

Keep exactly one canonical function, constant, type, schema, contract, or authoritative output for each business fact.

Allow consumers to present or transport the result differently. Do not let them reimplement the underlying status, amount, permission, classification, or business decision.

Do not merge code merely because it looks similar. Merge only when the business meaning, inputs, boundaries, and ownership are the same. Preserve and document intentional differences.

## Scope of Effort

Match the process weight to the task:

- **Small change** (touches one existing business fact, no consolidation intended): run a targeted search to locate and reuse the canonical source, then import or call it without adding a parallel implementation. Skip repo-wide counting, clone scans, and the full consolidation report unless the search reveals conflicting or duplicated definitions.
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

## Discover

Use layered discovery:

1. Use `rg` to find names, synonyms, constants, status values, magic values, inline conditions, types, schemas, and cross-repo copies.
2. When available, use `jscpd` to find identical or near-identical code and `ast-grep` to find structurally equivalent code. Fall back to `rg` and manual inspection when they are unavailable.
3. Inspect every candidate semantically. Tool matches are evidence of duplication, not proof that two implementations represent the same business fact.

## Before Editing

For code involving business facts:

1. Record the business fact, canonical name and file, duplicate definitions, all known consumers, implementation differences, expected behavior, test baseline, planned removals, and authorized behavior changes.
2. Count the definitions, consumers, old names, and detected clone clusters before editing so the result can be compared after consolidation.
3. If existing implementations disagree, stop and report the differences. Do not guess which behavior is correct.
4. If no canonical source exists, place it in the lowest stable shared layer that owns the business meaning.
5. Separate business meaning from presentation, transport, caching, and platform adapters.

## While Editing

1. Keep one canonical implementation and export it from the owning module.
2. Change every known consumer to import, call, or read the canonical result.
3. Remove old parallel functions, constants, types, state sets, and inline decisions.
4. Keep adapters, controllers, UI wrappers, and platform wrappers limited to input/output conversion.
5. Treat caches, read models, and derived fields as projections, not sources of truth. Document their source and invalidation boundary.
6. Use a shared package, generated artifact, or drift check for cross-repo mirrors. Do not rely on manual copying alone.
7. Treat consolidation as moving without behavior change unless the task explicitly authorizes a behavior change.
8. List every authorized behavior change separately in the final report.

## Validate

Before declaring completion:

1. Add canonical unit tests for normal, boundary, fallback, and invalid cases.
2. Run affected full test suites, type checks, linters, and builds that exist in the project.
3. Use `rg` to prove old names, copied conditions, duplicated state sets, and magic values are gone.
4. Re-run any `jscpd` or `ast-grep` checks used during discovery and record the before/after clone counts.
5. If the project already has lint rules, CI checks, or contract tests for this purpose, extend them to forbid old names from returning. Otherwise, recommend adding such a guard in the final report instead of adding CI or lint infrastructure uninvited.
6. Run cross-repo drift checks when mirrors exist. If drift remains, fail closed and list the files.
7. Report the canonical fact/name/file, definition count before/after, migrated consumer count, removed old-name count, clone scan before/after, authorized behavior changes, test/check/build outputs, and unresolved drift count.

## Allowed Differences

Allow different UI labels, colors, layouts, platform adapters, and transport wrappers when they only present or transport the canonical result.

Exclude pure layout helpers and helpers without business meaning from this rule.

## Completion Gate

Do not claim completion merely because a shared helper exists.

Require all of the following:

- one canonical implementation;
- all known consumers migrated;
- old definitions removed;
- intentional differences preserved and documented;
- tests and builds passing;
- old-name and drift checks executed; guards extended where infrastructure exists, or recommended in the report.
