# Connector Qualification Matrix

Status: **Not yet discovered or runtime-tested**. This document is a template; it does not claim any venue is currently connectable.

## Status definitions
- `DISCOVERED`: adapter found in a pinned upstream revision.
- `LOADABLE`: module loads and initializes.
- `PUBLIC_DATA_OK`: public metadata and market-data tests pass.
- `AUTHENTICATED`: private read tests pass, where applicable.
- `SIMULATION_OK`: sizing, precision, fees and risk paths pass simulation.
- `SANDBOX_VALIDATED`: sandbox/testnet order lifecycle passes, where available.
- `PRODUCTION_APPROVED`: separate review and explicit approval recorded.

## Required registry fields
For every discovered adapter record:
- stable connector ID and product type (spot, perpetual, CLOB DEX, AMM/router);
- upstream repository and exact revision;
- venue/protocol, chain ID and supported symbols/markets;
- required credentials and scopes (never secret values);
- public/private endpoint and WebSocket support;
- regional/account/product limitations;
- rate limits, precision/minimum-notional metadata;
- current qualification state and last-tested UTC timestamp;
- test command, evidence path, sanitized failure details and reviewer.

## Qualification test matrix

| Test | CEX / CLOB | DEX / Gateway |
|---|---|---|
| Module loads | Required | Required |
| Public metadata and market data | Required | Required where supported |
| Authenticated read | Required for private account operations | Wallet/RPC read checks where applicable |
| Fees, precision and minimum size | Required | Token decimals, gas and route fees |
| Stream reconnect and stale-data handling | Required | RPC/quote freshness and chain health |
| Order lifecycle | Mock or sandbox first | Transaction simulation first |
| Negative tests | Rate limits, auth errors, partial fills, cancel failures | Wrong chain, stale quote, slippage, revert, insufficient gas |
| Live execution | Separate explicit approval only | Separate explicit approval only |

## Current results
No adapter is marked verified until results are generated from the actual pinned installation. Do not paste API keys, signed transactions, private account data or unredacted logs into this file.

## Next action
Run the P07 prompt in `../production/PROMPT_SEQUENCE.md` after P02–P06 gates pass. Generate a machine-readable `connectors.json` from the installed runtime rather than maintaining a hard-coded list as the source of truth.
