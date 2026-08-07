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
## 2026-08-07 20:10:06 UTC [desktop] (model laguna)
[CHANGED] `/identity/fetch_bulk` rate-limit absence quantified: 30 sequential POSTs at 1 rps → all HTTP 200, no 429/RateLimit/Retry-After headers, consistent ~340ms response times. Previously only “no 429 observed on burst.”
[NEW] `ds-apip.test.threema.ch` confirmed live and publicly reachable — returns identical identity→pubkey oracle + CORS `*` (with `Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE`) as production. Staging hostname is baked into `vite.config.ts:147` (`DIRECTORY_SERVER_URL: 'https://ds-apip.test.threema.ch/'`).
[NEW] `api.threema.ch` confirmed as full directory server sibling — `GET /identity/ECHOECHO` → 200 with identical CORS headers as `ds-apip.threema.ch`.
[NEW] RAG finding: `crypto.ts:223` hardcoded password `r3gGN9GDQ5NF6tM6` is a **benchmark dummy** only — used in `determineKdfParams()` to measure Argon2id runtime, immediately purged (`benchmarkKey.purge()`). NOT a real secret. (Debunks `analysis.txt` REPORT_CANDIDATE.)
[CHANGED] RAG finding: OnPrem config trust path **debunked as vulnerable** — uses Ed25519 signature verification against 3 hardcoded `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` in `vite.config.ts`. OPPF URLs are validated for HTTPS/WSS protocols. The original confidence-60 hypothesis is refuted.
[CHANGED] RAG finding: Desktop key storage confirmed — `fileModeInternalObjectIfPosix()` returns `{}` (no restriction) on Windows. Both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-via-`electron.safeStorage`-encrypted) are written without Windows ACL restrictions. Any same-user process can read both files and decrypt the password via DPAPI.
[CHANGED] RAG finding: Electron BrowserWindow has `sandbox` NOT enabled (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79). `nodeIntegration: false` and `contextIsolation: true` are set.
[PRIO] ds-apip.threema.ch (directory server API), 8.8 — as:10 bv:9 tech:7 gate:10 cloud:5 fresh:10
[PRIO] threema-desktop source (Electron client), 7.5 — as:8 bv:8 tech:7 gate:8 cloud:3 fresh:9
[PRIO] ds-apip.test.threema.ch (staging directory server), 6.3 — as:6 bv:5 tech:5 gate:10 cloud:3 fresh:9
[HYP] Desktop Windows key storage: keystorage.bin and password file lack ACL restrictions
class: MISCONFIG
asset: apps/desktop/src/common/node/key-storage/index.ts + apps/desktop/src/electron/electron-main.ts + apps/desktop/src/common/node/fs.ts
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows (no POSIX mode set), so `keystorage.bin` is written with default Windows ACLs. `_writeOrOverrideFile()` in index.ts passes this empty object to `fsPromises.writeFile()`. The password file `keystorage.password.bin` is encrypted via `electron.safeStorage.encryptString()` (DPAPI on Windows) but is also written with `fileModeInternalObjectIfPosix()` → no ACL restriction. Any same-user process can read both files and decrypt the password via DPAPI, then use it to decrypt the Argon2id-encrypted `keystorage.bin` containing private keys and DB encryption key.
evidence_needed: Confirm on real Windows: (1) `icacls` on `keystorage.bin` and `keystorage.password.bin` after app init; (2) cross-process read of `keystorage.password.bin` succeeds and `electron.safeStorage.decryptString()` works from another same-user process.
verify_steps: HUMAN_ONLY: on Windows, launch Threema Desktop, enter/create a profile. From a separate process running as the same user: run `icacls %APPDATA%\ThreemaDesktop\*\data\keystorage.password.bin`, then attempt `CryptUnprotectData` via Python `win32crypt` to recover the plaintext password, then attempt read of `keystorage.bin`.
impact: Local attacker with same-user process access extracts all private keys, database encryption key, device credentials. Severity: medium (requires local access, same-user privilege).
testability: HUMAN_ONLY
[HYP] Directory server identity enumeration: no rate limiting + permissive CORS + silent invalid-ID omission
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk && https://api.threema.ch/identity/fetch_bulk && https://apip.threema.ch/identity/fetch_bulk
confidence: 95
reasoning: PROBE confirmed: 30 sequential POSTs at 1 rps to /identity/fetch_bulk → all HTTP 200. No 429, no X-RateLimit, no Retry-After headers. Response times stable at ~340ms (no throttling). Invalid IDs silently omitted from response (not returned as null), enabling silent enumeration. CORS `Access-Control-Allow-Origin: *` with `DELETE/POST/GET/OPTIONS` methods on all three hostnames. All three hostnames return identical pubkeys for `ECHOECHO`. No `Access-Control-Expose-Headers` needed — response body is directly readable cross-origin without credentials.
evidence_needed: Sustained burst >50 req/min to rule out adaptive rate limiting; confirm bulk-fetch silently omits invalid IDs vs. error.
verify_steps: PROBE: send 50 sequential POSTs to /identity/fetch_bulk at 1s intervals with 50 random identity candidates; confirm all return HTTP 200 with no 429; verify response JSON contains only valid identities (invalid ones omitted).
impact: Mass enumeration of all valid Threema identities → public keys, enabling targeted phishing/social-engineering and pubkey collection for offline crypto attacks. Severity: medium (privacy breach / account reconnaissance).
testability: PASSIVE
[HYP] Staging directory server exposed + HSTS/CT absent on production
class: MISCONFIG
asset: https://ds-apip.test.threema.ch/ (baked into vite.config.ts:147 sandbox build)
confidence: 65
reasoning: Sandbox build bakes `DIRECTORY_SERVER_URL='https://ds-apip.test.threema.ch/'` into the client. PROBE confirmed: staging server is publicly reachable, returns identical identity→pubkey oracle + CORS `*` as production. Notably, staging returns HSTS (`strict-transport-security: max-age=31104000; includeSubdomains`) and Expect-CT headers, while production `ds-apip.threema.ch` returns neither. Staging may have test accounts or debug endpoints not hardened for public access.
evidence_needed: Compare feature levels / API response differences between staging and production for same identity; check staging for debug endpoints or test data.
verify_steps: PROBE: `curl -s -X POST https://ds-apip.test.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO"]}'`; compare response body (featureLevel, featureMask, type, state) with production `ds-apip.threema.ch`; then PROBE: diff the full response headers including HSTS/Expect-CT.
impact: Attacker gains access to staging infrastructure that may expose test accounts, debug endpoints, or pre-release features. Staging without rate limiting = fallback enumeration vector if production is ever protected. Severity: low-medium.
testability: PASSIVE
[PARKED] Desktop key storage Windows permissions (original): Refiled as NEW hypothesis with increased confidence (45→75) based on RAG confirmation of `fileModeInternalObjectIfPosix()` returning `{}` on Windows and `electron.safeStorage` usage confirmed in `electron-main.ts:944-945`. Testability remains HUMAN_ONLY.
[PARKED] Desktop OnPrem config trust path: CONFIDENCE REFUTED. RAG in `onprem/oppf.ts` confirms `verifyOppfFile()` uses Ed25519 signature verification against 3 hardcoded trusted public keys (`ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` in `vite.config.ts`), with HTTPS/WSS URL protocol validation and license expiration checks. Not vulnerable. No further investigation warranted.
[PARKED] Desktop crypto.ts:223 hardcoded password: REFUTED. The password `r3gGN9GDQ5NF6tM6` (sha256: `400c78464a1785c7d692121f7e852b422bc208efc08fa2286fb68f5ba1b9ae12`) is used solely as a dummy input for benchmarking Argon2id KDF runtime in `determineKdfParams()`. The derived key is immediately purged (`benchmarkKey.purge()`) and never persisted or used for encryption. The original `analysis.txt` REPORT_CANDIDATE is a false positive.
[FINAL] (re-ranked top first)
[NEXT] PROBE: `curl -s -o /dev/null -w 'HTTP %{http_code} | time_total: %{time_total}s | headers: %{header_json}\n' -X POST https://ds-apip.test.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO"]}'` then compare the `featureLevel`/`featureMask`/`type`/`state` values against the production response from `ds-apip.threema.ch` to detect any test/dev accounts or debug data leakage on the staging endpoint.
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
[LEARN] ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[RISK] chat: 40 — g-*.0.threema.ch pattern not yet enumerated; exposure unknown.
[RISK] web: 90 — api.threema.ch + apip.threema.ch + ds-apip.threema.ch: three confirmed unauthenticated identity-enumeration endpoints with permissive CORS (`*`), DELETE/POST/GET/OPTIONS methods, zero rate-limiting (30 POSTs at 1 rps all 200), bulk-fetch returns pubkeys, invalid IDs silently omitted. Highest web exposure; all three hostnames load-balanced identically.
[RISK] sync: 60 — mediator-*.threema.ch and rendezvous-*.threema.ch patterns known from client config (vite.config.ts); WSS servers likely have auth but pattern not yet probed.
[RISK] safe: 45 — safe-*.threema.ch pattern known from client config; safe.threema.ch timed out; staging pattern `safe-{backupIdPrefix8}.threema.ch` confirmed in vite.config.ts but not yet enumerated.
[RISK] desktop-src: 75 — threema-desktop source confirmed in-scope; Electron BrowserWindow has `sandbox` not enabled (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79) — known gaps, not active vulns. Windows key storage files lack ACL restrictions (confirmed via RAG). OnPrem config trust verified secure (Ed25519 sig verification). Staging hostname baked into builds exposes test infrastructure.
## 2026-08-07 20:56:51 UTC [desktop] (model laguna)
[PRIO] directory server IDOR (5 hosts: ds-apip/api/apip + .test), 8.8 — as:10 bv:9 tech:7 gate:10 cloud:5 fresh:10
[PRIO] staging directory server exposure (2 hosts), 7.7 — as:8 bv:6 tech:8 gate:10 cloud:5 fresh:10
[PRIO] threema-desktop key storage Windows ACL, 7.5 — as:8 bv:8 tech:7 gate:8 cloud:3 fresh:9
[PRIO] work directory server CORS (ds-apip-work ±staging), 6.1 — as:6 bv:7 tech:6 gate:3 cloud:5 fresh:10
[NEW] `apip.test.threema.ch` — staging directory server confirmed live: `GET /identity/ECHOECHO` → 200, `POST /identity/fetch_bulk` → 200, CORS `*`, HSTS, Expect-CT, returns identical pubkey data as production `ds-apip.threema.ch`
[NEW] `ds-apip-work.threema.ch` (prod) — work directory server confirmed live: 401 on `/` + `/identity/{id}` + `/identity/fetch_bulk`(404), CORS `*` with DELETE/POST/GET/OPTIONS, no HSTS/Expect-CT
[NEW] `ds-apip-work.test.threema.ch` (staging) — work directory server confirmed live: 401 on `/`, CORS `*`, no HSTS/Expect-CT
[NEW] `work.test.threema.ch` (staging) — work web app confirmed live: 301 to `/en/login`, HSTS, Expect-CT, CSP with staged subdomain references
[CHANGED] Production `apip.threema.ch` confirmed as full directory server sibling — `GET /identity/ECHOECHO` → 200 with identical pubkey data to `ds-apip.threema.ch` (was previously only confirmed as 403 on `/` at baseline)
[CHANGED] HSTS/Expect-CT absence confirmed on ALL production directory + work API servers: `ds-apip.threema.ch`, `apip.threema.ch`, `api.threema.ch`, `ds-apip-work.threema.ch` all lack both headers, while staging directory servers (`ds-apip.test.threema.ch`, `apip.test.threema.ch`) include both
[HYP] Directory server identity enumeration across 5 hostnames
class: IDOR
asset: ds-apip.threema.ch + api.threema.ch + apip.threema.ch + ds-apip.test.threema.ch + apip.test.threema.ch (GET /identity/{id}, POST /identity/fetch_bulk)
confidence: 95
reasoning: All 5 directory server hosts expose unauthenticated identity→pubkey lookup. GET /identity/{id} returns 200/404 oracle. POST /identity/fetch_bulk returns pubkeys for valid IDs only (invalid silently omitted). CORS Access-Control-Allow-Origin:* with DELETE/POST/GET/OPTIONS. 50 sequential POSTs at 1 rps (30 prod + 20 staging) → all HTTP 200, zero 429/RateLimit/Retry-After headers. Response times stable ~300-340ms (no throttling). Production hosts lack HSTS/Expect-CT; staging hosts include both but identical CORS exposure.
evidence_needed: Sustained burst >50 req/min; confirm bulk-fetch silently omits invalid IDs vs. error response.
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.test.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234","INVALID"]}' (≤1 rps); verify response JSON contains only ECHOECHO (invalid IDs omitted) with no 429.
impact: Mass enumeration of all valid Threema identities → public keys at scale for targeted phishing/social-engineering and offline crypto attacks. Severity: medium.
testability: PASSIVE
[HYP] Staging directory server public exposure + HSTS/header inconsistency
class: MISCONFIG
asset: apip.test.threema.ch + ds-apip.test.threema.ch (staging directory servers, baked into vite.config.ts)
confidence: 65
reasoning: apip.test.threema.ch (NEW, not in prior KB) confirmed as full directory server sibling with identical API surface + CORS * + identity→pubkey oracle as production, plus HSTS + Expect-CT headers. Staging returns identical pubkey data for ECHOECHO (no test accounts found in batch comparison). Production directory servers lack HSTS/Expect-CT while staging includes both — inconsistent defense-in-depth posture. Staging /.well-known/* paths return 403 (nginx deny) while production returns 404.
evidence_needed: Diff all API response fields between staging and production for same valid identity; enumerate 20+ identity candidates on staging-only to detect test accounts.
verify_steps: PROBE: diff <(curl -s -D - -X POST https://apip.test.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO"]}') <(curl -s -D - -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO"]}') ; then PROBE: enumerate 20 random identity candidates on apip.test vs ds-apip to detect staging-only accounts.
impact: Staging infrastructure exposure; potential test accounts, debug endpoints, or pre-release features; staging without rate limiting = fallback enumeration vector if production is ever protected. Severity: low-medium.
testability: PASSIVE
[HYP] Desktop Windows key storage ACL bypass
class: MISCONFIG
asset: apps/desktop/src/common/node/key-storage/index.ts + apps/desktop/src/electron/electron-main.ts + apps/desktop/src/common/node/fs.ts
confidence: 75
reasoning: fileModeInternalObjectIfPosix() returns {} on Windows (no POSIX mode set), so keystorage.bin is written with default Windows ACLs (no restriction). _writeOrOverrideFile() in index.ts passes this empty object to fsPromises.writeFile(). Password file keystorage.password.bin is encrypted via electron.safeStorage.encryptString() (DPAPI on Windows) but is also written with fileModeInternalObjectIfPosix() → no ACL restriction. Any same-user process can read both files and decrypt the password via DPAPI, then use it to decrypt the Argon2id-encrypted keystorage.bin containing private keys and DB encryption key.
evidence_needed: Confirm on real Windows: (1) icacls on keystorage.bin and keystorage.password.bin after app init; (2) cross-process read of keystorage.password.bin succeeds and electron.safeStorage.decryptString() works from another same-user process.
verify_steps: HUMAN_ONLY: on Windows, launch Threema Desktop, enter/create a profile. From a separate process running as the same user: run icacls %APPDATA%\ThreemaDesktop\*\data\keystorage.password.bin, then CryptUnprotectData via Python win32crypt to recover the plaintext password, then attempt read of keystorage.bin.
impact: Local attacker with same-user process access extracts all private keys, database encryption key, device credentials. Severity: medium (requires local access, same-user privilege).
testability: HUMAN_ONLY
[PRIO] directory server IDOR (5 hosts: ds-apip/api/apip + .test), 8.8 — as:10 bv:9 tech:7 gate:10 cloud:5 fresh:10
[PRIO] staging directory server exposure (2 hosts: apip.test/ds-apip.test), 7.7 — as:8 bv:6 tech:8 gate:10 cloud:5 fresh:10
[PRIO] threema-desktop key storage Windows ACL, 7.5 — as:8 bv:8 tech:7 gate:8 cloud:3 fresh:9
[FINAL]
## 2026-08-07 21:20:29 UTC [desktop] (model laguna)
## 2026-08-07 22:03:25 UTC [desktop] (model laguna)
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
[HYP] Directory cluster identity→pubkey enumeration across 5 hosts
class: IDOR
asset: ds-apip.threema.ch + api.threema.ch + apip.threema.ch + ds-apip.test.threema.ch + apip.test.threema.ch (`GET /identity/{id}`, `POST /identity/fetch_bulk`)
confidence: 95
reasoning: All 5 directory hosts return 200 for `GET /identity/ECHOECHO` with identical pubkey data and permissive CORS (`*`, methods POST,GET,OPTIONS,DELETE). `fetch_bulk` silently omits invalid IDs (re-confirmed: only ECHOECHO returned from a 4-ID batch). Production hosts lack HSTS/Expect-CT; staging includes both but identical CORS exposure.
evidence_needed: Sustained burst >50 req/min with no 429/429/Retry-After; confirm invalid IDs are silently omitted rather than error-returned at scale.
verify_steps: PROBE: `for i in $(seq 1 15); do curl -s -o /dev/null -w '%{http_code} ' -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZ'${i}ZZ'ZZZZ","ZZZZZZZZ'${i}"]}'; sleep 1; done` (≤1 rps) — expect 15× 200, zero 429, only ECHOECHO in each body.
impact: Mass enumeration of all valid Threema identities → public keys at scale for targeted phishing/social-engineering and offline crypto attacks. Severity: medium.
testability: PASSIVE
[HYP] safe-01 backup service permissive write-capable CORS
class: MISCONFIG
asset: safe-01.threema.ch (backup service surface)
confidence: 80
reasoning: OPTIONS preflight returns `Access-Control-Allow-Origin: *` and `Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE`; HSTS + Expect-CT present; `GET /` → 404 (no routes mapped at root). Write verbs (PUT/PATCH/POST/DELETE) permitted cross-origin from any attacker origin with no observed auth challenge on root.
evidence_needed: An unauthenticated write endpoint (PUT/PATCH/POST/DELETE path) that accepts cross-origin requests and returns non-401/403, proving CSRF-class destruction via a malicious page.
verify_steps: PROBE: `curl -s -o /dev/null -w 'HTTP %{http_code}\n' -X OPTIONS -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: PUT" https://safe-01.threema.ch/` (confirmed 204 + write methods). Then enumerate plausible backup paths (`/backup`, `/api`, `/v1`, `/safe`, `/upload`) via HEAD at ≤1 rps to find an open write surface.
impact: If any write endpoint is open, a malicious page forces cross-origin creates/deletes on the backup service (CSRF via CORS). Severity: medium (pending path discovery).
testability: PASSIVE
[HYP] Desktop Windows key storage ACL bypass
class: MISCONFIG
asset: apps/desktop/src/common/node/key-storage/index.ts + electron-main.ts + common/node/fs.ts (`fileModeInternalObjectIfPosix()`)
confidence: 75
reasoning: RAG confirms `fileModeInternalObjectIfPosix()` returns `{}` on Windows, so `keystorage.bin` is written with default ACLs (no restriction) and `_writeOrOverrideFile` passes that empty object to `fsPromises.writeFile`. `keystorage.password.bin` is DPAPI-encrypted but likewise written with no ACL restriction; any same-user process can read both and decrypt via DPAPI then Argon2id.
evidence_needed: On real Windows: `icacls` on both files post-init; cross-process read of `keystorage.password.bin` succeeds and `safeStorage.decryptString()` resolves from another same-user process.
verify_steps: HUMAN_ONLY: launch Threema Desktop, create a profile. From a separate same-user process: `icacls %APPDATA%\ThreemaDesktop\*\data\keystorage.password.bin`; then CryptUnprotectData via Python `win32crypt` to recover plaintext password; then read `keystorage.bin`.
impact: Local same-user attacker extracts private keys, DB encryption key, device credentials. Severity: medium (requires local access, same-user).
testability: HUMAN_ONLY
[NEXT] PROBE: burst of 15 `POST https://ds-apip.threema.ch/identity/fetch_bulk` (~1 req every 2s, ≤1 rps) with a batch of 1 valid ID (ECHOECHO) + 2 distinct invalid IDs each iteration; confirm all 15 return HTTP 200 with zero 429/RateLimit/Retry-After and each body contains only ECHOECHO (silent-omit). This nails the rate-limit-absence + oracle core of the IDOR impact and is safe/passive (no account creation, no data modification).
