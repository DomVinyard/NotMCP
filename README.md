# notmcp

**Real tools for AI agents**

notmcp lets AI agents create, store, and run local scripts that connect to any API. No servers. No protocols. Just code.

## Install

Copy this line and paste it to your agent (Claude Code or Cursor):

```
Hey agent, use notmcp: curl -fsSL notmcp.com/use | bash
```

That's it. Your agent runs the command, and notmcp is ready.

## How it works

1. **Ask your agent** — "Connect to PostHog" or "Search my Gmail" — just describe what you need
2. **Agent writes a tool** — Your agent creates a Python script that handles auth and API calls, stored locally
3. **Tool runs locally** — JSON in, JSON out. Your agent gets structured data and continues working

```
~/.claude/skills/notmcp/
├── SKILL.md          # Teaches your agent how to use notmcp
├── bin/notmcp        # The CLI (stdlib Python, no dependencies)
├── scripts/          # Your tools live here
│   ├── posthog-get-users.py
│   └── gmail-search.py
└── .credentials      # API keys (chmod 600, never in repos)
```

## Why notmcp

**Tools without the ceremony.** Most tools just call an API. You don't need a protocol for that.

- **No server to run** — tools are scripts, not services
- **No protocol to learn** — JSON in, JSON out
- **No dependency hell** — stdlib Python only
- **Debuggable** — run any tool manually in your terminal
- **Evolvable** — your agent can improve tools over time

## The tool contract

Tools are Python scripts with a header:

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

## Security

Credentials are stored in `~/.claude/skills/notmcp/.credentials` with `chmod 600` (owner-only). Same security model as `~/.aws/credentials`.

- Never written to any repository
- Never included in tool output
- Injected into the tool's environment at runtime

## Supported agents

- **Claude Code** — works out of the box
- **Cursor** — requires [nightly channel](https://cursor.com/docs/context/skills) (Settings → Beta → Nightly)

Both auto-discover skills from `~/.claude/skills/`.

## License

MIT
