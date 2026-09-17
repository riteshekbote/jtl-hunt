## REPOSCAN 2026-09-17 21:45:00 UTC
Scanned 60 public repos from github.com/jtl-software (53) + github.com/jtl-scx (6) via depth-1 clone + pattern grep.

[HYP] Weak auth token generation via time-based uniqid + SHA1 in connector installers
class: SECRET
asset: connector-oxid4/jtlconnector.php:59; connector-gambio-gx3/src/Installer/Modules/Connector.php:17; connector-modified/src/jtl/Connector/Modified/Installer/Modules/Connector.php:17
confidence: 75
reasoning: Three connector repos generate connector auth tokens and OXID shop passwords via `substr(sha1(uniqid()), 0, 16)`. `uniqid()` is based on current time in microseconds — predictable within a narrow time window. An attacker who knows the installation time can brute-force the token, allowing unauthorized connector API access. The OXID connector at jtlconnector.php:59 generates the shop password the same way.
impact: MEDIUM — authentication bypass for JTL Connector API in affected shop plugins; OXID shop password crackable.
verify_steps: 1) Install connector in OXID/Gambio/Modified. 2) Note installation timestamp. 3) Brute-force `substr(sha1(uniqid()), 0, 16)` in a ±5s window. 4) Passively: confirm pattern at cited lines.

[HYP] SSRF via user-controlled endpoint path on authenticated JTL API proxy
class: SSRF
asset: jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:167; jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:268
confidence: 78
reasoning: Both demo backends expose `/erp-info/:tenantId/:endpoint` which constructs `https://api${Environment}.jtl-cloud.com/erp/${endpoint}` using the attacker-controlled `:endpoint` URL parameter (also overridable via `_endpoint` in POST body). The request is authenticated with the server's own client-credentials JWT. This allows proxying requests to arbitrary JTL Platform API paths (e.g., `/erp/v2/graphql`, admin endpoints) as the server's identity. The hello-world-app additionally allows overriding `_tenantId` via POST body, enabling cross-tenant request forgery.
impact: MEDIUM — authenticated request forgery to JTL Cloud API as the server identity; could expose tenant data or trigger state changes.
verify_steps: 1) Deploy either demo app with valid CLIENT_ID/CLIENT_SECRET. 2) `GET /erp-info/fake-tenantId/v2/graphql` — observe proxied GraphQL response. 3) Passively: confirm at line 167/268 that endpoint variable flows unsanitized into fetch URL.

[HYP] CORS wildcard on authenticated proxy routes
class: MISCONFIG
asset: jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:45; jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:10
confidence: 82
reasoning: Both demo backends call `app.use(cors())` with no origin restriction, enabling cross-origin requests to all endpoints including the authenticated `/graphql` proxy and `/erp-info` SSRF endpoint. A malicious webpage can trigger requests that carry the server's client-credentials JWT, exfiltrating tenant data or proxying arbitrary JTL API calls.
impact: MEDIUM — cross-origin abuse of authenticated JTL Cloud API proxy.
verify_steps: 1) Deploy backend. 2) From a different origin, `fetch('/graphql', {method:'POST', ...})` — confirm CORS headers `Access-Control-Allow-Origin: *`. 3) Passively: line 45/10.

[HYP] Credential logging — clientId and partial clientSecret in console.log
class: SECRET
asset: jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:94-95
confidence: 88
reasoning: Lines 94-95 call `console.log('clientId', clientId)` (full value) and `console.log('clientSecret', ...)` (first 3 + last 3 chars visible). These are OAuth2 client credentials used for server-to-server auth with `auth.jtl-cloud.com`. In containerized deployments with centralized log aggregation (CloudWatch, Datadog, etc.), these secrets persist in logs. The customer-voice backend does NOT have this issue — it only logs when credentials are missing, not the values themselves.
impact: MEDIUM — partial secret exposure via log aggregation; attacker with log access can reconstruct the secret.
verify_steps: 1) Deploy with real CLIENT_ID/CLIENT_SECRET. 2) Check stdout/container logs. 3) Passively: lines 94-95 in the committed file.

[HYP] IDOR — Unauthenticated tenant-scoped feedback writes in Customer-Voice demo app
class: IDOR
asset: jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:74-89
confidence: 85
reasoning: `POST /feedbacks` accepts `tenant_id` directly from `req.body` without any authentication or authorization check (unlike `GET /feedbacks` which calls `resolveTenantId`). Any caller can inject an arbitrary `tenant_id`, writing feedback under any tenant. No session token or JWT validation is performed on this route. The `customer_id` defaults to `"anonymous"` if omitted.
impact: MEDIUM — data integrity violation across tenants; attacker can pollute any tenant's feedback.
verify_steps: 1) Deploy customer-voice backend. 2) POST to `/feedbacks` with `{"tenant_id":"cv-evil","article_id":"x","rating":5}` — expect 201. 3) GET `/feedbacks` with a legitimate session token for that tenant — see injected record. Passively: confirm route at line 74 has no auth middleware.

[HYP] Developer logging enabled in committed OpenCart connector config
class: MISCONFIG
asset: connector-opencart2/jtlconnector/config/config.json:2
confidence: 62
reasoning: The config.json shipped with the OpenCart connector has `"developer_logging": true`. If deployed as-is to production, verbose debug logs may be written to disk, potentially exposing sensitive request/response data including authentication tokens and PII.
impact: LOW — debug data exposure if shipped to production without override.
verify_steps: 1) Deploy OpenCart connector. 2) Check for verbose log output in var/log. 3) Passively: confirm line 2 of config.json.

[HYP] Insecure PHP unserialize on database-derived data across multiple connectors
class: MISCONFIG
asset: connector-shopware5/src/Controller/CustomerOrder.php:465; connector-woocommerce3/src/Controllers/Product/ProductAttrController.php:482; connector-woocommerce3/src/Controllers/CrossSellingController.php:50; connector-opencart2/jtlconnector/src/jtl/Connector/OpenCart/Utility/CustomField.php:62
confidence: 55
reasoning: Multiple connectors call `unserialize()` on values read from the shop database (order attributes, product meta values, custom fields). While the data source is the shop's own DB (limiting exploitability), if the DB is compromised or the shop allows customer-controlled serialized data (e.g., Magento custom options), PHP object injection could lead to remote code execution.
impact: LOW-MEDIUM — conditional RCE if database is compromised or customer-controlled serialized data flows to these paths.
verify_steps: 1) Confirm which DB columns contain serialized PHP objects. 2) Trace if customer input can influence these columns. 3) Passively: confirm `unserialize()` at cited lines.

[HYP] Hardcoded RabbitMQ guest credentials in nachricht library examples and test fixtures
class: SECRET
asset: nachricht/examples/common/service.yaml:28-29; nachricht/tests/Integration/Fixtures/RabbitMqManagementClient.php:19-20
confidence: 45
reasoning: The nachricht messaging library ships example config and test fixtures with hardcoded `user: 'guest'` / `password: 'guest'` for RabbitMQ. These are well-known default credentials. Risk is LOW because these are example/test files, not production config — but developers may copy them to production.
impact: LOW — default credentials in example code; risk only if deployed without credential rotation.
verify_steps: 1) Check if any production deployment uses these example configs. 2) Passively: confirm hardcoded values at cited lines.
## REPOSCAN 2026-09-17 21:46:45 UTC
[HYP] Weak auth token generation via time-based uniqid + SHA1 in connector installers
class: SECRET
asset: connector-oxid4/jtlconnector.php:59; connector-gambio-gx3/src/Installer/Modules/Connector.php:17; connector-modified/src/jtl/Connector/Modified/Installer/Modules/Connector.php:17
confidence: 75
reasoning: Three connector repos generate connector auth tokens and OXID shop passwords via `substr(sha1(uniqid()), 0, 16)`. `uniqid()` is based on current time in microseconds — predictable within a narrow time window. An attacker who knows the installation time can brute-force the token, allowing unauthorized connector API access. The OXID connector at jtlconnector.php:59 generates the shop password the same way.
impact: MEDIUM — authentication bypass for JTL Connector API in affected shop plugins; OXID shop password crackable.
verify_steps: 1) Install connector in OXID/Gambio/Modified. 2) Note installation timestamp. 3) Brute-force `substr(sha1(uniqid()), 0, 16)` in a ±5s window. 4) Passively: confirm pattern at cited lines.
[HYP] SSRF via user-controlled endpoint path on authenticated JTL API proxy
class: SSRF
asset: jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:167; jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:268
confidence: 78
reasoning: Both demo backends expose `/erp-info/:tenantId/:endpoint` which constructs `https://api${Environment}.jtl-cloud.com/erp/${endpoint}` using the attacker-controlled `:endpoint` URL parameter (also overridable via `_endpoint` in POST body). The request is authenticated with the server's own client-credentials JWT. This allows proxying requests to arbitrary JTL Platform API paths (e.g., `/erp/v2/graphql`, admin endpoints) as the server's identity. The hello-world-app additionally allows overriding `_tenantId` via POST body, enabling cross-tenant request forgery.
impact: MEDIUM — authenticated request forgery to JTL Cloud API as the server identity; could expose tenant data or trigger state changes.
verify_steps: 1) Deploy either demo app with valid CLIENT_ID/CLIENT_SECRET. 2) `GET /erp-info/fake-tenantId/v2/graphql` — observe proxied GraphQL response. 3) Passively: confirm at line 167/268 that endpoint variable flows unsanitized into fetch URL.
[HYP] CORS wildcard on authenticated proxy routes
class: MISCONFIG
asset: jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:45; jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:10
confidence: 82
reasoning: Both demo backends call `app.use(cors())` with no origin restriction, enabling cross-origin requests to all endpoints including the authenticated `/graphql` proxy and `/erp-info` SSRF endpoint. A malicious webpage can trigger requests that carry the server's client-credentials JWT, exfiltrating tenant data or proxying arbitrary JTL API calls.
impact: MEDIUM — cross-origin abuse of authenticated JTL Cloud API proxy.
verify_steps: 1) Deploy backend. 2) From a different origin, `fetch('/graphql', {method:'POST', ...})` — confirm CORS headers `Access-Control-Allow-Origin: *`. 3) Passively: line 45/10.
[HYP] Credential logging — clientId and partial clientSecret in console.log
class: SECRET
asset: jtl-platform-app-samples/src/hello-world-app/packages/backend/src/index.ts:94-95
confidence: 88
reasoning: Lines 94-95 call `console.log('clientId', clientId)` (full value) and `console.log('clientSecret', ...)` (first 3 + last 3 chars visible). These are OAuth2 client credentials used for server-to-server auth with `auth.jtl-cloud.com`. In containerized deployments with centralized log aggregation (CloudWatch, Datadog, etc.), these secrets persist in logs. The customer-voice backend does NOT have this issue — it only logs when credentials are missing, not the values themselves.
impact: MEDIUM — partial secret exposure via log aggregation; attacker with log access can reconstruct the secret.
verify_steps: 1) Deploy with real CLIENT_ID/CLIENT_SECRET. 2) Check stdout/container logs. 3) Passively: lines 94-95 in the committed file.
[HYP] IDOR — Unauthenticated tenant-scoped feedback writes in Customer-Voice demo app
class: IDOR
asset: jtl-platform-demo-pacon-2026/customer-voice/packages/backend/src/index.ts:74-89
confidence: 85
reasoning: `POST /feedbacks` accepts `tenant_id` directly from `req.body` without any authentication or authorization check (unlike `GET /feedbacks` which calls `resolveTenantId`). Any caller can inject an arbitrary `tenant_id`, writing feedback under any tenant. No session token or JWT validation is performed on this route. The `customer_id` defaults to `"anonymous"` if omitted.
impact: MEDIUM — data integrity violation across tenants; attacker can pollute any tenant's feedback.
verify_steps: 1) Deploy customer-voice backend. 2) POST to `/feedbacks` with `{"tenant_id":"cv-evil","article_id":"x","rating":5}` — expect 201. 3) GET `/feedbacks` with a legitimate session token for that tenant — see injected record. Passively: confirm route at line 74 has no auth middleware.
[HYP] Developer logging enabled in committed OpenCart connector config
class: MISCONFIG
asset: connector-opencart2/jtlconnector/config/config.json:2
confidence: 62
reasoning: The config.json shipped with the OpenCart connector has `"developer_logging": true`. If deployed as-is to production, verbose debug logs may be written to disk, potentially exposing sensitive request/response data including authentication tokens and PII.
impact: LOW — debug data exposure if shipped to production without override.
verify_steps: 1) Deploy OpenCart connector. 2) Check for verbose log output in var/log. 3) Passively: confirm line 2 of config.json.
[HYP] Insecure PHP unserialize on database-derived data across multiple connectors
class: MISCONFIG
asset: connector-shopware5/src/Controller/CustomerOrder.php:465; connector-woocommerce3/src/Controllers/Product/ProductAttrController.php:482; connector-woocommerce3/src/Controllers/CrossSellingController.php:50; connector-opencart2/jtlconnector/src/jtl/Connector/OpenCart/Utility/CustomField.php:62
confidence: 55
reasoning: Multiple connectors call `unserialize()` on values read from the shop database (order attributes, product meta values, custom fields). While the data source is the shop's own DB (limiting exploitability), if the DB is compromised or the shop allows customer-controlled serialized data (e.g., Magento custom options), PHP object injection could lead to remote code execution.
impact: LOW-MEDIUM — conditional RCE if database is compromised or customer-controlled serialized data flows to these paths.
verify_steps: 1) Confirm which DB columns contain serialized PHP objects. 2) Trace if customer input can influence these columns. 3) Passively: confirm `unserialize()` at cited lines.
[HYP] Hardcoded RabbitMQ guest credentials in nachricht library examples and test fixtures
class: SECRET
asset: nachricht/examples/common/service.yaml:28-29; nachricht/tests/Integration/Fixtures/RabbitMqManagementClient.php:19-20
confidence: 45
reasoning: The nachricht messaging library ships example config and test fixtures with hardcoded `user: 'guest'` / `password: 'guest'` for RabbitMQ. These are well-known default credentials. Risk is LOW because these are example/test files, not production config — but developers may copy them to production.
impact: LOW — default credentials in example code; risk only if deployed without credential rotation.
verify_steps: 1) Check if any production deployment uses these example configs. 2) Passively: confirm hardcoded values at cited lines.
TARGET_ORG not configured for jtl; skipping public-org deep scan.
