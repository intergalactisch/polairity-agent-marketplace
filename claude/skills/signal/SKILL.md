---
name: signal
description: Write a private Polairity signal entry without uploading private work data.
model: inherit
disable-model-invocation: true
allowed-tools: 'AskUserQuestion mcp__polairity__polairity_prepare_signal mcp__polairity__polairity_submit_signal'
---

# Polairity Signal

Save a private Polairity signal for the current Claude Code session via the configured MCP server.

Canonical user-facing copy:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.
```

## Privacy

The canonical copy above is the full list of what the plugin saves and what it never sends. Additionally, do not inspect local files for this skill.

## Prepare

Call `mcp__polairity__polairity_prepare_signal` once before asking rating questions.

Use these best-effort hints:

```json
{
  "integration": "claude_code",
  "detected_stack": {
    "provider_id": "anthropic",
    "model_id": "<best known Claude model slug>",
    "harness_id": "claude_code",
    "raw_model_name": "<runtime model name if available>"
  }
}
```

Treat `detected_stack` in the response as server-verified guidance only. The user still chooses the final stack. Use `catalog`, `form_options`, `detected_stack.fallback_options`, `prefill.latest_for_detected_stack`, and `recent_entries` from the prepare response; do not use any other discovery call.

If prepare returns `auth_required`, tell the user:

```text
Polairity is not connected yet. Run `npx @polairity/install@latest` in your terminal, then restart or reconnect Claude Code and try again.
```

## Ask

Use `AskUserQuestion` for closed-choice fields only. `AskUserQuestion` requires 2–4 predefined options per question — it cannot collect open numeric input like a 0–100 score. Ask every open-numeric field (the required overall score and each of the six dimension ratings) as a plain chat message.

First, show the canonical privacy copy, the detected stack statuses, and any compact prior rating from `prefill.latest_for_detected_stack`.

Ask a stack batch with provider, model, and harness via `AskUserQuestion`:

- Put the matched detected option first and mark it `(Recommended)` when status is `matched`.
- Add fallback options from `detected_stack.fallback_options`.
- Accept only provider, model, and harness slugs that appear in the prepared `catalog` — let the user type a slug only when it is one of those.

If the user chooses a different stack, look only in `recent_entries` for a compact prior rating with the same provider/model/harness and show it as context. Prior ratings are never answers.

Ask the overall score in chat as a plain message: "What's your overall score for this run (0–100)?" Parse the reply, clamp to 0–100, re-ask once on non-numeric input. Never use `AskUserQuestion` for the score. Never invent, infer, or silently reuse a score.

Ask the remaining closed-choice fields via `AskUserQuestion` (one batch):

- Use case from `form_options.use_cases`.
- Optional behavior flags from `form_options.flags`.
- Whether to rate the six dimensions (yes/no).

If the user opts to rate dimensions, ask them in chat: "Rate intelligence, reliability, speed, value, usage_intensity, and design_sense, each 0–100 or 'skip'." Parse the reply; omit any field the user skipped.

Optional note (ask in chat): max 500 chars, no code/logs/prompts/file paths/repo names/project specifics.

## Submit

Before saving, summarize the structured fields and ask the user to confirm. Omit any optional field the user skipped, deduplicate flags and cap at 8, trim the note to 500 characters, and drop the note entirely if it leaks code, logs, prompts, transcripts, file paths, repo names, project links, secrets, or proprietary specifics.

Call `mcp__polairity__polairity_submit_signal` only after confirmation. Payload:

- `client_run_id`: fresh UUID v4.
- `stack`: confirmed provider/model/harness slugs.
- `score`: required overall score.
- optional metric fields from the user.
- `flags`, `use_case`, `note` from the user.
- `completed_at`: current UTC ISO 8601 timestamp.
- `run_metadata.integration`: `claude_code`.
- `run_metadata.model_id_hint`: runtime model name or slug if available.

## Respond

- Success: `Saved. (entry_id: <uuid>)`
- `auth_required`: use the install message from Prepare.
- Other error: `Polairity submit failed: <server message>`

Never ask the user to paste a token into chat. Do not retry automatically.
