# Frontend Setup Skill Template

This is a public-safe demo of a repository setup skill.

It intentionally mirrors a common internal-agent structure:

```text
.
├── AGENTS.md
└── demo-dashboard
    ├── SKILLS.md
    └── docs
        └── troubleshooting.md
```

## What This Demonstrates

- A root `AGENTS.md` router that maps user intent to project-specific guides.
- A project `SKILLS.md` with a deterministic setup workflow.
- Progressive disclosure through `docs/troubleshooting.md`.
- User-select decision points for clone folder, install mode, port, run script, and proxy options.
- Safety rules for private package tokens, env files, system files, nginx, and sudo.

## How to Adapt

1. Copy `demo-dashboard/` and rename it to your project.
2. Update the repo URL, default clone folder, default domain, and default port.
3. Replace `@example-org` with your private package scope, if any.
4. Add project-specific failure modes to `docs/troubleshooting.md`.
5. Add the project trigger phrases to `AGENTS.md`.

Keep secrets, real tokens, and private infrastructure details out of public examples.

