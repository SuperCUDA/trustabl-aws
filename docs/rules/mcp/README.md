# MCP rule rationale — the 14 shipped Model Context Protocol rules

This directory documents the **threat model and rationale** behind the Model
Context Protocol (MCP) rules the Trustabl scanner enforces when it runs in your
AWS pipeline. It exists so an AWS user reading a `trustabl.json` / `trustabl.sarif`
finding can understand *why* a rule fires, how confident the scanner is, and what
the fix is — without leaving this repo.

## Where these rules actually live

The AWS plugin does not embed rules. At scan time `scan/trustabl-scan.sh`
downloads the `trustabl` engine, which resolves the rule packs from
[`trustabl/trustabl-rules`](https://github.com/trustabl/trustabl-rules) (the MCP
pack is under [`mcp/`](https://github.com/trustabl/trustabl-rules/tree/main/mcp)).
The canonical, engine-maintained rationale is in
[`trustabl/trustabl-rulebook`](https://github.com/trustabl/trustabl-rulebook).

These files are an **AWS-plugin-local copy** of the rationale for the 14 rules in
the shipped MCP set (MCP-001–MCP-014, the Python + TypeScript era). They mirror
the upstream framing so results are self-explanatory here. If the upstream pack
adds or changes rules (it has since grown Go, C#, PHP, and Rust rules), the
rulebook is the source of truth; refresh these docs against it.

## The 14 rules

Risk score = `severity_weight × confidence × 100` (engine formula; weights:
low = 0.15, medium = 0.40, high = 0.70). Higher = worse.

| Id | Policy | Severity | Confidence | Risk | Doc |
|---|---|---|---|---|---|
| MCP-001 | Tool has no description | low | 0.90 | 13.5 | [tool_definition.md](tool_definition.md) |
| MCP-002 | Tool has no type-annotated parameters | medium | 0.85 | 34.0 | [tool_definition.md](tool_definition.md) |
| MCP-003 | Ambiguous tool name | low | 0.85 | 12.8 | [tool_definition.md](tool_definition.md) |
| MCP-004 | Network call has no timeout | high | 0.85 | 59.5 | [network.md](network.md) |
| MCP-005 | Path parameter used in I/O without validation | high | 0.70 | 49.0 | [path_safety.md](path_safety.md) |
| MCP-006 | Raises exceptions without a structured error contract | low | 0.60 | 9.0 | [error_handling.md](error_handling.md) |
| MCP-007 | Mutating tool has no idempotency key | medium | 0.55 | 22.0 | [idempotency.md](idempotency.md) |
| MCP-008 | Fetches a caller-controlled URL (SSRF) | high | 0.60 | 42.0 | [ssrf.md](ssrf.md) |
| MCP-009 | Body calls eval/exec/compile on dynamic input | high | 0.85 | 59.5 | [code_execution.md](code_execution.md) |
| MCP-010 | Body spawns a subprocess | high | 0.70 | 49.0 | [shell_safety.md](shell_safety.md) |
| MCP-011 | TypeScript MCP tool has no description | low | 0.85 | 12.8 | [tool_definition.md](tool_definition.md) |
| MCP-012 | TypeScript MCP tool spawns a subprocess | high | 0.70 | 49.0 | [shell_safety.md](shell_safety.md) |
| MCP-013 | TypeScript MCP tool fetches a caller-controlled URL (SSRF) | high | 0.60 | 42.0 | [ssrf.md](ssrf.md) |
| MCP-014 | TypeScript MCP tool evaluates dynamic code | high | 0.90 | 63.0 | [code_execution.md](code_execution.md) |

All 14 are **tool-scope** rules (`applies_to: mcp_tool`) — they evaluate
individual MCP tool registrations, not agents or the repo.

## Rationale docs by topic

| Topic | File | Rules |
|---|---|---|
| Tool definition hygiene | [tool_definition.md](tool_definition.md) | MCP-001, MCP-002, MCP-003, MCP-011 |
| Network hygiene | [network.md](network.md) | MCP-004 |
| Filesystem path safety | [path_safety.md](path_safety.md) | MCP-005 |
| Error contract hygiene | [error_handling.md](error_handling.md) | MCP-006 |
| Mutating-tool idempotency | [idempotency.md](idempotency.md) | MCP-007 |
| Server-side request forgery | [ssrf.md](ssrf.md) | MCP-008, MCP-013 |
| Shell invocation safety | [shell_safety.md](shell_safety.md) | MCP-010, MCP-012 |
| Dynamic code execution | [code_execution.md](code_execution.md) | MCP-009, MCP-014 |

## How severity maps to gating

The AWS scanner gates the build on the worst finding (see
[`docs/EVALUATION.md`](../../EVALUATION.md) for the full model). By default a run
fails on any finding at **medium or above**, so MCP-002, MCP-004, MCP-005,
MCP-007, MCP-008, MCP-009, MCP-010, MCP-012, MCP-013, and MCP-014 can break a
build; the `low`-severity hygiene rules (MCP-001, MCP-003, MCP-006, MCP-011)
report but do not gate unless you set `STRICT=true`.

## References

Each doc cites the OWASP LLM Top 10 categories it maps to (LLM02 Sensitive
Information Disclosure, LLM05 Improper Output Handling, LLM06 Excessive Agency,
LLM10 Unbounded Consumption).
