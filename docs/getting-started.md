# Getting started with the open-source Kortix starter

This starter is three files you copy into a Kortix project: a manifest, an agent and a skill. The first three commands bring the project live. The sections after that walk the agent and the skill line by line.

## The first three commands

```bash
# 1 · Install the Kortix CLI
curl -fsSL https://kortix.com/install | bash

# 2 · Scaffold a project, then copy this starter's kortix.yaml, agents/ and skills/ into it
kortix init

# 3 · Push the repo and bring the project live in the cloud
kortix ship
```

`kortix init` writes a `kortix.yaml`, an `agents/` directory and a `skills/` directory. Replace its manifest with this starter's `kortix.yaml`, and drop the two example files below into `agents/` and `skills/activity-digest/`.

## The example agent

`agents/daily-digest.md` has two parts: YAML frontmatter that configures the harness, and a body that is the system prompt.

```markdown
---
description: Turn a day of repository activity into a short digest.
mode: primary
permission:
  edit: allow
  bash: allow
---

You are the daily-digest agent for a Kortix project.

When a session starts, read the connected GitHub repository and collect the
activity since the last digest: merged change requests, new commits, and
opened issues.

Write a one-page digest with these sections:

- What changed
- What is blocked
- What needs a decision

Save it to memory/digests/YYYY-MM-DD.md, then open a change request against
main with the digest. Do not merge the change request yourself.

Keep every line short. Link each claim to the commit or issue it comes from.
```

The frontmatter sets how the harness runs this agent. `description` is required, `mode: primary` marks it as an agent you start a session with, and `permission` scopes what it may do: `edit` and `bash` are set to allow. Add a `model: provider/model` line to pin a model, or leave it out and the session default applies. The body is the system prompt, so it holds the agent's job and the shape of its output.

The agent holds no access on its own. Its `agents.daily-digest` block in `kortix.yaml` grants the GitHub connector and the `activity-digest` skill. Governance is deny by default: an agent with no grants gets no connectors, secrets or skills.

## The example skill

`skills/activity-digest/SKILL.md` is the know-how the agent loads on demand. A skill is a markdown file at `skills/<name>/SKILL.md`, and the harness loads its body only when the agent needs it.

```markdown
---
name: activity-digest
description: How to turn a period of repository activity into a one-page digest with sections for what changed, what is blocked, and what needs a decision.
---

# Activity digest

Use this skill when an agent has to summarize a period of repository activity.

Follow these steps:

1. List the merged change requests, commits and issues since the last digest.
2. Group them under What changed, What is blocked, and What needs a decision.
3. Put one line under each item, and link the commit or issue it comes from.
4. Keep the digest to one page, then save it and open a change request.

Do not invent an item. If a section has nothing in it, write "Nothing this period".
```

The `name` must match the directory, and `description` tells the agent when to reach for the skill. Because the skill is granted in the manifest, only this agent may load it.

## Run it and review what comes back

The trigger in `kortix.yaml` starts the agent on a weekday schedule. To run it by hand first, start a session:

```bash
kortix sessions new --prompt "Run the daily digest and open a change request"
kortix cr ls
```

Session work reaches main through a change request, and merge is default-deny for agents, so you read the diff before anything lands. The agent runs on its own isolated cloud computer and its own branch, and only what it commits survives.

## Where this fits a team

If your team already runs Open WebUI and wants agents that finish a job and hand back a file, see [Open WebUI for teams](https://openwebuialternative.com/open-webui-for-teams). For the full command surface, [Read the docs](https://kortix.com/docs).
