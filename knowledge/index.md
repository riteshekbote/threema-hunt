# Knowledge Base (seed)

## Program rules (from scope.yml)
- In scope: Android source, iOS source, Desktop 2.x source (threema-desktop), g-*.0.threema.ch, mediator-*.threema.ch, rendezvous-*.threema.ch, apip.threema.ch, safe-*.threema.ch, work.threema.ch, broadcast.threema.ch, gateway.threema.ch, shop.threema.ch, billing.threema.ch
- ALL other targets and third-party services are OUT of scope
- Passive-first: GET/HEAD only, <=1 rps, no account creation, no data modification
- Secrets in commits: sha256 only, never raw

## Baseline surface (2026-08-07 passive recon)
- apip.threema.ch: HTTP 403 on /
- work.threema.ch: 301 (redirect)
- shop.threema.ch: 301 (redirect)
- broadcast.threema.ch / gateway.threema.ch / billing.threema.ch: TIMEOUT (retry; note gating)
- GitHub org threema-ch (23 repos): threema-android (Kotlin), threema-ios (Swift), threema-desktop (TS), threema-web (TS), threema-web-electron, threema-msgapi-sdk-{php,java,python,rust}, threema-bot-sdk, push-relay (Rust), app-remote-protocol, webrtc-build-docker, apns-h2, etc.

## Rejected / parked
- (none yet)
- 2026-08-07 ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration enabling cross-origin API probes confirmed via passive HEAD/GET.
- 2026-08-07 ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
- 2026-08-07 ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive probes.
- 2026-08-07 ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth).
- 2026-08-07 ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin.
- 2026-08-07 ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS headers as ds-apip.threema.ch.
- 2026-08-07 REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
- 2026-08-07 REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
- 2026-08-07 ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
- 2026-08-07 ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
- 2026-08-07 ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
- 2026-08-07 ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
- 2026-08-07 ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
- 2026-08-07 ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPTIONS preflight returns CORS `*` with `Access-Control-Allow-Headers: Authorization` enabling credentialed cross-origin requests.
- 2026-08-07 ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` + `contextIsolation: true` are set.
- 2026-08-07 ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT.
- 2026-08-07 REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis mis-targeted.
- 2026-08-07 REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven.
- 2026-08-07 ACCEPTED MISCONFIG @ g-{XX}.0.test.threema.ch: Staging chat cluster resolves to 203.56.114.34 (split from prod chat IPs 203.56.112.202/.204); HTTPS not HTTP-accessible (likely WSS/TCP 5222); staging chat hostnames may be out of scope per scope.yml.
- 2026-08-07 ACCEPTED OTHER @ ds-apip-work.test.threema.ch: Work directory staging backend confirmed live — 401 on all paths, CORS `*`, no HSTS/Expect-CT
- 2026-08-07 ACCEPTED OTHER @ work.test.threema.ch: Staging work web app confirmed live — 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
- 2026-08-07 ACCEPTED OTHER @ safe-*.threema.ch DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 all resolve to 203.56.112.231 (single IP, 5 hostnames)
- 2026-08-07 ACCEPTED OTHER @ mediator-{0..f}/rendezvous-{0..f}.threema.ch DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all uniform 403 on HTTPS
- 2026-08-07 ACCEPTED OTHER @ Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HTTP 404 150 bytes (no route)
