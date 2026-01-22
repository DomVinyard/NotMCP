# notmcp

**Tools for agents, not protocols.**

notmcp is a local tool system for AI agents. Instead of wiring up MCP servers or installing integrations, your agent writes and maintains executable scripts that live on your machine.

## Install

Copy this line and paste it to your agent (Claude Code, Cursor, etc.):

```
Install notmcp: curl -fsSL https://notmcp.com/install.sh | bash
```

That's it. Your agent runs the command, and notmcp is ready.

## What it does

When you ask your agent to "connect to PostHog" or "search my Gmail," it doesn't negotiate with a server. It writes (or reuses) a script that does the job, stores credentials safely, and runs it locally. The tool returns structured output, and the agent continues.

**The user never has to think about any of this.** notmcp is agent-first. Your interface is just the chat. The agent quietly maintains a local toolbox at `~/.claude/skills/notmcp/`.

## How it works

```
~/.claude/skills/notmcp/
├── SKILL.md          # Teaches your agent how to use notmcp
├── bin/notmcp        # The CLI (stdlib Python, no dependencies)
├── scripts/          # Your tools live here
│   ├── posthog-get-users.py
│   └── gmail-search.py
└── .credentials      # API keys (chmod 600, never in repos)
```

Your agent discovers tools by running `notmcp list` or `notmcp search`. It runs them with `notmcp run <tool>`. It creates new tools with `notmcp create <name>`.

## The tool contract

Tools are just Python scripts with a header:

```python
#!/usr/bin/env python3
"""
name: my-tool
description: What this tool does
credentials:
  - MY_API_KEY
"""
import json, os, sys

def main():
    inp = json.load(sys.stdin) if not sys.stdin.isatty() else {}
    api_key = os.environ["MY_API_KEY"]
    
    # Do the work...
    
    print(json.dumps({"result": "..."}))

if __name__ == "__main__":
    main()
```

- JSON in (stdin) → JSON out (stdout)
- Credentials injected as environment variables
- Logs go to stderr
- Exit 0 = success

## Why not MCP?

MCP is great for structured tool ecosystems. But most tools are just "call an API, handle auth, return something sane." notmcp replaces the ceremony with the obvious thing: code.

- **No server to run** – tools are scripts, not services
- **No protocol to learn** – JSON in, JSON out
- **No dependency hell** – stdlib Python only
- **Debuggable** – run any tool manually: `./scripts/my-tool.py`
- **Evolvable** – your agent can improve tools over time

## Credential security

Credentials are stored in `~/.claude/skills/notmcp/.credentials` with `chmod 600` (owner-only access). This is the same security model as `~/.aws/credentials` or `~/.netrc`. Credentials are:

- Never written to any repository
- Never included in tool output
- Injected into the tool's environment at runtime

## Supported agents

notmcp works with any agent that can:
1. Read skill files from `~/.claude/skills/`
2. Run shell commands

This includes Claude Code and Cursor. The skill auto-discovery means your agent learns about notmcp automatically.

## License

MIT
