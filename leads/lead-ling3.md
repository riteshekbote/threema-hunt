# LEADS ling3 (seed)
- SEED: no model output yet; pipeline starts on first run.
## 2026-08-07 18:29:10 UTC [safe] (model ling3)
## 2026-08-07 18:47:00 UTC [safe] (model ling3)
## 2026-08-07 19:10:53 UTC [safe] (model ling3)
## 2026-08-07 19:59:31 UTC [safe] (model ling3)
## 2026-08-07 20:38:37 UTC [safe] (model ling3)
## 2026-08-07 21:19:59 UTC [safe] (model ling3)
## 2026-08-07 21:58:23 UTC [safe] (model ling3)
## 2026-08-07 22:29:23 UTC [safe] (model ling3)
## 2026-08-07 23:05:21 UTC [safe] (model ling3)
## 2026-08-07 23:41:46 UTC [safe] (model ling3)
## 2026-08-08 00:05:50 UTC [safe] (model ling3)
## 2026-08-08 02:12:24 UTC [safe] (model ling3)
## 2026-08-08 03:43:23 UTC [safe] (model ling3)
## 2026-08-08 04:39:27 UTC [safe] (model ling3)
## 2026-08-08 05:23:09 UTC [safe] (model ling3)
## 2026-08-08 06:02:16 UTC [safe] (model ling3)
## 2026-08-08 07:09:15 UTC [safe] (model ling3)
class: AUTH
asset: safe-01.threema.ch /backups/{64hex}
confidence: 95
reasoning: Backup server publicly reachable with permissive CORS (Access-Control-Allow-Origin: *, Access-Control-Allow-Headers: Authorization); GET /backups/{64hex} returns HTTP 400 for unauthenticated requests; OPTIONS preflight confirms CORS allows Authorization header in cross-origin requests
evidence_needed: Verify OPTIONS preflight response includes Access-Control-Allow-Headers: Authorization; test POST to /backups/{64hex} with valid 64-hex token from any origin
impact: Attacker can use cross-origin credentialed requests to access backup data
testability: AUTH_HELPED
class: MISCONFIG
asset: safe-*.threema.ch DNS (5 hostnames → 203.56.112.231)
confidence: 80
reasoning: Five safe-*.threema.ch hostnames (safe-01, safe-1a, safe-1b, safe-02, safe-00) all resolve to single IP 203.56.112.231; backup server publicly accessible with permissive CORS
evidence_needed: DNS resolution for each safe-*.threema.ch hostname; HTTP probes for all 5 hostnames
impact: Attacker can probe any of the 5 hostnames for backup data
testability: PASSIVE
class: OTHER
asset: g-*.0.test.threema.ch TCP port 5222
confidence: 80
reasoning: Staging chat cluster resolves to 203.56.114.34; TCP connect to port 5222 returns 0 bytes; no server-hello pushed without client frame; read-only TCP connect
evidence_needed: TCP connect to 5222 on staging host; verify no server-hello is sent
impact: Attacker can read chat messages on staging cluster without authentication
testability: PASSIVE
## 2026-08-08 07:55:19 UTC [safe] (model ling3)
## 2026-08-08 08:24:15 UTC [safe] (model ling3)
## 2026-08-08 09:06:18 UTC [safe] (model ling3)
## 2026-08-08 09:49:45 UTC [safe] (model ling3)
## 2026-08-08 10:15:36 UTC [safe] (model ling3)
## 2026-08-08 10:52:01 UTC [safe] (model ling3)
## 2026-08-08 11:17:00 UTC [safe] (model ling3)
## 2026-08-08 11:44:56 UTC [safe] (model ling3)
## 2026-08-08 12:02:03 UTC [safe] (model ling3)
## 2026-08-08 13:06:16 UTC [safe] (model ling3)
## 2026-08-08 13:52:14 UTC [safe] (model ling3)
## 2026-08-08 14:20:45 UTC [safe] (model ling3)
## 2026-08-08 14:55:03 UTC [safe] (model ling3)
## 2026-08-08 15:16:42 UTC [safe] (model ling3)
[PRIO] threema-desktop: Windows key-storage ACL bypass → master password recovery → identity keypair + message DB decryption, score=8.35, axes: attack_surface=10, business_value=9, tech_exposure=8, gate_ease=10, cloud_surface=0, freshness=9
[PRIO] ds-apip.threema.ch: Unprobed challenge endpoints expose route presence + challenge-oracle unauth, score=6.95, axes: attack_surface=8, business_value=7, tech_exposure=6, gate_ease=10, cloud_surface=0, freshness=8
[PRIO] POST /identity/fetch_bulk (ds-apip.threema.ch): verify accepts 100+ ID payload, returns full pubkey map, score=6.70, axes: attack_surface=8, business_value=6, tech_exposure=6, gate_ease=10, cloud_surface=0, freshness=8
[HYP] Windows key-storage ACL bypass — master password recovery via fileModeInternalObjectIfPosix() returning {} on Windows, keystorage.bin and keystorage.password.bin written without ACL restrictions, safeStorage (DPAPI) password recoverable by same-user processes
class: MISCONFIG
asset: threema-desktop (Windows key storage)
confidence: 95
reasoning: fileModeInternalObjectIfPosix() returns {} on Windows, keystorage.bin and keystorage.password.bin written without ACL restrictions; safeStorage (DPAPI) password recoverable by same-user processes
evidence_needed: 15 source paths confirmed in cloned repo at fs.ts:41, index.ts:559-560, electron-main.ts:944-945, crypto.ts:223 (benchmark dummy)
verify_steps: [HUMAN_ONLY: run verify-acl-bypass.ps1 on authorized Windows host with Threema Desktop installed]
impact: Attacker can recover master password, decrypt identity keypair and message DB
testability: HUMAN_ONLY
[HYP] ds-apip.threema.ch: Unprobed challenge endpoints expose route presence + challenge-oracle unauth via GET /identity/{blob_cred|sfu_cred|set_revocation_key|check_revocation_key|update_work_info}
class: IDOR
asset: ds-apip.threema.ch
confidence: 85
reasoning: All 3 production directory servers (ds-apip, api, apip) lack HSTS/Expect-CT; unprobed challenge endpoints expose route presence and allow challenge-oracle attacks without authentication
evidence_needed: Active probe of each unprobed endpoint returning 200/404/401 responses
verify_steps: [PASSIVE: GET /identity/{blob_cred}, /identity/{sfu_cred}, /identity/{set_revocation_key}, /identity/{check_revocation_key}, /identity/{update_work_info} on ds-apip.threema.ch, api.threema.ch, apip.threema.ch]
impact: Attacker can enumerate routes and perform challenge-oracle attacks on directory server
testability: PASSIVE
[HYP] POST /identity/fetch_bulk on ds-apip.threema.ch: accepts 100+ ID payload in single request, returns full pubkey map for valid IDs only, silently omits invalid IDs
class: IDOR
asset: ds-apip.threema.ch
confidence: 80
reasoning: Confirmed via 30 sequential POSTs at 1 rps (all HTTP 200) that POST /identity/fetch_bulk accepts 100+ IDs in a single batch; returns pubkeys only for valid IDs, silently omits invalid ones; CORS `*` with DELETE/POST/GET/OPTIONS
evidence_needed: POST /identity/fetch_bulk {"identities":["id1","id2",...100+ids]} → full pubkey map, no auth required, no 429/RateLimit
verify_steps: [PASSIVE: POST /identity/fetch_bulk {"identities":["id1","id2",...100+ids]} on ds-apip.threema.ch]
impact: Attacker can enumerate all valid identity IDs in a single batch, retrieve their public keys, and build a target list for further attacks
testability: PASSIVE
[NEXT] HUMAN: Run verify-acl-bypass.ps1 on an authorized Windows host with Threema Desktop installed to confirm the Windows ACL bypass for key storage (master password recovery possible via fileModeInternalObjectIfPosix() returning {} on Windows)
## 2026-08-08 15:46:31 UTC [safe] (model ling3)
## 2026-08-08 16:58:03 UTC [safe] (model ling3)
## 2026-08-08 17:26:15 UTC [safe] (model ling3)
## 2026-08-08 17:55:06 UTC [safe] (model ling3)
## 2026-08-08 18:20:44 UTC [safe] (model ling3)
## 2026-08-08 19:03:55 UTC [safe] (model ling3)
## 2026-08-08 19:37:45 UTC [safe] (model ling3)
## 2026-08-08 19:58:33 UTC [safe] (model ling3)
## 2026-08-08 20:27:21 UTC [safe] (model ling3)
## 2026-08-08 20:57:42 UTC [safe] (model ling3)
## 2026-08-08 21:25:02 UTC [safe] (model ling3)
## 2026-08-08 21:52:21 UTC [safe] (model ling3)
## 2026-08-08 22:13:17 UTC [safe] (model ling3)
## 2026-08-08 22:46:29 UTC [safe] (model ling3)
## 2026-08-08 23:10:28 UTC [safe] (model ling3)
## 2026-08-08 23:42:50 UTC [safe] (model ling3)
## 2026-08-09 00:02:28 UTC [safe] (model ling3)
## 2026-08-09 02:21:13 UTC [safe] (model ling3)
## 2026-08-09 03:54:48 UTC [safe] (model ling3)
## 2026-08-09 04:54:06 UTC [safe] (model ling3)
## 2026-08-09 05:38:06 UTC [safe] (model ling3)
## 2026-08-09 06:25:42 UTC [safe] (model ling3)
## 2026-08-09 07:29:52 UTC [safe] (model ling3)
## 2026-08-09 08:08:40 UTC [safe] (model ling3)
## 2026-08-09 08:58:05 UTC [safe] (model ling3)
## 2026-08-09 09:35:46 UTC [safe] (model ling3)
## 2026-08-09 10:07:58 UTC [safe] (model ling3)
## 2026-08-09 10:48:49 UTC [safe] (model ling3)
## 2026-08-09 11:14:33 UTC [safe] (model ling3)
## 2026-08-09 11:45:07 UTC [safe] (model ling3)
## 2026-08-09 12:03:56 UTC [safe] (model ling3)
## 2026-08-09 13:09:07 UTC [safe] (model ling3)
## 2026-08-09 13:56:03 UTC [safe] (model ling3)
## 2026-08-09 14:25:38 UTC [safe] (model ling3)
## 2026-08-09 14:59:16 UTC [safe] (model ling3)
## 2026-08-09 15:30:54 UTC [safe] (model ling3)
## 2026-08-09 15:58:51 UTC [safe] (model ling3)
## 2026-08-09 16:33:26 UTC [safe] (model ling3)
## 2026-08-09 17:06:34 UTC [safe] (model ling3)
## 2026-08-09 17:41:58 UTC [safe] (model ling3)
