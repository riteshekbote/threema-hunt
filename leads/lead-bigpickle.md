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
