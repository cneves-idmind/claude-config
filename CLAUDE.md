# project-init (session start)
At the start of every new conversation on a project, automatically invoke the `project-init` skill (via `Skill` tool with `skill: "project-init"`) BEFORE responding to the user's first message. This loads CLAUDE.md and graphify context, and creates them if missing. Do this once per session, not on every message.

# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.

# Delegation policy
For substantial tasks (multi-file implementation, package-wide documentation, long builds/verification, parallelizable work), plan first, then delegate execution to subagents — Sonnet (`implementer` agent) for code changes, Haiku (`scout` agent) for mechanical search/summarize tasks. Keep quick interactive work (single-file edits, git operations, config tweaks, Q&A) in the main conversation. When delegating, give subagents complete, self-contained briefs — they start with no context.
