# Provider-timeout incident walkthrough

This deterministic scenario is the recommended interview path for explaining detection,
investigation, replay, mitigation, and prevention without calling a live model provider.

## Incident contract

- **User-visible objective:** return a completed, policy-valid workflow within the latency budget.
- **Availability SLO:** 99.0% over a rolling 30-day window.
- **Successful request:** `workflow_status="completed"`; an HTTP 200 alone is not sufficient.
- **Expected policy block:** investigated separately and excluded from the failure numerator.
- **Page condition:** the 5-minute and 1-hour error ratios both exceed 14.4 times the 1% budget.

## Evidence timeline

| Relative time | Evidence | Operator interpretation |
| --- | --- | --- |
| +0 ms | Gateway creates the trace and records request/version attributes | One correlation ID now joins every service |
| +20 ms | Agent starts retrieval | The request reached orchestration normally |
| +70 ms | Retrieval completes with bounded relevance attributes | Retrieval is not the failing dependency |
| +90 ms | Provider span begins | Critical path enters the model boundary |
| +940 ms | `provider-timeout` raises `dependency_timeout` | First failing span identifies the dependency |
| +945 ms | Workflow records `failed`; error-budget numerator increases | Alert semantics match user impact |
| +960 ms | Sanitized fixture is written with raw content disabled | Evidence is retained without copying the prompt |
| +1.1 s | Operator opens the trace from the alert/runbook | Logs, metrics, and spans agree on category/version |
| +2 min | Multi-window fast-burn condition pages the AI platform owner | A single provider error does not page |
| +4 min | Fixture replay reproduces the timeout without a live call | Diagnosis is deterministic and safe |
| +8 min | Candidate profile applies bounded fallback/circuit-breaker behavior | Mitigation can be compared with the original |
| +12 min | Replay becomes a regression case | The incident strengthens the release gate |

## Commands

Run the fast, secret-free demo:

```bash
npm run demo
```

Run the full stack and select `provider-timeout` in the Console:

```bash
docker compose up --build
```

During the interview, pivot in this order:

1. Availability alert and error-budget meaning.
2. Trace critical path and first failing span.
3. Sanitized replay evidence.
4. Original-versus-candidate drift comparison.
5. Regression prevention and runbook owner.

## Boundaries

The provider is deterministic in this scenario so the failure can be reproduced in CI. This is
evidence of the observability and incident workflow, not a claim of production traffic volume or
of a live provider outage. A production implementation should add provider-specific circuit
breakers, regional routing, and contractual dependency SLOs.
