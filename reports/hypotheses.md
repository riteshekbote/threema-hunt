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
