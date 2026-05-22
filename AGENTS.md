# Frontend Setup Skill Template — Agent Router

This document is the entrypoint for agents handling local frontend repository setup requests.

Use this repository as a public-safe example of how to structure a setup skill. Replace the
demo project names, repository URLs, domains, package scopes, and troubleshooting notes with
your own project-specific values before publishing a real internal skill.

## Trigger Detection

Route to the appropriate guide when a user says any of the following, or close variants:

| User intent | Route to |
|---|---|
| "setup demo-dashboard" | `demo-dashboard/SKILLS.md` |
| "help me setup demo-dashboard" | `demo-dashboard/SKILLS.md` |
| "install demo dashboard locally" | `demo-dashboard/SKILLS.md` |
| "set up demo dashboard" | `demo-dashboard/SKILLS.md` |
| "get demo-dashboard running" | `demo-dashboard/SKILLS.md` |

## Routing Rules

- Match the user's intent against the table above. Case-insensitive partial matching is fine.
- Load and follow the referenced `SKILLS.md` document for the matched project.
- Do not perform any setup steps before reading the target guide.
- If no project matches, ask the user which project they want to set up and list what is available.
- If a guide asks for a selection, use the product's user-select tool when available. If no user-select tool is available, ask the same options in plain text.

## Available Projects

- **demo-dashboard** -> `demo-dashboard/SKILLS.md`
- **demo-admin** -> guide not yet available; inform the user

