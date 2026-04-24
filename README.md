# Token-Efficient OpenCode Agent Setup

A multi-agent configuration that maximizes coding capability while minimizing API costs by routing tasks to the cheapest model that can handle them.

## Architecture

```
User
 └── build (Sonnet) ← primary orchestrator / reasoning
      ├── @explorer (Haiku) ← read-only file search & codebase questions
      ├── @writer  (Haiku) ← file creation & boilerplate output
      ├── @executor (Haiku) ← shell commands, tests, builds
      └── @reviewer (Sonnet) ← high-quality code review (use sparingly)
```

**Rule of thumb:** Sonnet thinks, Haiku acts.

## Cost Model

| Agent     | Model   | Relative Cost | Use for                          |
|-----------|---------|---------------|----------------------------------|
| build     | Sonnet  | $$            | Reasoning, orchestration         |
| plan      | Sonnet  | $$            | Architecture, planning           |
| reviewer  | Sonnet  | $$            | Critical code review             |
| writer    | Haiku   | ¢             | File writes, scaffolding         |
| executor  | Haiku   | ¢             | Shell commands, tests, CI tasks  |
| explorer  | Haiku   | ¢             | Read files, search codebase      |
| compaction| Haiku   | ¢             | Context compaction (auto)        |

## Installation

### Option A: Project-specific (recommended)
Copy to your project root:
```
your-project/
├── opencode.json       ← copy the config here
└── prompts/            ← copy the prompts folder here
```

### Option B: Global (applies to all projects)
```bash
mkdir -p ~/.config/opencode/prompts
cp opencode.json ~/.config/opencode/opencode.json
cp prompts/* ~/.config/opencode/prompts/

# Update prompt paths in opencode.json to absolute paths:
# "{file:/home/you/.config/opencode/prompts/build.txt}"
```

## Usage

### Normal development
Just use the `build` agent (default). It will automatically delegate to cheap subagents.

```
> Add a /health endpoint to the Express app
```

The build agent will:
1. Dispatch @explorer to find the app entry point
2. Reason about where/how to add the endpoint
3. Dispatch @writer to create or edit the file
4. Dispatch @executor to run tests

### Planning mode
Switch to `plan` with Tab, or invoke directly:
```
> @plan Design the authentication system for this app
```

### Manual subagent invocation
You can always invoke subagents directly for specific tasks:
```
> @explorer find all files that import from './auth'
> @executor run the test suite and show failures only
> @writer create a new file src/utils/retry.ts with exponential backoff logic
> @reviewer review the changes in src/auth/
```

## Tuning

### Increase quality (higher cost)
Change `writer` and `executor` to Sonnet when working on critical code:
```json
"writer": {
  "model": "anthropic/claude-sonnet-4-20250514"
}
```

### Decrease cost further
Set `steps` limits on subagents to prevent runaway iteration:
```json
"executor": {
  "steps": 10
}
```

### Add a debugger agent
Create `.opencode/agents/debugger.md`:
```markdown
---
description: Investigates bugs and traces errors without making changes
mode: subagent
model: anthropic/claude-haiku-4-20250514
permission:
  edit: deny
  bash:
    "*": deny
    "cat *": allow
    "grep *": allow
    "git log *": allow
---

You are a debugging subagent. Trace errors, find root causes, read logs.
Never modify files. Report findings clearly with file:line references.
```

## Tips

- **Let the orchestrator orchestrate.** Don't micromanage which subagent does what — describe the task and let `build` delegate.
- **Use @plan before @build for large features.** Planning is cheap and prevents expensive wrong-direction work.
- **@reviewer is Sonnet** — use it for important PRs, not every change.
- **Context compaction is free** — it runs automatically on Haiku to keep costs low on long sessions.
# opencode
