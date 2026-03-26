1) Trend Analysis (fail + execution time)

DEV (run-41)

fail: 20, avgTime: 850 → stable baseline

UAT (run-42→44)

fail: 50 → 70 → 80 (+60%)
retries: 22 → 40 (~+80%)
avgTime: 870 → 920 (degradation)
slowest: 3200 → 3750 (latency increase)
👉 systemic degradation + retry storm

PROD (run-45)

fail: 10, avgTime: 845 → stable
👉 issue is UAT-specific
2) Instability Start + Correlation
📍 Start: run-42

Evidence:

run-41 → run-42: fail 20 → 50
first appearance of flag: "jwt_refresh_v2"
🔗 Deployment
03:10 — UAT — Auth — jwt_refresh_v2 rollout
run-42:
  fail: 50
  flags: ["jwt_refresh_v2"]

👉 direct correlation: feature → failure spike

⚠️ Infra (Redis)
03:15 — CPU 85%
04:40 — CPU 90%

👉 +5 min after deploy → load spike

📉 Causal Chain
jwt_refresh_v2
→ more auth/refresh calls
→ Redis CPU 85–90%
→ latency increase (3200 → 3750)
→ retries increase (22 → 40)
→ failures increase (50 → 80)
3) Forecast (next 2 runs)

Trend slowing: +20 → +10

run-46: ~85–90 fails
run-47: ~90–95 fails (plateau)

Assumptions:

no fix applied
Redis remains saturated
feature still enabled
4) Mitigation Steps
1. Disable jwt_refresh_v2
reason: clear starting point of regression
impact: fast recovery to baseline
2. Stabilize Redis
CPU 85–90% = bottleneck
actions: scaling / tuning
3. Optimize refresh logic
issue: retry storm (22 → 40)
solution: rate limiting / backoff / caching
Summary
instability started at run-42 (03:10 deploy)
confirmed by Redis overload (03:15+)
root cause: Auth change + infra bottleneck
without fix → plateau at ~90+ failures
