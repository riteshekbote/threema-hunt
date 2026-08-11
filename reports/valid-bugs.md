# Validated Bugs

- 2026-08-07 ~18:00 UTC - SEED STATE: 0 valid bugs. Pipeline not yet run; hypotheses are recon-based and UNVALIDATED.

- 3 lead(s) marked VALID at 2026-08-07 21:03:52 UTC
  - | Q5 Novel? | **NO** — already ACCEPTED (knowledge/index.md line 21: "POST /identity/fetch_bulk returns pubkeys for valid IDs only, no auth") |
  - | Q4 Passive proof? | **PARTIAL** — can confirm CORS headers + 404 on random IDs; cannot confirm oracle without valid id+key pair |
  - | **VALID** | 0 | — |

- 3 lead(s) marked VALID at 2026-08-07 21:56:25 UTC
  - **Verdict: HOLD** — Speculative; requires AUTH_HELPED with valid credentials to confirm. Needs program to provide test credentials.
  - | Q4 | PARTIAL — can confirm CORS headers; cannot confirm oracle without valid id+key |
  - | **VALID** | 0 | — |

- 2 lead(s) marked VALID at 2026-08-07 23:55:35 UTC
  - | Q3 | **NO** | All data-access routes return 400 (credential-gated). No unauthenticated data access demonstrated. CORS `*` + Authorization header on an auth-gated API only matters if the attacker alr
  - | Q7 | **NO** | Requires valid credentials to have any impact. No unauthenticated data access demonstrated. |

- 3 lead(s) marked VALID at 2026-08-08 04:24:31 UTC
  - **Verdict: VALID**
  - **Verdict: HOLD** — Defense-in-depth/hardening gap. `sandbox:false` + `nodeIntegrationInWorker:true` are real hardening concerns but constitute a vulnerability ONLY when chained with a demonstrated re
  - | **VALID** | 1 | #3 Desktop Windows key-storage ACL bypass |

- 6 lead(s) marked VALID at 2026-08-08 06:01:58 UTC
  - **Verdict: VALID** (duplicate of already-accepted KB finding — no new action needed)
  - **Verdict: HOLD** — credential-gated; real impact requires credential compromise; enumeration oracle is weak (route existence only). AUTH_HELPED needed to confirm (1) HTTP 200 for valid backupId+backu
  - | Q7 | ✅ YES | Reasonable triager accepts as valid local-privilege finding for privacy-focused messenger |
  - **Verdict: VALID** (duplicate of already-accepted KB finding — confidence strengthened 75→95)
  - | 1 | Directory identity→pubkey bulk enumeration + CORS * + no rate limit | IDOR | **VALID** | 5.3 Med | Already accepted |
  - | 3 | Desktop Windows key-storage ACL bypass → identity compromise | MISCONFIG | **VALID** | 5.5 Med | Already accepted (strengthened) |

- 8 lead(s) marked VALID at 2026-08-08 07:07:02 UTC
  - | Q4 Passive proof? | **YES** — `curl -s -X POST https://ds-apip.threema.ch/identity/fetch_bulk -H "Content-Type: application/json" -d '{"identities":["ECHOECHO","ZZZZZZZZ"]}'` returns 200 with pubkey
  - **Verdict: ✅ VALID**
  - **Verdict: ✅ VALID (low severity)**
  - **Verdict: ✅ VALID (medium, local access required)**
  - | Q7 Triager accept? | **NO** — no data access without valid credentials; CORS `*` on a credential-gated endpoint is not a vulnerability |
  - | 1 | Directory cluster identity enumeration (IDOR) | ✅ **VALID** | 5.3 | Medium |
  - | 2 | Staging directory server public exposure | ✅ **VALID** | 3.1 | Low |
  - | 3 | Desktop Windows key storage ACL bypass | ✅ **VALID** | 4.6 | Medium |

- 5 lead(s) marked VALID at 2026-08-08 07:15:49 UTC
  - | **Q4 Proof (GET/HEAD)** | ✅ YES — `GET /identity/ECHOECHO` → 200 with pubkey; `GET /identity/ZZZZZZZZ` → 404; `POST /identity/fetch_bulk` → pubkeys for valid IDs; CORS * confirmed |
  - | **Q6 Not always-rejected** | ❌ **REJECTED** — CORS * on a credential-gated endpoint is best practice / defense-in-depth. No unauthenticated data access. The 400-vs-404 distinction is an oracle but r
  - | **Q2 Reachability** | ⚠️ AUTH_HELPED — requires valid Work test license (401 on unauth) |
  - **Verdict: HOLD** — Interesting hypothesis (OpenAPI flags it "currently buggy" TWRK-1633) but requires AUTH_HELPED testing with a valid Work license. Cannot be proven with passive GET/HEAD only. Retai
  - **VALID leads for reporting: 0**

- 5 lead(s) marked VALID at 2026-08-08 08:07:09 UTC
  - | Q2 Reachable? | ⚠️ PARTIAL | Endpoint responds but returns 400 without valid credentials; auth-gated |
  - | Q3 Real impact? | ❌ NO | CORS `*` on a credential-gated endpoint (400, not 200) is defense-in-depth only. No unauthenticated data access demonstrated. Attacker still needs valid backupId+backupKey |
  - | Q6 Not always-rejected? | ❌ NO | This is on the always-rejected list: CORS `*` on credential-gated endpoint is best practice/defense-in-depth, not a vulnerability (per valid-bugs.md precedent) |
  - | Q1 In scope? | ❌ NO | scope.yml lists `mediator-*.threema.ch` and `rendezvous-*.threema.ch` — staging `.test` variants are not explicitly listed. Per valid-bugs.md precedent, staging variants are tr
  - | Q4 Passive proof? | ❌ NO | Requires AUTH_HELPED (valid credentials) to test session fixation |

- 2 lead(s) marked VALID at 2026-08-08 10:53:22 UTC
  - Verdict: HOLD — speculative. Backend route not deployed on staging (returns method-agnostic 404). If backend catches up, the route could return license credentials (username/password/expires/hasEmail)
  - | VALID   | 0     | — |

- 2 lead(s) marked VALID at 2026-08-08 14:00:44 UTC
  - | Q3 | NO — impact is "valid-ID enumeration at scale; low-medium (pubkeys public by design)" |
  - | **VALID** | 0 | No new, novel, reportable findings |

- 5 lead(s) marked VALID at 2026-08-08 14:52:12 UTC
  - **Verdict: VALID — Medium**
  - **Verdict: VALID — Medium**
  - **Verdict: HOLD** — valid RAG target to unlock chat surface; no vuln to report yet. Action: grep client source for `ServerConfig`/`g-*.0.threema.ch`/`wss://` chat URL.
  - | 1 | Directory cluster identity→pubkey enumeration (fetch_bulk + GET /identity/{id}, CORS*, no rate-limit) | **VALID** (Medium) | Mass enumeration of valid IDs+pubkeys at scale; CVSS 4.3 |
  - | 4 | Desktop Windows key storage: no ACL on keystorage.bin + keystorage.password.bin | **VALID** (Medium) | Same-user process reads DPAPI password + encrypted keystore; CVSS 5.5 |

- 14 lead(s) marked VALID at 2026-08-08 15:47:00 UTC
  - | Q5 Novel? | ❌ **NO** — already ACCEPTED in KB (lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104); triaged as VALID in valid-bugs.md multiple times |
  - | Q5 Novel? | ❌ **NO** — already ACCEPTED in KB (lines 26, 74, 77-78, 92, 108); triaged as VALID in valid-bugs.md |
  - | Q5 Novel? | ❌ **NO** — already ACCEPTED in KB (lines 28-29, 61, 68, 93); triaged as VALID in valid-bugs.md |
  - | Q3 Real impact? | ❌ NO — `GET /backups/{64hex}` returns 400 (not 200/401/404); requires valid backupId+backupKey; CORS* on credential-gated endpoint only matters if attacker already has credentials 
  - | Q4 Passive proof? | ⚠️ Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q6 Not always-rejected? | ❌ **NO** — CORS* on a credential-gated endpoint is best-practice/defense-in-depth, not a vulnerability (per valid-bugs.md precedent) |
  - | Q7 Triager accept? | ❌ NO — no data access demonstrated; requires valid credentials |
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — requires valid credentials to test login flow |
  - | Q5 Novel? | ⚠️ Not previously triaged in valid-bugs.md, but low confidence |
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — 401 on unauth; requires valid Work license |
  - | Q5 Novel? | ⚠️ Not previously triaged in valid-bugs.md |
  - **Verdict: HOLD.** OpenAPI flags it "currently buggy" but requires AUTH_HELPED with valid Work license. Retain for program-provided test credentials.
  - | Q3 Real impact? | ⚠️ LOW — route presence + parameter-validation oracle; challenge-response still requires valid identity+secret |
  - | **VALID (new)** | **0** | No novel, reportable findings |

- 5 lead(s) marked VALID at 2026-08-08 17:51:00 UTC
   - | Q5 Novel? | ❌ **NO** — all 5 leads are duplicates of already-ACCEPTED KB findings |
   - | Q6 Not always-rejected? | ❌ **NO** — safe-01 CORS* on credential-gated endpoint is defense-in-depth (per precedent) |
   - | **VALID (new)** | **0** | No novel, reportable findings |
   - DISCREPANCY: Challenge endpoints probed as 404 vs KB-documented 200 — needs re-probe |

- 8 lead(s) marked VALID at 2026-08-08 17:19:51 UTC
  - VERDICT: VALID
  - | **VALID** | 6 | Directory cluster IDOR, Desktop Windows key-storage ACL bypass, Safe backup CORS, Safe HSTS inconsistency, Challenge-parameter-oracle, work.test bundle divergence |
  - VERDICT: VALID
  - VERDICT: VALID
  - VERDICT: VALID (low severity)
  - VERDICT: VALID (low severity)
  - VERDICT: VALID (low severity)
  - VALID (report immediately):

- 4 lead(s) marked VALID at 2026-08-08 17:57:58 UTC
  - | **VALID** | 0 | — |
  - - **Verdict: HOLD.** OpenAPI flags it "currently buggy" but requires AUTH_HELPED with valid Work license. Retain for program-provided test credentials.
  - - VERDICT: VALID
  - - VERDICT: VALID

- 5 lead(s) marked VALID at 2026-08-08 18:19:30 UTC
  - | Q4 GET/HEAD proof? | ✅ Yes — `GET /identity/{id}` returns 200/404; `POST /identity/fetch_bulk` returns pubkeys for valid IDs |
  - **Verdict: VALID**
  - **Verdict: VALID** (with note: requires local/same-user access on Windows)
  - | 1 | Directory cluster identity→pubkey enumeration (fetch_bulk + CORS *, no rate limit) | **VALID** | 3.7 | Medium |
  - | 2 | Desktop Windows key-storage ACL bypass → identity + DB compromise | **VALID** | 7.1 | Medium-High |

- 11 lead(s) marked VALID at 2026-08-08 19:08:40 UTC
  - | Q6 Not always-rejected? | ❌ NO — CORS* on credential-gated endpoint is best-practice/defense-in-depth (per valid-bugs.md precedent) |
  - | Q7 Triager accept? | ❌ NO — no data access without valid credentials |
  - | Q3 Real impact? | ⚠️ LOW — route-presence + parameter-validation oracle; challenge-response still requires valid identity+secret |
  - | Q3 Real impact? | ✅ YES — mass enumeration of valid IDs + pubkeys |
  - | Q7 Triager accept? | ✅ YES — already accepted as finding #3 in valid-bugs.md |
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — 401 on all paths; requires valid Work license |
  - | Q4 Passive proof? | ❌ NO — requires AUTH_HELPED with valid Work test license |
  - | Q5 Novel? | ⚠️ Not previously triaged as a valid vuln (only hypothesized) |
  - **Verdict: HOLD** — Interesting hypothesis (OpenAPI flags it "currently buggy" TWRK-1633) but requires AUTH_HELPED with valid Work license. Cannot be proven with passive GET/HEAD only. Retain for prog
  - | Q7 Triager accept? | ⚠️ HOLD territory — valid hardening concern but constitutes a vulnerability ONLY when chained with a demonstrated RCE primitive |
  - | **VALID** | **0** | No new, novel, reportable findings this cycle |

- 9 lead(s) marked VALID at 2026-08-08 20:16:43 UTC
  - | Q3 Real impact? | ⚠️ CONTINGENT — valid backupId:backupKey required for data access; CORS `*` enables cross-origin credentialed reads but no data without creds |
  - | Q6 Not always-rejected? | ❌ NO — CORS `*` on credential-gated endpoint is defense-in-depth only per valid-bugs.md precedent |
  - | Q7 Triager accept? | ❌ NO — no data access without valid credentials; duplicate |
  - | Q7 Triager accept? | ✅ YES — already accepted as finding #3 in valid-bugs.md |
  - | Q3 Real impact? | ✅ YES — mass enumeration of valid IDs + pubkeys at scale |
  - | Q7 Triager accept? | ✅ YES — already accepted as finding #1 in valid-bugs.md |
  - | Q7 Triager accept? | ❌ NO — no data access without valid credentials |
  - | Q7 Triager accept? | ⚠️ Not standalone — valid hardening concern but not a vuln without secondary bug |
  - | **VALID** | **0** | No new, novel, reportable findings this cycle |

- 5 lead(s) marked VALID at 2026-08-08 21:53:06 UTC
  - **Verdict: HOLD → escalate to VALID after duplicate check**
  - | Q3 Impact | **YES** — backup-ID existence enumeration (400-vs-404 oracle); with stolen/valid credentials, full backup read = identity keypair + message history |
  - **Verdict: VALID**
  - | H1 | Directory identity→pubkey oracle (bulk) | **HOLD → VALID** | Needs duplicate check; bulk silent-omit + CORS `*` + no rate limit is the finding |
  - | H3 | Desktop Windows key-storage ACL bypass | **VALID** | Source-verified; same-user malware extracts Ed25519 private key + DB key via DPAPI |

- 9 lead(s) marked VALID at 2026-08-08 22:53:52 UTC
  - | Q5 Novel | **NO** — already ACCEPTED in KB (knowledge/index.md lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104); triaged as VALID in valid-bugs.md multiple times |
  - | Q7 Triager accept | **PASS** — already accepted as finding #1 in valid-bugs.md |
  - | Q3 Impact | **CONTINGENT** — backup-ID existence enumeration (400-vs-404 oracle); full backup read requires valid backupId+backupKey |
  - | Q5 Novel | **NO** — already ACCEPTED in KB (lines 30-31, 62, 79, 101, 106, 125, 128-129, 132, 135, 138); triaged as VALID in valid-bugs.md |
  - | Q7 Triager accept | **CONDITIONAL** — already accepted as finding in valid-bugs.md; the 400-vs-404 oracle alone is weak but the CORS+Authorization pattern was accepted |
  - | Q5 Novel | **NO** — already ACCEPTED in KB (lines 26, 74, 77-78, 92, 108, 121, 130, 133, 137); triaged as VALID in valid-bugs.md multiple times |
  - | Q7 Triager accept | **PASS** — already accepted as finding #3 in valid-bugs.md |
  - | Q5 Novel | **NO** — already ACCEPTED in KB (lines 32, 63, 75, 81, 95, 115, 118, 123); triaged as HOLD in valid-bugs.md |
  - | Q7 Triager accept | **HOLD** — already triaged as HOLD/contingent in valid-bugs.md; exploitability depends on undiscovered secondary bug |

- 6 lead(s) marked VALID at 2026-08-08 23:18:20 UTC
  - | H1 | Directory identity→pubkey bulk oracle (fetch_bulk + GET /identity/{id}) | IDOR | ALL | Duplicate of prior VALID |
  - | H2 | Safe backup CORS `*` + credentialed cross-origin read + existence oracle | AUTH | bigpickle, laguna, ling3, nemotron3 | Duplicate of prior VALID |
  - | H3 | Desktop Windows key-storage ACL bypass → DPAPI → identity keypair + DB | MISCONFIG | bigpickle, laguna, ling3, nemotron3 | Duplicate of prior VALID |
  - | Q4 Proof passive | **YES** — 100-ID batch → 200, only valid pubkey returned, silent omission; 30× POST no 429 |
  - | Q7 Triager accept | CONDITIONAL — already accepted in valid-bugs.md |
  - | Q5 Novel | **YES** — not previously triaged in valid-bugs.md or knowledge/index.md |

- 15 lead(s) marked VALID at 2026-08-09 00:41:41 UTC
  - | Q3 Real impact | ✅ YES | Mass valid-ID + pubkey harvesting → targeted phishing / recon |
  - | Q5 Novel | ❌ NO | ACCEPTED in KB lines 20-21,27,59,67,80,82,85-86,94,99,104; VALID in valid-bugs.md |
  - | Q5 Novel | ❌ NO | ACCEPTED in KB lines 26,74,77-78,92,108,121,130,133,137; VALID in valid-bugs.md |
  - | Q2 Reachable | ⚠️ AUTH_HELPED | Returns 400 without valid backupId+backupKey; no data access demonstrated unauthenticated |
  - | Q3 Real impact | ⚠️ CONTINGENT | 400-vs-404 oracle (route-existence) weak; full backup read requires valid creds |
  - | Q5 Novel | ❌ NO | ACCEPTED in KB lines 30-31,62,79,101,106,125,128-129,132,135,138; triaged in valid-bugs.md |
  - | Q6 Not always-rejected | ❌ NO | CORS \* on credential-gated endpoint is defense-in-depth only (per valid-bugs.md precedent) |
  - | Q3 Real impact | ⚠️ LOW | Route-presence + parameter oracle; challenge-response still requires valid identity+secret |
  - | Q5 Novel | ❌ NO | ACCEPTED in KB lines 32,63,75,81,95,115,118,123; REJECTED as standalone in valid-bugs.md (conditional) |
  - | Q7 Triager accept | ❌ NO | Already REJECTED as standalone in valid-bugs.md |
  - | Q4 Passive proof | ❌ NO | Requires AUTH_HELPED with valid Work test license |
  - | Q5 Novel | ❌ NO | ACCEPTED then deemed OUT OF SCOPE in KB lines 36,56; REJECTED in valid-bugs.md |
  - | Q2 Reachable | ⚠️ AUTH_HELPED | Requires login flow with valid creds to confirm |
  - | VALID (new, reportable) | **0** |
  - **Valid leads for reporting this cycle: 0.**

- 6 lead(s) marked VALID at 2026-08-09 03:00:14 UTC
  - | Q3 Real impact? | ❌ NO | 400 = credential-gated (HTTP Basic Auth backupId:backupKey). No data access without valid credentials. Route-existence oracle (400 vs 404) is weak. |
  - | Q6 Not always-rejected? | ❌ NO | CORS* on credential-gated endpoint is defense-in-depth per valid-bugs.md precedent |
   - | Q3 Real impact? | ❌ NO | No data access without valid credentials |
   - | Q3 Real impact? | ❌ NO | No data access without valid credentials |
   - | **VALID** | **0** | No novel, reportable findings |
   - **Valid leads for reporting this cycle: 0.**

- 15 lead(s) marked VALID at 2026-08-09 05:48:00 UTC
   - | Q5 Novel? | ❌ NO | All 15 leads are duplicates of already-ACCEPTED KB findings (knowledge/index.md) |
   - | Q3 Real impact? | ❌ NO | 400/401/404 = credential-gated or dead; no unauthenticated data access demonstrated |
   - | Q1 In scope? | ❌ NO | saltyrtc-*.threema.ch NOT in scope.yml |
   - | Q7 Triager accept? | ❌ NO | Login/signup/shop pages + 401-gated APIs are expected behavior, not vulns |
   - | **VALID** | **0** | No novel, reportable findings |
   - **Valid leads for reporting this cycle: 0.**

- 1 lead(s) marked VALID at 2026-08-09 06:39:26 UTC
  - VALID: 0

- 16 lead(s) marked VALID at 2026-08-09 07:44:09 UTC
  - | Q3 Real impact? | ✅ YES — mass valid-ID + pubkey harvesting → targeted phishing / recon |
  - | Q5 Novel? | ❌ NO — already ACCEPTED in knowledge base and triaged as VALID in `valid-bugs.md` multiple times |
  - **Verdict:** **VALID (duplicate)** — Medium | CVSS 3.1: **4.3 (AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N)**
  - | Q5 Novel? | ❌ NO — already ACCEPTED in knowledge base and triaged as VALID multiple times |
  - **Verdict:** **VALID (duplicate, confidence strengthened)** — Medium-High | CVSS 3.1: **5.5 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N)** (local attack vector, high confidentiality impact)
  - | Q5 Novel? | ❌ NO — already ACCEPTED in knowledge base, triaged as HOLD/conditional in `valid-bugs.md` |
  - | Q7 Accept? | ❌ NO as standalone — valid concern but not a vuln without demonstrated secondary bug |
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — `GET /backups/{64hex}` returns 400 without valid `backupId:backupKey`; data access requires credentials |
  - | Q4 Passive proof? | ⚠️ Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | ❌ NO — already triaged in `valid-bugs.md`, REJECTED as defense-in-depth |
  - | Q7 Accept? | ❌ NO — no data access without valid credentials |
  - | Q1 In scope? | ❌ **NO** — scope.yml lists `apip.threema.ch` and `mediator-*.threema.ch` etc., but `.test` staging variants are **not** listed. Per `valid-bugs.md` precedent, staging variants treated
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — returns 401 on all paths; requires valid Work test license |
  - | Q5 Novel? | ⚠️ Not triaged as valid vuln, only hypothesized; OpenAPI flags it "currently buggy" (TWRK-1633) |
  - **Verdict:** **HOLD** — OpenAPI flags it "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven with passive GET/HEAD only. Retain for program-provided t
  - | **VALID** | 2 | #1 Directory IDOR (duplicate), #2 Desktop Windows key-storage ACL (duplicate) |

- 2 lead(s) marked VALID at 2026-08-09 08:32:55 UTC
  - | **VALID (new)** | **0** | No novel, reportable findings this cycle. |
  - **Valid leads for reporting this cycle: 0.**

- 14 lead(s) marked VALID at 2026-08-09 10:21:23 UTC
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104; triaged as VALID in valid-bugs.md multiple times |
  - | **Q7 Triager accept?** | ✅ YES — already accepted as finding #1 in valid-bugs.md |
  - | **Q2 Reachable?** | ⚠️ AUTH_HELPED — returns 400 without valid `backupId:backupKey` (HTTP Basic Auth); no unauthenticated data access |
  - | **Q3 Real impact?** | ❌ NO — credential-gated; 400 = route exists but auth required; CORS `*` on credential-gated endpoint is defense-in-depth only (per valid-bugs.md precedent) |
  - | **Q4 Passive proof?** | ⚠️ Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | **Q6 Not always-rejected?** | ❌ NO — CORS `*` on credential-gated endpoint is best-practice/defense-in-depth (per valid-bugs.md precedent) |
  - | **Q7 Triager accept?** | ❌ NO — no data access without valid credentials; duplicate |
  - | **Q1 In scope?** | ❌ NO — scope.yml lists `work.threema.ch` but NOT `work.test.threema.ch` staging variants (per valid-bugs.md precedent, staging variants treated as out-of-scope) |
  - | **Q1 In scope?** | ❌ NO — `work.test.threema.ch` is staging; scope.yml lists `work.threema.ch` only (per valid-bugs.md precedent, staging variants treated as out-of-scope) |
  - | **Q5 Novel?** | ❌ NO — ACCEPTED in KB lines 36, 56; triaged as VALID then deemed OUT OF SCOPE |
  - | **Q5 Novel?** | ❌ NO — ACCEPTED in KB; validation oracle disproven in valid-bugs.md |
  - | **Q1 In scope?** | ❌ NO — `.test` staging variant NOT in scope.yml (per valid-bugs.md precedent) |
  - | **VALID** | **0** | No novel, reportable findings |
  - **VALID (new, reportable) leads: 0**

- 3 lead(s) marked VALID at 2026-08-09 11:53:46 UTC
  - VALID (new, reportable): 0
  - | 7 | ds-apip-work /identities cross-subscription metadata disclosure (TWRK-1633) | BUSLOGIC | bigpickle,ling3,ling3 | Y | AUTH_HELPED | med | N | Y(?) | Y | ? | **HOLD** | Requires AUTH_HELPED with v
  - | **VALID (new)** | 0 | — |

- 6 lead(s) marked VALID at 2026-08-09 12:22:28 UTC
  - **Verdict: ✅ VALID**
  - | Q3 | **Debatable** — Endpoint is credential-gated (400 for unauth). CORS * enables cross-origin requests *with* credentials, but attacker still needs valid `backupId:backupKey`. No unauthenticated d
  - | Q6 | **REJECTED** — CORS `*` on a credential-gated endpoint (HTTP 400 for unauthenticated requests) is a defense-in-depth gap, not a vulnerability. The always-rejected list includes "best practice" 
  - | Q2 | **NO** — Returns 401 on all paths. Requires valid Work license. |
  - | Q4 | **NO** — Requires AUTH_HELPED (valid credentials) to compare pre/post-login session IDs. Violates passive-first. |
  - | 1 | Directory cluster identity→pubkey oracle | ✅ **VALID** | Unauth bulk existence oracle + CORS * + no rate limit |

- 11 lead(s) marked VALID at 2026-08-09 13:28:52 UTC
  - | Q3 Real impact? | **YES** — mass valid-ID + pubkey enumeration at ~10k IDs/req, no rate limit |
  - | Q4 Passive proof? | **YES** — `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → 200, only valid pubkey returned |
  - | Q5 Novel? | **NO** — ACCEPTED in KB lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104; VALID in valid-bugs.md since 2026-08-07 |
  - | Q5 Novel? | **NO** — ACCEPTED in KB lines 30-31, 62, 79, 101, 106; triaged in valid-bugs.md |
  - | Q7 Triager accept? | **NO** — no data access without valid credentials |
  - | Q2 Reachable? | **NO** — 401 on all paths; requires valid Work license |
  - | Q4 Passive proof? | **NO** — requires AUTH_HELPED with valid Work license |
  - **Verdict: HOLD** — Requires AUTH_HELPED with valid Work test license. Retain for program-provided credentials.
  - | Q3 Real impact? | **LOW** — route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret |
  - | Q5 Novel? | **NO** — triaged in valid-bugs.md |
  - | **VALID** | **0** | — |

- 1 lead(s) marked VALID at 2026-08-09 14:24:02 UTC
  - | **VALID** | **0** | No novel, reportable findings |

- 5 lead(s) marked VALID at 2026-08-09 15:00:45 UTC
  - ### Verdict: **VALID** (Medium)
  - | Q2 | **Partial** — route is credential-gated (400 for unauth, not 404). CORS * with Allow-Headers: Authorization means an attacker page can make *credentialed* cross-origin requests — but only if th
  - | Q3 | **Low** — without valid backupId:backupKey (64-hex + high-entropy key, unguessable), no data access. The 400-vs-404 distinction reveals route existence only (already known from source). |
  - | Q7 | **Hold** — potentially valid but requires program-issued test credentials. |
  - | 1 | Directory cluster identity→pubkey oracle + CORS * + no rate limit | **VALID** | Medium (5.3) | Threema security (confirm channel) |

- 23 lead(s) marked VALID at 2026-08-09 17:59:49 UTC
  - | Q3 Real impact? | ✅ YES | Mass valid-ID + pubkey harvesting at ~10k IDs/request, no rate limit, cross-origin readable → targeted phishing/recon at scale |
  - | Q4 Passive proof? | ✅ YES | `GET /identity/ECHOECHO`→200, `/identity/ZZZZZZZZ`→404; `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}`→200 with only valid pubkey; 30× sequential POST
  - | Q5 Novel? | ❌ NO | Already ACCEPTED in knowledge/index.md (lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104, 111, 124, 131, 134, 136, etc.); triaged as VALID in valid-bugs.md since 2026-08-07 |
  - | Q7 Triager accept? | ✅ YES | Already accepted as finding #1 in valid-bugs.md |
  - **Verdict: VALID (duplicate)** — Confidence 95. Already reported. CVSS 3.1: **5.3 (AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N)**
  - | Q5 Novel? | ❌ NO | Already ACCEPTED in knowledge/index.md (lines 26, 74, 77-78, 92, 108, 121, 130, 133, 137, etc.); triaged as VALID in valid-bugs.md |
  - | Q7 Triager accept? | ✅ YES | Already accepted as finding #3 in valid-bugs.md |
  - **Verdict: VALID (duplicate)** — Confidence 95. Already reported. CVSS 3.1: **5.5 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N)** — needs Windows host validation for full PoC
  - | Q3 Real impact? | ❌ NO | Credential-gated (400, not 200). No unauthenticated data access demonstrated. `CORS *` + `Allow-Headers: Authorization` only matters if attacker already has valid `backupId:
  - | Q4 Passive proof? | ⚠️ PARTIAL | Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | ❌ NO | Already ACCEPTED in KB; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | ❌ NO | CORS `*` on credential-gated endpoint is defense-in-depth only (per valid-bugs.md precedent) |
  - | Q7 Triager accept? | ❌ NO | No data access without valid credentials |
  - | Q7 Triager accept? | ⚠️ CONDITIONAL | Valid hardening concern but not a vuln without demonstrated RCE chain |
  - **Verdict: VALID (duplicate, low severity)** — Already accepted in knowledge base. CVSS 3.1: **3.1 (AV:N/AC:H/PR:N/UI:R/S:U/C:L/I:N/A:N)** — low severity defense-in-depth gap
  - | Q3 Real impact? | ⚠️ LOW | Route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret |
  - **Verdict: VALID (duplicate, low severity)** — Already accepted. CVSS 3.1: **3.7 (AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N)** — low severity info disclosure
  - | Q2 Reachable? | ⚠️ AUTH_HELPED | Returns 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | ❌ NO | Requires AUTH_HELPED with valid Work credentials + cross-subscription contact probes |
  - | Q5 Novel? | ⚠️ YES | Not previously triaged as valid vuln; only hypothesized (OpenAPI flags it "currently buggy" TWRK-1633) |

- 2 lead(s) marked VALID at 2026-08-09 18:22:49 UTC
  - | VALID (duplicate) | 4 | Directory IDOR, Desktop key-storage ACL, Safe HSTS inconsistency, Challenge oracles |
  - | **VALID (duplicate)** | 4 | Directory IDOR, Desktop key-storage ACL, Safe HSTS inconsistency, Challenge oracles |

- 4 lead(s) marked VALID at 2026-08-09 19:10:15 UTC
  - | Q5 Novel? | **NO** | Already ACCEPTED — knowledge/index.md lines 20-21, 59, 67, 80, 82, 85-86, 94, 99, 104, 111, 124, 131, 134, 136, 142, 150, 158, 163, 167, 170, 172, 182, 189, 195, 199, 204, 220; 
  - | Q7 Triager accept? | NO | No data access without valid credentials |
  - | Q5 Novel? | **NO** | Already ACCEPTED — KB lines 20-21, 82, 85-86, 124; valid-bugs.md #1 |
  - | **VALID** | 0 | — |

- 17 lead(s) marked VALID at 2026-08-09 20:19:10 UTC
  - | Q3 Real impact? | YES | Mass valid-ID + pubkey harvesting at ~10k IDs/request, no rate limit, cross-origin readable → targeted phishing/recon at scale |
  - | Q4 Passive proof? | YES | `GET /identity/ECHOECHO`→200, `/identity/ZZZZZZZZ`→404; `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}`→200 with only valid pubkey; 30× sequential POSTs 
  - | Q5 Novel? | **NO** | Already ACCEPTED in knowledge base; triaged as VALID in `valid-bugs.md` since 2026-08-07 (finding #1) |
  - | Q7 Triager accept? | YES | Already accepted as finding #1 in valid-bugs.md |
  - | Q5 Novel? | **NO** | Already ACCEPTED in knowledge base; triaged as VALID in `valid-bugs.md` (finding #3) |
  - | Q7 Triager accept? | YES | Already accepted as finding #3 in valid-bugs.md |
  - | Q3 Real impact? | NO | Credential-gated (400, not 200). No unauthenticated data access demonstrated. CORS `*` + Allow-Headers: Authorization only matters if attacker already has valid `backupId:back
  - | Q4 Passive proof? | PARTIAL | Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | NO | Already ACCEPTED in KB; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | **NO** | CORS `*` on credential-gated endpoint is defense-in-depth only (per valid-bugs.md precedent) |
  - | Q7 Triager accept? | NO | No data access without valid credentials |
  - | Q3 Real impact? | LOW | Route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret |
  - | Q2 Reachable? | AUTH_HELPED | Returns 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | NO | Requires AUTH_HELPED with valid Work credentials + cross-subscription contact probes |
  - | Q5 Novel? | YES | Not previously triaged as valid vuln; only hypothesized (OpenAPI flags it "currently buggy" TWRK-1633) |
  - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven via passive GET/HEAD only. Retain for program-provided 
   - | **VALID (new)** | **0** | — |

- 12 lead(s) marked VALID at 2026-08-09 20:56 UTC
   - | Q5 Novel? | **NO** | Already ACCEPTED in knowledge/index.md; triaged as VALID in valid-bugs.md |
   - | Q5 Novel? | **NO** | Already ACCEPTED in knowledge/index.md; triaged as VALID in valid-bugs.md |
   - | Q3 Real impact? | **NO** | Credential-gated (400, not 200); CORS * on credential-gated endpoint is defense-in-depth only |
   - | Q5 Novel? | **NO** | Already ACCEPTED in KB; triaged in valid-bugs.md |
   - | Q5 Novel? | **NO** | Already ACCEPTED in KB; triaged in valid-bugs.md |
   - | Q4 Passive proof? | **NO** | Requires AUTH_HELPED with valid Work test license |
   - | Q1 In scope? | **NO** | staging `.test` variants not in scope.yml |
   - | Q1 In scope? | **NO** | staging not in scope.yml |
   - | Q7 Triager accept? | **NO** | Conditional RCE requires separate renderer exploit chain; not standalone |
   - | Q3 Real impact? | **NONE** | Debunked — Ed25519 signature verification confirmed secure |
   - | Q3 Real impact? | **NONE** | Debunked — benchmark-only dummy, key immediately purged |
   - | Q2 Reachable? | **NO** | Returns 404 — endpoint not publicly routable |
   - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Retain for program-provided test credentials.
   - | **VALID (new)** | **0** | — |

- 3 lead(s) marked VALID at 2026-08-09 20:59:29 UTC
  - - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven via passive GET/HEAD only. Retain for program-provide
  - +   - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Retain for program-provided test credentials.
  - | **VALID** | 4 | All duplicates of previously reported findings |

- 1 lead(s) marked VALID at 2026-08-09 21:58:03 UTC
  - | **VALID** | 4 | All duplicates — already in `valid-bugs.md` |

- 21 lead(s) marked VALID at 2026-08-09 22:21:15 UTC
  - | Q3 Real impact? | YES | Mass valid-ID + pubkey harvesting at ~10k IDs/request, no rate limit, cross-origin readable → targeted phishing/recon at scale |
  - | Q4 Passive proof? | YES | `GET /identity/ECHOECHO`→200, `/identity/ZZZZZZZZ`→404; `POST /identity/fetch_bulk`→200 with only valid pubkey; 35× sequential POSTs all 200, zero 429 |
  - | Q5 Novel? | NO | Already ACCEPTED in KB (lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104, 111, 124, 131, 134, 136, 142, 150, 158, 163, 167, 170, 172, 182, 189, 195, 199, 204, 220); triaged as VA
  - | Q7 Triager accept? | YES | Already accepted as finding #1 in valid-bugs.md |
  - **Verdict: VALID (duplicate)** — Confidence 95. Already reported. CVSS 3.1: **5.3 (AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N)**
  - | Q5 Novel? | NO | Already ACCEPTED in KB (lines 26, 74, 77-78, 92, 108, 121, 130, 133, 137, 143, 187, 192, 197, 203); triaged as VALID in valid-bugs.md |
  - | Q7 Triager accept? | YES | Already accepted as finding #3 in valid-bugs.md |
  - **Verdict: VALID (duplicate)** — Confidence 95. Already reported. CVSS 3.1: **5.5 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N)** — needs Windows host validation for full PoC
  - | Q3 Real impact? | NO | Credential-gated (400, not 200). No unauthenticated data access demonstrated. CORS `*` + Allow-Headers: Authorization only matters if attacker already has valid `backupId:back
  - | Q4 Passive proof? | PARTIAL | Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | NO | Already ACCEPTED in KB; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | NO | CORS `*` on credential-gated endpoint is defense-in-depth only (per valid-bugs.md precedent) |
  - | Q7 Triager accept? | NO | No data access without valid credentials |
  - **Verdict: VALID (duplicate, low severity)** — Already accepted. CVSS 3.1: **3.1 (AV:N/AC:H/PR:N/UI:R/S:U/C:L/I:N/A:N)**
  - | Q3 Real impact? | LOW | Route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret |
  - **Verdict: VALID (duplicate, low severity)** — Already accepted. CVSS 3.1: **3.7 (AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N)**
  - | Q2 Reachable? | AUTH_HELPED | Returns 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | NO | Requires AUTH_HELPED with valid Work credentials + cross-subscription contact probes |
  - | Q5 Novel? | YES | Not previously triaged as valid vuln; only hypothesized (OpenAPI flags it "currently buggy" TWRK-1633) |
  - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven via passive GET/HEAD only. Retain for program-provided 

- 1 lead(s) marked VALID at 2026-08-09 22:59:21 UTC
  - | **VALID** | 4 | All duplicates — already in `valid-bugs.md` |

- 22 lead(s) marked VALID at 2026-08-09 23:54:21 UTC
  - | Q3 Real impact? | YES — mass valid-ID + pubkey harvesting at ~10k IDs/req, silent omission oracle, CORS* makes it cross-origin readable, zero rate limit |
  - | Q4 Passive proof? | YES — `GET /identity/ECHOECHO`→200+pubkey, `/identity/ZZZZZZZZ`→404; `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}`→200 valid-only; OPTIONS→ACAO:* |
  - | Q5 Novel? | **NO** — ACCEPTED KB lines 20-21,27,59,67,80,82,85-86,94,99,104,111,124,131,134,136; VALID #1 in valid-bugs.md since 2026-08-07 |
  - | Q5 Novel? | **NO** — ACCEPTED KB lines 26,74,77-78,92,108,121,130,133,137,143,187; VALID #3 in valid-bugs.md |
  - | Q5 Novel? | **NO** — ACCEPTED KB lines 32,63,75,81,95,115,118,123; triaged as conditional/HOLD in valid-bugs.md |
  - | Q6 Not always-rejected? | Borderline — valid hardening concern, not a standalone vuln |
  - **Verdict: HOLD (conditional — valid defense-in-depth gap, not a standalone reportable vuln).** Retriaged as HOLD per valid-bugs.md precedent. Re-open only if a secondary worker exploit primitive is f
  - | Q2 Reachable? | Route reachable; data access requires valid `backupId:backupKey` (64-hex id + high-entropy key). Unauthenticated GET→400, not 200/404/401 |
  - | Q3 Real impact? | NO without valid credentials. 400-vs-404 oracle reveals route existence only (already known from source/KB). CORS* lets an attacker page send credentialed cross-origin requests, bu
  - | Q4 Proof? | Partial — CORS headers + 400 response confirmed; data access needs valid creds |
  - | Q5 Novel? | **NO** — ACCEPTED KB lines 30-31,62,79,101,106,125; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | **NO** — CORS* on a credential-gated endpoint (400) is best-practice/defense-in-depth only, per valid-bugs.md precedent (2026-08-08 08:07:09 UTC, 07:15:49 UTC, etc.) |
  - | Q3 Real impact? | LOW — route presence + parameter-validation oracle (e.g. `set_revocation_key`→"Bad revocation key length"); challenge-response still requires valid identity+secret. No auth bypass 
  - | Q2 Reachable? | AUTH_HELPED — returns 401 on all paths; requires valid Work test license |
  - | Q4 Proof? | **NO** — requires AUTH_HELPED with valid Work license + cross-subscription contact probes |
  - | Q5 Novel? | Not triaged as a valid vuln (only hypothesized) |
  - **Verdict: HOLD.** Interesting (OpenAPI TWRK-1633), but cannot be proven with passive GET/HEAD. Retain for program-provided Work test credentials. Consistent with valid-bugs.md HOLD precedent across m
  - | Q1 In scope? | **NO** — scope.yml lists `apip.threema.ch` and `safe-*.threema.ch` etc.; `.test` staging variants are **not** listed. Per valid-bugs.md precedent, staging variants treated as out-of-s
  - | Q1 In scope? | **NO** — scope.yml lists `work.threema.ch`, not `work.test.threema.ch`. Per valid-bugs.md precedent, staging treated as out-of-scope |
  - | Q1 | **NO** — scope.yml lists `g-*.0.threema.ch`, `mediator-*.threema.ch`, `rendezvous-*.threema.ch`; `.test` staging variants NOT listed. Per valid-bugs.md precedent, out of scope |

- 4 lead(s) marked VALID at 2026-08-10 00:46:43 UTC
  - | 1 | Directory cluster identity→pubkey bulk enumeration (fetch_bulk + CORS* + no rate limit) | **VALID (dup)** | Already reported. CVSS 5.3 |
  - | 2 | Desktop Windows key-storage ACL bypass (keystorage.bin no ACL) | **VALID (dup)** | Already reported. CVSS 5.5 |
  - | 4 | Safe backup HSTS/Expect-CT header inconsistency | **VALID (dup, low)** | Already reported. CVSS 3.1 |
  - | 5 | Challenge-response parameter-validation oracles | **VALID (dup, low)** | Already reported. CVSS 3.7 |

- 1 lead(s) marked VALID at 2026-08-10 05:03:45 UTC
  - | **VALID** | 4 | All duplicates of previously reported findings |

- 6 lead(s) marked VALID at 2026-08-10 06:51:01 UTC
  - | Q2 Reachable? | AUTH_HELPED | Returns 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | NO | Requires AUTH_HELPED with valid Work credentials + cross-subscription contact probes |
  - | Q5 Novel? | YES | Not previously triaged as valid vuln; only hypothesized (OpenAPI flags it "currently buggy" TWRK-1633) |
  - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven via passive GET/HEAD only. Retain for program-provided 
  - | **VALID (new)** | 0 | No new valid findings this cycle |
  - | **VALID (dup)** | 4 | #1 Directory IDOR, #2 Desktop key-storage ACL, #4 HSTS inconsistency, #5 Challenge oracles — all previously reported |

- 1 lead(s) marked VALID at 2026-08-10 11:00:50 UTC
  - | VALID (new) | 0 |

- 4 lead(s) marked VALID at 2026-08-10 13:54:42 UTC
  - | 1 | Directory cluster identity→pubkey bulk enumeration (fetch_bulk + CORS*) | **VALID** — Already reported | 5.3 Medium |
  - | 2 | Desktop Windows key-storage ACL bypass | **VALID** — Already reported | 5.5 Medium |
  - | 3 | Safe backup HSTS/Expect-CT header inconsistency | **VALID** — Already reported | 3.1 Low |
  - | 4 | Challenge-response parameter-validation oracles | **VALID** — Already reported | 3.7 Low |

- 4 lead(s) marked VALID at 2026-08-10 15:47:25 UTC
  - | Q3 | Real impact? | **NO** — 400 = route exists but requires valid `backupId:backupKey`. No unauthenticated data access. Consistent with Finding #3 family (safe-01 400 behavior) |
  - | Q2 | Reachable? | **YES** — but input is malformed (`...64hex` is not valid hex) |
  - | Q1 | In scope? | **NO** — scope.yml lists `work.threema.ch`, NOT `work.test.threema.ch`. Staging variants are out of scope per valid-bugs.md precedent |
  - | **VALID (new)** | **0** | All map to already-reported finding families or are expected behavior |

- 1 lead(s) marked VALID at 2026-08-10 16:43:38 UTC
  - | **VALID** | 0 | All map to already-reported findings |

- 31 lead(s) marked VALID at 2026-08-10 17:45:55 UTC
  - | **Q3 Real impact?** | ✅ YES — mass valid-ID + pubkey harvesting at ~10k IDs/req, silent omission oracle, zero rate limit |
  - | **Q4 Passive proof?** | ✅ YES — `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → 200 with only valid pubkey; 35× sequential POSTs all 200, zero 429; `GET /identity/ECHOECHO`→200,
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104, 111, 124, 131, 134, 136, 142, 150, 158, 163, 167, 170, 172, 182, 189, 195, 199, 204, 220; triaged as VA
  - | **Q7 Triager accept?** | ✅ YES — already accepted as finding #1 in valid-bugs.md |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 26, 74, 77-78, 92, 108, 121, 130, 133, 137, 143, 187, 192, 197, 203; triaged as VALID in `valid-bugs.md` (finding #3) |
  - | **Q2 Reachable?** | ⚠️ Route reachable; data access requires valid `backupId:backupKey` (64-hex id + high-entropy key) |
  - | **Q3 Real impact?** | ❌ NO without valid credentials — 400 vs 404 oracle reveals route existence only (already known from source/KB). CORS* lets an attacker page send credentialed cross-origin reque
  - | **Q4 Passive proof?** | ⚠️ Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 30-31, 62, 79, 101, 106, 125, 128-129, 132, 135, 138; triaged in valid-bugs.md |
  - | **Q6 Not always-rejected?** | ❌ **NO** — CORS `*` on a credential-gated endpoint (400) is best-practice/defense-in-depth only, per valid-bugs.md precedent (multiple cycles) |
  - | **Q7 Triager accept?** | ❌ NO — no data access without valid credentials |
  - | **Q3 Real impact?** | ⚠️ CONDITIONAL — valid hardening concern but constitutes a vulnerability ONLY when chained with a demonstrated RCE primitive |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED then formally REJECTED as standalone in KB lines 32, 63, 75, 81, 95, 115, 118, 123; triaged as HOLD/conditional in valid-bugs.md |
  - | **Q6 Not always-rejected?** | ⚠️ Borderline — valid hardening concern, not a standalone vuln |
  - | **Q1 In scope?** | ❌ **NO** — scope.yml lists `apip.threema.ch` (production); `.test` staging variants are **not** listed. Per valid-bugs.md precedent, staging variants treated as out-of-scope |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 28-29, 61, 68, 93; triaged in valid-bugs.md |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 30-31, 62, 79, 101, 106, 125, 128-129, 132, 135, 138, 144, 151, 177, 184, 190-191, 193, 201, 205, 221; triaged as VALID in valid-bugs.md |
  - | **Q7 Triager accept?** | ⚠️ LOW severity; accepted as valid but low |
  - | **Q2 Reachable?** | ⚠️ PARTIAL — route presence confirmed via GET returning 200 JSON errors, but challenge-response still requires valid identity+secret |
  - | **Q5 Novel?** | ❌ **NO** — ACCEPTED in KB lines 100, 105, 110, 112, 165, 176; triaged in valid-bugs.md |

- 2 lead(s) marked VALID at 2026-08-10 18:41:31 UTC
  - - **Verdict: HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Cannot be proven via passive GET/HEAD only. Retain for program-provide
  - | **VALID (new)** | **0** | No novel findings |

- 37 lead(s) marked VALID at 2026-08-10 20:30:46 UTC
  - **VALID (new, reportable):** 0
  - **VALID (duplicate, already reported):** 4 families
  - | Q3 Real impact? | ✅ YES — mass valid-ID + pubkey harvesting at ~10k IDs/req, silent-omission oracle, cross-origin readable → targeted phishing/recon at scale |
  - | Q4 Passive proof? | ✅ YES — `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → 200 with only valid pubkey; 35× sequential POSTs all 200, zero 429; `GET /identity/ECHOECHO`→200, `/i
  - | Q5 Novel? | ❌ **NO** — ACCEPTED in KB lines 20-21, 27, 59, 67, 80, 82, 85-86, 94, 99, 104+; VALID in valid-bugs.md since 2026-08-07 (finding #1) |
  - **Verdict: VALID (duplicate)** — Already reported. CVSS 3.1: **5.3 (AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N)**
  - | Q5 Novel? | ❌ **NO** — ACCEPTED in KB lines 26, 74, 77-78, 92, 108+; VALID in valid-bugs.md (finding #3) |
  - **Verdict: VALID (duplicate)** — Already reported. CVSS 3.1: **5.5 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N)**
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — route reachable; data access requires valid `backupId:backupKey` (64-hex + high-entropy key) |
  - | Q3 Real impact? | ❌ NO without valid credentials — 400 vs 404 oracle reveals route existence only (already known from source/KB). CORS* lets an attacker page send credentialed cross-origin requests,
  - | Q4 Passive proof? | ⚠️ Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | ❌ **NO** — ACCEPTED in KB lines 30-31, 62, 79, 101, 106+; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | ❌ **NO** — CORS* on a credential-gated endpoint (400) is best-practice/defense-in-depth only, per valid-bugs.md precedent |
  - | Q7 Triager accept? | ❌ NO — no data access without valid credentials |
  - | Q5 Novel? | ❌ **NO** — ACCEPTED in KB lines 30-31, 62, 79, 101, 106, 125+; triaged in valid-bugs.md |
  - | Q6 Not always-rejected? | ⚠️ Borderline — valid hardening concern, not a standalone vuln |
  - | Q7 Triager accept? | ⚠️ LOW severity; accepted as valid but low |
  - **Verdict: VALID (duplicate, low severity)** — Already reported. CVSS 3.1: **3.1 (AV:N/AC:H/PR:N/UI:R/S:U/C:L/I:N/A:N)**
  - | Q2 Reachable? | ⚠️ AUTH_HELPED — 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | ❌ NO — requires AUTH_HELPED with valid Work credentials + cross-subscription contact probes |

- 6 lead(s) marked VALID at 2026-08-10 21:30:20 UTC
  - | Q6 Not always-rejected | **YES** — this is not info disclosure of already-public data in the trivial sense; the *scale* (10k batch, no rate limit, cross-origin exfiltration) turns a public directory
  - ### Verdict: **VALID**
  - ### Verdict: **VALID** (with HUMAN_ONLY PoC gap)
  - | Q2 Attacker-reachable | **PARTIAL** — CORS enables credentialed cross-origin requests, but attacker still needs valid backupId:backupKey |
  - | 1 | Identity→pubkey mass enumeration (fetch_bulk, no rate limit, CORS *) | IDOR | **VALID** | 5.3 MED |
  - | 2 | Desktop Windows key-storage ACL bypass | MISCONFIG | **VALID** (HUMAN_ONLY PoC) | 8.1 HIGH |

- 21 lead(s) marked VALID at 2026-08-10 22:26:04 UTC
  - | Q3 Real impact? | **YES** — mass valid-ID + pubkey harvesting at ~10k IDs/request, no rate limit, cross-origin readable |
  - | Q4 Passive proof? | **YES** — `GET /identity/ECHOECHO`→200, `/identity/ZZZZZZZZ`→404; `POST /identity/fetch_bulk`→200 with only valid pubkey; 35× sequential POSTs all 200, zero 429 |
  - | Q5 Novel? | **NO** — already ACCEPTED as Finding #1 in `valid-bugs.md` since 2026-08-07; triaged in every prior cycle |
  - ### Verdict: **VALID (duplicate)**
  - | Q5 Novel? | **NO** — already ACCEPTED as Finding #2 in `valid-bugs.md` |
  - ### Verdict: **VALID (duplicate)**
  - | Q3 Real impact? | **NO** — credential-gated (400, not 200). No unauthenticated data access demonstrated. CORS `*` + Allow-Headers: Authorization only matters if attacker already has valid `backupId:
  - | Q4 Passive proof? | **PARTIAL** — can confirm CORS headers + 400; cannot confirm data access without valid creds |
  - | Q7 Triager accept? | **NO** — no data access without valid credentials |
  - | Q5 Novel? | **NO** — already ACCEPTED as Finding #4 in `valid-bugs.md` |
  - ### Verdict: **VALID (duplicate, low severity)**
  - | Q3 Real impact? | **LOW** — route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret |
  - | Q5 Novel? | **NO** — already ACCEPTED as Finding #5 in `valid-bugs.md` |
  - ### Verdict: **VALID (duplicate, low severity)**
  - | Q2 Reachable? | **AUTH_HELPED** — returns 401 on all paths; requires valid Work license |
  - | Q4 Passive proof? | **NO** — requires AUTH_HELPED with valid Work credentials |
  - | Q5 Novel? | **YES** — not previously triaged as valid vuln; only hypothesized (TWRK-1633) |
  - ### Verdict: **HOLD** — OpenAPI flags endpoint "currently buggy" (TWRK-1633) but requires AUTH_HELPED with valid Work test license. Retain for program-provided test credentials.
  - | Q2 Reachable? | **AUTH_HELPED** — passive only confirms CORS posture; data access requires valid `backupId:backupKey` |
  - | Q4 Passive proof? | **NO** — requires valid credentials |

- 34 lead(s) marked VALID at 2026-08-10 23:05:47 UTC
  - | Q3 | ✅ YES | Mass valid-ID + pubkey harvesting at ~10k IDs/req, silent-omission oracle, zero rate limit → targeted phishing/recon at scale |
  - | Q4 | ✅ YES | `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → 200 with only valid pubkey; 35+ sequential POSTs all 200, zero 429 |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 20-21, 27, 59, 67, 80+; triaged as VALID in `valid-bugs.md` since 2026-08-07 (Finding #1) |
  - | Q7 | ✅ YES | Already accepted as Finding #1 in valid-bugs.md |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 26, 74, 77-78, 92+; triaged as VALID in `valid-bugs.md` (Finding #3) |
  - | Q7 | ✅ YES | Already accepted as Finding #3 in valid-bugs.md |
  - | Q2 | ⚠️ PARTIAL | Route reachable but data access requires valid `backupId:backupKey` (64-hex + high-entropy key) |
  - | Q3 | ❌ NO | CORS* on a credential-gated endpoint (400, not 200) is defense-in-depth only. No unauthenticated data access demonstrated. Attacker still need valid credentials |
  - | Q4 | ⚠️ PARTIAL | Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 30-31, 62, 79, 101+; triaged in `valid-bugs.md` multiple times |
  - | Q6 | ❌ NO | CORS* on credential-gated endpoint is best-practice/defense-in-depth per `valid-bugs.md` precedent (multiple cycles) |
  - | Q7 | ❌ NO | No data access without valid credentials; CORS* alone is not a vulnerability |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 101, 106, 125+; triaged as VALID (low severity) in `valid-bugs.md` |
  - | Q7 | ⚠️ LOW | Accepted as valid but low severity |
  - | Q3 | ⚠️ LOW | Route-presence + parameter-validation oracle only; challenge-response still requires valid identity+secret. No auth bypass |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 100, 105, 110+; triaged as VALID (low severity) in `valid-bugs.md` |
  - | Q7 | ⚠️ LOW | Accepted as valid but low severity |
  - | Q5 | ❌ NO | ACCEPTED then formally REJECTED as standalone in KB lines 32, 63, 75+; triaged as HOLD/conditional in `valid-bugs.md` |
  - | Q1 | ❌ NO | scope.yml lists `apip.threema.ch` (production); `.test` staging variants are **NOT** listed. Per `valid-bugs.md` precedent, staging variants treated as out-of-scope |
  - | Q5 | ❌ NO | ACCEPTED in KB lines 28-29, 61, 68+; triaged in `valid-bugs.md` then deemed OUT OF SCOPE |

- 28 lead(s) marked VALID at 2026-08-11 02:35:01 UTC
  - | Q2 Reachable? | YES | Unauthenticated POST returns 200 with pubkeys for valid IDs |
  - | Q3 Real impact? | YES | Mass valid-ID + pubkey harvesting at ~10k IDs/req, silent-omission oracle, cross-origin readable → targeted phishing/recon at scale |
  - | Q4 Passive proof? | YES | `POST /identity/fetch_bulk {"identities":["ECHOECHO","ZZZZZZZZ"]}` → 200 with only valid pubkey; 35+ sequential POSTs all 200, zero 429; CORS ACAO:* confirmed |
  - | Q5 Novel? | **NO** | ACCEPTED in KB lines 20-21, 27, 59+; triaged as VALID in valid-bugs.md since 2026-08-07 (Finding #1) |
  - | Q6 Not always-rejected? | Borderline | The *scale* (10k batch, no rate limit, cross-origin) turns a public directory into mass enumeration; accepted as valid |
  - | Q5 Novel? | **NO** | ACCEPTED in KB; triaged as VALID in valid-bugs.md since 2026-08-07 (Finding #3) |
  - | Q2 Reachable? | Partial | Route reachable but data access requires valid `backupId:backupKey` (64-hex + high-entropy key) |
  - | Q3 Real impact? | NO without valid creds | 400 = credential-gated; CORS * + Allow-Headers: Authorization only matters if attacker already has valid credentials. 400-vs-404 oracle reveals route exist
  - | Q4 Passive proof? | Partial | Can confirm CORS headers + 400 response; cannot confirm data access without valid credentials |
  - | Q5 Novel? | NO | ACCEPTED in KB; triaged in valid-bugs.md multiple times |
  - | Q6 Not always-rejected? | **NO** | CORS * on a credential-gated endpoint (400) is best-practice/defense-in-depth only, per valid-bugs.md precedent |
  - | Q7 Triager accept? | NO | No data access without valid credentials |
  - | Q5 Novel? | NO | ACCEPTED in KB; triaged as VALID (low severity) in valid-bugs.md |
  - | Q6 Not always-rejected? | Borderline | Accepted as valid but low severity |
  - | Q2 Reachable? | Partial | Route presence confirmed (200 JSON errors), but challenge-response still requires valid identity+secret |
  - | Q5 Novel? | NO | ACCEPTED in KB; triaged as VALID (low severity) in valid-bugs.md |
  - | Q6 Not always-rejected? | Borderline | Accepted as valid but low severity |
  - | Q2 Reachable? | AUTH_HELPED | Returns 401 on all paths; requires valid Work test license |
  - | Q4 Passive proof? | NO | Requires AUTH_HELPED with valid Work license + cross-subscription contact probes |
  - | Q5 Novel? | Not triaged as valid | Only hypothesized; OpenAPI flags it "currently buggy" (TWRK-1633) |
