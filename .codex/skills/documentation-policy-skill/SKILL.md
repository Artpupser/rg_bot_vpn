---
name: documentation-policy-skill
description: Enforces Git-first development history, prevents unsolicited documentation, and minimizes documentation-related token overhead.
---

# Documentation Policy
**Goal:** Minimize token usage, documentation drift, and maintenance overhead.

## Rules
1. **Git is the source of history.** Use concise, meaningful Git commits to record development changes. Never generate separate development logs or changelogs.
2. **Graphify is the architecture reference.** Use Graphify for architecture context and relationships when available. Do not duplicate its information in Markdown.
3. **Scalar is the API reference.** Maintain API definitions through the existing API schema and Scalar integration. Never generate separate API documentation unless explicitly requested.
4. **No unsolicited documentation.** Never create, modify, regenerate, or delete documentation files without an explicit developer request. This includes `README.md`, `CHANGELOG.md`, `dev_log.md`, architecture notes, audits, progress reports, and other documentation artifacts.
5. **Release-only documentation.** Full documentation may be generated only when the developer explicitly requests it for a specific application release/version. A release event alone does not authorize documentation generation.
6. **No automatic documentation directories.** Never create `docs/`, version folders, documentation snapshots, or release documentation structures without explicit instructions.
7. **Protect existing documentation.** Treat all existing documentation as read-only unless the developer explicitly authorizes its modification.
8. **No documentation overhead.** Avoid unnecessary documentation scans, speculative summaries, repeated Markdown synchronization, and redundant context generation.
9. **Minimal reporting.** Provide a concise final response with completed changes, test results, and Git commit references. Do not save reports to files.

## Execution
Implement → Test → Fix → Verify → Commit → Brief final response.

**Priority:** Working code > Git history > Graphify / Scalar > Developer-requested documentation.

**Exception:** Required executable configuration, migrations, API schemas, and source-code comments are implementation artifacts, not standalone documentation. Modify them when necessary for the task.