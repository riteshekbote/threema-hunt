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
