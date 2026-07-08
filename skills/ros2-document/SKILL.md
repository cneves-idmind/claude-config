---
name: ros2-document
description: "For ROS2 workspaces: writes or updates README.MD and TODO.MD inside each non-submodule package. README.MD covers nodes, their descriptions, and full API (topics, services, actions). Uses one subagent per package to keep context clean between packages."
---

# /ros2-document

Walks every ROS2 package in the workspace that is NOT a git submodule, and for each one spawns a dedicated subagent to write or update its `README.MD` and `TODO.MD`.

## What This Skill Produces

**`README.MD`** (inside each package directory):
- Package name and a short description (derived from `package.xml` and source code)
- List of nodes with a one-line description each
- For each node: full API table (published topics, subscribed topics, service servers, service clients, action servers, action clients) with topic/service name, message type, and a brief description

**`TODO.MD`** (inside each package directory):
- `## Features` — functional improvements or missing capabilities inferred from code (TODOs, incomplete logic, commented-out blocks, placeholder descriptions)
- `## Bugs` — defects or fragile patterns spotted in the code

## Steps

### Step 1 — Find packages and exclude submodules

Run both commands:

```bash
# All packages
find . -name "package.xml" -not -path "*/install/*" -not -path "*/build/*" | sort

# Submodule paths (relative, e.g. "src/idmind_gazebo")
git submodule status 2>/dev/null | awk '{print $2}'
```

Build the list of **non-submodule packages**: keep every package whose directory does NOT start with a submodule path. Compute the **absolute path** of each package directory — subagents need absolute paths.

Print the list before dispatching:
```
Packages to document (submodules excluded):
  /abs/path/to/pkg_a
  /abs/path/to/pkg_b
  ...
```

### Step 2 — Dispatch one subagent per package (all in parallel)

Call the Agent tool **multiple times in the same response** — one call per package. Use `subagent_type="general-purpose"` (needs Write access). Do NOT call them sequentially.

Each subagent receives the prompt below with `PACKAGE_PATH` substituted for the absolute path of that package.

---

**Subagent prompt template** (substitute `PACKAGE_PATH`):

```
You are documenting a single ROS2 package. The package is at: PACKAGE_PATH

Your job is to write two files inside that directory:
  1. README.MD  — node list + full API
  2. TODO.MD    — features and bugs

## What to read

Read these files (use Read and Bash tools):
- package.xml           → package name, description, dependencies
- CMakeLists.txt or setup.py → executable targets / entry points
- Every Python (.py) or C++ (.cpp / .hpp) source file in the package
  (look in src/, include/, scripts/, <package_name>/ subdirectories)
- Any existing README.MD or TODO.MD (update rather than overwrite if present)

Do NOT read files outside PACKAGE_PATH.

## How to find the API

For Python nodes, grep each file for:
  create_publisher        → published topic
  create_subscription     → subscribed topic
  create_service          → service server
  create_client           → service client
  ActionServer(           → action server
  ActionClient(           → action client

For C++ nodes, grep for the same ROS2 patterns:
  create_publisher / create_subscription
  create_service / create_client
  rclcpp_action::create_server / create_client

For each call, read the surrounding code to extract:
  - The topic/service/action name (may be a string literal, a variable, or f-string)
  - The message/service/action type
  - What the callback does (one-line summary)

Parameters declared with declare_parameter() are NOT part of the API tables but can be mentioned in the node description.

## README.MD format

Write the file using this exact structure:

---
# <package name>

<2–4 sentence description of what the package does, derived from package.xml and source code.>

## Dependencies

<brief list of key ROS2 package dependencies from package.xml — skip standard ones like rclpy/rclcpp>

## Nodes

### `<executable_name>` (`<ClassName>`)

<One-line description of what this node does.>

**Published Topics**

| Topic | Type | Description |
|---|---|---|
| `/topic/name` | `pkg/MsgType` | One-line description |

**Subscribed Topics**

| Topic | Type | Description |
|---|---|---|

**Service Servers**

| Service | Type | Description |
|---|---|---|

**Service Clients**

| Service | Type | Description |
|---|---|---|

**Action Servers**

| Action | Type | Description |
|---|---|---|

**Action Clients**

| Action | Type | Description |
|---|---|---|

> Omit any API table section that has zero entries.
> If topic names contain the node namespace (e.g. built with node_prefix), show the relative part and note "(namespaced)".

---

## TODO.MD format

Write the file using this exact structure:

---
# TODO - <package name>

## Features

- [ ] <feature or enhancement — infer from TODO comments, incomplete logic, commented-out code, placeholder descriptions like "TODO: Package description", or obvious missing capabilities>

## Bugs

- [ ] <bug or fragile pattern — e.g. missing error handling on serial reads, hardcoded paths, magic numbers without constants, race conditions>

---

If you find no credible TODOs or bugs, write a single entry: "- [ ] No known issues."

## Rules

- Prefer updating an existing README.MD / TODO.MD over replacing it wholesale — keep sections the user wrote that are not covered by the template.
- Do not invent API entries. If you cannot find where a topic name comes from, write `(dynamic)` in the name column.
- Do not add generic advice like "add unit tests" or "follow PEP8".
- Write concisely — README should be scannable in 60 seconds.
- After writing both files, print a one-line summary: "✓ PACKAGE_NAME: README.MD and TODO.MD written."
```

---

### Step 3 — Collect results

Wait for all subagents to complete. Print a final summary:

```
Documentation complete:
  ✓ pkg_a  — README.MD, TODO.MD
  ✓ pkg_b  — README.MD, TODO.MD
  ...
  ✗ pkg_c  — failed (reason if known)
```

If any subagent failed, offer to re-run it individually.
