# Ranked Hypotheses

## SEED 2026-08-07 (from passive recon, not model-generated yet)
- [60] threema-desktop source: credential/token handling in desktop app (Electron-style) — static analysis first (from inventory seed)
- [55] apip.threema.ch ID service: enumeration/authz patterns in directory service — passive probes only (from inventory seed)
- [50] threema-android/iOS source: crypto/zero-knowledge implementation bugs (public key handling, ratchet, backup safe-*.threema.ch flow) (from inventory seed)

## RANKED HYPOTHESES 2026-08-07 18:40:46 UTC
- [70] apip.threema.ch: apip.threema.ch ID enumeration via CORS misconfiguration (from reports/hypotheses-nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://apip.threema.ch/api/v1/pubkeys/<8-char-threema-id> — test if public key lookup endpoint exists and returns data without auth (common pattern 
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration enabling cross-origin API probes confirmed via passive HEAD/GET.
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.

## RANKED HYPOTHESES 2026-08-07 18:55:12 UTC
- [78] https://ds-apip.threema.ch/identity/{identity}: Directory server identity enumeration via public `GET /identity/{id}` (from reports/hypotheses-laguna.txt)
- [70] apip.threema.ch: apip.threema.ch ID enumeration via CORS misconfiguration (from reports/hypotheses-nemotron3.txt)
- [55] ds-apip.threema.ch: ds-apip.threema.ch directory lookup / unauth enumeration (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://apip.threema.ch/api/v1/pubkeys/ABCD1234 — test if public key lookup endpoint exists and returns data without auth (common pattern in Threema 
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://ds-apip-work.threema.ch/identity/lookup and /directory (401 vs 404 distinguishes route-existence behind the auth gate), then GET https://ds-a
- NEXT(hypotheses-laguna.txt): PROBE: `curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOEC
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration enabling cross-origin API probes confirmed via passive HEAD/GET.
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.

## RANKED HYPOTHESES 2026-08-07 19:15:11 UTC
- [88] https://ds-apip.threema.ch/identity/fetch_bulk: Directory server identity enumeration via unauthenticated bulk-fetch + permissive CORS (from reports/hypotheses-laguna.txt)
- [78] https://ds-apip.threema.ch/identity/{identity}: ds-apip.threema.ch identity enumeration via public GET /identity/{id} and missing rate limit (from reports/hypotheses-nemotron3.txt)
- [60] ds-apip.test.threema.ch: ds-apip.test.threema.ch staging directory server publicly exposed (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECH
- NEXT(hypotheses-laguna.txt): PROBE: send 30 sequential POSTs to https://ds-apip.threema.ch/identity/fetch_bulk at 2s intervals (≤1 rps) using a batch of 30 identities; log every HTTP code, 
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he

## RANKED HYPOTHESES 2026-08-07 20:10:17 UTC
- [88] https://ds-apip.threema.ch/identity/fetch_bulk: ds-apip.threema.ch bulk identity enumeration via unauthenticated fetch_bulk + permissive CORS (from reports/hypotheses-nemotron3.txt)
- [75] apps/desktop/src/common/node/key-storage/index.ts: Desktop Windows key storage: keystorage.bin and password file lack ACL restrictions (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECH
- NEXT(hypotheses-laguna.txt): PROBE: `curl -s -o /dev/null -w 'HTTP %{http_code} | time_total: %{time_total}s | headers: %{header_json}\n' -X POST https://ds-apip.test.threema.ch/identity/fe
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on st

## RANKED HYPOTHESES 2026-08-07 20:57:04 UTC
- [95] ds-apip.threema.ch: Directory server identity enumeration across 5 hostnames (from reports/hypotheses-laguna.txt)
- [88] https://ds-apip.threema.ch/identity/fetch_bulk: ds-apip.threema.ch bulk identity enumeration via unauthenticated fetch_bulk + permissive CORS (from reports/hypotheses-nemotron3.txt)
- [55] safe-01.threema.ch: Safe backup server permissive CORS + unauthenticated probe surface (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECH
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on st

## RANKED HYPOTHESES 2026-08-07 21:23:50 UTC
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts (from reports/hypotheses-nemotron3.txt)
- [55] safe-01.threema.ch: Safe backup server permissive CORS + unauthenticated probe surface (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "A
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS
- LEARN: ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELE
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on st

## RANKED HYPOTHESES 2026-08-07 22:06:34 UTC
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts (from reports/hypotheses-nemotron3.txt)
- [45] apip.threema.ch/identity/ws/revoke: ds-apip.threema.ch directory lookup / unauth enumeration (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "A
- NEXT(hypotheses-bigpickle.txt): PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "A
- NEXT(hypotheses-laguna.txt): PROBE: burst of 15 `POST https://ds-apip.threema.ch/identity/fetch_bulk` (~1 req every 2s, ≤1 rps) with a batch of 1 valid ID (ECHOECHO) + 2 distinct invalid ID
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS
- LEARN: ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELE
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS
- LEARN: ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELE

## RANKED HYPOTHESES 2026-08-07 22:43:53 UTC
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts (from reports/hypotheses-nemotron3.txt)
- [65] https://g-*.0.test.threema.ch: Staging chat cluster g-*.0.test.threema.ch publicly reachable single-node mirror of prod chat (from reports/hypotheses-bigpickle.txt)
- [50] https://safe-01.threema.ch/backups/{id}: Safe backup API: credential-gated with permissive CORS allowing Authorization header (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "A
- NEXT(hypotheses-bigpickle.txt): RAG: fetch the chat connection handshake/constants from github.com/threema-ch/threema-android (public, in-scope) — `ChatConnection`/`ChatProtocol`/server hostna
- NEXT(hypotheses-laguna.txt): PROBE: `curl -s -m 12 -X POST -H "Authorization: Basic $(echo -n 'backupId:backupKey' | base64)" -H "Content-Type: application/json" https://safe-01.threema.ch/
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS
- LEARN: ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELE
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPT
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `node
- LEARN: ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT.
- LEARN: REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis 
- LEARN: REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven.
- LEARN: ACCEPTED MISCONFIG @ g-{XX}.0.test.threema.ch: Staging chat cluster resolves to 203.56.114.34 (split from prod chat IPs 203.56.112.202/.204); HTTPS not HTTP-acc

## RANKED HYPOTHESES 2026-08-07 23:15:48 UTC
- [70] https://safe-01.threema.ch/backups/{id}: Safe backup cross-origin credentialed access via CORS * + Authorization header (from reports/hypotheses-nemotron3.txt)
- [55] wss://mediator-{0..f}.test.threema.ch/{hexproto}/: Staging mediator/rendezvous WSS surface reachable with staging cert (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -i -X OPTIONS https://safe-01.threema.ch/backups/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa -H "Origin: https://attacker.ex
- NEXT(hypotheses-bigpickle.txt): PROBE: `curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}"`, then extract referenced
- LEARN: ACCEPTED MISCONFIG @ g-{XX}.0.test.threema.ch: Staging chat cluster resolves to 203.56.114.34 (split from prod chat IPs 203.56.112.202/.204); HTTPS not HTTP-acc
- LEARN: ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT
- LEARN: ACCEPTED OTHER @ ds-apip-work.test.threema.ch: Work directory staging backend confirmed live — 401 on all paths, CORS `*`, no HSTS/Expect-CT
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: Staging work web app confirmed live — 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPT
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `node
- LEARN: REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis 
- LEARN: REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven
- LEARN: ACCEPTED OTHER @ safe-*.threema.ch DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 all resolve to 203.56.112.231 (single IP, 5 hostnames)
- LEARN: ACCEPTED OTHER @ mediator-{0..f}/rendezvous-{0..f}.threema.ch DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all uniform 403 on 
- LEARN: ACCEPTED OTHER @ Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HT

## RANKED HYPOTHESES 2026-08-07 23:47:08 UTC
- [70] https://safe-01.threema.ch/backups/{id}: Safe backup cross-origin credentialed access via CORS * + Authorization header (from reports/hypotheses-nemotron3.txt)
- [65] https://work.test.threema.ch/api-app/public/*: Staging work API exposes public endpoints absent on production (from reports/hypotheses-bigpickle.txt)
- [55] wss://mediator-{0..f}.test.threema.ch/{hexproto}/: Staging mediator/rendezvous WSS surface reachable with staging cert (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -i -X OPTIONS https://safe-01.threema.ch/backups/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa -H "Origin: https://attacker.ex
- NEXT(hypotheses-bigpickle.txt): PROBE: enumerate all `/api-app/*` route literals from /tmp/opencode/worktest.js (bundle v2.25.1), then GET each `public` route on work.test.threema.ch AND work.
- NEXT(hypotheses-laguna.txt): PROBE: `curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}"`, then extract referenced
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPT
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `node
- LEARN: REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis 
- LEARN: REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven
- LEARN: ACCEPTED OTHER @ safe-*.threema.ch DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 all resolve to 203.56.112.231 (single IP, 5 hostnames)
- LEARN: ACCEPTED OTHER @ mediator-{0..f}/rendezvous-{0..f}.threema.ch DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all uniform 403 on 
- LEARN: ACCEPTED OTHER @ Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HT
- LEARN: ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT
- LEARN: ACCEPTED OTHER @ ds-apip-work.test.threema.ch: Work directory staging backend confirmed live — 401 on all paths, CORS `*`, no HSTS/Expect-CT
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: Staging work web app confirmed live — 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only the `/api-app/public/*` namespace is open (namespace gati
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed).
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: liveness endpoints `/ping` (204) and `/info/ping.php` (200 empty) identical on staging and prod — no divergence.

## RANKED HYPOTHESES 2026-08-08 00:15:46 UTC
- [75] https://work.test.threema.ch/api-app/public/*: Staging work public API namespace exposes endpoints absent on production (from reports/hypotheses-nemotron3.txt)
- [50] g-{00..ff}.0.test.threema.ch:5222: Staging chat mirror shows version/test-identity skew (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}", then extract referenced J
- NEXT(hypotheses-bigpickle.txt): AUTH_HELPED: with an authorized staging test identity, reconstruct the Threema CSP NaCl-box+HKDF handshake from threema-android (csp/connection) and perform one
- LEARN: ACCEPTED IDOR @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API en
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `__HOST-HTTP-SESSIONID` cookie set on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
- LEARN: ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only `/api-app/public/*` namespace is open (namespace gating c
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/info/ping.php` → 200 empty and `/ping` → 204 identical on staging and prod — no divergence

## RANKED HYPOTHESES 2026-08-08 02:19:22 UTC
- [95] apps/desktop/src/common/node/{fs,key-storage/index.ts,crypto.ts}: Desktop Windows key-storage ACL bypass → master-password recovery → identity keypair + DB decryption (from reports/hypotheses-laguna.txt)
- [75] https://work.test.threema.ch/api-app/public/*: Staging work public API namespace exposes endpoints absent on production (from reports/hypotheses-nemotron3.txt)
- [52] ds-apip-work.threema.ch/identities: Work directory /identities cross-subscription metadata disclosure (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}", then extract referenced J
- NEXT(hypotheses-bigpickle.txt): PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/000000000000000000000000000000000000000000000000000
- NEXT(hypotheses-laguna.txt): RAG: Run `grep -rn "require(\|import(\|eval(\|process\.mainModule\|child_process\|fs\." /tmp/opencode/threema-desktop/apps/desktop/src/worker/backend/backend-wo
- LEARN: ACCEPTED IDOR @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API en
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `__HOST-HTTP-SESSIONID` cookie set on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
- LEARN: ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only `/api-app/public/*` namespace is open (namespace gating c
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/info/ping.php` → 200 empty and `/ping` → 204 identical on staging and prod — no divergence
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch bundle divergence: work_public.js v2.25.1 DIFFERENT builds staging vs prod (staging sha256 e48e18f79df0125e8942f8fd956
- LEARN: ACCEPTED OTHER @ work.threema.ch: prod DOES route /api-app — GET /api-app/me/profile → 302 to /en/login?r=%2Fapi-app%2Fme%2Fprofile; only /public/* namespace ab
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/license/token/{64hex} → 404 HTML catch-all (900B) for GET, PUT, AND OPTIONS — method-agnostic; backend ro
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/* namespace map closed on both hosts — /, /global/, /config, /registration, /license/, /global/app-downlo
- LEARN: ACCEPTED OTHER @ g-*.0.{test.,}threema.ch chat: read-only TCP connect to 5222 returns 0 bytes on BOTH staging (203.56.114.34) and prod (203.56.112.202) — no ser
- LEARN: ACCEPTED OTHER @ hcaptcha-work.threema.ch: 200 serving hCaptcha's own Webflow marketing page (Last Published 2026-07-30) — third-party captcha host, out-of-scop
- LEARN: ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.tes

## RANKED HYPOTHESES 2026-08-08 03:47:32 UTC
- [75] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}: Desktop Windows key-storage ACL bypass → same-user DPAPI recovery → full key-store decryption (from reports/hypotheses-laguna.txt)
- [52] ds-apip-work.threema.ch/identities: Work directory /identities cross-subscription metadata disclosure (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: weekly staging catch-up check — single `GET https://work.test.threema.ch/api-app/public/license/token/000000000000000000000000000000000000000000000000000
- NEXT(hypotheses-laguna.txt): RAG: clone github.com/threema-ch/threema-desktop, then read `apps/desktop/src/electron/electron-main.ts` BrowserWindow webPreferences to capture exact line numb
- LEARN: ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.tes

## RANKED HYPOTHESES 2026-08-08 04:43:47 UTC
- [75] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}: Safe backup cross-origin credentialed access via CORS * + Authorization header (from reports/hypotheses-nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}" — test bulk
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 05:26:39 UTC
- [90] https://ds-apip.threema.ch/identity/{id}: Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (from reports/hypotheses-nemotron3.txt)
- [55] g-*.0.test.threema.ch: Staging chat cluster publicly reachable (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}"
- NEXT(hypotheses-bigpickle.txt): PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "A
- LEARN: (no new learnings this cycle)
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS
- LEARN: ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELE
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only the `/api-app/public/*` namespace is open (namespace gati
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed).
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: liveness endpoints `/ping` (204) and `/info/ping.php` (200 empty) identical on staging and prod — no divergence.
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch bundle divergence: work_public.js v2.25.1 DIFFERENT builds staging vs prod (staging sha256 e48e18f79df0125e8942f8fd956
- LEARN: ACCEPTED OTHER @ work.threema.ch: prod DOES route /api-app — GET /api-app/me/profile → 302 to /en/login?r=%2Fapi-app%2Fme%2Fprofile; only /public/* namespace ab
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/license/token/{64hex} → 404 HTML catch-all (900B) for GET, PUT, AND OPTIONS — method-agnostic; backend ro
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/* namespace map closed on both hosts — /, /global/, /config, /registration, /license/, /global/app-downlo
- LEARN: ACCEPTED OTHER @ g-*.0.{test.,}threema.ch chat: read-only TCP connect to 5222 returns 0 bytes on BOTH staging (203.56.114.34) and prod (203.56.112.202) — no ser
- LEARN: ACCEPTED OTHER @ hcaptcha-work.threema.ch: 200 serving hCaptcha's own Webflow marketing page (Last Published 2026-07-30) — third-party captcha host, out-of-scop
- LEARN: ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.tes
- LEARN: ACCEPTED OTHER @ avatar.test.threema.ch / companylogo.test.threema.ch: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; broadcast.tes
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 06:05:58 UTC
- [90] https://ds-apip.threema.ch/identity/{id}: Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (from reports/hypotheses-nemotron3.txt)
- [45] apip.threema.ch/identity/ws/revoke: Staging work frontend/backend skew persists — license-token oracle if backend ships (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}"
- NEXT(hypotheses-bigpickle.txt): PROBE: fetch `https://broadcast.threema.ch/cache/broadcast_public.js` (≤1 rps, single GET), resolve the minified template-literal URL builders feeding the WebAu
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: (no new learnings this cycle)

## RANKED HYPOTHESES 2026-08-08 07:09:30 UTC
- [90] https://ds-apip.threema.ch/identity/{id}: Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (from reports/hypotheses-nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}"
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 08:04:49 UTC
- [90] https://ds-apip.threema.ch/identity/{id}: Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (from reports/hypotheses-nemotron3.txt)
- [45] https://broadcast.threema.ch: Broadcast passkey ceremony — self-session challenge, at most self-DoS (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}"
- NEXT(hypotheses-bigpickle.txt): HUMAN: GO/NO-GO gate before any further live probing — confirm this engagement is an active, authorized Threema program (verified scope + reporting channel); tr
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 08:33:27 UTC
- [90] https://ds-apip.threema.ch/identity/{id}: Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (from reports/hypotheses-nemotron3.txt)
- [45] https://broadcast.threema.ch: Broadcast passkey ceremony — self-session challenge, at most self-DoS (from reports/hypotheses-bigpickle.txt)
- [0] `apps/desktop/src/common/node/{fs.ts:40-41,: Desktop key-storage Windows ACL bypass → identity + message-DB compromise (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): RAG: clone github.com/threema-ch/threema-desktop and verify exact line numbers for `fileModeInternalObjectIfPosix()` in `apps/desktop/src/common/node/fs.ts`, `_
- NEXT(hypotheses-bigpickle.txt): PROBE: weekly catch-up checks — single GET `https://work.test.threema.ch/api-app/public/global/settings` and single GET `https://work.test.threema.ch/api-app/pu
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`

## RANKED HYPOTHESES 2026-08-08 09:15:49 UTC
- [45] https://ds-apip.threema.ch/identity/fetch_bulk: fetch_bulk batch-size cap and enumeration throughput unmeasured (from reports/hypotheses-bigpickle.txt)
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`

## RANKED HYPOTHESES 2026-08-08 10:06:30 UTC
- [95] apps/desktop/src/common/node/{fs.ts:40-41,: Desktop key-storage Windows ACL bypass → identity + message-DB compromise (from reports/hypotheses-laguna.txt)
- [90] https://ds-apip.threema.ch/identity/fetch_bulk: Directory identity→pubkey enumeration at scale via fetch_bulk (from reports/hypotheses-nemotron3.txt)
- [45] https://ds-apip.threema.ch/identity/fetch_bulk: fetch_bulk batch-size cap and enumeration throughput unmeasured (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): RAG: Clone github.com/threema-ch/threema-desktop and verify exact line numbers for fileModeInternalObjectIfPosix() in apps/desktop/src/common/node/fs.ts, _write
- NEXT(hypotheses-bigpickle.txt): RAG: clone github.com/threema-ch/threema-desktop, then read `apps/desktop/src/electron/electron-main.ts` BrowserWindow webPreferences to capture exact line numb
- NEXT(hypotheses-laguna.txt): RAG: Read `/tmp/opencode/threema-desktop/apps/desktop/src/common/node/key-storage/layers/outer/v2.ts` and `outer/common.ts` to confirm the outer-layer encryptio
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: fileModeInternalObjectIfPosix() returns {} on Windows — keystorage.bin and keystorage.password.bin written
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELE
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/E
- LEARN: ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-A
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encr
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: (no new learnings this cycle)

## RANKED HYPOTHESES 2026-08-08 10:23:27 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [50] https://ds-apip.threema.ch/identity/fetch_bulk: fetch_bulk batch-size cap and invalid-only response shape unmeasured (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-bigpickle.txt): HUMAN: GO/NO-GO gate still open — no operator confirmation. Request (a) confirmation of active authorized program + reporting channel, (b) GO/NO-GO. If GO, firs
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 11:03:45 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] `%APPDATA%\ThreemaDesktop\data\keystorage.bin`: Desktop key-storage Windows ACL bypass → identity + message-DB compromise (from reports/hypotheses-laguna.txt)
- [40] https://work.test.threema.ch/api-app/public/license/token/{64hex}: work.test license-token backend ships → unauth 64-hex credential oracle (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-bigpickle.txt): HUMAN: GO/NO-GO gate still open — no operator confirmation arrived. No live HTTP probes executed or scheduled this cycle (static-only; desktop findings now veri
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 11:23:32 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] `%APPDATA%\ThreemaDesktop\data\keystorage.bin`: Desktop key-storage Windows ACL bypass → identity + message-DB compromise (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-laguna.txt): AUTH_HELPED-LOCAL: Execute the Windows ACL bypass verification procedure on a Windows machine with Threema Desktop installed — run `verify-acl-bypass.ps1` to co
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 11:51:30 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 12:06:35 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [78] https://ds-apip.threema.ch/identity/{identity}: Directory server identity enumeration via public `GET /identity/{id}` (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-bigpickle.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-laguna.txt): PROBE: `curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOEC
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pub
- LEARN: ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API pr
- LEARN: ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS he
- LEARN: ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin`
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL
- LEARN: ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on st
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateL

## RANKED HYPOTHESES 2026-08-08 13:10:07 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] github.com/threema-ch/threema-desktop: Windows key-storage ACL bypass enables master password recovery and identity keypair decryption (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 14:00:34 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] threema-desktop: Staging chat mirror shows version/test-identity skew (from reports/hypotheses-bigpickle.txt)
- [50] https://ds-apip.threema.ch/identity/fetch_bulk: Safe backup API: HTTP Basic Auth backupId:backupKey with permissive CORS enables credential brute-force / backup enumeration (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-bigpickle.txt): PROBE: fetch `https://broadcast.threema.ch/cache/broadcast_public.js` (≤1 rps, single GET), resolve the minified template-literal URL builders feeding the WebAu
- NEXT(hypotheses-laguna.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: (no new learnings this cycle)
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 14:23:10 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory cluster identity→pubkey bulk enumeration via fetch_bulk at scale across 3 production hosts (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-bigpickle.txt): PROBE: safe-01 backup existence-oracle test — 3 single GETs at 1 rps on `https://safe-01.threema.ch/backups/{rand1}` / `{rand2}` / known-baseline-64hex, capture
- NEXT(hypotheses-laguna.txt): PROBE: Verify `POST /identity/fetch_bulk` on `ds-apip.threema.ch` accepts a 100+ ID payload in a single request and returns the full pubkey map — then repeat ac
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: POST /identity/fetch_bulk returns pubkeys for valid IDs only, silently omits invalid IDs,
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: /api-app/public/global/settings returns 200 unauthenticated on staging but 404 on prod — public staging-only work API end
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: work_public.js v2.25.1 staging build (sha256 e48e18f7…, 1,443,948 B) implements /public/* routes; prod build (sha256 
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged (bench
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 15:03:05 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [45] https://ds-apip.threema.ch/identity/{blob_cred|sfu_cred|set_revocation_key|check_revocation_key|update_work_info}: Unprobed challenge endpoints expose route presence + challenge-oracle unauth (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: Verify `POST /identity/fetch_bulk` on `ds-apip.threema.ch` accepts a 100+ ID payload in a single request and returns the full pubkey map — then repeat ac
- NEXT(hypotheses-bigpickle.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: (no new learnings this cycle)

## RANKED HYPOTHESES 2026-08-08 15:30:23 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [95] threema-desktop: Windows key-storage ACL bypass — master password recovery via fileModeInternalObjectIfPosix() returning {} on Windows, keystorage.bin and keystorage.password.bin written without ACL restrictions, safeStorage (DPAPI) password recoverable by same-user processes (from reports/hypotheses-ling3.txt)
- [45] https://ds-apip.threema.ch/identity/{blob_cred|sfu_cred|set_revocation_key|check_revocation_key|update_work_info}: Unprobed challenge endpoints expose route presence + challenge-oracle unauth (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-laguna.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- NEXT(hypotheses-ling3.txt): HUMAN: Run verify-acl-bypass.ps1 on an authorized Windows host with Threema Desktop installed to confirm the Windows ACL bypass for key storage (master password
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoc
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST `/identity/fetch_bulk` accepts 100+ ID batch in single request, returns pubkeys for vali
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (`/identity/{sfu_cred|blob_cred|set_revocation_key|check_revo
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for `/backups/{64hex}` — header inconsis
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): `fileModeInternalObjectIfPosix()` returns `{}` on Windows → keystorage.bin + keystorage.password.bin
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not used for real encry
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not

## RANKED HYPOTHESES 2026-08-08 16:04:12 UTC
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory bulk identity enumeration + challenge endpoint parameter oracles at scale (from reports/hypotheses-nemotron3.txt)
- [95] github.com/threema-ch/threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ",...(98 more uniq
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoca
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: ACCEPTED IDOR @ api.threema.ch + apip.threema.ch: POST /identity/fetch_bulk 100+ ID batch confirmed this cycle — 200, returns only valid IDs' pubkeys, silently 
- LEARN: ACCEPTED OTHER @ all 3 directory hosts: 5 challenge endpoints confirmed live via GET returning 200 with JSON error bodies + CORS `*`; `update_work_info` returns
- LEARN: CONFIRMED @ all 3 directory hosts: No `Access-Control-Expose-Headers` on any response — ACAO:* enables cross-origin body read (unauthenticated), but response he
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not 

## RANKED HYPOTHESES 2026-08-08 17:01:24 UTC
- [95] https://ds-apip.threema.ch/identity/fetch_bulk: Directory bulk identity enumeration + challenge endpoint parameter oracles at scale (from reports/hypotheses-nemotron3.txt)
- [45] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}: Safe backup credentialed cross-origin read of backup store (from reports/hypotheses-bigpickle.txt)
- [45] https://safe-01.threema.ch/backups/{64hex}: Unprobed challenge endpoints expose route presence + challenge-oracle unauth (from reports/hypotheses-laguna.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ",...(998 more uni
- NEXT(hypotheses-laguna.txt): AUTH_HELPED-LOCAL: Execute Windows ACL bypass verification on authorized Windows host with Threema Desktop installed — run `verify-acl-bypass.ps1` to confirm 0 
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoca
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged, not used for real encryp
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoc
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not 
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST `/identity/fetch_bulk` accepts 100+ ID batch in single request, returns pubkeys for vali
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (`/identity/{sfu_cred|blob_cred|set_revocation_key|check_revo
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for `/backups/{64hex}` — header inconsis
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): `fileModeInternalObjectIfPosix()` returns `{}` on Windows → keystorage.bin + keystorage.password.bin
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not used for real encry
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ api.threema.ch + apip.threema.ch: POST /identity/fetch_bulk 100+ ID batch confirmed this cycle — 200, returns only valid IDs' pubkeys, silently 
- LEARN: ACCEPTED OTHER @ all 3 directory hosts: 5 challenge endpoints confirmed live via GET returning 200 with JSON error bodies + CORS `*`; `update_work_info` returns
- LEARN: CONFIRMED @ all 3 directory hosts: No `Access-Control-Expose-Headers` on any response — ACAO:* enables cross-origin body read (unauthenticated), but response he
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `r3gGN9GDQ5NF6tM6` is benchmark-only dummy in determineKdfParams(), derived key immediately purged — not 

## RANKED HYPOTHESES 2026-08-08 17:40:33 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [45] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}: Safe backup credentialed cross-origin read of backup store (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","AAAAAAAA","BBBB
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `52a0af982a9d15b5273a16f15334a5992af0b1e4e86a0203bd91b6e2b99f315c` is benchmark-only dummy in determineKd
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoc
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization

## RANKED HYPOTHESES 2026-08-08 17:59:19 UTC
- [95] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption (from reports/hypotheses-nemotron3.txt)
- [45] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}: Safe backup store credentialed cross-origin read (from reports/hypotheses-bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: curl -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","AAAAAAAA","BBBB
- NEXT(hypotheses-bigpickle.txt): HUMAN: Request program-issued test backupId:backupKey from the operator for the safe backup hypothesis; upon receipt run exactly one `curl -u "testId:testKey" h
- NEXT(hypotheses-laguna.txt): PROBE: curl -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","AAAAAAAA","BBBB
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `52a0af982a9d15b5273a16f15334a5992af0b1e4e86a0203bd91b6e2b99f315c` is benchmark-only dummy in determineKd
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoc
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
- LEARN: REJECTED class @ lead: Desktop BrowserWindow sandbox+worker gap — conditional RCE (requires separate renderer exploit chain), not a standalone class.
- LEARN: REJECTED class @ lead: g-*.0.test.threema.ch staging chat cluster — out of scope per scope.yml.
- LEARN: CONFIRMED MISCONFIG @ threema-desktop key-storage (Windows): ACL-bypass finding stable, no contradicting evidence this cycle.
- LEARN: ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): fileModeInternalObjectIfPosix() returns {} on Windows → keystorage.bin and keystorage.password.bin w
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fe
- LEARN: ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has sandbox: false (TODO DEK-79) and nodeIntegrationInWorker: true (TODO DEK-79) — L1240 co
- LEARN: ACCEPTED MISCONFIG @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work A
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
- LEARN: ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.
- LEARN: REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password `52a0af982a9d15b5273a16f15334a5992af0b1e4e86a0203bd91b6e2b99f315c` is benchmark-only dummy in determineKd
- LEARN: REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not
- LEARN: ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: POST /identity/fetch_bulk accepts 100+ ID batch in single request, returns pubkeys for valid 
- LEARN: ACCEPTED OTHER @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: 5 challenge-response endpoints (/identity/{sfu_cred|blob_cred|set_revocation_key|check_revoc
- LEARN: ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 response for /backups/{64hex} — header inconsiste
- LEARN: REJECTED MISCONFIG @ production directory servers lack HSTS/Expect-CT: missing security headers on by-design public read endpoints — defense-in-depth gap only, 
- LEARN: ACCEPTED AUTH @ safe-*.threema.ch: Backup API uses HTTP Basic Auth (backupId:backupKey); OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization
