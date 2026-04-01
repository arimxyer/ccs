# Memory

Memory is the **persistent, writable knowledge layer** in Claude Code. It survives across sessions and can be actively managed by both the user and agents.

---

## Storage Layout

Memory lives on disk under a configurable root (default `~/.config/claude-code/memories/`).

```
memories/
  MEMORY.md            ← global memory entrypoint (200 lines / 25 KB cap)
  <topic>.md           ← topic-scoped files written by agents
  teams/
    <teamName>.json    ← team membership roster
  mailboxes/           ← teammate inbox files (cross-agent messaging)
```

**Relevant files:**
- `restored-src/src/memdir/memdir.ts` — main orchestrator
- `restored-src/src/memdir/memoryTypes.ts` — type definitions and guidance strings
- `restored-src/src/memdir/findRelevantMemories.ts` — retrieval logic
- `restored-src/src/memdir/paths.ts` — path configuration, auto-memory settings
- `restored-src/src/memdir/teamMemPaths.ts` — team memory paths

---

## Write Path

Agents write memory via `FileEditTool` / `WriteFileTool` targeting paths inside the memory root. The assistant is guided (via system prompt instructions) on when and how to create or update memory files.

Session-level memory extraction runs in the background:

```
User turn ends
  → sessionMemory service triggers extractMemories()
  → Extraction prompt sent to model
  → Resulting facts written to MEMORY.md (or topic file)
```

**Relevant files:**
- `restored-src/src/services/SessionMemory/sessionMemory.ts`
- `restored-src/src/services/SessionMemory/prompts.ts`
- `restored-src/src/services/extractMemories/`
- `restored-src/src/services/teamMemorySync/` — keeps memory in sync across all members of an agent team

---

## Read Path (injection into prompts)

Memory content is injected into the system prompt each turn as a dedicated section:

```
loadMemoryPrompt()
  → reads MEMORY.md from disk (sync)
  → truncateEntrypointContent() enforces size limits
  → returns formatted string

buildMemoryPrompt()
  → prepends behavioral guidance (how to use/update memory)
  → appends MEMORY.md content

getSystemPrompt()
  → systemPromptSection('memory', () => loadMemoryPrompt())
  → memory section included alongside other prompt sections
```

**Relevant files:**
- `restored-src/src/memdir/memdir.ts` — `loadMemoryPrompt()`, `buildMemoryPrompt()`, `truncateEntrypointContent()`
- `restored-src/src/constants/prompts.ts` — `systemPromptSection('memory', ...)`

---

## UI

- `/slash memory` command — lets users inspect and edit memory interactively
- `restored-src/src/commands/memory/memory.tsx`
- `restored-src/src/components/memory/` — React components for memory display

---

## Key Characteristics

| Property | Value |
|---|---|
| **Persistence** | Cross-session (disk files) |
| **Mutability** | Read + write (agents and users can modify) |
| **Scope** | Global, topic-scoped, or team-scoped |
| **Size cap** | 200 lines / 25 KB for `MEMORY.md` entrypoint |
| **Injection point** | System prompt (`memory` section) |
| **Team support** | `teamMemorySync` keeps team members in sync |
