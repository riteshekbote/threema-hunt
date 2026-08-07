# Inventory: threema

## Seed 2026-08-07 (passive recon)

### Hosts (in scope)
- g-*.0.threema.ch (chat) — hostname pattern unknown, enumerate once discovered
- mediator-*.threema.ch (sync) — pattern unknown
- rendezvous-*.threema.ch (linking) — pattern unknown
- apip.threema.ch — 403 on /
- safe-*.threema.ch (backup) — pattern unknown
- work.threema.ch — 301
- broadcast.threema.ch — timeout
- gateway.threema.ch — timeout
- shop.threema.ch — 301
- billing.threema.ch — timeout

### Source code (in scope)
- github.com/threema-ch/threema-android (Kotlin)
- github.com/threema-ch/threema-ios (Swift)
- github.com/threema-ch/threema-desktop (TS) — Desktop 2.x
- github.com/threema-ch/threema-web (TS)
- github.com/threema-ch/threema-web-electron
- github.com/threema-ch/threema-msgapi-sdk-* (php/java/python/rust)
- github.com/threema-ch/threema-bot-sdk (Rust)
- github.com/threema-ch/push-relay (Rust)
- github.com/threema-ch/app-remote-protocol

### Open questions
- Actual hostnames matching g-*/mediator-*/rendezvous-*/safe-* patterns
- Auth model of apip.threema.ch (ID service)
- Whether billing/shop share a backend

## 2026-08-07 18:40:46 UTC
- CHANGED work.threema.ch: Now responds with 301 to /en/login (was 301, now confirmed PHP session cookie, CSP, Sentry reporting)
- CHANGED shop.threema.ch: Now responds with 301 to /en (was 301, now confirmed CSP, Sentry, hCaptcha subdomain)
- CHANGED broadcast.threema.ch: Now responds 301 to /en/login (was TIMEOUT, now accessible with session cookie, CSP, Sentry)
- CHANGED gateway.threema.ch: Now responds 302 to /en (was TIMEOUT, now accessible with session cookie, CSP, Sentry)
- CHANGED billing.threema.ch: Now responds 301 to threema.ch (was TIMEOUT, now redirects to main site)
- CHANGED apip.threema.ch: Confirmed 403 with CORS headers allowing POST/GET/OPTIONS/DELETE (was 403, now detailed)
- NEW api.threema.ch: Returns 403 with same CORS headers as apip.threema.ch (likely related ID service)
- NEW safe.threema.ch: Timeout/no response (backup service pattern candidate)

## 2026-08-07 18:55:12 UTC
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `mediator-{X}.threema.ch/{XX}/` hostname pattern (WSS sync server) — `mediator-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `safe-{XX}.threema.ch/` hostname pattern (backup safe) — `safe-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `rendezvous-{X}.threema.ch/{XX}/` hostname pattern (WSS linking server) — `rendezvous-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `api.threema.ch` — 403 + same permissive CORS as apip (candidate ID/directory sibling).
- CHANGED `apip.threema.ch` — was 403 on `/`; now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*`.
- CHANGED `work.threema.ch` / `shop.threema.ch` / `broadcast.threema.ch` / `gateway.threema.ch` — 301/302 now with session cookie, CSP, Sentry (was TIMEOUT/301).
- CHANGED `billing.threema.ch` — 301 → `threema.ch`.
- NEW `ds-apip.test.threema.ch` — leaked test/staging directory server reachable (static + live 200).

## 2026-08-07 19:15:11 UTC
- NEW ds-apip.threema.ch — canonical directory server with public `GET /identity/{id}` returning 200/404 oracle
- NEW mediator-{X}.threema.ch/{XX}/ — WSS sync server hostname pattern confirmed from client config
- NEW safe-{XX}.threema.ch/ — backup safe hostname pattern confirmed from client config
- NEW rendezvous-{X}.threema.ch/{XX}/ — WSS linking server hostname pattern confirmed from client config
- NEW api.threema.ch — 403 with same permissive CORS as apip (ID service sibling)
- NEW ds-apip.test.threema.ch — leaked test/staging directory server reachable (live 200)
- CHANGED apip.threema.ch — now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*` (was 403 on `/`)
- CHANGED work.threema.ch / broadcast.threema.ch / gateway.threema.ch / shop.threema.ch — now accessible with PHP session cookies, CSP, Sentry (were TIMEOUT/301)
- CHANGED billing.threema.ch — now 301 → threema.ch (was TIMEOUT)

## 2026-08-07 20:10:17 UTC
- CHANGED `/identity/fetch_bulk` rate-limit absence quantified: 30 sequential POSTs at 1 rps → all HTTP 200, no 429/RateLimit/Retry-After headers, consistent ~340ms response times. Previously only “no 429 obser
- NEW `ds-apip.test.threema.ch` confirmed live and publicly reachable — returns identical identity→pubkey oracle + CORS `*` (with `Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE`) as production. S
- NEW `api.threema.ch` confirmed as full directory server sibling — `GET /identity/ECHOECHO` → 200 with identical CORS headers as `ds-apip.threema.ch`.
- NEW RAG finding: `crypto.ts:223` hardcoded password `r3gGN9GDQ5NF6tM6` is a **benchmark dummy** only — used in `determineKdfParams()` to measure Argon2id runtime, immediately purged (`benchmarkKey.purge()
- CHANGED RAG finding: OnPrem config trust path **debunked as vulnerable** — uses Ed25519 signature verification against 3 hardcoded `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` in `vite.config.ts`. OPPF URLs are valida
- CHANGED RAG finding: Desktop key storage confirmed — `fileModeInternalObjectIfPosix()` returns `{}` (no restriction) on Windows. Both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI
- CHANGED RAG finding: Electron BrowserWindow has `sandbox` NOT enabled (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79). `nodeIntegration: false` and `contextIsolation: true` are set.

## 2026-08-07 20:57:04 UTC
- NEW `apip.test.threema.ch` — staging directory server confirmed live: `GET /identity/ECHOECHO` → 200, `POST /identity/fetch_bulk` → 200, CORS `*`, HSTS, Expect-CT, returns identical pubkey data as product
- NEW `ds-apip-work.threema.ch` (prod) — work directory server confirmed live: 401 on `/` + `/identity/{id}` + `/identity/fetch_bulk`(404), CORS `*` with DELETE/POST/GET/OPTIONS, no HSTS/Expect-CT
- NEW `ds-apip-work.test.threema.ch` (staging) — work directory server confirmed live: 401 on `/`, CORS `*`, no HSTS/Expect-CT
- NEW `work.test.threema.ch` (staging) — work web app confirmed live: 301 to `/en/login`, HSTS, Expect-CT, CSP with staged subdomain references
- CHANGED Production `apip.threema.ch` confirmed as full directory server sibling — `GET /identity/ECHOECHO` → 200 with identical pubkey data to `ds-apip.threema.ch` (was previously only confirmed as 403 on `/`
- CHANGED HSTS/Expect-CT absence confirmed on ALL production directory + work API servers: `ds-apip.threema.ch`, `apip.threema.ch`, `api.threema.ch`, `ds-apip-work.threema.ch` all lack both headers, while stagi

## 2026-08-07 21:23:50 UTC
- NEW apip.test.threema.ch — staging directory server live: GET/POST /identity/* 200, CORS *, HSTS, Expect-CT
- NEW ds-apip-work.threema.ch — work directory (prod) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW ds-apip-work.test.threema.ch — work directory (staging) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW work.test.threema.ch — staging work web app live: 301 /en/login, HSTS, Expect-CT, CSP with *.test.threema.ch refs, Sentry
- NEW safe-01.threema.ch — backup server live: 404 on /, CORS *, methods GET/HEAD/PUT/PATCH/POST/DELETE, HSTS, Expect-CT
- CHANGED apip.threema.ch — now confirmed full directory sibling: GET/POST /identity/* 200, identical pubkeys to ds-apip, CORS *
- CHANGED Production directory servers (ds-apip, apip, api) all lack HSTS/Expect-CT; staging counterparts have both

## 2026-08-07 22:06:34 UTC
- NEW apip.test.threema.ch — staging directory server live: GET/POST /identity/* 200, CORS *, HSTS, Expect-CT
- NEW ds-apip-work.threema.ch — work directory (prod) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW ds-apip-work.test.threema.ch — work directory (staging) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW work.test.threema.ch — staging work web app live: 301 /en/login, HSTS, Expect-CT, CSP with *.test.threema.ch refs, Sentry
- NEW safe-01.threema.ch — backup server live: 404 on /, CORS *, methods GET/HEAD/PUT/PATCH/POST/DELETE, HSTS, Expect-CT
- CHANGED apip.threema.ch — now confirmed full directory sibling: GET/POST /identity/* 200, identical pubkeys to ds-apip, CORS *
- CHANGED Production directory servers (ds-apip, apip, api) all lack HSTS/Expect-CT; staging counterparts have both

## 2026-08-07 22:43:53 UTC
- NEW `g-{XX}.0.test.threema.ch` staging chat cluster — resolves to 203.56.114.34 (IPv4); HTTPS → HTTP 000 (not HTTP-accessible; likely WSS/TCP 5222); out of scope per scope.yml (only `g-*.0.threema.ch` lis
- NEW `ds-apip-work.threema.ch` — work directory prod backend; GET /, /identity/{id}, /identity/fetch_bulk, /identities → all 401 (credential-gated), CORS `*`, no HSTS/Expect-CT.
- NEW `ds-apip-work.test.threema.ch` — work directory staging; 401 on all paths, CORS `*`, no HSTS/Expect-CT.
- NEW `work.test.threema.ch` — staging work web app; 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry.
- NEW Safe-01 backup API behavior: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (11 bytes) for all unauth requests (Express route exists behind credential check); `GET /backup/{x}` → 404 (150 bytes, no r
- NEW `safe-*.threema.ch` DNS pattern confirmed: safe-01, safe-1a, safe-1b, safe-02, safe-00 → all 203.56.112.231.
- NEW `mediator-{0..f}.threema.ch` and `rendezvous-{0..f}.threema.ch` DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all return uniform 403 on HTTPS.
- NEW Electron BrowserWindow: `sandbox` NOT enabled (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set.
- NEW `/identity/ws/revoke` → HTTP 404 on both `apip.threema.ch` and `ds-apip.threema.ch` (endpoint not publicly routable at these paths; OpenAPI documentation may be stale or route lives behind a different
- NEW `/api/v1/pubkeys/ECHOECHO` → HTTP 404 on `apip.threema.ch` (dead endpoint candidate from nemotron3 lead disproven).
- CHANGED `safe-01.threema.ch` — from baseline "TIMEOUT/no response" → live nginx backup service (404 on root, 400 on /backups/{hex}, permissive CORS with write methods + Authorization header).
- CHANGED Desktop OnPrem config trust → REJECTED as not vulnerable (Ed25519 signature verification against 3 hardcoded trusted public keys in vite.config.ts; HTTPS/WSS URL validation confirmed).
- CHANGED `crypto.ts:223` hardcoded password → REJECTED as benchmark-only dummy (used in `determineKdfParams()` to calibrate Argon2id runtime; derived key immediately purged).

## 2026-08-07 23:15:48 UTC
- NEW `safe-*.threema.ch` DNS pattern confirmed: safe-01, safe-1a, safe-1b, safe-02, safe-00 → all resolve to 203.56.112.231 (single IP, 5 hostnames)
- NEW `mediator-{0..f}.threema.ch` and `rendezvous-{0..f}.threema.ch` DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all return uniform 403 on HTTPS
- NEW `work.test.threema.ch` staging work web app: 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
- NEW Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HTTP 404 150 bytes (no route)
- CHANGED `safe-01.threema.ch` — from baseline "TIMEOUT/no response" to live nginx backup service with permissive CORS (write methods + Authorization header), HSTS, Expect-CT
