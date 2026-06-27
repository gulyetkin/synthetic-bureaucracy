# CI exit codes

SBR is designed for **fail-closed CI gates** on agent workflow traces. The CLI and hosted demo surface these exit codes:

| Code | Meaning | Typical cause |
|------|---------|---------------|
| **0** | Pass | Analysis completed; gate passed (or no gate configured) |
| **1** | Usage error | Missing arguments, unknown flags, invalid CLI usage |
| **2** | Gate fail | Analysis completed; SBI or `--fail-on` mode threshold failed |
| **3** | Analysis / schema error | Invalid JSON, native schema violation, malformed trace, timeout, internal error |

## Gate pass (0)

```bash
sbr analyze examples/healthy_workflow.json
# exit 0 — SBI within policy, no gate failure
```

## Gate fail (2)

```bash
sbr analyze examples/bloated_workflow.json --fail-above 50
# exit 2 — SBI exceeds threshold; gate.enabled=true, gate.passed=false
```

Gate failure is **deterministic** — no LLM in the loop.

## Schema / analysis error (3)

Invalid traces fail before scoring:

- `messages` uses `role` instead of `agent`
- `messages` is empty
- Missing required fields

These map to exit code **3** in the hosted demo and fail-closed CLI behavior.

## CI usage sketch

```yaml
- name: Gate agent workflow trace
  run: |
    sbr analyze traces/my_workflow.json --fail-above 75 --fail-on circular_handoff
```

Treat exit **2** as a policy failure (block merge/deploy). Treat exit **3** as an input or infrastructure problem (fail closed).

## Not covered here

This document describes **semantics only**. The private analyzer implementation, scoring engine, and gate implementation are not included in this showcase repository.
