# Yodex

**A provider-agnostic coding agent.** One harness that drives Anthropic, OpenAI,
Google — any model your gateway serves — through a single API key, with
cost-aware sub-agent routing built in.

```bash
npm install -g @mlpal/yodex
```

[![npm](https://img.shields.io/npm/v/%40mlpal%2Fyodex)](https://www.npmjs.com/package/@mlpal/yodex)

> **About this repo:** Yodex's engine source is not public (yet). This repo is
> the public home for the harness paper, benchmarks, docs — and **your feature
> requests and bug reports**: [open an issue](../../issues).

## Why Yodex

Most coding agents are welded to one provider. Yodex speaks one wire format
(Anthropic Messages) to an AI gateway, which translates to every provider — so
the harness carries **zero provider-specific code**, and new models light up
with no client change. Model retirements don't break your setup either: router
tags like `mlpal` always resolve to a current model.

- **Model-agnostic by construction** — pick any served model per invocation
  (`--model gpt-5.2`, `--model gemini-3.5-flash`, or the `mlpal` router tag).
- **Cost-aware delegation** — a difficulty gate rates each sub-task and routes
  it through the gateway's model catalog: trivial modules go to cheap fast
  models, hard ones to frontier models. Measured effect: **~10× cheaper
  sub-agents** on decomposable work ([benchmarks](benchmarks/vs-claude-code-opus5.md)).
- **A feedback loop that learns** — every sub-agent outcome (accepted/failed)
  feeds back into per-model routing scores, so the catalog's picks improve with
  your usage.
- **Production harness features** — permission engine, lifecycle hooks
  (including a Stop-hook verifier loop), MCP server support, parallel read-only
  tools, auto-compaction, structured logs with correlation IDs.

## Quickstart

Yodex talks to an MLPal Gateway — the managed one, or your own self-hosted box:

```bash
# Managed (default endpoint):
export YODEX_API_KEY=mlpal_sk_...
yodex "explain this repo"

# Self-hosted (github.com/mlpalOld/mlpal-gateway — Apache-2.0, one docker compose):
export YODEX_GATEWAY_URL=http://localhost:8000
export YODEX_API_KEY=mlpal_sk_...   # a key minted in your gateway console
yodex --model claude-opus-5 "fix the failing test"
```

## The paper

**[Yodex: a model-agnostic coding harness](paper/yodex-harness-paper-v2.pdf)** —
design of the headless engine (agentic loop, unified store, permission engine,
`SDKMessage` stream that terminal/web/IDE frontends all consume), the difficulty
gate, catalog-routed delegation, and the routing feedback loop.

## Benchmarks

Head-to-head vs Claude Code, both running `claude-opus-5`
(full write-up with methodology and caveats:
[benchmarks/vs-claude-code-opus5.md](benchmarks/vs-claude-code-opus5.md)):

| Workload | Correctness | Cost | Wall clock |
|---|---|---|---|
| Focused bug-fixes (5 tasks, 45 runs) | 15/15 both | **Yodex ~1.6–1.8× cheaper** | Yodex ~1.16× faster |
| Delegation-heavy 4-module build | near-matched (11–12/12) | **Yodex ~2× cheaper** (sub-agents ~10× via catalog routing) | Claude Code faster |
| One very hard task (regex engine from spec) | 3/3 both | **Yodex 1.44× cheaper** | ~equal |

These are our own runs at small N — read the write-up's limitations section
before quoting them.

## Roadmap & feedback

Feature requests and bug reports are welcome — [open an issue](../../issues).
Terminal TUI is shipping today; web and VS Code frontends consume the same
`SDKMessage` stream and are next.

## License

Docs, paper, and benchmark data in this repo: Apache-2.0. The `@mlpal/yodex`
binary is distributed under its npm license.
