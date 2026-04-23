# robonet-forge

Specification-driven AI super-agent for autonomous software development.

Forge reads structured use case specifications and produces tested, reviewed code — automatically. No human in the loop between "here's a spec" and "here's a tested PR."

## How It Works

```
Use Case Specification (preconditions, flow, postconditions, error handling)
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Forgemaster (Opus)                                  │
│  Reads the UC. Decomposes into thin task envelopes   │
│  with exact file paths, method signatures, and       │
│  acceptance criteria derived from postconditions.     │
└──────────────────┬───────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────┐
│  Monitor                                             │
│  Dependency-aware dispatch. Tasks execute only when   │
│  predecessors complete. Tracks token budgets, worker  │
│  allocation, file-level conflict detection.           │
└──────────────────┬───────────────────────────────────┘
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       ┌──────┐ ┌──────┐ ┌──────┐
       │Striker│ │Striker│ │Striker│
       │  (S)  │ │  (S)  │ │  (S)  │
       └──┬───┘ └──┬───┘ └──┬───┘
          │        │        │
          ▼        ▼        ▼
       worktree  worktree  worktree
       (isolated git branches)
          │        │        │
          ▼        ▼        ▼
┌──────────────────────────────────────────────────────┐
│  Review Gate                                         │
│  Automated verification against UC postconditions.    │
│  Pass → merge. Fail → recycle.                       │
└──────────────────────────────────────────────────────┘
```

**Forgemaster** (Claude Opus) — reads the use case, understands the architecture, decomposes into dependency-ordered task envelopes. Thinks like an architect.

**Monitor** — the orchestration engine. Scans for tasks, resolves file conflicts, manages worktrees, dispatches workers, tracks lifecycle state. Self-healing: stale lock recovery, retry tracking, degraded mode.

**Striker** (Claude Sonnet) — executes in a fully isolated git worktree. Writes code, runs tests, validates against postconditions. One striker per task. No cross-contamination between concurrent tasks.

**Review Gate** — automated verification. Output is checked against the original specification's postconditions. If they fail, the task recycles with failure context. If they pass, it merges and pushes.

## What Makes This Different

**This is not prompt-to-code.**

The input is not "write me a function." The input is a formal use case specification with:

- **Preconditions** — what must be true before execution
- **Flow steps** — ordered sequence of operations
- **Postconditions** — verifiable outcomes (these become the acceptance criteria)
- **Error handling** — explicit failure modes and recovery
- **Dependencies** — which other UCs must exist first
- **Domain placement** — where this fits in the tiered architecture

The agent reads all of it. The task decomposition respects the dependency graph. The review checks the postconditions, not just "does it compile."

## Key Properties

- **UC-first** — use cases are the source of truth. Tasks are derived from specifications, never hand-written.
- **Isolation** — each striker works in its own git worktree. Concurrent tasks cannot interfere with each other.
- **Dependency-aware** — tasks execute only when predecessors complete successfully. The dispatch graph mirrors the UC dependency tree.
- **Self-validating** — output is verified against specification postconditions before merge.
- **Distributed** — runs on [robonet-mesh](https://github.com/xsubi/robonet-pub). Workers can dispatch across mesh nodes.
- **Self-healing** — stale lock recovery, automatic retry with failure context, degraded mode operation.
- **Zero external dependencies** — pure Python stdlib (core). No frameworks.

## Architecture

```
forge/
  config.py              — ForgeConfig dataclass, env var loading
  task_api.py            — Inbox task CRUD (thin task envelopes)
  dispatch/
    base.py              — TaskInfo, StrikerStatus, DispatchBackend ABC
    local.py             — LocalBackend — strikers as threads
    swarm.py             — SwarmBackend — distribute to mesh nodes
  monitor/
    core.py              — Monitor — scan → lock → worktree → striker cycle
    striker.py           — Striker thread — runs AI worker in worktree
    scanner.py           — Project/task discovery
    status.py            — Lifecycle state machine
    worktree.py          — Git worktree create/remove
    events.py            — EventLog ring buffer
    history.py           — Persistent completion log
    lock.py              — Stale lock detection + recovery
    notify.py            — Desktop notifications (macOS, Linux, Windows)
    web/                 — Optional web dashboard (FastAPI + WebSocket)
```

## Stats

| Metric | Value |
|--------|-------|
| Tests | 463 |
| Managed codebase | 59K LOC, 663 classes, 2,599 tests |
| Use cases managed | 201 |
| Concurrent strikers | Configurable (default 3) |
| External dependencies (core) | 0 |
| First commit | March 20, 2026 |

## The Codebase It Manages

robonet-forge was built to manage [robonet](https://github.com/xsubi/robonet-pub) — a 59K-line distributed mesh orchestration platform with 201 use cases across 22 domains. The forge reads robonet's use case library and autonomously implements, tests, and ships code changes.

The UC library covers: primitives, networking, cryptography, leader election, pub/sub, federation, cloud provisioning, hardware abstraction, media capture, terminal UI, CLI composition, storage, monitoring, and AI orchestration.

## How It Fits Together

```
UC Library (201 specifications)
    │
    ├── robonet-forge (this project)
    │   Reads UCs → decomposes → dispatches → validates → ships
    │
    ├── robonet (the mesh runtime)
    │   59K LOC codebase that forge manages
    │   Also provides the distributed infrastructure forge runs on
    │
    ├── robonet-foundry (CI/CD)
    │   Continuous integration pipeline consuming forge output
    │
    └── robonet-dashboard (React SPA)
        Real-time visualization of mesh + forge state
```

## Author

Joel Johnston — [github.com/xsubi](https://github.com/xsubi)

Solo build. Architecture, design, implementation, testing, deployment, and operations.

## License

All rights reserved. Published for prior art disclosure and architectural review.

&copy; 2026 Joel Johnston / xsubi
