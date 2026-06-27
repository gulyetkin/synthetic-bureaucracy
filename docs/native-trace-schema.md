# Native trace schema

SBR accepts **native** workflow traces as JSON objects. This document describes the input contract used by the [browser demo](https://syntheticbureaucracy.com/analyze) and the private scoring engine. This showcase repo contains **examples only** — not the analyzer implementation.

## Required top-level fields

| Field | Type | Description |
|-------|------|-------------|
| `workflow_id` | string | Non-empty workflow identifier |
| `expected_outcome` | string | Non-empty stated goal |
| `messages` | array | Non-empty list of message objects |
| `tool_calls` | array | List of tool call objects (may be empty) |
| `final_status` | string | Non-empty terminal status (e.g. `completed`, `not_completed`) |

## Message objects

Each item in `messages` must be an object with:

| Field | Type | Required |
|-------|------|----------|
| `agent` | string | Yes — non-empty agent name |
| `content` | string | Yes — non-empty message body |

**Do not use `role`.** Traces that use `role` instead of `agent` are rejected with a schema error (exit code `3`).

Optional: `purpose` (execution, policy, audit, etc.)

## Tool call objects

Each item in `tool_calls` must be an object with:

| Field | Type | Required |
|-------|------|----------|
| `agent` | string | Yes — non-empty agent name |
| `tool` | string | Yes — non-empty tool name |
| `result` | string | Yes — may be empty string |
| `args` | object | No — if present, must be a JSON object |

**Do not use `name` for the tool field.** Traces that use `name` instead of `tool` are rejected with a schema error (exit code `3`).

## Validation rules (summary)

- All required top-level fields must be present.
- `messages` must contain **at least one** message.
- Wrong field names (`role`, `name`) are **not** auto-mapped — they fail closed.
- Arbitrary extra fields may be ignored by adapters but are not part of this contract.

## Example outcomes in this repo

| Trace | Result |
|-------|--------|
| `examples/healthy_workflow.json` | SBI 17 / A · Gate PASS · exit `0` |
| `examples/bloated_workflow.json` | SBI 100 / F · Gate FAIL · exit `2` (with `--fail-above 50`) |
| `examples/invalid_role.json` | Schema error · exit `3` |
| `examples/empty_workflow.json` | Schema error · exit `3` |

See [ci-exit-codes.md](ci-exit-codes.md) for exit code semantics.

## Report output

Successful analysis returns a JSON report (see `reports/*.report.json` in this repo). Reports include `schema_version`, `synthetic_bureaucracy_index`, `grade`, `failure_modes`, `diagnosis`, and optional `gate` evaluation.

Scores in this showcase were produced by the private SBR engine for **synthetic fixtures only**. They are not production- or customer-validated.
