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

## Example

Ran `lean-context` against a .NET e-commerce project (that specializes in selling electronics) with 8 documentation files totaling 1,263 lines (README, CONTRIBUTING, docker-compose, .env.example, and 4 docs/ files covering architecture, setup, API reference, and deployment).

lean-context stripped all the architecture diagrams, API endpoint tables, getting started walkthroughs, and code structure docs. What survived:

```markdown
## Commands
- `dotnet build VoltCart.sln /p:TreatWarningsAsErrors=true /p:EnforceCodeStyleInBuild=true`: CI build flags; must pass locally before PR
- `pwsh scripts/create-db-roles.ps1`: must run before first migration to create the `voltcart_app` role
- `dotnet run --project src/VoltCart.Workers -- reindex --entity products`: rebuild Elasticsearch search index (~15 min for full catalog)
- `wsl -d docker-desktop sysctl -w vm.max_map_count=262144`: required for Elasticsearch on Windows/WSL, resets on every reboot

## Caveats
- Redis uses port 6380 (non-standard) to avoid conflicts with local Redis installations
- RabbitMQ uses port 5673 (non-standard); integration tests require `docker-compose.override.yml` to be loaded
- Hot reload (`dotnet watch`) only works for the Api project, not the Workers project; restart Workers manually after changes
- Seed script (`seed-catalog.ps1`) blocks waiting for Elasticsearch health; times out after 120s if ES is still initializing
- Never modify an existing migration merged to `develop`; always create a new migration instead
- Cross-context DB references use IDs only, no foreign keys across schemas; do not add cross-schema FK constraints
- Production migrations may lock tables; run during maintenance windows
- Not all EF Core migrations are reversible; check the `Down()` method before attempting rollback

## Environment
- Docker Desktop needs at least 4 GB memory allocated (6 GB recommended) to run all infrastructure services
- Node.js 20+ required for admin dashboard frontend (not obvious from .NET solution)
- PowerShell Core 7+ (`pwsh`) required for helper scripts (`seed-catalog.ps1`, `create-db-roles.ps1`)
- Elasticsearch `vm.max_map_count=262144` must be set in WSL; resets on Windows reboot; add to `/etc/sysctl.conf` inside WSL2 distro for permanence
```

1,263 lines in. 23 lines out. Everything an agent needs, nothing it doesn't.

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
