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
