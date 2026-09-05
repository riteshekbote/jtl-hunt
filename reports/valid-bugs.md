# Validated findings (running count 0)

- 10 lead(s) marked VALID at 2026-09-05 16:16:27 UTC
  - **Verdict: VALID**
  - **Verdict: VALID**
  - **Verdict: VALID**
  - | Q2 Reachable? | **PARTIAL** — endpoint live (401 without JWT) but requires valid token to test |
  - | Q4 Provable non-invasively? | **NO** — requires valid JWT for tenant A, then query with `x-tenant-id: tenant-B`. Cannot obtain token without OAuth registration or test account |
  - | Q2 Reachable? | **PARTIAL** — OIDC discovery confirms device flow + public client support, but no valid `client_id` enumerated |
  - | Q4 Provable non-invasively? | **NO** — needs valid client_id to initiate device flow |
  - | 1 | FFN OAuth leaked credentials | **VALID** | 7.5 |
  - | 2 | FFN OAuth scope escalation | **VALID** | 8.1 |
  - | 3 | FFN OAuth redirect_uri bypass | **VALID** | 6.8 |
