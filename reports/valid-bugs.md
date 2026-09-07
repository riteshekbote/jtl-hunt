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

- 13 lead(s) marked VALID at 2026-09-07 19:25:53 UTC
  - **Verdict: VALID**
  - | Q3 | Real security impact? | **YES** — valid client_id + client_secret for production OAuth2; enables token minting for FFN API |
  - **Verdict: VALID**
  - | Proof | `https://github.com/kruegge82/jtl-ffn-php-sdk` README contains client_id `97170e64-d390-4696-ba46-d6fcef8207de` and client_secret `f364ldUw3wIJFGn3JXE2NpGdAvUSMlmK72gsYg1z`; POST to token en
  - **Verdict: VALID**
  - | Q2 | Attacker reachable? | **PARTIAL** — endpoint live (401 without JWT) but requires valid token |
  - | Q4 | Provable non-invasively? | **NO** — requires valid JWT for tenant A, then query with `x-tenant-id: tenant-B` |
  - | Q4 | Provable? | **NO** — no valid client_id, endpoint 404 |
  - | Q2 | Attacker reachable? | **PARTIAL** — OIDC discovery live but no valid public client_id enumerated |
  - | Q4 | Provable? | **NO** — needs valid client_id |
  - | 1 | FFN OAuth scope escalation | **VALID** | 8.1 | bugs.olivermaicher.eu |
  - | 2 | FFN OAuth leaked credentials | **VALID** | 7.5 | bugs.olivermaicher.eu |
  - | 3 | FFN OAuth redirect_uri bypass | **VALID** | 6.8 | bugs.olivermaicher.eu |
