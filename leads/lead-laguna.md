# LEADS laguna (seed)
- SEED: no model output yet; pipeline starts on first run.
## 2026-08-07 18:40:35 UTC [desktop] (model laguna)
## 2026-08-07 18:55:02 UTC [desktop] (model laguna)
[NEW] `ds-apip.threema.ch` — canonical directory server hostname (source `config/vite.config.ts` + OpenAPI); public `GET /identity/{id}` returns 200/404 oracle.
[NEW] `mediator-{X}.threema.ch/{XX}/` hostname pattern (WSS sync server) — `mediator-*.threema.ch` in scope, pattern confirmed from client config.
[NEW] `safe-{XX}.threema.ch/` hostname pattern (backup safe) — `safe-*.threema.ch` in scope, pattern confirmed from client config.
[NEW] `rendezvous-{X}.threema.ch/{XX}/` hostname pattern (WSS linking server) — `rendezvous-*.threema.ch` in scope, pattern confirmed from client config.
[NEW] `api.threema.ch` — 403 + same permissive CORS as apip (candidate ID/directory sibling).
[CHANGED] `apip.threema.ch` — was 403 on `/`; now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*`.
[CHANGED] `work.threema.ch` / `shop.threema.ch` / `broadcast.threema.ch` / `gateway.threema.ch` — 301/302 now with session cookie, CSP, Sentry (was TIMEOUT/301).
[CHANGED] `billing.threema.ch` — 301 → `threema.ch`.
[NEW] `ds-apip.test.threema.ch` — leaked test/staging directory server reachable (static + live 200).
[PRIO] ds-apip.threema.ch (directory server API), 9.2
[PRIO] threema-desktop source (github.com/threema-ch/threema-desktop), 7.2
[PRIO] api.threema.ch (directory/ID service sibling), 6.9
[PRIO] safe-{XX}.threema.ch (backup service), 5.4
[HYP] Directory server identity enumeration via public `GET /identity/{id}`
class: IDOR
asset: https://ds-apip.threema.ch/identity/{identity}
confidence: 78
reasoning: Verified live: valid ID → 200 with publicKey; invalid ID → 404; no server-side 429 on 10× burst; permissive CORS `*`, so cross-origin from attacker page is possible. Endpoint is unauthenticated by design (directory). The gap is rate-limit/anti-automation enforcement, enabling mass valid-identity enumeration.
evidence_needed: 1) No 429 on burst > server capacity; 2) confirm `/identity/fetch_bulk` also unauth; 3) confirm bulk returns pubkey list.
verify_steps: PROBE: curl -s -w "\n%{http_code}" https://ds-apip.threema.ch/identity/fetch_bulk -X POST -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); observe whether bulk returns pubkeys for valid IDs without auth.
impact: Attacker enumerates valid Threema identities (→ pubkey) for targeted phishing/social-engineering. Severity: medium (privacy breach / recon).
testability: PASSIVE
[HYP] Desktop source: build-time env embedding test/staging server URLs + static chat server key
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/config/vite.config.ts` (147,153,154)
confidence: 60
reasoning: Source (read-only) bakes `DIRECTORY_SERVER_URL=https://ds-apip.test.threema.ch/`, `WORK_SERVER_LEGACY_URL`, `WORK_SERVER_URL=https://work.test.threema.ch/` into the built client. Verified live: `ds-apip.test.threema.ch/identity/ECHOECHO` → 200 (test dir server reachable). Leaking staging/test hostnames aids targeted attacks; if client trusts OnPrem config via `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` with weak validation, attacker-hosted config could redirect traffic (phishing/MITM).
evidence_needed: 1) Confirm OnPrem config fetch/verify path (`onprem-config-fetcher.ts`); 2) how `CHAT_SERVER_KEY` is verified (raw key vs cert pin) in `tls-cert-verifier.ts`/`electron-main.ts`.
verify_steps: RAG: read `apps/desktop/src/network/onprem-config-fetcher.ts` and `apps/desktop/src/network/tls-cert-verifier.ts`; static search for `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS`, `CHAT_SERVER_KEY`, `nodeIntegration`, `contextIsolation`, `sandbox`.
impact: If OnPrem trust relies on unpinned HTTP fetch or key-only verification, malicious config → MITM/phishing of all desktop traffic. Severity: high (mitm of desktop comms) if unconfirmed TLS.
testability: RAG
[HYP] Desktop key storage Windows file permissions not enforced
class: MISCONFIG
asset: `apps/desktop/src/common/node/key-storage/index.ts` (`FileSystemKeyStorage`)
confidence: 45
reasoning: `fileModeInternalObjectIfPosix()` → `0600` on POSIX only; Windows writes no mode → `keystorage.bin` readable by other local processes; password file uses Electron `safeStorage` (OS-level). Local-access prerequisite.
evidence_needed: Confirm Windows ACL behavior of the created files and that `electron.safeStorage` is actually used for the password file.
verify_steps: HUMAN_ONLY: on Windows, after app init, `icacls` the keystore files; attempt cross-process read of `keystorage.bin`.
impact: Local attacker reads encrypted keystore (Argon2id-protected); offline brute-force if weak password. Severity: medium.
testability: HUMAN_ONLY
[PARKED] Desktop key storage Windows permissions: confidence 45 < 50 and testability HUMAN_ONLY (non-reproducible on this Linux host); retained only for human follow-up.
[FINAL] (re-ranked top first)
[NEXT] PROBE: `curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ"]}'` — confirm the bulk-fetch endpoint returns public keys for valid IDs with no auth (strengthens enumeration impact from list-to-individual); then a single spaced follow-up to check `Access-Control-Expose-Headers` so CORS-based exfil from an attacker page is real.
## 2026-08-07 19:13:47 UTC [desktop] (model laguna)
[PRIO] ds-apip.threema.ch (directory server API), 8.8 — as:10 bv:9 tech:7 gate:10 cloud:5 fresh:10
[PRIO] api.threema.ch (directory server sibling), 8.5 — as:9 bv:9 tech:7 gate:10 cloud:5 fresh:9
[PRIO] threema-desktop source (Electron client), 7.0 — as:7 bv:8 tech:6 gate:8 cloud:3 fresh:8
[PRIO] safe-{XX}.threema.ch (backup service), 3.9 — as:4 bv:5 tech:3 gate:6 cloud:3 fresh:3
[HYP] Directory server identity enumeration via unauthenticated bulk-fetch + permissive CORS
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk (and /identity/{id})
confidence: 88
reasoning: Verified live: POST /identity/fetch_bulk with no auth → HTTP 200 returning pubkeys for valid IDs (ECHOECHO → publicKey returned); invalid IDs silently omitted from response. GET /identity/{id} returns 200/404 oracle. CORS Access-Control-Allow-Origin: * with DELETE/POST/GET/OPTIONS methods — cross-origin from an attacker page works without credentials. No 429 observed on burst. api.threema.ch is an identical sibling (same endpoints, same CORS).
evidence_needed: 1) Confirm rate-limit absence under sustained burst (>50 req/min sustained); 2) confirm /identity/fetch_bulk silently drops invalid IDs vs. returning null (confirmed: omitted).
verify_steps: PROBE: send 50 sequential POSTs to /identity/fetch_bulk at 1s intervals; log HTTP codes & response times for 429/RateLimit headers. Then PROBE: OPTIONS https://ds-apip.threema.ch/ to enumerate all CORS-exposed methods.
impact: Mass enumeration of all valid Threema identities → public keys, enabling targeted phishing/social-engineering and pubkey-collection for offline crypto attacks. Severity: medium (privacy breach / account recon).
testability: PASSIVE
[HYP] api.threema.ch directory server identity enumeration (confirmed full sibling)
class: IDOR
asset: https://api.threema.ch/identity/fetch_bulk
confidence: 82
reasoning: Confirmed identical to ds-apip: POST /identity/fetch_bulk → 200 with pubkey for valid ID; GET /identity/ECHOECHO → 200; CORS * with DELETE method. Same lack of auth/rate-limit. api.threema.ch was previously only a "candidate" — now confirmed as an active directory server.
evidence_needed: Confirm api.threema.ch is load-balanced same as ds-apip or a distinct server with its own identity space.
verify_steps: PROBE: GET https://api.threema.ch/ — observe 403 (no index); POST https://api.threema.ch/identity/fetch_bulk -d '{"identities":["ECHOECHO","INVALID9"]}' — observe selective return.
impact: Duplicate attack surface — same enumeration capability on a distinct hostname; if one is rate-limited, the other provides fallback. Severity: medium.
testability: PASSIVE
[HYP] Desktop source: OnPrem config trust path & key storage
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop (apps/desktop/config/vite.config.ts, apps/desktop/src/network/onprem-config-fetcher.ts, apps/desktop/src/common/node/key-storage/index.ts)
confidence: 60
reasoning: Source bakes DIRECTORY_SERVER_URL=https://ds-apip.test.threema.ch/ (staging hostname reachable live). OnPrem config via ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS — need to confirm fetch-and-verify path. Key storage: POSIX-only 0600 mode; Windows files have no ACL restriction.
evidence_needed: 1) Read onprem-config-fetcher.ts to see if config fetch is HTTP-pinnable; 2) confirm key-storage Windows ACL behavior.
verify_steps: RAG: read apps/desktop/src/network/onprem-config-fetcher.ts and apps/desktop/src/network/tls-cert-verifier.ts; search for ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS, CHAT_SERVER_KEY, nodeIntegration, contextIsolation, sandbox.
impact: If OnPrem trust relies on unpinned HTTP fetch or key-only verification, attacker-hosted config → MITM/phishing. Windows key storage → local attacker reads encrypted keystore. Severity: high (mitm) / medium (local).
testability: RAG / HUMAN_ONLY
[FINAL]
[NEXT] PROBE: send 30 sequential POSTs to https://ds-apip.threema.ch/identity/fetch_bulk at 2s intervals (≤1 rps) using a batch of 30 identities; log every HTTP code, response body, and any `Retry-After` / `X-RateLimit-*` headers to prove (or disprove) absence of rate-limiting.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth).
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin.
[LEARN] ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS headers as ds-apip.threema.ch.
[RISK] chat: 40 — g-* pattern hostnames discovered but not yet enumerated; exposure unknown.
[RISK] web: 85 — api.threema.ch + apip.threema.ch + ds-apip.threema.ch: three confirmed unauthenticated identity-enumeration endpoints with permissive CORS (* + DELETE), no rate-limit observed, bulk-fetch returns pubkeys. Highest web exposure.
[RISK] sync: 60 — mediator-*.threema.ch and rendezvous-*.threema.ch patterns known from client config; WSS servers likely have auth but pattern not yet probed.
[RISK] safe: 45 — safe-*.threema.ch pattern known from client config; safe.threema.ch timed out; pattern not yet enumerated.
[RISK] desktop-src: 75 — threema-desktop source confirmed in-scope; Electron attack surface + staging URL embedding + OnPrem config trust + key-storage Windows ACL concerns identified from source read; needs RAG to confirm.
