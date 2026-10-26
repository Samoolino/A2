# Upstream Version Register

Status: **Not yet installed or runtime-verified**. Populate this file only after inspecting official documentation and actual installed artifacts.

| Component | Official source | Selected release/tag | Commit SHA or image digest | Compatibility evidence | Last verified UTC |
|---|---|---|---|---|---|
| Hummingbot Client/source | https://github.com/hummingbot/hummingbot | TBD | TBD | TBD | TBD |
| Hummingbot API/deploy | https://hummingbot.org/hummingbot-api/installation/ | TBD | TBD | TBD | TBD |
| Gateway | https://hummingbot.org/gateway/installation/ | TBD | TBD | TBD | TBD |
| Condor | https://github.com/hummingbot/condor | TBD | TBD | TBD | TBD |
| Condor documentation | https://github.com/hummingbot/condor-docs | TBD | TBD | TBD | TBD |
| Ollama | https://ollama.com/ | TBD | TBD | TBD | TBD |
| Qwen3 model | https://ollama.com/library/qwen3 | TBD | Model digest TBD | Tool-use test TBD | TBD |

## Pinning policy
- Do not use floating tags such as `latest` for production.
- Record immutable Git commit SHAs and container image digests.
- Verify the API, Gateway and Condor versions are mutually compatible before deployment.
- Model tag alone is not sufficient for reproducibility; record the resolved digest and runtime options.
- Review release notes and security advisories before updating.
- A planned version is not an installed version. Keep status explicit.
