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
