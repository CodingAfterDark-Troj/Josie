# Josie

**A desktop manager for Claude Code and Codex terminal sessions.**

Josie keeps your coding-agent sessions, terminals, ideas and tasks together in one window. It also chains
agents into **workflows**: repeatable pipelines such as *Read → Research → Plan → Code → Test → Review →
Commit*, where each step is a Claude Code or Codex agent with its own instructions, permissions and model.

- [What Josie does](#what-josie-does)
- [How it works](#how-it-works)
- [Templates that come with Josie](#templates-that-come-with-josie)
- [Step types](#step-types)
- [Example: build in a session, review with a workflow, fix in the session](#example-build-in-a-session-review-with-a-workflow-fix-in-the-session)
- [Build your own subagents](#build-your-own-subagents)
- [Download](#download)

## What Josie does

- **Sessions.** Start, name, group, restart and resume Claude Code, Codex and shell terminals. Show one, two
  or four side by side. Resume past Claude or Codex conversations from their history.
- **Workflows.** Run a template of agent steps against a project folder. Josie hands each step's result to
  the next, pauses where you asked it to, and shows every step's live activity, output and cost.
- **Safe by default.** Steps that change code work in an isolated git worktree, so your checkout is untouched
  until you apply, commit or merge the result. Read-only steps can't edit files.
- **Sessions and workflows work together.** Start a workflow from a session, hand it the conversation so far,
  and have the results sent back to that session when the run ends.
- **Library.** Your own subagents and saved steps, reusable across workflows.
- **Ideas/Tasks.** A task list per project. Send a task to a session or start a workflow from it. Any
  coding agent can add tasks with `Josie.exe --import-ideas=<file>`.
- **Git.** Changes view, diffs, file viewer, git activity and branch/worktree cleanup.

## How it works

A **workflow template** is an ordered list of **steps**. When you run it, Josie starts the step's agent
(`claude` or `codex`) in your project with the step's instructions plus the task and the hand-offs from earlier
steps. Each step ends with a **Hand-off**: the short summary the next step needs.

Every step has settings you can change in **Workflows → Templates**:

| Setting | What it does |
| --- | --- |
| **Agent / model** | Claude Code or Codex, and optionally a specific model. |
| **Access** | `read` (look only), `auto` (may edit, with the agent's normal checks) or `full` (no checks; avoid). |
| **Context** | What the step sees from earlier steps: `all`, only the `previous` step, or `none`. |
| **Approve** | Pause after the step until you approve, edit or reject its result. |
| **Parallel** | Run at the same time as the step before it (for example two reviewers). |
| **Worktree** | For editing steps: work in an isolated git worktree instead of the project folder. |
| **Ask questions** | Let the step stop with *Questions for you* and wait for your answers (on by default for Research and Plan). |
| **Subagents** | Claude steps only: which subagents the step's agent may start. |
| **Allowed tools / extra args** | Fine-grained tool permissions and extra CLI arguments. |

A run can start from:

- **Workflows → Run workflow…**, with a task as text, document paths or a Jira issue key.
- An idea/task in **Ideas/Tasks**.
- The **Workflow** button next to a session's name (see the [example](#example-build-in-a-session-review-with-a-workflow-fix-in-the-session)).

When a review step finds problems, **↩ Send findings back to Code…** runs the Code step and every step after
it again in the same worktree, with the findings as input. Repeat until the review says *ready*, then commit.

## Templates that come with Josie

| Template | Steps | Use it for |
| --- | --- | --- |
| **Feature** | Read → Research → Plan → Code → Test → Review → Commit & push → Merge Into | Taking a ticket, document or description all the way to reviewed, committed code. Pauses after the plan and before committing. Concise hand-offs between steps. |
| **Everyday** | Understand & Plan → Implement & Verify → Review → Commit | Routine changes with fewer steps. Pauses after the plan and before committing. |
| **Review: Claude + Codex in parallel** | Review (Claude) ‖ Review (Codex) → Consolidate | A second opinion on any change. Both agents review independently at the same time, then one step merges their findings, marks issues both found as higher confidence and checks disagreements. Leave the task empty to review your uncommitted changes and current branch. |

You can also import **[review-squad.josie.json](review-squad.josie.json)** (**Workflows → Templates → Import**). It
adds the *Feature with review squad* template and five specialist subagents:

| Subagent | Job |
| --- | --- |
| `code-hunter` | Maps every place a change touches: definitions, callers, config, tests and code to reuse. |
| `data-impact-hunter` | Finds what a change does to stored or exchanged data: compatibility, migrations, loss and leaks. |
| `plan-challenger` | Stress-tests the plan against the real code before anything is written. |
| `code-adversary` | Attacks the change to find real bugs, each with a concrete failure scenario. |
| `security-risk` | Looks for injection, path traversal, secrets exposure and destructive operations, rated by real exploitability. |

## Step types

| Step | Default agent | Access | What it does |
| --- | --- | --- | --- |
| **Read** | Claude | read | Turns the task (text, documents or a Jira issue) into a requirements brief. |
| **Research** | Claude | read | Finds the relevant files, data flow, conventions, tests and risks. |
| **Plan** | Claude | read | Writes an ordered implementation and test plan. |
| **Code** | Claude | auto, worktree | Implements the plan. |
| **Test** | Claude | auto | Runs the project's tests and checks, fixes failures caused by the change. |
| **Review** | Codex | read | Reviews the changes; findings by severity with `file:line`, then a verdict. |
| **Commit & push** | Claude | read | The agent writes the commit message; Josie commits and pushes. |
| **Merge Into** | none | — | Josie merges the run's branch into a branch you choose (an epic or release branch). |
| **Custom** | Claude | read | Anything else: you write the instructions. |

Save a configured step to **Library → Saved steps** to reuse it in other templates. Templates, saved steps
and subagents can be exported and imported as `.josie.json` files, and Josie shows what an import would let
agents do before adding anything.

## Example: build in a session, review with a workflow, fix in the session

This loop keeps one Claude session in charge of the work and uses a workflow as an independent reviewer.

**1. Build the change in a session.** Start a Claude Code session on your project and talk it through:

> Add a "Export as CSV" button to the reports page. It should export the rows currently shown, respect the
> active filters, and name the file `report-<date>.csv`. Ask me anything that's unclear before you start.

Answer its questions, let it implement and test, and keep going until you're happy with the change.

**2. Start a review workflow on the session.** Click the **Workflow** button next to the session's name and
choose **Review: Claude + Codex in parallel** (or *Feature with review squad* if you only want its Review step,
or your own review template). Under **Session**, pick how the run works with the session:

- **Just link** — the run and session are linked so you can jump between them.
- **Report back** — Josie also keeps a report file for the run up to date while it runs.
- **Hand off** — the session's conversation becomes context for the run's first step, so reviewers know
  what you asked for and why. You can preview exactly what is handed off.

**Hand off** is a good fit here: the reviewers see the requirements you discussed, not just the diff. Whichever
you choose, the results go back to the session when the run ends (step 3). With Hand off, untick **Stop the
session after handing off** so the session is still running to take them. Leave the task empty to review the
uncommitted changes and the branch, or write what to focus on:

> Review the CSV export. Focus on filter handling, large result sets and file names on Windows.

Claude and Codex review at the same time, then **Consolidate** merges their findings into one list ordered by
severity, with `file:line`, a suggested fix and who found each issue.

**3. The parent session fixes the findings.** When the run ends, Josie waits until the session is idle at its
input box, then types a line pointing at the run's full report and presses Enter. The session reads the report
and starts fixing. The run shows **Sent to parent**, then **Parent processed** once the session has worked
through it. You can turn automatic sending off under **Settings → Workflows** and use **Send results to session**
on the run instead.

**4. Repeat.** Run the review again on the same session until the verdict is *ready*, then commit from the
session or with a Commit & push step.

> Tip: If you'd rather the workflow fix its own findings, use a template with a Code step (for example
> **Feature**) and click **↩ Send findings back to Code…** when the review is done.

## Build your own subagents

A subagent is a Claude Code helper with its own instructions and tools that a **Claude** step can start for a
focused job: hunting down every caller, attacking a change, checking a plan. Steps only get the subagents you
pick for them, and in read-only steps a subagent only gets read tools, whatever it lists.

**Create one** in **Workflows → Library → Subagents → + New subagent**:

| Field | Notes |
| --- | --- |
| **Name** | Lowercase letters, numbers and hyphens, e.g. `test-gap-finder`. Can't reuse a built-in name. |
| **Description** | When Claude should use it, and what to give it. Be specific: Claude decides from this. |
| **Instructions** | The subagent's prompt: its job, how to work, rules, and the exact shape of its final message (that message is all the calling step sees). |
| **Tools** | Space-separated, e.g. `Read Grep Glob`. Leave empty for all tools. |
| **Model** | Optional. Empty uses the default. |

Or click **Help** in the Library and Josie opens a Claude Code or Codex session that interviews you, writes the
subagent for you and saves a file to import.

**Use it in a workflow.** Open a template, select a Claude step, and tick the subagent under **Subagents**. Then
tell the step when to call it in the step's instructions. For example, a Review step:

```text
Review the code changes against the brief and plan.
First collect the changes yourself: git status, git diff, and this branch's commits that aren't on main.
Your subagents can't run git, so they only see what you give them.
Start test-gap-finder and code-adversary at the same time. Give each the brief, the changed files and the diff.
Check every finding they return against the code yourself. Drop findings you can't confirm and merge duplicates.
Do not change any files.
Hand-off: findings ordered by severity, each with file:line, a failure scenario, a fix, and who found it.
End with a verdict: ready, or needs changes.
```

**An example subagent** you can paste into a `.josie.json` file and import from the Library:

```json
{
  "format": "josie-workflows",
  "version": 1,
  "subagents": [
    {
      "name": "test-gap-finder",
      "description": "Finds behaviour in a change that no test covers, and tests that would still pass if the code were wrong. Give it the brief and the diff or changed files. Read-only.",
      "tools": "Read Grep Glob",
      "model": "",
      "prompt": "You find gaps in test coverage for a code change. You don't change files.\n\nFor each changed function or behaviour: find the tests that exercise it (Grep for its name and its callers). Note behaviour with no test, edge cases and error paths no test reaches, and tests that would still pass if the code were wrong.\n\nRules: cite file:line for every claim. Only report gaps that could hide a real bug.\n\nFinal message: each gap with file:line, the behaviour that isn't tested, and the test to add. If coverage is good, say so."
    }
  ]
}
```

Tips for good subagents:

- **One job each.** A focused subagent beats a general one, and you can run several in parallel.
- **Read-only where you can.** `Read Grep Glob` is enough for research and review. Let the calling step run git
  and pass the results in.
- **Demand evidence.** Ask for `file:line` and concrete scenarios, and tell it that "nothing found" is a valid
  answer, so it doesn't invent problems.
- **Have the step verify.** Tell the calling step to check what subagents return before relying on it.

## Download

Get the latest installer from **[Releases](../../releases/latest)**: download `Josie-Setup-<version>.exe` and run it.

Josie updates itself: when a new version is available it downloads in the background and installs the next time you restart Josie.

### Requirements

- Windows 10 or 11 (64-bit)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and/or [Codex](https://github.com/openai/codex), installed and signed in with your own account
- Git (optional), for the Git features and worktrees

## Help and feedback

- **Bugs and feature requests:** open an [issue](../../issues).
- **Everything else:** troj@codingafterdark.com

## License

Josie is free to download and use, but it is **proprietary software, not open source**. Its use is governed by the [Josie End User License Agreement](LICENSE). In short, you may not copy, redistribute, modify or reverse engineer it.

This repository contains release downloads only. The source code is not public.

See the [Privacy Policy](PRIVACY.md) for what Josie collects. In short: your code and files stay on your computer, and signing in, which is optional, shares your account details and app usage with us.

Josie includes open-source components, each under its own license. See [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).

Copyright © 2026 Douglas Trojanowski. All rights reserved. Josie is a trademark of Douglas Trojanowski.
