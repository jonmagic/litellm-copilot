# CLI Tools + LiteLLM + GitHub Copilot Development Harness

This directory contains a development harness for testing CLI tool integrations with LiteLLM proxy using GitHub Copilot models.

## Supported CLIs

| CLI | API Format | Status | Install |
|-----|------------|--------|---------|
| **Codex CLI** | OpenAI `/chat/completions` | ✅ Works | `npm install -g @openai/codex@alpha` |
| **Gemini CLI** | Google `generateContent` | ✅ Works* | `npm install -g @google/gemini-cli@preview` |

*Gemini CLI requires the LiteLLM fix from [PR #18935](https://github.com/BerriAI/litellm/pull/18935) - this dev harness uses a local fork with the fix applied.

## Quick Start

```bash
# Start the proxy on port 4001 (uses local LiteLLM fork)
./dev-harness start

# Test the OpenAI-compatible endpoint
./dev-harness test

# Test Codex CLI (non-interactive)
./dev-harness test-codex

# Run Codex CLI interactively
./dev-harness codex 'explain this codebase'

# Test Google-native endpoint (for Gemini CLI)
./dev-harness test-google

# Run Gemini CLI
./dev-harness gemini 'Hello!'

# View logs if something goes wrong
./dev-harness logs

# Stop when done
./dev-harness stop
```

## How It Works

### Codex CLI
Codex CLI uses OpenAI-compatible format (`/chat/completions`), which works directly with the LiteLLM proxy. The wildcard model routing handles any model Codex requests.

### Gemini CLI
Gemini CLI uses Google-native format (`generateContent`), which requires LiteLLM to translate to OpenAI format. This translation is handled by `GenerateContentToCompletionHandler`.

## LiteLLM Bug Fix (PR #18935)

We discovered and fixed a bug where `extra_headers` were not forwarded when translating `generateContent` to `acompletion`. This caused GitHub Copilot to reject requests with "missing Editor-Version header".

**Fix**: [BerriAI/litellm#18935](https://github.com/BerriAI/litellm/pull/18935)

The fix is in `litellm/google_genai/adapters/handler.py`:
```python
# Forward extra_headers for providers that require custom headers (e.g., github_copilot)
if "extra_headers" in extra_kwargs:
    completion_kwargs["extra_headers"] = extra_kwargs["extra_headers"]
```

This dev harness uses a local fork with the fix applied until the PR is merged upstream.

## Files

- `dev-harness` - Shell script for managing the dev proxy
- `config.yaml` - LiteLLM configuration with GitHub Copilot models

## Environment Variables

- `DEV_PORT` - Port for dev proxy (default: 4001)
- `DEV_MODEL` - Default model to use (default: gemini-2.5-pro)
- `COPILOT_PROXY_TOKEN_CMD` - Command to get GitHub token (required)

## Debugging Commands

```bash
# Test OpenAI endpoint
./dev-harness test

# Test Codex CLI
./dev-harness test-codex

# Test Google-native endpoint
./dev-harness test-google

# See the API format differences
./dev-harness debug-request

# List available models
./dev-harness models

# Check logs for detailed errors
./dev-harness logs
```

## References

- [LiteLLM Gemini CLI Tutorial](https://docs.litellm.ai/docs/tutorials/litellm_gemini_cli)
- [LiteLLM OpenAI Codex Tutorial](https://docs.litellm.ai/docs/tutorials/openai_codex)
- [LiteLLM GitHub Copilot Provider](https://docs.litellm.ai/docs/providers/github_copilot)
- [LiteLLM GitHub Repository](https://github.com/BerriAI/litellm)
