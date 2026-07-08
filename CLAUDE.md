# project-init (session start)
At the start of every new conversation on a project, automatically invoke the `project-init` skill (via `Skill` tool with `skill: "project-init"`) BEFORE responding to the user's first message. This loads CLAUDE.md and graphify context, and creates them if missing. Do this once per session, not on every message.

# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.
