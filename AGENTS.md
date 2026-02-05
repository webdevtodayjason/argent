# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Argent

**Argent** is a fork of OpenClaw being transformed into an always-on personal AI agent with:
- Hybrid model routing (local Llama for simple tasks, Claude for complex)
- Persistent memory (Memo - SQLite + FTS5)
- Automatic backup (Phoenix)
- Task tracking with accountability
- Dashboard UI with Live2D avatar

**Status**: Migration in progress. See `ARGENT_ARCHITECTURE.md` for the full vision.

## Repository

- **Original**: https://github.com/openclaw/openclaw
- **Naming**: Use **Argent** for the new project; `argent` for CLI/binary

## Build, Test, and Development

```bash
# Install dependencies
pnpm install

# Type-check and build
pnpm build

# Lint and format
pnpm check

# Run tests (Vitest)
pnpm test

# Run CLI in dev mode
pnpm openclaw <command>   # Still uses openclaw until renamed
```

### Runtime Requirements

- Node **22+** required
- Prefer Bun for TypeScript execution: `bun <file.ts>`
- Pre-commit hooks: `prek install`

## Architecture Overview

### Current (OpenClaw)
```
src/
├── cli/          # CLI wiring
├── commands/     # CLI commands
├── gateway/      # WebSocket control plane
├── agents/       # Pi agent runtime
├── channels/     # telegram, discord, slack, signal, whatsapp
├── config/       # Configuration
└── plugins/      # Plugin system
```

### Target (Argent)
```
src/
├── core/         # NEW: Always-on loop, state machine
│   ├── loop.ts
│   ├── router.ts
│   └── scheduler.ts
├── memory/       # FROM openclaw-mem (Memo)
├── tasks/        # NEW: Task queue with accountability
├── backup/       # FROM openclaw-self-backup (Phoenix)
├── models/       # NEW: Model routing (local + frontier)
├── gateway/      # FROM OpenClaw
├── channels/     # FROM OpenClaw
└── agents/       # FROM OpenClaw (modified)

dashboard/        # FROM argent-dashboard
maintenance/      # OpenClaw original dashboard (temp)
```

## Related Projects (To Integrate)

| Project | Location | Purpose |
|---------|----------|---------|
| openclaw-mem | `../openclaw-mem/` | Persistent memory (Memo) |
| openclaw-self-backup | `../openclaw-self-backup/` | Backup system (Phoenix) |
| argent-dashboard | `/Users/sem/argent/dashboard/` | React dashboard with Live2D |

## Key New Concepts

### Always-On Loop
See `src/core/loop.ts.sketch` and `ARGENT_ARCHITECTURE.md`
- Event queue with priority (urgent > high > normal > low > background)
- State machine: IDLE → PROCESSING → WAITING_TOOL → RESPONDING
- Heartbeat integration with task accountability

### Model Router
Route tasks to appropriate model tier:
- **Local** (score < 0.3): Llama via Ollama — free
- **Fast** (0.3-0.5): Claude Haiku — cheap
- **Balanced** (0.5-0.8): Claude Sonnet — standard
- **Powerful** (> 0.8): Claude Opus — expensive

### Task System
```typescript
interface Task {
  id: string;
  title: string;
  status: 'pending' | 'in_progress' | 'blocked' | 'completed' | 'failed';
  priority: 'urgent' | 'high' | 'normal' | 'low' | 'background';
  source: 'user' | 'agent' | 'heartbeat' | 'schedule';
  // ... accountability fields
}
```

Agent tools: `tasks_list`, `tasks_add`, `tasks_start`, `tasks_complete`, `tasks_block`

## Migration Checklist

- [ ] Phase 1: Fork & Restructure
  - [ ] Wipe `.git`, reinitialize
  - [ ] Rename package to `argent`
  - [ ] Update CLAUDE.md (this file)

- [ ] Phase 2: Integrate Memory (Memo)
  - [ ] Copy `../openclaw-mem/src/` → `src/memory/`
  - [ ] Convert plugin to core module
  - [ ] Update imports

- [ ] Phase 3: Integrate Backup (Phoenix)
  - [ ] Copy backup scripts → `src/backup/`
  - [ ] Create TypeScript API
  - [ ] Add CLI commands

- [ ] Phase 4: Integrate Dashboard
  - [ ] Copy `/Users/sem/argent/dashboard/` → `dashboard/`
  - [ ] Update gateway connection
  - [ ] Integrate task system

- [ ] Phase 5: Task System
  - [ ] Create `src/tasks/`
  - [ ] Implement task tools
  - [ ] Integrate with heartbeat

- [ ] Phase 6: Model Router
  - [ ] Create `src/models/`
  - [ ] Add Ollama provider
  - [ ] Implement complexity scoring

- [ ] Phase 7: Always-On Loop
  - [ ] Create `src/core/loop.ts`
  - [ ] Implement event queue
  - [ ] Add state machine

## Coding Style

- TypeScript (ESM) with strict typing; avoid `any`
- Oxlint and Oxfmt for linting/formatting
- Keep files under ~500-700 LOC
- CLI progress: use `src/cli/progress.ts`
- Terminal tables: use `src/terminal/table.ts`

## Commit Guidelines

- Create commits with `scripts/committer "<msg>" <file...>`
- Concise, action-oriented messages
- Changelog: keep latest version at top

## Hardware Context

The owner runs enterprise-grade infrastructure:
- 2x NVIDIA DGX Spark (can run Llama 405B locally)
- Dell R750: 2TB RAM, 72 cores
- Multiple Mac Minis and Mac Studios running agents
- 400 Gbps network backbone

Assume no resource limitations when suggesting solutions.

## Key Files

- `ARGENT_ARCHITECTURE.md` — Full vision and architecture
- `src/core/loop.ts.sketch` — Always-on loop implementation sketch
- `../openclaw-mem/` — Memory system to integrate
- `../openclaw-self-backup/` — Backup system to integrate
- `/Users/sem/argent/dashboard/` — Dashboard to integrate
