# A2 Codex Prompt Sequence

Run one phase at a time from the repository root. A phase is complete only when its gate is evidenced. Stop on failed prerequisites. Do not let an agent claim work was executed if it only wrote a plan.

## Global instruction
Read `AGENTS.md` and `docs/production/IMPLEMENTATION_PLAN.md` first. Inspect existing files before editing. Preserve existing work. Never expose secrets. Default to read-only, mock, simulation and paper modes. Do not place live orders, sign transactions, withdraw assets, deploy production, or merge without explicit authorization. Report files, commands, test results, failures and evidence.

## P00 — Repository audit
"Inspect the current A2 checkout and GitHub state. Verify branch, working tree, remotes, repository access, current files and recent commits. Do not initialize a duplicate repo, overwrite files or change branches while other writers may be active. Record only sanitized evidence. Stop and report if access or branch state is ambiguous."

## P01 — WSL2 environment
"Validate WSL2 Ubuntu, Git, Python, Docker and Compose against official current documentation. Confirm the canonical path is `~/same@Boanerges/Projects/A2`. Do not install software blindly; identify missing prerequisites and give exact commands. Verify the workspace and Docker health. Do not touch secrets."

## P02 — Upstream pinning
"Review official Hummingbot, Hummingbot API/deploy, Gateway, Condor and Ollama docs. Select mutually compatible tagged versions/SHAs and image digests. Verify every install command and configuration path against that revision. Write `docs/production/UPSTREAM_VERSIONS.md` and a compatibility matrix. Do not install from floating main branches or execute unreviewed remote scripts."

## P03 — API infrastructure
"Set up the official version-matched Hummingbot API stack with its supported PostgreSQL and EMQX dependencies. Do not duplicate services. Verify health checks and authenticated read-only API access. Bind to localhost/private networking and document recovery. Do not add exchange secrets or enable trading."

## P04 — Credential lifecycle
"Implement a secret-safe credential inventory and read-only connection tester. Verify exact connector schemas from installed OpenAPI/docs. Support least privilege, rotation, revocation and redacted diagnostics. Never store secret values in repository, logs or prompts. Never request withdrawals or broaden scopes automatically."

## P05 — Local model
"Install Ollama using reviewed official instructions, pull the selected Qwen3 model and record its digest and host resource profile. Verify inference and structured tool calling, timeout and failure behavior. Do not infer model size from RAM footprint. No model tool may place orders at this stage."

## P06 — Condor
"Install pinned Condor in local mode and connect it to the existing Hummingbot API. Verify health and model integration. Do not duplicate API, PostgreSQL or EMQX. Keep the UI on loopback; do not expose unauthenticated local mode. Test read-only tools, prompt injection and malformed model output."

## P07 — Connector catalog
"Discover all spot, perpetual, CLOB DEX and Gateway adapters in the pinned source and running installation. Generate a machine-readable registry and a Markdown matrix with capability, auth requirements, constraints, test status, timestamp and sanitized evidence. Test every discovered connector using public data and mocks; do not call it production verified without explicit evidence. No real transactions in generic smoke tests."

## P08 — Strategy and risk
"Implement strategy modules through supported Hummingbot interfaces. Add deterministic risk authorization independent of the LLM: exposure, order-size, drawdown, freshness, liquidity, precision, fee/gas/slippage, idempotency, reconciliation and kill-switch checks. Build tests and simulations before paper trading. Do not claim guaranteed profits or cross-venue atomicity."

## P09 — Verification and CI
"Add unit, integration, connector contract, strategy replay, secret scan, dependency audit and container checks. Separate dev, integration, paper and production credentials. Test restart, reconnect, partial fills, cancel failure, stale data and kill switch. Record exact test outcomes and fix failures rather than hiding them."

## P10 — Release review
"Prepare a release-readiness report with evidence, known risks, rollback, monitoring, key recovery, backups, account eligibility and independent sign-off. Keep live trading disabled. Do not merge, deploy, sign transactions or place live orders unless the user explicitly authorizes that exact action."
