# A2 — AI Trading Infrastructure

A2 is the integration and governance layer for a modular AI-assisted trading platform built around official Hummingbot components. The initial milestone is reproducible WSL2 development, read-only connectivity validation, strategy simulation, and paper trading—not unattended live execution.

## Status

- [x] Repository foundation and implementation gates defined
- [ ] WSL2/Docker environment installed and verified
- [ ] Hummingbot API, PostgreSQL and EMQX health verified
- [ ] Hummingbot/Gateway upstream versions pinned
- [ ] Condor + local Ollama model validated
- [ ] CEX/DEX connector inventory generated from installed versions
- [ ] Connector tests and strategy simulation passing
- [ ] Paper trading and operational controls accepted
- [ ] Production deployment independently approved

No checkbox is marked complete without recorded evidence in `docs/production/evidence/`.

## Canonical WSL workspace

```text
~/same@Boanerges/Projects/A2
```

Keep upstream Hummingbot and Condor checkouts outside this repository, for example under `~/same@Boanerges/Projects/.a2-upstream/`, unless an explicit vendoring decision is approved.

## Implementation order

1. Verify GitHub identity, repository access and branch state.
2. Install WSL2 Ubuntu, Docker Desktop WSL integration, Git and Python tooling.
3. Review and pin upstream Hummingbot API, Hummingbot, Gateway and Condor revisions.
4. Install the official Hummingbot API stack and verify API, PostgreSQL and EMQX health.
5. Connect Hummingbot engine and Gateway without duplicating services.
6. Create exchange credentials with least privilege; begin with read-only access.
7. Install Ollama and a selected local model; validate inference and tool calling.
8. Install Condor in local mode and connect it to the existing API.
9. Discover every connector from the pinned runtime and qualify it through evidence-based test tiers.
10. Implement A2 strategy contracts, risk authorization, simulation and paper trading.
11. Add CI, secret scanning, monitoring, recovery and release gates.
12. Only after independent review, consider explicitly authorized live execution.

See [the implementation plan](docs/production/IMPLEMENTATION_PLAN.md), [credential policy](docs/credentials/KEY_MANAGEMENT.md), and [repository instructions](AGENTS.md).

## Safety invariants

- Default to read-only and simulated trading.
- LLM output is a proposal, never an authorization to trade.
- A deterministic risk gate must approve every proposed execution.
- No withdrawal permissions for exchange API keys.
- Never commit credentials, wallet keys, tokens, real account data or populated environment files.
- Never label a connector production-ready without a timestamped test record.
- Never claim guaranteed profits, zero-loss strategies, or cross-venue atomicity.
- No live order, wallet signing, production deployment, merge or irreversible action without the required explicit approval.

## Upstream references

- Hummingbot: https://github.com/hummingbot/hummingbot
- Hummingbot API: https://hummingbot.org/hummingbot-api/installation/
- Gateway: https://hummingbot.org/gateway/installation/
- Condor: https://github.com/hummingbot/condor
- Condor LLM integration: https://github.com/hummingbot/condor-docs/blob/main/llm-integration.mdx
- Ollama Qwen3: https://ollama.com/library/qwen3

Upstream URLs and commands must be re-checked against the selected release before execution.