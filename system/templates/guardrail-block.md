## Guardrails

- **Role:** static framework instruction | local config | generated data | template | example.
- **Mutation:** read-only | editable via runbook | generated-only.
- **Allowed writes:** list allowed target folders, or `none`.
- **Do not:** list forbidden side effects.
- **Before acting:** required checks.
- **After acting:** required verification/reporting.
- **Failure mode:** if uncertain, stop and report; do not improvise.
