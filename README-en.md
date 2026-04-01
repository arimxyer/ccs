# css

> [!WARNING]
> This repository is **unofficial** and is reconstructed from the public npm package and source map analysis, **for research purposes only**.
> It does **not** represent the original internal development repository structure.
>
> This repository is an **unofficial** reconstructed version, recovered from the public npm package and source map analysis, **for research use only**.
> It does **not** represent the official original internal development repository structure.
> All information is based on tips from L-station "Piaoran yu wo tong".

## Overview

This repository reconstructs TypeScript source code from the source map (`cli.js.map`) bundled in the npm package (`@anthropic-ai/claude-code`), version `2.1.88`.

## Source

- npm package: [@anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- Reconstructed version: `2.1.88`
- Number of reconstructed files: **4,756** (including 1,884 `.ts`/`.tsx` source files)
- Reconstruction method: extracted the `sourcesContent` field from `cli.js.map`

## Directory Structure

```text
restored-src/src/
├── main.tsx              # CLI entry point
├── tools/                # Tool implementations (Bash, FileEdit, Grep, MCP, and 30+ others)
├── commands/             # Command implementations (commit, review, config, etc., 40+)
├── services/             # Services for API, MCP, analysis, etc.
├── utils/                # Utility functions (git, model, auth, env, etc.)
├── context/              # React Context
├── coordinator/          # Multi-agent coordination mode
├── assistant/            # Assistant mode (KAIROS)
├── buddy/                # AI companion UI
├── remote/               # Remote sessions
├── plugins/              # Plugin system
├── skills/               # Skill system
├── voice/                # Voice interaction
└── vim/                  # Vim mode
```

## Declaration

- The source code copyright belongs to [Anthropic](https://www.anthropic.com)
- This repository is for technical research and learning only; please do not use it for commercial purposes
- If there is any infringement, please contact us for removal.