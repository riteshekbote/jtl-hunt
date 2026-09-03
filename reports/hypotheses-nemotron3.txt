## 2026-09-03 16:04:49 UTC [target] (model nemotron3)
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
