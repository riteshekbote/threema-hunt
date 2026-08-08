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
