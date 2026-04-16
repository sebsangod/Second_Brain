---
aliases:
  - Skills
tags:
  - learning
date: 2026-04-14
---
**Sources**: [Claude Code Skills](https://code.claude.com/docs/en/skills)

**Tags:** #development #LLM #AI

**Related:** [[Claude]], [[Claude Code]], [[Anthropic]]

---

## Description

*Skills* **are folders of instructions, scripts, and resources that** `Claude` **loads dynamically to improve performance on specialized tasks.** Think of them as expertise packages—they teach _`Claude`_ how to complete specific tasks in a repeatable way.

You've already seen *Skills* at work if you've used _`Claude`_ to create `Excel` spreadsheets, PowerPoint presentations, Word documents, or PDFs. Those file creation capabilities are powered by *Skills* running behind the scenes.

---

## Details

#### Types of Skills

There are two categories of _Skills_ you'll encounter:

- **Anthropic** _Skills_ are created and maintained by `Anthropic`. These include enhanced document creation capabilities for `Excel`, Word, PowerPoint, and PDF files. `Anthropic` _Skills_ are available to all paid users and _`Claude`_ invokes them automatically when relevant—you don't need to do anything special to use them.

- **Custom Skills** are ones you or your organization create for specialized workflows and domain-specific tasks. For example, you might create a _skill_ that applies your company's brand guidelines to presentations, structures meeting notes in a specific format, or executes your organization's data analysis workflows.


#### Enabling *Skills*

_Skills_ are currently available as a feature preview for users on Pro, Max, Team, and Enterprise plans. To use _Skills_, you'll need to have `Code` execution and file creation enabled, since _Skills_ require _`Claude`_'s secure sandboxed computing environment to function.

Here's how to enable _Skills_:

1. Navigate to _Settings_ > _Capabilities_
2. Ensure that _Code execution and file creation_ is toggled on
3. Scroll to the _Skills_ section
4. Toggle individual _skills_ on or off as needed

For **Enterprise plans**, organization Owners must first enable both Code execution and _Skills_ in Admin settings before individual members can access them.

For **Team plans**, this feature preview is enabled by default at the organization level.

Once enabled, you'll see available _Skills_ listed in your settings, including `Anthropic's` built-in _Skills_ and any custom _Skills_ you've uploaded.


#### Using _Skills_ in practice

The beauty of _Skills_ is that you typically don't need to think about them. _`Claude`_ handles _skill_ selection automatically based on your request. Here are some examples of prompts that would invoke _Skills_:

- "Create an `Excel` spreadsheet tracking monthly expenses with formulas for totals"
- "Turn this meeting notes document into a PowerPoint presentation"
- "Generate a PDF report summarizing this data"
- "Build a financial model in `Excel` with scenario analysis"

When _`Claude`_ uses a _Skill_, you'll see it mentioned in _`Claude`_'s chain of thought as it works. The output will be a downloadable file you can save to your computer or directly to Google Drive.


#### Security considerations

Because _Skills_ can include executable code, it's important to use them thoughtfully:

- Only install custom _Skills_ from trusted sources
- `Anthropic's` built-in _Skills_ are tested and maintained by `Anthropic`
- Custom _Skills_ you upload are private to your individual account

If you're installing a custom _Skill_ from an external source, review its contents before use to understand what it does.


#### Creating custom _skills_

While `Anthropic's` built-in _Skills_ cover common document creation tasks, the real power of _Skills_ comes from creating your own. Custom _Skills_ let you teach _`Claude`_ your specific workflows, brand guidelines, and ways of working. So _`Claude`_ can apply that knowledge automatically whenever it's relevant.

The easiest way to create a custom _Skill_ is through conversation with _`Claude`_ itself. You don't need to write code or manually create files. _`Claude`_ handles the technical structure for you.

Here's how to create a Skill through conversation:

1. **Start a new chat** and tell _`Claude`_ what you want to create. For example: "I want to create a _skill_ for writing quarterly business reviews" or "I need a _skill_ that applies our brand guidelines to presentations."
2. **Answer** `Claude_'s` **questions.** _`Claude`_ will interview you about your workflow, asking things like: What should this _skill_ do? What makes good output for this type of work? Can you give examples of when you'd use this _skill_?
3. **Upload reference materials** if you have them. Templates, style guides, brand assets, or examples of work you're proud of all help _`Claude`_ understand exactly what you're looking for.
4. **Download your skill.** When finished, _`Claude`_ generates a downloadable ZIP file containing your properly structured _skill_.
5. **Upload to your settings.** Go to _Settings_ > _Capabilities_, scroll to the _Skills_ section, click "Upload _skill_", and select your ZIP file.

Your custom _Skill_ will appear in your _Skills_ list alongside `Anthropic's` built-in _Skills_. From that point forward, _`Claude`_ will automatically invoke it whenever you work on relevant tasks—no manual triggering needed.


#### Skills vs. Projects

You might be wondering—if both _skills_ and projects can be used to give more context to _`Claude`_, when should I use each? Think of it this way: **projects store knowledge, skills perform tasks**.

_Projects_ are knowledge hubs. They hold the reference materials _`Claude`_ needs to understand your work—project specs, meeting notes, research documents. When you upload files to a _project_, _`Claude`_ draws on that information across every conversation within that _project_.

_Skills_ are procedural machines. They encode **how** _`Claude`_ should execute a task—the specific steps, order of operations, and methodology you want followed every time. _Skills_ shine when you have repeatable workflows you want _`Claude`_ to run consistently.

The two features complement each other. A _skill_ can reference knowledge stored in a _project_. Your "customer call prep" _skill_ might pull from customer profiles uploaded to a project's knowledge base. The _project_ provides the **what** (information), the _skill_ provides the **how** (process).

|             | Projects                                                   | Skills                                                         |
| ----------- | ---------------------------------------------------------- | -------------------------------------------------------------- |
| Purpose     | Store knowledge _`Claude`_ references                          | Define processes _`Claude`_ executes                               |
| Best for    | Long-term context, reference materials, team collaboration | Repeatable workflows, multi-step tasks, consistent methodology |
| Example     | Customer hub, research buddy, feedback generator           | Brand guidelines, Blog drafting, PDF creation                  |
| Persistence | Knowledge available across all chats in the project        | Instructions applied when the skill is invoked                 |

___

## Claude Sessions
