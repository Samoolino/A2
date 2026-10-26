# A2 Production Runbook: Tailscale for Condor and Hummingbot API

**Repository:** https://github.com/Samoolino/A2  
**Canonical WSL workspace:** `$HOME/same@Boanerges/Projects/A2`  
**Upstream guide:** https://hummingbot.org/hummingbot-api/tailscale/  
**Security model:** private tailnet + Hummingbot API authentication + strong configuration encryption password. Tailscale does not replace API authentication.

## 1. When to use this

Enable Tailscale for production and for remote access to Hummingbot API. It is useful even when Condor and the API run on one machine, and especially when Condor runs on a laptop while the API runs on a VPS.

Do not expose API port 8000 to the public internet. PostgreSQL (normally 5432) and EMQX/MQTT must remain private and do not need to be reachable directly from other tailnet devices.

## 2. Prerequisites

- Hummingbot API / Condor deployment is compatible with the current official installer.
- Docker Engine and Docker Compose work in the target environment.
- You can administer the Tailscale tailnet.
- You have a password manager or secure way to generate and store unique passwords.
- You know whether Condor and the API are on the same machine or separate machines.

## 3. Create and configure the Tailscale account

1. Open https://tailscale.com and sign in or create an account.
2. Open the Tailscale admin console: https://login.tailscale.com/admin.
3. Open **DNS** and enable **MagicDNS**. This lets clients use the API hostname rather than a changing IP address.
4. Open **Settings → Keys**: https://login.tailscale.com/admin/settings/keys.
5. Generate an auth key. Use an ephemeral or single-use key when suitable for one-time device enrollment; if the same key must enroll multiple devices, follow Tailscale's current guidance for a reusable key and protect it carefully. Apply expiry and device-tag/ACL controls where supported.
6. Copy the auth key directly into the installer prompt. Never put it in Git, a prompt, an issue, a screenshot, shell history, or a committed environment file. If exposed, revoke it in the admin console and create a replacement.

Use the current Tailscale admin UI because available key options can change.

## 4. Review the Hummingbot installer before execution

From Ubuntu/WSL, use a separate upstream installation directory rather than running the installer inside A2:

```bash
mkdir -p "$HOME/same@Boanerges/Projects/.a2-upstream"
cd "$HOME/same@Boanerges/Projects/.a2-upstream"
curl -fsSLo setup.sh https://raw.githubusercontent.com/hummingbot/deploy/main/setup.sh
less setup.sh
```

Review the script and the version-matched official documentation before executing. The upstream installer changes over time; do not assume the moving `main` branch is an immutable production release.

For an API-only deployment, the current documented command is:

```bash
bash setup.sh --hummingbot-api
```

For the integrated Condor wizard, run the reviewed setup script without arguments and select the desired Condor mode. Choose to install Hummingbot API in the same deployment only if that is the intended architecture. Do not install the API stack twice.

## 5. Complete the interactive installer carefully

When prompted:

1. Choose Condor's **Local** mode for initial local testing, or **Telegram** only if you intend to operate it through a Telegram bot.
2. If installing Hummingbot API, enable Tailscale when asked: answer `y`.
3. Paste the Tailscale auth key into the installer prompt. Do not include it in any other command or configuration shared with Codex.
4. Set a unique, strong API `USERNAME` and `PASSWORD` for HTTP Basic Authentication.
5. Set a separate, unique `CONFIG_PASSWORD` to encrypt stored connector credentials and support Gateway mTLS configuration.
6. Set unique `BROKER_USERNAME` and `BROKER_PASSWORD` for EMQX/MQTT service authentication. Keep `BROKER_DASHBOARD_PASSWORD` distinct from the broker client password.
7. Use the default Tailscale hostname `hummingbot-api` unless you have a documented reason to change it.
8. Do not change `API_BIND` to `0.0.0.0`. Keep the default loopback bind unless the version-matched official guide explicitly requires a specific tailnet IP bind.
9. Finish setup and note the actual API checkout path. Installer versions may create files in the directory from which the script was launched.

Store passwords in a password manager. Do not paste populated `.env` values into Codex or terminal transcripts.

The deployment configuration typically includes `TAILSCALE_ENABLED=true`, `TAILSCALE_AUTH_KEY`, and `TAILSCALE_HOSTNAME=hummingbot-api`. Verify exact variable names and behavior against the installed API version.

## 6. Deploy and verify on the API host

Change into the API checkout created by the installer (the path may vary):

```bash
cd "$HOME/same@Boanerges/Projects/.a2-upstream/hummingbot-api"
make doctor
make tailscale-status
docker compose ps
```

If the installer completed configuration but services did not start, inspect the Makefile and official documentation for the installed version, then use its documented setup/deploy targets. The current guide uses `make setup` followed by `make deploy`.

Expected checks:

- API, PostgreSQL and EMQX containers are healthy.
- `make doctor` reports no critical configuration, exposure, authentication, or tailnet issue.
- `make tailscale-status` shows the `hummingbot-api` peer online and the expected Tailscale Serve forwarding for port 8000.
- API port 8000 is not exposed on the VPS public interface.
- PostgreSQL and EMQX remain loopback/private and are not exposed publicly.
- No credentials are printed in logs.

If the API is already installed and Tailscale must be enabled later, use the version-matched setup procedure to configure Tailscale, then redeploy using the documented targets. Do not manually edit Docker networking as a substitute for the official procedure.

## 7. Connect Condor or another tailnet device

On every client that needs remote access:

1. Install Tailscale using the official platform instructions.
2. Sign in to the same tailnet (or use the organization's approved device-sharing/access policy).
3. Confirm the device appears online in the admin console.
4. Confirm MagicDNS is enabled.
5. Configure Condor's `/servers` integration to use `http://hummingbot-api:8000` and the API username/password. On the API host itself, `http://localhost:8000` may be used for local checks.
6. Do not use a public IP or open the API firewall to make connectivity work.

To test from another authorized tailnet device without placing the password in shell history:

```bash
read -rp "Hummingbot API username: " HB_USER
read -rsp "Hummingbot API password: " HB_PASS
echo
curl --fail-with-body --user "$HB_USER:$HB_PASS" http://hummingbot-api:8000/
unset HB_USER HB_PASS
```

Use the exact hostname configured in `TAILSCALE_HOSTNAME` if it differs from `hummingbot-api`. An authenticated response confirms basic network and API reachability, not exchange-key validity or trading readiness.

## 8. Common failures and remediation

| Symptom | Diagnostic sequence |
|---|---|
| Hostname `hummingbot-api` does not resolve | Verify MagicDNS is enabled; verify both devices are on the same tailnet and online; inspect the actual configured hostname. |
| Auth key rejected | Check that it has not expired or been revoked; generate a replacement and enter it directly into the approved setup prompt. |
| Tailscale status is online but API cannot connect | Run `make tailscale-status` and `make doctor`; confirm the port 8000 Serve forward and the mounted serve configuration; redeploy through the documented command if required. |
| Connection refused | Verify API/container health, the API bind and the version-specific Serve forwarding setup. |
| HTTP 401 | Use the API `USERNAME` and `PASSWORD` from the installed API configuration, not an exchange key or EMQX credential. |
| API still reachable on public IP | Remove port 8000 from cloud firewall/security-group rules and check Docker port bindings and host firewall. Do not rely on Tailscale alone while a public listener remains. |
| EMQX authentication errors | Check broker service credentials and ACLs using the version-matched documented diagnostics. The broker password is not the API password. Rotate broker credentials through the supported procedure. |
| Still failing | Collect sanitized `make doctor`, `make tailscale-status` and container status output. Redact tokens, passwords, authorization headers and account data before sharing. |

## 9. Security invariants

- No public ingress for API port 8000.
- No public PostgreSQL or EMQX listener.
- Keep API authentication enabled; Tailscale is an additional network control.
- Keep API and configuration passwords unique and strong.
- Keep `BROKER_PASSWORD` distinct from `BROKER_DASHBOARD_PASSWORD`; the dashboard grants administrative broker access.
- Use least-privilege tailnet ACLs/device authorization where available.
- Rotate/revoke exposed auth keys and credentials immediately.
- Keep secrets out of repository, Codex prompts, logs, screenshots and evidence.
- Keep trading in read-only/paper mode until separate strategy, risk and production gates pass.
- Never treat a successful Tailscale connection as proof that an exchange connector or trading strategy is verified.

## 10. Official references

- Hummingbot Tailscale guide: https://hummingbot.org/hummingbot-api/tailscale/
- Hummingbot API installation: https://hummingbot.org/hummingbot-api/installation/
- Condor repository and install guide: https://github.com/hummingbot/condor
- Hummingbot deployment installer: https://github.com/hummingbot/deploy
- Tailscale admin console: https://login.tailscale.com/admin
- Tailscale keys: https://login.tailscale.com/admin/settings/keys
- Tailscale DNS / MagicDNS: https://login.tailscale.com/admin/dns
