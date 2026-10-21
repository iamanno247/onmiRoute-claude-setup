# NixOS AI Development Setup: OmniRoute & Claude Code

A clean, isolated setup running **OmniRoute** in Docker and **Claude Code** installed in user space (`~/.npm-packages`), configured with project-level environment variables to bypass global pollution and interactive login prompts.

## 🐋 1. Deploying OmniRoute (Docker)

Deploy OmniRoute as a background Docker container running on port `20128`.

```bash
docker run -d \
  --name omniroute \
  --restart unless-stopped \
  --stop-timeout 40 \
  -p 20128:20128 \
  -v omniroute-data:/app/data \
  diegosouzapw/omniroute:latest
```

* **Dashboard Web UI:** `http://localhost:20128`
* **API Gateway Endpoint:** `http://localhost:20128/v1`
* **Setup Action:** Open `http://localhost:20128` -> Go to **API Keys** -> Generate an API Key (e.g., `sk-...`).
* **Model Configuration Note:** Ensure the model or combo route assigned in OmniRoute (e.g., `auto/coding`) contains target models that explicitly support **Tool Calling / Function Calling** (e.g., Gemini Flash, DeepSeek, Qwen) so Claude Code's agentic requests function properly.

## 📦 2. Declarative & Isolated Claude Code Installation

To keep the NixOS system clean and avoid installing Node modules as `root` or polluting global system directories:

1. Configure `npm` to route global installs into user home directory (`~/.npm-packages`):

   ```zsh
   mkdir -p ~/.npm-packages
   npm config set prefix '~/.npm-packages'
   ```

2. Add the custom package path to your `~/.zshrc`:

   ```zsh
   echo 'export PATH="$HOME/.npm-packages/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. Install Claude Code globally in user-space:

   ```zsh
   npm install -g @anthropic-ai/claude-code
   ```

*(Note: Uninstalling everything cleanly in the future is as simple as running `rm -rf ~/.npm-packages`)*.

## ⚙️ 3. Project Configuration

To auto-connect Claude Code to OmniRoute without interactive auth dialogs or global `~/.claude/settings.json` conflicts, place the following file in your project root (`.claude/settings.json`):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:20128/v1",
    "ANTHROPIC_AUTH_TOKEN": "YOUR_OMNIROUTE_API_KEY",
    "ANTHROPIC_MODEL": "auto/coding",
    "ANTHROPIC_SMALL_FAST_MODEL": "auto/coding",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "auto/coding",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "auto/coding",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "auto/coding",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

## 🚀 4. Launching Claude Code

In your project directory terminal, simply run:

```zsh
claude
```

### Manual / One-Line Terminal Execution

If launching outside your configured environment or forcing explicit overrides in standard terminal shells:

```zsh
ANTHROPIC_BASE_URL="http://localhost:20128/v1" ANTHROPIC_API_KEY="YOUR_OMNIROUTE_API_KEY" claude
```

## 🔧 Troubleshooting & Notes

* **`HTTP 401 Invalid API Key`:** Ensure `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_API_KEY` matches the exact key generated from your local OmniRoute UI (`http://localhost:20128`).
* **`HTTP 400 No target in combo supports tool calling`:** Claude Code sends tool definitions per request. Make sure your assigned OmniRoute combo (such as `auto/coding`) includes models with tool calling capabilities enabled.
* **`Model auto/coding isn't recognized` warning:** This is a cosmetic notice indicating Claude Code is defaulting to a 200k token context limit for non-standard model strings; it does not impede tool execution or streaming responses.