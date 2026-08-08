# LEADS bigpickle (seed)
- SEED: no model output yet; pipeline starts on first run.
## 2026-08-07 18:34:07 UTC [chat] (model bigpickle)
class: BUSLOGIC
asset: apip.threema.ch/identity/ws/revoke
confidence: 45
reasoning: OpenAPI (directory.openapi.yml) shows /identity/ws/revoke is the ONLY identity endpoint with a single-step request (identity + revocationKey = SHA256(revocation-password)[:4], 32 bits) and NO challenge-response step, and unlike blob_cred/sfu_cred/work endpoints it documents NO 429 rate-limit response.
evidence_needed: server enforces/omits rate limiting on ws/revoke; error/timing differences per guessed key; whether a correct 4-byte key revokes any ID.
verify_steps: AUTH_HELPED: on a program-provided test ID, POST /identity/ws/revoke with a deliberately wrong revocationKey (observe error shape and timing), repeat small count to test for 429; never target third-party IDs and do not create accounts.
impact: if unrate-limited, brute-force of 2^32 space → force-revoke/delete any Threema ID (permanent identity destruction / DoS). Severity: medium-high.
testability: AUTH_HELPED
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of in-scope work.threema.ch)
confidence: 50
reasoning: directory.openapi.yml line 1172 states "/identities ... currently buggy. See TWRK-1633"; endpoint returns "a subset of the provided contacts that are part of the same Work subscription" + work properties (first/last name, jobTitle, department, availability); GET / returns 401 confirming a live credential-gated backend.
evidence_needed: whether contact matching can be induced to return contacts outside the caller's subscription.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities (contacts:[...]) mixing own- and other-subscription IDs and compare membership + properties; also probe /directory pagination bounds (page/size, wildcard queries).
impact: cross-subscription disclosure of work-directory metadata (names, titles, departments, availability) → targeted phishing. Severity: medium.
testability: AUTH_HELPED
class: OTHER
asset: mediator-*.threema.ch / rendezvous-*.threema.ch / blob-mirror-*.threema.ch
confidence: 40
reasoning: all sampled shards (0..f) resolve; GET / → uniform 403; shard prefix is derived from the first byte of a public-key (routing, not auth per config.rs) — no passive distinguishing signal observed.
evidence_needed: any shard or path (wss /{prefix}/) responding differently than 403.
verify_steps: PASSIVE: GET https://{rendezvous,mediator,blob-mirror}-{0,7,f}.threema.ch/ and a few /XX/ paths at ≤1rps, compare status/body fingerprints.
impact: none observed; would only indicate a routing/edge misconfiguration. Severity: n/a
testability: PASSIVE
## 2026-08-07 18:50:01 UTC [chat] (model bigpickle)
[HYP] ds-apip.threema.ch directory lookup / unauth enumeration
class: AUTH
asset: ds-apip.threema.ch
confidence: 55
reasoning: directory.openapi.yml lists `ds-apip.*` as the directory API base; production `ds-apip.threema.ch` returns root 403 with `Access-Control-Allow-Origin: *` and allows POST/GET/OPTIONS/DELETE cross-origin. apip.threema.ch returns 404 on all openapi subpaths (host mismatch), so any live directory routes live on the ds-apip host. If any lookup route is unauthenticated, CORS-* enables cross-origin probing from an attacker page.
evidence_needed: an endpoint under ds-apip.threema.ch returning ≠403/404 to GET; response-body fingerprints differing from apip.
verify_steps: PASSIVE: GET https://ds-apip.threema.ch/identity/lookup and /directory (compare vs apip 404), GET https://ds-apip-work.threema.ch/identity/lookup (401=route-exists-behind-auth vs 404=no-route), ≤1 rps.
impact: Threema ID/pubkey enumeration → spam, phishing, targeted abuse. Severity: medium.
testability: PASSIVE
[HYP] threema-desktop key-storage KDF parameter weakness
class: OTHER
asset: github.com/threema-ch/threema-desktop (apps/desktop/src/common/node/key-storage/crypto.ts)
confidence: 45
reasoning: reposcan found crypto.ts:223 uses fixed benchmark password `r3gGN9GDQ5NF6tM6` to calibrate Argon2id params at runtime, and test-data.ts defines `keyStoragePassword: 'CHANGE_ME'`; no committed credentials exist, so only the KDF/key-handling logic can be weak. Electron 2.x stores user password/keys through this module.
evidence_needed: how Argon2id memory/time params are derived (floors/caps) and whether the derived key protects an offline-readable store.
verify_steps: PASSIVE: static read of crypto.ts `determineKdfParams()` and the storage backend (safeStorage vs plaintext file). No live requests.
impact: if params are attacker-favorable, offline recovery of stored user credentials/keys from disk. Severity: low-medium.
testability: PASSIVE
[HYP] ds-apip-work.threema.ch /identities cross-subscription disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities
confidence: 52
reasoning: directory.openapi.yml:1172 flags "/identities ... currently buggy. See TWRK-1633"; endpoint returns a subset of provided contacts in the same Work subscription plus work properties (first/last name, jobTitle, department, availability). Host now confirmed live returning 401 with permissive CORS.
evidence_needed: whether the membership filter can be induced to return out-of-subscription contacts.
verify_steps: AUTH_HELPED: with authorized Work test license, POST /identities mixing own- and other-subscription IDs and compare membership + property leak; probe pagination bounds (page/size, wildcard).
impact: cross-subscription disclosure of work-directory metadata → targeted phishing. Severity: medium.
testability: AUTH_HELPED
[NEXT] PROBE: GET https://ds-apip-work.threema.ch/identity/lookup and /directory (401 vs 404 distinguishes route-existence behind the auth gate), then GET https://ds-apip.threema.ch/identity/lookup + /directory at ≤1 rps to fingerprint the directory host independently of apip.
## 2026-08-07 19:12:20 UTC [chat] (model bigpickle)
[HYP] ds-apip.test.threema.ch staging directory server publicly exposed
class: MISCONFIG
asset: ds-apip.test.threema.ch
confidence: 60
reasoning: verified 200 on GET /identity/ECHOECHO and /identity/fetch_bulk with same JSON shape and CORS `*` as production ds-apip.threema.ch; hostname is a staging/test env (ds-apip.test.*), yet answers the live identity API with no auth and no documented client reference.
evidence_needed: whether the test host mirrors live production data or an isolated dataset (byte-diff a real ID's publicKey across hosts) and whether it exposes extra routes or an older/newer build (error-shape/header skew).
verify_steps: PASSIVE: GET https://ds-apip.test.threema.ch/identity/ECHOECHO and /identity/fetch_bulk with full headers, diff Server/Date/body fingerprints vs ds-apip.threema.ch; HEAD a few extra paths (/, /identity/lookup, /swagger) at ≤1rps; no third-party IDs.
impact: internet-reachable staging env; version skew, unpatched/test-only routes, or a full unauthenticated mirror of the identity directory. Severity: low-medium.
testability: PASSIVE
[HYP] Unauthenticated bulk identity lookup + CORS `*` enables cross-origin directory enumeration
class: BUSLOGIC
asset: apip.threema.ch/identity/fetch_bulk
confidence: 55
reasoning: GET /identity/{id} returns 200 + publicKey/featureLevel/state/type for existing IDs, 404 otherwise; GET /identity/fetch_bulk → `{"identities":[]}` proves a bulk route parsing an `identities` array; CORS allows Origin `*`, methods POST/GET/OPTIONS/DELETE, header Content-Type — a malicious page can POST bulk lookups cross-origin.
evidence_needed: whether POST fetch_bulk returns records without any token, batch-size caps, and whether the `state` field discriminates active vs revoked/suspended IDs.
verify_steps: AUTH_HELPED: with an authorized test ID set {ECHOECHO active, official revoked/echo test IDs, random-invalid}, POST /identity/fetch_bulk and compare presence/state/type; only official test IDs, no third-party real IDs; GET probes ≤1rps only.
impact: mass existence/state oracle + publicKey harvesting for spam/phishing/profile building; endpoint is by-design public, so impact is scale via bulk+CORS. Severity: low-medium.
testability: AUTH_HELPED
[HYP] ds-apip-work.threema.ch /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities
confidence: 52
reasoning: directory OpenAPI flags /identities "currently buggy" (TWRK-1633); returns same-subscription contacts plus first/last name, jobTitle, department, availability; host answers 401 (route exists behind credential gate, permissive CORS).
evidence_needed: whether the subscription membership filter can be induced (batch/wildcard/pagination bounds) to return out-of-subscription contacts or their work properties.
verify_steps: AUTH_HELPED: with authorized Work test license, POST /identities mixing own- and foreign-subscription IDs, compare membership + property leak; probe page/size bounds.
impact: cross-subscription disclosure of work-directory metadata (names, titles, departments, availability) → targeted phishing. Severity: medium.
testability: AUTH_HELPED
## 2026-08-07 20:01:16 UTC [chat] (model bigpickle)
## 2026-08-07 20:41:53 UTC [chat] (model bigpickle)
[HYP] Safe backup server permissive CORS + unauthenticated probe surface
class: MISCONFIG
asset: safe-01.threema.ch (safe-*.threema.ch)
confidence: 55
reasoning: live nginx, 404 with app-level routing (154b vs 146b bodies), Access-Control-Allow-Origin `*`, Allow-Methods GET,HEAD,PUT,PATCH,POST,DELETE, HSTS+Expect-CT — same permissive-CORS posture as the already-confirmed directory servers. Threema Safe API serves `GET/PUT/DELETE /backup/{id}` where valid id+key → 200 and random → 404; multi-host same backend (safe-1a byte-identical 154b).
evidence_needed: 404-vs-401 distinction on `/backup/{id}` (route exists vs auth-required), and absence of 429/Retry-After on repeated failed lookups → unauthenticated oracle + rate-limit posture.
verify_steps: PASSIVE: 5× GET https://safe-01.threema.ch/backup/{random16} at 1s intervals logging full headers; GET https://safe-01.threema.ch/backup/ + OPTIONS preflight (already 204, CORS \*); compare body bytes vs safe-1a. ≤1 rps, no real IDs.
impact: cross-origin page can probe backup existence and (with leaked backupKey) attempt PUT/DELETE overwrite of user backups; at minimum unauthenticated API surface on the backup service. Severity: low-medium.
testability: PASSIVE
[HYP] chat service hostname discoverable from in-scope client source
class: OTHER
asset: g-*.0.threema.ch (hostname pattern unresolved)
confidence: 45
reasoning: g-1-0…g-9-0 NXDOMAIN while other in-scope hosts resolve; real chat hostname must be defined in threema-android/threema-ios/threema-web connection config (in-scope static-analysis-valid); not resolvable by guesswork.
evidence_needed: a hostname from client config that resolves and answers (TLS/WSS 4xx) under the g-*.0.chat pattern.
verify_steps: RAG: grep threema-android/threema-ios/threema-web source for chat/connection server URL (ServerConfig, g-*.0.threema.ch, wss://), then DNS + single HEAD at ≤1 rps.
impact: unlocks the core in-scope chat surface; severity n/a until hostname confirmed.
testability: PASSIVE
[HYP] mediator/rendezvous shared-edge 403 masks service-level routes
class: MISCONFIG
asset: mediator-1.threema.ch / rendezvous-1.threema.ch (203.56.112.247)
confidence: 45
reasoning: both hostnames → identical IP and byte-identical 403 across 4 paths; TLS terminates (403 not TLS error) so an edge nginx fronts both WSS services; per-account `{XX}/` paths are high-entropy and unguessable, so only unauthenticated edge routes are testable.
evidence_needed: any path or Host variant returning ≠403 (health/version/well-known), or cert SAN overlap across mediator/rendezvous/safe IPs.
verify_steps: PASSIVE: HEAD /health /status /version /.well-known/* with default + swapped Host headers at ≤1rps; `openssl s_client` SAN comparison across 203.56.112.247 and 203.56.112.231.
impact: staging/edge exposure or cross-service virtual-host confusion; severity: low.
testability: PASSIVE
## 2026-08-07 21:20:46 UTC [chat] (model bigpickle)
[HYP] Safe backup server permissive CORS + unauthenticated probe surface
class: MISCONFIG
asset: safe-01.threema.ch (safe-*.threema.ch)
confidence: 55
reasoning: live nginx, 404 with app-level routing (154b vs 146b bodies), Access-Control-Allow-Origin `*`, Allow-Methods GET,HEAD,PUT,PATCH,POST,DELETE, HSTS+Expect-CT — same permissive-CORS posture as the already-confirmed directory servers. Threema Safe API serves `GET/PUT/DELETE /backup/{id}` where valid id+key → 200 and random → 404; multi-host same backend (safe-1a byte-identical 154b).
evidence_needed: 404-vs-401 distinction on `/backup/{id}` (route exists vs auth-required), and absence of 429/Retry-After on repeated failed lookups → unauthenticated oracle + rate-limit posture.
verify_steps: PASSIVE: 5× GET https://safe-01.threema.ch/backup/{random16} at 1s intervals logging full headers; GET https://safe-01.threema.ch/backup/ + OPTIONS preflight (already 204, CORS \*); compare body bytes vs safe-1a. ≤1 rps, no real IDs.
impact: cross-origin page can probe backup existence and (with leaked backupKey) attempt PUT/DELETE overwrite of user backups; at minimum unauthenticated API surface on the backup service. Severity: low-medium.
testability: PASSIVE
[HYP] chat service hostname discoverable from in-scope client source
class: OTHER
asset: g-*.0.threema.ch (hostname pattern unresolved)
confidence: 45
reasoning: g-1-0…g-9-0 NXDOMAIN while other in-scope hosts resolve; real chat hostname must be defined in threema-android/threema-ios/threema-web connection config (in-scope static-analysis-valid); not resolvable by guesswork.
evidence_needed: a hostname from client config that resolves and answers (TLS/WSS 4xx) under the g-*.0.chat pattern.
verify_steps: RAG: grep threema-android/threema-ios/threema-web source for chat/connection server URL (ServerConfig, g-*.0.threema.ch, wss://), then DNS + single HEAD at ≤1 rps.
impact: unlocks the core in-scope chat surface; severity n/a until hostname confirmed.
testability: PASSIVE
[HYP] mediator/rendezvous shared-edge 403 masks service-level routes
class: MISCONFIG
asset: mediator-1.threema.ch / rendezvous-1.threema.ch (203.56.112.247)
confidence: 45
reasoning: both hostnames → identical IP and byte-identical 403 across 4 paths; TLS terminates (403 not TLS error) so an edge nginx fronts both WSS services; per-account `{XX}/` paths are high-entropy and unguessable, so only unauthenticated edge routes are testable.
evidence_needed: any path or Host variant returning ≠403 (health/version/well-known), or cert SAN overlap across mediator/rendezvous/safe IPs.
verify_steps: PASSIVE: HEAD /health /status /version /.well-known/* with default + swapped Host headers at ≤1rps; `openssl s_client` SAN comparison across 203.56.112.247 and 203.56.112.231.
impact: staging/edge exposure or cross-service virtual-host confusion; severity: low.
testability: PASSIVE
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive probes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth).
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin.
[LEARN] ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS headers as ds-apip.threema.ch.
[LEARN] ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
[LEARN] ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 80 reason: ds-apip.threema.ch confirmed with public identity oracle + CORS*; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry; api.threema.ch sibling with same CORS
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 55 reason: safe-{XX}.threema.ch/ pattern confirmed; backup service high value but no live access yet
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass
## 2026-08-07 22:06:21 UTC [chat] (model bigpickle)
[NEW] apip.test.threema.ch — staging directory server live: GET/POST /identity/* 200, CORS *, HSTS, Expect-CT
[NEW] ds-apip-work.threema.ch — work directory (prod) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
[NEW] ds-apip-work.test.threema.ch — work directory (staging) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
[NEW] work.test.threema.ch — staging work web app live: 301 /en/login, HSTS, Expect-CT, CSP with *.test.threema.ch refs, Sentry
[NEW] safe-01.threema.ch — backup server live: 404 on /, CORS *, methods GET/HEAD/PUT/PATCH/POST/DELETE, HSTS, Expect-CT
[CHANGED] apip.threema.ch — now confirmed full directory sibling: GET/POST /identity/* 200, identical pubkeys to ds-apip, CORS *
[CHANGED] Production directory servers (ds-apip, apip, api) all lack HSTS/Expect-CT; staging counterparts have both
[PRIO] ds-apip.threema.ch + api.threema.ch + apip.threema.ch (directory cluster), 8.9, attack=10 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] safe-01.threema.ch (backup service), 7.4, attack=8 business=9 tech=7 gate=9 cloud=8 fresh=6
[PRIO] apip.test.threema.ch (staging directory), 7.1, attack=8 business=6 tech=8 gate=10 cloud=6 fresh=8
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] ds-apip-work.threema.ch (work directory prod), 5.8, attack=6 business=7 tech=6 gate=4 cloud=6 fresh=6
[HYP] Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk (also api.threema.ch, apip.threema.ch)
confidence: 95
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth on all 3 production hosts; CORS Access-Control-Allow-Origin: * with POST/GET/OPTIONS/DELETE enables cross-origin exfiltration; 30 sequential POSTs at 1 rps returned all HTTP 200 (no 429/RateLimit headers); invalid IDs silently omitted
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request across all 3 hosts; verify Access-Control-Expose-Headers allows credentialed reads from attacker origin
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); repeat for api.threema.ch and apip.threema.ch; check OPTIONS for Access-Control-Expose-Headers
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon across 3 production endpoints. Severity: High (privacy breach + reconnaissance at scale, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] Safe backup service unauthenticated probe surface with permissive CORS and broad method allowance
class: AUTH
asset: https://safe-01.threema.ch/
confidence: 65
reasoning: safe-01.threema.ch responds with 404 on root but returns CORS Access-Control-Allow-Origin: * and Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE; HSTS/Expect-CT present; backup service likely holds high-value encrypted backups; no authentication observed on root probes
evidence_needed: Discovery of API endpoints (e.g., /api/*, /backup/*, /restore/*) that accept unauthenticated requests and leak backup metadata or allow backup enumeration
verify_steps: PASSIVE: GET https://safe-01.threema.ch/api/ — observe 404; PROBE: OPTIONS https://safe-01.threema.ch/api/ — confirm allowed methods; PASSIVE: enumerate common backup API paths (/v1/backups, /restore, /list) at ≤1 rps
impact: Unauthenticated access to backup metadata or backup enumeration → targeted backup extraction or deletion. Severity: Medium-High (high-value asset, permissive CORS, unknown auth model)
testability: PASSIVE
[HYP] threema-desktop Windows key storage: keystorage.bin and keystorage.password.bin written without ACL restrictions
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/common/node/key-storage/index.ts`
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-protected) written without explicit DACL/ACL; safeStorage (DPAPI) password blob recoverable by any same-user process via CryptUnprotectData
evidence_needed: Confirm Windows file creation path lacks explicit SetFileSecurity/DACL hardening; verify DPAPI blob decryptable by same-user process
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access)
testability: RAG
[PARKED] Safe backup service unauthenticated probe surface: confidence 65 but no concrete vulnerable endpoint discovered yet; only root 404 with permissive CORS; verify_steps speculative for API paths
[FINAL]
[NEXT] PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" — confirm Access-Control-Expose-Headers for credentialed cross-origin exfiltration
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[LEARN] ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 85 reason: ds-apip/api/apip directory cluster confirmed with public identity oracle + bulk fetch + CORS* + no rate limit across 3 production hosts; safe-01 backup service live with permissive CORS; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 70 reason: safe-01.threema.ch confirmed live with permissive CORS and broad methods; backup service high value but API surface unenumerated
[HYP] Staging chat cluster publicly reachable
class: MISCONFIG
asset: g-*.0.test.threema.ch (203.56.114.34)
confidence: 55
reasoning: Android test flavor `CHAT_SERVER_SUFFIX=".0.test.threema.ch"` yields g-{2hex}.0.test.threema.ch, which resolves live (203.56.114.34) with identical naming to prod chat (203.56.112.202/.204); mirrors the already-accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: staging chat host answers 5222/443 TLS differently from prod (accepts standard client hello, distinct cert SANs, or reachable routes), proving a separate unpatched/test env.
verify_steps: PASSIVE: `openssl s_client -connect 203.56.114.34:5222 -servername g-00.0.test.threema.ch` vs prod 203.56.112.202:5222 — compare handshake/cert; DNS sweep of g-XX.0.test.threema.ch; ≤1 rps, single attempts.
impact: internet-reachable staging messenger backend → version skew, test accounts/routes, or full reachable mirror; severity low-medium.
testability: PASSIVE
[HYP] Safe backup API is fully credential-gated (no unauthenticated surface)
class: AUTH
asset: safe-01.threema.ch/backups/{64hex} + /config
confidence: 50
reasoning: both endpoints return 400 for every unauth/no-cred GET (incl. arbitrary Basic and shard-matching IDs) while non-route paths give Express 404; client uses Basic auth derived from backupId+backupKey (SafeApiService); CORS `*` persists on all responses.
evidence_needed: whether any request without valid derived credentials ever returns ≠400 (preflight Allow-Methods vs Allow-Headers, HEAD variants).
verify_steps: PASSIVE: OPTIONS https://safe-01.threema.ch/backups/{hex} (already 204 CORS *) → record Allow-Methods/Allow-Headers; single HEAD /config; ≤1 rps, no real IDs.
impact: no unauth data access observed; CORS `*` + broad methods on credential-gated API is defense-in-depth only; severity low.
testability: PASSIVE
[NEXT] PROBE: `openssl s_client -connect 203.56.114.34:5222 -servername g-00.0.test.threema.ch` and `openssl s_client -connect 203.56.112.202:5222 -servername g-00.0.threema.ch` (compare handshake/cert: staging vs prod chat), then one `curl -sv https://g-00.0.test.threema.ch/` at ≤1 rps.
## 2026-08-07 22:33:29 UTC [chat] (model bigpickle)
[HYP] Staging chat cluster g-*.0.test.threema.ch publicly reachable single-node mirror of prod chat
class: MISCONFIG
asset: https://g-*.0.test.threema.ch (203.56.114.34:5222)
confidence: 65
reasoning: g-{2hex}.0.test.threema.ch all resolve to single IPv4 203.56.114.34 (reverse chliwindgaellen.threema.ch); 5222 OPEN and accepts TCP while prod spreads across 203.56.112.202/.204; staging network 203.56.114.0/24 + IPv6 2a14:3e44:114:314::* confirmed distinct from prod — mirrors the accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: observable behavioral divergence from prod (lenient/broken auth, test identities, version skew) — requires decoding the custom chat protocol
verify_steps: AUTH_HELPED: reconstruct ChatConnection handshake from threema-android (public source, in scope), single passive connect g-00.0.test.threema.ch:5222 vs g-00.0.threema.ch:5222, compare stream/auth response; ≤1 rps, one attempt each
impact: internet-reachable staging chat backend → version skew, test accounts/routes, unpatched-surface; severity low-medium
testability: PASSIVE
[HYP] Staging sync/linking WSS servers publicly resolvable and reachable
class: MISCONFIG
asset: wss://mediator-1.test.threema.ch/{XX}/ , wss://rendezvous-1.test.threema.ch/{XX}/
confidence: 60
reasoning: both resolve to staging IPv6 2a14:3e44:114:314::62 vs prod 2a14:3e40:112:312::247; same staging segment as the accepted ds-apip.test/apip.test exposure; pattern mediator-*.threema.ch/{XX}/ confirmed for prod.
evidence_needed: WSS/TLS handshake completes on staging and yields distinct staging cert SANs
verify_steps: PASSIVE: `openssl s_client -connect [2a14:3e44:114:314::62]:443 -servername mediator-1.test.threema.ch` vs prod `[2a14:3e40:112:312::247]` — compare cert/subject; single connect each, ≤1 rps
impact: reachable staging multi-device pairing/sync → test device-pairing surface, pairing-token recon; severity low-medium
testability: PASSIVE
[PARKED] Prod chat 5222 unauthenticated TCP accept: confidence < 40 — only "accepts TCP" proven; no in-band observable difference from a firewall/load balancer; no concrete verify step without protocol spec.
[FINAL]
[NEXT] RAG: fetch the chat connection handshake/constants from github.com/threema-ch/threema-android (public, in-scope) — `ChatConnection`/`ChatProtocol`/server hostname derivation — to reconstruct the g-*.0.threema.ch:5222 client stream, then single passive connect to staging vs prod to fingerprint version skew (the one fact that turns the staging-chat exposure from "reachable" into "exploitable difference").
## 2026-08-07 23:07:25 UTC [chat] (model bigpickle)
[HYP] Staging mediator/rendezvous WSS surface reachable with staging cert
class: MISCONFIG
asset: wss://mediator-{0..f}.test.threema.ch/{hexproto}/ (203.56.114.62)
confidence: 55
reasoning: staging hosts resolve on staging net with distinct `*.test.threema.ch` cert, mirroring accepted ds-apip.test exposure; prod pattern mediator-{1hex}.threema.ch/{hex ClientUrlInfo} confirmed from source (MediatorUrl.get).
evidence_needed: a staging WSS path that accepts a handshake/token differently than prod, or staging-specific pairing route.
verify_steps: AUTH_HELPED: reconstruct D2M WSS URL from source (need a dgid) — not passive-enumerable; passive TLS/cert comparison already done (identical 403, distinct cert).
impact: reachable staging pairing/sync infra → test device-group surface, token-handling skew; severity low-medium.
testability: AUTH_HELPED
[HYP] Staging chat cluster remains a reachable mirror without observable divergence
class: MISCONFIG
asset: g-*.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: hostname pattern identity-derived (confirmed in source); 5222 TCP-accepting on both staging and prod, both silent, both hold open → only distinguishing feature is DNS/net split, same as accepted staging pattern.
evidence_needed: in-band protocol difference (test identities, version skew) — requires decoding the custom CSP handshake.
verify_steps: AUTH_HELPED: reconstruct CSP handshake (NaCl-box + HKDF) from threema-android `csp/connection`, single connect to staging vs prod, compare stream/auth response; ≤1 rps one attempt each.
impact: internet-reachable staging messenger backend; severity low.
testability: AUTH_HELPED
[HYP] Staging work web app exposes test-account/recon surface
class: MISCONFIG
asset: https://work.test.threema.ch
confidence: 45
reasoning: confirmed live (301 → /en/login, HSTS, Expect-CT, CSP, Sentry) — a full staging cockpit with no observed auth on root; mirrors accepted staging-exposure pattern.
evidence_needed: unauthenticated-visible routes (registration, API base, JS bundles with staging endpoints) differing from prod.
verify_steps: PASSIVE: GET https://work.test.threema.ch/en/login and follow redirect; fetch referenced JS bundle, grep for fetch('/api'|login|register) endpoints; ≤1 rps.
impact: staging work app recon → test credentials, config leaks, version skew; severity low.
testability: PASSIVE
[FINAL] ranked: 1) mediator/rendezvous staging exposure (55), 2) staging chat mirror (50), 3) work.test web app surface (45).
[NEXT] PROBE: `curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}"`, then extract referenced JS bundle URLs and grep for API routes (fetch/login/register/oauth) — fresh reachable staging web surface, fully passive, ≤1 rps.
## 2026-08-07 23:43:30 UTC [chat] (model bigpickle)
[NEW] work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated; IDENTICAL path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence on the work web app.
[NEW] `/api-app/public/license/token/{licenseToken}` route exists in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route present, token lookup fails).
[NEW] work.test.threema.ch login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch (form-action), test.threema.com (img-src).
[NEW] Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict).
[NEW] `/api-app/me/profile` + `/api-app/global/settings` → 302 on staging (session-gated); only the explicit `/api-app/public/*` namespace is open.
[NEW] `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod (no divergence).
[PRIO] work.test.threema.ch /api-app (staging work REST API) score=5.9 | attack=6 business=6 tech=6 gate=6 cloud=3 fresh=8
[PRIO] g-*.0.test.threema.ch:5222 (staging chat) score=5.6 | attack=5 business=7 tech=7 gate=5 cloud=2 fresh=6
[PRIO] safe-01.threema.ch /backups/{id} score=5.3 | attack=5 business=8 tech=5 gate=4 cloud=3 fresh=4
[HYP] Staging work API exposes public endpoints absent on production
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/* (staging)
confidence: 65
reasoning: `/api-app/public/global/settings` returns 200 unauthenticated on staging, 404 on prod work.threema.ch. JS bundle (v2.25.1) defines `/public/license/token/{licenseToken}` (64-char) and `/public/global/settings` under baseUrl `/api-app`; fake token → 404 means route exists. Mirrors accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: full enumeration of `/api-app/public/*` routes and whether any leaks license/user data; compare each route staging vs prod.
verify_steps: PASSIVE: grep all `"/api-app...` + `public` route literals from work_public.js, GET each on staging vs prod work host, diff status codes; ≤1 rps.
impact: unauthenticated staging work API → trial/license/registration recon, staging data leak; severity low.
testability: PASSIVE
[HYP] Staging chat cluster reachable but no in-band divergence proven
class: MISCONFIG
asset: g-{2hex}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: 5222 TCP-accepts on staging (203.56.114.34) and prod (203.56.112.202/.204); hostname pattern identity-derived from source; no in-band protocol difference observable without decoding CSP handshake. Only DNS/net split distinguishes it (already ACCEPTED as MISCONFIG).
evidence_needed: version/test-identity skew in the custom NaCl-box+HKDF CSP handshake.
verify_steps: AUTH_HELPED: reconstruct CSP handshake from threema-android `csp/connection`, one connect each staging vs prod, compare stream response; ≤1 rps.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Safe backup API CORS * is defense-in-depth only
class: MISCONFIG
asset: https://safe-01.threema.ch/backups/{64hex}
confidence: 45
reasoning: all unauth/no-cred GET → 400 (credential-gated); only OPTIONS preflight returns CORS * with Allow-Headers Authorization; non-route path → 404. No unauth data access observed across prior probes.
evidence_needed: whether any request without derived Basic credentials ever returns ≠400.
verify_steps: PASSIVE: OPTIONS + single HEAD /config and /backups/{hex}; record Allow-Methods/Allow-Headers; ≤1 rps, no real IDs.
impact: CORS * on credential-gated API is weak defense-in-depth; severity low.
testability: PASSIVE
[PARKED] safe-01 CORS hypothesis: class/target largely exhausted — prior probes already confirmed credential-gating (400) and route distinction; residual evidence (HEAD variants, Allow-Methods) has near-zero impact.
[FINAL] ranked: 1) work.test /api-app public endpoint divergence (65), 2) staging chat mirror (50).
[NEXT] PROBE: enumerate all `/api-app/*` route literals from /tmp/opencode/worktest.js (bundle v2.25.1), then GET each `public` route on work.test.threema.ch AND work.threema.ch, diff status codes to map the full staging-only public API set; ≤1 rps, GET/HEAD only.
[LEARN] ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API endpoint confirmed.
[LEARN] ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only the `/api-app/public/*` namespace is open (namespace gating confirmed).
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed).
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: liveness endpoints `/ping` (204) and `/info/ping.php` (200 empty) identical on staging and prod — no divergence.
[RISK] chat: 45 — staging chat cluster TCP-reachable and DNS-split from prod but no in-band divergence proven; requires protocol decoding | web: 50 — staging work API exposes public endpoints absent on prod (public/global/settings 200 vs 404); auth-gated namespaces intact; low severity | sync: 40 — staging mediator/rendezvous WSS reachable with distinct staging cert; no token-level surface | safe: 30 — credential-gated (400 for no-creds); CORS * on preflight only; defense-in-depth gap | desktop-src: 40 — two accepted source misconfigs (Windows keystorage ACL, electron sandbox:false + nodeIntegrationInWorker) with no remote exploitation path.
## 2026-08-08 00:15:31 UTC [chat] (model bigpickle)
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: staging chat cluster consolidated to a single IP (203.56.114.34) vs prod's split .112.202/.204; TCP 5222+443 accept but server is silent until client speaks (custom CSP protocol, no TLS cert obtainable). Only DNS/net split currently separates it from prod.
evidence_needed: in-band protocol difference in the NaCl-box+HKDF CSP handshake (stream/auth response fields, version, test identity acceptance).
verify_steps: AUTH_HELPED: reconstruct CSP handshake from threema-android `csp/connection` code; one connect to staging and one to prod; compare stream/auth responses; ≤1 rps one attempt each.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging sync cluster (203.56.114.62) reachable beyond nginx gate
class: MISCONFIG
asset: mediator-{0..f}.test.threema.ch / rendezvous-{0..f}.test.threema.ch / blob-mirror-{0..f}.test.threema.ch
confidence: 40
reasoning: all staging sync hosts consolidate to one IP 203.56.114.62 with wildcard `*.test.threema.ch` cert; every HTTP path AND a valid-format ClientUrlInfo WS upgrade (hex from `md-d2m.proto`) returns nginx 403 — gate is host/path-agnostic. No divergence from prod 403 behavior observed.
evidence_needed: whether any Host/path combination (real deviceGroupId, server group ≠ "0", direct IP + SNI variants) bypasses the nginx 403 to reach the WSS handler.
verify_steps: AUTH_HELPED: WS upgrade with real DGPK-derived ClientUrlInfo hex on staging vs prod; ≤1 rps, no handshake payload sent.
impact: staging sync/linking/blob backend exposure → test credential recon; severity low.
testability: AUTH_HELPED
[HYP] Staging work bundle/backend license-route skew
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 40
reasoning: bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT (void) for `/public/license/token/{64hex}` with 64-char token validation, but the staging backend returns SPA catch-all 404 (md5-identical to bogus paths) — frontend ahead of backend.
evidence_needed: whether backend catches up and serves the route (unauthenticated token→credentials oracle); currently non-live.
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64hex}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token oracle returning license username/password; severity low (tokens are high-entropy).
testability: PASSIVE
[NEXT] AUTH_HELPED: with an authorized staging test identity, reconstruct the Threema CSP NaCl-box+HKDF handshake from threema-android (csp/connection) and perform one connect to `g-00.0.test.threema.ch:5222` (203.56.114.34) vs one to `g-00.0.threema.ch:5222` (203.56.112.202); compare stream/auth response fields for version or test-identity skew; ≤1 rps, one attempt each, no data modification.
[RISK] chat: 40 — staging chat cluster TCP-reachable on 5222/443 with custom CSP protocol; no in-band divergence proven without handshake decode; version-skew risk unproven | web: 45 — staging work API exposes exactly one public config route (app-download links) absent on prod; auth-gated namespaces intact on both hosts; license-credential route not live on staging | sync: 35 — staging mediator/rendezvous/blob-mirror consolidated to 203.56.114.62 with staging cert, but nginx 403 gates all hosts/paths incl. valid WS upgrade; no reachable surface | safe: 30 — unchanged; credential-gated (400 for no-creds), CORS `*` on preflight only | desktop-src: 40 — two accepted source misconfigs (Windows keystorage ACL, electron sandbox:false + nodeIntegrationInWorker) with no remote exploitation path.
## 2026-08-08 02:19:13 UTC [chat] (model bigpickle)
[LEARN] ACCEPTED MISCONFIG @ work.test.threema.ch bundle divergence: work_public.js v2.25.1 DIFFERENT builds staging vs prod (staging sha256 e48e18f79df0125e8942f8fd9566f0c7924d15d7229377b553cf666b1bca7b87, 1,443,948 B; prod 96501e2139ad9647e578ad1e03befca380d4f82c49434f3f3b13a14f354b67a7, 1,400,541 B); staging implements /api-app/public/global/settings (GET) + /api-app/public/license/token/{licenseToken} (GET returns {username?,password?,expired,hasEmail}; PUT void; zod 64-char token); prod bundle has ZERO /public/* handlers.
[LEARN] ACCEPTED OTHER @ work.threema.ch: prod DOES route /api-app — GET /api-app/me/profile → 302 to /en/login?r=%2Fapi-app%2Fme%2Fprofile; only /public/* namespace absent on prod (404 catch-all) — divergence is public-namespace-specific.
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/license/token/{64hex} → 404 HTML catch-all (900B) for GET, PUT, AND OPTIONS — method-agnostic; backend route not deployed; 64-char validation is client-side-only zod; not an oracle today.
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/* namespace map closed on both hosts — /, /global/, /config, /registration, /license/, /global/app-downloads all 404 staging+prod; sole live public route = /api-app/public/global/settings (200, 299B, only appLinkHost + 3 app-download URLs, no license/user data).
[LEARN] ACCEPTED OTHER @ g-*.0.{test.,}threema.ch chat: read-only TCP connect to 5222 returns 0 bytes on BOTH staging (203.56.114.34) and prod (203.56.112.202) — no server-hello pushed without client frame (source: CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32); 443 also closes without TLS handshake both hosts. No passive in-band divergence obtainable; handshake requires authenticated login frame.
[LEARN] ACCEPTED OTHER @ hcaptcha-work.threema.ch: 200 serving hCaptcha's own Webflow marketing page (Last Published 2026-07-30) — third-party captcha host, out-of-scope service.
[LEARN] ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.test / billing.test → 000 (unreachable, matches prod TIMEOUT).
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties (first/last name, jobTitle, department, availability); host 401s on all paths (route exists behind credential gate) with CORS `*`. Same-version bundle divergence (STEP 1) shows the work build is actively iterating on prod-facing endpoints.
evidence_needed: whether the subscription-membership filter can be induced (batch/wildcard/pagination bounds) to return out-of-subscription contacts or their work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own- and foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory metadata → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing now concrete (CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32; plain Socket, no TLS on 5222; host = prefix + identityStore server-group + suffix). Both staging and prod return 0 bytes to read-only connect — server waits for the client login frame, so only an authenticated frame elicits an in-band response. Only DNS/net split distinguishes staging from prod.
evidence_needed: in-band difference in the CSP server-hello/login-ack stream fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend is ahead of backend — license-token oracle on catch-up
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle (v2.25.1) implements GET (returns license username/password/expired/hasEmail) + PUT for the route with 64-char zod validation, but backend returns method-agnostic 404 catch-all — frontend shipped ahead of backend on staging. Prod bundle lacks the route entirely.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[FINAL] ranked: 1) work /identities cross-subscription (52), 2) staging chat skew (50), 3) work.test license-token catch-up (45).
[NEXT] PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000` and single `GET https://work.test.threema.ch/api-app/public/global/settings`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens). ≤1 rps, GET only. (Chat CSP divergence test is defined but blocked on an authorized staging test identity — AUTH_HELPED.)
## 2026-08-08 03:46:04 UTC [chat] (model bigpickle)
[LEARN] ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.test / billing.test → 000 (unreachable, matches prod TIMEOUT).
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties (first/last name, jobTitle, department, availability); host 401s on all paths (route exists behind credential gate) with CORS `*`. Same-version bundle divergence (STEP 1) shows the work build is actively iterating on prod-facing endpoints.
evidence_needed: whether the subscription-membership filter can be induced (batch/wildcard/pagination bounds) to return out-of-subscription contacts or their work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own- and foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory metadata → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing now concrete (CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32; plain Socket, no TLS on 5222; host = prefix + identityStore server-group + suffix). Both staging and prod return 0 bytes to read-only connect — server waits for the client login frame, so only an authenticated frame elicits an in-band response. Only DNS/net split distinguishes staging from prod.
evidence_needed: in-band difference in the CSP server-hello/login-ack stream fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend is ahead of backend — license-token oracle on catch-up
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle (v2.25.1) implements GET (returns license username/password/expired/hasEmail) + PUT for the route with 64-char zod validation, but backend returns method-agnostic 404 catch-all — frontend shipped ahead of backend on staging. Prod bundle lacks the route entirely.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[FINAL] ranked: 1) work /identities cross-subscription (52), 2) staging chat skew (50), 3) work.test license-token catch-up (45).
[NEXT] PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000` and single `GET https://work.test.threema.ch/api-app/public/global/settings`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens). ≤1 rps, GET only. (Chat CSP divergence test is defined but blocked on an authorized staging test identity — AUTH_HELPED.)
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties; host 401s on all paths (credential-gated) with CORS `*`; both prod+staging bundles frozen at v2.25.1 this cycle, work build still actively iterating.
evidence_needed: whether subscription-membership filter can be induced (batch/wildcard/page bounds) to return out-of-subscription contacts or work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own-/foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory PII → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for client login frame; DNS topology newly mapped: staging is single-IP consolidation of a two-shard prod cluster (boundary 0x80), only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend/backend skew persists — license-token oracle if backend ships
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT for the route; backend still returns method-agnostic 404 catch-all (900B) — re-confirmed this cycle; prod bundle has zero /public/* handlers; both builds frozen (hashes unchanged) so no catch-up in either direction.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[NEXT] PROBE: weekly staging catch-up re-check — `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000`, `GET https://work.test.threema.ch/api-app/public/global/settings`, and re-hash `GET https://work.test.threema.ch/cache/work_public.js`; escalate if license-token ≠404 (never real tokens). Executed this cycle: all three unchanged (404/200/hash-match), next run ~2026-08-15. ≤1 rps, GET only. Chat CSP handshake and /identities tests remain AUTH_HELPED (blocked on an authorized staging test identity).
## 2026-08-08 04:43:34 UTC [chat] (model bigpickle)
class: AUTH
asset: https://broadcast.threema.ch (v2.28.1, csrf_public_login_access)
confidence: 50
reasoning: Fresh prod surface (was TIMEOUT). Live login form embeds `csrf_public_login_access`; bundle implements full attestation+assertion ceremonies (challenge POST → `assertion_response` POST). Served without CSP/HSTS. Ceremony endpoint paths are minified/derived and not yet resolved.
evidence_needed: Resolved ceremony endpoint paths; whether challenge endpoint is unauth-reachable; rate limiting on challenge issuance.
verify_steps: PASSIVE: resolve minified endpoint map feeding the `${l}`/`${o}`/`${s}`/`${r}` ceremony URLs in broadcast_public.js; then single GET of each endpoint on https://broadcast.threema.ch, record status + rate-limit headers; full assertion requires a passkey (AUTH_HELPED). ≤1 rps.
impact: unauth passkey-ceremony DoS/abuse; assertion-validation flaw would be account-level — unproven; severity low→high contingent on endpoint reachability.
testability: PASSIVE
class: AUTH
asset: https://gateway.threema.ch (/en/login, /en/signup, /en/forgot-password, /api/*)
confidence: 50
reasoning: Fresh messaging-gateway control plane; login served with NO HSTS/Expect-CT/CSP (weaker posture than shop); `/api/`+`/api/v1/` nginx-403 (routes gated, not absent); `trusted_redirect_origins` field present on login. No staging mirror (gateway.test 403) → no divergence oracle.
evidence_needed: POST targets + CSRF/captcha presence on signup/forgot-password; whether any /api path escapes the nginx deny.
verify_steps: PASSIVE: GET /en/signup and /en/forgot-password, capture CSRF field names/POST targets/captcha; GET a small set of plausible /api paths; do NOT submit POSTs (side effects). ≤1 rps.
impact: unauthenticated surface of a customer-facing messaging gateway; credential-page transport hardening gap; severity low (no vuln proven).
testability: PASSIVE
class: BUSLOGIC
asset: https://shop.threema.ch/en/redeem + /en/retrieve-keys
confidence: 45
reasoning: Two unauthenticated POST forms (email+promo_code; email+invoice_ref) on fresh prod; if responses differentiate valid/invalid code+ref with no rate limit they become redemption/receipt oracles. Counter-weight: full HSTS/Expect-CT/CSP + hCaptcha frame — likely captcha-gated.
evidence_needed: whether a GET variant exists; whether response delta leaks code validity without submission.
verify_steps: PASSIVE-first: inspect forms for CSRF/hCaptcha and probe for GET variants; any submission oracle requires one-shot authorized POSTs (AUTH_HELPED). ≤1 rps.
impact: promo-code/invoice enumeration → discount or PII; severity low (high-entropy codes, captcha likely).
testability: AUTH_HELPED
## 2026-08-08 05:24:57 UTC [chat] (model bigpickle)
evidence_needed: Confirm Windows file creation path lacks explicit SetFileSecurity/DACL hardening; verify DPAPI blob decryptable by same-user process
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access)
testability: RAG
[PARKED] Safe backup service unauthenticated probe surface: confidence 65 but no concrete vulnerable endpoint discovered yet; only root 404 with permissive CORS; verify_steps speculative for API paths
[FINAL]
[NEXT] PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" — confirm Access-Control-Expose-Headers for credentialed cross-origin exfiltration
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[LEARN] ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 85 reason: ds-apip/api/apip directory cluster confirmed with public identity oracle + bulk fetch + CORS* + no rate limit across 3 production hosts; safe-01 backup service live with permissive CORS; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 70 reason: safe-01.threema.ch confirmed live with permissive CORS and broad methods; backup service high value but API surface unenumerated
[HYP] Staging chat cluster publicly reachable
class: MISCONFIG
asset: g-*.0.test.threema.ch (203.56.114.34)
confidence: 55
reasoning: Android test flavor `CHAT_SERVER_SUFFIX=".0.test.threema.ch"` yields g-{2hex}.0.test.threema.ch, which resolves live (203.56.114.34) with identical naming to prod chat (203.56.112.202/.204); mirrors the already-accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: staging chat host answers 5222/443 TLS differently from prod (accepts standard client hello, distinct cert SANs, or reachable routes), proving a separate unpatched/test env.
verify_steps: PASSIVE: `openssl s_client -connect 203.56.114.34:5222 -servername g-00.0.test.threema.ch` vs prod 203.56.112.202:5222 — compare handshake/cert; DNS sweep of g-XX.0.test.threema.ch; ≤1 rps, single attempts.
impact: internet-reachable staging messenger backend → version skew, test accounts/routes, or full reachable mirror; severity low-medium.
testability: PASSIVE
[HYP] Safe backup API is fully credential-gated (no unauthenticated surface)
class: AUTH
asset: safe-01.threema.ch/backups/{64hex} + /config
confidence: 50
reasoning: both endpoints return 400 for every unauth/no-cred GET (incl. arbitrary Basic and shard-matching IDs) while non-route paths give Express 404; client uses Basic auth derived from backupId+backupKey (SafeApiService); CORS `*` persists on all responses.
evidence_needed: whether any request without valid derived credentials ever returns ≠400 (preflight Allow-Methods vs Allow-Headers, HEAD variants).
verify_steps: PASSIVE: OPTIONS https://safe-01.threema.ch/backups/{hex} (already 204 CORS *) → record Allow-Methods/Allow-Headers; single HEAD /config; ≤1 rps, no real IDs.
impact: no unauth data access observed; CORS `*` + broad methods on credential-gated API is defense-in-depth only; severity low.
testability: PASSIVE
[NEXT] PROBE: `openssl s_client -connect 203.56.114.34:5222 -servername g-00.0.test.threema.ch` and `openssl s_client -connect 203.56.112.202:5222 -servername g-00.0.threema.ch` (compare handshake/cert: staging vs prod chat), then one `curl -sv https://g-00.0.test.threema.ch/` at ≤1 rps.
[HYP] Staging chat cluster g-*.0.test.threema.ch publicly reachable single-node mirror of prod chat
class: MISCONFIG
asset: https://g-*.0.test.threema.ch (203.56.114.34:5222)
confidence: 65
reasoning: g-{2hex}.0.test.threema.ch all resolve to single IPv4 203.56.114.34 (reverse chliwindgaellen.threema.ch); 5222 OPEN and accepts TCP while prod spreads across 203.56.112.202/.204; staging network 203.56.114.0/24 + IPv6 2a14:3e44:114:314::* confirmed distinct from prod — mirrors the accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: observable behavioral divergence from prod (lenient/broken auth, test identities, version skew) — requires decoding the custom chat protocol
verify_steps: AUTH_HELPED: reconstruct ChatConnection handshake from threema-android (public source, in scope), single passive connect g-00.0.test.threema.ch:5222 vs g-00.0.threema.ch:5222, compare stream/auth response; ≤1 rps, one attempt each
impact: internet-reachable staging chat backend → version skew, test accounts/routes, unpatched-surface; severity low-medium
testability: PASSIVE
[HYP] Staging sync/linking WSS servers publicly resolvable and reachable
class: MISCONFIG
asset: wss://mediator-1.test.threema.ch/{XX}/ , wss://rendezvous-1.test.threema.ch/{XX}/
confidence: 60
reasoning: both resolve to staging IPv6 2a14:3e44:114:314::62 vs prod 2a14:3e40:112:312::247; same staging segment as the accepted ds-apip.test/apip.test exposure; pattern mediator-*.threema.ch/{XX}/ confirmed for prod.
evidence_needed: WSS/TLS handshake completes on staging and yields distinct staging cert SANs
verify_steps: PASSIVE: `openssl s_client -connect [2a14:3e44:114:314::62]:443 -servername mediator-1.test.threema.ch` vs prod `[2a14:3e40:112:312::247]` — compare cert/subject; single connect each, ≤1 rps
impact: reachable staging multi-device pairing/sync → test device-pairing surface, pairing-token recon; severity low-medium
testability: PASSIVE
[PARKED] Prod chat 5222 unauthenticated TCP accept: confidence < 40 — only "accepts TCP" proven; no in-band observable difference from a firewall/load balancer; no concrete verify step without protocol spec.
[FINAL]
[NEXT] RAG: fetch the chat connection handshake/constants from github.com/threema-ch/threema-android (public, in-scope) — `ChatConnection`/`ChatProtocol`/server hostname derivation — to reconstruct the g-*.0.threema.ch:5222 client stream, then single passive connect to staging vs prod to fingerprint version skew (the one fact that turns the staging-chat exposure from "reachable" into "exploitable difference").
[HYP] Staging mediator/rendezvous WSS surface reachable with staging cert
class: MISCONFIG
asset: wss://mediator-{0..f}.test.threema.ch/{hexproto}/ (203.56.114.62)
confidence: 55
reasoning: staging hosts resolve on staging net with distinct `*.test.threema.ch` cert, mirroring accepted ds-apip.test exposure; prod pattern mediator-{1hex}.threema.ch/{hex ClientUrlInfo} confirmed from source (MediatorUrl.get).
evidence_needed: a staging WSS path that accepts a handshake/token differently than prod, or staging-specific pairing route.
verify_steps: AUTH_HELPED: reconstruct D2M WSS URL from source (need a dgid) — not passive-enumerable; passive TLS/cert comparison already done (identical 403, distinct cert).
impact: reachable staging pairing/sync infra → test device-group surface, token-handling skew; severity low-medium.
testability: AUTH_HELPED
[HYP] Staging chat cluster remains a reachable mirror without observable divergence
class: MISCONFIG
asset: g-*.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: hostname pattern identity-derived (confirmed in source); 5222 TCP-accepting on both staging and prod, both silent, both hold open → only distinguishing feature is DNS/net split, same as accepted staging pattern.
evidence_needed: in-band protocol difference (test identities, version skew) — requires decoding the custom CSP handshake.
verify_steps: AUTH_HELPED: reconstruct CSP handshake (NaCl-box + HKDF) from threema-android `csp/connection`, single connect to staging vs prod, compare stream/auth response; ≤1 rps one attempt each.
impact: internet-reachable staging messenger backend; severity low.
testability: AUTH_HELPED
[HYP] Staging work web app exposes test-account/recon surface
class: MISCONFIG
asset: https://work.test.threema.ch
confidence: 45
reasoning: confirmed live (301 → /en/login, HSTS, Expect-CT, CSP, Sentry) — a full staging cockpit with no observed auth on root; mirrors accepted staging-exposure pattern.
evidence_needed: unauthenticated-visible routes (registration, API base, JS bundles with staging endpoints) differing from prod.
verify_steps: PASSIVE: GET https://work.test.threema.ch/en/login and follow redirect; fetch referenced JS bundle, grep for fetch('/api'|login|register) endpoints; ≤1 rps.
impact: staging work app recon → test credentials, config leaks, version skew; severity low.
testability: PASSIVE
[FINAL] ranked: 1) mediator/rendezvous staging exposure (55), 2) staging chat mirror (50), 3) work.test web app surface (45).
[NEXT] PROBE: `curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}"`, then extract referenced JS bundle URLs and grep for API routes (fetch/login/register/oauth) — fresh reachable staging web surface, fully passive, ≤1 rps.
[NEW] work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated; IDENTICAL path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence on the work web app.
[NEW] `/api-app/public/license/token/{licenseToken}` route exists in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route present, token lookup fails).
[NEW] work.test.threema.ch login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch (form-action), test.threema.com (img-src).
[NEW] Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict).
[NEW] `/api-app/me/profile` + `/api-app/global/settings` → 302 on staging (session-gated); only the explicit `/api-app/public/*` namespace is open.
[NEW] `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod (no divergence).
[PRIO] work.test.threema.ch /api-app (staging work REST API) score=5.9 | attack=6 business=6 tech=6 gate=6 cloud=3 fresh=8
[PRIO] g-*.0.test.threema.ch:5222 (staging chat) score=5.6 | attack=5 business=7 tech=7 gate=5 cloud=2 fresh=6
[PRIO] safe-01.threema.ch /backups/{id} score=5.3 | attack=5 business=8 tech=5 gate=4 cloud=3 fresh=4
[HYP] Staging work API exposes public endpoints absent on production
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/* (staging)
confidence: 65
reasoning: `/api-app/public/global/settings` returns 200 unauthenticated on staging, 404 on prod work.threema.ch. JS bundle (v2.25.1) defines `/public/license/token/{licenseToken}` (64-char) and `/public/global/settings` under baseUrl `/api-app`; fake token → 404 means route exists. Mirrors accepted ds-apip.test/apip.test staging-exposure pattern.
evidence_needed: full enumeration of `/api-app/public/*` routes and whether any leaks license/user data; compare each route staging vs prod.
verify_steps: PASSIVE: grep all `"/api-app...` + `public` route literals from work_public.js, GET each on staging vs prod work host, diff status codes; ≤1 rps.
impact: unauthenticated staging work API → trial/license/registration recon, staging data leak; severity low.
testability: PASSIVE
[HYP] Staging chat cluster reachable but no in-band divergence proven
class: MISCONFIG
asset: g-{2hex}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: 5222 TCP-accepts on staging (203.56.114.34) and prod (203.56.112.202/.204); hostname pattern identity-derived from source; no in-band protocol difference observable without decoding CSP handshake. Only DNS/net split distinguishes it (already ACCEPTED as MISCONFIG).
evidence_needed: version/test-identity skew in the custom NaCl-box+HKDF CSP handshake.
verify_steps: AUTH_HELPED: reconstruct CSP handshake from threema-android `csp/connection`, one connect each staging vs prod, compare stream response; ≤1 rps.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Safe backup API CORS * is defense-in-depth only
class: MISCONFIG
asset: https://safe-01.threema.ch/backups/{64hex}
confidence: 45
reasoning: all unauth/no-cred GET → 400 (credential-gated); only OPTIONS preflight returns CORS * with Allow-Headers Authorization; non-route path → 404. No unauth data access observed across prior probes.
evidence_needed: whether any request without derived Basic credentials ever returns ≠400.
verify_steps: PASSIVE: OPTIONS + single HEAD /config and /backups/{hex}; record Allow-Methods/Allow-Headers; ≤1 rps, no real IDs.
impact: CORS * on credential-gated API is weak defense-in-depth; severity low.
testability: PASSIVE
[PARKED] safe-01 CORS hypothesis: class/target largely exhausted — prior probes already confirmed credential-gating (400) and route distinction; residual evidence (HEAD variants, Allow-Methods) has near-zero impact.
[FINAL] ranked: 1) work.test /api-app public endpoint divergence (65), 2) staging chat mirror (50).
[NEXT] PROBE: enumerate all `/api-app/*` route literals from /tmp/opencode/worktest.js (bundle v2.25.1), then GET each `public` route on work.test.threema.ch AND work.threema.ch, diff status codes to map the full staging-only public API set; ≤1 rps, GET/HEAD only.
[LEARN] ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API endpoint confirmed.
[LEARN] ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only the `/api-app/public/*` namespace is open (namespace gating confirmed).
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed).
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: liveness endpoints `/ping` (204) and `/info/ping.php` (200 empty) identical on staging and prod — no divergence.
[RISK] chat: 45 — staging chat cluster TCP-reachable and DNS-split from prod but no in-band divergence proven; requires protocol decoding | web: 50 — staging work API exposes public endpoints absent on prod (public/global/settings 200 vs 404); auth-gated namespaces intact; low severity | sync: 40 — staging mediator/rendezvous WSS reachable with distinct staging cert; no token-level surface | safe: 30 — credential-gated (400 for no-creds); CORS * on preflight only; defense-in-depth gap | desktop-src: 40 — two accepted source misconfigs (Windows keystorage ACL, electron sandbox:false + nodeIntegrationInWorker) with no remote exploitation path.
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: staging chat cluster consolidated to a single IP (203.56.114.34) vs prod's split .112.202/.204; TCP 5222+443 accept but server is silent until client speaks (custom CSP protocol, no TLS cert obtainable). Only DNS/net split currently separates it from prod.
evidence_needed: in-band protocol difference in the NaCl-box+HKDF CSP handshake (stream/auth response fields, version, test identity acceptance).
verify_steps: AUTH_HELPED: reconstruct CSP handshake from threema-android `csp/connection` code; one connect to staging and one to prod; compare stream/auth responses; ≤1 rps one attempt each.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging sync cluster (203.56.114.62) reachable beyond nginx gate
class: MISCONFIG
asset: mediator-{0..f}.test.threema.ch / rendezvous-{0..f}.test.threema.ch / blob-mirror-{0..f}.test.threema.ch
confidence: 40
reasoning: all staging sync hosts consolidate to one IP 203.56.114.62 with wildcard `*.test.threema.ch` cert; every HTTP path AND a valid-format ClientUrlInfo WS upgrade (hex from `md-d2m.proto`) returns nginx 403 — gate is host/path-agnostic. No divergence from prod 403 behavior observed.
evidence_needed: whether any Host/path combination (real deviceGroupId, server group ≠ "0", direct IP + SNI variants) bypasses the nginx 403 to reach the WSS handler.
verify_steps: AUTH_HELPED: WS upgrade with real DGPK-derived ClientUrlInfo hex on staging vs prod; ≤1 rps, no handshake payload sent.
impact: staging sync/linking/blob backend exposure → test credential recon; severity low.
testability: AUTH_HELPED
[HYP] Staging work bundle/backend license-route skew
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 40
reasoning: bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT (void) for `/public/license/token/{64hex}` with 64-char token validation, but the staging backend returns SPA catch-all 404 (md5-identical to bogus paths) — frontend ahead of backend.
evidence_needed: whether backend catches up and serves the route (unauthenticated token→credentials oracle); currently non-live.
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64hex}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token oracle returning license username/password; severity low (tokens are high-entropy).
testability: PASSIVE
[NEXT] AUTH_HELPED: with an authorized staging test identity, reconstruct the Threema CSP NaCl-box+HKDF handshake from threema-android (csp/connection) and perform one connect to `g-00.0.test.threema.ch:5222` (203.56.114.34) vs one to `g-00.0.threema.ch:5222` (203.56.112.202); compare stream/auth response fields for version or test-identity skew; ≤1 rps, one attempt each, no data modification.
[RISK] chat: 40 — staging chat cluster TCP-reachable on 5222/443 with custom CSP protocol; no in-band divergence proven without handshake decode; version-skew risk unproven | web: 45 — staging work API exposes exactly one public config route (app-download links) absent on prod; auth-gated namespaces intact on both hosts; license-credential route not live on staging | sync: 35 — staging mediator/rendezvous/blob-mirror consolidated to 203.56.114.62 with staging cert, but nginx 403 gates all hosts/paths incl. valid WS upgrade; no reachable surface | safe: 30 — unchanged; credential-gated (400 for no-creds), CORS `*` on preflight only | desktop-src: 40 — two accepted source misconfigs (Windows keystorage ACL, electron sandbox:false + nodeIntegrationInWorker) with no remote exploitation path.
[LEARN] ACCEPTED MISCONFIG @ work.test.threema.ch bundle divergence: work_public.js v2.25.1 DIFFERENT builds staging vs prod (staging sha256 e48e18f79df0125e8942f8fd9566f0c7924d15d7229377b553cf666b1bca7b87, 1,443,948 B; prod 96501e2139ad9647e578ad1e03befca380d4f82c49434f3f3b13a14f354b67a7, 1,400,541 B); staging implements /api-app/public/global/settings (GET) + /api-app/public/license/token/{licenseToken} (GET returns {username?,password?,expired,hasEmail}; PUT void; zod 64-char token); prod bundle has ZERO /public/* handlers.
[LEARN] ACCEPTED OTHER @ work.threema.ch: prod DOES route /api-app — GET /api-app/me/profile → 302 to /en/login?r=%2Fapi-app%2Fme%2Fprofile; only /public/* namespace absent on prod (404 catch-all) — divergence is public-namespace-specific.
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/license/token/{64hex} → 404 HTML catch-all (900B) for GET, PUT, AND OPTIONS — method-agnostic; backend route not deployed; 64-char validation is client-side-only zod; not an oracle today.
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/* namespace map closed on both hosts — /, /global/, /config, /registration, /license/, /global/app-downloads all 404 staging+prod; sole live public route = /api-app/public/global/settings (200, 299B, only appLinkHost + 3 app-download URLs, no license/user data).
[LEARN] ACCEPTED OTHER @ g-*.0.{test.,}threema.ch chat: read-only TCP connect to 5222 returns 0 bytes on BOTH staging (203.56.114.34) and prod (203.56.112.202) — no server-hello pushed without client frame (source: CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32); 443 also closes without TLS handshake both hosts. No passive in-band divergence obtainable; handshake requires authenticated login frame.
[LEARN] ACCEPTED OTHER @ hcaptcha-work.threema.ch: 200 serving hCaptcha's own Webflow marketing page (Last Published 2026-07-30) — third-party captcha host, out-of-scope service.
[LEARN] ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.test / billing.test → 000 (unreachable, matches prod TIMEOUT).
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties (first/last name, jobTitle, department, availability); host 401s on all paths (route exists behind credential gate) with CORS `*`. Same-version bundle divergence (STEP 1) shows the work build is actively iterating on prod-facing endpoints.
evidence_needed: whether the subscription-membership filter can be induced (batch/wildcard/pagination bounds) to return out-of-subscription contacts or their work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own- and foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory metadata → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing now concrete (CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32; plain Socket, no TLS on 5222; host = prefix + identityStore server-group + suffix). Both staging and prod return 0 bytes to read-only connect — server waits for the client login frame, so only an authenticated frame elicits an in-band response. Only DNS/net split distinguishes staging from prod.
evidence_needed: in-band difference in the CSP server-hello/login-ack stream fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend is ahead of backend — license-token oracle on catch-up
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle (v2.25.1) implements GET (returns license username/password/expired/hasEmail) + PUT for the route with 64-char zod validation, but backend returns method-agnostic 404 catch-all — frontend shipped ahead of backend on staging. Prod bundle lacks the route entirely.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[FINAL] ranked: 1) work /identities cross-subscription (52), 2) staging chat skew (50), 3) work.test license-token catch-up (45).
[NEXT] PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000` and single `GET https://work.test.threema.ch/api-app/public/global/settings`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens). ≤1 rps, GET only. (Chat CSP divergence test is defined but blocked on an authorized staging test identity — AUTH_HELPED.)
[LEARN] ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.test / billing.test → 000 (unreachable, matches prod TIMEOUT).
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties (first/last name, jobTitle, department, availability); host 401s on all paths (route exists behind credential gate) with CORS `*`. Same-version bundle divergence (STEP 1) shows the work build is actively iterating on prod-facing endpoints.
evidence_needed: whether the subscription-membership filter can be induced (batch/wildcard/pagination bounds) to return out-of-subscription contacts or their work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own- and foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory metadata → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing now concrete (CspSocket reads SERVER_HELLO_LEN=80 then SERVER_LOGIN_ACK_LEN=32; plain Socket, no TLS on 5222; host = prefix + identityStore server-group + suffix). Both staging and prod return 0 bytes to read-only connect — server waits for the client login frame, so only an authenticated frame elicits an in-band response. Only DNS/net split distinguishes staging from prod.
evidence_needed: in-band difference in the CSP server-hello/login-ack stream fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend is ahead of backend — license-token oracle on catch-up
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle (v2.25.1) implements GET (returns license username/password/expired/hasEmail) + PUT for the route with 64-char zod validation, but backend returns method-agnostic 404 catch-all — frontend shipped ahead of backend on staging. Prod bundle lacks the route entirely.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[FINAL] ranked: 1) work /identities cross-subscription (52), 2) staging chat skew (50), 3) work.test license-token catch-up (45).
[NEXT] PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000` and single `GET https://work.test.threema.ch/api-app/public/global/settings`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens). ≤1 rps, GET only. (Chat CSP divergence test is defined but blocked on an authorized staging test identity — AUTH_HELPED.)
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties; host 401s on all paths (credential-gated) with CORS `*`; both prod+staging bundles frozen at v2.25.1 this cycle, work build still actively iterating.
evidence_needed: whether subscription-membership filter can be induced (batch/wildcard/page bounds) to return out-of-subscription contacts or work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own-/foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory PII → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for client login frame; DNS topology newly mapped: staging is single-IP consolidation of a two-shard prod cluster (boundary 0x80), only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend/backend skew persists — license-token oracle if backend ships
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT for the route; backend still returns method-agnostic 404 catch-all (900B) — re-confirmed this cycle; prod bundle has zero /public/* handlers; both builds frozen (hashes unchanged) so no catch-up in either direction.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[NEXT] PROBE: weekly staging catch-up re-check — `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000`, `GET https://work.test.threema.ch/api-app/public/global/settings`, and re-hash `GET https://work.test.threema.ch/cache/work_public.js`; escalate if license-token ≠404 (never real tokens). Executed this cycle: all three unchanged (404/200/hash-match), next run ~2026-08-15. ≤1 rps, GET only. Chat CSP handshake and /identities tests remain AUTH_HELPED (blocked on an authorized staging test identity).
class: AUTH
asset: https://broadcast.threema.ch (v2.28.1, csrf_public_login_access)
confidence: 50
reasoning: Fresh prod surface (was TIMEOUT). Live login form embeds `csrf_public_login_access`; bundle implements full attestation+assertion ceremonies (challenge POST → `assertion_response` POST). Served without CSP/HSTS. Ceremony endpoint paths are minified/derived and not yet resolved.
evidence_needed: Resolved ceremony endpoint paths; whether challenge endpoint is unauth-reachable; rate limiting on challenge issuance.
verify_steps: PASSIVE: resolve minified endpoint map feeding the `${l}`/`${o}`/`${s}`/`${r}` ceremony URLs in broadcast_public.js; then single GET of each endpoint on https://broadcast.threema.ch, record status + rate-limit headers; full assertion requires a passkey (AUTH_HELPED). ≤1 rps.
impact: unauth passkey-ceremony DoS/abuse; assertion-validation flaw would be account-level — unproven; severity low→high contingent on endpoint reachability.
testability: PASSIVE
class: AUTH
asset: https://gateway.threema.ch (/en/login, /en/signup, /en/forgot-password, /api/*)
confidence: 50
reasoning: Fresh messaging-gateway control plane; login served with NO HSTS/Expect-CT/CSP (weaker posture than shop); `/api/`+`/api/v1/` nginx-403 (routes gated, not absent); `trusted_redirect_origins` field present on login. No staging mirror (gateway.test 403) → no divergence oracle.
evidence_needed: POST targets + CSRF/captcha presence on signup/forgot-password; whether any /api path escapes the nginx deny.
verify_steps: PASSIVE: GET /en/signup and /en/forgot-password, capture CSRF field names/POST targets/captcha; GET a small set of plausible /api paths; do NOT submit POSTs (side effects). ≤1 rps.
impact: unauthenticated surface of a customer-facing messaging gateway; credential-page transport hardening gap; severity low (no vuln proven).
testability: PASSIVE
class: BUSLOGIC
asset: https://shop.threema.ch/en/redeem + /en/retrieve-keys
confidence: 45
reasoning: Two unauthenticated POST forms (email+promo_code; email+invoice_ref) on fresh prod; if responses differentiate valid/invalid code+ref with no rate limit they become redemption/receipt oracles. Counter-weight: full HSTS/Expect-CT/CSP + hCaptcha frame — likely captcha-gated.
evidence_needed: whether a GET variant exists; whether response delta leaks code validity without submission.
verify_steps: PASSIVE-first: inspect forms for CSRF/hCaptcha and probe for GET variants; any submission oracle requires one-shot authorized POSTs (AUTH_HELPED). ≤1 rps.
impact: promo-code/invoice enumeration → discount or PII; severity low (high-entropy codes, captcha likely).
testability: AUTH_HELPED
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/common/node/key-storage/index.ts` (fileModeInternalObjectIfPosix) + keystorage.password.bin (DPAPI)
confidence: 75
reasoning: Accepted RAG: fileModeInternalObjectIfPosix() returns {} on Windows, so keystorage.bin (Argon2id-wrapped, key = user master password) and keystorage.password.bin (Electron safeStorage = Windows DPAPI under current user) are written with NO POSIX ACL restriction on disk. On Windows, CryptProtectData is reversible by ANY same-user process — so a co-located malicious app/service running as the same Windows identity can decrypt keystorage.password.bin → recover master key → decrypt keystorage.bin → plaintext identity keypair + message DB.
evidence_needed: Source confirm fileModeInternalObjectIfPosix() path returns {} for Windows (not {mode:0o600}); confirm keystorage.password.bin written with safeStorage/encryption; confirm master password is the Argon2id input to keystorage.bin (no per-install salt/seal beyond the password).
verify_steps: RAG: clone threema-desktop, read `apps/desktop/src/common/node/key-storage/index.ts` around fileModeInternalObjectIfPosix; read crypto.ts KDF flow to confirm master-password→keystorage.bin; confirm safeStorage usage on Windows path.
impact: Local privilege-escalation-style theft: any same-user malware extracts the victim's entire encrypted Threema identity (keypair) + decrypts local message database without the Threema master password. Severity: High (full message identity compromise, no network auth needed).
testability: RAG
[HYP] Desktop BrowserWindow sandbox disabled + nodeIntegrationInWorker → worker-context RCE via message/link content
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 65
reasoning: Accepted RAG: sandbox:false (explicit TODO DEK-79) + nodeIntegrationInWorker:true (TODO DEK-79); nodeIntegration:false + contextIsolation:true set. nodeIntegrationInWorker explicitly grants Node require() to Web Worker contexts; with sandbox off, subprocesses retain wider privileges. If any worker thread processes attacker-controlled content (link preview, attachment decode, message render offloaded to worker), Node APIs (fs/child_process) → RCE.
evidence_needed: Source line numbers for sandbox:false + nodeIntegrationInWorker:true; enumeration of worker spawn sites and content sources (msg-link-preview, attachment, image decode).
verify_steps: RAG: clone threema-desktop, grep `nodeIntegrationInWorker`/`sandbox` in electron-main.ts; grep `new Worker(` across src/ to find content sources; trace message/link handling that feeds a Worker.
impact: Renderer/worker XSS → Node require → OS command execution on the desktop user's machine via a malicious Threema message or link. Severity: High (RCE, no auth beyond receiving a message).
testability: RAG
[NEXT] RAG: clone github.com/threema-ch/threema-desktop, then read `apps/desktop/src/electron/electron-main.ts` BrowserWindow webPreferences to capture exact line numbers for `sandbox`/`nodeIntegrationInWorker`/`nodeIntegration`/`contextIsolation`, and read `apps/desktop/src/common/node/key-storage/index.ts` `fileModeInternalObjectIfPosix` to confirm Windows returns `{}` (no ACL). Also grep `new Worker(` across `apps/desktop/src/` to enumerate worker content sources. This materially advances the desktop POC (top-ranked RAG hypothesis) and resolves the 65+75 confidence desktop leads to verified line-level evidence.
[HYP] Desktop Windows key-storage ACL bypass → master-password recovery → identity keypair + DB decryption
class: MISCONFIG
asset: apps/desktop/src/common/node/{fs,key-storage/index.ts,crypto.ts} + electron-main.ts STORE_USER_PASSWORD (line 944)
confidence: 95
reasoning: `fileModeInternalObjectIfPosix()` (fs.ts:41) returns `{}` on win32 so `keystorage.bin` (_writeOrOverrideFile @ index.ts:559) and `keystorage.password.bin` (electron-main.ts:944-945) are persisted with NO ACL restriction. The latter is encrypted via `electron.safeStorage.encryptString` (electron-main.ts:943) = Windows DPAPI under the current user. `keystorage.bin` outer layer is Argon2id-wrapped by the master password (crypto.ts:53-88), inner layer exposes `identityData.identity` (private key) + `databaseKey` (restore-db.ts:44-47).
evidence_needed: Windows file-object ACL audit of `keystorage.bin` and `keystorage.password.bin` (DACL = blank/default) + reproduction: co-located same-user process calling `CryptUnprotectData` on the password file, then Argon2id(masterPassword)+salt-from-file → XSalsa20 decrypt → identity private key.
verify_steps: RAG (PASSIVE): confirm on-disk DACL — source already proves `mode` omitted on Windows. AUTH_HELPED (same-user only): `powershell Get-Acl` on both files to show no explicit ACE; then `CryptProtect`/`CryptUnprotectData` round-trip via DPAPI on the password blob; then `argon2` node derivation with salt embedded in keystorage.bin outer layer to decrypt the intermediate key storage. ≤1 rps, no remote call.
impact: Any same-user malware/process exfiltrates the victim's full Threema identity (Ed25519 private key) + message-database encryption key → offline decrypt of local message store WITHOUT the Threema master password. No network auth required. Severity: High.
testability: RAG
[HYP] Desktop BrowserWindow sandbox disabled + nodeIntegrationInWorker → worker-context Node escape path
class: MISCONFIG
asset: apps/desktop/src/electron/electron-main.ts (BrowserWindow webPreferences, lines 1234-1268)
confidence: 65
reasoning: `nodeIntegration: false` + `contextIsolation: true` (hardened) BUT `nodeIntegrationInWorker: true` (electron-main.ts:1252, TODO DESK-79) and `sandbox` is never set → Electron default `sandbox=false` (the line-1240 comment "sandboxing is enabled by default" is contradicted by the explicit TODO to enable it). Two workers exist: backend-worker (app/app.ts:407, Threema protocol message/attachment processing) and media-crypto-worker (group-call.ts:281).
evidence_needed: A code path where attacker-influenced message/link/attachment content deserialized or executed inside a Worker reaches a Node `require()` sink enabled by `nodeIntegrationInWorker`.
verify_steps: RAG (PASSIVE): grep `new Worker(`, `require(`, `eval(`, `import(` in apps/desktop/src/worker/** to enumerate worker content sources + Node sinks; trace backend-worker message dispatch for attacker-data parsing (protobuf/URL fetch). Confirm `sandbox` default via Electron docs (false).
impact: If a deserialization/rendering bug exists in the backend-worker processing a malicious Threema message/attachment, `nodeIntegrationInWorker:true` + no sandbox → renderer/worker XSS → Node require → OS command execution. Severity: High (RCE via a message; conditional on secondary bug).
testability: RAG
[HYP] Safe backup cross-origin credential-gated read + enumeration oracle
class: AUTH
asset: https://safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex} (single IP 203.56.112.231)
confidence: 75
reasoning: Prior lead; all 5 hosts return HTTP 400 (not 404) for unauth `GET /backups/{64hex}`, OPTIONS exposes `Access-Control-Allow-Origin: *`, `Access-Control-Allow-Headers: Authorization`, methods incl. PUT/PATCH/POST/DELETE, no 429/RateLimit/Retry-After across 30 probes; route-vs-path distinction (400 on /backups/{hex}, 404 on /backup/{x}) confirms existence oracle.
evidence_needed: (1) 400 body content differs for existing-vs-nonexisting 64-hex id; (2) with program-provided test backupId+backupKey → HTTP 200; (3) `Access-Control-Expose-Headers` on preflight.
verify_steps: PASSIVE: `curl -s -i -m 12 -X OPTIONS https://safe-01.threema.ch/backups/aaaa...00 -H "Origin: https://attacker.ex" -H "Access-Control-Request-Method: GET" -H "Access-Control-Request-Headers: Authorization"`; AUTH_HELPED: `curl -s -i -m 12 -u "${backupId}:${backupKey}" https://safe-01.threema.ch/backups/${backupId}`. ≤1 rps.
impact: Cross-origin backup-ID existence enumeration + credentialed cross-origin reads of the backup store from any attacker origin. Valid credentials → full identity keypair + message-history backup. Severity: High.
testability: PASSIVE + AUTH_HELPED
[PARKED] "Staging mediator/rendezvous WSS surface" (conf 55) — out of POC scope (web tier, not desktop) and requires a dgid for reconstruction; not pursued per phase target=desktop. No verification performed in this round.
[PARKED] "Staging chat cluster 5222 mirror" (conf 50) — out of scope and requires AUTH_HELPED + custom CSP handshake; not pursued.
[PARKED] "Staging work web app surface" (conf 45) — superseded by desktop POC priority; network tier work shifted to lower-ranked tier.
[FINAL] ranked: 1) Desktop Windows key-storage ACL bypass (95, RAG/AUTH_HELPED-local), 2) Safe backup cross-origin credentialed access (75, PASSIVE/AUTH_HELPED), 3) Desktop BrowserWindow sandbox+worker gap (65, RAG).
[NEXT] RAG: Run `grep -rn "require(\|import(\|eval(\|process\.mainModule\|child_process\|fs\." /tmp/opencode/threema-desktop/apps/desktop/src/worker/backend/backend-worker.ts && `grep -rn "sandbox\|webPreferences" /tmp/opencode/threema-desktop/apps/desktop/src/electron/electron-main.ts` to enumerate Node sinks inside the nodeIntegrationInWorker backend-worker content path and definitively confirm `sandbox` is unset (default false) — closes the conditional-exploitability question for the worker RCE lead.
[HYP] <title>
class: <IDOR|SSRF|AUTH|XSS|BUSLOGIC|MISCONFIG|OATH|OTHER>
asset: <host/endpoint>
confidence: <0-100>
reasoning: <facts only, 2-3 lines>
evidence_needed: <what proves it>
verify_steps: <passive-first concrete HTTP requests, or AUTH_HELPED:...>
impact: <what attacker gets + severity>
testability: <PASSIVE|AUTH_HELPED|HUMAN_ONLY>
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
[PRIO] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}, 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] https://ds-apip.threema.ch/identity/{id} + /identity/fetch_bulk + https://api.threema.ch/identity/{id} + https://apip.threema.ch/identity/{id}, 7.8, attack=8 business=8 tech=7 gate=10 cloud=6 fresh=9
[PRIO] threema-desktop `apps/desktop/src/electron/electron-main.ts` BrowserWindow webPreferences, 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[PRIO] https://work.test.threema.ch/api-app/public/*, 7.5, attack=7 business=7 tech=7 gate=10 cloud=6 fresh=10
[PRIO] https://ds-apip-work.threema.ch/identities, 6.8, attack=7 business=8 tech=6 gate=8 cloud=6 fresh=6
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}
confidence: 75
reasoning: OPTIONS returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, Allow-Headers: Authorization — explicitly enabling cross-origin requests with credentials from any origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5 hostnames resolve to same IP; zero 429 observed at 1 rps
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit
class: IDOR
asset: https://ds-apip.threema.ch/identity/{id} + https://api.threema.ch/identity/{id} + https://apip.threema.ch/identity/{id} + /identity/fetch_bulk
confidence: 80
reasoning: All three prod hosts return identical pubkeys for valid IDs via GET /identity/{id} (200/404 oracle) and POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth); CORS * with DELETE/POST/GET/OPTIONS; 30 sequential POSTs at 1 rps all HTTP 200, no 429/RateLimit; invalid IDs silently omitted in bulk response
evidence_needed: Confirm bulk endpoint accepts arbitrary ID lists and returns pubkey map; verify no rate limit at higher volumes (passive 1 rps only); confirm CORS * on all three hosts for both endpoints
verify_steps: PASSIVE: GET https://ds-apip.threema.ch/identity/{valid_id} and {invalid_id} — confirm 200/404; POST https://ds-apip.threema.ch/identity/fetch_bulk with JSON [{id}] — confirm pubkey returned; repeat for api.threema.ch and apip.threema.ch; OPTIONS on each — confirm CORS *; 30 POSTs at 1 rps to fetch_bulk — confirm no 429
impact: Unauthenticated enumeration of all valid Threema IDs + public key extraction at scale across 3 production directory servers; enables targeted attacks, social engineering, metadata correlation. Severity: High (mass identity enumeration, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 70
reasoning: Source confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[PARKED] None dropped — all three hypotheses have confidence ≥ 70, classes not on REJECTED list (REJECTED: MISCONFIG@crypto.ts:223 benchmark dummy, MISCONFIG@desktop OnPrem config trust Ed25519 verified), and all have concrete verify_steps (PASSIVE or RAG).
[FINAL] 1) Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (80) 2) Safe backup cross-origin credentialed access via CORS * + Authorization header (75) 3) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (70)
[NEXT] PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}" — test bulk fetch endpoint on primary directory server for pubkey oracle + rate limit absence
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth) — confirmed via passive probes
[LEARN] ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production
[LEARN] ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization enabling credentialed cross-origin requests
[LEARN] ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encryption
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry; staging work public API divergence confirmed
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
reasoning: staging bundle (v2.25.1) implements GET (returns license username/password/expired/hasEmail) + PUT for the route with 64-char zod validation, but backend returns method-agnostic 404 catch-all — frontend shipped ahead of backend on staging. Prod bundle lacks the route entirely.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
impact: if route enabled: unauthenticated 64-hex license-token→credential oracle; severity low (tokens high-entropy).
testability: PASSIVE
[FINAL] ranked: 1) work /identities cross-subscription (52), 2) staging chat skew (50), 3) work.test license-token catch-up (45).
[NEXT] PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000` and single `GET https://work.test.threema.ch/api-app/public/global/settings`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens). ≤1 rps, GET only. (Chat CSP divergence test is defined but blocked on an authorized staging test identity — AUTH_HELPED.)
[HYP] Work directory /identities cross-subscription metadata disclosure
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml:1172 flags `/identities` "currently buggy" (TWRK-1633); returns same-subscription contacts + work properties; host 401s on all paths (credential-gated) with CORS `*`; both prod+staging bundles frozen at v2.25.1 this cycle, work build still actively iterating.
evidence_needed: whether subscription-membership filter can be induced (batch/wildcard/page bounds) to return out-of-subscription contacts or work properties.
verify_steps: AUTH_HELPED: with authorized work test license, POST /identities mixing own-/foreign-subscription IDs; compare membership + property leak; probe page/size bounds; ≤1 rps.
impact: cross-subscription disclosure of work-directory PII → targeted phishing; severity medium.
testability: AUTH_HELPED
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for client login frame; DNS topology newly mapped: staging is single-IP consolidation of a two-shard prod cluster (boundary 0x80), only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend/backend skew persists — license-token oracle if backend ships
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT for the route; backend still returns method-agnostic 404 catch-all (900B) — re-confirmed this cycle; prod bundle has zero /public/* handlers; both builds frozen (hashes unchanged) so no catch-up in either direction.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
[HYP] Broadcast passkey ceremony endpoints reachable unauthenticated
class: AUTH
asset: https://broadcast.threema.ch (bundle v2.28.1, csrf_public_login_access)
confidence: 50
reasoning: Fresh prod surface (was TIMEOUT). Live login form embeds `csrf_public_login_access`; bundle implements full attestation+assertion ceremonies (challenge POST → `assertion_response` POST); no CSP/HSTS on served pages. Ceremony URL paths are minified/derived (template `${l}`/`${o}`/`${s}`/`${r}`), not yet resolved.
evidence_needed: Resolved ceremony endpoint paths; whether challenge endpoint answers unauthenticated; any rate-limit/429 headers on challenge issuance.
verify_steps: PASSIVE: resolve the minified URL-builder map feeding the `${l}`/`${o}`/`${s}`/`${r}` ceremony URLs in broadcast_public.js, then one GET each on https://broadcast.threema.ch, record status + rate-limit headers; full assertion needs a passkey (AUTH_HELPED). ≤1 rps.
impact: unauth passkey-ceremony abuse/DoS; an assertion-validation flaw would be account-level — unproven; severity low→high contingent on endpoint reachability.
testability: PASSIVE
[HYP] Gateway control-plane unauth surface and /api nginx-deny escape
class: AUTH
asset: https://gateway.threema.ch (/en/login, /en/signup, /en/forgot-password, /api/*)
confidence: 50
reasoning: Fresh messaging-gateway control plane; login served with NO HSTS/Expect-CT/CSP (weaker than shop); `/api/`+`/api/v1/` return nginx-403 (routes gated, not absent); `trusted_redirect_origins` field present on login. No staging mirror (gateway.test 403) → no divergence oracle.
evidence_needed: POST targets + CSRF field names + captcha presence on signup/forgot-password; whether any /api subpath escapes the nginx deny.
verify_steps: PASSIVE: GET /en/signup and /en/forgot-password, capture CSRF/captcha/POST targets; GET a small set of plausible /api paths; do NOT submit POSTs (side effects). ≤1 rps.
impact: unauthenticated customer-facing gateway surface; credential-page transport hardening gap; severity low (no vuln proven).
testability: PASSIVE
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: Source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for the client login frame. Staging is a single-IP consolidation of the two-shard prod cluster (boundary 0x80); only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[NEXT] PROBE: fetch `https://broadcast.threema.ch/cache/broadcast_public.js` (≤1 rps, single GET), resolve the minified template-literal URL builders feeding the WebAuthn ceremony calls (grep for the `/${l}`/`${o}`/`${s}`/`${r}` path segments and the `fetch(`/`axios` targets adjacent to `assertion_response`/`challenge`), then single GET each resolved ceremony path on https://broadcast.threema.ch recording status + 429/RateLimit headers. No POSTs, no passkey interaction this round.
## 2026-08-08 06:04:46 UTC [chat] (model bigpickle)
class: BUSLOGIC
asset: apip.threema.ch/identity/ws/revoke
reasoning: source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for client login frame; DNS topology newly mapped: staging is single-IP consolidation of a two-shard prod cluster (boundary 0x80), only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[HYP] Staging work frontend/backend skew persists — license-token oracle if backend ships
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle v2.25.1 implements GET (returns license username/password/expired/hasEmail) + PUT for the route; backend still returns method-agnostic 404 catch-all (900B) — re-confirmed this cycle; prod bundle has zero /public/* handlers; both builds frozen (hashes unchanged) so no catch-up in either direction.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
[HYP] Broadcast passkey ceremony endpoints reachable unauthenticated
class: AUTH
asset: https://broadcast.threema.ch (bundle v2.28.1, csrf_public_login_access)
confidence: 50
reasoning: Fresh prod surface (was TIMEOUT). Live login form embeds `csrf_public_login_access`; bundle implements full attestation+assertion ceremonies (challenge POST → `assertion_response` POST); no CSP/HSTS on served pages. Ceremony URL paths are minified/derived (template `${l}`/`${o}`/`${s}`/`${r}`), not yet resolved.
evidence_needed: Resolved ceremony endpoint paths; whether challenge endpoint answers unauthenticated; any rate-limit/429 headers on challenge issuance.
verify_steps: PASSIVE: resolve the minified URL-builder map feeding the `${l}`/`${o}`/`${s}`/`${r}` ceremony URLs in broadcast_public.js, then one GET each on https://broadcast.threema.ch, record status + rate-limit headers; full assertion needs a passkey (AUTH_HELPED). ≤1 rps.
impact: unauth passkey-ceremony abuse/DoS; an assertion-validation flaw would be account-level — unproven; severity low→high contingent on endpoint reachability.
testability: PASSIVE
[HYP] Gateway control-plane unauth surface and /api nginx-deny escape
class: AUTH
asset: https://gateway.threema.ch (/en/login, /en/signup, /en/forgot-password, /api/*)
confidence: 50
reasoning: Fresh messaging-gateway control plane; login served with NO HSTS/Expect-CT/CSP (weaker than shop); `/api/`+`/api/v1/` return nginx-403 (routes gated, not absent); `trusted_redirect_origins` field present on login. No staging mirror (gateway.test 403) → no divergence oracle.
evidence_needed: POST targets + CSRF field names + captcha presence on signup/forgot-password; whether any /api subpath escapes the nginx deny.
verify_steps: PASSIVE: GET /en/signup and /en/forgot-password, capture CSRF/captcha/POST targets; GET a small set of plausible /api paths; do NOT submit POSTs (side effects). ≤1 rps.
impact: unauthenticated customer-facing gateway surface; credential-page transport hardening gap; severity low (no vuln proven).
testability: PASSIVE
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: Source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for the client login frame. Staging is a single-IP consolidation of the two-shard prod cluster (boundary 0x80); only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[NEXT] PROBE: fetch `https://broadcast.threema.ch/cache/broadcast_public.js` (≤1 rps, single GET), resolve the minified template-literal URL builders feeding the WebAuthn ceremony calls (grep for the `/${l}`/`${o}`/`${s}`/`${r}` path segments and the `fetch(`/`axios` targets adjacent to `assertion_response`/`challenge`), then single GET each resolved ceremony path on https://broadcast.threema.ch recording status + 429/RateLimit headers. No POSTs, no passkey interaction this round.
evidence_needed: backend deploys the route (returns ≠404 for a valid-format token).
verify_steps: PASSIVE: single GET `/api-app/public/license/token/{64zeros}` weekly; record 404 vs non-404; ≤1 rps.
[HYP] Broadcast passkey ceremony endpoints reachable unauthenticated
class: AUTH
asset: https://broadcast.threema.ch (bundle v2.28.1, csrf_public_login_access)
confidence: 50
reasoning: Fresh prod surface (was TIMEOUT). Live login form embeds `csrf_public_login_access`; bundle implements full attestation+assertion ceremonies (challenge POST → `assertion_response` POST); no CSP/HSTS on served pages. Ceremony URL paths are minified/derived (template `${l}`/`${o}`/`${s}`/`${r}`), not yet resolved.
evidence_needed: Resolved ceremony endpoint paths; whether challenge endpoint answers unauthenticated; any rate-limit/429 headers on challenge issuance.
verify_steps: PASSIVE: resolve the minified URL-builder map feeding the `${l}`/`${o}`/`${s}`/`${r}` ceremony URLs in broadcast_public.js, then one GET each on https://broadcast.threema.ch, record status + rate-limit headers; full assertion needs a passkey (AUTH_HELPED). ≤1 rps.
impact: unauth passkey-ceremony abuse/DoS; an assertion-validation flaw would be account-level — unproven; severity low→high contingent on endpoint reachability.
testability: PASSIVE
[HYP] Gateway control-plane unauth surface and /api nginx-deny escape
class: AUTH
asset: https://gateway.threema.ch (/en/login, /en/signup, /en/forgot-password, /api/*)
confidence: 50
reasoning: Fresh messaging-gateway control plane; login served with NO HSTS/Expect-CT/CSP (weaker than shop); `/api/`+`/api/v1/` return nginx-403 (routes gated, not absent); `trusted_redirect_origins` field present on login. No staging mirror (gateway.test 403) → no divergence oracle.
evidence_needed: POST targets + CSRF field names + captcha presence on signup/forgot-password; whether any /api subpath escapes the nginx deny.
verify_steps: PASSIVE: GET /en/signup and /en/forgot-password, capture CSRF/captcha/POST targets; GET a small set of plausible /api paths; do NOT submit POSTs (side effects). ≤1 rps.
impact: unauthenticated customer-facing gateway surface; credential-page transport hardening gap; severity low (no vuln proven).
testability: PASSIVE
[HYP] Staging chat mirror shows version/test-identity skew
class: MISCONFIG
asset: g-{00..ff}.0.test.threema.ch:5222 (203.56.114.34)
confidence: 50
reasoning: Source-derived framing concrete (CspSocket reads SERVER_HELLO_LEN=80, SERVER_LOGIN_ACK_LEN=32, plain Socket on 5222); read-only connect returns 0 bytes on prod .202/.204 and staging .114.34 alike — server waits for the client login frame. Staging is a single-IP consolidation of the two-shard prod cluster (boundary 0x80); only group "0" resolves on either.
evidence_needed: in-band difference in CSP server-hello/login-ack fields once a valid login frame is sent (version, test identity acceptance).
verify_steps: AUTH_HELPED: with authorized staging test identity, send one framed login (16B cookie + 64B box + 32B ext-indicator + 24B reserved + 32B vouch) to staging vs prod 5222; compare server-hello/login-ack; ≤1 rps one attempt each, no data modification.
impact: internet-reachable staging messenger backend → version skew/unpatched surface; severity low.
testability: AUTH_HELPED
[NEXT] PROBE: fetch `https://broadcast.threema.ch/cache/broadcast_public.js` (≤1 rps, single GET), resolve the minified template-literal URL builders feeding the WebAuthn ceremony calls (grep for the `/${l}`/`${o}`/`${s}`/`${r}` path segments and the `fetch(`/`axios` targets adjacent to `assertion_response`/`challenge`), then single GET each resolved ceremony path on https://broadcast.threema.ch recording status + 429/RateLimit headers. No POSTs, no passkey interaction this round.
[LEARN] (no new learnings this cycle)
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 90 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry; staging work public API divergence confirmed
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 85 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 85 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
[HYP] Staging work license-token backend catch-up → credential oracle
class: MISCONFIG
asset: https://work.test.threema.ch/api-app/public/license/token/{64hex}
confidence: 45
reasoning: staging bundle v2.25.1 implements GET (license username/password/expired/hasEmail) + PUT for this route; backend re-confirmed method-agnostic 404 catch-all (900B); prod bundle has zero /public/* handlers; both builds frozen (hashes unchanged).
evidence_needed: backend deploys the route (returns ≠404 for a valid-format 64-char token).
verify_steps: PASSIVE: weekly single GET `/api-app/public/license/token/{64zeros}` + `/api-app/public/global/settings`; record 404 vs non-404; ≤1 rps.
impact: if shipped, unauthenticated 64-hex license-token → credential oracle; low (tokens high-entropy).
testability: PASSIVE
[HYP] Gateway control-plane transport/CAPTCHA surface and /api hard-block
class: BUSLOGIC
asset: https://gateway.threema.ch (/en/signup, /en/forgot-password, /api/*)
confidence: 45
reasoning: signup+forgot both 200 with POST-to-self forms, per-form CSRF (`client[csrf_client]`, `csrf_gw-forgot-password`) and hCaptcha; login served with NO HSTS/Expect-CT/CSP (weaker than shop); all 9 probed /api paths → uniform nginx 403 (146B) so route map not reachable.
evidence_needed: forgot-password response behavior for known vs unknown email (enumeration) — requires authorized account mapping (AUTH_HELPED); captcha-skippable forms unproven without side-effect POSTs.
verify_steps: AUTH_HELPED: with authorized gateway test credentials, compare forgot-password responses for own-valid vs invalid email; GET-only /api enumeration exhausted (blanket 403).
impact: transport-hardening gap + potential email-enumeration; severity low (no vuln proven).
testability: AUTH_HELPED
[HYP] Broadcast passkey ceremony answerable with server-issued session+CSRF — self-provisioned
class: AUTH
asset: https://broadcast.threema.ch/en/login/passkey/start
confidence: 42
reasoning: endpoints resolved to POST-only routes; GET → 404; POST with orphaned csrf → 401; server freely issues session+csrf on unauthenticated GET /, so any visitor can complete a self-challenge ceremony; no cross-user leverage observed.
evidence_needed: whether a valid-session POST issues a challenge with 429/RateLimit headers (one valid POST — side-effect, deferred).
verify_steps: PASSIVE (limited): single GET / captures fresh csrf+SESSIONID to confirm issuance; do NOT repeat challenge POST this cycle.
impact: self-session challenge issuance at most self-DoS; severity low.
testability: PASSIVE
[NEXT] PROBE: weekly catch-up checks — single GET `https://work.test.threema.ch/api-app/public/global/settings` and single GET `https://work.test.threema.ch/api-app/public/license/token/0000000000000000000000000000000000000000000000000000000000000000`; if license-token returns ≠404, escalate to valid-format token probes (never real tokens); ≤1 rps, GET only.
## 2026-08-08 07:08:54 UTC [chat] (model bigpickle)
