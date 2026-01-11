# Gemini CLI + LiteLLM + GitHub Copilot Development Harness

This directory contains a development harness for testing Gemini CLI integration with LiteLLM proxy using GitHub Copilot models.

## Quick Start

```bash
# Start the proxy on port 4001 (uses LiteLLM nightly)
./dev-harness start

# Test the OpenAI-compatible endpoint (works!)
./dev-harness test

# Try Gemini CLI (see limitations below)
./dev-harness gemini 'Hello!'

# View logs if something goes wrong
./dev-harness logs

# Stop when done
./dev-harness stop
```

## Current Status: Bug in LiteLLM

**Gemini CLI cannot work with GitHub Copilot models through LiteLLM due to a bug in the adapter code.**

### The Problem

GitHub Copilot requires specific headers (`Editor-Version`, `Editor-Plugin-Version`, etc.) for authentication. These are configured in `extra_headers` in our config.yaml.

When using the OpenAI-compatible `/chat/completions` endpoint, these headers work correctly. However, when Gemini CLI calls the Google-native `generateContent` endpoint:

1. LiteLLM detects this is a non-Google provider (`github_copilot`)
2. It uses `GenerateContentToCompletionHandler` to translate to OpenAI format
3. **BUG: The adapter does NOT forward `extra_headers` to the underlying `acompletion` call**
4. GitHub Copilot rejects the request: `missing Editor-Version header for IDE auth`

### Root Cause (LiteLLM Code)

In `litellm/google_genai/adapters/handler.py`, the `_prepare_completion_kwargs` method only forwards `metadata`, not `extra_headers`:

```python
# Only metadata is forwarded, not extra_headers!
if extra_kwargs is not None and "metadata" in extra_kwargs:
    completion_kwargs["metadata"] = extra_kwargs["metadata"]
```

The fix would be to also forward `extra_headers`:
```python
if extra_kwargs is not None and "extra_headers" in extra_kwargs:
    completion_kwargs["extra_headers"] = extra_kwargs["extra_headers"]
```

### What Works

| Path | Status | Notes |
|------|--------|-------|
| OpenAI clients → `/chat/completions` → GitHub Copilot | ✅ Works | Headers passed correctly |
| Gemini CLI → `generateContent` → GitHub Copilot | ❌ Broken | Headers not forwarded |
| Gemini CLI → `generateContent` → Real Gemini API | ✅ Works | No translation needed |

## Potential Solutions

### Option 1: Contribute Fix to LiteLLM (Recommended)
File an issue and/or PR on [BerriAI/litellm](https://github.com/BerriAI/litellm) to fix the adapter to forward `extra_headers`.

**Proposed fix location**: `litellm/google_genai/adapters/handler.py` in `_prepare_completion_kwargs()`

### Option 2: Use OpenAI-Compatible CLI
Use a CLI that speaks OpenAI format instead of Google format:
- [aichat](https://github.com/sigoden/aichat)
- [llm](https://github.com/simonw/llm)
- Direct curl to `/chat/completions`

### Option 3: Build a Translation Proxy
Create a small proxy that:
1. Receives Google `generateContent` requests
2. Translates to OpenAI `/chat/completions` format with headers
3. Forwards to LiteLLM

### Option 4: Fork LiteLLM
Apply the fix locally in a forked version.

## Files

- `dev-harness` - Shell script for managing the dev proxy
- `config.yaml` - LiteLLM configuration with GitHub Copilot models

## Environment Variables

- `DEV_PORT` - Port for dev proxy (default: 4001)
- `DEV_MODEL` - Default model to use (default: gemini-2.5-pro)
- `COPILOT_PROXY_TOKEN_CMD` - Command to get GitHub token (required)

## Debugging Commands

```bash
# Test OpenAI endpoint (should work)
./dev-harness test

# Test Google-native endpoint (will fail with github_copilot)
./dev-harness test-google

# See the API format differences
./dev-harness debug-request

# Check logs for detailed errors
./dev-harness logs
```

## References

- [LiteLLM Gemini CLI Tutorial](https://docs.litellm.ai/docs/tutorials/litellm_gemini_cli)
- [LiteLLM GitHub Copilot Provider](https://docs.litellm.ai/docs/providers/github_copilot)
- [LiteLLM Google AI Studio Pass-Through](https://docs.litellm.ai/docs/pass_through/google_ai_studio)
- [LiteLLM GitHub Repository](https://github.com/BerriAI/litellm)
