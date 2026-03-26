## High-Level Summary (Last 3 Runs)

Metric     run-42   run-43   run-44   Trend
-------------------------------------------
Fails        50       70       80     ↑ worsening
Flaky        15       22       30     ↑ unstable
Retries      22       30       40     ↑ increasing
Avg Time     870      890      920    ↑ slower

Status: ❌ Degrading (UAT only)

---

## Trend Analysis

Failures:   50 → 70 → 80 (+60%)
Flakiness:  15 → 30 (x2)
Latency:    870 → 920 ms

Pattern:
Deploy → Load ↑ → Retries ↑ → Failures ↑

Conclusion: Regression after jwt_refresh_v2

---

## Visual Overview

Failures (ASCII):
run-42   ██████████████████████████ (50)
run-43   ███████████████████████████████████ (70)
run-44   ████████████████████████████████████████ (80)

Environment:
ENV   FAILS   STATUS
-------------------------
DEV     20    stable
UAT     50→80 degraded
PROD    10    stable

---

## Root Cause

jwt_refresh_v2 deploy
      ↓
Redis CPU 85–90%
      ↓
Retries 22 → 40
      ↓
Failures 50 → 80

Primary: feature impact
Secondary: Redis bottleneck

---

## Forecast

run-46 ≈ 95 fails
run-47 ≈ 105+ fails

Assumptions:
- No rollback
- Redis still overloaded

---

## Recommendations

1. DevOps — Scale Redis → remove bottleneck  
2. Backend — Rollback/fix jwt_refresh_v2 → stop spike  
3. QA — Limit retries → reduce amplification  

---

## Audience Sections

QA:
- Flaky ↑, retry noise
- Separate infra vs test failures

Developers:
- Issue tied to jwt_refresh_v2
- Optimize token/Redis usage

Management:
- UAT unstable, PROD safe
- Decide: rollback vs fix
