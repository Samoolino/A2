# A2 Model Provider Selection and Installation Matrix

This guide distinguishes a model runtime from an agent CLI and from a provider API. Condor's currently documented integration paths include ACP-based CLIs (Claude Code, Gemini CLI, GitHub Copilot CLI and Codex) and PydanticAI providers (OpenRouter, Ollama and LM Studio). Provider support and exact commands can change; verify the pinned Condor documentation before installation.

## Recommended default

**Start with Ollama + Qwen3 8B Q4_K_M** if the host has enough memory and the goal is local, no per-token API billing. The current Ollama library lists the 8B model at approximately 5.2 GB; Qwen3 4B is approximately 2.5 GB and is the fallback for more constrained hosts. Download size is not the same as RAM/VRAM requirement. This is a local inference choice, not a guarantee of better trading returns or safe tool use.

Official source: https://ollama.com/library/qwen3

### Local model installation

Use the official Ollama Linux/WSL installation instructions. After installation:
```bash
ollama --version
ollama pull qwen3:8b
ollama list
ollama show qwen3:8b
ollama run qwen3:8b
```
Fallback:
```bash
ollama pull qwen3:4b
ollama run qwen3:4b
```

Verify actual tool calling against the pinned Condor model integration. Basic chat response is not enough. Keep the model's tools read-only until prompt-injection, malformed-output, timeout and denial tests pass.

## Provider options

| Provider / tool | Integration path documented by Condor | Installation / authentication | Cost and size considerations | Verification |
|---|---|---|---|---|
| Ollama + Qwen3 | Local provider through PydanticAI | Install Ollama, download model, select supported `ollama:<model>` backend using version-matched Condor config | No per-token provider bill; requires local compute, electricity, disk and RAM/VRAM | `ollama run`, Condor doctor, live backend/tool-call test |
| LM Studio | Local provider through PydanticAI | Install from https://lmstudio.ai; download a compatible tool-capable model and configure local endpoint per Condor docs | Local compute; model size and quantization determine memory | Endpoint health, structured tool call, Condor test |
| Claude Code | ACP CLI | Install using official Claude Code docs, sign in/authenticate in CLI, then select `claude-code` in Condor's `/agent` flow | Plan/account limits or usage charges depend on current Anthropic terms; not a local model | CLI auth/status, Condor provider selection, read-only tool test |
| Gemini CLI | ACP CLI | Install current official Gemini CLI package, authenticate through its supported Google flow, select `gemini` in Condor | Free tier/limits depend on current Google account and policy; do not assume unlimited free access | CLI auth/status, Condor provider selection, read-only tool test |
| GitHub Copilot CLI | ACP CLI | Install current official Copilot CLI, authenticate with GitHub, select `copilot` in Condor | Entitlements and usage limits depend on current Copilot plan | CLI auth/status, Condor provider selection, read-only tool test |
| OpenAI Codex | ACP CLI | Install/authenticate via current official Codex instructions; Condor docs list the Codex ACP package | Plan/usage limits depend on account and product terms | CLI auth/status, Condor provider selection, read-only tool test |
| OpenRouter | API provider through PydanticAI | Create an account/key at https://openrouter.ai and configure `OPENROUTER_API_KEY` only in the runtime secret environment | Per-model pricing and limits vary; some models may offer free tiers with rate/capability constraints | Minimal request, budget limit, tool-call test |
| Direct model APIs | Only if the pinned Condor version documents the provider or a reviewed adapter exists | Follow provider's official API docs; never assume all providers are natively supported | Provider-specific billing, quotas and data handling | Contract test and explicit privacy/cost review |

## Claude, Gemini, Copilot and Codex are not local model downloads

These options are command-line agents that authenticate with their respective services and are invoked through Condor's ACP integration. Installing a CLI does not download a 5 GB local model, and local Ollama installation does not install those cloud CLIs. They have separate accounts, policies, rate limits and authentication flows.

## Security and trading policy

1. Do not give any model provider raw CEX secrets, seed phrases, or unrestricted shell access.
2. Keep deterministic risk authorization outside the LLM.
3. Treat market messages, web pages, tokens and strategy files as untrusted input (prompt injection).
4. Set cost/usage limits for cloud providers.
5. Verify data retention and account policy before sending private portfolio data to a cloud provider.
6. Begin with read-only market research, then simulation/paper trading.
7. Never allow an LLM to change its own risk limits or independently switch live trading on.

## Official provider documentation
- Condor LLM integration: https://github.com/hummingbot/condor-docs/blob/main/llm-integration.mdx
- Condor setup: https://github.com/hummingbot/condor
- Ollama model library: https://ollama.com/library/qwen3
- Gemini CLI: https://github.com/google-gemini/gemini-cli
- Claude Code: https://docs.anthropic.com/en/docs/claude-code/overview
- GitHub Copilot CLI: https://docs.github.com/en/copilot/github-copilot-in-the-cli
- OpenAI Codex: https://developers.openai.com/codex/
- OpenRouter: https://openrouter.ai/docs
