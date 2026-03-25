# Claude Code Setup Guide

Step-by-step instructions to install all plugins, MCP servers, and skills.

## TL;DR

This guide sets up a fully loaded Claude Code environment with **superpowers** (brainstorming, TDD, debugging, plan execution, code review), **frontend-design**, **feature-dev**, **code-simplifier**, and **context7** plugins, plus MCP servers for sequential thinking and library docs, a custom status line, and specialized agents — all working together so Claude Code can handle end-to-end development workflows out of the box.

---

## For Users

### ccstatusline

A lightweight status line for Claude Code that displays real-time session info — model name, token usage, and cost — at the bottom of your terminal.

```bash
npx ccstatusline@latest
```

Run it once to see it in action, or configure it as your permanent status line:

```
/config set statusLine.type command
/config set statusLine.command "npx -y ccstatusline@latest"
/config set statusLine.padding 0
```

### Useful Resources

- [100+ ready-to-use agent personas](https://github.com/msitarzewski/agency-agents) — drop into `~/.claude/agents/` for specialized subagents

### Understanding the `.claude` Folder

Before diving into this setup guide, it helps to understand what you're configuring. The `.claude` folder is Claude Code's control center — it holds your instructions, custom commands, permission rules, and session memory. There are actually **two** `.claude` directories: one in your project (team config, committed to git) and one in your home directory `~/.claude/` (personal preferences, machine-local state).

Here's what lives where:

| File / Folder | Purpose |
|---|---|
| **`CLAUDE.md`** | Main instruction file — Claude reads it every session. Keep under 200 lines: build/test commands, architecture decisions, conventions, gotchas |
| **`CLAUDE.local.md`** | Personal overrides (auto-gitignored) |
| **`.claude/settings.json`** | Permissions — `allow` (no confirmation), `deny` (blocked entirely), everything else asks first |
| **`.claude/settings.local.json`** | Personal permission overrides (auto-gitignored) |
| **`.claude/rules/`** | Modular instruction files that load alongside CLAUDE.md. Support path-scoped rules via YAML frontmatter |
| **`.claude/commands/`** | Custom slash commands — each `.md` file becomes `/project:command-name`. Supports shell output injection via `` !`command` `` and `$ARGUMENTS` |
| **`.claude/skills/`** | Auto-invoked workflows — Claude triggers them based on the task context. Each skill is a folder with `SKILL.md` + supporting files |
| **`.claude/agents/`** | Specialized subagent personas with their own system prompt, tool access, and model preference |
| **`~/.claude/CLAUDE.md`** | Global instructions across all projects |
| **`~/.claude/commands/`** | Personal commands available everywhere (show as `/user:command-name`) |
| **`~/.claude/skills/`** | Personal skills available everywhere |
| **`~/.claude/agents/`** | Personal agents available everywhere |
| **`~/.claude/projects/`** | Session history + auto-memory per project |

**Getting started:** Run `/init` to generate a starter `CLAUDE.md`, add `.claude/settings.json` with allow/deny rules, create a command or two for your most common workflows, and split instructions into `.claude/rules/` as your config grows.

---

## Prerequisites

- Claude Code CLI installed and authenticated
- Node.js 22+ installed

---

## Step 1: Install Plugins

Open Claude Code in any project and run these commands one at a time.

### Superpowers (via Plugin Marketplace)

Register the marketplace first, then install superpowers from it:

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### Other Plugins

```
/plugin install frontend-design
/plugin install code-review
/plugin install feature-dev
/plugin install code-simplifier
/plugin install context7
```

After installing all plugins, reload them:

```
/reload-plugins
```

### What each plugin does

| Plugin | Source | What it provides |
|--------|--------|-----------------|
| **superpowers** | superpowers-marketplace | Brainstorming, plan writing, plan execution, subagent-driven development, TDD, debugging, code review, git worktrees, skill authoring |
| **frontend-design** | claude-plugins-official | High-quality frontend UI generation (avoids generic AI aesthetics) |
| **code-review** | claude-plugins-official | Structured code review for PRs |
| **feature-dev** | claude-plugins-official | Guided feature development with codebase analysis and architecture focus |
| **code-simplifier** | claude-plugins-official | Reviews and simplifies code for clarity and maintainability |
| **context7** | claude-plugins-official | MCP server for fetching up-to-date library documentation and code examples |

---

## Step 2: Fix Superpowers Deprecated Commands (Optional)

Superpowers v5.0.5 has deprecated `commands/` stubs that intercept the newer `skills/`. If `/superpowers:brainstorm` shows a deprecation message instead of running, delete the stubs:

**Windows (PowerShell):**
```powershell
$cachePath = "$env:USERPROFILE\.claude\plugins\cache\claude-plugins-official\superpowers"
$version = Get-ChildItem $cachePath | Select-Object -First 1 -ExpandProperty Name
Remove-Item "$cachePath\$version\commands" -Recurse -Force -ErrorAction SilentlyContinue
```

**macOS/Linux:**
```bash
rm -rf ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/commands/
```

Then run `/reload-plugins`.

---

## Step 3: Verify Installation

Run `/skills` to see all available skills. You should see:

### Superpowers Skills
- `/superpowers:brainstorming` — Collaborative design before implementation
- `/superpowers:writing-plans` — Create implementation plans from specs
- `/superpowers:executing-plans` — Execute plans with checkpoints
- `/superpowers:subagent-driven-development` — Dispatch subagents per task
- `/superpowers:test-driven-development` — TDD workflow
- `/superpowers:systematic-debugging` — Structured debugging
- `/superpowers:requesting-code-review` — Request code review
- `/superpowers:receiving-code-review` — Handle review feedback
- `/superpowers:dispatching-parallel-agents` — Run independent tasks in parallel
- `/superpowers:using-git-worktrees` — Isolated git worktrees for features
- `/superpowers:finishing-a-development-branch` — Merge/PR/cleanup options
- `/superpowers:verification-before-completion` — Verify before claiming done
- `/superpowers:writing-skills` — Create custom skills

### Other Skills
- `/frontend-design:frontend-design` — Build polished frontend interfaces
- `/code-review:code-review` — Review a pull request
- `/feature-dev:feature-dev` — Guided feature development
- `/simplify` — Simplify and refine code

---

## Step 4: MCP Servers

### Sequential Thinking

Gives Claude a tool for breaking down complex problems step by step. Adds a `sequential_thinking` tool that Claude can use internally for multi-step reasoning.

**macOS/Linux:**
```bash
claude mcp add sequential-thinking --scope user -- npx -y @modelcontextprotocol/server-sequential-thinking
```

**Windows:**
```bash
claude mcp add sequential-thinking --scope user -- cmd /c npx -y @modelcontextprotocol/server-sequential-thinking
```

> **Windows note:** `npx` is a `.cmd` script on Windows and needs the `cmd /c` wrapper to execute properly as an MCP server. If you skip this, `/doctor` will show a warning. This applies to any MCP server that uses `npx`.

This registers the server globally (all projects). You'll need to restart Claude Code after adding it.

### Context7 (via plugin)

The **context7** plugin (installed in Step 1) automatically registers an MCP server for fetching up-to-date library documentation and code examples. No manual configuration needed.

### Verify MCP Servers

After restarting Claude Code, you should see MCP tools available. You can check by asking Claude to use `sequential_thinking` or to look up library docs.

---

## Step 5: Status Line

The status line shows useful info (model, tokens, cost) at the bottom of the Claude Code terminal.

This is already configured in `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "npx -y ccstatusline@latest",
    "padding": 0
  }
}
```

If it's not set up yet, run inside Claude Code:

```
/config set statusLine.type command
/config set statusLine.command "npx -y ccstatusline@latest"
/config set statusLine.padding 0
```

Or manually add the `statusLine` block to `~/.claude/settings.json`.

---

## Step 6: Agents

The plugins register specialized agents automatically. After `/reload-plugins`, you should see ~11 agents in the reload output. These are used internally by the skills (e.g., spec reviewers, code quality reviewers, implementers) and don't need manual setup.

---

## Quick Reference

### Common Workflows

**Start a new feature:**
```
/superpowers:brainstorming <describe what you want>
```
This guides you through design → spec → plan → implementation.

**Execute an existing plan:**
```
/superpowers:subagent-driven-development <path to plan>
```

**Review code before merging:**
```
/superpowers:requesting-code-review
```

**Debug a failing test:**
```
/superpowers:systematic-debugging
```

**Build a UI component:**
```
/frontend-design:frontend-design <describe the component>
```

### Plugin Management

```
/plugin                    # Install/uninstall/update plugins
/plugin list               # Show installed plugins
/reload-plugins            # Reload after changes
/skills                    # Show all available skills
```
