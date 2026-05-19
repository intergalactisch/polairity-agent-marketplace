---
name: signal
description: Save a private Polairity signal entry from the active Codex session without uploading private work data.
---

# Polairity Signal

Save a private Polairity signal for the current Codex session via the configured Polairity MCP server.

Canonical user-facing copy:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.
```

## Privacy

The canonical copy above is the full list of what the plugin saves and what it never sends. Additionally, do not inspect local files for this skill.

## Prepare

Call `polairity_prepare_signal` once before asking rating questions.

Use these best-effort hints:

```json
{
  "integration": "codex",
  "detected_stack": {
    "provider_id": "openai",
    "model_id": "<best known model slug>",
    "harness_id": "codex",
    "raw_model_name": "<runtime model name if available>"
  }
}
```

Treat `detected_stack` in the response as server-verified guidance only. The user still chooses the final stack. Use `catalog`, `form_options`, `detected_stack.fallback_options`, `prefill.latest_for_detected_stack`, and `recent_entries` from the prepare response; do not use any other discovery call.

If prepare returns `auth_required`, tell the user:

```text
Polairity is not connected yet. Run `npx @polairity/install@latest` in your terminal, then restart or reconnect Codex and try again.
```

## Ask

Ask one concise combined prose questionnaire:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.

Detected stack:
- Provider: <name or status> (<slug if matched>)
- Model: <name or status> (<slug if matched>)
- Harness: <name or status> (<slug if matched>)

<If available: Previous private rating for this stack: <overall>/100 on <date>.>

Reply with:
1. Stack: "keep" or provider/model/harness slugs.
2. Overall score: 0-100.
3. Use case: one of the prepared use_case keys, or a short label I can map.
4. Optional flags: any prepared flag keys, or "skip".
5. Optional metric ratings: intelligence, reliability, speed, value, usage_intensity, design_sense, each 0-100 or "skip".
6. Optional note: max 500 chars, no code/logs/prompts/file paths/repo names/project specifics, or "skip".
```

Use prepared fallback options in the prompt when helpful, but keep it compact. Accept only provider, model, and harness slugs that appear in the prepared `catalog`. If the user chooses a different stack, look only in `recent_entries` for a compact prior rating with the same provider/model/harness and show it as context. Prior ratings are never answers. Require an explicit overall score — never invent, infer, or silently reuse one. Use all six optional metric fields (`intelligence`, `reliability`, `speed`, `value`, `usage_intensity`, `design_sense`) only when the user supplies them.

## Submit

Before saving, summarize the structured fields and ask the user to confirm. Omit any optional field the user skipped, deduplicate flags and cap at 8, trim the note to 500 characters, and drop the note entirely if it leaks code, logs, prompts, transcripts, file paths, repo names, project links, secrets, or proprietary specifics.

Call `polairity_submit_signal` only after confirmation. Payload:

- `client_run_id`: fresh UUID v4.
- `stack`: confirmed provider/model/harness slugs.
- `score`: required overall score.
- optional metric fields from the user.
- `flags`, `use_case`, `note` from the user.
- `completed_at`: current UTC ISO 8601 timestamp.
- `run_metadata.integration`: `codex`.
- `run_metadata.model_id_hint`: runtime model name or slug if available.

## Respond

- Success: `Saved. (entry_id: <uuid>)`
- `auth_required`: use the install message from Prepare.
- Other error: `Polairity submit failed: <server message>`

Never ask the user to paste a token into chat. Do not retry automatically.
