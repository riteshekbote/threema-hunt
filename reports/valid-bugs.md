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
