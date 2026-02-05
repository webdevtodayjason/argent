# Argent Documentation Index

> Always-On Personal AI Agent — Forked from OpenClaw

## Architecture

| Document | Description |
|----------|-------------|
| [ARGENT_ARCHITECTURE.md](../../ARGENT_ARCHITECTURE.md) | Full vision, project structure, always-on loop, task system, model router |
| [SIS_ARCHITECTURE.md](./SIS_ARCHITECTURE.md) | Self-Improving System — lessons learned, pattern detection, feedback loops |
| [loop.ts.sketch](../../src/core/loop.ts.sketch) | Always-on loop implementation sketch |

## Integrated Systems

| System | Source | Documentation |
|--------|--------|---------------|
| **Memo** (Memory) | `openclaw-mem` | [Memory README](../../../openclaw-mem/README.md) |
| **Phoenix** (Backup) | `openclaw-self-backup` | [Backup README](../../../openclaw-self-backup/README.md) |
| **Dashboard** | `argent-dashboard` | [Dashboard Architecture](/Users/sem/argent/dashboard/ARCHITECTURE.md) |

## Migration Status

See [CLAUDE.md](../../CLAUDE.md) for the migration checklist.

| Phase | Status | Description |
|-------|--------|-------------|
| 1. Fork & Restructure | 🔄 In Progress | Wipe git, reinitialize, rename |
| 2. Integrate Memory | ⏳ Pending | Memo → `src/memory/` |
| 3. Integrate Backup | ⏳ Pending | Phoenix → `src/backup/` |
| 4. Integrate Dashboard | ⏳ Pending | Dashboard → `dashboard/` |
| 5. Task System | ⏳ Pending | New `src/tasks/` |
| 6. Model Router | ⏳ Pending | New `src/models/` |
| 7. Always-On Loop | ⏳ Pending | New `src/core/` |

## Key Concepts

### Always-On Loop
```
EVENT SOURCES          EVENT QUEUE              AGENT RUNTIME
  Channels   ─┐
  Heartbeat  ─┼──►  Priority Queue  ──►  State Machine  ──►  Model Router
  Tasks      ─┤     (urgent→low)        (idle→processing)   (local→opus)
  Calendar   ─┤                                │
  Webhooks   ─┘                                ▼
                                         Context Assembly
                                               │
                                               ▼
                                         OUTPUT HANDLERS
                                         Reply │ Task │ Memory │ Dashboard
```

### Model Routing
```
Complexity Score → Model Tier
─────────────────────────────
   < 0.3        → Local (Llama via Ollama)     FREE
   0.3 - 0.5    → Fast (Claude Haiku)          $
   0.5 - 0.8    → Balanced (Claude Sonnet)     $$
   > 0.8        → Powerful (Claude Opus)       $$$
```

### Task Lifecycle
```
CREATED → PENDING → IN_PROGRESS → COMPLETED
                         │
                         ├──→ BLOCKED (waiting on dependency)
                         └──→ FAILED (error after max attempts)
```

## Configuration

Main config: `config/argent.json`

```json
{
  "agent": { "id": "argent-main" },
  "gateway": { "port": 18789 },
  "models": { "default": "balanced", "routing": { "enabled": true } },
  "heartbeat": { "enabled": true, "interval": "30s" },
  "memory": { "enabled": true, "workerPort": 37778 },
  "backup": { "enabled": true, "schedule": "0 */6 * * *" },
  "dashboard": { "port": 8080 }
}
```

## Commands (Target)

```bash
argent start              # Start all services
argent gateway start      # Start gateway only
argent dashboard start    # Start dashboard only
argent tasks list         # List pending tasks
argent backup now         # Run backup
argent status             # Show system status
```

---

*Last updated: 2026-02-05*
