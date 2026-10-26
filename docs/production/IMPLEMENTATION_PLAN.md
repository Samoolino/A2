# A2 Canonical Production Plan — WSL2

**Target workspace:** `~/same@Boanerges/Projects/A2`  
**Initial operating mode:** public data, read-only account checks, mocks, simulation and paper trading  
**Live execution:** disabled until separately reviewed and explicitly authorized

## Phase 00 — Repository and access discovery

### Procedure
1. Confirm GitHub identity: `gh auth status`.
2. Confirm repository and default branch: `gh repo view Samoolino/A2 --json name,visibility,defaultBranchRef,url`.
3. Check current work: `git status --short --branch`; inspect remotes and recent commits.
4. Never initialize a second repository or overwrite existing files to resolve an access error.
5. Record repository URL, branch, commit and permissions (never tokens) in `docs/production/evidence/`.

### Gate
Repository is accessible; default branch and working state are known; no secret appears in logs.

## Phase 01 — WSL2 and workspace

### Windows PowerShell
If WSL is not already installed, follow Microsoft's current official WSL installation instructions. Then run:
```powershell
wsl --update
wsl --list --verbose
```
Confirm Ubuntu uses WSL 2. Install Docker Desktop separately if needed, enable the WSL2 backend and Ubuntu integration, and verify Microsoft's/Docker's current instructions before proceeding.

### Ubuntu / WSL
```bash
sudo apt update
sudo apt install -y git git-lfs curl wget jq unzip make gcc g++ python3 python3-venv python3-pip build-essential openssl ca-certificates
mkdir -p "$HOME/same@Boanerges/Projects"
cd "$HOME/same@Boanerges/Projects"
git clone https://github.com/Samoolino/A2.git A2
cd A2
git status --short --branch
docker version
docker compose version
```
If the repository is already cloned, do not clone over it; inspect and use the existing checkout.

### Gate
WSL2, Git, Python, Docker and Compose are functional. Repository path matches the requested canonical workspace.

## Phase 02 — Pin upstream components

Keep upstream code outside A2, for example:
```bash
mkdir -p "$HOME/same@Boanerges/Projects/.a2-upstream"
cd "$HOME/same@Boanerges/Projects/.a2-upstream"
```
Before installing, review the official repositories and their current installation documentation. Pin exact release tags or commit SHAs for:
- Hummingbot;
- Hummingbot API/deployment repository;
- Gateway (using the API-compatible version);
- Condor and its documentation;
- Ollama and selected model digest.

Record versions in `docs/production/UPSTREAM_VERSIONS.md`. Do not use an unpinned `main` branch as a production dependency. Verify installation scripts before executing them; do not pipe remote scripts directly into a shell.

### Gate
Compatibility matrix and immutable revision identifiers are recorded.

## Phase 03 — Hummingbot API infrastructure

Use the official Hummingbot API deployment procedure for the pinned revision. Its documented stack may include REST API, PostgreSQL and EMQX; do not install duplicate databases or MQTT brokers if the official compose stack already provisions them.

1. Review installer scripts and Compose manifests.
2. Set unique, strong API and configuration-encryption credentials through the documented setup flow.
3. Keep services bound to localhost/private network; do not expose API, database or MQTT ports publicly.
4. Run the upstream documented health/doctor commands.
5. Check container status and authenticated API health.
6. Record redacted output only.

### Gate
API, database and MQTT health checks pass; no public unauthenticated service is reachable.

## Phase 04 — Hummingbot engine and Gateway

Use API-managed bot and Gateway lifecycle where supported by the pinned release. Do not launch a second Gateway unless the compatibility documentation requires it.

1. Verify Hummingbot API ↔ bot lifecycle.
2. Verify Gateway health, configured chain IDs and RPC reachability.
3. Verify market metadata and public data without placing orders.
4. For development against Hummingbot source, use the upstream documented Conda/source workflow and pin the checkout.
5. Document version compatibility and recovery steps.

### Gate
Bot lifecycle, market-data smoke tests and Gateway read-only checks pass.

## Phase 05 — Credential onboarding and rotation

1. Create each exchange API key from the exchange's official account portal.
2. Start with read-only permissions; enable trading only after explicit review. Never enable withdrawals.
3. Configure IP restrictions where available. Understand the actual public egress IP of WSL/VPS before allowlisting.
4. Enter secrets only through the supported Hummingbot API/Dashboard/Condor credential flow after confirming the exact schema in the installed version.
5. Run authenticated read-only checks and record sanitized results.
6. For rotation: create replacement, validate, pause affected bots, switch credentials, confirm health, revoke old key, update inventory.
7. Treat blockchain signing keys separately from CEX API keys. Use a dedicated low-value development wallet and testnet before any live signing.

### Gate
All secrets are stored outside Git and logs; key scope, rotation owner and last validation are documented without secret values.

## Phase 06 — Local model and Condor

1. Install Ollama using the current official instructions after reviewing the installer.
2. Pull the chosen model (initial candidate: `qwen3:8b`; fallback for memory-constrained hosts: `qwen3:4b`).
3. Record model tag, digest, download size, host RAM/VRAM and inference settings.
4. Test inference and structured tool calling independently.
5. Install the pinned Condor release according to its official guide.
6. Use local mode initially; bind the UI to loopback and do not expose it publicly.
7. Point Condor at the existing Hummingbot API and explicitly select the model. Do not duplicate API/DB/EMQX services.
8. Test read-only tools, malformed model output, prompt injection, unavailable model, timeout and API failure cases.

### Gate
Model inference and supported tool-use tests pass; no order placement is available to the agent in the initial role.

## Phase 07 — Complete connector inventory

Discover connectors from the exact installed source and running Gateway registration, not a copied static list.

Create `docs/connectors/CONNECTOR_MATRIX.md` and machine-readable `connectors.json`. Include:
- upstream revision and connector ID;
- venue/protocol, chain, product type and symbols;
- public/private endpoints and auth fields;
- geographic/account restrictions;
- status, last-tested time and redacted evidence reference.

Qualification states:
1. DISCOVERED;
2. LOADABLE;
3. PUBLIC_DATA_OK;
4. AUTHENTICATED (where applicable);
5. SIMULATION_OK;
6. SANDBOX_OR_CONTROLLED_EXECUTION_VALIDATED;
7. PRODUCTION_APPROVED.

Test REST/WebSocket connectivity, timestamps, metadata, precision, fees, rate limits, reconnects, order lifecycle using mocks, and venue-specific failure cases. For DEX test chain ID, router/token addresses, decimals, gas, allowances, slippage, transaction simulation and finality. Do not run real transactions as a generic connector smoke test.

### Gate
Every discovered connector has an explicit status. No unsupported or untested connector is marked production-ready.

## Phase 08 — Strategy and risk framework

Implement custom strategies through supported Hummingbot scripts, Strategy V2 controllers, executors and market-data interfaces.

Initial strategy modules may include cross-exchange research, triangular route evaluation, market making, MACD/EMA/RSI, VWAP/TWAP, DCA/grid simulation, liquidity monitoring and CEX/DEX route analysis. Flashloan execution is a separately reviewed on-chain module and must not be assumed atomic across venues.

Every proposal must include venue, pair, data timestamp, side, size, order type, estimated fees/slippage/gas, expiry, expected net P&L and strategy version.

A deterministic risk service must validate:
- allowed venues/assets and maximum order size;
- per-venue and total exposure;
- daily loss/drawdown limits and stop conditions;
- market-data freshness and liquidity;
- price bands, precision, minimum notional and rate limits;
- estimated fees, slippage, gas and route failure risk;
- duplicate order/idempotency and reconciliation state;
- emergency halt and manual approval policy.

The LLM may propose research or strategy parameters but must not access raw secrets, change its own risk limits or authorize execution.

### Gate
Unit tests, deterministic simulations, negative tests and independent risk-gate tests pass.

## Phase 09 — Backtest, paper trade and CI

1. Separate development, integration, paper and production configurations and credentials.
2. Add unit tests, connector contract tests with fixtures, strategy event-replay tests, secret scanning, dependency auditing and container checks.
3. Backtest with out-of-sample periods, realistic fees, slippage, latency assumptions and failed-fill scenarios.
4. Paper trade against live public market data without submitting live orders.
5. Test restarts, reconnects, duplicate events, partial fills, cancel failures, stale data, database recovery and kill switch.
6. Store evidence and sanitized metrics; never publish private account data.

### Gate
Critical tests pass, failure behavior is understood, paper-trading metrics are reproducible and rollback is tested.

## Phase 10 — Controlled release

Production requires a human approval record covering account permissions, jurisdiction/provider eligibility, capital limits, monitoring, alert routing, key recovery, backup/restore, rollback and incident response.

Use a separate least-privilege trading key; withdrawals remain disabled. Start with a narrowly bounded exposure and human-supervised execution. Do not automatically promote a strategy from paper to live.

### Gate
Independent technical, security and operational sign-off is recorded. If any critical gate fails, remain in paper mode.

## Evidence record template

For every phase, save a dated Markdown record under `docs/production/evidence/` containing:
- phase and timestamp (UTC);
- OS/runtime and upstream SHAs;
- commands run (with secrets removed);
- checks and exact pass/fail results;
- sanitized logs or artifact references;
- known limitations and remediation;
- reviewer and next gate decision.
