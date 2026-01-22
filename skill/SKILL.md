---
name: notmcp
description: Local tool system for API integrations and automation. Use when connecting to external services, fetching data from APIs, or performing tasks that require credentials or network access.
---

# notmcp

You have access to a local toolbox of executable scripts. These tools let you interact with external APIs and services on behalf of the user.

## Discovering Tools

To see what tools are available:

```bash
~/.claude/skills/notmcp/bin/notmcp list
```

To search for tools by keyword:

```bash
~/.claude/skills/notmcp/bin/notmcp search <query>
```

## Running Tools

Run tools with JSON input:

```bash
~/.claude/skills/notmcp/bin/notmcp run <tool-name> --input '{"key": "value"}'
```

Tools return JSON to stdout. Exit code 0 means success.

If a tool doesn't need input, you can omit the `--input` flag:

```bash
~/.claude/skills/notmcp/bin/notmcp run <tool-name>
```

## Handling Credentials

If a tool needs credentials (API keys, tokens), it will fail with an error like:

```
Error: Missing credential(s): POSTHOG_API_KEY
```

When this happens:
1. Ask the user for the credential value
2. Store it securely:

```bash
echo "the-secret-value" | ~/.claude/skills/notmcp/bin/notmcp creds set CREDENTIAL_NAME
```

To see what credentials are already stored:

```bash
~/.claude/skills/notmcp/bin/notmcp creds list
```

## Creating Tools

**Only create new tools when the user explicitly asks** (e.g., "save this as a tool", "make this reusable", "create a tool for this").

To create a new tool:

```bash
~/.claude/skills/notmcp/bin/notmcp create tool-name
```

This creates a template at `~/.claude/skills/notmcp/scripts/tool-name.py`.

Then edit the script to implement the tool logic. Follow these conventions:

### Tool Contract

- **Input**: JSON via stdin (parsed with `json.load(sys.stdin)`)
- **Output**: JSON to stdout (use `print(json.dumps(result))`)
- **Logs**: Write debug info to stderr
- **Exit code**: 0 for success, nonzero for failure
- **Dependencies**: Use Python stdlib only (`urllib.request`, `json`, `os`, etc.)

### Tool Header Format

Every tool must have a docstring header declaring its metadata:

```python
#!/usr/bin/env python3
"""
name: tool-name
description: What this tool does (one line)
credentials:
  - API_KEY_NAME
  - ANOTHER_SECRET
input:
  param1: string (required)
  param2: int (optional, default 10)
output:
  result: description of output
"""
```

### Example Tool Structure

```python
#!/usr/bin/env python3
"""
name: example-api
description: Fetch data from Example API
credentials:
  - EXAMPLE_API_KEY
input:
  query: string (required)
output:
  results: list of matching items
"""

import json
import os
import sys
from urllib.request import urlopen, Request

def main():
    # Read input
    inp = json.load(sys.stdin) if not sys.stdin.isatty() else {}
    
    # Get credentials (injected by notmcp run)
    api_key = os.environ["EXAMPLE_API_KEY"]
    
    # Make API call
    query = inp.get("query", "")
    req = Request(f"https://api.example.com/search?q={query}")
    req.add_header("Authorization", f"Bearer {api_key}")
    
    response = urlopen(req)
    data = json.loads(response.read())
    
    # Return result
    print(json.dumps({"results": data["items"]}))

if __name__ == "__main__":
    main()
```

## Best Practices

1. **Handle pagination** - Don't return unbounded results. Implement limits and cursors.
2. **Handle rate limits** - Add delays or retry logic for APIs with rate limits.
3. **Return compact output** - Summarize large responses. Agents work better with concise data.
4. **Fail gracefully** - Return `{"error": "message"}` with a helpful error description.
5. **Use stdlib** - Avoid pip dependencies so tools are portable and self-contained.
