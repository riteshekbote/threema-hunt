# LEADS longcat (seed)
- SEED: no model output yet; pipeline starts on first run.
## 2026-08-07 18:29:10 UTC [sync] (model longcat)
## 2026-08-07 18:47:05 UTC [sync] (model longcat)
## 2026-08-07 19:10:58 UTC [sync] (model longcat)
## 2026-08-07 19:59:39 UTC [sync] (model longcat)
## 2026-08-07 20:38:53 UTC [sync] (model longcat)
## 2026-08-07 21:20:02 UTC [sync] (model longcat)
## 2026-08-07 21:58:20 UTC [sync] (model longcat)
## 2026-08-07 22:29:21 UTC [sync] (model longcat)
## 2026-08-07 23:05:16 UTC [sync] (model longcat)
## 2026-08-07 23:41:56 UTC [sync] (model longcat)
## 2026-08-08 00:05:59 UTC [sync] (model longcat)
## 2026-08-08 02:12:24 UTC [sync] (model longcat)
## 2026-08-08 03:43:31 UTC [sync] (model longcat)
## 2026-08-08 04:39:34 UTC [sync] (model longcat)
## 2026-08-08 05:23:16 UTC [sync] (model longcat)
## 2026-08-08 06:02:23 UTC [sync] (model longcat)
## 2026-08-08 07:07:30 UTC [sync] (model longcat)
## 2026-08-08 07:55:28 UTC [sync] (model longcat)
## 2026-08-08 08:24:27 UTC [sync] (model longcat)
## 2026-08-08 09:06:19 UTC [sync] (model longcat)
## 2026-08-08 09:49:48 UTC [sync] (model longcat)
## 2026-08-08 10:15:42 UTC [sync] (model longcat)
## 2026-08-08 10:52:01 UTC [sync] (model longcat)
## 2026-08-08 11:17:05 UTC [sync] (model longcat)
## 2026-08-08 11:45:03 UTC [sync] (model longcat)
## 2026-08-08 12:02:09 UTC [sync] (model longcat)
## 2026-08-08 13:06:23 UTC [sync] (model longcat)
## 2026-08-08 13:52:24 UTC [sync] (model longcat)
## 2026-08-08 14:20:48 UTC [sync] (model longcat)
## 2026-08-08 14:54:57 UTC [sync] (model longcat)
## 2026-08-08 15:15:54 UTC [sync] (model longcat)
## 2026-08-08 15:46:40 UTC [sync] (model longcat)
## 2026-08-08 16:58:07 UTC [sync] (model longcat)
## 2026-08-08 17:26:20 UTC [sync] (model longcat)
## 2026-08-08 17:55:10 UTC [sync] (model longcat)
## 2026-08-08 18:20:49 UTC [sync] (model longcat)
## 2026-08-08 19:03:58 UTC [sync] (model longcat)
## 2026-08-08 19:37:58 UTC [sync] (model longcat)
## 2026-08-08 19:58:37 UTC [sync] (model longcat)
## 2026-08-08 20:27:30 UTC [sync] (model longcat)
## 2026-08-08 20:57:41 UTC [sync] (model longcat)
## 2026-08-08 21:25:10 UTC [sync] (model longcat)
## 2026-08-08 21:52:19 UTC [sync] (model longcat)
## 2026-08-08 22:13:26 UTC [sync] (model longcat)
## 2026-08-08 22:46:29 UTC [sync] (model longcat)
## 2026-08-08 23:10:30 UTC [sync] (model longcat)
## 2026-08-08 23:42:49 UTC [sync] (model longcat)
## 2026-08-09 00:02:35 UTC [sync] (model longcat)
## 2026-08-09 02:21:21 UTC [sync] (model longcat)
## 2026-08-09 03:54:53 UTC [sync] (model longcat)
## 2026-08-09 04:54:11 UTC [sync] (model longcat)
## 2026-08-09 05:38:14 UTC [sync] (model longcat)
## 2026-08-09 06:26:04 UTC [sync] (model longcat)
## 2026-08-09 07:29:58 UTC [sync] (model longcat)
## 2026-08-09 08:08:44 UTC [sync] (model longcat)
## 2026-08-09 08:58:11 UTC [sync] (model longcat)
## 2026-08-09 09:35:52 UTC [sync] (model longcat)
## 2026-08-09 10:07:56 UTC [sync] (model longcat)
## 2026-08-09 10:48:59 UTC [sync] (model longcat)
## 2026-08-09 11:14:36 UTC [sync] (model longcat)
## 2026-08-09 11:45:14 UTC [sync] (model longcat)
## 2026-08-09 12:04:00 UTC [sync] (model longcat)
## 2026-08-09 13:09:59 UTC [sync] (model longcat)
[PRIO] ds-apip.threema.ch/identity/fetch_bulk — score 8.75 (attack:9, business:8, tech:6, gate:10, cloud:4, fresh:5)
[PRIO] safe-01.threema.ch/backups/{id} — score 7.90 (attack:8, business:9, tech:7, gate:6, cloud:7, fresh:4)
[PRIO] threema-desktop key-storage (Windows) — score 7.65 (attack:7, business:8, tech:7, gate:5, cloud:3, fresh:9)
[HYP] fetch_bulk identity enumeration — 10k-ID batch, no rate limit, CORS *
class: IDOR
asset: ds-apip.threema.ch/api.threema.ch/apip.threema.ch identity/fetch_bulk
confidence: 95
reasoning: Confirmed across all 3 prod hosts. 10000 IDs/req returns only valid pubkeys; 10001 → 400 (no partial leak). CORS ACAO:* + Allow-Methods on both 200 and 400. Zero 429s across ~30 sequential POSTs. Overflow guard does not leak.
evidence_needed: None — already fully confirmed.
verify_steps: [PASSIVE — already executed; finding is stable]
impact: Full identity→pubkey enumeration at ~10k IDs/request, no auth, cross-origin readable.
testability: PASSIVE
[HYP] safe backup API credential brute via CORS cross-origin
class: AUTH
asset: safe-01.threema.ch/backups/{64hex}
confidence: 55
reasoning: OPTIONS returns ACAO:* + Allow-Headers: Authorization (credentialed cross-origin requests enabled). Route existence oracle stable (/backups/{64hex}→400 vs /backup/{x}→404). No observed rate limit. HSTS/Expect-CT absent on GET 400 but present on OPTIONS — header inconsistency weakens transport enforcement.
evidence_needed: Auth-gated 400 baseline; no confirmed credential leak. Brute-force of backupId:backupKey tuple is high-entropy (64 hex chars); low practical yield without additional weakness.
verify_steps: AUTH_HELPED — requires valid backupId:backupKey pair to test unauth vs auth response divergence. Passive probe only confirms route gating.
impact: If credential exists: unauthorized backup download (encrypted blobs). High value but low feasibility without credential.
testability: AUTH_HELPED
[HYP] Windows ACL bypass on keystorage.bin — same-user key recovery
class: MISCONFIG
asset: threema-desktop key-storage (Windows)
confidence: 95
reasoning: RAG-verified at 15 source paths. fs.ts:41 returns {} on win32 → keystorage.bin + keystorage.password.bin written without ACL. safeStorage (DPAPI) password recoverable by same-user processes. Inner/v3.ts exposes ck (Ed25519 identity) + databaseKey (SQLCipher). Argon2id→XSalsa20-Poly1305 key purged at crypto.ts:113.
evidence_needed: Source chain confirmed. PoC artifact (poc/key-storage-acl-bypass-poc.js) exists but untested on actual Windows host.
verify_steps: HUMAN_ONLY — requires Windows host with Threema Desktop installed + attacker process running as same user. Node --check passed on Linux (graceful no-op).
impact: Same-user attacker recovers full Ed25519 identity key + SQLCipher database key → decrypts full local message DB.
testability: HUMAN_ONLY
[PARKED] safe backup API credential brute via CORS cross-origin — confidence 55 but testability is AUTH_HELPED + HUMAN_ONLY (requires valid credential); passive-only scope cannot advance; parked pending token/credential from program.
[FINAL]
[NEXT] HUMAN: Request program-issued test credentials for safe-01.threema.ch backup API (backupId:backupKey pair) to validate whether CORS * credentialed requests succeed cross-origin with valid auth header. Alternative: confirm whether rate-limit is absent on GET /backups/{64hex} with valid Basic auth.
[LEARN] ACCEPTED IDOR @ ds-apip.threema.ch/api.threema.ch/apip.threema.ch: fetch_bulk ceiling precisely bounded at 10000 IDs/req; overflow→400 empty body with zero partial pubkey leak; CORS * stable.
[LEARN] ACCEPTED MISCONFIG @ safe-01.threema.ch: HSTS/Expect-CT header inconsistency (present on OPTIONS 204, absent on GET 400) stable across all 5 hosts.
[LEARN] ACCEPTED MISCONFIG @ threema-desktop key-storage (Windows): 15 source-path chain re-verified; PoC artifact exists; needs Windows validation.
[RISK] chat: 15 reason | web: 72 reason | sync: 30 reason | safe: 58 reason | desktop-src: 68 reason
## 2026-08-09 13:56:14 UTC [sync] (model longcat)
## 2026-08-09 14:25:48 UTC [sync] (model longcat)
## 2026-08-09 14:59:10 UTC [sync] (model longcat)
## 2026-08-09 15:31:11 UTC [sync] (model longcat)
## 2026-08-09 15:59:00 UTC [sync] (model longcat)
## 2026-08-09 16:33:36 UTC [sync] (model longcat)
