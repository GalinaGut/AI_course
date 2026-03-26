## 1) Evidence supporting conclusion (jwt_refresh_v2 + Redis)

### Timeline correlation

```
03:10  deploy: jwt_refresh_v2
03:15  Redis CPU 85%
run-42 fails=50  flags=[jwt_refresh_v2]

04:40  Redis CPU 90%
run-43 fails=70
run-44 fails=80
```

### Metric jump (start of instability)

```
run-41 (DEV)   fails=20   avg=850
run-42 (UAT)   fails=50   avg=870   <-- spike starts
```

### Consistent degradation (UAT only)

```
Fails:    50 → 70 → 80
Retries:  22 → 30 → 40
Time:     870 → 920
```

### Isolation check

```
Env   Fails   Flags             Status
DEV   20      -                 stable
UAT   50→80   jwt_refresh_v2    degraded
PROD  10      -                 stable
```

Conclusion:

* Spike starts immediately after deploy (03:10)
* Same flag only in failing runs (42–44)
* Infra alerts align with retry growth

---

## 2) Alternative possible cause

### Hypothesis: Redis bottleneck as primary cause

Evidence:

```
CPU:      85% → 90%
Retries:  22 → 40
Latency:  870 → 920
```

Interpretation:

```
High Redis load
      ↓
Slow responses
      ↓
More retries
      ↓
Higher failures
```

### Comparison of causes

```
Cause             Supports trend                Weakness
jwt_refresh_v2    exact timing, flag match      indirect infra impact
Redis overload    matches retries + latency     no issue in DEV/PROD
```

---

## Final

```
Most likely:
jwt_refresh_v2 → increased load → Redis saturation → failures

Alternative:
Redis issue alone (less likely, no cross-env impact)
```
