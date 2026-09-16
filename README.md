# TaskP3 Skill

Agent skill for managing TaskP3 tasks through the TaskP3 MCP server or the public `p3` CLI.

## Install

```bash
npx skills add Raidon-Inc/taskp3-skill -a cursor
npx skills add Raidon-Inc/taskp3-skill -a windsurf
npx skills add Raidon-Inc/taskp3-skill -a github-copilot
npx skills add Raidon-Inc/taskp3-skill -a cline
```

Any supported agent:

```bash
npx skills add Raidon-Inc/taskp3-skill
```

## Requirements

- `p3` CLI ≥ 1.0.0 with its matching API deployment, installed and authenticated (`npm i -g @taskp3/cli`), or a connected TaskP3 MCP server
- Access to the relevant TaskP3 organization/project

## Task organization

Tasks belong directly to projects. Feature trees are retired; use project-wide task listing and search.

`SKILL.md` and `references/` are a mirror of the CLI-bundled skill at
`packages/p3-cli/skills/p3-task-tracking` in `Raidon-Inc/p3`. Propose changes there;
CI pushes them here when they reach `main`. Direct edits are overwritten on the next sync.

## Safety

This repo contains only public workflow instructions. It does not include private TaskP3 source code, credentials, internal endpoints, or customer data.
