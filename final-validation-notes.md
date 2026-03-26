## Verified
- Failure spike matches deploy timing
- UAT-only degradation confirmed
- Redis CPU correlates with retries
- Consistent worsening across runs

---

## Corrected
- Root cause: jwt_refresh_v2
- Amplifier: Redis overload

---

## Clarified
- Not infra-only (DEV/PROD stable)
- Not test-only (system metrics affected)
- Retries amplify failures, not root cause
