---
name: project-init
description: "Run automatically at the start of every project session. Checks for CLAUDE.md and graphify-out/graph.json, creates them if missing (via /init and /graphify), then reads both to establish project context for the session."
---

# /project-init

Loads project context at the start of a session: ensures CLAUDE.md and the graphify knowledge graph exist, creates them if not, then reads both to brief you on the project.

## When This Skill Runs

This skill is invoked automatically at the start of every new conversation on a project (via the global CLAUDE.md instruction). It can also be invoked manually with `/project-init`.

## Steps

### Step 1 — Detect what exists

Run the following check silently:

```bash
echo "CLAUDE_MD=$([ -f CLAUDE.md ] && echo exists || echo missing)"
echo "GRAPHIFY=$([ -f graphify-out/graph.json ] && echo exists || echo missing)"
```

### Step 2 — Create missing artifacts

**If CLAUDE.md is missing:** invoke the `init` skill (via `Skill` tool with `skill: "init"`) to generate a CLAUDE.md from the codebase. Wait for it to complete.

**If graphify-out/graph.json is missing:** invoke the `graphify` skill (via `Skill` tool with `skill: "graphify"`) to build the knowledge graph. Wait for it to complete.

If both are missing, create CLAUDE.md first (it's faster), then run graphify.

If both already exist, skip this step entirely.

### Step 3 — Read and load context

Read `CLAUDE.md` from the project root. Extract:
- Project purpose (1-2 sentences)
- Key architectural decisions or constraints
- Important conventions or rules for Claude to follow

If `graphify-out/GRAPH_REPORT.md` exists, read it and extract:
- God Nodes (most connected concepts)
- Community names and what they represent
- Suggested Questions

### Step 4 — Present a session brief

Print a compact session brief in this format:

```
Project: <name from CLAUDE.md or directory name>
─────────────────────────────────────────────────
<1-2 sentence project description>

Key conventions: <bullet list of top 3-5 rules/conventions from CLAUDE.md>

Graph: <N> nodes · <M> communities: <comma-separated community names>
God nodes: <top 3 god nodes>

Ready.
```

If graphify-out does not exist (e.g., it was just created and you skipped building it), omit the Graph lines.

Do not dump the full CLAUDE.md or GRAPH_REPORT.md — only the summary above. The goal is a fast, dense brief that sets context in 10-15 lines.

## Notes

- Never block on graphify if it takes too long — if the user has a large corpus and graphify is not yet built, mention it and offer to build it in the background.
- If CLAUDE.md was just created by the `init` skill, briefly confirm it was created before reading it.
- If both existed already, open with "Context loaded from existing CLAUDE.md and graph." to signal this was a fast path.
