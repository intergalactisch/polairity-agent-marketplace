# Polairity Agent Marketplace

Polairity helps you remember how an AI tool actually felt in use.

This repository is the public plugin shell for Claude Code, Codex, and OpenCode.
It contains only manifests, skills, commands, and public setup copy. It does not
contain tokens, private user config, server code, or private runtime behavior.

## Recommended Setup

Run the installer from a terminal:

```bash
npx @polairity/install@latest
```

The installer asks for your Polairity plugin token when needed and configures
the MCP connection for the agents you choose. The skills in this marketplace
expect that MCP connection to exist.

For token-free agent config, choose env-token mode in the installer. The
installer will show the exact shell command for the variable name you choose.

## What It Saves

- Provider, model, and harness.
- A 0-100 score.
- Optional non-sensitive dimensions, flags, use case, and short note.
- Compact prior ratings for prefill/display, without notes or private context.

Private signal entries stay private. Publishing is a separate action on Polairity.

## Privacy Rules

The plugin must not upload code, prompts, logs, files, transcripts, stack traces,
telemetry dumps, secrets, file paths, repo names, project links, or proprietary
output.

Do not paste a token into chat. Use the installer from your terminal.
