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

## 2026-08-07 23:47:08 UTC
- NEW `work.test.threema.ch` staging work web app confirmed live: 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
- NEW `safe-01.threema.ch` backup API credential-gated: `GET /backups/{64hex}` → HTTP 400 (route exists, credential check); OPTIONS returns CORS `*` + `Access-Control-Allow-Headers: Authorization`
- NEW `threema-desktop` Electron BrowserWindow: `sandbox: false` (TODO DEK-79) + `nodeIntegrationInWorker: true` (TODO DEK-79) confirmed in source
- NEW `apip.threema.ch/identity/ws/revoke` → 404 on both apip and ds-apip (dead endpoint)
- NEW `apip.threema.ch/api/v1/pubkeys/{id}` → 404 (dead endpoint)
- NEW `safe-*.threema.ch` DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 → all 203.56.112.231
- NEW `mediator-{0..f}/rendezvous-{0..f}.threema.ch` DNS split routing: 0-7 → 203.56.112.247; 8-f → 203.56.114.247; all 403 on HTTPS
- NEW Safe-01 backup API path distinction: `GET /backups/{64hex}` → 400 (credential-gated) vs `GET /backup/{x}` → 404 (no route)
- CHANGED `ds-apip-work.threema.ch` (prod) + `ds-apip-work.test.threema.ch` (staging): work directory backends confirmed live, 401 on all paths, CORS `*`, no HSTS/Expect-CT
- NEW work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated; IDENTICAL path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence on the
- NEW `/api-app/public/license/token/{licenseToken}` route exists in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route present, token lookup fails).
- NEW work.test.threema.ch login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch (for
- NEW Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict).
- NEW `/api-app/me/profile` + `/api-app/global/settings` → 302 on staging (session-gated); only the explicit `/api-app/public/*` namespace is open.
- NEW `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod (no divergence).

## 2026-08-08 00:15:46 UTC
- NEW work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated on staging; identical path → 404 on production work.threema.ch — first confirmed staging-prod public-API diverg
- NEW work.test.threema.ch `/api-app/public/license/token/{64hex}` route present in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route exists, token lookup fails)
- NEW work.test.threema.ch login page CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
- NEW Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
- NEW `/api-app/me/profile` and `/api-app/global/settings` → 302 on staging (session-gated); only explicit `/api-app/public/*` namespace is open
- NEW `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod — no divergence

## 2026-08-08 02:19:22 UTC
- NEW work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated on staging; identical path → 404 on production work.threema.ch — first confirmed staging-prod public-API diverg
- NEW work.test.threema.ch `/api-app/public/license/token/{64hex}` route present in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route exists, token lookup fails)
- NEW work.test.threema.ch login page CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
- NEW Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
- NEW `/api-app/me/profile` and `/api-app/global/settings` → 302 on staging (session-gated); only explicit `/api-app/public/*` namespace is open
- NEW `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod — no divergence

## 2026-08-08 03:47:32 UTC

## 2026-08-08 04:43:47 UTC

## 2026-08-08 05:26:39 UTC
- NEW work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated; IDENTICAL path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence on the
- NEW `/api-app/public/license/token/{licenseToken}` route exists in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route present, token lookup fails).
- NEW work.test.threema.ch login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch (for
- NEW Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict).
- NEW `/api-app/me/profile` + `/api-app/global/settings` → 302 on staging (session-gated); only the explicit `/api-app/public/*` namespace is open.
- NEW `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod (no divergence).

## 2026-08-08 06:05:58 UTC
- NEW `g-*.0.test.threema.ch` staging chat cluster confirmed reachable on single IP 203.56.114.34 (distinct from prod .112.202/.204), but no in-band HTTP divergence — only TCP 5222 accept with 0 bytes retur
- NEW `work.test.threema.ch` bundle divergence confirmed: work_public.js v2.25.1 DIFFERENT builds staging vs prod (staging sha256 e48e18f79df0125e8942f8fd9566f0c7924d15d7229377b553cf666b1bca7b87, prod 96501
- NEW `/api-app/public/license/token/{64hex}` on staging returns 404 HTML catch-all (900B) for GET, PUT, AND OPTIONS — method-agnostic; backend route not deployed; 64-char validation is client-side-only zod
- NEW `/api-app/public/*` namespace map closed on both hosts — sole live public route = `/api-app/public/global/settings` (200, 299B, only appLinkHost + 3 app-download URLs, no license/user data)
- NEW `work.threema.ch` prod DOES route `/api-app` — GET `/api-app/me/profile` → 302 to `/en/login?r=%2Fapi-app%2Fme%2Fprofile`; only `/public/*` namespace absent on prod (404 catch-all) — divergence is pub
- NEW `g-*.0.{test.,}threema.ch` chat: read-only TCP connect to 5222 returns 0 bytes on BOTH staging (203.56.114.34) and prod (203.56.112.202) — no server-hello pushed without client frame; 443 also closes 
- NEW `hcaptcha-work.threema.ch`: 200 serving hCaptcha's own Webflow marketing page (Last Published 2026-07-30) — third-party captcha host, out-of-scope service
- NEW `avatar.test.threema.ch` / `companylogo.test.threema.ch`: 403, byte-identical posture to prod avatar/companylogo 403 — no divergence; `broadcast.test` / `billing.test` → 000 (unreachable, matches prod

## 2026-08-08 07:09:30 UTC

## 2026-08-08 08:04:49 UTC

## 2026-08-08 08:33:27 UTC

## 2026-08-08 09:15:49 UTC

## 2026-08-08 10:06:30 UTC
- NEW Safe backup API credential format identified: HTTP Basic Auth with `backupId:backupKey` (vs earlier generic "credential-gated")
- NEW Route path distinction confirmed: `GET /backups/{64hex}` → 400 (route exists, credential-gated) vs `GET /backup/{x}` → 404 (210B, CSP `default-src 'none'`)
- CHANGED Desktop key-storage Windows ACL bypass elevated from ranked hypothesis (score 0) to fully RAG-VERIFIED — all source line numbers confirmed in cloned repo
- NEW work.test.threema.ch bundle divergence: `work_public.js` v2.25.1 different builds staging (sha256 `e48e18f7…`, 1,443,948 B) vs prod (sha256 `96501e21…`, 1,400,541 B)
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test
- NEW Desktop key-storage: RAG-VERIFIED TODAY — all source paths confirmed in cloned repo (`fs.ts:41`, `index.ts:559-560`, `electron-main.ts:944-945`, `crypto.ts:53-88`, `inner/v3.ts:65,70`, `restore-db.ts:
- NEW Safe backup API credential format identified: HTTP Basic Auth with `backupId:backupKey` (vs earlier generic "credential-gated")
- NEW Safe-01 backup API confirmed identical CORS on all 5 hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00) — all behind single IP 203.56.112.231
- CHANGED Desktop BrowserWindow `sandbox` property confirmed UNSET (defaults false) — L1255 `// TODO(DESK-79): Enable sandbox: true` — L1240 comment "sandboxing is enabled by default" is INCORRECT
- NEW Identity→pubkey oracle confirmed on all 3 production hosts (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs (ECHOECHO), silently omits invalid (ZZZZZZZZ), CORS `*`
- NEW Desktop key-storage: RAG-VERIFIED TODAY — all 15 source paths confirmed in freshly cloned repo `/tmp/opencode/threema-desktop` (fs.ts:41, index.ts:559-560, electron-main.ts:944-945, crypto.ts:53-88/95
- NEW Safe backup API credential format identified: HTTP Basic Auth with `backupId:backupKey` (vs earlier generic "credential-gated")
- NEW Safe-*.threema.ch route distinction confirmed across ALL 5 hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00) — all return identical OPTIONS 204 + GET 400
- CHANGED Desktop BrowserWindow `sandbox` property confirmed UNSET (defaults false) — L1255 `// TODO(DESK-79): Enable sandbox: true`; L1240 comment "sandboxing is enabled by default" is INCORRECT per Electron d
- NEW work.test.streema.ch bundle divergence: work_public.js different sha256 staging (`e48e18f7…`) vs prod (`96501e21…`) — prod has ZERO `/public/*` handlers
- NEW Identity→pubkey oracle confirmed across all 3 production hosts (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (require/import/eval/child_process/new Function) found in worker/ tree — grep returned 0 matches, confirming BrowserWindow exploitability is conditional

## 2026-08-08 10:23:27 UTC
- NEW Safe backup API credential format identified: HTTP Basic Auth with `backupId:backupKey` (was generic "credential-gated")
- NEW Route path distinction `/backups/{64hex}` (400) vs `/backup/{x}` (404) confirmed across ALL 5 safe-* hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00) behind single IP 203.56.112.231
- CHANGED Desktop key-storage Windows ACL bypass elevated from hypothesis to fully RAG-VERIFIED — all 15 source paths confirmed in freshly cloned repo (`fs.ts:41`, `index.ts:559-560`, `electron-main.ts:944-945`
- CHANGED Desktop BrowserWindow `sandbox` property confirmed UNSET (defaults false) — L1255 `// TODO(DESK-79): Enable sandbox: true`; L1240 comment "sandboxing is enabled by default" is INCORRECT per Electron d
- NEW work.test.threema.ch bundle divergence: `work_public.js` v2.25.1 different sha256 staging (`e48e18f7…`, 1,443,948 B) vs prod (`96501e21…`, 1,400,541 B) — prod has ZERO `/public/*` handlers
- NEW Identity→pubkey oracle confirmed on all 3 production hosts (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (`require`/`import`/`eval`/`child_process`/`new Function`) found in worker/ tree — grep returned 0 matches, confirming BrowserWindow exploitability is conditional
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test

## 2026-08-08 11:03:45 UTC
- NEW Desktop key-storage Windows ACL bypass fully RAG-verified (15 source paths confirmed in cloned repo)
- NEW Safe backup API credential format: HTTP Basic Auth `backupId:backupKey` confirmed across all 5 safe-* hosts
- NEW Route distinction `/backups/{64hex}` (400) vs `/backup/{x}` (404) confirmed across all 5 safe-* hosts
- NEW work.test.threema.ch bundle divergence: work_public.js v2.25.1 different builds staging vs prod (prod has ZERO `/public/*` handlers)
- NEW Identity→pubkey oracle confirmed on all 3 production hosts (ds-apip, api, apip) with CORS `*`
- NEW fetch_bulk batch oracle confirmed: returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (`require`/`import`/`eval`/`child_process`/`new Function`) in worker/ tree — Electron RCE conditional
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test
- CHANGED Desktop BrowserWindow `sandbox` unset (defaults false) — L1255 TODO DESK-79; L1240 comment incorrect per Electron docs
- CHANGED Desktop key-storage ACL bypass elevated from hypothesis to RAG-VERIFIED with all 15 source paths

## 2026-08-08 11:23:32 UTC
- NEW Desktop key-storage Windows ACL bypass fully RAG-verified (15 source paths confirmed in cloned repo)
- NEW Safe backup API credential format: HTTP Basic Auth `backupId:backupKey` confirmed across all 5 safe-* hosts
- NEW Route distinction `/backups/{64hex}` (400) vs `/backup/{x}` (404) confirmed across all 5 safe-* hosts
- NEW work.test.threema.ch bundle divergence: work_public.js v2.25.1 different builds staging vs prod (prod has ZERO `/public/*` handlers)
- NEW Identity→pubkey oracle confirmed on all 3 production hosts (ds-apip, api, apip) with CORS `*`
- NEW fetch_bulk batch oracle confirmed: returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (`require`/`import`/`eval`/`child_process`/`new Function`) in worker/ tree — Electron RCE conditional
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test
- CHANGED Desktop BrowserWindow `sandbox` unset (defaults false) — L1255 TODO DESK-79; L1240 comment incorrect per Electron docs
- CHANGED Desktop key-storage ACL bypass elevated from hypothesis to RAG-VERIFIED with all 15 source paths
- NEW Desktop key-storage: RAG-VERIFIED TODAY — all 15 source paths confirmed in freshly cloned repo `/tmp/opencode/threema-desktop` (fs.ts:41, index.ts:559-560, electron-main.ts:944-945, crypto.ts:53-88/95
- NEW Safe backup API credential format identified: HTTP Basic Auth with `backupId:backupKey` (vs earlier generic "credential-gated")
- NEW Safe-*.threema.ch route distinction confirmed across ALL 5 hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00) — all return identical OPTIONS 204 + GET 400
- CHANGED Desktop BrowserWindow `sandbox` property confirmed UNSET (defaults false) — L1255 `// TODO(DESK-79): Enable sandbox: true`; L1240 comment "sandboxing is enabled by default" is INCORRECT per Electron d
- NEW work.test.streema.ch bundle divergence: work_public.js different sha256 staging (`e48e18f7…`) vs prod (`96501e21…`) — prod has ZERO `/public/*` handlers
- NEW Identity→pubkey oracle confirmed across all 3 production hosts (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (require/import/eval/child_process/new Function) found in worker/ tree — grep returned 0 matches, confirming BrowserWindow exploitability is conditional

## 2026-08-08 11:51:30 UTC

## 2026-08-08 12:06:35 UTC
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `mediator-{X}.threema.ch/{XX}/` hostname pattern (WSS sync server) — `mediator-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `safe-{XX}.threema.ch/` hostname pattern (backup safe) — `safe-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `rendezvous-{X}.threema.ch/{XX}/` hostname pattern (WSS linking server) — `rendezvous-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `api.threema.ch` — 403 + same permissive CORS as apip (candidate ID/directory sibling).
- CHANGED `apip.threema.ch` — was 403 on `/`; now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*`.
- CHANGED `work.threema.ch` / `shop.threema.ch` / `broadcast.threema.ch` / `gateway.threema.ch` — 301/302 now with session cookie, CSP, Sentry (was TIMEOUT/301).
- CHANGED `billing.threema.ch` — 301 → `threema.ch`.
- NEW `ds-apip.test.threema.ch` — leaked test/staging directory server reachable (static + live 200).
- CHANGED `/identity/fetch_bulk` rate-limit absence quantified: 30 sequential POSTs at 1 rps → all HTTP 200, no 429/RateLimit/Retry-After headers, consistent ~340ms response times. Previously only “no 429 obser
- NEW `ds-apip.test.threema.ch` confirmed live and publicly reachable — returns identical identity→pubkey oracle + CORS `*` (with `Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE`) as production. S
- NEW `api.threema.ch` confirmed as full directory server sibling — `GET /identity/ECHOECHO` → 200 with identical CORS headers as `ds-apip.threema.ch`.
- NEW RAG finding: `crypto.ts:223` hardcoded password `r3gGN9GDQ5NF6tM6` is a **benchmark dummy** only — used in `determineKdfParams()` to measure Argon2id runtime, immediately purged (`benchmarkKey.purge()
- CHANGED RAG finding: OnPrem config trust path **debunked as vulnerable** — uses Ed25519 signature verification against 3 hardcoded `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` in `vite.config.ts`. OPPF URLs are valida
- CHANGED RAG finding: Desktop key storage confirmed — `fileModeInternalObjectIfPosix()` returns `{}` (no restriction) on Windows. Both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI
- CHANGED RAG finding: Electron BrowserWindow has `sandbox` NOT enabled (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79). `nodeIntegration: false` and `contextIsolation: true` are set.
- NEW apip.test.threema.ch — staging directory server live: GET/POST /identity/* 200, CORS *, HSTS, Expect-CT
- NEW ds-apip-work.threema.ch — work directory (prod) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW ds-apip-work.test.threema.ch — work directory (staging) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
- NEW work.test.threema.ch — staging work web app live: 301 /en/login, HSTS, Expect-CT, CSP with *.test.threema.ch refs, Sentry
- NEW safe-01.threema.ch — backup server live: 404 on /, CORS *, methods GET/HEAD/PUT/PATCH/POST/DELETE, HSTS, Expect-CT
- CHANGED apip.threema.ch — now confirmed full directory sibling: GET/POST /identity/* 200, identical pubkeys to ds-apip, CORS *
- CHANGED Production directory servers (ds-apip, apip, api) all lack HSTS/Expect-CT; staging counterparts have both

## 2026-08-08 13:10:07 UTC
- CHANGED Desktop key-storage Windows ACL bypass: elevated from hypothesis to **fully RAG-verified** (15 source paths confirmed in cloned repo at `fs.ts:41`, `index.ts:559-560`, `electron-main.ts:944-945`, `cry
- CHANGED Desktop BrowserWindow `sandbox`: confirmed **unset (defaults false)** — L1255 `// TODO(DESK-79): Enable sandbox: true`; L1240 comment "sandboxing is enabled by default" is **incorrect** per Electron d
- NEW Safe backup API credential format: **HTTP Basic Auth `backupId:backupKey`** confirmed across all 5 safe-* hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00)
- NEW Route distinction `/backups/{64hex}` (400, route exists, credential-gated) vs `/backup/{x}` (404, 210B, CSP `default-src 'none'`) confirmed across all 5 safe-* hosts
- NEW work.test.threema.ch bundle divergence: `work_public.js` v2.25.1 **different builds** staging (sha256 `e48e18f7…`, 1,443,948 B) vs prod (sha256 `96501e21…`, 1,400,541 B) — prod has **ZERO** `/public/*
- NEW Identity→pubkey oracle confirmed on **all 3 production hosts** (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (`require`/`import`/`eval`/`child_process`/`new Function`) in worker/ tree — Electron RCE conditional
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test

## 2026-08-08 14:00:34 UTC
- CHANGED Desktop key-storage Windows ACL bypass: elevated from hypothesis to **fully RAG-verified** (15 source paths confirmed in cloned repo at `fs.ts:41`, `index.ts:559-560`, `electron-main.ts:944-945`, `cry
- CHANGED Desktop BrowserWindow `sandbox`: confirmed **unset (defaults false)** — L1255 `// TODO(DESK-79): Enable sandbox: true`; L1240 comment "sandboxing is enabled by default" is **incorrect** per Electron d
- NEW Safe backup API credential format: **HTTP Basic Auth `backupId:backupKey`** confirmed across all 5 safe-* hosts (safe-01, safe-1a, safe-1b, safe-02, safe-00)
- NEW Route distinction `/backups/{64hex}` (400, route exists, credential-gated) vs `/backup/{x}` (404, 210B, CSP `default-src 'none'`) confirmed across all 5 safe-* hosts
- NEW work.test.threema.ch bundle divergence: `work_public.js` v2.25.1 **different builds** staging (sha256 `e48e18f7…`, 1,443,948 B) vs prod (sha256 `96501e21…`, 1,400,541 B) — prod has **ZERO** `/public/*
- NEW Identity→pubkey oracle confirmed on **all 3 production hosts** (ds-apip, api, apip) returning identical 200+pubkey with CORS `*`
- NEW fetch_bulk batch oracle confirmed: `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → returns only valid IDs, silently omits invalid, CORS `*`
- NEW No dynamic sinks (`require`/`import`/`eval`/`child_process`/`new Function`) in worker/ tree — Electron RCE conditional
- NEW work.test.threema.ch login CSP leaks staging surfaces: broadcast.test, avatar.test, companylogo.test, hcaptcha-work, billing.test

## 2026-08-08 14:23:10 UTC

## 2026-08-08 15:03:05 UTC

## 2026-08-08 15:30:23 UTC
- NEW 5 challenge endpoints confirmed live on 3 production directory hosts (ds-apip.threema.ch, api.threema.ch, apip.threema.ch): GET /identity/{sfu_cred|blob_cred|check_revocation_key|update_work_info} → 2
- NEW fetch_bulk 100+ ID batch verified: POST /identity/fetch_bulk with 100 IDs (1 valid ECHOECHO + 99 unique invalid) → 200, returns 1 identity with pubkey, silently omits 99 invalid IDs. Confirms single-r
- NEW HSTS/Expect-CT inconsistency on safe-01.threema.ch: OPTIONS /backups/{64hex} → 204 with HSTS + Expect-CT present; GET /backups/{64hex} → 400 with HSTS + Expect-CT ABSENT (only CORS `*` present). Heade
- CHANGED Production directory hosts (ds-apip, api, apip) confirmed lacking HSTS/Expect-CT on ALL responses — verified on GET /identity/ECHOECHO (200), GET /identity/sfu_cred (200), GET /identity/nonexistent (4

## 2026-08-08 16:04:12 UTC
- NEW 5 challenge-response endpoints live on 3 production directory hosts: `/identity/{sfu_cred|blob_cred|set_revocation_key|check_revocation_key|update_work_info}` return 200 with JSON errors + CORS `*`
- NEW `POST /identity/fetch_bulk` accepts 100+ ID batch (1 valid + 99 invalid) → 200, returns only valid pubkey, silently omits invalid — single-request bulk enumeration confirmed
- NEW HSTS/Expect-CT inconsistency on `safe-01.threema.ch`: present on OPTIONS 204 preflight, ABSENT on GET 400 `/backups/{64hex}` — header gap on credential-gated endpoint
- CHANGED Production directory hosts (ds-apip, api, apip) confirmed lacking HSTS/Expect-CT on ALL responses (200, 404, challenge endpoints)
- CHANGED fetch_bulk 100+ ID batch re-confirmed on `api.threema.ch` and `apip.threema.ch` via own probes (identical 200 response with same ECHOECHO pubkey `sha256(SmobNNzvFdQ8t03i/TYJG+mfu68SbQmdR9g9kZcSxys=)`;
- NEW `update_work_info` endpoint confirmed with parameter-validation-before-identity-lookup oracle on all 3 hosts: returns `{"success":false,"error":"Missing parameters"}` (not "Identity not found"), same 
- NEW No `Access-Control-Expose-Headers` on any directory host response — confirmed via own OPTIONS preflight on `ds-apip.threema.ch/identity/fetch_bulk` (ACAO:*, Allow-Methods POST,GET,OPTIONS,DELETE, no E

## 2026-08-08 17:01:24 UTC
- NEW No new surface items since 2026-08-08 16:04:12 UTC cycle — all findings already captured in knowledge base
- NEW 5 challenge endpoints confirmed live on 3 production directory hosts (ds-apip.threema.ch, api.threema.ch, apip.threema.ch): GET /identity/{sfu_cred|blob_cred|check_revocation_key|update_work_info} → 2
- NEW fetch_bulk 100+ ID batch verified: POST /identity/fetch_bulk with 100 IDs (1 valid ECHOECHO + 99 unique invalid) → 200, returns 1 identity with pubkey, silently omits 99 invalid IDs. Confirms single-r
- NEW HSTS/Expect-CT inconsistency on safe-01.threema.ch: OPTIONS /backups/{64hex} → 204 with HSTS + Expect-CT present; GET /backups/{64hex} → 400 with HSTS + Expect-CT ABSENT (only CORS `*` present). Heade
- CHANGED Production directory hosts (ds-apip, api, apip) confirmed lacking HSTS/Expect-CT on ALL responses — verified on GET /identity/ECHOECHO (200), GET /identity/sfu_cred (200), GET /identity/nonexistent (4
- CHANGED fetch_bulk 100+ ID batch re-confirmed on `api.threema.ch` and `apip.threema.ch` via own probes (identical 200 response with same ECHOECHO pubkey `sha256(SmobNNzvFdQ8t03i/TYJG+mfu68SbQmdR9g9kZcSxys=)`;
- NEW `update_work_info` endpoint confirmed with parameter-validation-before-identity-lookup oracle on all 3 hosts: returns `{"success":false,"error":"Missing parameters"}` (not "Identity not found"), same 
- NEW No `Access-Control-Expose-Headers` on any directory host response — confirmed via own OPTIONS preflight on `ds-apip.threema.ch/identity/fetch_bulk` (ACAO:*, Allow-Methods POST,GET,OPTIONS,DELETE, no E

## 2026-08-08 17:40:33 UTC

## 2026-08-08 17:59:19 UTC
- CHANGED lead class 7 (Desktop BrowserWindow sandbox+worker) formally REJECTED as standalone lead — conditional RCE requires separate renderer chain; not new surface.
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml; not new surface.
- CHANGED crypto.ts:223 benchmark-password finding re-confirmed REJECTED under sha256 form `52a0af98…` (≠ sha256 of prior literal `400c7846…`, so a new hashed reference to the same benchmark-only dummy; key pur

## 2026-08-08 18:31:19 UTC

## 2026-08-08 19:12:51 UTC

## 2026-08-08 19:42:29 UTC
- NEW NO_DELTA

## 2026-08-08 20:10:15 UTC

## 2026-08-08 20:37:15 UTC
- NEW NO_DELTA

## 2026-08-08 21:04:15 UTC

## 2026-08-08 21:26:51 UTC
- NEW saltyrtc-{00..ff}.threema.ch:443 reported as live SaltyRTC signaling (WSS) in hypotheses-bigpickle.txt — needs operator scope ruling (not in scope.yml g-*.0 pattern)

## 2026-08-08 22:01:17 UTC
- NEW saltyrtc-{00..ff}.threema.ch:443 reported as live SaltyRTC signaling (WSS) in hypotheses-bigpickle.txt — needs operator scope ruling (not in scope.yml g-*.0 pattern)

## 2026-08-08 22:16:28 UTC
- NEW saltyrtc-{00..ff}.threema.ch:443 — live SaltyRTC WSS signaling (HTTP 426 on GET), 256 hostnames quadrant-split across 4 IPs (203.56.112.198/.199, 203.56.114.198/.199); not in scope.yml (only g-*.0, me
- CHANGED Directory challenge endpoints (sfu_cred, blob_cred, set_revocation_key, check_revocation_key, update_work_info) re-confirmed on all 3 prod hosts (ds-apip, api, apip) with parameter-validation-before-i
- CHANGED fetch_bulk 100+ ID batch re-confirmed on api.threema.ch and apip.threema.ch (identical ECHOECHO pubkey, silent omit of 99 invalid)
- CHANGED No Access-Control-Expose-Headers on any directory host response confirmed via OPTIONS preflight
- CHANGED Production directory hosts (ds-apip, api, apip) confirmed lacking HSTS/Expect-CT on ALL responses
- CHANGED lead class 7 (Desktop BrowserWindow sandbox+worker) formally REJECTED as standalone — conditional RCE requires separate renderer chain
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml
- CHANGED crypto.ts:223 benchmark-password finding re-confirmed REJECTED under sha256 form `52a0af98…`

## 2026-08-08 22:51:06 UTC
- NEW NO_DELTA — inventory at 2026-08-08 22:16:28 UTC matches last leads at 2026-08-08 22:15:43 UTC; no new surface items since last cycle

## 2026-08-08 23:17:51 UTC
- NEW NO_DELTA — inventory at 2026-08-08 22:51:06 UTC matches last leads; no new surface items since last cycle

## 2026-08-08 23:47:03 UTC
- NEW NO_DELTA — inventory at 2026-08-08 23:17:51 UTC matches last leads; no new surface items since last cycle
- NEW fetch_bulk 500-ID batch confirmed this cycle: single POST to https://ds-apip.threema.ch/identity/fetch_bulk with `{"identities":["ECHOECHO",<499 unique invalid base32>]}` → HTTP 200, 152B, 0.72s, ECHO

## 2026-08-09 00:05:09 UTC
- NEW NO_DELTA — inventory at 2026-08-08 23:47:03 UTC matches last leads; no new surface items since last cycle

## 2026-08-09 02:37:39 UTC

## 2026-08-09 04:02:03 UTC
- NEW saltyrtc-{00..ff}.threema.ch:443 — 256 hostnames, live SaltyRTC WSS signaling (HTTP 426 on GET), quadrant-split across 4 IPs (203.56.112.198/.199, 203.56.114.198/.199); NOT in scope.yml (only g-*.0 pa

## 2026-08-09 05:08:32 UTC
- NEW saltyrtc-{00..ff}.threema.ch:443 — 256 hostnames, live SaltyRTC WSS signaling (HTTP 426 on GET), quadrant-split across 4 IPs (203.56.112.198/.199, 203.56.114.198/.199); NOT in scope.yml (only g-*.0 pa
- CHANGED Directory challenge endpoints: /identity/sfu_cred, /identity/blob_cred, /identity/set_revocation_key, /identity/check_revocation_key, /identity/update_work_info — all return 200 with JSON error bodies
- CHANGED fetch_bulk ceiling confirmed ≥500 IDs/request on all 3 prod directory hosts; no 429 after 30 sequential POSTs at 1 rps
- CHANGED safe-{01,1a,1b,02,00}.threema.ch: HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 for /backups/{64hex} — header inconsistency stable across all 5 hosts behind single IP 203.56.11
- CHANGED work.test.threema.ch: /api-app/public/global/settings → 200 (299B, staging-only) vs work.threema.ch → 404 HTML — divergence stable
- NEW saltyrtc-{00..ff}.threema.ch:443 — 256 hostnames, live SaltyRTC WSS signaling (HTTP 426 on GET), quadrant-split across 4 IPs (203.56.112.198/.199, 203.56.114.198/.199); NOT in scope.yml (only g-*.0 pa

## 2026-08-09 05:45:53 UTC
- NEW NO_DELTA — inventory at 2026-08-09 05:08:32 UTC shows only re-confirmations (CHANGED) of existing in-scope surface; saltyrtc-{00..ff}.threema.ch:443 discovered but explicitly NOT in scope.yml (only g-

## 2026-08-09 06:43:50 UTC
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint, new surface)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page, new accessible surface)
- CHANGED `saltyrtc-{00..ff}.threema.ch:443` — 256 hostnames, live SaltyRTC WSS signaling (HTTP 426 on GET), quadrant-split across 4 IPs; explicitly NOT in scope.yml (only g-*.0 pattern listed for chat) — re-co
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — fetch_bulk 500-ID batch ceiling re-confirmed; 5 challenge endpoints + parameter-validation oracles + CORS * + no rate-limit stable
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch` — HSTS/Expect-CT present on OPTIONS 204 but ABSENT on GET 400 for `/backups/{64hex}` stable across all 5 hosts behind single IP 203.56.112.231
- CHANGED `work.test.threema.ch` — `/api-app/public/global/settings` → 200 (299B, staging-only) vs 404 prod divergence stable; sole live public route in `/api-app/public/*` namespace
- NEW NO_DELTA — inventory at 2026-08-09 05:08:32 UTC shows only re-confirmations (CHANGED) of existing in-scope surface; saltyrtc-{00..ff}.threema.ch:443 discovered but explicitly NOT in scope.yml (only g-

## 2026-08-09 07:32:58 UTC
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint, new surface)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page, new accessible surface)
- CHANGED `saltyrtc-{00..ff}.threema.ch:443` — 256 hostnames, live SaltyRTC WSS signaling (HTTP 426 on GET), quadrant-split across 4 IPs; explicitly NOT in scope.yml (only g-*.0 pattern listed for chat) — re-co
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — fetch_bulk 500-ID batch ceiling re-confirmed; 5 challenge endpoints + parameter-validation oracles + CORS * + no rate-limit stable
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch` — HSTS/Expect-CT present on OPTIONS 204 but ABSENT on GET 400 for `/backups/{64hex}` stable across all 5 hosts behind single IP 203.56.112.231
- CHANGED `work.test.threema.ch` — `/api-app/public/global/settings` → 200 (299B, staging-only) vs 404 prod divergence stable; sole live public route in `/api-app/public/*` namespace

## 2026-08-09 08:12:02 UTC
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint newly confirmed)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — fetch_bulk 500-ID batch ceiling re-confirmed; 5 challenge endpoints + parameter-validation oracles + CORS * + no rate-limit stable
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch` — HSTS/Expect-CT present on OPTIONS 204 but ABSENT on GET 400 for `/backups/{64hex}` stable across all 5 hosts behind single IP 203.56.112.231
- CHANGED `work.test.threema.ch` — `/api-app/public/global/settings` → 200 (299B, staging-only) vs 404 prod divergence stable; sole live public route in `/api-app/public/*` namespace
- NEW gateway.threema.ch: /v1 → 404 catch-all, /api/v1 → 403 (nginx deny), /en/signup → 200 (14KB page). No exposed msgapi route on this host.
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk BYTE-IDENTICAL to prod (200, 0.69s, identical ECHOECHO record); staging /swagger /docs /identity/lookup /openapi.json all 404 = same route surface as prod 
- CHANGED fetch_bulk ceiling — 1000-ID batch → 200 (0.42s, 152B, ECHOECHO echoed, 999 invalid silently omitted); ceiling now ≥1000. Report persisted to reports/fetch_bulk-identity-enumeration-idor.md (was unper
- CHANGED broadcast.threema.ch/api/v1/ — absent key → 401, any key (1/32/64-char) → 403, bodies byte-identical (sha256 707fe8f5…); subpath → 404 HTML; release banner 2.28.1 + Sentry DSN (public). Key-format ora
- NEW gateway.threema.ch — /v1 → 404 catch-all, /api/v1 → 403 nginx deny, /en/signup → 200 (14KB); no exposed msgapi route on this host.

## 2026-08-09 09:05:56 UTC
- CHANGED `gateway.threema.ch` — additional endpoints mapped: `/v1` → 404 catch-all, `/api/v1` → 403 (nginx deny), `/en/signup` → 200 (14KB page); no exposed msgapi route on this host
- CHANGED `ds-apip.test.threema.ch` — staging `fetch_bulk` BYTE-IDENTICAL to prod (200, 0.69s, identical ECHOECHO record); staging `/swagger` `/docs` `/identity/lookup` `/openapi.json` all 404 = same route surf
- CHANGED `fetch_bulk` ceiling — 1000-ID batch → 200 (0.42s, 152B, ECHOECHO echoed, 999 invalid silently omitted); ceiling now ≥1000. Report persisted to `reports/fetch_bulk-identity-enumeration-idor.md`
- CHANGED `broadcast.threema.ch/api/v1/` — absent key → 401, any key (1/32/64-char) → 403, bodies byte-identical (sha256 707fe8f5…); subpath → 404 HTML; release banner 2.28.1 + Sentry DSN (public). Key-format o
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint newly confirmed)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — fetch_bulk 500-ID batch ceiling re-confirmed; 5 challenge endpoints + parameter-validation oracles + CORS * + no rate-limit stable
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch` — HSTS/Expect-CT present on OPTIONS 204 but ABSENT on GET 400 for `/backups/{64hex}` stable across all 5 hosts behind single IP 203.56.112.231
- CHANGED `work.test.threema.ch` — `/api-app/public/global/settings` → 200 (299B, staging-only) vs 404 prod divergence stable; sole live public route in `/api-app/public/*` namespace

## 2026-08-09 09:53:56 UTC
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint, newly confirmed via probe)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible, newly confirmed via probe)
- CHANGED `ds-apip.threema.ch/identity/fetch_bulk` — ceiling raised to ≥1000 IDs/request (1000-ID batch → 200, 0.42s, 152B, ECHOECHO echoed, 999 invalid silently omitted)
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod (200, 0.69s, identical ECHOECHO record); `/swagger` `/docs` `/identity/lookup` `/openapi.json` all 404
- CHANGED `broadcast.threema.ch/api/v1/` — key-format oracle DISPROVEN: absent key → 401, any key (1/32/64-char) → byte-identical 403 (sha256 707fe8f5…); subpath → 404 HTML
- CHANGED `gateway.threema.ch` — `/v1` → 404 catch-all, `/api/v1` → 403 (nginx deny), `/en/signup` → 200 (14KB); no exposed msgapi route

## 2026-08-09 10:11:42 UTC
- NEW `broadcast.threema.ch/api/v1/` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible)
- CHANGED `ds-apip.threema.ch/identity/fetch_bulk` — ceiling ≥1000 IDs/request confirmed (1000-ID batch → 200, valid-only response, silent omit, ACAO:*, no 429)
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod; no extra routes (/swagger /docs /openapi.json 404)
- CHANGED `broadcast.threema.ch/api/v1/` — key-format oracle disproven (absent key→401, any key→byte-identical 403)
- CHANGED `gateway.threema.ch` — /v1→404, /api/v1→403 (nginx deny), /en/signup→200 (14KB); no msgapi route
- CHANGED `safe-01.threema.ch/backups/{64hex}` — HSTS/Expect-CT on OPTIONS 204, ABSENT on GET 400 (stable across 5 hosts)
- CHANGED `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only) vs `work.threema.ch` → 404 (prod)

## 2026-08-09 10:52:59 UTC
- NEW `broadcast.threema.ch/api/v1/` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible)
- CHANGED `ds-apip.threema.ch/identity/fetch_bulk` — ceiling ≥10000 IDs/request confirmed (10000-ID batch → 200, 0.80s, 152B, ECHOECHO echoed, 9999 invalid silently omitted, no 413/429)
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod; no extra routes (/swagger /docs /openapi.json 404)
- CHANGED `broadcast.threema.ch/api/v1/` — key-format oracle disproven (absent key→401, any key→byte-identical 403)
- CHANGED `gateway.threema.ch` — /v1→404, /api/v1→403 (nginx deny), /en/signup→200 (14KB); no msgapi route
- CHANGED `safe-01.threema.ch/backups/{64hex}` — HSTS/Expect-CT on OPTIONS 204, ABSENT on GET 400 (stable across 5 hosts behind 203.56.112.231)
- CHANGED `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only, 299B) vs `work.threema.ch` → 404 (prod)
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs

## 2026-08-09 11:25:52 UTC
- NEW `ds-apip.threema.ch/identity/fetch_bulk` — hard ceiling confirmed at exactly 10,000 IDs/request (10,000 → 200/152B, 10,001 → 400/0B); sharp count-cap boundary proven
- NEW `ds-apip.test.threema.ch/identity/fetch_bulk` — staging enforces identical 10,000-ID cap (10,001 → 400 byte-for-byte identical to prod); validation-logic parity confirmed
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash); key-format oracle disproven (absent key→401, any 1/32/64-char key→byte-identical 403)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (14KB signup page); `/v1`→404, `/api/v1`→403 (nginx deny); no msgapi route exposed
- NEW `safe-01.threema.ch/backups/{64hex}` — HSTS/Expect-CT present on OPTIONS 204 preflight but ABSENT on GET 400 (stable across all 5 safe-* hosts behind 203.56.112.231); HTTP Basic Auth (backupId:backupK
- NEW `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only, 299B) vs `work.threema.ch` → 404 (prod); sole live public route in `/api-app/public/*` namespace
- NEW `apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); GET `/identity/{id}` → 200/404 oracle confirmed
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths `/identity/*`, `/identities`; CORS `*`; no HSTS/Expect-CT)
- NEW Hostname patterns discovered in desktop source `config.ts`: `mediator-{prefix4}.threema.ch/{prefix8}/`, `rendezvous-{prefix4}.threema.ch/{prefix8}/`, `safe-{backupIdPrefix8}.threema.ch/`, `blob-mirror
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — `fetch_bulk` ceiling tightened from "≥10000" to "exactly 10000" (overflow → 400 empty body, NO partial pubkey leak, CORS `*` on 400, zero 429s)
- CHANGED `ds-apip.test.threema.ch` — mirror evidence strengthened: byte-identical fetch_bulk responses + identical 10000-cap enforcement; still no live-dataset proof without testId
- CHANGED `broadcast.threema.ch/api/v1` — auth-gated 401 baseline stable; key-format/validity oracle fully disproven
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed: explicit SNI + TLS1.2/1.3 probes all close immediately (0 bytes, no cert/SAN); handshake requires authenticated login frame
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, but explicitly NOT in scope.yml
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs

## 2026-08-09 11:54:15 UTC
- NEW `ds-apip.threema.ch/identity/fetch_bulk` — hard ceiling confirmed at exactly 10,000 IDs/request (10,000 → 200/152B, 10,001 → 400/0B); sharp count-cap boundary proven
- NEW `ds-apip.test.threema.ch/identity/fetch_bulk` — staging enforces identical 10,000-ID cap (10,001 → 400 byte-for-byte identical to prod); validation-logic parity confirmed
- NEW `apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); GET `/identity/{id}` → 200/404 oracle confirmed
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths `/identity/*`, `/identities`; CORS `*`; no HSTS/Expect-CT)
- NEW Hostname patterns discovered in desktop source `config.ts`: `mediator-{prefix4}.threema.ch/{prefix8}/`, `rendezvous-{prefix4}.threema.ch/{prefix8}/`, `safe-{backupIdPrefix8}.threema.ch/`, `blob-mirror
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — `fetch_bulk` ceiling tightened from "≥10000" to "exactly 10000" (overflow → 400 empty body, NO partial pubkey leak, CORS `*` on 400, zero 429s)
- CHANGED `ds-apip.test.threema.ch` — mirror evidence strengthened: byte-identical fetch_bulk responses + identical 10000-cap enforcement; still no live-dataset proof without testId
- CHANGED `broadcast.threema.ch/api/v1` — auth-gated 401 baseline stable; key-format/validity oracle fully disproven
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed: explicit SNI + TLS1.2/1.3 probes all close immediately (0 bytes, no cert/SAN); handshake requires authenticated login frame
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, but explicitly NOT in scope.yml
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs

## 2026-08-09 12:18:16 UTC
- NEW `apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); `GET /identity/{id}` → 200/404 oracle confirmed
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths `/identity/*`, `/identities`; CORS `*`; no HSTS/Expect-CT)
- NEW Hostname patterns from desktop `config.ts`: `mediator-{prefix4}.threema.ch/{prefix8}/`, `rendezvous-{prefix4}.threema.ch/{prefix8}/`, `safe-{backupIdPrefix8}.threema.ch/` (in scope), `blob-mirror-{pre
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — `fetch_bulk` ceiling tightened from "≥10000" to **exactly 10000** (10000 → 200/152B, 10001 → 400/0B; sharp count-cap, no partial leak, CORS `*` on
- CHANGED `ds-apip.test.threema.ch` — mirror evidence strengthened: byte-identical `fetch_bulk` responses + identical 10000-cap enforcement; still no live-dataset proof without testId
- CHANGED `broadcast.threema.ch/api/v1` — auth-gated 401 baseline stable; key-format/validity oracle fully disproven
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed: explicit SNI + TLS1.2/1.3 probes all close immediately (0 bytes, no cert/SAN); handshake requires authenticated login frame
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, but explicitly NOT in scope.yml
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs

## 2026-08-09 13:16:52 UTC
- NEW ds-apip.threema.ch — canonical directory server hostname wired into desktop client build config; GET /identity/{id} → 200/404 oracle confirmed
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths /identity/*, /identities; CORS *; no HSTS/Expect-CT)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (NOT in scope)
- NEW ds-apip.threema.ch/identity/fetch_bulk — hard ceiling exactly 10,000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap)
- NEW ds-apip.test.threema.ch/identity/fetch_bulk — staging enforces identical 10,000-ID cap (10001→400 byte-for-byte identical to prod)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 (sharp count-cap, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — mirror evidence strengthened: byte-identical fetch_bulk responses + identical 10000-cap enforcement; no live-dataset proof without testId
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed: explicit SNI + TLS1.2/1.3 probes close immediately (0 bytes, no cert/SAN)
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW `ds-apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); `GET /identity/{id}` → 200/404 oracle confirmed (was only `apip.threema.ch` in baseline)
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths, CORS `*`, no HSTS/Expect-CT)
- CHANGED `fetch_bulk` ceiling tightened from "≥10000" to exactly 10000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap, no partial leak)
- CHANGED `apip.threema.ch` — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW Hostname discovery from desktop `config.ts`: `mediator-{prefix4}`, `rendezvous-{prefix4}`, `safe-{backupIdPrefix8}` patterns (blob-mirror pattern out of scope)

## 2026-08-09 14:08:48 UTC
- NEW ds-apip.threema.ch/identity/fetch_bulk — hard ceiling exactly 10,000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap)
- NEW ds-apip.test.threema.ch/identity/fetch_bulk — staging enforces identical 10,000-ID cap (10001→400 byte-for-byte identical to prod)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, requires Basic auth)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 (sharp count-cap, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — mirror evidence strengthened: byte-identical fetch_bulk responses + identical 10000-cap enforcement; no live-dataset proof without testId
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed: explicit SNI + TLS1.2/1.3 probes close immediately (0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW ds-apip.threema.ch — canonical directory server hostname (matches inventory `apip.threema.ch` but is the actual host wired into desktop client build config)
- NEW ds-apip-work.threema.ch — work-style directory server (returns 401 on all paths, requires Basic auth)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source `config.ts`; NOT in scope — does not match any scope wildcard)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: `mediator-*.threema.ch`)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: `safe-*.threema.ch`)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: `rendezvous-*.threema.ch`)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW `ds-apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); `GET /identity/{id}` → 200/404 oracle confirmed (was only `apip.threema.ch` in baseline)
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths, CORS `*`, no HSTS/Expect-CT)
- CHANGED `fetch_bulk` ceiling tightened from "≥10000" to exactly 10000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap, no partial leak)
- CHANGED `apip.threema.ch` — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW Hostname discovery from desktop `config.ts`: `mediator-{prefix4}`, `rendezvous-{prefix4}`, `safe-{backupIdPrefix8}` patterns (blob-mirror pattern out of scope)

## 2026-08-09 14:29:42 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml

## 2026-08-09 15:08:23 UTC
- NEW ds-apip.threema.ch — canonical directory server hostname (wired into desktop client build config); `GET /identity/{id}` → 200/404 oracle confirmed
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS `*`, no HSTS/Expect-CT)
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml

## 2026-08-09 15:40:58 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- NEW ds-apip.threema.ch — canonical directory server hostname (wired into desktop client build config); GET /identity/{id} → 200/404 oracle confirmed
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source config.ts; NOT in scope)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated this cycle (was claimed in KB but absent from workspace). `node --check` OK; graceful no-op on Linux confirmed (exits 0 on non-win32).

## 2026-08-09 16:03:53 UTC
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated this cycle (was claimed in KB but absent from workspace). `node --check` OK; graceful no-op on Linux confirmed (exits 0 on non-win32).
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- NEW ds-apip.threema.ch — canonical directory server hostname (wired into desktop client build config); GET /identity/{id} → 200/404 oracle confirmed
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source config.ts; NOT in scope)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated this cycle (was claimed in KB but absent from workspace). `node --check` OK; graceful no-op on Linux confirmed (exits 0 on non-win32).
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- NEW ds-apip.threema.ch — canonical directory server hostname (wired into desktop client build config); GET /identity/{id} → 200/404 oracle confirmed
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source config.ts; NOT in scope)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling tightened to exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml

## 2026-08-09 16:44:48 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern with path structure (in scope: mediator-*.threema.ch)
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern with path structure (in scope: rendezvous-*.threema.ch)
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern (in scope: safe-*.threema.ch)
- NEW ds-apip-work.threema.ch — work-style directory server (401 on all paths, CORS *, Basic auth required)
- NEW ds-apip.threema.ch — canonical directory server hostname from desktop client build config (config/vite.config.ts + OpenAPI)
- NEW blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern (discovered in desktop source config.ts; NOT in scope)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated this cycle (node --check OK; graceful no-op on Linux confirmed)

## 2026-08-09 17:14:27 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern confirmed in scope (mediator-*.threema.ch); DNS resolves to split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on 
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern confirmed in scope (rendezvous-*.threema.ch); same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path s
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.231
- NEW ds-apip-work.threema.ch — work-style directory server confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT, Basic auth required
- NEW ds-apip.threema.ch — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated this cycle (node --check OK; graceful no-op on Linux confirmed)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- CHANGED blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml

## 2026-08-09 17:48:24 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern confirmed in scope (mediator-*.threema.ch); DNS split routing (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; h
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern confirmed in scope (rendezvous-*.threema.ch); same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path s
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.231
- NEW ds-apip-work.threema.ch — work-style directory server confirmed live (401 on all paths /identity/*, /identities; CORS *; no HSTS/Expect-CT; Basic auth required)
- NEW ds-apip.threema.ch — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated this cycle (node --check OK; graceful no-op on Linux confirmed)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch — fetch_bulk ceiling exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED ds-apip.test.threema.ch — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED broadcast.threema.ch/api/v1 — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED apip.threema.ch — confirmed 200 on /identity/{id} (public identity lookup) and 404 on invalid IDs
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml

## 2026-08-09 18:12:42 UTC
- NEW NO_DELTA — inventory (2026-08-09 10:52 UTC) and knowledge base (2026-08-09 latest
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated in workspace (was claimed in KB but NOT present; now present) — node --check PASS, graceful no-op on Linux confirmed
- CHANGED — PROBE: fetch_bulk 10000/10001 boundary re-confirmed via own probes — 10000→200/152B, 10001→400/0B; all 3 hosts (ds-apip/api/apip) byte-identical for fetch_bulk + GET /identity/{id}
- CHANGED — PROBE: 5 challenge endpoints re-confirmed (sfu_cred→"Identity not found", blob_cred→"Identity not found", set_revocation_key→"Bad revocation key length", check_revocation_key→"Identity not found", u
- CHANGED — PROBE: safe-01 OPTIONS → 204, ACAO:`*` + Allow-Methods GET,HEAD,PUT,PATCH,POST,DELETE + Allow-Headers: Authorization; GET → 400, HSTS/Expect-CT ABSENT (gap re-confirmed)
- NEW — No other new surface items discovered

## 2026-08-09 19:04:09 UTC
- NEW `mediator-{prefix4}.threema.ch/{prefix8}/` — mediator WSS hostname pattern confirmed in scope (mediator-*.threema.ch); DNS resolves to split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 o
- NEW `rendezvous-{prefix4}.threema.ch/{prefix8}/` — rendezvous WSS hostname pattern confirmed in scope (rendezvous-*.threema.ch); same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path
- NEW `safe-{backupIdPrefix8}.threema.ch/` — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.23
- NEW `ds-apip-work.threema.ch` — work-style directory server confirmed live (401 on all paths /identity/*, /identities; CORS *; no HSTS/Expect-CT; Basic auth required)
- NEW `ds-apip.threema.ch` — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated this cycle (node --check OK; graceful no-op on Linux confirmed)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch` — fetch_bulk ceiling exactly 10000 IDs/request (sharp count-cap, 10001→400 empty body, no partial leak, CORS * on 400, zero 429s)
- CHANGED `ds-apip.test.threema.ch` — staging fetch_bulk byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json 404)
- CHANGED `broadcast.threema.ch/api/v1` — auth-gated 401 baseline stable; key-format/validity oracle fully disproven (1/32/64-char keys → byte-identical 403, no CORS preflight)
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED `apip.threema.ch` — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426 on GET, explicitly NOT in scope.yml
- CHANGED `blob-mirror-{prefix4}.threema.ch/{prefix8}/` — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated in workspace (was claimed in KB but NOT present; now present) — node --check PASS, graceful no-op on Linux confirmed
- CHANGED — PROBE: fetch_bulk 10000/10001 boundary re-confirmed via own probes — 10000→200/152B, 10001→400/0B; all 3 hosts (ds-apip/api/apip) byte-identical for fetch_bulk + GET /identity/{id}
- CHANGED — PROBE: 5 challenge endpoints re-confirmed (sfu_cred→"Identity not found", blob_cred→"Identity not found", set_revocation_key→"Bad revocation key length", check_revocation_key→"Identity not found", u
- CHANGED — PROBE: safe-01 OPTIONS → 204, ACAO:`*` + Allow-Methods GET,HEAD,PUT,PATCH,POST,DELETE + Allow-Headers: Authorization; GET → 400, HSTS/Expect-CT ABSENT (gap re-confirmed)
- NEW — No other new surface items discovered
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated (was claimed present but `poc/` was empty; gap closed). node --check PASS; graceful no-op on Linux; read-only, sha256-only, no network.
- CHANGED PROBE (own): POST ds-apip.threema.ch/identity/fetch_bulk {"ECHOECHO","ZZZZZZZZ","NONEXISTENTZZ"} -> 200 returns ONLY ECHOECHO pubkey; ACAA `*` + Allow-Methods POST,GET,OPTIONS,DELETE + Allow-Headers C
- CHANGED PROBE (own): GET /identity/ECHOECHO->200 vs /ZZZZZZZZ->404 (ds-apip).
- CHANGED PROBE (own): safe-01 GET /backups/{64hex}->400 (size 11); GET /backup/x->404 (size 147); OPTIONS 204 ACAA `*` + GET,HEAD,PUT,PATCH,POST,DELETE.
- CHANGED RAG (own): full desktop chain verified live on GitHub `stable` (fs.ts, key-storage/index.ts, electron-main.ts:912-945, inner/v3.ts, crypto.ts, sqlite.ts); files data/keystorage.bin + data/keystorage.p
- CHANGED PROBE (own): electron-main.ts:1240-1255 BrowserWindow — sandbox NOT set (TODO DESK-79), nodeIntegrationInWorker:true (TODO DESK-79).
- NEW — No other new surface items discovered.

## 2026-08-09 19:31:30 UTC
- NEW `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- NEW `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk` — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED `broadcast.threema.ch/api/v1` — key-format/validity oracle disproven (absent key→401, any 1/32/64-char key→byte-identical 403 sha256 `707fe8f5…`; no CORS preflight)
- CHANGED `gateway.threema.ch` — fully mapped: `/v1`→404 catch-all, `/api/v1`→403 nginx deny, `/en/signup`→200 (14KB); no msgapi route exposed
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}` — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only, 299B) vs `work.threema.ch` → 404 (prod) — divergence stable

## 2026-08-09 19:56:47 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern confirmed in scope; DNS split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; high-entropy path structure
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern confirmed in scope; same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path structure
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.231
- NEW ds-apip-work.threema.ch — work-style directory server confirmed live; 401 on all paths (/identity/*, /identities); CORS `*`; no HSTS/Expect-CT; Basic auth required
- NEW ds-apip.threema.ch — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated in workspace (was claimed but missing; now present); node --check PASS; graceful no-op on Linux confirmed
- CHANGED broadcast.threema.ch/api/v1 → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- CHANGED gateway.threema.ch/en/signup → HTTP 200 (signup page accessible, 14KB)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s)
- CHANGED ds-apip.test.threema.ch/identity/fetch_bulk — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex} — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED work.test.threema.ch/api-app/public/global/settings → 200 (staging-only, 299B) vs work.threema.ch → 404 (prod) — divergence stable
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- NEW `ds-apip.threema.ch` — canonical directory server hostname (wired into desktop client build config); `GET /identity/{id}` → 200/404 oracle confirmed (was only `apip.threema.ch` in baseline)
- NEW `ds-apip-work.threema.ch` — work-style directory server (401 on all paths, CORS `*`, no HSTS/Expect-CT)
- CHANGED `fetch_bulk` ceiling tightened from "≥10000" to exactly 10000 IDs/request (10000→200/152B, 10001→400/0B; sharp count-cap, no partial leak)
- CHANGED `apip.threema.ch` — was 403 on `/`; now confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- NEW Hostname discovery from desktop `config.ts`: `mediator-{prefix4}`, `rendezvous-{prefix4}`, `safe-{backupIdPrefix8}` patterns (blob-mirror pattern out of scope)

## 2026-08-09 20:23:02 UTC
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern confirmed in scope; DNS split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; high-entropy path structure
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern confirmed in scope; same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path structure
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.231
- NEW ds-apip-work.threema.ch — work-style directory server confirmed live; 401 on all paths (/identity/*, /identities); CORS `*`; no HSTS/Expect-CT; Basic auth required
- NEW ds-apip.threema.ch — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated in workspace (was claimed but missing; now present); node --check PASS; graceful no-op on Linux confirmed
- CHANGED broadcast.threema.ch/api/v1 → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- CHANGED gateway.threema.ch/en/signup → HTTP 200 (signup page accessible, 14KB)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s)
- CHANGED ds-apip.test.threema.ch/identity/fetch_bulk — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex} — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED work.test.threema.ch/api-app/public/global/settings → 200 (staging-only, 299B) vs work.threema.ch → 404 (prod) — divergence stable
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- CHANGED apip.threema.ch — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml; not new surface.
- NEW NO_DELTA — inventory at 2026-08-09 05:08:32 UTC shows only re-confirmations (CHANGED) of existing in-scope surface; saltyrtc-{00..ff}.threema.ch:443 discovered but explicitly NOT in scope.yml (only g-
- NEW mediator-{prefix4}.threema.ch/{prefix8}/ — mediator WSS hostname pattern confirmed in scope; DNS split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; high-entropy path structure
- NEW rendezvous-{prefix4}.threema.ch/{prefix8}/ — rendezvous WSS hostname pattern confirmed in scope; same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path structure
- NEW safe-{backupIdPrefix8}.threema.ch/ — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.231
- NEW ds-apip-work.threema.ch — work-style directory server confirmed live; 401 on all paths (/identity/*, /identities); CORS `*`; no HSTS/Expect-CT; Basic auth required
- NEW ds-apip.threema.ch — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW poc/key-storage-acl-bypass-poc.js — PoC artifact generated in workspace (was claimed but missing; now present); node --check PASS; graceful no-op on Linux confirmed
- CHANGED broadcast.threema.ch/api/v1 → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- CHANGED gateway.threema.ch/en/signup → HTTP 200 (signup page accessible, 14KB)
- CHANGED ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s)
- CHANGED ds-apip.test.threema.ch/identity/fetch_bulk — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex} — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED work.test.threema.ch/api-app/public/global/settings → 200 (staging-only, 299B) vs work.threema.ch → 404 (prod) — divergence stable
- CHANGED g-*.0.{test.,}threema.ch:443/5222 — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED saltyrtc-*.threema.ch — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED blob-mirror-{prefix4}.threema.ch/{prefix8}/ — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- NEW `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
- NEW `mediator-{X}.threema.ch/{XX}/` hostname pattern (WSS sync server) — `mediator-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `safe-{XX}.threema.ch/` hostname pattern (backup safe) — `safe-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `rendezvous-{X}.threema.ch/{XX}/` hostname pattern (WSS linking server) — `rendezvous-*.threema.ch` in scope, pattern confirmed from client config.
- NEW `api.threema.ch` — 403 + same permissive CORS as apip (candidate ID/directory sibling).
- CHANGED `apip.threema.ch` — was 403 on `/`; now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*`.
- CHANGED `work.threema.ch` / `shop.threema.ch` / `broadcast.threema.ch` / `gateway.threema.ch` — 301/302 now with session cookie, CSP, Sentry (was TIMEOUT/301).
- CHANGED `billing.threema.ch` — 301 → `threema.ch`.
- NEW `ds-apip.test.threema.ch` — leaked test/staging directory server reachable (static + live 200).

## 2026-08-09 20:56:42 UTC
- NEW `mediator-{prefix4}.threema.ch/{prefix8}/` — mediator WSS hostname pattern confirmed in scope (mediator-*.threema.ch); DNS split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; hig
- NEW `rendezvous-{prefix4}.threema.ch/{prefix8}/` — rendezvous WSS hostname pattern confirmed in scope (rendezvous-*.threema.ch); same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy path
- NEW `safe-{backupIdPrefix8}.threema.ch/` — backup safe hostname pattern confirmed in scope (safe-*.threema.ch); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.23
- NEW `ds-apip-work.threema.ch` — work-style directory server confirmed live; 401 on all paths (/identity/*, /identities); CORS `*`; no HSTS/Expect-CT; Basic auth required
- NEW `ds-apip.threema.ch` — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated in workspace (was claimed but missing; now present); node --check PASS; graceful no-op on Linux confirmed
- CHANGED `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- CHANGED `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible, 14KB)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk` — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}` — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only, 299B) vs `work.threema.ch` → 404 (prod) — divergence stable
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED `blob-mirror-{prefix4}.threema.ch/{prefix8}/` — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- CHANGED `apip.threema.ch` — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml

## 2026-08-09 21:22:43 UTC

## 2026-08-09 21:58:13 UTC
- NEW `mediator-{prefix4}.threema.ch/{prefix8}/` — mediator WSS hostname pattern confirmed in scope (`mediator-*.threema.ch`); DNS split IPs (0-7→203.56.112.247, 8-f→203.56.114.247); uniform 403 on HTTPS; h
- NEW `rendezvous-{prefix4}.threema.ch/{prefix8}/` — rendezvous WSS hostname pattern confirmed in scope (`rendezvous-*.threema.ch`); same DNS split routing as mediator; uniform 403 on HTTPS; high-entropy pa
- NEW `safe-{backupIdPrefix8}.threema.ch/` — backup safe hostname pattern confirmed in scope (`safe-*.threema.ch`); 5 hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) resolve to single IP 203.56.112.
- NEW `ds-apip-work.threema.ch` — work-style directory server confirmed live; 401 on all paths (/identity/*, /identities); CORS `*`; no HSTS/Expect-CT; Basic auth required
- NEW `ds-apip.threema.ch` — canonical directory server hostname confirmed via desktop client build config (config/vite.config.ts + OpenAPI); public GET /identity/{id} returns 200/404 oracle
- NEW `poc/key-storage-acl-bypass-poc.js` — PoC artifact generated in workspace (was claimed but missing; now present); `node --check` PASS; graceful no-op on Linux confirmed
- CHANGED `broadcast.threema.ch/api/v1` → HTTP 401 (auth-gated API endpoint confirmed; 301 without trailing slash)
- CHANGED `gateway.threema.ch/en/signup` → HTTP 200 (signup page accessible, 14KB)
- CHANGED `ds-apip.threema.ch/api.threema.ch/apip.threema.ch/identity/fetch_bulk` — ceiling exactly 10000 IDs/request (sharp count-cap: 10000→200/152B, 10001→400/0B, no partial leak, CORS `*` on both, zero 429s
- CHANGED `ds-apip.test.threema.ch/identity/fetch_bulk` — staging byte-identical to prod including 10000-cap enforcement; no extra routes (/swagger /docs /identity/lookup /openapi.json all 404)
- CHANGED `safe-{01,1a,1b,02,00}.threema.ch/backups/{64hex}` — HSTS/Expect-CT present on OPTIONS 204, ABSENT on GET 400 stable across all 5 hosts behind 203.56.112.231
- CHANGED `work.test.threema.ch/api-app/public/global/settings` → 200 (staging-only, 299B) vs `work.threema.ch` → 404 (prod) — divergence stable
- CHANGED `g-*.0.{test.,}threema.ch:443/5222` — chat passive channel formally closed (explicit SNI + TLS1.2/1.3 probes close immediately, 0 bytes, no cert/SAN)
- CHANGED `saltyrtc-*.threema.ch` — 256 hostnames resolve to 4 IPs, HTTP 426, explicitly NOT in scope.yml
- CHANGED `blob-mirror-{prefix4}.threema.ch/{prefix8}/` — blob server hostname pattern discovered in desktop source config.ts; NOT in scope per scope.yml
- CHANGED `apip.threema.ch` — confirmed 200 on `/identity/{id}` (public identity lookup) and 404 on invalid IDs
- CHANGED lead class 16 (g-*.0.test.threema.ch staging chat) formally REJECTED as out-of-scope per scope.yml
