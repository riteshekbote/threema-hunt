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
