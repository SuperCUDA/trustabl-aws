# Vendored rules (reference)

This directory holds a **vendored reference copy** of detection rules authored
for the Trustabl scanner. It exists so the rules that back this plugin's
findings can be read in-repo. It is **not** what the scanner loads at runtime.

## What's here

`openai_sdk/` — the six OpenAI Agents SDK rules added in the companion change
[`SuperCUDA/trustabl-rules#1`](https://github.com/SuperCUDA/trustabl-rules/pull/1):

| Rule | File | Sev | Area |
|---|---|---|---|
| OAI-025 | `openai_sdk/error_handling.yaml` | medium | Privileged op (shell/code/write) with no try/except |
| OAI-026 | `openai_sdk/idempotency.yaml` | medium | Exact-named mutation verb with no idempotency key |
| OAI-027 | `openai_sdk/approvals.yaml` | high | Dynamic-URL tool with no `needs_approval` gate |
| OAI-028 | `openai_sdk/shell_safety.yaml` | high | TypeScript tool spawns a subprocess |
| OAI-029 | `openai_sdk/network.yaml` | high | aiohttp write verb with no timeout |
| OAI-203 | `openai_sdk/tracing.yaml` | medium | Default tracing + no AGENTS.md/CLAUDE.md |

Each file carries **only the newly-added rule(s)** under the canonical policy
header, not the full upstream pack. The rationale docs for the MCP pack live in
[`docs/rules/mcp/`](../docs/rules/mcp/); example agents that exercise rules and
surface uncaught gaps live in [`examples/agent-templates/`](../examples/agent-templates/).

## Important: these files are a reference, not the runtime source

`scan/trustabl-scan.sh` downloads the `trustabl` engine, which resolves rule
packs from the **canonical** [`trustabl/trustabl-rules`](https://github.com/trustabl/trustabl-rules)
repository (or whatever `RULES_REPO` / `RULES_REF` point at) at scan time. It
does **not** read this directory. Copies here can drift from canonical; treat
`trustabl-rules` as the source of truth.

## If you want the scanner to actually use these

Point the scan at a rules source that contains them:

```bash
# Use the fork/branch these rules were authored on:
RULES_REPO=https://github.com/SuperCUDA/trustabl-rules \
RULES_REF=feat/openai-sdk-reliability-rules \
bash scan/trustabl-scan.sh
```

(The full, loadable pack — with every other rule and the `manifest.yaml` schema
gate — lives in that repo. A partial copy like this directory is not a complete
pack and is not meant to be loaded on its own.)

## Validating a copy

If you edit these, validate them against an engine build:

```bash
trustabl rules validate rules
```

Note that strict validation expects a complete pack layout (a top-level
`manifest.yaml` and non-duplicated rule IDs), so validate the canonical
`trustabl-rules` checkout rather than this partial reference set.
