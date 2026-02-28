# lean-context

A Claude Code plugin that distills verbose project docs into terse, agent-optimized context files.

## Why

Research from ["Evaluating AGENTS.md"](https://arxiv.org/abs/2602.11988) (ICSE 2026) found that verbose, LLM-generated context files reduce agent success by ~3% and increase costs by 20%. The only content that actually helps is non-discoverable essentials: commands with non-obvious flags, caveats, hidden constraints, environment requirements, and implicit dependencies. Everything else (architecture overviews, code structure, API docs) is noise that agents find faster by reading source code.

## What it does

lean-context scans your project's documentation sources (README, CONTRIBUTING, docs/, package.json, Makefile, docker-compose, .env.example, etc.) and produces a single terse file at `.claude/docs/lean-context.md` containing only what an agent cannot discover on its own. It also writes a manifest with source file hashes so subsequent runs skip regeneration when nothing has changed.

Every piece of information is tested against two questions:

1. Could an agent discover this by reading source code? If yes, it's discarded.
2. Would not knowing this cause the agent to fail or waste time? If yes, it's extracted.

Both conditions must be met. "Nice to know" doesn't qualify.

### Output categories

- **Commands** - invocations with non-obvious flags or required ordering
- **Caveats** - surprising behavior, hidden ordering dependencies, silent failure modes
- **Deprecated Patterns** - existing code that works but should not be replicated
- **Environment** - version requirements, env vars, services that must be running
- **Dependencies** - system tools and cross-service contracts not captured in manifests

Categories with zero items are omitted. The output is one dash-prefixed line per item, no prose.

## Install

Add the marketplace:

```
/plugin marketplace add asarnaout/lean-context
```

Install the plugin:

```
/plugin install lean-context@lean-context
```

## Usage

The plugin activates automatically when Claude detects a project with verbose documentation. You can also invoke it directly:

```
/lean-context
```

## License

MIT
