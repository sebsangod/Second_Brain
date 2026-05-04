---
aliases:
  - Hooks
tags:
  - dev/ai/llm
date: 2026-05-03
---
**Related:** [[Claude Code]], [[Hooks]]

---

## Description

Avoid ``Claude Code`` to read the secret keys of any project within _.env_ files.

---

## Settings

- **Type:** Global
- **Definition:** _~/.claude/settings.local.json_
- **Implementation:** _~/.claude/hooks/dont_read_dotenv.py_

---

## Implementation

### Definition

```json title:.claude/settings.local.json
{
    "hooks": {
        "PreToolUse": [
            {
                "matcher": "Read|Grep",
                "hooks": [
                    {
                        "type": "command",
                        "command": "py C:/Users/USER/.claude/hooks/dont_read_dotenv.py",
                        "command": "python3 ~/.claude/hooks/dont_read_dotenv.py"
                    }
                ]
            }
        ]
    }
}
```


### Implementation

```python title:dont_read_dotenv.py
#!/usr/bin/env python3
from json import loads
from sys import exit, stdin, stderr


def main():
	raw = stdin.read()
	tool_args = loads(raw)
	
	# Extract the file path Claude is trying to read
	read_path = (
		tool_args.get("tool_input", {}).get("file_path")
		or tool_args.get("tool_input", {}).get("path")
		or ""
	)
	
	# Check if Claude is trying to read the .env file
	if ".env" in read_path:
		print(f"You cannot read the .env file: {stderr}")
		exit(2)

if __name__ == "__main__":
	main()

```

---

## Claude Sessions
