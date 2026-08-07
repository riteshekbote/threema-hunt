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
