# Alerting and SLO standard

Alerts should indicate urgent user impact and provide a next action. Dashboards answer questions; paging alerts require action.

## Recommended objectives

| Objective | Target | Window | Paging condition |
| --- | --- | --- | --- |
| Workflow availability | 99.0% | rolling 30 days | sustained failed workflows; policy blocks excluded |
| Workflow latency | p95 < 3 s | 10 minutes | p95 > 3 s after evaluation delay |
| Tool reliability | > 99% | 15 minutes | dependency failures consume error budget |
| Grounding quality | top score ≥ 0.70 | per request | alert only on sustained distribution shift |
| Cost | workload-specific budget | 15 minutes / daily | sudden increase by model or prompt version |

The checked-in `ScanReplayAvailabilityBudgetFastBurn` rule implements the 99.0% availability
objective as a multi-window burn alert: both the 5-minute and 1-hour failure ratios must exceed
14.4 times the 1% error budget. This avoids paging on one isolated provider error while detecting
rapid, sustained user impact. Expected policy blocks are not included in the failure numerator.

## Alert contract

Every alert must include:

1. Symptom, threshold, and affected environment/service.
2. Dashboard and trace-search entry point.
3. First diagnostic query or breakdown.
4. Runbook section.
5. Owning team and escalation route.
6. Recovery condition.

Avoid paging on a single LLM error, a single low-relevance retrieval, or expected policy blocks. Group by affected dependency and protect monitors from low-traffic division-by-zero behavior.
