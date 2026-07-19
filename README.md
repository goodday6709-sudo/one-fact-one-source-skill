# One Fact, One Source

An agent-neutral coding skill for consolidating business rules into one canonical source and one canonical name.

It is not a programming tutorial. It is a governance and review skill that helps coding agents avoid creating a second version of an existing business rule.

It helps coding agents:

- reuse existing business-rule implementations before writing new ones;
- distinguish semantic duplication from merely similar-looking code;
- consolidate statuses, amounts, permissions, schemas, contracts, and shared constants;
- preserve behavior unless a task explicitly authorizes a change;
- migrate all known consumers and remove old parallel definitions;
- verify old names, clone counts, tests, and cross-repository drift.

## Install

Install globally for supported coding agents with the Skills CLI:

```bash
npx skills add goodday6709-sudo/one-fact-one-source-skill -g -y
```

Or copy the skill directory manually:

```text
Codex personal:       ~/.codex/skills/one-fact-one-source/
Claude Code personal: ~/.claude/skills/one-fact-one-source/
Project shared:       <project>/.agents/skills/one-fact-one-source/
```

Claude Code project installations may also use:

```text
<project>/.claude/skills/one-fact-one-source/
```

## Guides

- [繁體中文快速指南](docs/quickstart.zh-TW.md)
- [简体中文快速指南](docs/quickstart.zh-CN.md)

## Usage

The skill supports three levels of effort:

- **Audit only**: discover and report definitions, consumers, and conflicts without modifying files.
- **Small change**: locate and reuse the existing canonical source without adding parallel logic.
- **Consolidation or refactor**: migrate consumers, remove old definitions, and run the full validation workflow.

Invoke the skill with the syntax supported by the coding agent:

```text
Codex audit:             Use $one-fact-one-source to audit this rule.
Codex consolidate:       Use $one-fact-one-source to consolidate this rule.
Claude Code audit:       /one-fact-one-source audit this rule.
Claude Code consolidate: /one-fact-one-source consolidate this rule.
Other agents: Ask the agent to use the one-fact-one-source skill by name.
```

Audit first:

```text
Audit this business rule. Do not modify files.
Report its definitions, consumers, conflicts, and the recommended canonical
source when the evidence is sufficient. Include search coverage and blind spots.
```

Consolidate after review:

```text
Consolidate this business rule without changing behavior or public identifiers.
Before editing, verify a clean working tree and record the baseline commit.
Create a separate consolidation commit only when git writes are explicitly authorized.
Migrate consumers found through recorded search coverage, remove obsolete
parallel definitions, run the available checks, and report the before/after
evidence and any inaccessible mirrors.
```

The skill uses `rg` for baseline discovery. `jscpd` and `ast-grep` are optional enhancements when they support the project language, not hard dependencies.

## License

MIT
