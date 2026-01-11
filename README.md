# litellm-copilot

A [LiteLLM](https://github.com/BerriAI/litellm) proxy that lets you use GitHub Copilot models via the OpenAI-compatible API. Point any tool that supports `ANTHROPIC_BASE_URL` or `OPENAI_BASE_URL` at this proxy to use Copilot models.

Unlike other "copilot-proxy" projects that implement custom proxy servers, this is just a thin wrapper script and config for LiteLLM's built-in `github_copilot` provider.

## Prerequisites

- Python 3.8+
- [fzf](https://github.com/junegunn/fzf) (for interactive model selection)
- A GitHub account with Copilot access
- A secret manager (1Password CLI or macOS Keychain)

## Setup

### 1. Install LiteLLM

```bash
python -m pip install --upgrade 'litellm[proxy]' backoff
```

### 2. Create a GitHub Personal Access Token

Create a PAT with the `copilot` scope:

https://github.com/settings/tokens/new?scopes=copilot&description=LiteLLM%20Proxy

### 3. Store your token

Store the token in your preferred secret manager.

**1Password:**
```bash
# Store in 1Password (via web UI or CLI), then reference it as:
op://YourVault/YourItem/token
```

**macOS Keychain:**
```bash
security add-generic-password -a "$USER" -s copilot-token -w "ghp_your_token_here"
```

### 4. Configure environment variables

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
# Required: Command to retrieve your GitHub PAT
# Pick ONE of these based on your secret manager:

# 1Password:
export COPILOT_PROXY_TOKEN_CMD='op read "op://YourVault/YourItem/token"'

# macOS Keychain:
export COPILOT_PROXY_TOKEN_CMD='security find-generic-password -a "$USER" -s copilot-token -w'

# Optional: Default model (skips fzf selection)
export COPILOT_MODEL="claude-opus-4.5"
```

### 5. Install the script

Clone this repo and add it to your PATH:

```bash
git clone https://github.com/jonmagic/litellm-copilot.git ~/.litellm-copilot
```

Then either symlink the script:
```bash
ln -s ~/.litellm-copilot/litellm-copilot /usr/local/bin/litellm-copilot
```

Or add the directory to your PATH:
```bash
export PATH="$HOME/.litellm-copilot:$PATH"
```

## Usage

Start the proxy:

```bash
litellm-copilot
```

If `COPILOT_MODEL` is not set, you'll get an interactive model picker via fzf. Otherwise, it starts immediately with your configured model.

The proxy runs on `http://localhost:4000`.

## Using with AI Agents

Configure your favorite AI tools to use the proxy. Add these to your shell profile or set them before running the tool.

### Claude Code

```bash
export ANTHROPIC_BASE_URL="http://localhost:4000"
export ANTHROPIC_AUTH_TOKEN="fake-key"  # Required but not used
```

### OpenAI Codex CLI

```bash
export OPENAI_BASE_URL="http://localhost:4000"
export OPENAI_API_KEY="fake-key"  # Required but not used
```

Then run: `codex --model claude-sonnet-4`

### Gemini CLI

```bash
export GOOGLE_GEMINI_BASE_URL="http://localhost:4000"
export GEMINI_API_KEY="fake-key"  # Required but not used
```

### Qwen Code CLI

```bash
export OPENAI_BASE_URL="http://localhost:4000"
export OPENAI_API_KEY="fake-key"  # Required but not used
export OPENAI_MODEL="claude-sonnet-4"
```

### Open WebUI

In Open WebUI, go to **Settings → Connections** and set:
- **URL**: `http://localhost:4000`
- **Key**: `fake-key`

## Supported Models

Tested and working as of January 2026:

| Model | API Type |
| --- | --- |
| gpt-4.1 | Chat |
| gpt-4o | Chat |
| gpt-5-mini | Chat |
| grok-code-fast-1 | Chat |
| claude-haiku-4.5 | Chat |
| claude-opus-4.5 | Chat |
| claude-sonnet-4 | Chat |
| claude-sonnet-4.5 | Chat |
| gemini-2.5-pro | Chat |
| gpt-5 | Responses |
| gpt-5-codex | Responses |
| gpt-5.1 | Responses |
| gpt-5.1-codex | Responses |
| gpt-5.1-codex-max | Responses |
| gpt-5.1-codex-mini | Responses |
| gpt-5.2 | Responses |

## License

ISC
