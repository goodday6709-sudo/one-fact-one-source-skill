# One Fact, One Source

An agent-neutral coding skill for consolidating business rules into one canonical source and one canonical name.

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

## Usage

Ask the coding agent to consolidate or review a business rule, or invoke the skill explicitly when the agent supports named skill invocation:

```text
Use $one-fact-one-source to consolidate this eligibility rule without changing behavior.
```

The skill uses `rg` for baseline discovery. `jscpd` and `ast-grep` are optional enhancements, not hard dependencies.

## License

MIT
