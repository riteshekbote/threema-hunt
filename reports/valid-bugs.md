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
