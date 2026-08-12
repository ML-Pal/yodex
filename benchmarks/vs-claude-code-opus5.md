# Yodex vs Claude Code on claude-opus-5

**Date:** 2026-07-28 · **Model:** claude-opus-5 in both harnesses · **Grading:**
hidden edge-case tests run after each agent finishes · **Cost:** measured
per-run; CU = MLPal compute units, ≈ $ at opus-5's rate card ($5/$25 per M
tokens).

Three experiments: a same-model harness head-to-head on focused bug-fixes, a
delegation test built to exercise Yodex's difficulty gate + catalog-routed
sub-agents, and one very hard single-file task.

## Experiment 1 — Harness head-to-head (5 tasks, N=3, 45 runs)

2 medium + 3 hard single-file bug-fixes. Conditions: Yodex with opus-5 pinned,
Yodex at defaults (opus-5 + full machinery), Claude Code at defaults (opus-5).

**Correctness: 45/45 — a dead heat.** Every condition solved every task at every
rep. opus-5 saturates correctness here, so it doesn't separate the harnesses.

| condition | resolved | turns | subagents | $/task | wall |
|---|---|---|---|---|---|
| Yodex @ opus-5 pinned | 15/15 | 4.5 | 0 | $0.153 | 34.0s |
| **Yodex @ default** | 15/15 | 4.4 | 0 | **$0.140** | **30.7s** |
| Claude Code @ default | 15/15 | 4.6 | 0 | $0.250 | 35.5s |

Findings:

- **Cost: Yodex ~1.6–1.8× cheaper than Claude Code on the same model**, same
  correctness. Traces show Claude Code quietly ran haiku alongside opus-5 and
  still cost ~2×.
- **Latency: Yodex ~1.16× faster.**
- **The delegation machinery stayed dormant — correctly.** Zero sub-agent spawns
  in all 45 Yodex runs; escalation is verifier-gated and off by default. On a
  single-file fix, opus-5 solves directly (Read → Edit → Bash → Bash), so Yodex
  adds no overhead. Good — but it means this task set doesn't exercise the new
  machinery. Hence Experiment 2.

## Experiment 2 — Delegation test (N=3, 9 runs)

A 4-module build (`semver`, `lru`, `intervals`, `rle`) with an explicit
instruction to delegate each module to its own sub-agent. Conditions: Yodex at
defaults (catalog-routed sub-agents), Yodex with opus-5 pinned (sub-agents
inherit opus-5 — no catalog routing), Claude Code at defaults.

**The machinery fires exactly as designed.** Yodex spawned 4–5 sub-agents per
run; the difficulty gate rated each sub-task and routed it through the gateway's
model catalog — complexity-0 sub-tasks to `gemini-2.5-flash-lite` (cheapest),
complexity-1 to `gemini-2.5-flash`. When pinned, sub-agents correctly inherited
opus-5. Each sub-agent completion also fires the routing feedback loop
(accepted/failed outcomes adjust per-model scores).

Complete cost accounting (main loop + all sub-agent sessions):

| condition | funcs correct | main-loop | sub-agents | **total** | wall |
|---|---|---|---|---|---|
| Yodex @ default (catalog) | 11/12 | 0.361 CU | **0.022 CU** (Gemini) | **0.383 CU** | 272s† |
| Yodex @ opus-5 (opus-5 subs) | 12/12 | 0.210 CU | 0.212 CU (opus-5) | 0.422 CU | 168s |
| Claude Code @ default | 11/12 | — | — | **$0.774** | 62s |

Findings:

- **Catalog routing cut sub-agent cost ~10× (0.212 → 0.022 CU)** by sending
  complexity-0/1 sub-tasks to Gemini instead of opus-5.
- **Total: Yodex ~2× cheaper than Claude Code** (~0.38–0.42 CU ≈ $ vs $0.774) at
  near-matched correctness.
- **The correctness/cost tradeoff is the feedback loop's job:** cheap Gemini
  sub-agents hit 11/12 vs opus-5 sub-agents' 12/12 — one Gemini module missed an
  edge case. That's exactly the signal the feedback loop captures (a failed
  outcome lowers that model's measured score for the task type, so routing
  avoids it there over time). Day-1 it's a small correctness cost for a 10×
  sub-agent saving; it self-corrects with usage.
- **Latency: Yodex is slower on delegation** (168–272s vs 62s). opus-5's slow
  decode on the main loop plus orchestration; Claude Code delegates faster.

† One Yodex-default run was a 567s / $0.63 outlier (a sub-agent/main-loop stall
near the step budget), inflating that row's cost and wall. Without it,
Yodex-default's total sits clearly below the pinned condition. The outlier is a
real tail-latency risk worth watching.

## Experiment 3 — One very hard task (N=3)

Implement full regular-expression matching (`.` and `*`, full-string) from spec
— 20 adversarial hidden cases, reference-validated. Both harnesses at defaults
on opus-5.

| condition | resolved | mean $ | mean wall | turns | subagents |
|---|---|---|---|---|---|
| Yodex @ default | **3/3** | **$0.173** | 37.8s | 4.7 | 0 |
| Claude Code @ default | 3/3 | $0.249 | 39.2s | 5.0 | 0 |

Even a very hard *single-function* task doesn't invite delegation — Yodex solved
it directly on opus-5. Delegation (and catalog routing) is triggered by
*decomposable* work, not raw difficulty.

## Bottom line

- On **focused single-file work**, opus-5 makes correctness a wash and Yodex is
  ~1.6–1.8× cheaper and slightly faster — the harness-efficiency edge, with the
  machinery correctly staying out of the way.
- On **delegation-heavy work**, the difficulty gate + catalog routing land ~2×
  cheaper than Claude Code at near-matched correctness, with cheap-model
  edge-case misses being exactly what the feedback loop corrects over time.
- **Open items:** delegation latency (slower than Claude Code), a rare long-run
  tail, and the cheap-sub-agent correctness gap shrinking as measured scores
  accrue.

## Limitations

Small N (3 reps per cell); tasks authored by us; graded by hidden tests, not
human review; both harnesses run by the Yodex authors — weight the conflict of
interest accordingly. Costs are as-billed through the gateway's per-response
cost envelope for Yodex and Claude Code's own reporting for Claude Code.
