# 🦙 Ollama + Claude Code Integration

Run local LLMs via Ollama and connect them to Claude Code for AI-assisted development — without sending every request to the Anthropic API.

---

## Prerequisites

Before getting started, ensure you have the following installed and configured:

| Requirement | Version | Notes |
|---|---|---|
| [Claude Code](https://docs.anthropic.com/claude-code) | Latest | Anthropic's agentic coding CLI |
| [Ollama](https://ollama.com) | Latest | Local LLM runtime |
| [Docker](https://www.docker.com) *(optional)* | 24+ | Only if running Ollama in a container |
| Node.js | 18+ | Required by Claude Code |
| macOS / Linux / Windows (WSL2) | — | Windows requires WSL2 |

> **Apple Silicon (M1/M2/M3/M4) users:** For best performance, install Ollama natively rather than via Docker so it can leverage Metal GPU acceleration. Docker on Apple Silicon runs through virtualization and will use CPU only.

---

## Installation

### Option A — Native Install (Recommended for macOS)

**1. Install Ollama**

```bash
brew install ollama
```

Or download directly from [ollama.com/download](https://ollama.com/download).

**2. Start the Ollama server**

```bash
ollama serve
```

Ollama will now be listening at `http://localhost:11434`.

**3. Pull a model**

For coding tasks, the following models are recommended:

```bash
# Lightweight, fast — good for 8–16GB RAM
ollama pull qwen2.5-coder:7b

# Balanced performance
ollama pull llama3.2:3b

# Higher capability — requires 32GB+ RAM
ollama pull qwen2.5-coder:32b
```

> **16GB RAM users:** Stick with `qwen2.5-coder:7b` (~4.7GB). Models like `gpt-oss:20b` (~14GB) will consume nearly all available memory, causing significant slowdowns.

**4. Verify Ollama is running**

```bash
curl http://localhost:11434/api/tags
```

You should receive a JSON response listing your installed models.

---

### Option B — Docker Install

**1. Pull the Ollama Docker image**

```bash
docker pull ollama/ollama
```

**2. Run the container**

```bash
docker run -d \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama
```

| Flag | Purpose |
|---|---|
| `-d` | Run container in detached (background) mode |
| `-v ollama:/root/.ollama` | Persist downloaded models across restarts |
| `-p 11434:11434` | Expose the Ollama API on localhost |

**3. Pull a model inside the container**

```bash
docker exec -it ollama ollama pull qwen2.5-coder:7b
```

**4. Verify**

```bash
curl http://localhost:11434/api/tags
```

---

### Install Claude Code

If you haven't installed Claude Code yet:

```bash
npm install -g @anthropic-ai/claude-code
```

---

## References

- https://docs.ollama.com/integrations/claude-code
- https://ollama.com/library/gpt-oss

---

## Usage

### Connecting Claude Code to Ollama

Ollama exposes an Anthropic-compatible API, so you can point Claude Code directly at it by overriding the base URL and API key:

```bash
export ANTHROPIC_BASE_URL=http://localhost:11434
export ANTHROPIC_API_KEY=ollama

claude --model qwen2.5-coder:7b
```

To avoid setting these environment variables every session, add them to your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
echo 'export ANTHROPIC_BASE_URL=http://localhost:11434' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY=ollama' >> ~/.zshrc
source ~/.zshrc
```

Then simply run:

```bash
claude --model qwen2.5-coder:7b
```

---

### Using Ollama as an MCP Server inside Claude Code

This approach lets Claude Code delegate sub-tasks to a local Ollama model, saving Anthropic API tokens for complex reasoning.

Add the following to your Claude Code MCP config (`~/.claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "ollama": {
      "command": "npx",
      "args": ["-y", "ollama-mcp"],
      "env": {
        "OLLAMA_HOST": "http://localhost:11434"
      }
    }
  }
}
```

Restart Claude Code after saving the config.

---

### Switching Models

You can swap models at any time without restarting the server:

```bash
# Pull a new model
ollama pull deepseek-coder:6.7b

# Run Claude Code with the new model
claude --model deepseek-coder:6.7b
```

List all locally available models:

```bash
ollama list
```

---

## Model Reference

| Model | Size | Context | Best For |
|---|---|---|---|
| `qwen2.5-coder:7b` | ~4.7GB | 128K | Coding tasks, low RAM systems |
| `llama3.2:3b` | ~2GB | 128K | Fast responses, very limited RAM |
| `deepseek-coder:6.7b` | ~3.8GB | 16K | Code generation & completion |
| `qwen2.5-coder:32b` | ~19GB | 128K | High-quality coding, 32GB+ RAM |

---

## References

- https://docs.ollama.com/integrations/claude-code
- https://ollama.com/library/gpt-oss
- https://ollama.com/library/qwen3-coder
- https://docs.anthropic.com/claude-code/docs/mcp-servers

