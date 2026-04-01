# Context

Context is the **ephemeral, read-only session snapshot** injected into every API call. It captures the current state of the environment (git, date, CLAUDE.md files) and is recalculated each turn — nothing is persisted to disk.

---

## Two Halves of Context

### System Context — `getSystemContext()`

Environmental state appended to the **system message**.

```typescript
{
  gitStatus: string,   // current branch, diff stats, untracked files
  cacheBreaker?: string
}
```

- Memoized per session — computed once and reused.
- Gives the model awareness of the working repo state without burning user-turn tokens.

### User Context — `getUserContext()`

Session metadata prepended to the **first user message** each turn.

```typescript
{
  claudeMd: ClaudeMdFile[],   // contents of all discovered CLAUDE.md files
  currentDate: string
}
```

- Scans upward from CWD to find `CLAUDE.md` files (project-level instructions).
- Memoized per session.

**Relevant files:**
- `restored-src/src/context.ts` — `getSystemContext()`, `getUserContext()`

---

## Injection into API Calls

Both halves are loaded in parallel at the start of each query, then composed around the message list before the API call:

```
REPL.tsx (query loop)
  → Promise.all([getSystemContext(), getUserContext()])

utils/api.ts
  → appendSystemContext(systemPrompt, systemContext)
    // appends git status block to system message
  → prependUserContext(messages, userContext)
    // prepends CLAUDE.md content + date to first user message
  → sent to Claude API
```

**Relevant files:**
- `restored-src/src/utils/api.ts` — `appendSystemContext()`, `prependUserContext()`
- `restored-src/src/screens/REPL.tsx` — query loop orchestration

---

## Context Window Management

Separate utilities manage token budgets and context overflow:

- **`restored-src/src/utils/context.ts`** — token budget constants and helpers
- **`restored-src/src/utils/contextAnalysis.ts`** — analyzes current context utilization
- **`restored-src/src/utils/contextSuggestions.ts`** — suggests actions when context is near-full (e.g., `/compact`)
- **`restored-src/src/utils/analyzeContext.ts`** — lower-level analysis helpers

The `/context` slash command lets users inspect current context usage interactively:
- `restored-src/src/commands/context/context.tsx`
- `restored-src/src/commands/context/context-noninteractive.ts`

---

## Note on React Context

`/src/context/` contains React context providers for **UI state** (modals, notifications, voice, overlays). These are unrelated to the LLM conversation context described above.

---

## Key Characteristics

| Property | Value |
|---|---|
| **Persistence** | None — recalculated each turn |
| **Mutability** | Read-only (no agent writes) |
| **Scope** | Per-session |
| **Injection point** | System message (git status) + first user message (CLAUDE.md, date) |
| **Caching** | Memoized within a session |
| **CLAUDE.md** | Project-level instructions; read by context, NOT written by memory |
