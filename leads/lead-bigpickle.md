## 2026-09-03 15:49:40 UTC [target] (model bigpickle)
[NEXT] RAG: Search for JTL-Software AG's main website, API endpoints, and documentation to identify live targets with HTTP responses.  
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.  
[RISK] JTL-Software AG: 5/10 — No active testing performed. Limited reconnaissance depth.
## 2026-09-03 19:09:46 UTC [target] (model bigpickle)
[PRIO] api.jtl-cloud.com,7.7,data_exposure(bank-accounts/customers/orders via client_credentials)
[PRIO] auth.jtl-cloud.com,7.4,oauth(public Ory,OAuth code/device/token endpoints)
[PRIO] erp.jtl-cloud.com,6.9,config_leak(Sentry/PostHog/hidden auth URLs in public HTML)
[HYP] Ory OAuth client-secret/auth bypass via device flow and permissive response_types
[HYP] ERP REST `application.runas` / `all.read` scope over-grant in client_credentials tenant context
[HYP] ERP Cloud env-config leak enables Sentry/PostHog data exfil or internal service discovery
[HYP] Cross-tenant BOLA via x-tenant-id header on ERP GraphQL
class: IDOR
asset: https://api.jtl-cloud.com/erp/v2/graphql
confidence: 60
reasoning: GraphQL SDL mandates `x-tenant-id` header for target ERP instance; multi-tenant cloud; REST authentication model separates token (OAuth2) from tenant scope. If tenant isolation relies solely on the client-supplied `x-tenant-id` header rather than validating it against the JWT's granted tenant, a token from tenant A with `x-tenant-id: <victim>` may return victim tenant data.
evidence_needed: 401 vs 200/403 difference when sending a token with a mismatched/unknown x-tenant-id; a probing account/registration on one tenant then querying another.
verify_steps: passive-first — GET https://api.jtl-cloud.com/erp/v2/graphql without token → expect 401 "JWT not present" (confirm). Requires valid token for full test (AUTH_HELPED). Fetch GraphQL playground to see if it offers unauth schema/introspection.
impact: cross-tenant PII/ERP data dump (customers, orders, bank accounts, invoices). Severity: high/critical.
testability: AUTH_HELPED
[HYP] Power scope over-grant in client_credentials (system.all / all.read / application.runas)
class: AUTH
asset: https://auth.jtl-cloud.com/oauth2/token + https://api.jtl-cloud.com
confidence: 50
reasoning: GraphQL authorizes every mutation/query with `system.all` as OR-branch; REST publishes `all.read` ("read all data") and `application.runas` ("execute requests on behalf of another user") as grantable scopes to third-party apps. If a client_credentials/waviapp-registration flow permits a caller to assert these scopes (or `application.runas` impersonates another app), full ERP read/write results.
evidence_needed: obtaining a client grant that includes system.all/all.read/application.runas; then reaching a data endpoint.
verify_steps: passive — try GET on GraphQL playground / appregistration docs; cannot obtain scopes without authorized client (AUTH_HELPED). Route to HUMAN for sanctioned test app.
impact: full ERP compromise, mass assignment, runas impersonation. Severity: critical.
testability: AUTH_HELPED
[HYP] ERP Cloud env-config leak → Sentry/PostHog data exposure / internal service discovery
class: MISCONFIG
asset: https://erp.jtl-cloud.com/
confidence: 55
reasoning: Public prod HTML ships full `jtl-cloud-env-variables` JSON incl. active Sentry DSN (`82a877a92404e1ad4ca46e308d4ddf27@o4508919293149184.ingest.de.sentry.io/4509087222595664`), PostHog token `phc_SkFAxTIa4RUQxotci1Ftdw7xefZK2Ax0LEIGcl8zuD2`, Zitadel client ID, ORY URL. Public client IDs are normal; but if Sentry is misconfigured (open API/replays, SDK+DSN same in `-qa` block), stack traces/PII may be queryable.
evidence_needed: whether Sentry project/DSN allows unauthenticated event queries or exposes recent error events with PII.
verify_steps: passive — check if Sentry DSN projects are publicly queryable (usually require auth); not directly curlable. Route to HUMAN review of Sentry config.
impact: PII/stack-trace exposure from client-side error telemetry. Severity: low/medium.
testability: HUMAN_ONLY
## 2026-09-03 21:46:16 UTC [target] (model bigpickle)
[HYP] Cross-tenant BOLA via client-supplied `x-tenant-id` on ERP GraphQL
class: IDOR
asset: https://api.jtl-cloud.com/erp/v2/graphql
confidence: 60
reasoning: Live probe confirms POST /erp/v2/graphql returns 401 without token ("JWT not present" gate). Multi-tenant cloud separates OAuth2 token from tenant via client-supplied x-tenant-id header (per prior SDL). If tenant isolation relies on that header rather than JWT tenant claims, a token from tenant A with x-tenant-id of victim returns victim data.
evidence_needed: 401 vs 200/403 with a mismatched/unknown x-tenant-id using a valid scoped token; a test account on one tenant queried from another.
verify_steps: requires valid scoped token (AUTH_HELPED). Passive next: GET graphql playground/index to check for unauth introspection. Route to HUMAN for sanctioned trial on test tenant.
impact: cross-tenant PII dump (customers, orders, bank accounts, invoices). Severity: high/critical.
testability: AUTH_HELPED
[HYP] Power scope over-grant via client_credentials (system.all / all.read / application.runas)
class: AUTH
asset: https://auth.jtl-cloud.com/oauth2/token + https://api.jtl-cloud.com
confidence: 50
reasoning: Live OIDC advertises client_credentials + jwt-bearer grants. GraphQL authorizes mutations with `system.all` OR-branch; REST publishes `all.read` ("read all data") and `application.runas` ("execute on behalf of another user") as third-party scopes. If a registered app's client_credentials can assert these, full ERP read/write or runas impersonation.
evidence_needed: obtaining a client grant asserting these scopes; reaching a data endpoint.
verify_steps: passive — inspect appregistration docs/GraphQL SDL for scope description; cannot mint scopes without authorized client (AUTH_HELPED). Route to HUMAN for sanctioned test app.
impact: full ERP read/write, mass assignment, runas impersonation. Severity: critical.
testability: AUTH_HELPED
[HYP] ERP env-config secrets abuse → Sentry/PII exposure
class: MISCONFIG
asset: https://erp.jtl-cloud.com/
confidence: 55
reasoning: Live probe CONFIRMED erp.jtl-cloud.com serves jtl-cloud-env-variables JSON with active Sentry DSN, PostHog token, Zitadel client ID, ORY URL, and internal service URLs (account/hub). Public client IDs normal, but the same DSN/config block in `-qa` variant and a public Sentry project may expose client-side stack traces/PII or allow event query if misconfigured.
evidence_needed: whether Sentry project/DSN is publicly queryable or exposes PII-laden error events.
verify_steps: Sentry DSN projects normally require auth; not directly curlable. Route to HUMAN review of Sentry/PostHog config.
impact: PII/stack-trace exposure from client telemetry. Severity: low/medium.
testability: HUMAN_ONLY
