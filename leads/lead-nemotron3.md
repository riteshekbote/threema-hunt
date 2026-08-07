# LEADS nemotron3 (seed)
- SEED: no model output yet; pipeline starts on first run.
## 2026-08-07 18:31:35 UTC [web] (model nemotron3)
[CHANGED] work.threema.ch: Now responds with 301 to /en/login (was 301, now confirmed PHP session cookie, CSP, Sentry reporting)
[CHANGED] shop.threema.ch: Now responds with 301 to /en (was 301, now confirmed CSP, Sentry, hCaptcha subdomain)
[CHANGED] broadcast.threema.ch: Now responds 301 to /en/login (was TIMEOUT, now accessible with session cookie, CSP, Sentry)
[CHANGED] gateway.threema.ch: Now responds 302 to /en (was TIMEOUT, now accessible with session cookie, CSP, Sentry)
[CHANGED] billing.threema.ch: Now responds 301 to threema.ch (was TIMEOUT, now redirects to main site)
[CHANGED] apip.threema.ch: Confirmed 403 with CORS headers allowing POST/GET/OPTIONS/DELETE (was 403, now detailed)
[NEW] api.threema.ch: Returns 403 with same CORS headers as apip.threema.ch (likely related ID service)
[NEW] safe.threema.ch: Timeout/no response (backup service pattern candidate)
[PRIO] apip.threema.ch, 7.8, attack=8 business=9 tech=7 gate=9 cloud=6 fresh=8
[PRIO] threema-desktop (source), 7.5, attack=8 business=8 tech=8 gate=10 cloud=5 fresh=7
[PRIO] work.threema.ch, 7.2, attack=7 business=8 tech=7 gate=6 cloud=7 fresh=8
[PRIO] broadcast.threema.ch, 7.0, attack=7 business=7 tech=7 gate=6 cloud=7 fresh=8
[PRIO] gateway.threema.ch, 6.8, attack=7 business=7 tech=7 gate=6 cloud=7 fresh=8
[PRIO] threema-android/iOS (source), 6.5, attack=7 business=9 tech=8 gate=10 cloud=4 fresh=5
[PRIO] shop.threema.ch, 6.3, attack=6 business=7 tech=6 gate=6 cloud=7 fresh=7
[PRIO] safe-*.threema.ch (unknown pattern), 5.5, attack=6 business=8 tech=5 gate=4 cloud=6 fresh=6
[PRIO] g-*.0.threema.ch / mediator-*.threema.ch / rendezvous-*.threema.ch (unknown patterns), 5.0, attack=6 business=8 tech=5 gate=3 cloud=5 fresh=5
[HYP] apip.threema.ch ID enumeration via CORS misconfiguration
class: AUTH
asset: apip.threema.ch
confidence: 70
reasoning: apip.threema.ch returns 403 with permissive CORS (Access-Control-Allow-Origin: *, allows POST/GET/OPTIONS/DELETE). This suggests API endpoints exist behind the 403. If any endpoint lacks auth checks or has weak authz, Threema ID enumeration could be possible via cross-origin requests from attacker-controlled pages.
evidence_needed: Discovery of API endpoint paths (e.g., /api/v1/lookup, /api/v1/pubkey) that accept unauthenticated requests and return user data.
verify_steps: PASSIVE: GET https://apip.threema.ch/api/v1/lookup?id=test (and common API paths) — observe response codes/body. PASSIVE: OPTIONS requests to discover allowed methods on suspected endpoints.
impact: Enumeration of valid Threema IDs → privacy violation, targeted attacks, spam. Severity: Medium.
testability: PASSIVE
[HYP] threema-desktop Electron preload/contextIsolation bypass for nodeIntegration
class: OTHER
asset: github.com/threema-ch/threema-desktop
confidence: 65
reasoning: Desktop 2.x is Electron-based (threema-web-electron repo exists). Electron apps historically vulnerable to contextIsolation/preload bypasses leading to RCE if nodeIntegration enabled or preload scripts expose dangerous APIs. Source code review needed for webPreferences config and preload script exposure.
evidence_needed: Source code showing webPreferences: { nodeIntegration: true } or contextIsolation: false, or preload scripts exposing require/process/eval.
verify_steps: PASSIVE: Static analysis of threema-desktop source (main.ts, preload.ts, package.json) for Electron security settings. Search for nodeIntegration, contextIsolation, sandbox, preload, enableRemoteModule.
impact: RCE on user's machine via malicious message/link handling. Severity: Critical.
testability: PASSIVE
[HYP] work.threema.ch/broadcast.threema.ch/gateway.threema.ch shared auth bypass via session fixation
class: AUTH
asset: work.threema.ch, broadcast.threema.ch, gateway.threema.ch
confidence: 55
reasoning: All three cockpit services set __HOST-HTTP-SESSIONID/SESSIONID cookies with HttpOnly; Secure; SameSite=Strict on initial 301 response before login. If session IDs are generated pre-auth and not rotated post-login, session fixation possible. Shared Sentry reporting (bugs.threema.ch) suggests shared infra.
evidence_needed: Confirm session ID remains same after successful login flow (requires AUTH_HELPED). Check if logout invalidates server-side session.
verify_steps: PASSIVE: Compare Set-Cookie headers on initial GET vs after simulated login (AUTH_HELPED: need valid creds). PASSIVE: Check cookie attributes for __HOST- prefix compliance.
impact: Account takeover on work/broadcast/gateway cockpits. Severity: High.
testability: AUTH_HELPED
[PARKED] work.threema.ch/broadcast.threema.ch/gateway.threema.ch shared auth bypass via session fixation: confidence 55 < 60 threshold for AUTH class; verify_steps require AUTH_HELPED (credentials) which violates passive-first for initial validation; no concrete passive evidence of fixation yet.
[FINAL] 
[NEXT] PROBE: GET https://apip.threema.ch/api/v1/pubkeys/<8-char-threema-id> — test if public key lookup endpoint exists and returns data without auth (common pattern in Threema ID directory services)
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration enabling cross-origin API probes confirmed via passive HEAD/GET.
[LEARN] ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
[RISK] chat: 65 reason: g-*.0.threema.ch pattern unknown; core messaging infra but no visible endpoints yet
[RISK] web: 75 reason: work/broadcast/gateway/shop now confirmed accessible with PHP sessions, CSP, Sentry; auth surfaces exposed
[RISK] sync: 55 reason: mediator-*/rendezvous-* patterns unknown; critical for multi-device but unenumerated
[RISK] safe: 50 reason: safe.threema.ch timeout; backup service high value but no access yet
[RISK] desktop-src: 70 reason: Electron app with large attack surface (RCE potential); source available for static analysis
## 2026-08-07 18:48:44 UTC [web] (model nemotron3)
[PRIO] apip.threema.ch, 7.8, attack=8 business=9 tech=7 gate=9 cloud=6 fresh=8
[PRIO] threema-desktop (source), 7.5, attack=8 business=8 tech=8 gate=10 cloud=5 fresh=7
[PRIO] work.threema.ch, 7.2, attack=7 business=8 tech=7 gate=6 cloud=7 fresh=8
[HYP] apip.threema.ch ID enumeration via CORS misconfiguration
class: AUTH
asset: apip.threema.ch
confidence: 70
reasoning: apip.threema.ch returns 403 with permissive CORS (Access-Control-Allow-Origin: *, allows POST/GET/OPTIONS/DELETE). API endpoints likely exist behind the 403. If any endpoint lacks auth checks, Threema ID enumeration could be possible via cross-origin requests.
evidence_needed: Discovery of API endpoint paths (e.g., /api/v1/lookup, /api/v1/pubkeys/<id>) that accept unauthenticated requests and return user data.
verify_steps: PASSIVE: GET https://apip.threema.ch/api/v1/pubkeys/<8-char-threema-id> — observe response codes/body. PASSIVE: OPTIONS requests to discover allowed methods on suspected endpoints.
impact: Enumeration of valid Threema IDs → privacy violation, targeted attacks, spam. Severity: Medium.
testability: PASSIVE
[HYP] threema-desktop Electron preload/contextIsolation bypass for nodeIntegration
class: OTHER
asset: github.com/threema-ch/threema-desktop
confidence: 65
reasoning: Desktop 2.x is Electron-based (threema-web-electron repo exists). Electron apps historically vulnerable to contextIsolation/preload bypasses leading to RCE if nodeIntegration enabled or preload scripts expose dangerous APIs.
evidence_needed: Source code showing webPreferences: { nodeIntegration: true } or contextIsolation: false, or preload scripts exposing require/process/eval.
verify_steps: PASSIVE: Static analysis of threema-desktop source (main.ts, preload.ts, package.json) for Electron security settings. Search for nodeIntegration, contextIsolation, sandbox, preload, enableRemoteModule.
impact: RCE on user's machine via malicious message/link handling. Severity: Critical.
testability: PASSIVE
[HYP] work.threema.ch session fixation via pre-auth session cookie
class: AUTH
asset: work.threema.ch
confidence: 55
reasoning: work.threema.ch sets __HOST-HTTP-SESSIONID cookie with HttpOnly; Secure; SameSite=Strict on initial 301 response before login. If session ID is generated pre-auth and not rotated post-login, session fixation possible. Shared Sentry reporting suggests shared infra with broadcast/gateway.
evidence_needed: Confirm session ID remains same after successful login flow (requires AUTH_HELPED). Check if logout invalidates server-side session.
verify_steps: PASSIVE: Compare Set-Cookie headers on initial GET vs after simulated login (AUTH_HELPED: need valid creds). PASSIVE: Check cookie attributes for __HOST- prefix compliance.
impact: Account takeover on work cockpit. Severity: High.
testability: AUTH_HELPED
[PARKED] work.threema.ch session fixation via pre-auth session cookie: confidence 55 < 60 threshold for AUTH class; verify_steps require AUTH_HELPED (credentials) which violates passive-first for initial validation; no concrete passive evidence of fixation yet.
[FINAL]
[NEXT] PROBE: GET https://apip.threema.ch/api/v1/pubkeys/ABCD1234 — test if public key lookup endpoint exists and returns data without auth (common pattern in Threema ID directory services)
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration enabling cross-origin API probes confirmed via passive HEAD/GET.
[LEARN] ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
[RISK] chat: 65 reason: g-*.0.threema.ch pattern unknown; core messaging infra but no visible endpoints yet
[RISK] web: 75 reason: work/broadcast/gateway/shop now confirmed accessible with PHP sessions, CSP, Sentry; auth surfaces exposed
[RISK] sync: 55 reason: mediator-*/rendezvous-* patterns unknown; critical for multi-device but unenumerated
[RISK] safe: 50 reason: safe.threema.ch timeout; backup service high value but no access yet
[RISK] desktop-src: 70 reason: Electron app with large attack surface (RCE potential); source available for static analysis
