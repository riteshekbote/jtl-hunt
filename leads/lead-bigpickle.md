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
## 2026-09-03 23:48:13 UTC [target] (model bigpickle)
[NEXT] RAG: Search for JTL-Software AG's main website, API endpoints, and documentation to identify live targets with HTTP responses.  
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.  
[RISK] JTL-Software AG: 5/10 — No active testing performed. Limited reconnaissance depth.
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
[NEW] 300 containerized test hosts under `docker.jtl-software.de` discovered via passive DNS/CT (131 JTL-Shop, 81 Shopware6, 47 WooCommerce, 25 Shopware5, 9 PrestaShop, 3 Gambio, 5 Modified)
[NEW] 23 shared test configuration profiles (suffixes like `a-b-4db87dad`, `p-g-443d1d50`, `f-b-e5fa382e`, `l-w-ab0f5ac0`, `t-p-817daf04`) deployed across multiple platforms and environment instances (prefixes 1-, 10-, 11-, 12-, 13-, 14-, 100-, etc.)
[NEW] 0 live HTTP probes performed — all hosts unvalidated for reachability, tech stack, or attack surface
[CHANGED] Dedicated deep recon confirms wildcard-dominated DNS (159 resolving, 0 genuinely dedicated after shared-IP filtering)
[PRIO] jtl-shop:a-b-4db87dad,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] jtl-shop:p-g-443d1d50,8.5,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:f-b-e5fa382e,8.2,attack_surface=8,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:l-w-ab0f5ac0,8.0,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:t-p-817daf04,7.8,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=6,freshness=8
[PRIO] shopware6:a-b-4db87dad,7.5,attack_surface=9,business_value=7,tech_exposure=8,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] woocommerce:p-g-443d1d50,7.0,attack_surface=8,business_value=6,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de (representative of 3 JTL-Shop + 29 Shopware6 + 5 WooCommerce instances)
confidence: 65
reasoning: Profile a-b-4db87dad is the most widely deployed cross-platform config (37 instances). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms — if one leaks API, all do.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de (representative of 8 JTL-Shop + 12 Shopware6 + 9 WooCommerce = 29 instances)
confidence: 55
reasoning: Profile p-g-443d1d50 is second most deployed (29 instances). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs often use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e
class: OTHER
asset: 1-jtl-shop-f-b-e5fa382e.docker.jtl-software.de (representative of 3 JTL-Shop + 7 Shopware6 + 5 WooCommerce = 15 instances)
confidence: 45
reasoning: Profile f-b-e5fa382e deployed across 15 containers. JTL-Shop media manager allows file uploads (product images, exports). Test profiles may disable MIME validation, allow .php/.phtml upload, or lack extension filtering. Shared profile = shared vulnerability.
evidence_needed: POST /admin/media/upload accepts .php/.phtml with Content-Type image/png; uploaded file executable at /media/uploads/shell.php; MIME bypass via polyglot or magic bytes
verify_steps: GET /admin/media (check for upload UI); OPTIONS /admin/media/upload (check allowed methods); passive fingerprint of upload endpoint via robots.txt/sitemap — no actual upload (mutating)
impact: RCE on test containers → lateral movement to internal network, source code access, CI/CD credential theft — HIGH
testability: PASSIVE (fingerprint only; upload test requires AUTH_HELPED)
[PARKED] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e: confidence 45 < 50 threshold; verify_steps require mutating upload test (not PASSIVE-only); class OTHER borderline for REJECTED-class "known-vulnerable library without program-specific exploit"
[FINAL] 1. JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (confidence 65)
[FINAL] 2. JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50 (confidence 55)
[NEXT] PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response structure
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[RISK] jtl: 75 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth exposure in test envs could leak production keys or enable supply-chain attacks; no live validation yet — risk increases with each confirmed finding
[NEW] Knowledge base adds 3 LEARN entries (ACCEPTED MISCONFIG a-b-4db87dad, ACCEPTED AUTH p-g-443d1d50, REJECTED OTHER f-b-e5fa382e) not in prior leads
[NEW] Inventory provides full 300-host enumeration (131 JTL-Shop, 81 Shopware6, 47 WooCommerce, 25 Shopware5, 9 PrestaShop, 3 Gambio, 5 Modified) for targeting
[CHANGED] Phase advanced to SURFACE with target=api — requires live HTTP validation of top hypotheses
[PRIO] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de,8.9,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de,8.4,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de
confidence: 65
reasoning: Profile a-b-4db87dad deployed across 37 containers (3 JTL-Shop, 29 Shopware6, 5 WooCommerce). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de
confidence: 55
reasoning: Profile p-g-443d1d50 deployed across 29 containers (8 JTL-Shop, 12 Shopware6, 9 WooCommerce). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs historically use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0
class: MISCONFIG
asset: 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de
confidence: 50
reasoning: Profile l-w-ab0f5ac0 deployed on 2 JTL-Shop containers (116, 118) plus Shopware6/WooCommerce. JTL-Shop v5+ may expose GraphQL at `/graphql` or `/api/graphql`. Test profiles often enable introspection (`__schema`, `__type`) for development. Shared config = shared exposure.
evidence_needed: POST /graphql with `{"query":"{__schema{types{name}}}"}` returns 200 with full type schema; GET /graphql?query=... also works; no auth required
verify_steps: POST https://116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de/graphql Content-Type: application/json body: {"query":"{__schema{types{name}}}"}; GET /graphql?query={__schema{types{name}}}; GET /api/graphql — read-only
impact: Full API schema enumeration → field-level auth bypass discovery, hidden mutations, PII field mapping — HIGH (enables targeted IDOR/BOLA/mass assignment)
testability: PASSIVE
[PARKED] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e: confidence 45 < 50 threshold; verify_steps require mutating upload test (not PASSIVE-only); class OTHER borderline for REJECTED-class "known-vulnerable library without program-specific exploit"
[FINAL] 1. JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (confidence 65)
[FINAL] 2. JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50 (confidence 55)
[FINAL] 3. JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0 (confidence 50)
[NEXT] PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response structure
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
[RISK] jtl: 78 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth/GraphQL exposure in test envs could leak production keys or enable supply-chain attacks; no live validation yet — risk increases with each confirmed finding
[NEW] All 300 hosts resolve to single IP 31.172.91.250 (wildcard DNS) but TCP 80/443 connection timeout — no live HTTP reachability from internet
[CHANGED] Phase SURFACE target=api blocked: top-priority assets (1-jtl-shop-a-b-4db87dad, 1-jtl-shop-p-g-443d1d50, 116-jtl-shop-l-w-ab0f5ac0) unreachable; cannot validate API/OAuth/GraphQL hypotheses passively
[PRIO] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de,8.9,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de,8.4,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de
confidence: 65
reasoning: Profile a-b-4db87dad deployed across 37 containers (3 JTL-Shop, 29 Shopware6, 5 WooCommerce). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de
confidence: 55
reasoning: Profile p-g-443d1d50 deployed across 29 containers (8 JTL-Shop, 12 Shopware6, 9 WooCommerce). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs historically use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0
class: MISCONFIG
asset: 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de
confidence: 50
reasoning: Profile l-w-ab0f5ac0 deployed on 2 JTL-Shop containers (116, 118) plus Shopware6/WooCommerce. JTL-Shop v5+ may expose GraphQL at `/graphql` or `/api/graphql`. Test profiles often enable introspection (`__schema`, `__type`) for development. Shared config = shared exposure.
evidence_needed: POST /graphql with `{"query":"{__schema{types{name}}}"}` returns 200 with full type schema; GET /graphql?query=... also works; no auth required
verify_steps: POST https://116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de/graphql Content-Type: application/json body: {"query":"{__schema{types{name}}}"}; GET /graphql?query={__schema{types{name}}}; GET /api/graphql — read-only
impact: Full API schema enumeration → field-level auth bypass discovery, hidden mutations, PII field mapping — HIGH (enables targeted IDOR/BOLA/mass assignment)
testability: PASSIVE
[PARKED] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad: cannot verify — host unreachable (TCP timeout on 31.172.91.250:443/80); all 300 hosts share same IP, containers not internet-accessible
[PARKED] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50: cannot verify — same network unreachability
[PARKED] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0: cannot verify — same network unreachability
[FINAL] No hypotheses survive live validation — all require network access to docker.jtl-software.de containers
[NEXT] RAG: Determine if docker.jtl-software.de test environment requires VPN/SSH tunnel or internal network access; check JTL-Software AG bug bounty program docs for test environment access instructions
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
[LEARN] REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack surface; test containers not internet-routable
[RISK] jtl: 78 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth/GraphQL exposure in test envs could leak production keys or enable supply-chain attacks; no live validation possible — containers not internet-accessible; risk confirmation requires internal network access
[NEXT] RAG: Search for JTL-Software AG's main website, API endpoints, and documentation to identify live targets with HTTP responses.  
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.  
[RISK] JTL-Software AG: 5/10 — No active testing performed. Limited reconnaissance depth.
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
[NEW] 300 containerized test hosts under `docker.jtl-software.de` discovered via passive DNS/CT (131 JTL-Shop, 81 Shopware6, 47 WooCommerce, 25 Shopware5, 9 PrestaShop, 3 Gambio, 5 Modified)
[NEW] 23 shared test configuration profiles (suffixes like `a-b-4db87dad`, `p-g-443d1d50`, `f-b-e5fa382e`, `l-w-ab0f5ac0`, `t-p-817daf04`) deployed across multiple platforms and environment instances (prefixes 1-, 10-, 11-, 12-, 13-, 14-, 100-, etc.)
[NEW] 0 live HTTP probes performed — all hosts unvalidated for reachability, tech stack, or attack surface
[CHANGED] Dedicated deep recon confirms wildcard-dominated DNS (159 resolving, 0 genuinely dedicated after shared-IP filtering)
[PRIO] jtl-shop:a-b-4db87dad,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] jtl-shop:p-g-443d1d50,8.5,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:f-b-e5fa382e,8.2,attack_surface=8,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:l-w-ab0f5ac0,8.0,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] jtl-shop:t-p-817daf04,7.8,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=6,freshness=8
[PRIO] shopware6:a-b-4db87dad,7.5,attack_surface=9,business_value=7,tech_exposure=8,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] woocommerce:p-g-443d1d50,7.0,attack_surface=8,business_value=6,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de (representative of 3 JTL-Shop + 29 Shopware6 + 5 WooCommerce instances)
confidence: 65
reasoning: Profile a-b-4db87dad is the most widely deployed cross-platform config (37 instances). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms — if one leaks API, all do.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de (representative of 8 JTL-Shop + 12 Shopware6 + 9 WooCommerce = 29 instances)
confidence: 55
reasoning: Profile p-g-443d1d50 is second most deployed (29 instances). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs often use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e
class: OTHER
asset: 1-jtl-shop-f-b-e5fa382e.docker.jtl-software.de (representative of 3 JTL-Shop + 7 Shopware6 + 5 WooCommerce = 15 instances)
confidence: 45
reasoning: Profile f-b-e5fa382e deployed across 15 containers. JTL-Shop media manager allows file uploads (product images, exports). Test profiles may disable MIME validation, allow .php/.phtml upload, or lack extension filtering. Shared profile = shared vulnerability.
evidence_needed: POST /admin/media/upload accepts .php/.phtml with Content-Type image/png; uploaded file executable at /media/uploads/shell.php; MIME bypass via polyglot or magic bytes
verify_steps: GET /admin/media (check for upload UI); OPTIONS /admin/media/upload (check allowed methods); passive fingerprint of upload endpoint via robots.txt/sitemap — no actual upload (mutating)
impact: RCE on test containers → lateral movement to internal network, source code access, CI/CD credential theft — HIGH
testability: PASSIVE (fingerprint only; upload test requires AUTH_HELPED)
[PARKED] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e: confidence 45 < 50 threshold; verify_steps require mutating upload test (not PASSIVE-only); class OTHER borderline for REJECTED-class "known-vulnerable library without program-specific exploit"
[FINAL] 1. JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (confidence 65)
[FINAL] 2. JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50 (confidence 55)
[NEXT] PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response structure
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[RISK] jtl: 75 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth exposure in test envs could leak production keys or enable supply-chain attacks; no live validation yet — risk increases with each confirmed finding
[NEW] Knowledge base adds 3 LEARN entries (ACCEPTED MISCONFIG a-b-4db87dad, ACCEPTED AUTH p-g-443d1d50, REJECTED OTHER f-b-e5fa382e) not in prior leads
[NEW] Inventory provides full 300-host enumeration (131 JTL-Shop, 81 Shopware6, 47 WooCommerce, 25 Shopware5, 9 PrestaShop, 3 Gambio, 5 Modified) for targeting
[CHANGED] Phase advanced to SURFACE with target=api — requires live HTTP validation of top hypotheses
[PRIO] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de,8.9,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de,8.4,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de
confidence: 65
reasoning: Profile a-b-4db87dad deployed across 37 containers (3 JTL-Shop, 29 Shopware6, 5 WooCommerce). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de
confidence: 55
reasoning: Profile p-g-443d1d50 deployed across 29 containers (8 JTL-Shop, 12 Shopware6, 9 WooCommerce). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs historically use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0
class: MISCONFIG
asset: 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de
confidence: 50
reasoning: Profile l-w-ab0f5ac0 deployed on 2 JTL-Shop containers (116, 118) plus Shopware6/WooCommerce. JTL-Shop v5+ may expose GraphQL at `/graphql` or `/api/graphql`. Test profiles often enable introspection (`__schema`, `__type`) for development. Shared config = shared exposure.
evidence_needed: POST /graphql with `{"query":"{__schema{types{name}}}"}` returns 200 with full type schema; GET /graphql?query=... also works; no auth required
verify_steps: POST https://116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de/graphql Content-Type: application/json body: {"query":"{__schema{types{name}}}"}; GET /graphql?query={__schema{types{name}}}; GET /api/graphql — read-only
impact: Full API schema enumeration → field-level auth bypass discovery, hidden mutations, PII field mapping — HIGH (enables targeted IDOR/BOLA/mass assignment)
testability: PASSIVE
[PARKED] JTL-Shop file upload RCE via unrestricted media handler in profile f-b-e5fa382e: confidence 45 < 50 threshold; verify_steps require mutating upload test (not PASSIVE-only); class OTHER borderline for REJECTED-class "known-vulnerable library without program-specific exploit"
[FINAL] 1. JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (confidence 65)
[FINAL] 2. JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50 (confidence 55)
[FINAL] 3. JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0 (confidence 50)
[NEXT] PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response structure
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
[RISK] jtl: 78 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth/GraphQL exposure in test envs could leak production keys or enable supply-chain attacks; no live validation yet — risk increases with each confirmed finding
[NEW] All 300 hosts resolve to single IP 31.172.91.250 (wildcard DNS) but TCP 80/443 connection timeout — no live HTTP reachability from internet
[CHANGED] Phase SURFACE target=api blocked: top-priority assets (1-jtl-shop-a-b-4db87dad, 1-jtl-shop-p-g-443d1d50, 116-jtl-shop-l-w-ab0f5ac0) unreachable; cannot validate API/OAuth/GraphQL hypotheses passively
[PRIO] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de,8.9,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=8,freshness=8
[PRIO] 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de,8.8,attack_surface=9,business_value=10,tech_exposure=7,gate_ease=10,cloud_surface=7,freshness=8
[PRIO] 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de,8.4,attack_surface=8,business_value=10,tech_exposure=6,gate_ease=10,cloud_surface=7,freshness=8
[HYP] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad
class: MISCONFIG
asset: 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de
confidence: 65
reasoning: Profile a-b-4db87dad deployed across 37 containers (3 JTL-Shop, 29 Shopware6, 5 WooCommerce). JTL-Shop exposes REST API at `/api/v1/` or `/api/` by default. Test environments often lack auth middleware or have debug endpoints enabled (`/api/_debug`, `/api/v1/_info`, `/api/v1/system/info`). Shared profile suggests identical config across platforms.
evidence_needed: HTTP 200 on GET /api/v1/ or /api/ with JSON response containing version/users/products data; absent auth headers; debug endpoints returning stack traces or config
verify_steps: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/; GET /api/; GET /api/_debug; GET /api/v1/system/info; HEAD /api/v1/products — all read-only, no auth
impact: Unauthenticated access to JTL-Shop REST API → product/customer/order enumeration, potential mass assignment on write endpoints, API key leakage — HIGH severity (core product, multi-tenant exposure across 37 containers)
testability: PASSIVE
[HYP] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50
class: AUTH
asset: 1-jtl-shop-p-g-443d1d50.docker.jtl-software.de
confidence: 55
reasoning: Profile p-g-443d1d50 deployed across 29 containers (8 JTL-Shop, 12 Shopware6, 9 WooCommerce). JTL-Shop supports OAuth2/OIDC for third-party integrations. Test/staging envs historically use loose redirect_uri validation (allowing subdomain wildcard, HTTP localhost, or path traversal). Shared config across platforms means misconfig replicates.
evidence_needed: OAuth authorize endpoint accepts redirect_uri with path traversal (`/../evil.com`), open redirect on subdomain (`evil.docker.jtl-software.de`), or HTTP scheme; state parameter not enforced
verify_steps: GET /oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; GET /oauth/authorize?redirect_uri=http://localhost:8080/callback; GET /oauth/authorize?redirect_uri=https://1-jtl-shop-p-g-443d1d50.docker.jtl-software.de/../evil.com — read-only
impact: OAuth code theft → account takeover on connected JTL-Shop instances — CRITICAL if production OAuth clients share config; HIGH for test env credential harvesting
testability: PASSIVE
[HYP] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0
class: MISCONFIG
asset: 116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de
confidence: 50
reasoning: Profile l-w-ab0f5ac0 deployed on 2 JTL-Shop containers (116, 118) plus Shopware6/WooCommerce. JTL-Shop v5+ may expose GraphQL at `/graphql` or `/api/graphql`. Test profiles often enable introspection (`__schema`, `__type`) for development. Shared config = shared exposure.
evidence_needed: POST /graphql with `{"query":"{__schema{types{name}}}"}` returns 200 with full type schema; GET /graphql?query=... also works; no auth required
verify_steps: POST https://116-jtl-shop-l-w-ab0f5ac0.docker.jtl-software.de/graphql Content-Type: application/json body: {"query":"{__schema{types{name}}}"}; GET /graphql?query={__schema{types{name}}}; GET /api/graphql — read-only
impact: Full API schema enumeration → field-level auth bypass discovery, hidden mutations, PII field mapping — HIGH (enables targeted IDOR/BOLA/mass assignment)
testability: PASSIVE
[PARKED] JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad: cannot verify — host unreachable (TCP timeout on 31.172.91.250:443/80); all 300 hosts share same IP, containers not internet-accessible
[PARKED] JTL-Shop OAuth/OIDC redirect_uri validation bypass in profile p-g-443d1d50: cannot verify — same network unreachability
[PARKED] JTL-Shop GraphQL introspection enabled in profile l-w-ab0f5ac0: cannot verify — same network unreachability
[FINAL] No hypotheses survive live validation — all require network access to docker.jtl-software.de containers
[NEXT] RAG: Determine if docker.jtl-software.de test environment requires VPN/SSH tunnel or internal network access; check JTL-Software AG bug bounty program docs for test environment access instructions
[LEARN] ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
[LEARN] ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
[LEARN] REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
[LEARN] ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
[LEARN] REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack surface; test containers not internet-routable
[RISK] jtl: 78 — High-value core product (JTL-Shop) deployed in 131 test containers with 23 shared configs; wildcard DNS hides true attack surface; unauthenticated API/OAuth/GraphQL exposure in test envs could leak production keys or enable supply-chain attacks; no live validation possible — containers not internet-accessible; risk confirmation requires internal network access
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
[PRIO] api.jtl-cloud.com/erp/v2/graphql,7.7,idor(cross-tenant x-tenant-id BOLA vs JWT tenant claim)
[PRIO] auth.jtl-cloud.com,7.4,oauth(Ory grants: client_credentials+jwt-bearer+device; permissive response_types)
[PRIO] erp.jtl-cloud.com,6.9,misconfig(Sentry/PostHog env-config leak / internal service URLs)
[NEXT] RAG: Check JTL bug bounty program (bugs.olivermaicher.eu) / Ory setup for whether sanctioned test tenants, OAuth test clients, or a test ERP instance are provided — the top two hypotheses (x-tenant-id BOLA, jwt-bearer scope) both require a valid scoped token to verify.
[RISK] jtl: 72 — Primary cloud surface (auth/erp/api) live and code-mounted with high-value ERP/tenant-boundary targets (x-tenant-id BOLA, jwt-bearer scope); the decisive tests require a sanctioned token which the program may not provide, so real attacks are unverified vs a real logic flaw; container test-farm adds systemic-supply-chain theoretical risk but is unreachable. All in-scope but POC-limited.
[HYP] Cross-tenant BOLA via client-supplied `x-tenant-id` on ERP GraphQL
class: IDOR
asset: https://api.jtl-cloud.com/erp/v2/graphql
confidence: 60
reasoning: Live probe confirms 401 "JWT not present" gate (no valid token). Multi-tenant cloud separates OAuth2 token from tenant scope via client-supplied `x-tenant-id` header (per GraphQL SDL). If tenant isolation relies on that header rather than JWT tenant claims, a token from tenant A with `x-tenant-id: <victim>` returns victim data.
evidence_needed: 200/403 vs 40x difference sending a valid scoped token with a mismatched/unknown x-tenant-id; a test account on one tenant queried from another.
verify_steps: passive confirmed graphiql/playground 401 (no unauth introspection). Full test requires a valid scoped token — route to HUMAN for sanctioned test-tenant/registration (AUTH_HELPED). No token===401; token+mismatched-tenant===200 would prove isolation-by-header only.
impact: cross-tenant PII/ERP dump (customers, orders, bank accounts, invoices). Severity: high/critical.
testability: AUTH_HELPED
[HYP] Ory jwt-bearer client_assertion / permissive grant scope over-grant
class: AUTH
asset: https://auth.jtl-cloud.com/oauth2/token + https://api.jtl-cloud.com
confidence: 50
reasoning: Live OIDC advertises `client_credentials`, `jwt-bearer`, and device flow grants plus implicit `token`/`id_token` response_types. If jwt-bearer `client_assertion` accepts a loose issuer/aud or a self-minted JWT tied to a known public client_id, a caller can mint a token asserting `system.all`/`all.read`/`application.runas` and reach ERP data endpoints.
evidence_needed: token endpoint returns a valid access_token for a forged/loose assertion carrying a power scope; that token reaches a data endpoint.
verify_steps: cannot mint without an authorized client (AUTH_HELPED). Passive: read OIDC metadata + appregistration docs for scope descriptions. Route HUMAN for sanctioned test app.
impact: full ERP read/write, mass assignment, runas impersonation. Severity: critical.
testability: AUTH_HELPED
[HYP] ERP env-config leak → Sentry/PostHog client-telemetry PII / internal service discovery
class: MISCONFIG
asset: https://erp.jtl-cloud.com/
confidence: 55
reasoning: Confirmed prod HTML ships `jtl-cloud-env-variables` JSON with active Sentry DSN, PostHog token, Zitadel client ID, ORY URL, and internal account/hub service URLs. Public client IDs are normal; exposure only if Sentry/PostHog project misconfigured for unauth event query or replay of PII-laden stack traces.
evidence_needed: Sentry project/DSN publicly queryable or error events with PII fetchable.
verify_steps: Sentry DSN projects normally require auth; not directly curlable. Route to HUMAN review of Sentry/PostHog project config.
impact: PII/stack-trace exposure from client-side telemetry. Severity: low/medium.
testability: HUMAN_ONLY
[NEXT] RAG: Check the JTL program (bugs.olivermaicher.eu) and Ory setup for whether sanctioned test tenants, OAuth test clients, or a test ERP instance are provided — both top survivors (x-tenant-id BOLA, jwt-bearer scope) require a valid scoped token to verify.
## 2026-09-04 03:02:49 UTC [target] (model bigpickle)
[NEW] PROBE: GET https://auth.jtl-cloud.com/.well-known/openid-configuration — OIDC discovery to enumerate endpoints, grants, device_flow support.
[PRIO] api.jtl-cloud.com/erp/v2/graphql,7.75,attack_surface=8 business_value=9 tech_exposure=9 gate_ease=3 cloud_surface=9 freshness=8
[PRIO] auth.jtl-cloud.com,7.50,attack_surface=7 business_value=9 tech_exposure=9 gate_ease=3 cloud_surface=9 freshness=8
[PRIO] erp.jtl-cloud.com,6.25,attack_surface=6 business_value=7 tech_exposure=6 gate_ease=4 cloud_surface=8 freshness=7
[PRIO] bountyshop.jtl-software.com,6.00,attack_surface=7 business_value=5 tech_exposure=7 gate_ease=5 cloud_surface=4 freshness=8
[HYP] Cross-tenant BOLA via client-supplied x-tenant-id on ERP GraphQL
class: IDOR
asset: https://api.jtl-cloud.com/erp/v2/graphql
confidence: 60
reasoning: Live probe confirms 401 "JWT not present" gate (no valid token). Multi-tenant cloud separates OAuth2 token from tenant scope via client-supplied x-tenant-id header (per GraphQL SDL). If tenant isolation relies on that header rather than JWT tenant claims, a token from tenant A with x-tenant-id: <victim> returns victim data.
evidence_needed: 200/403 vs 40x difference sending a valid scoped token with a mismatched/unknown x-tenant-id; a test account on one tenant queried from another.
verify_steps: passive confirmed graphiql/playground 401 (no unauth introspection). Full test requires a valid scoped token — route to HUMAN for sanctioned test-tenant/registration (AUTH_HELPED). No token===401; token+mismatched-tenant===200 would prove isolation-by-header only.
impact: cross-tenant PII/ERP dump (customers, orders, bank accounts, invoices). Severity: high/critical.
testability: AUTH_HELPED
[HYP] Ory jwt-bearer client_assertion / permissive grant scope over-grant
class: AUTH
asset: https://auth.jtl-cloud.com/oauth2/token + https://api.jtl-cloud.com
confidence: 50
reasoning: Live OIDC advertises client_credentials, jwt-bearer, and device flow grants plus implicit token/id_token response_types. If jwt-bearer client_assertion accepts a loose issuer/aud or a self-minted JWT tied to a known public client_id, a caller can mint a token asserting system.all/all.read/application.runas and reach ERP data endpoints.
evidence_needed: token endpoint returns a valid access_token for a forged/loose assertion carrying a power scope; that token reaches a data endpoint.
verify_steps: cannot mint without an authorized client (AUTH_HELPED). Passive: read OIDC metadata + appregistration docs for scope descriptions. Route HUMAN for sanctioned test app.
impact: full ERP read/write, mass assignment, runas impersonation. Severity: critical.
testability: AUTH_HELPED
[HYP] ERP env-config leak → Sentry/PostHog client-telemetry PII / internal service discovery
class: MISCONFIG
asset: https://erp.jtl-cloud.com/
confidence: 55
reasoning: Confirmed prod HTML ships jtl-cloud-env-variables JSON with active Sentry DSN, PostHog token, Zitadel client ID, ORY URL, and internal account/hub service URLs. Public client IDs are normal; exposure only if Sentry/PostHog project misconfigured for unauth event query or replay of PII-laden stack traces.
evidence_needed: Sentry project/DSN publicly queryable or error events with PII fetchable.
verify_steps: Sentry DSN projects normally require auth; not directly curlable. Route to HUMAN review of Sentry/PostHog project config.
impact: PII/stack-trace exposure from client-side telemetry. Severity: low/medium.
testability: HUMAN_ONLY
[FINAL] 1. Cross-tenant BOLA via client-supplied x-tenant-id on ERP GraphQL (60)
[FINAL] 2. Ory jwt-bearer client_assertion / permissive grant scope over-grant (50)
[FINAL] 3. ERP env-config leak → Sentry/PostHog client-telemetry PII / internal service discovery (55)
[NEXT] PROBE: GET https://auth.jtl-cloud.com/.well-known/openid-configuration — OIDC discovery to enumerate endpoints, grants, device_flow support.
[RISK] JTL: 70 — Primary cloud surface (auth/erp/api) live and code-mounted with high-value ERP/tenant-boundary targets; the decisive tests require a sanctioned token which the program may not provide, so real attacks are unverified vs a real logic flaw; container test-farm adds systemic-supply-chain theoretical risk but is unreachable. All in-scope but POC-limited.
## 2026-09-04 08:04:23 UTC [target] (model bigpickle)
