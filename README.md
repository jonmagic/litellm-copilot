# litellm-copilot

A [LiteLLM](https://github.com/BerriAI/litellm) proxy that lets you use GitHub Copilot models via the OpenAI-compatible API. Point any tool that supports `ANTHROPIC_BASE_URL` or `OPENAI_BASE_URL` at this proxy to use Copilot models.

Unlike other "copilot-proxy" projects that implement custom proxy servers, this is just a thin wrapper script and config for LiteLLM's built-in `github_copilot` provider.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (Python package runner)
- A GitHub account with Copilot access
- A secret manager (1Password CLI or macOS Keychain)

## Setup

### 1. Install uv

**macOS:**
```bash
brew install uv
```

**Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
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

To use a custom port:

```bash
litellm-copilot --port 8080
```

To use LiteLLM nightly from GitHub main:

```bash
litellm-copilot --nightly
```

The proxy runs on `http://localhost:4000` by default.

## Using with AI Agents

Configure your favorite AI tools to use the proxy. Add these to your shell profile or set them before running the tool.

### [Claude Code](https://code.claude.com/docs/en/overview)

Install:
```bash
npm install -g @anthropic-ai/claude-code
```

Configure:
```bash
export ANTHROPIC_BASE_URL="http://localhost:4000"
export ANTHROPIC_AUTH_TOKEN="fake-key"  # Required but not used
```

Test:
```bash
claude -p "What is 2+2?"
```

### [OpenAI Codex CLI](https://chatgpt.com/features/codex)

Install:
```bash
npm install -g @openai/codex@alpha
```

Configure:
```bash
export OPENAI_BASE_URL="http://localhost:4000"
export OPENAI_API_KEY="fake-key"  # Required but not used
```

Test:
```bash
codex exec "What is 2+2?"
```

### [Gemini CLI](https://geminicli.com)

> **Note:** Gemini CLI requires a fix in LiteLLM that forwards `extra_headers` through the generateContent adapter. This fix is pending in [PR #18935](https://github.com/BerriAI/litellm/pull/18935). Until it's merged, you can run the proxy using a patched version:
> ```bash
> # Clone the patched fork
> git clone https://github.com/jonmagic/litellm.git ~/litellm-patched
>
> # Create a venv and install (Python 3.13 required, uvloop doesn't support 3.14+)
> python3.13 -m venv ~/litellm-patched/.venv
> ~/litellm-patched/.venv/bin/pip install -e ~/litellm-patched[proxy]
>
> # Run the proxy using the patched version
> GITHUB_TOKEN=$(eval "$COPILOT_PROXY_TOKEN_CMD") ~/litellm-patched/.venv/bin/litellm --config ~/.litellm-copilot/config.yaml --port 4000
> ```

Install:
```bash
npm install -g @google/gemini-cli@preview
```

Configure:
```bash
export GOOGLE_GEMINI_BASE_URL="http://localhost:4000"
export GEMINI_API_KEY="fake-key"  # Required but not used
```

Test:
```bash
gemini -p "What is 2+2?"
```

### [Aider](https://aider.chat)

Install:
```bash
pip install aider-chat
```

Configure:
```bash
export AIDER_OPENAI_API_BASE="http://localhost:4000"
export OPENAI_API_KEY="fake-key"  # Required but not used
```

Test:
```bash
aider --model openai/claude-sonnet-4 --message "What is 2+2?"
```

### [llm](https://llm.datasette.io) (Simon Willison's LLM CLI)

Install llm:

**Homebrew (recommended):**
```bash
brew install llm
```

**pip:**
```bash
pip install llm
```

Install the Azure plugin:
```bash
llm install llm-azure
```

Verify the plugin is installed:
```bash
llm plugins
# Should show llm-azure in the list
```

Configure a dummy API key (llm requires an API key even though the proxy doesn't use it):
```bash
llm keys set litellm
# Enter: fake-api-key
```

Add model configurations to `~/.config/io.datasette.llm/azure/config.yaml`:
```yaml
# Local litellm proxy models
- model_id: copilot-gpt-4.1
  provider: openai
  model_name: gpt-4.1
  endpoint: http://localhost:4000/v1/
  api_key_name: litellm
- model_id: copilot-claude-sonnet-4.5
  provider: openai
  model_name: claude-sonnet-4.5
  endpoint: http://localhost:4000/v1/
  api_key_name: litellm
```

> **Note:** The `copilot-` prefix helps distinguish litellm-copilot models from other configured models in llm. You can add any of the models from the Supported Models section below using this pattern.

Set your default model:
```bash
llm models default copilot-gpt-4.1
```

Test:
```bash
llm "What is 2+2?"
```

## Supported Models

Last updated: 2026-02-04

| Model | Vendor | Category |
| --- | --- | --- |
| `claude-haiku-4.5` | Anthropic | versatile |
| `claude-opus-4.5` | Anthropic | powerful |
| `claude-sonnet-4` | Anthropic | versatile |
| `claude-sonnet-4.5` | Anthropic | versatile |
| `gemini-2.5-pro` | Google | powerful |
| `gemini-3-flash-preview` | Google | lightweight |
| `gemini-3-pro-preview` | Google | powerful |
| `gpt-4.1` | Azure OpenAI | versatile |
| `gpt-4o` | Azure OpenAI | versatile |
| `gpt-5` | Azure OpenAI | versatile |
| `gpt-5-codex` | OpenAI | powerful |
| `gpt-5-mini` | Azure OpenAI | lightweight |
| `gpt-5.1` | OpenAI | versatile |
| `gpt-5.1-codex` | OpenAI | powerful |
| `gpt-5.1-codex-max` | OpenAI | powerful |
| `gpt-5.1-codex-mini` | OpenAI | powerful |
| `gpt-5.2` | OpenAI | versatile |
| `gpt-5.2-codex` | OpenAI | powerful |
| `grok-code-fast-1` | xAI | lightweight |

## License

ISC
