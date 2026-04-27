---
aliases:
  - Learning
tags:
  - learning
  - dev/ai/llm
  - dev/backend
date: 2026-03-26
---
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Related:** [[Claude Code]]

---

## Description

The Claude Code _SDK_ lets you run Claude Code programmatically from within your own applications and scripts. It's available for `TypeScript`, `Python`, and via the _CLI_, giving you the same Claude Code functionality you use at the terminal but integrated into larger workflows.

![[sdk.png]]

**The** _SDK_ **runs the exact same Claude Code you're already familiar with.** It has access to all the same tools and will use them to complete whatever task you give it. This makes it particularly powerful for automation and integration scenarios.

---

## Key concepts

- Runs Claude Code programmatically
- Same Claude Code functionality as the terminal version
- Inherits all settings from Claude Code instances in the same directory
- Read-only permissions by default
- Most useful as part of larger pipelines or tools

---

## Details

### Basic Usage

Here's a simple `TypeScript` example that asks `Claude` to analyze code for duplicate queries:

```typescript title:integration.ts
import { query } from "@anthropic-ai/claude-code";

const prompt = "Look for duplicate queries in the ./src/queries dir";

for await (const message of query({ prompt, })) {
    console.log(JSON.stringify(message, null, 2));
}
```

When you run this code, you'll see the raw conversation between your local Claude Code and the `Claude` language model, message by message. The final message contains `Claude's` complete response.

### Permissions and Tools

**By default, the** _SDK_ **only has read-only permissions.** It can read files, search directories, and perform grep operations, but **it cannot write, edit, or create files.**

To enable write permissions, you can add the _allowedTools_ option to your query:

```typescript title:integration.ts
for await (
    const message of query({
        prompt,
        options: {
            allowedTools: ["Edit"]
        }
    })
) {
    console.log(JSON.stringify(message, null, 2));
}
```

Alternatively, you can configure permissions in your settings file within the _.claude_ directory for project-wide access.

### Practical Applications

The Claude Code _SDK_ shines when integrated into larger development workflows. Consider using it for:

- Git hooks that automatically review code changes
- Build scripts that analyze and optimize code
- Helper commands for code maintenance tasks
- Automated documentation generation
- Code quality checks in `CI/CD` pipelines

The _SDK_ essentially lets you add `AI-powered` intelligence to any part of your development process where programmatic access would be valuable.

---

## Claude Sessions
