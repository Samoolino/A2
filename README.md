# A2 — AI Trading Infrastructure

A2 is the integration and governance layer for an AI-assisted trading platform built around official Hummingbot components. The initial milestone is reproducible WSL2 setup, read-only connectivity validation, strategy simulation, and paper trading—not unattended live execution.

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

No checkbox is marked complete without evidence in `docs/production/evidence/`.

## Canonical WSL workspace

```text
~/same@Boanerges/Projects/A2
```

Keep upstream Hummingbot, API, Gateway and Condor checkouts outside this repository, for example under `~/same@Boanerges/Projects/.a2-upstream/`, unless an explicit vendoring decision is approved.

## Canonical production documentation

1. [WSL2 installation and execution runbook](docs/production/WSL2_RUNBOOK.md) — folder creation, upstream installer review, API/EMQX setup, model install, Condor integration, key onboarding and verification.
2. [Canonical Codex prompt sequence](docs/production/SEQUENCED_CODEX_PROMPTS.md) — staged implementation prompts P00–P11, each with an evidence-backed gate.
3. [Implementation plan](docs/production/IMPLEMENTATION_PLAN.md) — enterprise-grade phased execution and acceptance criteria.
4. [Tailscale production runbook](docs/production/TAILSCALE_RUNBOOK.md) — private networking, interactive installer settings, MagicDNS, API credentials, remote client verification and exposure checks.
4. [Model provider matrix](docs/production/MODEL_PROVIDER_MATRIX.md) — Ollama/Qwen3, LM Studio, Claude Code, Gemini CLI, GitHub Copilot CLI, Codex ACP and OpenRouter.
5. [Connector qualification matrix](docs/connectors/CONNECTOR_MATRIX.md) — CEX/CLOB DEX/Gateway DEX qualification states.
6. [Credential policy](docs/credentials/KEY_MANAGEMENT.md) — CEX keys, DEX signing keys, API authentication, EMQX and rotation.
7. [Upstream version register](docs/production/UPSTREAM_VERSIONS.md) — exact releases, SHAs, image/model digests and compatibility evidence.
8. [Repository instructions for agents](AGENTS.md) — persistent rules for Codex and other contributors.

## Implementation order

1. Verify GitHub identity, repository access, branch and worktree state.
2. Install WSL2 Ubuntu and Docker Desktop WSL integration; verify Git, Python and Docker.
3. Inspect official install scripts; select compatible pinned versions of Hummingbot, API, Gateway, Condor and model runtime.
4. Install the documented API stack and verify API, PostgreSQL and EMQX health.
5. Connect Hummingbot engine and Gateway without duplicating services.
6. Create least-privilege exchange credentials; start with read-only access.
7. Install Ollama and Qwen3 8B (about 5.2 GB model download) if host resources permit; use Qwen3 4B (about 2.5 GB) for a smaller option. Validate inference and tool calling separately.
8. Install Condor and connect it to the existing API and chosen provider.
9. Discover every connector from the installed runtime and qualify it with public-data, authenticated-read, simulation and sandbox evidence as applicable.
10. Implement strategy interfaces and deterministic risk authorization.
11. Run backtests, paper trading, CI, security checks, monitoring and recovery tests.
12. Only after independent review, consider explicitly authorized live execution.

## Hummingbot and Condor

- **Hummingbot** supplies trading engines/connectors and supported strategy execution.
- **Hummingbot API** exposes authenticated programmatic control and bot-management functions.
- **Gateway** provides middleware for supported AMM/DEX protocols.
- **Condor** is the AI harness/orchestration layer that can connect an agent/model to Hummingbot API.
- **PostgreSQL and EMQX** are provisioned by the documented API deployment; do not install duplicate copies without a reviewed reason.
- **Ollama/model provider** supplies inference; it is not the trading engine.

## Safety invariants

- Default to read-only and simulated trading.
- LLM output is a proposal, never execution authorization.
- A deterministic risk gate must approve every proposed execution.
- Never grant exchange withdrawal permissions.
- Never commit credentials, wallet keys, tokens, real account data or populated environment files.
- Never label a connector production-ready without timestamped evidence.
- Never claim guaranteed profits, zero-loss strategies, or cross-venue atomicity.
- No live order, wallet signing, production deployment, merge or irreversible action without the required explicit approval.

## Official upstream references

- Hummingbot installation overview: https://hummingbot.org/installation/
- Hummingbot API installation: https://hummingbot.org/hummingbot-api/installation/
- Hummingbot API Tailscale guide: https://hummingbot.org/hummingbot-api/tailscale/
- Exchange connector catalogue: https://hummingbot.org/exchanges/
- Hummingbot Client: https://github.com/hummingbot/hummingbot
- Condor: https://github.com/hummingbot/condor
- Condor LLM integration: https://github.com/hummingbot/condor-docs/blob/main/llm-integration.mdx
- Ollama Qwen3 model library: https://ollama.com/library/qwen3

Commands, paths, environment variables and connector availability must be re-checked against the exact selected release before execution. Documentation alone is not proof that an exchange or account is currently connectable.
