1) Environment summary
Env	Fail trend	Time trend	Notes
DEV	20	850	stable
UAT	50 → 80	870 → 920	degrading
PROD	10	845	stable

Conclusion: issue isolated to UAT, worsening each run

2) When instability started

Timeline (UAT)

03:10 — deploy: jwt_refresh_v2
03:15 — Redis CPU 85%
run-42 — fails 50 (first spike, flag appears)
04:40 — Redis CPU 90%
run-43 — fails 70
run-44 — fails 80

Evidence

Fail jump: 20 → 50 at run-42
Flag present only in failing runs (42–44)
Retries: 22 → 40 (growing load)

Conclusion: instability starts at run-42, directly after deploy, amplified by Redis overload

3) Forecast
Run	Expected fails
run-46	~95
run-47	~105+

Assumption: no rollback, Redis still saturated

4) Actions
Rollback jwt_refresh_v2
Reason: direct correlation with failure spike
Reduce Redis load
Reason: CPU 85–90%, bottleneck
Limit retries
Reason: retries (22 → 40) amplify failures
Summary

Root cause: jwt_refresh_v2 + Redis overload
Trend: failures increasing each run
Fix: rollback + stabilize Redis
