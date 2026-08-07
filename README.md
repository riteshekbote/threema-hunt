# threema-hunt

24/7 multi-model bug-hunting automation for **Threema**.

Scope: see `scope.yml` (in-scope assets, exclusions, rules).

How it works:

- 5 opencode models (Big Pickle, Nemotron 3 Ultra, Longcat, Ling 3.0, Laguna) hunt in parallel every **10 minutes**, rotating targets
- Deep public-repo scan of the `threema/` GitHub org(s) referenced in `scope.yml` every **30 minutes**
- Triager job validates every lead with a second model + live passive probe, keeping a running count in `reports/valid-bugs.md`
- All testing is **passive and read-only** (GET/HEAD, ≤1 rps, no account creation, no data modification) per program rules
- Secrets are committed only as hashes; findings must be reported through the program channel referenced in `scope.yml`

| Artifact | Purpose |
|---|---|
| `research/` | Per-model research journals |
| `leads/` | Candidate findings (UNVALIDATED) |
| `triage/` | Validation verdicts |
| `reports/valid-bugs.md` | Validated findings + running count |
| `scope.yml` | Program scope and rules (edit to adjust) |
