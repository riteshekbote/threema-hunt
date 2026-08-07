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
