# Hypotheses (ranked)

## RANKED HYPOTHESES 2026-09-02 21:38:57 UTC

## RANKED HYPOTHESES 2026-09-02 23:34:39 UTC

## RANKED HYPOTHESES 2026-09-03 01:28:22 UTC

## RANKED HYPOTHESES 2026-09-03 06:30:41 UTC

## RANKED HYPOTHESES 2026-09-03 11:43:15 UTC

## RANKED HYPOTHESES 2026-09-03 16:05:03 UTC
- [65] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de: JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response str
- NEXT(hypotheses-bigpickle.txt): RAG: Search for JTL-Software AG's main website, API endpoints, and documentation to identify live targets with HTTP responses.
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.

## RANKED HYPOTHESES 2026-09-03 19:09:54 UTC
- [65] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de: JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (from art/lead_nemotron3.txt)
- [60] https://api.jtl-cloud.com/erp/v2/graphql: Ory OAuth client-secret/auth bypass via device flow and permissive response_types (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://1-jtl-shop-a-b-4db87dad.docker.jtl-software.de/api/v1/ HTTP/1.1 — read-only, no auth, single request to confirm API exposure and response str
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.

## RANKED HYPOTHESES 2026-09-03 21:52:56 UTC
- [65] 1-jtl-shop-a-b-4db87dad.docker.jtl-software.de: JTL-Shop API endpoint exposure in shared test profile a-b-4db87dad (from art/lead_nemotron3.txt)
- [60] https://api.jtl-cloud.com/erp/v2/graphql: Cross-tenant BOLA via client-supplied `x-tenant-id` on ERP GraphQL (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): RAG: Determine if docker.jtl-software.de test environment requires VPN/SSH tunnel or internal network access; check JTL-Software AG bug bounty program docs for 
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack su

## RANKED HYPOTHESES 2026-09-03 23:53:50 UTC
- [60] https://api.jtl-cloud.com/erp/v2/graphql: Ory OAuth client-secret/auth bypass via device flow and permissive response_types (from art/lead_bigpickle.txt)
- [60] bountyshop.jtl-software.com: JTL-Shop v5.x unauthenticated SSTI/RCE via email template (CVE-2026-54390) on bountyshop (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: Search for JTL-Software AG's main website, API endpoints, and documentation to identify live targets with HTTP responses.
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.jtl-cloud.com/.well-known/openid-configuration — read-only OAuth/OIDC discovery to enumerate endpoints, grants, device_flow support, and 
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack su
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack su
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack su
- LEARN: ACCEPTED TARGET @ bountyshop.jtl-software.com: Official bug bounty test shop confirmed live (HTTP 200), JTL-Shop v5.x NOVA template, admin panel at /admin/
- LEARN: ACCEPTED TARGET @ api.jtl-cloud.com/erp/v2/graphql: Production GraphQL ERP API live, requires JWT, x-tenant-id header for multi-tenancy
- LEARN: ACCEPTED TARGET @ jtl-software.github.io/devdocs-graphql-playground/: GraphiQL playground live, embedded in developer portal, targets production API

## RANKED HYPOTHESES 2026-09-04 03:12:27 UTC
- [70] api.jtl-cloud.com/erp/v2/graphql: JTL Cloud ERP GraphQL cross-tenant BOLA via client-supplied x-tenant-id header (from art/lead_nemotron3.txt)
- [60] https://api.jtl-cloud.com/erp/v2/graphql: Cross-tenant BOLA via client-supplied x-tenant-id on ERP GraphQL (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://auth.jtl-cloud.com/.well-known/openid-configuration — OIDC discovery to enumerate endpoints, grants, device_flow support.
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.jtl-cloud.com/oauth2/.well-known/openid-configuration — read-only Ory Hydra standard discovery path for OAuth/OIDC endpoints, grants, dev
- LEARN: ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, 
- LEARN: ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- LEARN: REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data
- LEARN: ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- LEARN: REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack su
- LEARN: ACCEPTED TARGET @ bountyshop.jtl-software.com: Official bug bounty test shop confirmed live (HTTP 200), JTL-Shop v5.x NOVA template, admin panel at /admin/, con
- LEARN: ACCEPTED TARGET @ api.jtl-cloud.com/erp/v2/graphql: Production GraphQL ERP API live, requires JWT, x-tenant-id header for multi-tenancy
- LEARN: ACCEPTED TARGET @ jtl-software.github.io/devdocs-graphql-playground/: GraphiQL playground live, embedded in developer portal, targets production API
- LEARN: ACCEPTED TARGET @ developer.jtl-software.com/cloud/api-reference/graphql-playground: Embeds playground iframe pointing to api.jtl-cloud.com/erp/v2/graphql
- LEARN: ACCEPTED AUTH: OIDC discovery endpoints return 404 on api.jtl-cloud.com — OAuth may use Ory non-standard paths (/oauth2/, /hydra/, /.ory/)
- LEARN: REJECTED OTHER @ bountyshop: SSTI via contact form nachricht field — payload reflected not executed; version unconfirmed; requires backend email trigger unobser
