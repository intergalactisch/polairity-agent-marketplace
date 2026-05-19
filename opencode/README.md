# Polairity For OpenCode

Use the installer to configure the Polairity MCP connection:

```bash
npx @polairity/install@latest
```

Then run:

```text
/polairity-signal
```

The installer also writes the `polairity-signal` skill and allows it in
OpenCode's skill permissions. The command saves only a private high-level signal
entry and can display compact prior ratings. It must not upload code, prompts,
logs, files, transcripts, stack traces, telemetry dumps, secrets, file paths,
repo names, project links, or proprietary output.
