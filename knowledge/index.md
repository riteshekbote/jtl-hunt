# Knowledge Base (seed)
- 2026-09-03 ACCEPTED MISCONFIG @ jtl-shop:a-b-4db87dad: Cross-platform shared test profiles (37 instances) create systemic risk — one misconfig replicates across JTL-Shop, Shopware6, WooCommerce
- 2026-09-03 ACCEPTED AUTH @ jtl-shop:p-g-443d1d50: OAuth redirect_uri validation in test/staging environments historically loose; 29-instance profile amplifies impact
- 2026-09-03 REJECTED OTHER @ jtl-shop:f-b-e5fa382e: File upload RCE requires mutating test; confidence <50; program prohibits data modification on live customer data (these may be test envs but unverified)
- 2026-09-03 ACCEPTED: passive DNS/CT enumeration alone insufficient for JTL bug bounty. Main services likely on primary domains.
- 2026-09-03 REJECTED NETWORK @ docker.jtl-software.de: All 300 container hosts resolve to single IP 31.172.91.250 but TCP 80/443 timeout — wildcard DNS masks true attack surface; test containers not internet-routable
