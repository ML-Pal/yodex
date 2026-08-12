# SWE-bench recheck on claude-opus-5 (2026-08-11)

A follow-up to the paper's SWE-bench_Lite results, rerun on `claude-opus-5` in
both harnesses. Same protocol as the paper: same-model isolation, hermetic
Docker instances, grading by the unmodified official SWE-bench harness with
maintainer-authored hidden tests. Two instances from the paper's divergent set;
Claude Code 2.1.227 at defaults.

## Result: 2/2 vs 2/2 — both agents resolve both

| Instance | Agent | Resolved | Wall | Turns | Tool calls | Output tok | Compute cost (CU) |
|---|---|---|---|---|---|---|---|
| pytest-5495 | Yodex | ✅ | 137s | 20 | 20 | 6,225 | 0.158 |
| pytest-5495 | Claude Code | ✅ | 297s | 56 | 33 | 11,952 | 0.926 |
| django-13315 | Yodex | ✅ | 215s | 16 | 15 | 6,954 | 0.138 |
| django-13315 | Claude Code | ✅ | 380s | 48 | 30 | 14,159 | 0.832 |

Efficiency ratios (Claude Code / Yodex): **output tokens 1.9× / 2.0×; wall
clock 2.2× / 1.8×; compute cost 5.9× / 6.0×.** The paper-era margins hold or
widen on the newer model.

Notable: Claude Code now also resolves django-13315 (it failed there on the
paper's model) — the newer model is stronger — and both agents produced the
byte-identical canonical patch on that instance, plus byte-identical source
edits on pytest-5495. At the correctness ceiling, the difference is what the
run costs.

## Where the extra spend goes (from full traces)

- Roughly half of Claude Code's extra output tokens are extended thinking
  (Yodex ships thinking off by default — a measured choice, not an omission).
- The rest: tests-first ordering that forces environment repair before any fix,
  extra full-suite re-runs, and metadata hygiene (issue lookups, docs greps,
  diff self-review).
- Claude Code's rigor is real — e.g. stash-verified pre-existing failures —
  but on these instances it changed confidence, not artifacts.

## Limitations

Two instances, one rep each — a spot-check that the paper's margins survive a
model generation, not a standalone benchmark. Runs performed by the Yodex
authors; weight the conflict of interest accordingly. Costs are pass-through
compute units as billed by the gateway (1 CU = $10), no markup.
