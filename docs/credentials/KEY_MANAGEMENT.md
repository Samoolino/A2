# A2 Key Management and Connectivity Policy

## Credential classes

| Class | Purpose | Storage/handling |
|---|---|---|
| GitHub authentication | Clone/push repository | Git credential manager or `gh auth login`; never commit tokens |
| Hummingbot API credentials | Authenticate REST requests | Protected deployment secret store; never print in logs |
| Hummingbot configuration password | Encrypt supported credential configuration | Store separately from encrypted config and restrict access |
| CEX API key/secret/passphrase | Private exchange endpoints | Enter through supported UI/API flow; start read-only; withdrawals disabled |
| DEX signing key/seed | Authorize blockchain transactions | Dedicated wallet/key manager; never in prompts, Git, shell history or ordinary logs |
| RPC provider key | Blockchain RPC access | Secret store; scope and rate-limit where supported |
| EMQX credentials | MQTT broker client authentication | Dedicated service identities with least privilege |
| Telegram token | Optional Condor control interface | Secret store; restrict authorized Telegram user IDs |
| Tailscale auth key | Private networking | Use short-lived/reusable policy appropriate to device enrollment; do not publish |
| Model provider credentials | Optional cloud model APIs | Separate from exchange credentials; local Ollama generally needs no paid provider key |

## CEX key creation checklist

1. Open the exchange's official site/app directly and confirm the domain.
2. Enable MFA and account recovery safeguards.
3. Create a dedicated key labelled for A2 and the environment.
4. Start with read-only account/balance permissions. Public market data should not require a private key where the connector supports public endpoints.
5. Never enable withdrawal permissions. Enable trading only after a separate reviewed approval.
6. Use IP allowlisting if supported. Determine the actual public egress IP; a WSL private address is not the public address seen by the exchange.
7. Enter credentials using the installed connector's documented interface and exact field schema.
8. Verify authenticated reads, clock synchronization, account/product mode and permissions.
9. Store metadata only: venue, account alias, permission set, creation/rotation date, last test and revocation state. Never store secret values in the registry.

## API schema verification

Before adding credentials, inspect the installed Hummingbot API's OpenAPI/Swagger schema or official version-matched docs. Do not assume request fields or query/body placement from a different release. Prefer the supported Dashboard, API or Condor `/keys` flow. Use a disposable test account if available.

## Rotation and incident response

Normal rotation:
1. Create a replacement key with the same minimal required scopes.
2. Validate the new key using read-only checks.
3. Pause impacted bots.
4. Update the credential through the supported interface.
5. Verify recovery and record a sanitized test.
6. Revoke the old key and update metadata.

Suspected compromise:
1. Revoke the compromised CEX key immediately from the venue portal.
2. Halt affected bots and disable the relevant A2 execution path.
3. Review venue audit logs and A2 access logs.
4. Rotate dependent secrets and invalidate exposed sessions.
5. Check balances/orders/withdrawal history and escalate to the venue as needed.
6. Preserve redacted incident evidence; never paste the compromised secret into an issue or chat.

## DEX signing keys

- Use testnet or local fork for development.
- Use a separate development wallet with no material funds.
- Verify chain ID, token/router addresses, contract code, spender, allowance, gas and simulation output before signing.
- Do not request seed phrases or private keys in chat or Codex.
- Prefer a dedicated signer service or hardware-backed wallet for approved production operations.
- CEX key revocation does not revoke on-chain allowances; revoke approvals separately when needed.

## EMQX / MQTT

EMQX is a broker, not an exchange key. Use unique service credentials, TLS where traffic leaves a trusted local boundary, explicit topic ACLs, private network bindings, no anonymous public access, rotation, health checks and redacted logs. Use the broker instance provisioned by the compatible Hummingbot API stack unless a documented architecture requires a separate one.

## Credential connectivity diagnostics

Check in order:
1. Correct venue and connector ID.
2. API key/secret/passphrase fields match the installed connector.
3. Key is active and permissions are correct.
4. IP allowlist matches actual outbound public IP.
5. Local system time is synchronized.
6. Endpoint, network and API region are correct.
7. Account mode, market type and product permissions are supported.
8. Rate limits, nonce/timestamp requirements and venue status.
9. Logs contain error codes only, never secret values.

Never solve a connectivity error by granting broader permissions without reviewing the cause.
