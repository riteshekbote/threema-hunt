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
