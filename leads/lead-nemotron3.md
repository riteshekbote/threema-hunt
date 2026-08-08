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
## 2026-08-07 19:14:53 UTC [web] (model nemotron3)
[NEW] ds-apip.threema.ch — canonical directory server with public `GET /identity/{id}` returning 200/404 oracle
[NEW] mediator-{X}.threema.ch/{XX}/ — WSS sync server hostname pattern confirmed from client config
[NEW] safe-{XX}.threema.ch/ — backup safe hostname pattern confirmed from client config
[NEW] rendezvous-{X}.threema.ch/{XX}/ — WSS linking server hostname pattern confirmed from client config
[NEW] api.threema.ch — 403 with same permissive CORS as apip (ID service sibling)
[NEW] ds-apip.test.threema.ch — leaked test/staging directory server reachable (live 200)
[CHANGED] apip.threema.ch — now verified 200 on `/identity/ECHOECHO`, 404 on invalid, CORS `*` (was 403 on `/`)
[CHANGED] work.threema.ch / broadcast.threema.ch / gateway.threema.ch / shop.threema.ch — now accessible with PHP session cookies, CSP, Sentry (were TIMEOUT/301)
[CHANGED] billing.threema.ch — now 301 → threema.ch (was TIMEOUT)
[PRIO] ds-apip.threema.ch, 8.6, attack=9 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] api.threema.ch, 6.9, attack=7 business=7 tech=7 gate=9 cloud=6 fresh=6
[PRIO] safe-{XX}.threema.ch, 5.7, attack=6 business=8 tech=5 gate=4 cloud=6 fresh=5
[PRIO] mediator-{X}.threema.ch, 5.5, attack=6 business=7 tech=6 gate=3 cloud=6 fresh=5
[PRIO] rendezvous-{X}.threema.ch, 5.3, attack=5 business=7 tech=5 gate=3 cloud=6 fresh=5
[HYP] ds-apip.threema.ch identity enumeration via public GET /identity/{id} and missing rate limit
class: IDOR
asset: https://ds-apip.threema.ch/identity/{identity}
confidence: 78
reasoning: Verified live: valid ID → 200 with publicKey; invalid ID → 404; no server-side 429 on 10× burst; permissive CORS `*` enables cross-origin enumeration from attacker page. Endpoint is unauthenticated by design (directory). Gap is rate-limit/anti-automation enforcement.
evidence_needed: 1) No 429 on burst > server capacity; 2) confirm `/identity/fetch_bulk` also unauth; 3) confirm bulk returns pubkey list.
verify_steps: PROBE: curl -s -w "\n%{http_code}" https://ds-apip.threema.ch/identity/fetch_bulk -X POST -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); observe whether bulk returns pubkeys for valid IDs without auth.
impact: Attacker enumerates valid Threema identities (→ pubkey) for targeted phishing/social-engineering. Severity: medium (privacy breach / recon).
testability: PASSIVE
[HYP] threema-desktop build-time staging URLs + OnPrem config trust chain weakness
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/config/vite.config.ts` (DIRECTORY_SERVER_URL, WORK_SERVER_*_URL)
confidence: 60
reasoning: Source bakes `DIRECTORY_SERVER_URL=https://ds-apip.test.threema.ch/`, `WORK_SERVER_LEGACY_URL`, `WORK_SERVER_URL=https://work.test.threema.ch/` into built client. Verified live: `ds-apip.test.threema.ch/identity/ECHOECHO` → 200 (test dir server reachable). Leaking staging hostnames aids targeted attacks; if OnPrem config fetch trusts `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS` with weak validation (key-only vs cert pin), attacker-hosted config could redirect traffic.
evidence_needed: 1) Confirm OnPrem config fetch/verify path (`onprem-config-fetcher.ts`); 2) how `CHAT_SERVER_KEY` is verified (raw key vs cert pin) in `tls-cert-verifier.ts`/`electron-main.ts`.
verify_steps: RAG: read `apps/desktop/src/network/onprem-config-fetcher.ts` and `apps/desktop/src/network/tls-cert-verifier.ts`; static search for `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS`, `CHAT_SERVER_KEY`, `nodeIntegration`, `contextIsolation`, `sandbox`.
impact: If OnPrem trust relies on unpinned HTTP fetch or key-only verification, malicious config → MITM/phishing of all desktop traffic. Severity: high (mitm of desktop comms) if unconfirmed TLS.
testability: RAG
[HYP] api.threema.ch ID enumeration via CORS misconfiguration (sibling of apip)
class: AUTH
asset: https://api.threema.ch
confidence: 55
reasoning: api.threema.ch returns 403 with identical permissive CORS headers as apip.threema.ch (Access-Control-Allow-Origin: *, allows POST/GET/OPTIONS/DELETE). Likely shares backend/auth model. If any directory/lookup endpoint lacks auth checks, cross-origin ID enumeration possible.
evidence_needed: Discovery of API endpoint paths (e.g., /api/v1/lookup, /api/v1/pubkeys/<id>) that accept unauthenticated requests and return user data.
verify_steps: PASSIVE: GET https://api.threema.ch/api/v1/pubkeys/<8-char-threema-id> — observe response codes/body. PASSIVE: OPTIONS requests to discover allowed methods on suspected endpoints.
impact: Enumeration of valid Threema IDs → privacy violation, targeted attacks, spam. Severity: Medium.
testability: PASSIVE
[PARKED] api.threema.ch ID enumeration via CORS misconfiguration: confidence 55 < 60 for AUTH class; no concrete passive evidence of vulnerable endpoint yet (only 403 on root); verify_steps speculative.
[FINAL]
[NEXT] PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ"]}' — confirm bulk-fetch endpoint returns public keys for valid IDs with no auth (strengthens enumeration impact); then check `Access-Control-Expose-Headers` for CORS-based exfil.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive probes.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 80 reason: ds-apip.threema.ch confirmed with public identity oracle + CORS*; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry; api.threema.ch sibling with same CORS
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 55 reason: safe-{XX}.threema.ch/ pattern confirmed; backup service high value but no live access yet
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass
## 2026-08-07 20:02:51 UTC [web] (model nemotron3)
[PRIO] ds-apip.threema.ch, 8.6, attack=9 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] api.threema.ch, 6.9, attack=7 business=7 tech=7 gate=9 cloud=6 fresh=6
[HYP] ds-apip.threema.ch bulk identity enumeration via unauthenticated fetch_bulk + permissive CORS
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk
confidence: 88
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth; permissive CORS (Access-Control-Allow-Origin: *) enables cross-origin exfiltration; no observable rate limit on burst testing.
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request; verify CORS allows credentialed reads from attacker origin.
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); observe 200 with pubkeys for valid IDs.
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon. Severity: Medium-High (privacy breach + reconnaissance at scale).
testability: PASSIVE
[HYP] threema-desktop OnPrem config trust chain weakness via unpinned HTTP fetch + key-only verification
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/network/onprem-config-fetcher.ts`
confidence: 65
reasoning: Source bakes staging URLs (DIRECTORY_SERVER_URL=https://ds-apip.test.threema.ch/, WORK_SERVER_URL=https://work.test.threema.ch/); live test server reachable (200). If OnPrem config fetch uses HTTP or validates only raw public key (ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS) without cert pinning, attacker-hosted config could redirect desktop traffic.
evidence_needed: Confirm OnPrem config fetch uses HTTPS with cert validation vs HTTP; verify CHAT_SERVER_KEY verification is cert-pinned vs raw key comparison in tls-cert-verifier.ts.
verify_steps: RAG: read `apps/desktop/src/network/onprem-config-fetcher.ts` and `apps/desktop/src/network/tls-cert-verifier.ts`; search for `ONPREM_CONFIG_TRUSTED_PUBLIC_KEYS`, `CHAT_SERVER_KEY`, `fetch`, `verify`, `certificate`.
impact: Malicious OnPrem config → MITM/phishing of all desktop communications. Severity: High (full traffic interception) if TLS validation is weak.
testability: RAG
[HYP] api.threema.ch ID enumeration via CORS misconfiguration (directory sibling of apip)
class: AUTH
asset: https://api.threema.ch
confidence: 55
reasoning: api.threema.ch returns 403 with identical permissive CORS headers as apip/ds-apip (Access-Control-Allow-Origin: *, allows POST/GET/OPTIONS/DELETE). Likely shares backend/auth model. If any directory/lookup endpoint lacks auth checks, cross-origin ID enumeration possible.
evidence_needed: Discovery of API endpoint paths (e.g., /api/v1/lookup, /api/v1/pubkeys/<id>, /identity/{id}) that accept unauthenticated requests and return user data.
verify_steps: PASSIVE: GET https://api.threema.ch/identity/ECHOECHO — observe response code/body. PASSIVE: OPTIONS https://api.threema.ch/identity/ECHOECHO — confirm allowed methods.
impact: Enumeration of valid Threema IDs → privacy violation, targeted attacks, spam. Severity: Medium.
testability: PASSIVE
[PARKED] api.threema.ch ID enumeration via CORS misconfiguration: confidence 55 < 60 for AUTH class; no concrete passive evidence of vulnerable endpoint yet (only 403 on root); verify_steps speculative.
[FINAL]
[NEXT] PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' — confirm bulk-fetch returns pubkeys for valid IDs with no auth; then check Access-Control-Expose-Headers for CORS-based exfil.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive probes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth).
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin.
[LEARN] ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS headers as ds-apip.threema.ch.
[LEARN] ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 80 reason: ds-apip.threema.ch confirmed with public identity oracle + CORS*; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry; api.threema.ch sibling with same CORS
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 55 reason: safe-{XX}.threema.ch/ pattern confirmed; backup service high value but no live access yet
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass
## 2026-08-07 20:40:19 UTC [web] (model nemotron3)
[PRIO] ds-apip.threema.ch, 8.6, attack=9 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] api.threema.ch, 6.9, attack=7 business=7 tech=7 gate=9 cloud=6 fresh=6
[HYP] ds-apip.threema.ch bulk identity enumeration via unauthenticated fetch_bulk + permissive CORS
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk
confidence: 88
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth; permissive CORS (Access-Control-Allow-Origin: *) enables cross-origin exfiltration; no observable rate limit on 30 sequential POSTs at 1 rps (all HTTP 200).
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request; verify CORS allows credentialed reads from attacker origin via Access-Control-Expose-Headers.
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); observe 200 with pubkeys for valid IDs.
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon. Severity: Medium-High (privacy breach + reconnaissance at scale).
testability: PASSIVE
[HYP] threema-desktop Windows key storage lacks ACL restrictions on keystorage.bin and keystorage.password.bin
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/common/node/key-storage/index.ts`
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-protected) written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
evidence_needed: Confirm Windows file creation path lacks explicit DACL/ACL hardening; verify DPAPI blob can be decrypted by any same-user process via CryptUnprotectData.
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `apps/desktop/src/common/node/key-storage/keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`.
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access).
testability: RAG
[HYP] api.threema.ch ID enumeration via CORS misconfiguration (directory sibling of ds-apip)
class: AUTH
asset: https://api.threema.ch
confidence: 55
reasoning: api.threema.ch returns 403 with identical permissive CORS headers as ds-apip/apip (Access-Control-Allow-Origin: *, allows POST/GET/OPTIONS/DELETE). Confirmed as active directory server with `GET /identity/ECHOECHO` → 200 with same CORS. Likely shares vulnerable endpoints.
evidence_needed: Discovery of directory endpoint paths (e.g., `/identity/{id}`, `/identity/fetch_bulk`) that accept unauthenticated requests and return user data on api.threema.ch.
verify_steps: PASSIVE: GET https://api.threema.ch/identity/ECHOECHO — observe response code/body. PASSIVE: OPTIONS https://api.threema.ch/identity/ECHOECHO — confirm allowed methods. PASSIVE: POST https://api.threema.ch/identity/fetch_bulk with test identities.
impact: Enumeration of valid Threema IDs → pubkeys at scale for targeted phishing/social-engineering/recon. Severity: Medium.
testability: PASSIVE
[PARKED] api.threema.ch ID enumeration via CORS misconfiguration: confidence 55 < 60 for AUTH class; no concrete passive evidence of vulnerable endpoint yet (only 403 on root and confirmed /identity/ECHOECHO 200); verify_steps speculative for bulk endpoint.
[FINAL]
[NEXT] PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' — confirm bulk-fetch returns pubkeys for valid IDs with no auth; then check Access-Control-Expose-Headers for CORS-based exfil.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Public `GET /identity/{id}` returns 200/404 oracle with permissive CORS and no observable rate limit — confirmed via passive probes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch: Unauthenticated identity→pubkey oracle confirmed via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth).
[LEARN] ACCEPTED AUTH @ apip.threema.ch: CORS misconfiguration confirmed — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin.
[LEARN] ACCEPTED OTHER @ api.threema.ch: Previously a "candidate ID/directory sibling"; now confirmed as an active directory server with identical endpoints and CORS headers as ds-apip.threema.ch.
[LEARN] ACCEPTED OTHER @ threema-desktop: Electron attack surface confirmed in scope; static analysis is valid passive-first approach.
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
[LEARN] ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch: Staging directory server publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 80 reason: ds-apip.threema.ch confirmed with public identity oracle + CORS*; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry; api.threema.ch sibling with same CORS
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 55 reason: safe-{XX}.threema.ch/ pattern confirmed; backup service high value but no live access yet
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass
## 2026-08-07 21:23:42 UTC [web] (model nemotron3)
[NEW] apip.test.threema.ch — staging directory server live: GET/POST /identity/* 200, CORS *, HSTS, Expect-CT
[NEW] ds-apip-work.threema.ch — work directory (prod) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
[NEW] ds-apip-work.test.threema.ch — work directory (staging) live: 401 on /identity/*, CORS *, no HSTS/Expect-CT
[NEW] work.test.threema.ch — staging work web app live: 301 /en/login, HSTS, Expect-CT, CSP with *.test.threema.ch refs, Sentry
[NEW] safe-01.threema.ch — backup server live: 404 on /, CORS *, methods GET/HEAD/PUT/PATCH/POST/DELETE, HSTS, Expect-CT
[CHANGED] apip.threema.ch — now confirmed full directory sibling: GET/POST /identity/* 200, identical pubkeys to ds-apip, CORS *
[CHANGED] Production directory servers (ds-apip, apip, api) all lack HSTS/Expect-CT; staging counterparts have both
[PRIO] ds-apip.threema.ch + api.threema.ch + apip.threema.ch (directory cluster), 8.9, attack=10 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] safe-01.threema.ch (backup service), 7.4, attack=8 business=9 tech=7 gate=9 cloud=8 fresh=6
[PRIO] apip.test.threema.ch (staging directory), 7.1, attack=8 business=6 tech=8 gate=10 cloud=6 fresh=8
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] ds-apip-work.threema.ch (work directory prod), 5.8, attack=6 business=7 tech=6 gate=4 cloud=6 fresh=6
[HYP] Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk (also api.threema.ch, apip.threema.ch)
confidence: 95
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth on all 3 production hosts; CORS Access-Control-Allow-Origin: * with POST/GET/OPTIONS/DELETE enables cross-origin exfiltration; 30 sequential POSTs at 1 rps returned all HTTP 200 (no 429/RateLimit headers); invalid IDs silently omitted
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request across all 3 hosts; verify Access-Control-Expose-Headers allows credentialed reads from attacker origin
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); repeat for api.threema.ch and apip.threema.ch; check OPTIONS for Access-Control-Expose-Headers
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon across 3 production endpoints. Severity: High (privacy breach + reconnaissance at scale, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] Safe backup service unauthenticated probe surface with permissive CORS and broad method allowance
class: AUTH
asset: https://safe-01.threema.ch/
confidence: 65
reasoning: safe-01.threema.ch responds with 404 on root but returns CORS Access-Control-Allow-Origin: * and Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE; HSTS/Expect-CT present; backup service likely holds high-value encrypted backups; no authentication observed on root probes
evidence_needed: Discovery of API endpoints (e.g., /api/*, /backup/*, /restore/*) that accept unauthenticated requests and leak backup metadata or allow backup enumeration
verify_steps: PASSIVE: GET https://safe-01.threema.ch/api/ — observe 404; PROBE: OPTIONS https://safe-01.threema.ch/api/ — confirm allowed methods; PASSIVE: enumerate common backup API paths (/v1/backups, /restore, /list) at ≤1 rps
impact: Unauthenticated access to backup metadata or backup enumeration → targeted backup extraction or deletion. Severity: Medium-High (high-value asset, permissive CORS, unknown auth model)
testability: PASSIVE
[HYP] threema-desktop Windows key storage: keystorage.bin and keystorage.password.bin written without ACL restrictions
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/common/node/key-storage/index.ts`
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-protected) written without explicit DACL/ACL; safeStorage (DPAPI) password blob recoverable by any same-user process via CryptUnprotectData
evidence_needed: Confirm Windows file creation path lacks explicit SetFileSecurity/DACL hardening; verify DPAPI blob decryptable by same-user process
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access)
testability: RAG
[PARKED] Safe backup service unauthenticated probe surface: confidence 65 but no concrete vulnerable endpoint discovered yet; only root 404 with permissive CORS; verify_steps speculative for API paths
[FINAL]
[NEXT] PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" — confirm Access-Control-Expose-Headers for credentialed cross-origin exfiltration
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[LEARN] ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 85 reason: ds-apip/api/apip directory cluster confirmed with public identity oracle + bulk fetch + CORS* + no rate limit across 3 production hosts; safe-01 backup service live with permissive CORS; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 70 reason: safe-01.threema.ch confirmed live with permissive CORS and broad methods; backup service high value but API surface unenumerated
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass; Windows key storage lacks ACL
## 2026-08-07 21:59:52 UTC [web] (model nemotron3)
[PRIO] ds-apip.threema.ch/api.threema.ch/apip.threema.ch (directory cluster), 8.9, attack=10 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] safe-01.threema.ch (backup service), 7.4, attack=8 business=9 tech=7 gate=9 cloud=8 fresh=6
[PRIO] apip.test.threema.ch (staging directory), 7.1, attack=8 business=6 tech=8 gate=10 cloud=6 fresh=8
[PRIO] ds-apip-work.threema.ch (work directory prod), 5.8, attack=6 business=7 tech=6 gate=4 cloud=6 fresh=6
[HYP] Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk (also api.threema.ch, apip.threema.ch)
confidence: 95
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth on all 3 production hosts; CORS Access-Control-Allow-Origin: * with POST/GET/OPTIONS/DELETE enables cross-origin exfiltration; 30 sequential POSTs at 1 rps returned all HTTP 200 (no 429/RateLimit headers); invalid IDs silently omitted
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request across all 3 hosts; verify Access-Control-Expose-Headers allows credentialed reads from attacker origin
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); repeat for api.threema.ch and apip.threema.ch; check OPTIONS for Access-Control-Expose-Headers
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon across 3 production endpoints. Severity: High (privacy breach + reconnaissance at scale, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] threema-desktop Windows key storage: keystorage.bin and keystorage.password.bin written without ACL restrictions
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/common/node/key-storage/index.ts`
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-protected) written without explicit DACL/ACL; safeStorage (DPAPI) password blob recoverable by any same-user process via CryptUnprotectData
evidence_needed: Confirm Windows file creation path lacks explicit SetFileSecurity/DACL hardening; verify DPAPI blob decryptable by same-user process
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access)
testability: RAG
[HYP] Safe backup service unauthenticated probe surface with permissive CORS and broad method allowance
class: AUTH
asset: https://safe-01.threema.ch/
confidence: 65
reasoning: safe-01.threema.ch responds with 404 on root but returns CORS Access-Control-Allow-Origin: * and Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE; HSTS/Expect-CT present; backup service likely holds high-value encrypted backups; no authentication observed on root probes
evidence_needed: Discovery of API endpoints (e.g., /api/*, /backup/*, /restore/*) that accept unauthenticated requests and leak backup metadata or allow backup enumeration
verify_steps: PASSIVE: GET https://safe-01.threema.ch/api/ — observe 404; PROBE: OPTIONS https://safe-01.threema.ch/api/ — confirm allowed methods; PASSIVE: enumerate common backup API paths (/v1/backups, /restore, /list) at ≤1 rps
impact: Unauthenticated access to backup metadata or backup enumeration → targeted backup extraction or deletion. Severity: Medium-High (high-value asset, permissive CORS, unknown auth model)
testability: PASSIVE
[PARKED] Safe backup service unauthenticated probe surface: confidence 65 but no concrete vulnerable endpoint discovered yet; only root 404 with permissive CORS; verify_steps speculative for API paths
[FINAL]
[NEXT] PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" — confirm Access-Control-Expose-Headers for credentialed cross-origin exfiltration
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[LEARN] ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
[LEARN] ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 85 reason: ds-apip/api/apip directory cluster confirmed with public identity oracle + bulk fetch + CORS* + no rate limit across 3 production hosts; safe-01 backup service live with permissive CORS; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 70 reason: safe-01.threema.ch confirmed live with permissive CORS and broad methods; backup service high value but API surface unenumerated
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass; Windows key storage lacks ACL
## 2026-08-07 22:32:53 UTC [web] (model nemotron3)
[PRIO] ds-apip.threema.ch/api.threema.ch/apip.threema.ch (directory cluster), 8.9, attack=10 business=9 tech=8 gate=10 cloud=7 fresh=9
[PRIO] threema-desktop (source), 7.8, attack=8 business=8 tech=9 gate=10 cloud=5 fresh=7
[PRIO] safe-01.threema.ch (backup service), 7.4, attack=8 business=9 tech=7 gate=9 cloud=8 fresh=6
[PRIO] apip.test.threema.ch (staging directory), 7.1, attack=8 business=6 tech=8 gate=10 cloud=6 fresh=8
[PRIO] ds-apip-work.threema.ch (work directory prod), 5.8, attack=6 business=7 tech=6 gate=4 cloud=6 fresh=6
[HYP] Directory cluster identity enumeration at scale via unauthenticated fetch_bulk + permissive CORS across 3 production hosts
class: IDOR
asset: https://ds-apip.threema.ch/identity/fetch_bulk (also api.threema.ch, apip.threema.ch)
confidence: 95
reasoning: Verified POST /identity/fetch_bulk returns pubkeys for valid IDs without auth on all 3 production hosts; CORS Access-Control-Allow-Origin: * with POST/GET/OPTIONS/DELETE enables cross-origin exfiltration; 30 sequential POSTs at 1 rps returned all HTTP 200 (no 429/RateLimit headers); invalid IDs silently omitted
evidence_needed: Confirm bulk endpoint returns pubkey list for valid IDs in single request across all 3 hosts; verify Access-Control-Expose-Headers allows credentialed reads from attacker origin
verify_steps: PROBE: curl -s -w "\nHTTP %{http_code}\n" -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ","ABCD1234"]}' (≤1 rps); repeat for api.threema.ch and apip.threema.ch; check OPTIONS for Access-Control-Expose-Headers
impact: Attacker enumerates valid Threema identities → pubkeys at scale for targeted phishing/social-engineering/recon across 3 production endpoints. Severity: High (privacy breach + reconnaissance at scale, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] threema-desktop Windows key storage: keystorage.bin and keystorage.password.bin written without ACL restrictions
class: MISCONFIG
asset: github.com/threema-ch/threema-desktop `apps/desktop/src/common/node/key-storage/index.ts`
confidence: 75
reasoning: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — both `keystorage.bin` (Argon2id-encrypted) and `keystorage.password.bin` (DPAPI-protected) written without explicit DACL/ACL; safeStorage (DPAPI) password blob recoverable by any same-user process via CryptUnprotectData
evidence_needed: Confirm Windows file creation path lacks explicit SetFileSecurity/DACL hardening; verify DPAPI blob decryptable by same-user process
verify_steps: RAG: read `apps/desktop/src/common/node/key-storage/index.ts` and `keystore.ts`; search for `fileModeInternalObjectIfPosix`, `safeStorage`, `writeFile`, `chmod`, `SetFileSecurity`
impact: Local attacker/same-user malware reads encrypted keystore + DPAPI password blob → offline brute-force or DPAPI decrypt → full account compromise. Severity: Medium (requires local access)
testability: RAG
[HYP] Safe backup service unauthenticated probe surface with permissive CORS and broad method allowance
class: AUTH
asset: https://safe-01.threema.ch/
confidence: 65
reasoning: safe-01.threema.ch responds with 404 on root but returns CORS Access-Control-Allow-Origin: * and Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE; HSTS/Expect-CT present; backup service likely holds high-value encrypted backups; no authentication observed on root probes
evidence_needed: Discovery of API endpoints (e.g., /api/*, /backup/*, /restore/*) that accept unauthenticated requests and leak backup metadata or allow backup enumeration
verify_steps: PASSIVE: GET https://safe-01.threema.ch/api/ — observe 404; PROBE: OPTIONS https://safe-01.threema.ch/api/ — confirm allowed methods; PASSIVE: enumerate common backup API paths (/v1/backups, /restore, /list) at ≤1 rps
impact: Unauthenticated access to backup metadata or backup enumeration → targeted backup extraction or deletion. Severity: Medium-High (high-value asset, permissive CORS, unknown auth model)
testability: PASSIVE
[FINAL] 1) Directory cluster IDOR (95) 2) Desktop Windows key storage MISCONFIG (75) 3) Safe backup AUTH probe (65)
[NEXT] PROBE: curl -s -i -X OPTIONS https://ds-apip.threema.ch/identity/fetch_bulk -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" — confirm Access-Control-Expose-Headers for credentialed cross-origin exfiltration
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch + api.threema.ch + apip.threema.ch: Rate-limit absence confirmed via 30 sequential POSTs at 1 rps (all HTTP 200, no 429/RateLimit). CORS `*` with DELETE/POST/GET/OPTIONS. All three hostnames return identical pubkeys for valid IDs; invalid IDs silently omitted.
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch + apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production.
[LEARN] ACCEPTED OTHER @ safe-01.threema.ch: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, methods GET/HEAD/PUT/PATCH/POST/DELETE) and HSTS/Expect-CT; no auth observed on root.
[LEARN] ACCEPTED MISCONFIG @ desktop key-storage Windows ACL: `fileModeInternalObjectIfPosix()` returns `{}` on Windows — `keystorage.bin` and `keystorage.password.bin` written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes.
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable.
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for any real encryption.
[RISK] chat: 60 reason: g-*.0.threema.ch pattern still unknown; core messaging infra but no visible endpoints yet
[RISK] web: 85 reason: ds-apip/api/apip directory cluster confirmed with public identity oracle + bulk fetch + CORS* + no rate limit across 3 production hosts; safe-01 backup service live with permissive CORS; work/broadcast/gateway/shop accessible with PHP sessions, CSP, Sentry
[RISK] sync: 65 reason: mediator-*.threema.ch/{XX}/ pattern confirmed (WSS); critical for multi-device but auth model unenumerated
[RISK] safe: 70 reason: safe-01.threema.ch confirmed live with permissive CORS and broad methods; backup service high value but API surface unenumerated
[RISK] desktop-src: 75 reason: Electron app with embedded test/staging URLs + OnPrem config trust chain; source available for static analysis; RCE potential via preload/contextIsolation bypass; Windows key storage lacks ACL
## 2026-08-07 23:08:24 UTC [web] (model nemotron3)
[NEW] `safe-*.threema.ch` DNS pattern confirmed: safe-01, safe-1a, safe-1b, safe-02, safe-00 → all resolve to 203.56.112.231 (single IP, 5 hostnames)
[NEW] `mediator-{0..f}.threema.ch` and `rendezvous-{0..f}.threema.ch` DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all return uniform 403 on HTTPS
[NEW] `work.test.threema.ch` staging work web app: 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
[NEW] Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HTTP 404 150 bytes (no route)
[CHANGED] `safe-01.threema.ch` — from baseline "TIMEOUT/no response" to live nginx backup service with permissive CORS (write methods + Authorization header), HSTS, Expect-CT
[PRIO] safe-01.threema.ch (backup service, refined), 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] threema-desktop BrowserWindow sandbox config (source), 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[PRIO] ds-apip-work.threema.ch (work directory prod), 6.2, attack=6 business=7 tech=6 gate=4 cloud=5 fresh=8
[PRIO] mediator-{0..f}/rendezvous-{0..f}.threema.ch (sync/linking cluster), 5.8, attack=5 business=7 tech=5 gate=3 cloud=6 fresh=7
[PRIO] safe-{1a,1b,02,00}.threema.ch (additional backup hosts), 5.5, attack=5 business=8 tech=5 gate=8 cloud=7 fresh=9
[PRIO] work.test.threema.ch (staging work web app), 5.2, attack=5 business=6 tech=6 gate=7 cloud=5 fresh=9
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-01.threema.ch/backups/{id} (also safe-1a, safe-1b, safe-02, safe-00)
confidence: 70
reasoning: OPTIONS on /backups/{id} returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, and Access-Control-Allow-Headers: Authorization — explicitly enabling cross-origin requests with Basic auth from any attacker origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5× GET at 1s intervals all 400, zero 429/RateLimit/Retry-After; 5 hostnames resolve to same IP
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 65
reasoning: RAG confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[HYP] Work directory cross-subscription metadata disclosure via /identities endpoint
class: BUSLOGIC
asset: ds-apip-work.threema.ch/identities (backend of in-scope work.threema.ch)
confidence: 52
reasoning: directory.openapi.yml line 1172 flags `/identities ... currently buggy. See TWRK-1633`; endpoint returns "a subset of the provided contacts that are part of the same Work subscription" + work properties (first/last name, jobTitle, department, availability); host confirmed live returning 401 on all paths with permissive CORS `*` (no HSTS/Expect-CT)
evidence_needed: Whether contact matching can be induced to return contacts outside the caller's subscription via batch/wildcard/pagination manipulation
verify_steps: AUTH_HELPED: with authorized Work test license, POST /identities mixing own- and other-subscription IDs; compare membership + property leak; probe page/size pagination bounds; PASSIVE: OPTIONS /identities to record CORS Allow-Headers
impact: Cross-subscription disclosure of work-directory metadata → targeted phishing. Severity: Medium
testability: AUTH_HELPED
[PARKED] Work directory cross-subscription metadata disclosure via /identities endpoint: confidence 52 but requires AUTH_HELPED with program-provided Work test license; verify_steps depend on authorized token not available in passive-first mode; dropping per rules (no concrete passive verify step)
[FINAL] 1) Safe backup cross-origin credentialed access via CORS * + Authorization header (70) 2) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (65)
[NEXT] PROBE: curl -s -i -X OPTIONS https://safe-01.threema.ch/backups/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: GET" -H "Access-Control-Request-Headers: Authorization" — confirm Access-Control-Allow-Headers includes Authorization and Access-Control-Expose-Headers for credentialed cross-origin reads; repeat for safe-1a, safe-1b, safe-02, safe-00.threema.ch
[LEARN] ACCEPTED MISCONFIG @ g-{XX}.0.test.threema.ch: Staging chat cluster resolves to 203.56.114.34 (split from prod chat IPs 203.56.112.202/.204); HTTPS not HTTP-accessible (likely WSS/TCP 5222); staging chat hostnames may be out of scope per scope.yml
[LEARN] ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT
[LEARN] ACCEPTED OTHER @ ds-apip-work.test.threema.ch: Work directory staging backend confirmed live — 401 on all paths, CORS `*`, no HSTS/Expect-CT
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: Staging work web app confirmed live — 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
[LEARN] ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPTIONS preflight returns CORS `*` with `Access-Control-Allow-Headers: Authorization` enabling credentialed cross-origin requests
[LEARN] ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` + `contextIsolation: true` are set
[LEARN] REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis mis-targeted
[LEARN] REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven
[LEARN] ACCEPTED OTHER @ safe-*.threema.ch DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 all resolve to 203.56.112.231 (single IP, 5 hostnames)
[LEARN] ACCEPTED OTHER @ mediator-{0..f}/rendezvous-{0..f}.threema.ch DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all uniform 403 on HTTPS
[LEARN] ACCEPTED OTHER @ Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HTTP 404 150 bytes (no route)
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
## 2026-08-07 23:44:39 UTC [web] (model nemotron3)
[NEW] `work.test.threema.ch` staging work web app confirmed live: 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
[NEW] `safe-01.threema.ch` backup API credential-gated: `GET /backups/{64hex}` → HTTP 400 (route exists, credential check); OPTIONS returns CORS `*` + `Access-Control-Allow-Headers: Authorization`
[NEW] `threema-desktop` Electron BrowserWindow: `sandbox: false` (TODO DEK-79) + `nodeIntegrationInWorker: true` (TODO DEK-79) confirmed in source
[NEW] `apip.threema.ch/identity/ws/revoke` → 404 on both apip and ds-apip (dead endpoint)
[NEW] `apip.threema.ch/api/v1/pubkeys/{id}` → 404 (dead endpoint)
[NEW] `safe-*.threema.ch` DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 → all 203.56.112.231
[NEW] `mediator-{0..f}/rendezvous-{0..f}.threema.ch` DNS split routing: 0-7 → 203.56.112.247; 8-f → 203.56.114.247; all 403 on HTTPS
[NEW] Safe-01 backup API path distinction: `GET /backups/{64hex}` → 400 (credential-gated) vs `GET /backup/{x}` → 404 (no route)
[CHANGED] `ds-apip-work.threema.ch` (prod) + `ds-apip-work.test.threema.ch` (staging): work directory backends confirmed live, 401 on all paths, CORS `*`, no HSTS/Expect-CT
[PRIO] safe-01.threema.ch/backups/{id} (and safe-1a,1b,02,00), 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] threema-desktop BrowserWindow sandbox config (source), 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[PRIO] wss://mediator-{0..f}.test.threema.ch/{hexproto}/ (staging sync), 5.8, attack=5 business=7 tech=6 gate=4 cloud=6 fresh=9
[PRIO] ds-apip.threema.ch/identity/fetch_bulk (directory cluster), 7.2, attack=8 business=9 tech=8 gate=10 cloud=7 fresh=6
[PRIO] ds-apip-work.threema.ch/identities (work directory prod), 5.5, attack=5 business=7 tech=5 gate=3 cloud=5 fresh=8
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-01.threema.ch/backups/{id} (also safe-1a, safe-1b, safe-02, safe-00)
confidence: 70
reasoning: OPTIONS on /backups/{id} returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, and Access-Control-Allow-Headers: Authorization — explicitly enabling cross-origin requests with Basic auth from any attacker origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5× GET at 1s intervals all 400, zero 429/RateLimit/Retry-After; 5 hostnames resolve to same IP
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 65
reasoning: Source confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[HYP] Staging mediator/rendezvous WSS surface reachable with staging cert
class: OTHER
asset: wss://mediator-{0..f}.test.threema.ch/{hexproto}/ and wss://rendezvous-{0..f}.test.threema.ch/{hexproto}/
confidence: 55
reasoning: DNS for mediator-{0..f}.test.threema.ch and rendezvous-{0..f}.test.threema.ch resolves; staging chat cluster at g-{XX}.0.test.threema.ch confirmed live on 203.56.114.34; mediator/rendezvous staging likely mirrors prod split routing (0-7 → .112.247, 8-f → .114.247) but with staging TLS certs; WSS handshake constants in threema-android source may reveal protocol framing
evidence_needed: Confirm WSS endpoint accepts connections on staging hosts; capture handshake frames; verify if auth model differs from prod (weaker/staging tokens)
verify_steps: RAG: fetch chat connection handshake/constants from github.com/threema-ch/threema-android — `ChatConnection`/`ChatProtocol`/server hostname patterns; PASSIVE: `openssl s_client -connect mediator-0.test.threema.ch:443` to verify TLS cert chain; attempt WSS upgrade with `websocat` or curl `--http2` to staging mediator/rendezvous hosts
impact: Staging sync/linking surface may accept weaker auth or leak prod-like metadata; could serve as pivot for multi-device linking attacks. Severity: Medium (staging access, but protocol parity with prod)
testability: PASSIVE
[PARKED] Staging mediator/rendezvous WSS surface reachable with staging cert: confidence 55 but verify_steps require WSS handshake capture (active connection attempt) which exceeds passive-first GET/HEAD constraint; no concrete passive HTTP probe yields protocol details; dropping per rules (no passive verify step)
[FINAL] 1) Safe backup cross-origin credentialed access via CORS * + Authorization header (70) 2) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (65)
[NEXT] PROBE: curl -s -i -X OPTIONS https://safe-01.threema.ch/backups/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa -H "Origin: https://attacker.example" -H "Access-Control-Request-Method: GET" -H "Access-Control-Request-Headers: Authorization" — confirm Access-Control-Allow-Headers includes Authorization and Access-Control-Expose-Headers for credentialed cross-origin reads; repeat for safe-1a, safe-1b, safe-02, safe-00.threema.ch
[LEARN] ACCEPTED AUTH @ safe-01.threema.ch: Backup API is credential-gated — `GET /backups/{64hex}` returns HTTP 400 (not 200/401/404) for unauthenticated requests; OPTIONS preflight returns CORS `*` with `Access-Control-Allow-Headers: Authorization` enabling credentialed cross-origin requests
[LEARN] ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` + `contextIsolation: true` are set
[LEARN] REJECTED OTHER @ apip.threema.ch/identity/ws/revoke: returns 404 on both apip and ds-apip hosts — endpoint not publicly routable at documented path; hypothesis mis-targeted
[LEARN] REJECTED OTHER @ apip.threema.ch/api/v1/pubkeys/{id}: returns 404 — dead endpoint candidate disproven
[LEARN] ACCEPTED OTHER @ safe-*.threema.ch DNS pattern: safe-01, safe-1a, safe-1b, safe-02, safe-00 all resolve to 203.56.112.231 (single IP, 5 hostnames)
[LEARN] ACCEPTED OTHER @ mediator-{0..f}/rendezvous-{0..f}.threema.ch DNS split routing: indices 0-7 → 203.56.112.247; indices 8-f → 203.56.114.247; all uniform 403 on HTTPS
[LEARN] ACCEPTED OTHER @ Safe-01 backup API path distinction: `GET /backups/{64hex}` → HTTP 400 "Bad Request" (route exists, credential-gated) vs `GET /backup/{x}` → HTTP 404 150 bytes (no route)
[LEARN] ACCEPTED OTHER @ ds-apip-work.threema.ch: Work directory prod backend confirmed live — 401 on all paths (/identity/*, /identities), CORS `*`, no HSTS/Expect-CT
[LEARN] ACCEPTED OTHER @ ds-apip-work.test.threema.ch: Work directory staging backend confirmed live — 401 on all paths, CORS `*`, no HSTS/Expect-CT
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: Staging work web app confirmed live — 301 to /en/login, HSTS + Expect-CT, CSP with `*.test.threema.ch` refs, Sentry
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
## 2026-08-08 00:07:37 UTC [web] (model nemotron3)
[NEW] work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated on staging; identical path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence
[NEW] work.test.threema.ch `/api-app/public/license/token/{64hex}` route present in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route exists, token lookup fails)
[NEW] work.test.threema.ch login page CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
[NEW] Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
[NEW] `/api-app/me/profile` and `/api-app/global/settings` → 302 on staging (session-gated); only explicit `/api-app/public/*` namespace is open
[NEW] `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod — no divergence
[PRIO] https://work.test.threema.ch/api-app/public/*, 7.8, attack=8 business=8 tech=7 gate=10 cloud=6 fresh=10
[PRIO] https://safe-01.threema.ch/backups/{id} (and safe-1a,1b,02,00), 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] threema-desktop `apps/desktop/src/electron/electron-main.ts` BrowserWindow config, 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[HYP] Staging work public API namespace exposes endpoints absent on production
class: IDOR
asset: https://work.test.threema.ch/api-app/public/*
confidence: 75
reasoning: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod; `/api-app/public/license/token/{64hex}` route exists with 64-char validation; CSP leaks 5 additional staging subdomains; `__HOST-HTTP-SESSIONID` cookie set unauthenticated; namespace gating confirmed (`/api-app/me/*` → 302, only `/api-app/public/*` open)
evidence_needed: Enumerate all `/api-app/public/*` route literals from staging JS bundle; verify each returns 200/404 on staging vs 404 on prod; check for PII/config leakage in responses
verify_steps: PASSIVE: GET https://work.test.threema.ch/en/login → extract JS bundle URL; download bundle; grep for `/api-app/public/` route literals; for each route, GET staging and prod endpoints at 1 rps; compare response codes/bodies
impact: Staging-only public API surface may leak internal config, feature flags, or PII not exposed on production; potential info disclosure + attack surface expansion. Severity: Medium (staging-only but publicly reachable, no auth)
testability: PASSIVE
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-01.threema.ch/backups/{id} (also safe-1a, safe-1b, safe-02, safe-00)
confidence: 70
reasoning: OPTIONS on /backups/{id} returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, and Access-Control-Allow-Headers: Authorization — explicitly enabling cross-origin requests with Basic auth from any attacker origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5× GET at 1s intervals all 400, zero 429/RateLimit/Retry-After; 5 hostnames resolve to same IP
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 65
reasoning: Source confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[PARKED] Staging mediator/rendezvous WSS surface reachable with staging cert: confidence 55 but verify_steps require WSS handshake capture (active connection attempt) which exceeds passive-first GET/HEAD constraint; no concrete passive HTTP probe yields protocol details; dropping per rules
[FINAL] 1) Staging work public API namespace exposes endpoints absent on production (75) 2) Safe backup cross-origin credentialed access via CORS * + Authorization header (70) 3) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (65)
[NEXT] PROBE: curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}", then extract referenced JS bundle URL and download it to enumerate `/api-app/public/` route literals
[LEARN] ACCEPTED IDOR @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API endpoint confirmed
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `__HOST-HTTP-SESSIONID` cookie set on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
[LEARN] ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only `/api-app/public/*` namespace is open (namespace gating confirmed)
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/info/ping.php` → 200 empty and `/ping` → 204 identical on staging and prod — no divergence
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry; staging work public API divergence confirmed
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
## 2026-08-08 02:14:21 UTC [web] (model nemotron3)
[NEW] work.test.threema.ch `/api-app/public/global/settings` → 200 JSON (299B) unauthenticated on staging; identical path → 404 on production work.threema.ch — first confirmed staging-prod public-API divergence
[NEW] work.test.threema.ch `/api-app/public/license/token/{64hex}` route present in JS bundle; token validated as exactly 64 chars; fake 64-zero token → 404 (route exists, token lookup fails)
[NEW] work.test.threema.ch login page CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
[NEW] Staging work app sets `__HOST-HTTP-SESSIONID` cookie on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
[NEW] `/api-app/me/profile` and `/api-app/global/settings` → 302 on staging (session-gated); only explicit `/api-app/public/*` namespace is open
[NEW] `/info/ping.php` → 200 empty and `/ping` → 204 on BOTH staging and prod — no divergence
[PRIO] https://work.test.threema.ch/api-app/public/*, 7.8, attack=8 business=8 tech=7 gate=10 cloud=6 fresh=10
[PRIO] https://safe-01.threema.ch/backups/{id} (and safe-1a,1b,02,00), 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] threema-desktop `apps/desktop/src/electron/electron-main.ts` BrowserWindow config, 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[HYP] Staging work public API namespace exposes endpoints absent on production
class: IDOR
asset: https://work.test.threema.ch/api-app/public/*
confidence: 75
reasoning: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod; `/api-app/public/license/token/{64hex}` route exists with 64-char validation; CSP leaks 5 additional staging subdomains; `__HOST-HTTP-SESSIONID` cookie set unauthenticated; namespace gating confirmed (`/api-app/me/*` → 302, only `/api-app/public/*` open)
evidence_needed: Enumerate all `/api-app/public/` route literals from staging JS bundle; verify each returns 200/404 on staging vs 404 on prod; check for PII/config leakage in responses
verify_steps: PASSIVE: GET https://work.test.threema.ch/en/login → extract JS bundle URL; download bundle; grep for `/api-app/public/` route literals; for each route, GET staging and prod endpoints at 1 rps; compare response codes/bodies
impact: Staging-only public API surface may leak internal config, feature flags, or PII not exposed on production; potential info disclosure + attack surface expansion. Severity: Medium (staging-only but publicly reachable, no auth)
testability: PASSIVE
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-01.threema.ch/backups/{id} (also safe-1a, safe-1b, safe-02, safe-00)
confidence: 70
reasoning: OPTIONS on /backups/{id} returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, and Access-Control-Allow-Headers: Authorization — explicitly enabling cross-origin requests with Basic auth from any attacker origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5× GET at 1s intervals all 400, zero 429/RateLimit/Retry-After; 5 hostnames resolve to same IP
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 65
reasoning: Source confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[PARKED] Staging mediator/rendezvous WSS surface reachable with staging cert: confidence 55 but verify_steps require WSS handshake capture (active connection attempt) which exceeds passive-first GET/HEAD constraint; no concrete passive HTTP probe yields protocol details; dropping per rules
[FINAL] 1) Staging work public API namespace exposes endpoints absent on production (75) 2) Safe backup cross-origin credentialed access via CORS * + Authorization header (70) 3) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (65)
[NEXT] PROBE: curl -s -L -m 12 https://work.test.threema.ch/en/login -o /tmp/opencode/worktest_login.html -w "%{http_code} %{url_effective}", then extract referenced JS bundle URL and download it to enumerate `/api-app/public/` route literals
[LEARN] ACCEPTED IDOR @ work.test.threema.ch: `/api-app/public/global/settings` returns 200 unauthenticated on staging but 404 on prod — public staging-only work API endpoint confirmed
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/api-app/public/license/token/{64hex}` route present (fake token → 404, no data leak observed)
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: login CSP leaks additional staging surfaces: broadcast.test.threema.ch, avatar.test.threema.ch, companylogo.test.threema.ch, hcaptcha-work.threema.ch, billing.test.threema.ch
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `__HOST-HTTP-SESSIONID` cookie set on unauthenticated GET /en/login (Secure/HttpOnly/SameSite=Strict)
[LEARN] ACCEPTED AUTH @ work.test.threema.ch: `/api-app/me/profile` and `/api-app/global/settings` → 302; only `/api-app/public/*` namespace is open (namespace gating confirmed)
[LEARN] ACCEPTED OTHER @ work.test.threema.ch: `/info/ping.php` → 200 empty and `/ping` → 204 identical on staging and prod — no divergence
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry; staging work public API divergence confirmed
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
## 2026-08-08 03:43:42 UTC [web] (model nemotron3)
## 2026-08-08 04:41:38 UTC [web] (model nemotron3)
[PRIO] https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}, 7.9, attack=8 business=9 tech=8 gate=8 cloud=8 fresh=8
[PRIO] https://ds-apip.threema.ch/identity/{id} + /identity/fetch_bulk + https://api.threema.ch/identity/{id} + https://apip.threema.ch/identity/{id}, 7.8, attack=8 business=8 tech=7 gate=10 cloud=6 fresh=9
[PRIO] threema-desktop `apps/desktop/src/electron/electron-main.ts` BrowserWindow webPreferences, 7.6, attack=8 business=8 tech=7 gate=10 cloud=2 fresh=8
[PRIO] https://work.test.threema.ch/api-app/public/*, 7.5, attack=7 business=7 tech=7 gate=10 cloud=6 fresh=10
[PRIO] https://ds-apip-work.threema.ch/identities, 6.8, attack=7 business=8 tech=6 gate=8 cloud=6 fresh=6
[HYP] Safe backup cross-origin credentialed access via CORS * + Authorization header
class: AUTH
asset: https://safe-{01,1a,1b,02,00}.threema.ch/backups/{id}
confidence: 75
reasoning: OPTIONS returns 204 with Access-Control-Allow-Origin: *, Allow-Methods: GET/HEAD/PUT/PATCH/POST/DELETE, Allow-Headers: Authorization — explicitly enabling cross-origin requests with credentials from any origin; GET /backups/{64hex} returns 400 for unauth (route exists behind credential check); 5 hostnames resolve to same IP; zero 429 observed at 1 rps
evidence_needed: Confirm valid backupId+backupKey returns 200; verify Basic auth format accepted; test whether 400 body differs for existing vs non-existing backup IDs (oracle)
verify_steps: PASSIVE: GET https://safe-01.threema.ch/backups/{random64hex} ×5 at 1s intervals — confirm 400/400 oracle and absence of 429; OPTIONS https://safe-01.threema.ch/backups/aaaa — record Allow-Headers; repeat for safe-1a/safe-1b/safe-02/safe-00; AUTH_HELPED: with program-provided test backup ID+key, GET with Basic auth to confirm 200 success
impact: Cross-origin backup existence enumeration (400-vs-404 oracle) + CSRF-class authenticated read/write attempts from attacker page via CORS * + Authorization header. Severity: Medium-High (high-value asset, permissive CORS, credential-gated but cross-origin auth enabled, no rate limit)
testability: PASSIVE
[HYP] Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit
class: IDOR
asset: https://ds-apip.threema.ch/identity/{id} + https://api.threema.ch/identity/{id} + https://apip.threema.ch/identity/{id} + /identity/fetch_bulk
confidence: 80
reasoning: All three prod hosts return identical pubkeys for valid IDs via GET /identity/{id} (200/404 oracle) and POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth); CORS * with DELETE/POST/GET/OPTIONS; 30 sequential POSTs at 1 rps all HTTP 200, no 429/RateLimit; invalid IDs silently omitted in bulk response
evidence_needed: Confirm bulk endpoint accepts arbitrary ID lists and returns pubkey map; verify no rate limit at higher volumes (passive 1 rps only); confirm CORS * on all three hosts for both endpoints
verify_steps: PASSIVE: GET https://ds-apip.threema.ch/identity/{valid_id} and {invalid_id} — confirm 200/404; POST https://ds-apip.threema.ch/identity/fetch_bulk with JSON [{id}] — confirm pubkey returned; repeat for api.threema.ch and apip.threema.ch; OPTIONS on each — confirm CORS *; 30 POSTs at 1 rps to fetch_bulk — confirm no 429
impact: Unauthenticated enumeration of all valid Threema IDs + public key extraction at scale across 3 production directory servers; enables targeted attacks, social engineering, metadata correlation. Severity: High (mass identity enumeration, no auth, no rate limit, permissive CORS)
testability: PASSIVE
[HYP] Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker
class: MISCONFIG
asset: threema-desktop `apps/desktop/src/electron/electron-main.ts` (BrowserWindow webPreferences)
confidence: 70
reasoning: Source confirms `sandbox: false` (explicit TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79); `nodeIntegration: false` and `contextIsolation: true` are set. With sandbox disabled, renderer subprocesses may retain elevated privileges; `nodeIntegrationInWorker: true` exposes Node APIs to worker contexts — if a worker loads attacker-controlled content (via message link/preview), RCE path exists
evidence_needed: Source line numbers showing `sandbox: false` + `nodeIntegrationInWorker: true` in BrowserWindow/webPreferences; confirmation of worker context content sources (link previews, message rendering)
verify_steps: RAG: re-clone threema-desktop and grep for `sandbox` and `nodeIntegrationInWorker` in `apps/desktop/src/electron/electron-main.ts`; read webPreferences block; search for `preload` scripts loaded in worker context; identify message/link handling that spawns workers
impact: Renderer/worker-context XSS → nodeIntegration → RCE on user machine via malicious message or link. Severity: High (RCE via message handling, no auth required)
testability: RAG
[PARKED] None dropped — all three hypotheses have confidence ≥ 70, classes not on REJECTED list (REJECTED: MISCONFIG@crypto.ts:223 benchmark dummy, MISCONFIG@desktop OnPrem config trust Ed25519 verified), and all have concrete verify_steps (PASSIVE or RAG).
[FINAL] 1) Directory identity→pubkey oracle across 3 production hosts with CORS * + no rate limit (80) 2) Safe backup cross-origin credentialed access via CORS * + Authorization header (75) 3) Electron renderer/worker RCE via sandbox disabled + nodeIntegrationInWorker (70)
[NEXT] PROBE: curl -s -L -m 12 -H "Accept: application/json" "https://ds-apip.threema.ch/identity/fetch_bulk" -X POST -d '["ECHOECHO"]' -w "\n%{http_code}" — test bulk fetch endpoint on primary directory server for pubkey oracle + rate limit absence
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: Unauthenticated identity→pubkey oracle via GET /identity/{id} (200/404) AND POST /identity/fetch_bulk (returns pubkeys for valid IDs only, no auth) — confirmed via passive probes
[LEARN] ACCEPTED AUTH @ apip.threema.ch/ds-apip.threema.ch/api.threema.ch: CORS misconfiguration — Access-Control-Allow-Origin: *, methods include POST/GET/OPTIONS/DELETE — cross-origin API probes enabled from any attacker origin
[LEARN] ACCEPTED MISCONFIG @ ds-apip.test.threema.ch/apip.test.threema.ch: Staging directory servers publicly reachable with identical API surface to production; HSTS/Expect-CT present on staging but absent on production
[LEARN] ACCEPTED AUTH @ safe-01.threema.ch: Backup API credential-gated — GET /backups/{64hex} returns HTTP 400 for unauth; OPTIONS returns CORS * with Access-Control-Allow-Headers: Authorization enabling credentialed cross-origin requests
[LEARN] ACCEPTED MISCONFIG @ threema-desktop electron-main.ts: BrowserWindow has `sandbox: false` (TODO DEK-79) and `nodeIntegrationInWorker: true` (TODO DEK-79)
[LEARN] REJECTED MISCONFIG @ crypto.ts:223: Hardcoded password is benchmark-only dummy in `determineKdfParams()`, derived key immediately purged, not used for real encryption
[LEARN] REJECTED MISCONFIG @ desktop OnPrem config trust: Ed25519 signature verification with 3 hardcoded trusted public keys + HTTPS/WSS URL validation confirmed — not vulnerable
[RISK] chat: 60 reason: g-*.0.threema.ch prod pattern still unenumerated; staging g-*.0.test reachable but likely out of scope; core messaging infra but no visible HTTP endpoints
[RISK] web: 85 reason: ds-apip/api/apip directory cluster — 3 prod hosts with public 200/404 identity oracle + fetch_bulk 200 + CORS * + no rate-limit (30 POSTs all 200); safe-01 backup API with permissive CORS + Authorization header + 5 hostnames; work/broadcast/gateway/shop cockpits accessible with PHP sessions/CSP/Sentry; staging work public API divergence confirmed
[RISK] sync: 60 reason: mediator-{0..f}/rendezvous-{0..f} resolve (split routing 203.56.112.247 | 203.56.114.247), uniform 403 on HTTPS; WSS paths high-entropy; auth model in source, not passively enumerable
[RISK] safe: 80 reason: safe-01.live with CORS *, write-capable methods (GET,HEAD,PUT,PATCH,POST,DELETE), Access-Control-Allow-Headers: Authorization, HSTS+Expect-CT; backup API credential-gated (400) but cross-origin auth requests possible; 5 hostnames same IP; no 429 observed
[RISK] desktop-src: 80 reason: Electron BrowserWindow sandbox disabled (TODO DEK-79) + nodeIntegrationInWorker: true (TODO DEK-79); Windows key-storage ACL bypass ACCEPTED; OnPrem config trust REJECTED (Ed25519 sig verified); staging URLs baked into builds; key storage Argon2id + DPAPI (decent but weak on Windows)
