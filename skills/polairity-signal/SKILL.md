---
name: polairity-signal
description: Save a private Polairity signal entry from a high-level AI tool assessment without uploading private work data.
license: MIT
compatibility: Claude Code, Codex, and OpenCode.
allowed-tools: 'AskUserQuestion mcp__polairity__polairity_prepare_signal mcp__polairity__polairity_submit_signal'
metadata:
  author: Polairity
---

# Polairity Signal

Save a private Polairity signal for the current AI tool session via the configured MCP server.

Canonical user-facing copy:

```text
I'll save a private Polairity signal for this AI tool run. I'll only send structured rating fields: stack, score, optional dimensions, flags, use case, and a short note. I won't send code, prompts, logs, transcripts, file paths, repo names, secrets, or proprietary output.
```

## Privacy

The canonical copy above is the full list of what the plugin saves and what it never sends. Additionally, do not inspect local files for this skill.

## Prepare

Call the Polairity prepare tool exactly once before asking rating questions:

- Claude Code tool name: `mcp__polairity__polairity_prepare_signal`
- Other MCP tool name: `polairity_prepare_signal`

Arguments:

- `integration`: `claude_code`, `codex`, or `opencode`
- `detected_stack.provider_id`: best known provider slug
- `detected_stack.model_id`: best known model slug
- `detected_stack.harness_id`: `claude_code`, `codex`, or `opencode`
- `detected_stack.raw_model_name`: runtime model name if available

Use only the prepare response for catalog options, form options, detected-stack status, compact prior ratings, and fallback choices. The detected stack is guidance only; the user still confirms or overrides the final stack.

If prepare returns `auth_required`, tell the user:

```text
Polairity is not connected yet. Run `npx @polairity/install@latest` in your terminal, then restart or reconnect your agent and try again.
```

## Ask

Show the canonical privacy copy, detected stack statuses, and any compact prior rating from `prefill.latest_for_detected_stack`.

Confirm stack:

- Prefer a batched chooser when the host supports it.
- Put matched detected options first and mark them `(Recommended)`.
- Add fallback options from `detected_stack.fallback_options`.
- Accept only provider, model, and harness slugs from the prepared `catalog`.
- If the user chooses a different stack, look only in `recent_entries` for a compact prior rating with that exact stack.

Collect rating. Use closed-choice pickers (e.g. `AskUserQuestion`) only for fields with a fixed set of values. Open-numeric inputs cannot be collected via choice pickers (they require 2–4 predefined options) — ask them in chat as plain messages.

- Required overall score, 0-100. **Ask in chat.** Never use `AskUserQuestion`.
- Use case from `form_options.use_cases`. Closed-choice.
- Optional flags from `form_options.flags`. Closed-choice.
- Optional metric ratings for all six dimensions: `intelligence`, `reliability`, `speed`, `value`, `usage_intensity`, `design_sense`. Each is 0-100 — **ask in chat.**
- Optional note, max 500 chars, no code/logs/prompts/file paths/repo names/project specifics. Free text — ask in chat.

Never invent, infer, silently reuse, or pre-fill a score as the user's answer. Prior ratings are display context only.

## Submit

Before saving, summarize the structured fields and ask the user to confirm.

Call the submit tool only after confirmation:

- Claude Code tool name: `mcp__polairity__polairity_submit_signal`
- Other MCP tool name: `polairity_submit_signal`

Payload:

- `client_run_id`: fresh UUID v4.
- `stack`: confirmed provider/model/harness slugs.
- `score`: required overall score.
- optional metric fields from the user.
- `flags`, `use_case`, `note` from the user.
- `completed_at`: current UTC ISO 8601 timestamp.
- `run_metadata.integration`: current host integration.
- `run_metadata.model_id_hint`: runtime model name or slug if available.

## Respond

- Success: `Saved. (entry_id: <uuid>)`
- `auth_required`: use the install message from Prepare.
- Other error: `Polairity submit failed: <server message>`

Never ask the user to paste a token into chat. Do not retry automatically.
