# A2 WSL2 Installation and Execution Runbook

**Repository:** https://github.com/Samoolino/A2  
**Canonical terminal path:** `$HOME/same@Boanerges/Projects/A2`  
**Mode:** local/private network, public market data, read-only checks, simulation and paper trading first.

This runbook was authored against the official Hummingbot installation overview, Hummingbot API installation page, Condor repository instructions, Condor LLM integration documentation and Hummingbot exchange catalog. Re-check the version-matched docs before executing because upstream installation scripts and configuration schemas change.

## A. Architecture decision

- Condor is the AI harness/orchestrator.
- Hummingbot API is the REST execution/control surface.
- Hummingbot Client/engine provides exchange connectors and strategy execution.
- Gateway provides supported AMM/DEX middleware.
- PostgreSQL and EMQX are dependencies in the documented API deployment.
- Ollama or a supported cloud CLI/API is the model backend.
- A2 is the project workspace for plans, connector registry, risk logic, tests and evidence; it is not a fork of every upstream repository.

Do not clone upstream code into A2 unless an approved engineering decision requires vendoring. Keep upstream checkouts under `$HOME/same@Boanerges/Projects/.a2-upstream/`.

## B. Windows host preparation

Open PowerShell as an administrator only if installation is required.

```powershell
wsl --update
wsl --list --verbose
```

Confirm the chosen Ubuntu distribution reports version 2. If WSL/Ubuntu is missing, follow the current Microsoft WSL installation documentation. Install Docker Desktop using its official instructions, enable the WSL 2 engine and integrate the chosen Ubuntu distribution. Restart Docker Desktop after changing integration settings.

## C. Create or reuse the canonical folder

Open Ubuntu/WSL Terminal. First inspect, do not overwrite an existing checkout.

```bash
mkdir -p "$HOME/same@Boanerges/Projects"
cd "$HOME/same@Boanerges/Projects"
if [ -d A2/.git ]; then
  echo "Existing A2 clone found; preserving it."
  git -C A2 status --short --branch
else
  git clone https://github.com/Samoolino/A2.git A2
fi
cd "$HOME/same@Boanerges/Projects/A2"
git remote -v
git status --short --branch
```

Install basic utilities if needed:

```bash
sudo apt update
sudo apt install -y git git-lfs curl wget jq unzip make gcc g++ python3 python3-venv python3-pip build-essential openssl ca-certificates
docker version
docker compose version
```

If Docker commands fail, stop and fix Docker/WSL integration before installing the trading stack.

## D. Create the upstream install area

```bash
mkdir -p "$HOME/same@Boanerges/Projects/.a2-upstream"
cd "$HOME/same@Boanerges/Projects/.a2-upstream"
```

The official Condor/API deployment setup creates upstream checkouts and runtime configuration in the directory from which it is launched. Keep that separate from A2 to prevent third-party installers from modifying the project repository.

## E. Inspect the official installer before running

Official entry point: https://github.com/hummingbot/deploy/blob/main/setup.sh

Download and inspect the script before execution:

```bash
cd "$HOME/same@Boanerges/Projects/.a2-upstream"
curl -fsSLo setup.sh https://raw.githubusercontent.com/hummingbot/deploy/main/setup.sh
less setup.sh
grep -nE 'curl|wget|docker|git clone|\.env|PASSWORD|TOKEN|API_BIND|BROKER_' setup.sh
bash setup.sh --help || true
```

Review the current script and release notes. If the script does not support the desired pinned revision or its behavior is unclear, stop and resolve the version/installation method first. Do not assume the current `main` installer equals a tested immutable release.

For an integrated first-time setup, use the current official Condor setup flow and choose Local mode for initial validation. If Condor asks whether to install Hummingbot API on the same machine, choose yes only if this is the intended single-machine deployment. The documented flow can install the API stack (API, PostgreSQL and EMQX). Do not install the API a second time afterward.

For API-only deployment, the official documented entry point is:
```bash
bash setup.sh --hummingbot-api
```
Use this only when the downloaded script's current help confirms that flag. For the integrated Condor setup, launch the reviewed script without arguments and follow the interactive wizard as documented.

## F. Configure API and broker secrets

During the setup wizard, create unique, strong values for:
- `USERNAME` and `PASSWORD`: HTTP Basic Auth for Hummingbot API;
- `CONFIG_PASSWORD`: encryption of supported bot/exchange credential configuration;
- `BROKER_USERNAME` and `BROKER_PASSWORD`: EMQX/MQTT client authentication;
- `BROKER_DASHBOARD_PASSWORD`: EMQX dashboard login, where supported;
- optional Tailscale auth key for private networking.

Generate strong values locally using a password manager or a secure random generator. Do not reuse exchange passwords, model API keys or GitHub tokens. Do not paste secrets into Codex, issues, chat, source files or screenshots.

The API installer creates an `.env` file in the Hummingbot API checkout. Do not commit it. Use the deployment's documented setup/rotation commands; current documentation describes `make emqx-auth-reset` for broker credential reseeding after updating broker variables. Confirm the current version before running it.

## G. Deploy and verify API services

From the actual API checkout created by the installer (path may differ by version):

```bash
cd "$HOME/same@Boanerges/Projects/.a2-upstream/hummingbot-api"
make deploy
make doctor
docker compose ps
```

Run only commands that exist in the pinned checkout; inspect `make help`/Makefile if the folder or target differs.

Verify:
1. API, PostgreSQL and EMQX containers are healthy.
2. `make doctor` reports no critical issue.
3. API binds to loopback/private interface by default.
4. No unauthenticated API, database or MQTT port is exposed to the public internet.
5. API authentication works with a read-only health/status request.
6. Logs and terminal output contain no passwords or tokens.

API Swagger/OpenAPI is normally available at `http://127.0.0.1:8000/docs` and `http://127.0.0.1:8000/openapi.json` on a local install. Confirm from the installed version. Do not expose port 8000 publicly. For remote access, use the documented Tailscale/private networking setup rather than opening the API to the internet.

## H. Install Ollama and a model

Use the current official Ollama installation page for Linux and WSL compatibility: https://ollama.com/download/linux

After reviewing the install command and installation completes:

```bash
ollama --version
ollama pull qwen3:8b
ollama list
ollama show qwen3:8b
ollama run qwen3:8b
```

Recommended starting model: `qwen3:8b`, listed by Ollama at approximately 5.2 GB for its Q4_K_M build. Lower-resource fallback: `qwen3:4b`, approximately 2.5 GB. These are model-download sizes, not total RAM requirements. Inference also needs memory for runtime, context and concurrent services; assess actual system RAM/VRAM before choosing.

The official Condor LLM integration docs describe Ollama as a local provider and note that tool calling must be supported. Set the version-documented model selector (the current Condor guide documents `CONDOR_LIVE_BACKEND_MODEL=ollama:<model>` for the live-backend test) only in the appropriate Condor configuration, then run its documented doctor and live-backend checks. Confirm the exact environment/config key in the installed revision; do not assume a setting for a different release.

A successful `ollama run` verifies basic text inference only. It does not prove tool calling, Condor integration or safe trading behavior. Test those separately.

## I. Connect Condor to Hummingbot API

For a same-machine deployment, the documented API base URL is normally `http://localhost:8000`. For a different machine on the same Tailscale network, use the API's tailnet/MagicDNS name (commonly `http://hummingbot-api:8000` if that is the configured device name), not a public IP.

1. In Condor, use `/servers` to add the API URL and the same API username/password configured in the API `.env`.
2. Use `/agent` to select the installed model provider and start an agent only after read-only checks pass.
3. Run Condor's `make doctor`; confirm each configured API server is reachable and authenticating.
4. Run `make live-backend` only when you want the documented test prompt/tool-calling tick; it actually sends a model prompt.
5. Start with read-only tools and mock/paper mode. Do not enable live execution merely because the agent can connect.

## J. CEX API key creation and connection

For each exchange, open the official website directly and create a dedicated key for A2:
1. Enable MFA.
2. Create a key with read-only permissions first; never grant withdrawals.
3. Add IP restrictions when supported, using the actual public egress IP if applicable.
4. Confirm the account, product type (spot/perpetual), region and permissions support the connector.
5. Use Condor's documented `/keys` flow or the installed Dashboard/API credential interface. Verify the current command and required fields against the pinned Condor/API version.
6. Run authenticated read-only balance/account checks.
7. Only after separate review, enable trading permissions if needed. Do not use withdrawal permissions.
8. Record key alias, scopes, creation date and last validation only; never record the secret value.

Troubleshooting order: connector ID → key/secret/passphrase fields → key status and scopes → actual outbound IP → time synchronization → endpoint/region → account/product mode → rate limits and venue status. Do not solve errors by widening permissions blindly.

## K. DEX / Gateway connection

1. Determine whether the venue is a CLOB DEX connector or a Gateway AMM/router integration.
2. Verify the chain, network/RPC URL, chain ID and Gateway compatibility.
3. Use testnet or a local fork and a dedicated low-value development wallet.
4. Validate token/router addresses, decimals, spender/allowance, gas balance, slippage and route simulation.
5. Never enter seed phrases into chat or a model prompt.
6. Verify quotes and transaction simulation before any wallet signature.
7. Treat a real transaction as a separately approved action; a connector test must not send a live transaction.

## L. Complete adapter discovery and verification

Do not promise that every listed adapter works on the user's account. The official exchange catalogue includes multiple connector classes, but actual availability depends on installed revision, account/region, products, credentials, network and venue uptime.

Generate the inventory from:
- installed Hummingbot source connector registry;
- Hummingbot API's available connector endpoints/OpenAPI;
- Gateway's supported connector/chain registry for the installed version.

For every discovered adapter, record ID, revision, product type, auth requirements, symbol/market metadata, network requirements, rate limits, test time, evidence and qualification state. Run public data and mock tests first; private read tests only with user-created keys; sandbox order tests where supported. Mark production-approved only after separate review.

## M. Update procedure

Before updating Condor/API or model:
1. Save a sanitized evidence record and current versions.
2. Review official release notes and breaking changes.
3. Back up configuration/database using documented procedures; never commit backups containing secrets or personal data.
4. Stop bots and ensure no order is in an ambiguous state.
5. Use the official update mechanism for the installed revision (Condor docs describe an admin update flow; verify current behavior).
6. Run `make doctor`, API health checks, model tool-call tests and connector smoke tests.
7. Keep paper mode until the acceptance gates pass.
8. Roll back using documented version/config/database recovery steps if critical checks fail.

## N. Troubleshooting record

Every issue must include UTC time, component/version, sanitized error code, command, expected/actual result and remediation. Redact API keys, passphrases, authorization headers, wallet addresses where private, account balances and personally identifying information.

## Official documentation reviewed
- https://hummingbot.org/installation/
- https://hummingbot.org/hummingbot-api/installation/
- https://github.com/hummingbot/condor
- https://github.com/hummingbot/condor-docs/blob/main/llm-integration.mdx
- https://hummingbot.org/exchanges/
- https://ollama.com/library/qwen3
