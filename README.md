# Playwright Skill for Claude Code

A lightweight, skill-based alternative to Playwright MCP for Claude Code sub-agents. Zero context bloat, full control.

## Why This Exists

### The Problem with MCP

Claude Code's MCP (Model Context Protocol) has issues:

1. **Context bloat** - All 26 Playwright tool schemas load at session start (~2000 tokens)
2. **Broken sub-agent support** - Inline MCP in sub-agent YAML [doesn't work](https://github.com/anthropics/claude-code/issues/13898)
3. **All or nothing** - Can't give Playwright to sub-agents without bloating the main agent
4. **Black box** - Hard to customize tool behavior or see what's happening

### The Skill-Based Solution

This project uses a **CLI + markdown instructions** approach:

- **Zero main agent bloat** - Playwright tools only exist in the sub-agent's context
- **Progressive disclosure** - Sub-agent loads instructions only when invoked
- **Full visibility** - Plain Python scripts you can read and customize
- **Session persistence** - Browser stays alive between commands via background daemon

## Architecture

```
Main Agent (no Playwright context)
    │
    └── Task(playwright-agent)
            │
            ├── Reads: agents/playwright-agent.md (instructions)
            └── Calls: Bash("./scripts/playwright/pw <command>")
                    │
                    └── pw CLI sends command to daemon
                            │
                            └── pw-daemon executes Playwright
                                    │
                                    └── Returns JSON result
```

## Installation

### 1. Install Dependencies

```bash
pip install playwright
playwright install chromium
```

### 2. Copy Files to Your Project

```bash
# Copy scripts
mkdir -p scripts/playwright
cp playwright-skill/scripts/playwright/* scripts/playwright/
chmod +x scripts/playwright/pw scripts/playwright/pw-daemon

# Copy agent definition
mkdir -p .claude/agents
cp playwright-skill/agents/playwright-agent.md .claude/agents/
```

### 3. Restart Claude Code

Claude Code loads agents at session start. Restart to pick up the new agent.

## Usage

### Invoke the Sub-Agent

From your main Claude Code session:

```
Use the playwright-agent to test the login flow on localhost:3000
```

Or programmatically:

```python
Task(
    subagent_type="playwright-agent",
    prompt="Navigate to localhost:3000 and verify the homepage loads",
    model="haiku"
)
```

### Direct CLI Usage

You can also use the CLI directly:

```bash
# Navigate
./scripts/playwright/pw navigate "http://localhost:3000"

# Get page structure
./scripts/playwright/pw snapshot

# Click
./scripts/playwright/pw click "button:has-text('Login')"

# Type
./scripts/playwright/pw type "#email" "user@example.com"

# Close
./scripts/playwright/pw close
```

## Commands

| Command | Description |
|---------|-------------|
| `navigate <url>` | Navigate to URL |
| `click <selector>` | Click element |
| `type <selector> <text>` | Type into input |
| `fill-form <json>` | Fill multiple fields |
| `select <selector> <value>` | Select dropdown option |
| `hover <selector>` | Hover over element |
| `drag <from> <to>` | Drag and drop |
| `press-key <key>` | Press keyboard key |
| `upload <selector> <files>` | Upload files |
| `evaluate <js>` | Run JavaScript |
| `screenshot` | Take screenshot |
| `snapshot` | Get accessibility tree |
| `resize <w> <h>` | Resize viewport |
| `close` | Close browser |
| `back` | Navigate back |
| `dialog <accept\|dismiss>` | Handle dialogs |
| `wait` | Wait for condition |
| `console` | Get console messages |
| `network` | Get network requests |
| `tabs <action>` | Manage tabs |
| `install` | Install browser |
| `run-code <python>` | Run Playwright code |

See [agents/playwright-agent.md](agents/playwright-agent.md) for full documentation.

## How It Works

### Session Persistence

The `pw` CLI communicates with a background `pw-daemon` process via file-based IPC:

1. First command starts the daemon (launches browser)
2. Commands write to `/tmp/pw-daemon/command.json`
3. Daemon reads command, executes, writes to `/tmp/pw-daemon/result.json`
4. CLI reads result, returns JSON to caller
5. `close` command terminates daemon

This keeps the browser alive between CLI invocations.

### JSON Output

All commands return structured JSON:

```json
{
  "success": true,
  "result": "Navigated to http://localhost:3000",
  "error": null
}
```

On failure:

```json
{
  "success": false,
  "result": null,
  "error": "Element not found: #missing"
}
```

## Comparison to MCP

| Aspect | MCP | Skill-Based (this) |
|--------|-----|-------------------|
| Context usage | ~2000 tokens always | 0 tokens in main agent |
| Sub-agent support | Broken (known bug) | Works perfectly |
| Customization | Requires forking MCP | Edit Python scripts |
| Visibility | Black box | Plain code |
| Session persistence | Built-in | Daemon-based |
| Tool count | 26 tools | 22 commands |

## Extending the Pattern

This architecture generalizes to any external tool:

1. **Write a CLI wrapper** - Python script that accepts commands
2. **Create markdown instructions** - Document commands for the agent
3. **Add to `.claude/agents/`** - Claude Code discovers it automatically

Examples you could build:
- Database query skill (wrap `psql` or an ORM)
- API testing skill (wrap `curl` or `httpx`)
- Git operations skill (wrap `git` commands)
- Docker management skill (wrap `docker` CLI)

## Project Structure

```
playwright-skill/
├── README.md                    # This file
├── LICENSE                      # MIT
├── scripts/
│   └── playwright/
│       ├── pw                   # CLI client
│       └── pw-daemon            # Background browser process
├── agents/
│   └── playwright-agent.md      # Sub-agent instructions
├── test/
│   └── pw-test.html            # Test page with all interactions
└── examples/
    └── workflows.md            # Example automation workflows
```

## Requirements

- Python 3.10+
- Playwright (`pip install playwright`)
- Chromium (`playwright install chromium`)
- Claude Code with sub-agent support

## License

MIT

## Contributing

Issues and PRs welcome. This is a proof-of-concept showing that skill-based tools can be simpler and more effective than MCP for many use cases.

## Credits

Built as an alternative to the [official Playwright MCP](https://github.com/microsoft/playwright-mcp) after encountering context bloat and sub-agent compatibility issues in Claude Code.
