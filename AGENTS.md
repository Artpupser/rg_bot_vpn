## Launch project
use podman-compose driver

## Centralized Systems Ownership
Each project/package owns its centralized systems registry
in its nearest project-level AGENTS.md.

## Documentation/Audit/Logs Policy
Follow `.codex/skills/documentation-policy-skill/SKILL.md` for all development tasks
- Git: sole source of development history
- Graphify: architecture context and relationships
- Scalar: API reference via the existing API schema
- Never generate release documentation automatically. It requires an explicit request and target version
- Keep final reports concise and in chat only
- Do not create documentation directories or snapshots
- Preserve existing documentation as read-only

## Developing OR Backend
Follow `.codex/skills/backend-skill/SKILL.md`

## Testing
Follow `.codex/skills/testing-skill/SKILL.md`
- Complete the entire specification before testing
- Reuse existing tests, fix all detected regressions, and avoid unnecessary test creation or repeated runs

## Conditions
- don't create README.md

## Centralized Systems
| Name | Purpose | Canonical path |
|------|---------|----------------|
