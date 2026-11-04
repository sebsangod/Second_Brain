---
aliases:
  - Hooks
tags:
  - learning
  - dev/ai/llm
date: 2026-11-02
---
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Related:** [[Large Language Model]], [[Claude Code]]

---

## Description

_Hooks_ **allow you to run commands before or after `Claude` attempts to run a tool.** They're incredibly useful for implementing automated workflows like running code formatters after file edits, executing tests when files change, or blocking access to specific files.

---

## Key Benefits

This approach provides several advantages:

- **Proactive protection**: blocks access before sensitive data is read
- **Transparent operation**: `Claude` understands why the operation failed
- **Flexible matching**: works with multiple tools (_read_, _grep_, etc.)
- **Clear feedback**: provides meaningful error messages

While this specific example focuses on _.env_ files, the same pattern can protect any sensitive files or directories in your project. You can extend the logic to check for multiple file patterns or implement more sophisticated access controls based on your security requirements.

---

## Details

### How Hooks Work

To understand _hooks_, let's first review the normal flow when you interact with Claude Code. When you ask `Claude` something, your query gets sent to the `Claude` model along with tool definitions. `Claude` might decide to use a tool by providing a formatted response, and then Claude Code executes that tool and returns the result.

_Hooks_ **insert themselves into this process, allowing you to execute code just before or just after the tool execution happens.**

![[hooks.png]]

There are two types of _hooks_:

- **PreToolUse** _hooks_: Run **before** a tool is called
- **PostToolUse** _hooks_: Run **after** a tool is called

![[hooks2.png]]


### _Hooks_ Configuration

_Hooks_ are defined in Claude settings files. You can add them to:

- **Global**: _~/.claude/settings.json_ (affects all projects)
- **Project**: _.claude/settings.json_ (shared with team)
- **Project (not committed)**: _.claude/settings.local.json_ (personal settings)

You can write _hooks_ by hand in these files or use the _/hooks_ command inside Claude Code.

```json title:.claude/settings.local.json
{
    "other_stuff": {},

    "hooks": {
        "PreToolUse": [ // Runs before a tool is called
            {
                "matcher": "Read|Grep",
                "hooks": [
                    {
                        "type": "command",
                        "command": "node /home/hooks/read_hook.js"
                    }
                ]
            }
        ],
        "PostToolUse": [ // Runs after a tool is called
            {
                "matcher": "Write|Edit|MultiEdit",
                "hooks": [
                    {
                        "type": "command",
                        "command": "node /home/hooks/edit_hook.js"
                    }
                ]
            }
        ],
    },

    "other_stuff": {},
}
```


### _PreToolUse Hooks_

_PreToolUse hooks_ run before a tool is executed. They include a matcher that specifies which tool types to target (as in the example above).

Before the _Read_ tool is executed, this configuration runs the specified command. Your command receives details about the tool call `Claude` wants to make, and you can:

- Allow the operation to proceed normally
- Block the tool call and send an error message back to `Claude`


### _PostToolUse Hooks_

_PostToolUse hooks_ run after a tool has been executed. Here's an example that triggers after write, edit, or multi-edit operations (as in the example above).

Since the tool call has already occurred, _PostToolUse hooks_ can't block the operation. However, they can:

- Run follow-up operations (like formatting a file that was just edited)
- Provide additional feedback to `Claude` about the tool use


### Practical Applications

Here are some common ways to use _hooks_:

- **Code formatting**: Automatically format files after `Claude` edits them
- **Testing**: Run tests automatically when files are changed
- **Access control**: Block `Claude` from reading or editing specific files
- **Code quality**: Run linters or type checkers and provide feedback to `Claude`
- **Logging**: Track what files `Claude` accesses or modifies
- **Validation**: Check naming conventions or coding standards

**The key insight is that** _hooks_ **let you extend Claude Code's capabilities by integrating your own tools and processes into the workflow.** _PreToolUse hooks_ give you control over what `Claude` can do, while _PostToolUse hooks_ let you enhance what `Claude` has done.

___

### Building a _Hook_

Creating a _hook_ involves four main steps:

![[building_hooks_steps.png]]

1. **Decide on a** _PreToolUse_ **or** _PostToolUse_ _hook_: _PreToolUse hooks_ can prevent tool calls from executing, while _PostToolUse hooks_ run after the tool has already been used
2. **Determine which type of tool calls you want to watch for**: You need to specify exactly which tools should trigger your _hook_
3. **Write a command that will receive the tool call**: This command gets JSON data about the proposed tool call via standard input
4. **If needed, command should provide feedback to `Claude`**: Your command's exit code tells `Claude` whether to allow or block the operation


#### Available Tools

Claude Code provides several built-in tools that you can monitor with _hooks_:

![[tools.png]]

To see exactly which tools are available in your current setup, you can ask `Claude` directly for a list. This is especially useful since the available tools can change when you add custom `MCP servers`.

```bash title:Claude
> List out the names of all the tools you have access to, bullet point list.
```


#### Tool Call Data Structure

When your _hook_ command executes, `Claude` sends JSON data through standard input containing details about the proposed tool call:

![[tool_data_call.png]]

```json
{
    "session_id": "2d6a1e4d-6...",
    "transcript_path": "/Users/sg/...",
    "hook_event_name": "PreToolUse",
    "tool_name": "Read",
    "tool_input": {
        "file_path": "/code/queries/.env"
    }
}
```

Your command reads this JSON from standard input, parses it, and then decides whether to allow or block the operation based on the tool name and input parameters


#### Exit Codes and Control Flow

Your _hook_ command communicates back to `Claude` through exit codes:

![[hooks_exit_codes.png]]

- **Exit Code 0**: Everything is fine, allow the tool call to proceed
- **Exit Code 2**: Block the tool call (_PreToolUse hooks_ only)
- **Anything else** is non-blocking. Standard error gets logged, and Claude carries on.

When you exit with code 2 in a _PreToolUse hook_, any error messages you write to standard error will be sent to `Claude` as feedback, explaining why the operation was blocked.


#### Example Use Case

A common use case is preventing `Claude` from reading sensitive files like _.env_ files. Since both the _Read_ and _Grep_ tools can access file contents, you'd want to monitor both tool types and check if they're trying to access restricted file paths.

This approach gives you complete control over `Claude's` file system access while providing clear feedback about why certain operations are restricted.

___

### Implementing a _hook_

Let's build a custom _hook_ to prevent `Claude` from reading sensitive files like _.env_. This is a practical example of how _hooks_ can protect your environment variables and other confidential data during development sessions.


#### Setting Up the _Hook_ Configuration

First, we need to configure our hook in the settings file. Open your _.claude/settings.local.json_ file and locate the hooks section. We'll create a _PreToolUse hook_ since we want to intercept tool calls before they execute.

The configuration requires two key pieces:

- **Matcher**: specifies which tools to watch for
- **Command**: the script that runs when those tools are called

For the matcher, we want to catch both _read_ and _grep_ operations that might access the _.env_ file:

```
"matcher": "Read|Grep"
```

The pipe symbol (_|_) acts as an _OR_ operator, so this will trigger on either tool. For the command, we'll point to a `Node.js` script:

```
"command": "node ./hooks/read_hook.js"
```


#### Understanding Tool Call Data

**When** `Claude` **attempts to use a tool, your** _hook_ **receives detailed information about that call through standard input as JSON.** This data includes:

- Session ID and transcript path
- _Hook_ event name (_PreToolUse_ in our case)
- Tool name (_Read_, _Grep_, etc.)
- Tool input parameters, including the file path

Your _hook_ script processes this data and can either allow the operation to continue or block it by exiting with a specific code.


#### Implementing the _Hook_ Script

##### Using `NodeJS`

The _hook_ script needs to read the tool call data from standard input and check if `Claude` is trying to access the _.env_ file. Here's the core logic:

```javascript title:read_hook.js
async function main() {
    const chunks = [];
    for await (const chunk of process.stdin) {
        chunks.push(chunk);
    }

    const toolArgs = JSON.parse(Buffer.concat(chunks).toString());

    // Extract the file path Claude is trying to read
    const readPath = toolArgs.tool_input?.file_path || toolArgs.tool_input?.path || "";

    // Check if Claude is trying to read the .env file
    if (readPath.includes('.env')) {
        console.error("You cannot read the .env file");
        process.exit(2);
    }
}
```

The script checks for _.env_ in the file path and blocks the operation if found. When you exit with _code 2_, `Claude` receives an error message and understands the operation was blocked by a _hook_.


##### Using `Python`

Claude Code's _hooks_ work with every executable script with access to _stdin_ values and returns _stdout_ / _stderr_ values too. So the `Python` equivalent to the previous `NodeJS` _hook_ script is:

```json title:.claude/settings.local.json
{
    "other_stuff": {},

    "hooks": {
        "PreToolUse": [ // Runs before a tool is called
            {
                "matcher": "Read|Grep",
                "hooks": [
                    {
                        "type": "command",
                        "command": "python /home/hooks/read_hook.py"
                    }
                ]
            }
        ]
    },

    "other_stuff": {},
}
```

```python title:read_hook.py
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

**You must give execution permissions to the script:**

```bash title:bash
chmod +x /home/hooks/read_hook.py
```


##### Using Another

Claude Code simply passes the tool's context through _stdin_ by a JSON and reads the result, so the important thing for **any language** are the exit codes:

* **exit 0**: allows the tool call
* **exit 1**: raises an error during the _hook_ execution (tool call is not blocked but it raises an error)
- **exit 2**: blocks the tool call (the message within _stderr_ is shown to the user and sent to Claude Code to analyze)

With this, you can use `NodeJS`, `Python`, `Bash`, or any other language that can run as a process. Here is an example using `Bash`:

```bash title:read_hook.sh
#!/bin/bash
input=$(cat)
path=$(echo "$input" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))")

if [[ "$path" == *".env"* ]]; then
  echo "You cannot read the .env file" >&2
  exit 2
fi
```


#### Testing Your Hook

**After saving your configuration and** _hook_ **script, restart Claude Code for the changes to take effect.** Then test it by asking `Claude` to read your _.env_ file.

When `Claude` attempts the read operation, your hook will intercept it and return an error message. `Claude` will recognize that the operation was blocked and explain this to you, often mentioning that a read hook prevented access to the file.

The same protection works for _grep_ operations - if `Claude` tries to search within the _.env_ file, the hook will block that as well.

---

## Examples

### Block env Files Access

#### Description

Avoid `Claude Code` to read the secret keys of any project within _.env_ files.

#### Settings

- **Type:** Global
- **Definition:** _~/.claude/settings.local.json_
- **Implementation:** _~/.claude/hooks/dont_read_dotenv.py_

#### Implementation

##### Definition

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

##### Implementation

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

### Obsidian Syncing

#### Description

Save every `Claude Code` conversation within my _Obsidian Second Brain Vault_ automatically after end it.

This _hook_ is complemented by the `/distribute` `command`

#### Settings

- **Type:** Global
- **Definition:** _~/.claude/settings.local.json_
- **Implementation:** _~/.claude/hooks/sync_to_obsidian.py_

#### Implementation

##### Definition

```json title:.claude/settings.local.json
{
    "hooks": {
        "PostToolUse": [
            {
                "matcher": "Stop",
                "hooks": [
                    {
                        "type": "command",
                        "command": "py C:/Users/USER/.claude/hooks/sync_to_obsidian.py",
                        "command": "python3 ~/.claude/hooks/sync_to_obsidian.py"
                    }
                ]
            }
        ]
    }
}
```

##### Implementation

```python title:dont_read_dotenv.py
#!/usr/bin/env python3
"""
sync_to_obsidian.py — Claude Code Stop Hook

Syncs conversation transcripts to an Obsidian vault.
Triggered by the Stop event in Claude Code hooks.

- Reads the session JSONL transcript
- Generates a Markdown transcript note in Journal/Claude Sessions/<date>/
- Scans the vault to build a classification index (titles, aliases, tags, wikilinks)
- Classifies the conversation and links it to ALL relevant existing notes
- Updates existing notes with a ## Claude Sessions section + wikilinks

Zero additional Claude tokens consumed — runs entirely locally.
"""

import json
import os
import re
import sys
from datetime import datetime
from pathlib import Path


# ─── Configuration ───────────────────────────────────────────────────────────

VAULT_PATH = Path(r"C:\Users\USER\Documents\Obsidian\Main")
SESSIONS_DIR = VAULT_PATH / "Journal" / "Claude Sessions"
CLAUDE_DIR = Path.home() / ".claude"

# Directories to exclude from vault scanning
EXCLUDE_DIRS = {
    ".obsidian", ".git", "Images", "Templates",
    "Excalidraw", ".vscode", "Journal",
}

# Spanish month abbreviations
SPANISH_MONTHS = {
    1: "ene", 2: "feb", 3: "mar", 4: "abr",
    5: "may", 6: "jun", 7: "jul", 8: "ago",
    9: "sep", 10: "oct", 11: "nov", 12: "dic",
}

# ── Classification weights ───────────────────────────────────────────────────
WEIGHT_TITLE = 3.0
WEIGHT_ALIAS = 2.5
WEIGHT_TAG = 2.0
WEIGHT_FOLDER = 1.5
WEIGHT_WIKILINK = 1.0
MIN_SCORE_THRESHOLD = 2.0

# ── Stopwords (EN + ES) ─────────────────────────────────────────────────────
STOPWORDS = {
    # English
    "the", "a", "an", "is", "are", "was", "were", "be", "been", "being",
    "have", "has", "had", "do", "does", "did", "will", "would", "could",
    "should", "may", "might", "can", "shall", "to", "of", "in", "for",
    "on", "with", "at", "by", "from", "as", "into", "through", "during",
    "before", "after", "above", "below", "between", "under", "again",
    "further", "then", "once", "here", "there", "when", "where", "why",
    "how", "all", "each", "every", "both", "few", "more", "most", "other",
    "some", "such", "nor", "not", "only", "own", "same", "than", "too",
    "very", "just", "because", "but", "and", "or", "if", "this", "that",
    "these", "those", "its", "my", "your", "our", "their", "what", "which",
    "who", "whom", "me", "him", "her", "them", "you", "he", "she", "we",
    "they", "about", "also", "use", "using", "used", "file", "files",
    "want", "need", "like", "make", "get", "set", "new", "one", "two",
    # Spanish
    "el", "la", "los", "las", "un", "una", "unos", "unas", "de", "del",
    "al", "en", "que", "se", "es", "por", "con", "para", "como", "pero",
    "su", "ya", "mi", "si", "no", "lo", "le", "les", "me", "te", "nos",
    "también", "hay", "sobre", "entre", "cuando", "tiene", "todo", "esta",
    "ser", "son", "desde", "está", "era", "fue", "han", "muy", "sin",
    "puede", "dónde", "qué", "cómo", "quiero", "necesito", "puedo",
    "debo", "hacer", "usar", "esto", "eso", "aquí", "ahí", "así",
    "más", "cada", "sólo", "solo", "bien", "otro", "otra", "otros",
    "otra", "sus", "mis", "tus", "ese", "esa", "esos", "esas",
}


# ─── Data structures ────────────────────────────────────────────────────────

class Message:
    """A single user or assistant message extracted from the JSONL."""

    def __init__(self, role: str, text: str, timestamp: str):
        self.role = role
        self.text = text
        self.timestamp = timestamp
        self.time_str = ""
        try:
            dt = datetime.fromisoformat(timestamp.replace("Z", "+00:00"))
            self.time_str = dt.strftime("%H:%M")
        except (ValueError, AttributeError):
            pass


class NoteInfo:
    """Metadata about a single Obsidian note, used for classification."""

    def __init__(self, path, title, aliases, tags, wikilinks, folder_parts,
                 is_virtual=False):
        self.path = path
        self.title = title
        self.aliases = aliases
        self.tags = tags
        self.wikilinks = wikilinks
        self.folder_parts = folder_parts
        self.is_virtual = is_virtual  # True for orphan-folder stubs


class Match:
    """A classification match between a conversation and a vault note."""

    def __init__(self, note_path, note_title, score):
        self.note_path = note_path
        self.note_title = note_title
        self.score = score


# ─── Helpers ─────────────────────────────────────────────────────────────────

def spanish_date(dt: datetime) -> str:
    """Format *dt* as ``DD/mon/YYYY`` in Spanish."""
    return f"{dt.day}/{SPANISH_MONTHS[dt.month]}/{dt.year}"


def slugify(text: str, max_len: int = 60) -> str:
    """Return a filename-safe version of *text* (keeps spaces, caps)."""
    text = re.sub(r'[<>:"/\\|?*\r\n]', "", text)
    text = re.sub(r"\s+", " ", text).strip()
    if len(text) > max_len:
        text = text[:max_len].rsplit(" ", 1)[0]
    return text


def extract_title(messages: list) -> str:
    """Derive a human-readable title from the first substantive user message."""
    for msg in messages:
        if msg.role == "user" and len(msg.text.strip()) >= 20:
            raw = re.sub(r"\s+", " ", msg.text.strip())
            title = slugify(raw, max_len=60)
            if title:
                return title.title()
    return "Untitled Session"


def tokenize(text: str) -> list:
    """Split *text* into lowercase tokens, dropping stopwords and short words."""
    words = re.findall(r"[a-záéíóúñü\d_]+", text.lower())
    return [w for w in words if w not in STOPWORDS and len(w) > 2]


# ─── JSONL parser ────────────────────────────────────────────────────────────

def find_transcript(session_id: str):
    """Search ``~/.claude/projects/*/`` for a JSONL matching *session_id*."""
    projects_dir = CLAUDE_DIR / "projects"
    if not projects_dir.exists():
        return None
    for project_dir in projects_dir.iterdir():
        if project_dir.is_dir():
            candidate = project_dir / f"{session_id}.jsonl"
            if candidate.exists():
                return str(candidate)
    return None


def parse_transcript(path: str) -> list:
    """Parse the session JSONL and return a list of :class:`Message`."""
    messages = []
    try:
        with open(path, "r", encoding="utf-8") as fh:
            for line in fh:
                line = line.strip()
                if not line:
                    continue
                try:
                    entry = json.loads(line)
                except json.JSONDecodeError:
                    continue

                if entry.get("type") == "file-history-snapshot":
                    continue

                msg = entry.get("message", {})
                role = msg.get("role", "")
                content = msg.get("content", "")
                ts = entry.get("timestamp", "")

                if role == "user" and isinstance(content, str) and content.strip():
                    # Skip internal Claude Code commands
                    stripped_content = content.strip()
                    if stripped_content.startswith("<local-command") or stripped_content.startswith("<command-name>"):
                        continue
                    messages.append(Message("user", content, ts))

                elif role == "assistant" and isinstance(content, list):
                    parts = [
                        blk.get("text", "")
                        for blk in content
                        if isinstance(blk, dict) and blk.get("type") == "text"
                    ]
                    full = "\n".join(parts).strip()
                    if full:
                        messages.append(Message("assistant", full, ts))
    except (IOError, OSError):
        pass
    return messages


def extract_session_metadata(path: str) -> dict:
    """Pull ``session_id``, ``cwd``, etc. from the first few JSONL entries."""
    meta = {"session_id": "", "cwd": "", "version": "", "git_branch": ""}
    try:
        with open(path, "r", encoding="utf-8") as fh:
            for line in fh:
                try:
                    entry = json.loads(line.strip())
                except json.JSONDecodeError:
                    continue
                if entry.get("sessionId"):
                    meta["session_id"] = entry["sessionId"]
                if entry.get("cwd"):
                    meta["cwd"] = entry["cwd"]
                if entry.get("version"):
                    meta["version"] = entry["version"]
                if entry.get("gitBranch"):
                    meta["git_branch"] = entry["gitBranch"]
                if all(meta.values()):
                    break
    except (IOError, OSError):
        pass
    return meta


# ─── Markdown generator ─────────────────────────────────────────────────────

def generate_transcript_md(
    messages: list,
    meta: dict,
    title: str,
    now: datetime,
    related_titles: list | None = None,
) -> str:
    """Build a full Obsidian-formatted transcript note."""
    date_nice = spanish_date(now)
    date_iso = now.strftime("%Y-%m-%d")

    related = ["[[Claude Code]]"]
    for rt in related_titles or []:
        link = f"[[{rt}]]"
        if link not in related:
            related.append(link)

    lines = [
        "---",
        "aliases:",
        f"  - {title}",
        "tags:",
        "  - claude-session",
        f"date: {date_iso}",
        f"session_id: {meta.get('session_id', '')}",
        f"cwd: {meta.get('cwd', '')}",
        "distributed: false",
        "---",
        f"**Date**: {date_nice}",
        f"**Update**: {date_nice}",
        "**Sources**: Claude Code Session",
        "",
        "**Tags:** #claude-session",
        "",
        f"**Related:** {', '.join(related)}",
        "",
        "---",
        "",
        "## Summary",
        "",
        "Write here...",
        "",
        "---",
        "",
        "## Conversación",
        "",
    ]

    for msg in messages:
        if msg.role == "user":
            lines.append(f"### 🧑 Usuario — {msg.time_str}")
        else:
            lines.append(f"### 🤖 Claude — {msg.time_str}")
        lines.append("")
        lines.append(msg.text)
        lines.append("")

    lines.append("---")
    lines.append("")
    return "\n".join(lines)


# ─── Vault scanner ──────────────────────────────────────────────────────────

def _extract_frontmatter(filepath: str) -> dict:
    """Return aliases, tags, and wikilinks found in *filepath*."""
    result = {"aliases": [], "tags": [], "wikilinks": []}
    try:
        with open(filepath, "r", encoding="utf-8") as fh:
            content = fh.read()
    except (IOError, OSError, UnicodeDecodeError):
        return result

    # YAML frontmatter
    fm = re.match(r"^---\s*\n(.*?)\n---", content, re.DOTALL)
    if fm:
        fm_text = fm.group(1)
        for key in ("aliases", "tags"):
            block = re.search(rf"{key}:\s*\n((?:\s+-\s+.+\n?)*)", fm_text)
            if block:
                result[key] = [v.strip() for v in re.findall(r"-\s+(.+)", block.group(1))]

    # Inline #tags
    result["tags"] = list(set(result["tags"] + re.findall(r"#(\w+)", content)))

    # Wikilinks [[Target]] or [[Target|Alias]]
    result["wikilinks"] = re.findall(r"\[\[([^\]|]+?)(?:\|[^\]]+)?\]\]", content)
    return result


def scan_vault(vault_path: Path) -> list:
    """Walk the vault and return a list of :class:`NoteInfo`."""
    notes = []
    for root, dirs, files in os.walk(vault_path):
        dirs[:] = [d for d in dirs if d not in EXCLUDE_DIRS]
        for fname in files:
            if not fname.endswith(".md") or ".excalidraw" in fname:
                continue
            full = os.path.join(root, fname)
            rel = os.path.relpath(full, vault_path)
            title = fname[:-3]
            folder_parts = list(Path(rel).parent.parts)
            fm = _extract_frontmatter(full)
            notes.append(
                NoteInfo(full, title, fm["aliases"], fm["tags"], fm["wikilinks"], folder_parts)
            )
    return notes


def find_orphan_folders(vault_path: Path, notes: list) -> list:
    """Detect folders that hold ``.md`` files but lack a same-name root note.

    For example ``Languages/Python/`` contains ``Beanie.md`` but no
    ``Python.md`` — so a *virtual* :class:`NoteInfo` for ``Python`` is
    returned, allowing the classifier to match conversations about Python
    and auto-create the stub note later.
    """
    # folder_abs_path → set of lowercase note titles it contains
    folders: dict[str, set[str]] = {}
    for note in notes:
        parent = str(Path(note.path).parent)
        folders.setdefault(parent, set()).add(note.title.lower())

    orphans: list[NoteInfo] = []
    for folder_path, titles in folders.items():
        folder_name = Path(folder_path).name
        # Skip if a root note already exists
        if folder_name.lower() in titles:
            continue
        # Skip the vault root itself
        if Path(folder_path) == vault_path:
            continue

        rel = os.path.relpath(folder_path, vault_path)
        virtual_path = os.path.join(folder_path, f"{folder_name}.md")
        folder_parts = list(Path(rel).parts)

        orphans.append(NoteInfo(
            path=virtual_path,
            title=folder_name,
            aliases=[folder_name],
            tags=[],
            wikilinks=[],
            folder_parts=folder_parts,
            is_virtual=True,
        ))

    return orphans


# ─── Classifier ──────────────────────────────────────────────────────────────────────

def build_index(notes: list) -> tuple:
    """Create an inverted index ``term → [(note_index, weight), …]`` and
    a per-note total-weight map used for score normalization.

    Returns ``(idx, note_total_weight)``.
    """
    idx = {}
    note_total_weight: dict[int, float] = {}

    def _add(term, note_i, weight):
        idx.setdefault(term, []).append((note_i, weight))
        note_total_weight[note_i] = note_total_weight.get(note_i, 0.0) + weight

    for i, note in enumerate(notes):
        # Skip previously-generated session notes
        if "claude-session" in [t.lower() for t in note.tags]:
            continue
        # Skip bare scaffold notes (no aliases and no meaningful tags)
        if not note.aliases and not note.tags:
            continue

        # Title tokens (high weight)
        for tok in tokenize(note.title):
            _add(tok, i, WEIGHT_TITLE)
        # Full title as phrase
        _add(note.title.lower().strip(), i, WEIGHT_TITLE * 1.5)

        # Aliases
        for alias in note.aliases:
            for tok in tokenize(alias):
                _add(tok, i, WEIGHT_ALIAS)
            _add(alias.lower().strip(), i, WEIGHT_ALIAS * 1.5)

        # Tags
        for tag in note.tags:
            t = tag.lower().strip()
            if t and t not in STOPWORDS:
                _add(t, i, WEIGHT_TAG)

        # Folder names
        for part in note.folder_parts:
            p = part.lower().strip()
            if p and p not in STOPWORDS:
                _add(p, i, WEIGHT_FOLDER)

        # Wikilinks
        for link in note.wikilinks:
            for tok in tokenize(link):
                _add(tok, i, WEIGHT_WIKILINK)

    return idx, note_total_weight


def classify(text: str, notes: list, idx: dict, note_total_weight: dict) -> list:
    """Score every note and return :class:`Match` list (descending score).

    Raw scores are normalized by sqrt(total_weight_of_note) to prevent
    content-rich notes from dominating purely by volume.
    """
    import math
    words = tokenize(text)
    text_lower = text.lower()
    scores: dict[int, float] = {}

    # Token matching
    for w in words:
        for note_i, weight in idx.get(w, []):
            scores[note_i] = scores.get(note_i, 0) + weight

    # Phrase matching (multi-word terms, full titles)
    for term, entries in idx.items():
        if (" " in term or len(term) > 12) and term in text_lower:
            for note_i, weight in entries:
                scores[note_i] = scores.get(note_i, 0) + weight * 2

    matches = [
        Match(
            notes[i].path,
            notes[i].title,
            sc / math.sqrt(max(1.0, note_total_weight.get(i, 1.0))),
        )
        for i, sc in scores.items()
        if sc >= MIN_SCORE_THRESHOLD
    ]
    matches.sort(key=lambda m: m.score, reverse=True)
    return matches


def _is_mentioned(term: str, text: str) -> bool:
    """Check if *term* appears in *text* with word boundaries."""
    if not term or len(term) < 3:
        return False
    return bool(re.search(rf"\b{re.escape(term)}\b", text, re.IGNORECASE))


def filter_matches(matches: list, full_text: str, notes: list) -> list:
    """Keep only the top match + notes whose title/alias is explicitly mentioned.

    Rules:
    - The highest-scoring match is **always** included.
    - Every other match is included **only** if its note title or one of its
      frontmatter aliases appears in the conversation text (case-insensitive,
      word-boundary match).

    Examples (given notes Python.md and Docker.md):
    - "Python es un lenguaje de programación"         → [Python]
    - "Python … se puede desplegar usando docker"     → [Python, Docker]
    """
    if not matches:
        return []

    result = [matches[0]]  # top match always included

    if len(matches) <= 1:
        return result

    # Path → NoteInfo lookup for alias checking
    notes_by_path = {n.path: n for n in notes}

    for match in matches[1:]:
        # Check title
        if _is_mentioned(match.note_title, full_text):
            result.append(match)
            continue

        # Check aliases
        note = notes_by_path.get(match.note_path)
        if note and note.aliases:
            if any(_is_mentioned(a, full_text) for a in note.aliases):
                result.append(match)

    return result


# ─── File operations ──────────────────────────────────────────────────────────────────────

def create_stub_note(
    note: NoteInfo,
    transcript_title: str,
    now: datetime,
) -> bool:
    """Create a stub ``.md`` note for a virtual (orphan-folder) match.

    The stub follows the vault's Learning template conventions and already
    includes the first Claude Sessions link.
    """
    if not note.is_virtual:
        return False

    parent = Path(note.path).parent
    parent.mkdir(parents=True, exist_ok=True)

    date_nice = spanish_date(now)
    date_iso = now.strftime("%Y-%m-%d")

    # Derive root tag from vault location (Learning → learning, Projects → projects)
    root_tag = "learning"
    if note.folder_parts:
        root_tag = note.folder_parts[0].lower()

    content = (
        f"---\n"
        f"aliases:\n"
        f"  - {note.title}\n"
        f"tags:\n"
        f"  - {root_tag}\n"
        f"  - claude-generated\n"
        f"date: {date_iso}\n"
        f"---\n"
        f"**Date**: {date_nice}\n"
        f"**Update**: {date_nice}\n"
        f"**Sources**: Claude Code Session\n"
        f"\n"
        f"**Tags:** #{root_tag} #claude-generated\n"
        f"\n"
        f"**Related:** [[Claude Code]]\n"
        f"\n"
        f"---\n"
        f"\n"
        f"## Description\n"
        f"\n"
        f"Write here...\n"
        f"\n"
        f"---\n"
        f"\n"
        f"## Claude Sessions\n"
        f"\n"
        f"- [[{transcript_title}]] \u2014 {date_nice}\n"
        f"\n"
        f"---\n"
    )

    try:
        with open(note.path, "w", encoding="utf-8") as fh:
            fh.write(content)
        return True
    except (IOError, OSError):
        return False

def save_transcript(md: str, note_title: str, now: datetime, session_id: str = "") -> Path:
    """Write the transcript to ``Journal/Claude Sessions/<YYYY-MM-DD>/``.

    Idempotent: if a file with the same session_id already exists today,
    returns the existing path without overwriting it.
    """
    day_dir = SESSIONS_DIR / now.strftime("%Y-%m-%d")
    day_dir.mkdir(parents=True, exist_ok=True)

    if session_id:
        for existing in day_dir.glob("*.md"):
            try:
                if f"session_id: {session_id}" in existing.read_text(encoding="utf-8"):
                    return existing
            except (IOError, OSError):
                pass

    safe = re.sub(r'[<>:"/\\|?*]', "", note_title)
    dest = day_dir / f"{safe}.md"

    with open(dest, "w", encoding="utf-8") as fh:
        fh.write(md)
    return dest


def update_note_with_link(
    note_path: str,
    transcript_title: str,
    now: datetime,
) -> bool:
    """Append a wikilink to *transcript_title* inside an existing note.

    - Creates a ``## Claude Sessions`` section if it doesn't exist.
    - Skips if the link is already present (idempotent).
    """
    try:
        with open(note_path, "r", encoding="utf-8") as fh:
            content = fh.read()
    except (IOError, OSError):
        return False

    wikilink = f"[[{transcript_title}]]"
    if wikilink in content:
        return False  # already linked

    date_str = spanish_date(now)
    link_line = f"- {wikilink} — {date_str}"
    section_hdr = "## Claude Sessions"

    if section_hdr in content:
        # ── Append inside existing section ──────────────────────────────
        lines = content.split("\n")
        insert_at = None
        in_section = False
        for i, ln in enumerate(lines):
            if ln.strip() == section_hdr:
                in_section = True
                continue
            if in_section:
                if ln.strip().startswith("- [["):
                    insert_at = i  # track last link line
                elif ln.strip().startswith("## ") or ln.strip() in ("---", "___"):
                    break
        if insert_at is not None:
            lines.insert(insert_at + 1, link_line)
        else:
            # Section exists but has no links yet — add after header + blank
            hdr_i = next(i for i, l in enumerate(lines) if l.strip() == section_hdr)
            lines.insert(hdr_i + 2, link_line)
        content = "\n".join(lines)
    else:
        # ── Create new section at the end of the file ───────────────────
        stripped = content.rstrip()
        # Remove trailing separator if present so we can re-add after section
        if stripped.endswith("---") or stripped.endswith("___"):
            sep = stripped[-3:]
            stripped = stripped[:-3].rstrip()
            content = stripped + f"\n\n{section_hdr}\n\n{link_line}\n\n{sep}\n"
        else:
            content = stripped + f"\n\n{section_hdr}\n\n{link_line}\n\n---\n"

    try:
        with open(note_path, "w", encoding="utf-8") as fh:
            fh.write(content)
        return True
    except (IOError, OSError):
        return False


# ─── Entry point ─────────────────────────────────────────────────────────────

def main():
    try:
        # 1 — Read hook input (stdin JSON from Claude Code)
        raw = sys.stdin.read()
        if not raw.strip():
            return
        hook = json.loads(raw)
        session_id = hook.get("session_id", "")
        transcript_file = hook.get("transcript_path", "")

        # 2 — Locate the JSONL transcript
        if not transcript_file or not os.path.isfile(transcript_file):
            transcript_file = find_transcript(session_id) if session_id else None
        if not transcript_file:
            return

        # 3 — Parse messages
        messages = parse_transcript(transcript_file)
        if not messages:
            return
        has_user = any(m.role == "user" for m in messages)
        has_assistant = any(m.role == "assistant" for m in messages)
        if not has_user or not has_assistant:
            return

        # 4 — Metadata & title
        meta = extract_session_metadata(transcript_file)
        if not meta["session_id"]:
            meta["session_id"] = session_id
        now = datetime.now()
        title = extract_title(messages)
        transcript_note_title = f"{now.strftime('%Y-%m-%d')} - {title}"

        # 5 — Classify against vault (real notes + orphan-folder virtuals)
        vault_notes = scan_vault(VAULT_PATH)
        orphans = find_orphan_folders(VAULT_PATH, vault_notes)
        vault_notes.extend(orphans)
        idx, note_total_weight = build_index(vault_notes)
        full_text = " ".join(m.text for m in messages)
        all_matches = classify(full_text, vault_notes, idx, note_total_weight)

        # 6 — Filter: top match always + others only if explicitly mentioned
        matches = filter_matches(all_matches, full_text, vault_notes)

        # 7 — Related note titles for the transcript frontmatter
        related_titles = [m.note_title for m in matches[:10]]

        # 8 — Generate & save transcript
        md = generate_transcript_md(messages, meta, title, now, related_titles)
        save_transcript(md, transcript_note_title, now, meta.get("session_id", ""))

        # 9 — Update filtered notes (create stubs for virtuals, link existing)
        notes_by_path = {n.path: n for n in vault_notes}
        for match in matches:
            note = notes_by_path.get(match.note_path)
            if note and note.is_virtual:
                create_stub_note(note, transcript_note_title, now)
            else:
                update_note_with_link(match.note_path, transcript_note_title, now)

    except Exception as exc:
        # Never crash — errors go to stderr for `claude --debug`
        sys.stderr.write(f"[sync_to_obsidian] {exc}\n")

    sys.exit(0)


if __name__ == "__main__":
    main()

```

## Claude Sessions
