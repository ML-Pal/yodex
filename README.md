# Yodex

**A provider-agnostic coding agent.** One harness that drives Anthropic, OpenAI,
Google, and open-weight models — anything your gateway serves — through a single
API key, with cost-aware routing built in.

[![npm](https://img.shields.io/npm/v/%40mlpal%2Fyodex)](https://www.npmjs.com/package/@mlpal/yodex)

```bash
npm install -g @mlpal/yodex
```

> **About this repo:** Yodex's source is not published — the CLI ships via
> [npm](https://www.npmjs.com/package/@mlpal/yodex). This repo is the public
> home for the technical report, benchmarks, docs — and **your feature requests
> and bug reports**: [open an issue](../../issues/new/choose).

## Why Yodex

A coding agent is a *harness* — the loop of tool use, verification, context
management, and recovery — wrapped around a model. Leading agents weld a mature
harness to one vendor's models, so you pay that vendor's premium on every token
whether the task needs it or not.

Our hypothesis: **provider-agnosticism matters, but it is not a model-count
contest.** Models are retiring at a record pace — models that topped leaderboards
a year ago are being end-of-lifed today. The metric that matters is whether your
agent's *utility survives model churn*. Yodex speaks one wire format to an AI
gateway that translates to every provider, and references **stable cost tiers**
(`max / frontier / mid / cheap`) rather than model names — so a model retirement
is a server-side curation update, never a client migration, and the same task
can run on a premium model, a cheap model, or a per-task mix chosen by a router.

- **Model-agnostic by construction** — zero provider-specific code in the
  harness; pick any served model or tier per invocation (`--model claude-opus-5`,
  `--model cheap`, or the `mlpal` router tag, which always resolves to a current
  served model).
- **Cost-aware delegation** — a difficulty gate rates each sub-task and routes it
  down the gateway's cost ladder: mechanical work to cheap fast models, hard
  reasoning to frontier ones. Measured effect: **~10× cheaper sub-agents** on
  decomposable work ([write-up](benchmarks/vs-claude-code-opus5.md)).
- **A feedback loop that learns** — every delegation outcome feeds per-model
  routing scores back to the gateway, so catalog picks improve with usage.
- **Verification without premium tokens** — an edit-gated self-check, an
  optional adversarial verifier that runs on a *cheap* tier, and an anti-churn
  breaker.
- **Deterministic safety** — read/write/edit/shell tools pass a rule layer that
  runs before permission logic and is never delegated to an LLM.
- **Parallel and background work** — read-only tool batches and sub-agent
  fan-out run concurrently; background agents, shells, and monitors report into
  a live panel; multi-agent workflows are scriptable and resumable.
- **Cross-repo coordination** — sessions share one store, so an agent in one
  repository can discover agents in others and hand them self-contained work
  through a mailbox. No mainstream coding agent offers this.
- **Extensible** — MCP servers, skills, and Claude-Code-compatible plugin packs
  install as-is.

## Quickstart

Yodex talks to an MLPal Gateway — the managed one, or your own self-hosted box:

```bash
# Managed (default endpoint):
export YODEX_API_KEY=mlpal_sk_...        # create a key at https://mlpal.ai
yodex "explain this repo"

# Self-hosted (github.com/mlpalOld/mlpal-gateway — Apache-2.0, one docker compose):
export YODEX_GATEWAY_URL=http://localhost:8000
export YODEX_API_KEY=mlpal_sk_...        # a key minted in your gateway console
yodex "fix the failing test"
```

Both serve the identical API surface, so nothing else changes. On a small
self-hosted box the default model resolves availability-aware — a one-provider
box just works.

## The paper

📄 **[Decoupling the Harness from the Model](paper/yodex-harness-paper-v2.pdf)**
*(yodex team · MLPal Research)* — an empirical study under **same-model
isolation**: competing harnesses run on the identical served model, so any
difference is the harness. Covers the isolation protocol, official-oracle
SWE-bench grading with hidden tests, cost/latency analysis across four workload
tiers and multiple providers, and a transparent account of divergent patches —
including the one clean Yodex loss.

## Benchmarks

**From the paper** — same-model isolation on `claude-opus-4-8`, official
SWE-bench harness, hidden tests:

| Task tier | Yodex | Claude Code | Output tokens (CC/Y) | Wall clock (CC/Y) |
|---|---|---|---|---|
| Easy (18) | 18/18 | 18/18 | 1.49× | 1.63× |
| Hard (14) | 14/14 | 13/14 | 1.18× | 1.35× |
| Long (6) | 6/6 | 6/6 | 1.65× | 1.72× |
| **SWE-bench_Lite (15)** | **12/15 (80%)** | 11/15 (73%) | **1.76×** | **1.59×** |

On a shared GPT model, Yodex resolved 8/10 vs OpenAI Codex's 7/10 with 6.3×
fewer input tokens; the same harness on cheaper models held correctness while
cutting cost 7.7–10.1×, and a budget router matched the premium baseline at
3.08× lower cost.

**Follow-up runs on `claude-opus-5`:**

- [Harness head-to-head + delegation](benchmarks/vs-claude-code-opus5.md)
  (2026-07-28): ~1.6–1.8× cheaper on focused fixes at equal correctness; ~2×
  cheaper end-to-end on delegation-heavy work with catalog routing cutting
  sub-agent cost ~10×.
- [SWE-bench recheck](benchmarks/swe-recheck-opus5-2026-08-11.md) (2026-08-11):
  both agents resolve 2/2 sampled instances; the efficiency margins hold or
  widen (≈2× output tokens, ≈2× wall clock, ≈6× compute cost).

These are our own runs — each write-up carries its methodology and limitations,
including small N and the authors' conflict of interest. Read those before
quoting numbers.

## Roadmap & feedback

Feature requests and bug reports are welcome —
[open an issue](../../issues/new/choose). The terminal TUI is shipping today;
web and editor frontends consume the same engine stream and are next.

## License

© 2026 mlpal inc. All rights reserved — see [LICENSE.md](LICENSE.md). The
documentation, paper, and benchmark data in this repository may be shared
unmodified with attribution.
