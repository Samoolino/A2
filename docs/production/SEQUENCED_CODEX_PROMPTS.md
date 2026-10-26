# A2 Canonical Codex Production Prompt Sequence

**Run from:** `~/same@Boanerges/Projects/A2` in Ubuntu/WSL.  
**Repository:** https://github.com/Samoolino/A2

Run prompts in order. Do not start a later phase until the earlier phase's evidence-backed gate passes. Codex must inspect current repository state and upstream docs before changing files. The agent must distinguish commands it executed from commands it merely recommends.

## Global policy prompt — use for every phase

> You are the implementation engineer for A2. Read AGENTS.md and the relevant docs before making changes. Work from the existing checkout at `$HOME/same@Boanerges/Projects/A2`. Preserve unrelated changes. Follow current official Hummingbot, Hummingbot API, Gateway, Condor and model-provider documentation for the exact pinned revisions. Do not invent commands, environment variables, connector IDs, endpoints or compatibility. Do not put credentials in Git, logs, chat or prompts. Start with read-only, mock, simulation and paper modes. Do not place live orders, sign transactions, transfer assets, expose services publicly, deploy production or merge without explicit approval for that exact action. For every phase report files changed, exact commands actually executed, versions/SHAs, test results, failures, sanitized evidence paths, remaining risks and the next gate. If unable to execute, label the work NOT RUN rather than claiming success.

## P00 — GitHub and workspace discovery

> Inspect `gh auth status`, `gh repo view Samoolino/A2`, `git remote -v`, `git status --short --branch`, the default branch and recent commits. Confirm the canonical path. If the checkout exists, reuse it; do not clone over or reset it. Check existing uncommitted work before changes. Record repository URL, branch and commit, but never tokens. Gate: repository access and worktree state are unambiguous.

## P01 — WSL2 and Docker readiness

> Follow `docs/production/WSL2_RUNBOOK.md`. Verify WSL2 Ubuntu, Docker Desktop integration, Docker Engine, Compose and disk/memory availability. Install only missing prerequisites. Do not continue if Docker is unavailable or if the canonical path differs. Gate: reproducible version output and successful Docker health checks.

## P02 — Upstream source and compatibility

> Read the current official Hummingbot installation overview, API install docs, Gateway docs, Condor setup and LLM integration docs. Inspect the deploy installer before executing it. Select and pin compatible release tags/SHAs and image digests. Record evidence in UPSTREAM_VERSIONS.md; do not call a floating main branch production-pinned. Confirm whether integrated Condor setup will provision API/Postgres/EMQX so no duplicate stack is installed. Gate: compatibility matrix reviewed.

## P03 — API, PostgreSQL and EMQX

> Install the official documented API stack from the separate upstream install area, not inside the A2 repository. Generate unique API and config passwords and EMQX credentials via the supported setup flow. Keep services on loopback/private networking. Run documented deploy, doctor and health checks. Inspect `.env` safely without printing secret values. Verify Basic Auth, database health, broker authentication and ACLs. Gate: all services healthy and no public exposure.

## P04 — Credential lifecycle

> Implement or document a secret-safe credential inventory and diagnostics. Verify exact current Condor `/keys` and API schema from pinned docs/OpenAPI. Create CEX keys only through official exchange portals; start read-only and never enable withdrawals. Verify IP restrictions, account/region/product support and authenticated read checks. Treat DEX signing keys and RPC provider keys separately. Add rotation/revocation procedures. Gate: read-only checks pass and no secret appears in files/logs.

## P05 — Local model installation

> Follow MODEL_PROVIDER_MATRIX.md and the current Ollama install guide. Check host RAM/VRAM and free disk. Install Ollama, pull `qwen3:8b` if resources permit (fallback `qwen3:4b`), and record the resolved model digest. Test basic inference and then Condor tool calling separately. Run malformed output, prompt injection, timeout, model unavailable and no-tool authorization tests. Gate: tool-call contract passes while execution remains disabled.

## P06 — Condor installation and API integration

> Install Condor using the pinned official setup. Choose Local mode for initial tests. Reuse the existing API and dependency services; do not create duplicate API/Postgres/EMQX stacks. Configure the local model or a separately authenticated cloud CLI using version-matched documentation. Add the API via `/servers`; verify credentials and URL. Run `make doctor` and the documented live-backend check only when ready. Keep dashboard private. Gate: Condor authenticates to API and selected model passes read-only tool checks.

## P07 — Connector discovery and qualification

> Discover all connectors from the installed Hummingbot source, API schema and Gateway registry. Do not copy a list from a web page and claim every item works. Generate `connectors.json` and CONNECTOR_MATRIX.md with connector ID, product, venue/chain, revision, market types, credentials, region constraints, status, last-tested time and evidence. Run load tests, public market-data checks, auth reads where appropriate, mocks and sandbox tests. Mark each connector by evidence-backed qualification level. Gate: every discovered adapter has a status and no untested adapter is called verified.

## P08 — Strategy freedom within risk controls

> Implement strategies through supported Hummingbot interfaces and document extension points. Separate strategy proposals from execution. Add deterministic risk checks for market freshness, venue/asset allowlists, size, exposure, drawdown, fees, slippage, gas, liquidity, precision, idempotency, order reconciliation and emergency halt. Include cross-exchange, triangular, market making and indicator strategies only after simulation. Do not promise no-loss profits or assume cross-venue atomicity. Gate: unit, integration and adversarial tests pass.

## P09 — Backtest and paper trading

> Add CI for unit/integration tests, connector fixtures, strategy replay, secret scanning, dependency checks and container checks. Backtest with fees, slippage, latency, partial fills and failed transactions. Run paper trading on live public market data without submitting orders. Test restart/reconnect, stale data, partial fill, cancel failure, duplicate events, database recovery and kill switch. Gate: reproducible evidence, clear metrics and tested rollback.

## P10 — Production readiness review

> Produce a release-readiness report covering account/provider eligibility, jurisdiction, capital limits, key custody, exposure, monitoring, incident response, backup/restore, rollback, access controls and independent review. Keep live execution disabled. No merge, deployment, real order, transaction signature or fund transfer without explicit approval. Gate: documented sign-off; otherwise remain in paper mode.

## P11 — Update and drift checks

> Compare installed Condor/API/Gateway/model versions with official releases and compatibility constraints. Review release notes, back up state securely, stop active bots safely, apply the official update mechanism, then rerun doctor, health, model tool-call, connector and paper tests. Do not update all components blindly or delete database volumes. Gate: regression suite passes or rollback is executed.

## Definition of complete

A phase is complete only when:
- the commands and tests actually ran;
- the installed versions and hashes are recorded;
- secrets are absent from the repository and evidence;
- expected and actual results match;
- failures and limitations are explicit;
- the next phase prerequisites are satisfied.
