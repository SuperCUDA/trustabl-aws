---
policy_id: mcp_error_handling
category: mcp
topic: error_handling
rules:
  - id: MCP-006
    severity: low
    confidence: 0.6
    scope: tool
    fix_type: code
references: [LLM05]
---

# Policy Rationale: MCP Error Contract Hygiene

**Policy ID:** `mcp_error_handling`
**Rule source:** [`mcp/error_handling.yaml`](https://github.com/trustabl/trustabl-rules/blob/main/mcp/error_handling.yaml)
**Rules (shipped set):** MCP-006
**References:** LLM05 (Improper Output Handling)

> This is the AWS-plugin copy of the rationale. The canonical rulebook shares
> the structured-error threat model with the OpenAI Agents SDK pack
> ([openai_sdk/error_handling.md](https://github.com/trustabl/trustabl-rulebook/blob/main/docs/Policy/openai_sdk/error_handling.md));
> this document covers the MCP-specific angle only.

---

## What this policy covers

An MCP tool handler that can raise without catching, detected by
`all: [has_raise: true, has_try_except: false]`.

---

## Rule-by-rule defense

### MCP-006 — Tool raises exceptions without a structured error contract (Severity: low, Confidence: 0.6, Fix type: code)

**What we detect:** a handler body that contains a `raise` and no `try`/`except`.

**Why it is flaggable:** when an MCP tool handler raises, the runtime surfaces the
exception to the connecting client as an opaque protocol error. The model on the
other end often cannot recover or retry intelligently, and the raw message may
leak internal detail — stack frames, absolute paths, secrets in arguments —
across the server's trust boundary to whatever client connected (improper output
handling, LLM05). Low severity because the impact is degraded recovery plus a
modest disclosure channel, and a handler often raises intentionally for a caller
or runtime that structures it; confidence 0.6 because the body-only check does
not see a `try` in a calling frame.

**Fix type — code:** returning a structured `{"error": ..., "retryable": ...}`
result instead of raising is a source edit.

---

## Fix guidance

Catch known failure modes and return a structured result the model can branch on
(e.g. `{"error": ..., "retryable": bool}`) instead of letting the exception
propagate as a protocol error.

---

## What this policy does not cover

Whether the raised message actually contains sensitive data; exception handling
done by a wrapping frame outside the handler body; and the TypeScript MCP error
surface (no TS raise/catch predicate is wired).
