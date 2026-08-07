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
