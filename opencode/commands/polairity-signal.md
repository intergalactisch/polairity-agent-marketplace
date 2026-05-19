---
description: Save a private Polairity signal entry
---

Save a private Polairity signal entry for the current OpenCode AI tool experience.

Command arguments: `$ARGUMENTS`

Use the configured Polairity MCP server.

Canonical user-facing copy:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.
```

## Privacy

The canonical copy above is the full list of what the plugin saves and what it never sends. Additionally, do not inspect local files for this command.

## Flow

1. Call `polairity_prepare_signal` once with:

   ```json
   {
     "integration": "opencode",
     "detected_stack": {
       "provider_id": "<best known provider slug>",
       "model_id": "<best known model slug>",
       "harness_id": "opencode",
       "raw_model_name": "<runtime model name if available>"
     }
   }
   ```

2. Show the canonical privacy copy, detected stack statuses, and any compact prior rating from `prefill.latest_for_detected_stack`.

3. Ask one combined prompt:

   ```text
   Reply with:
   1. Stack: "keep" or provider/model/harness slugs.
   2. Overall score: 0-100.
   3. Use case: one prepared use_case key, or a short label I can map.
   4. Optional flags: prepared flag keys, or "skip".
   5. Optional metric ratings: intelligence, reliability, speed, value, usage_intensity, design_sense, each 0-100 or "skip".
   6. Optional note: max 500 chars, no code/logs/prompts/file paths/repo names/project specifics, or "skip".
   ```

4. Accept only provider/model/harness slugs from the prepared `catalog`. If the user chooses a different stack, look only in `recent_entries` for a compact prior rating with that exact stack.

5. Require an explicit overall score. Never invent, infer, silently reuse, or pre-fill a score as the user's answer.

6. Before saving, summarize the structured fields and ask the user to confirm.

7. Call `polairity_submit_signal` only after confirmation. Payload:

   - `client_run_id`: fresh UUID v4.
   - `stack`: confirmed provider/model/harness slugs.
   - `score`: required overall score.
   - optional metric fields from the user.
   - `flags`, `use_case`, `note` from the user.
   - `completed_at`: current UTC ISO 8601 timestamp.
   - `run_metadata.integration`: `opencode`.
   - `run_metadata.model_id_hint`: runtime model name or slug if available.

## Respond

- Success: `Saved. (entry_id: <uuid>)`
- `auth_required`: `Polairity is not connected yet. Run \`npx @polairity/install@latest\` in your terminal, then restart or reconnect OpenCode and try again.`
- Other error: `Polairity submit failed: <server message>`

Never ask the user to paste a token into chat. Do not retry automatically.
