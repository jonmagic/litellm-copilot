# Copilot Instructions

This is a LiteLLM proxy wrapper for GitHub Copilot models. It consists of shell scripts and YAML configuration—no build system, tests, or linting.

## Architecture

The project provides two ways to run the LiteLLM proxy:

1. **Production script** (`litellm-copilot`) - Runs via `uvx`, fetching LiteLLM dynamically
2. **Development harness** (`dev/dev-harness`) - Uses a local LiteLLM fork for testing upstream fixes

Both use `config.yaml` which defines:
- Wildcard model routing (`github_copilot/*`)
- Model aliases (mapping model names to Copilot-supported models)
- Required headers for Copilot API authentication

## Key Files

- `litellm-copilot` - Main executable, runs proxy via `uvx`
- `config.yaml` - LiteLLM configuration with model routing and aliases
- `update-models` - Fetches available models from Copilot API, tests each one, updates config and README
- `dev/dev-harness` - Development proxy for testing with local LiteLLM fork

## Updating Models

The model list in `config.yaml` and `README.md` is maintained by:

```bash
# Requires GITHUB_TOKEN with copilot scope
./update-models           # Update files
./update-models --dry-run # Preview changes
```

A GitHub Actions workflow runs this daily and creates PRs for changes.

## Development Testing

For iterating on LiteLLM compatibility issues:

```bash
cd dev
./dev-harness start      # Start proxy on port 4001
./dev-harness test       # Test OpenAI endpoint
./dev-harness test-codex # Test Codex CLI
./dev-harness logs       # View proxy logs
./dev-harness stop       # Stop proxy
```

## Conventions

- Scripts use `zsh` with `set -e` for fail-fast behavior
- Token retrieval is deferred to runtime via `COPILOT_PROXY_TOKEN_CMD`
- Python 3.13 is pinned because `uvloop` doesn't support 3.14+
- Model aliases in `config.yaml` are auto-generated between `# Supported models` and `# Gemini CLI aliases` comments
