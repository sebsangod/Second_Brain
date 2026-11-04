---
aliases:
  - Plugins
tags:
  - learning
  - dev/ai/llm
date: 2026-05-12
---
**Sources**: [Source]()

**Related:** [[Large Language Model]], [[Skills]], [[Subagents]], [[Hooks]], [[Development/AI/LLMs/MCP/MCP|MCP]]

---

## Description

A plugin is one installable unit. It bundles everything you'd otherwise share by hand: ``skills``, ``subagents``, ``hooks``, and ``MCP`` server configs, plus the longer tail of stuff like language server protocol servers, background monitors, themes, and a slice of _settings.json_. One version, one install.

___

## Details

### Installation

Where the plugin lives decides how you install it. Inside a session, you can install one directly by name:

```bash title:bash
/plugin install github@claude-plugins-official
```

The command opens the plugin's details first, so you can review it and choose a scope. Then Claude Code installs it and prints a short summary. Either it says the plugin is already active in this session, or it mentions _/reload-plugins_ and Claude Code runs that reload for you. You step in only if the reload warns that your next message would re-read the whole conversation: run _/reload-plugins --force_ to activate the plugin anyway.

### Adding a marketplace for your team

For a team, the better move is to add a private marketplace once. A marketplace is a shared source that plugins resolve through:

```bash title:bash
/plugin marketplace add your-org/claude-plugins
```

Call it whatever you want. Once it's added, every install after that resolves through it. You get centralized discovery, version tracking, and updates in one place instead of scattered across everyone's laptop.

You can browse what's available from the Discover tab. It lists the plugins on your marketplaces so you can search and pick.

---

## Examples

### Main

#### [Context7](https://context7.com/), [GitHub](https://github.com/upstash/context7)
Get the latest documentation and code into Cursor, Claude, or other LLMs

#### [Everything Claude Code](https://github.com/affaan-m/everything-claude-code)
The performance optimization system for AI agent harnesses. From an Anthropic hackathon winner.

### Development

#### [code-review-graph](https://github.com/tirth8205/code-review-graph)
Seamlessly preserves context across sessions by automatically capturing tool usage observations, generating semantic summaries, and making them available to future sessions. This enables Claude to maintain continuity of knowledge about projects even after sessions end or reconnect.

#### [The Agency: AI Specialists Ready to Transform Your Workflow](https://github.com/msitarzewski/agency-agents)
**A complete AI agency at your fingertips** - From frontend wizards to Reddit community ninjas, from whimsy injectors to reality checkers. Each agent is a specialized expert with personality, processes, and proven deliverables.

### Context Memory

#### [Claude Mem](https://github.com/thedotmack/claude-mem)
Seamlessly preserves context across sessions by automatically capturing tool usage observations, generating semantic summaries, and making them available to future sessions. This enables Claude to maintain continuity of knowledge about projects even after sessions end or reconnect.

#### [Planning with Files](https://github.com/othmanadi/planning-with-files)
A Claude Code plugin that transforms your workflow to use persistent markdown files for planning, progress tracking, and knowledge storage — the exact pattern that made Manus worth billions.

### Videogames

#### [Claude Code Game Studios](https://github.com/Donchitos/Claude-Code-Game-Studios)
Building a game solo with AI is powerful — but a single chat session has no structure. No one stops you from hardcoding magic numbers, skipping design docs, or writing spaghetti code. There's no QA pass, no design review, no one asking "does this actually fit the game's vision?"

---

## Claude Sessions
