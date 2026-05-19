---
name: polairity-signal
description: Save a private Polairity signal entry from the current OpenCode session without uploading private work data.
license: MIT
compatibility: opencode
metadata:
  author: Polairity
---

# Polairity Signal

Save a private Polairity signal for the current OpenCode session via the configured MCP server.

Canonical user-facing copy:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.
```

## Privacy

The canonical copy above is the full list of what the plugin saves and what it never sends. Additionally, do not inspect local files for this skill.

## Flow

Follow the same flow as the `/polairity-signal` command: prepare once with `integration: "opencode"` and best-effort `detected_stack` hints, show the canonical privacy copy and the detected stack status, ask the user to confirm or override provider/model/harness using slugs from the prepared `catalog`, collect the rating (required overall score 0–100, use case, optional flags, optional dimensions, optional 500-char note), get explicit confirmation, and only then call `polairity_submit_signal`. Never invent, infer, silently reuse, or pre-fill a score.

## Respond

- Success: `Saved. (entry_id: <uuid>)`
- `auth_required`: `Polairity is not connected yet. Run \`npx @polairity/install@latest\` in your terminal, then restart or reconnect OpenCode and try again.`
- Other error: `Polairity submit failed: <server message>`

Never ask the user to paste a token into chat. Do not retry automatically.
