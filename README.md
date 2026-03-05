# llm-agents

> 🇰🇷 [한국어 문서](README.ko.md)

A Claude Code plugin that uses Gemini CLI and Codex CLI as specialized sub-agents.

## Design Principles

| Role | Model | Characteristics |
|------|-------|-----------------|
| **Orchestrator** | Claude | Planning, synthesis, judgment |
| **Scout / Analysis** | Gemini | 1M token context, read-only |
| **Execution / Generation** | Codex | Sandboxed code modification |

## Agents

### Gemini Agents (Analysis)

| Agent | Model | When to Use |
|-------|-------|-------------|
| `gemini-pro` | gemini-3.0-pro | Architecture analysis, security audits, complex dependency analysis |
| `gemini-flash` | gemini-3.0-flash | Feature lookup, module summary, convention overview |
| `gemini-flash-lite` | gemini-3.0-flash-lite | YES/NO judgment, simple existence checks, one-liner summaries |

### Codex Agents (Execution)

| Agent | reasoning | When to Use |
|-------|-----------|-------------|
| `codex-xhigh` | xhigh | Large-scale refactoring, security patches, complex bugs |
| `codex-high` | high | Feature implementation, test writing, general bug fixes |
| `codex-medium` | medium | Boilerplate, CRUD, formatting, simple transformations |

## Slash Commands

| Command | Skill | Description |
|---------|-------|-------------|
| `/gemini-cli [prompt]` | gemini-cli | Analyze / explore codebase with Gemini |
| `/codex-cli [task]` | codex-cli | Implement / modify code with Codex |
| `/llm-review [branch]` | llm-review | Cross-review with Gemini + Codex |

## Recommended Workflows

### 1. New Feature Development

```
1. /gemini-cli "Identify related modules and existing patterns"
2. Implement with codex-high
3. Cross-review with /llm-review
```

### 2. Complex Bug Fix

```
1. Delegate full analysis of bug-related code to gemini-pro
2. Once root cause is identified, apply precise fix with codex-xhigh
3. Final verification with codex-xhigh review
```

### 3. Routine Refactoring

```
1. Summarize target module with gemini-flash
2. Quick cleanup with codex-medium
3. Light cross-check with /llm-review
```

## Installation

### Prerequisites

Gemini CLI and Codex CLI must be installed before installing this plugin.

#### Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini  # Authenticate with Google account (first time only)
```

#### Codex CLI

```bash
npm install -g @openai/codex
export OPENAI_API_KEY=sk-...  # Add to ~/.zshrc or ~/.bashrc, then source
```

#### Verify Installation

```bash
command -v gemini && echo "Gemini OK"
command -v codex && echo "Codex OK"
```

---

### Install as Claude Code Plugin

#### Option 1: Via Marketplace (Recommended)

Register the GitHub repo as a marketplace source in Claude Code:

```
/plugin marketplace add Dev-RubinJo/claude-llm-agents
```

Then install the plugin:

```
/plugin install claude-llm-agents@claude-llm-agents
```

#### Option 2: Local Path (Development)

Clone the repo and register as a local marketplace:

```bash
git clone https://github.com/Dev-RubinJo/claude-llm-agents
```

```
/plugin marketplace add /path/to/claude-llm-agents
```

---

### Verify Plugin Installation

After installation, restart Claude Code to activate the following:

**Agents** (available via the Task tool):
```
claude-llm-agents:gemini-pro
claude-llm-agents:gemini-flash
claude-llm-agents:gemini-flash-lite
claude-llm-agents:codex-xhigh
claude-llm-agents:codex-high
claude-llm-agents:codex-medium
```

**Slash Commands** (type `/` in chat to see the list):
```
/gemini-cli
/codex-cli
/llm-review
```

---

## Development

```bash
git clone https://github.com/Dev-RubinJo/claude-llm-agents
cd claude-llm-agents
# Edit files under agents/ or skills/, then restart Claude Code
```
