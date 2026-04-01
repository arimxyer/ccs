# Agent Teams vs. Individual Agent Calls

## Overview

Claude Code supports two modes for running agents: **individual agent calls** (always available) and **agent teams** (opt-in, feature-gated). This document summarizes how each works and the key differences between them.

---

## Individual Agent Calls

When `AgentTool` is invoked without a `name` + `team_name`, it runs a standard isolated agent:

- Registered as a `LocalAgentTaskState` (or `RemoteAgentTask` for cloud) in `AppState`
- Executes via `runAgent()` in the current process's event loop
- Identity is a random, generic ID (e.g., `a1b2c3d4`)
- No cross-agent coordination — fully isolated from other agents
- `run_in_background=true` makes it fully async, with a UI notification on completion

**Relevant files:**
- `restored-src/src/tools/AgentTool/AgentTool.tsx`
- `restored-src/src/tasks/LocalAgentTask/LocalAgentTask.tsx`

---

## Agent Teams (Teammates)

When `AgentTool` is called with both `name=` and `team_name=`, it enters the teammate spawning path. This requires the `tengu_amber_flint` feature flag (`--agent-teams` CLI flag or `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var).

### Identity
- Deterministic: `name@teamName` (e.g., `researcher@data-team`)
- Persisted in a team JSON file at `~/.config/claude-code/teams/{teamName}.json`

### Spawning Backends
Chosen automatically at runtime:

| Backend | Description |
|---|---|
| **In-process** | Async context within the same process — faster, shares `AppState` directly |
| **Pane-based (tmux/iTerm2)** | Independent Claude Code CLI process in a new tmux window |
| **CCR (cloud)** | Remote cloud-based execution |

### Communication
- Leader sends the initial prompt (and subsequent messages) via a **mailbox** (`~/.config/claude-code/mailboxes/`), not CLI args
- Teammates poll their inbox for messages
- `SendMessageTool` can address teammates **by name**, enabling peer-to-peer async communication

### Constraints
- Teammates **cannot** spawn other teammates
- In-process teammates **cannot** spawn background agents
- Leader's permission mode is propagated to all teammates

### Shared Memory
A `teamMemorySync` service keeps memory context synchronized across the whole team.

**Relevant files:**
- `restored-src/src/tools/AgentTool/AgentTool.tsx`
- `restored-src/src/tools/shared/spawnMultiAgent.ts`
- `restored-src/src/tools/TeamCreateTool/TeamCreateTool.ts`
- `restored-src/src/tasks/InProcessTeammateTask/types.ts`
- `restored-src/src/utils/swarm/spawnInProcess.ts`
- `restored-src/src/utils/swarm/inProcessRunner.ts`
- `restored-src/src/utils/teammateMailbox.ts`
- `restored-src/src/utils/agentSwarmsEnabled.ts`

---

## Comparison

| Aspect | Individual Agent | Agent Team Member |
|---|---|---|
| **Execution** | In-process event loop | Separate process or async in-process |
| **Identity** | Random ID | `name@teamName` (deterministic) |
| **Communication** | None (isolated) | Mailbox + `SendMessageTool` |
| **Coordination** | None | Team file + shared memory sync |
| **Spawn constraint** | None | Cannot spawn sub-teammates |
| **Backend** | Single (local or remote) | In-process, tmux, iTerm2, or CCR |
| **Feature gate** | Always available | `tengu_amber_flint` flag required |
| **Permission propagation** | Inherits from caller | Propagated from team leader |

---

## Processing Flow

### Individual Agent
```
AgentTool.call()
  → registerAsyncAgent()
  → LocalAgentTaskState created
  → runAgent() executes in event loop
  → Task completes, UI notified
```

### Team Member
```
AgentTool.call(name=X, team_name=Y)
  → spawnTeammate()
    → isInProcessEnabled() ? handleSpawnInProcess() : handleSpawnSplitPane()
    → InProcessTeammateTaskState created (in-process) OR new CLI process (pane)
    → Mailbox message sent with initial prompt
    → Teammate polls inbox, executes via runAgent()
    → Results streamed back via mailbox / AppState
    → Task notified when idle
```
