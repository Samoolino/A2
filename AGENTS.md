# A2 repository instructions

## Mission
Build an auditable AI-assisted trading platform using official Hummingbot components. Initial scope is reproducible WSL2 setup, read-only connectivity, connector qualification, strategy simulation and paper trading.

## Canonical workspace
`~/same@Boanerges/Projects/A2`

## Required execution order
Follow `docs/production/IMPLEMENTATION_PLAN.md` in order. Do not skip a gate because a later component appears available.

## Rules for every change
1. Inspect repository state and existing conventions before editing.
2. Keep Hummingbot, Gateway, Condor and their official deployment repositories as separately pinned upstream dependencies unless a reviewed ADR explicitly approves vendoring.
3. Verify upstream commands, paths, APIs and configuration schemas against the pinned revision. Do not invent endpoints or assume old Gateway directory layouts.
4. Keep changes narrow, testable, documented and reversible.
5. Never commit credentials, private keys, seed phrases, tokens, populated `.env` files, personal account data or exchange responses containing sensitive data.
6. Use least-privilege exchange credentials. No withdrawal permissions. Start with public market data and read-only account checks.
7. LLM output is untrusted strategy input. It must not directly authorize an order, access raw credentials, change risk limits or bypass the deterministic A2 risk gate.
8. Default to mock execution, simulation and paper trading. Do not submit live orders, sign transactions, move funds, merge to production or deploy externally without explicit authorization.
9. Never claim guaranteed profits, a zero-loss strategy, or atomicity across independent venues.
10. Do not label a connector connected, verified or production-ready without reproducible evidence and a timestamp.
11. Never force-push, overwrite unrelated work, perform destructive cleanup, or broadly stage changes while other tasks may be active.
12. Do not mark a phase complete unless its acceptance criteria and evidence are recorded.

## Required implementation report
For each phase report:
- files changed;
- upstream versions/commit SHAs;
- exact commands run;
- checks and outcomes;
- known failures and limitations;
- evidence paths;
- next phase prerequisites.

## Definition of done
The change has focused tests, updated documentation, secret-safe configuration, clean diff checks, and an explicit statement of what was and was not verified.
