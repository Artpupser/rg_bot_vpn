---
name: testing-skill
description: Token-efficient, autonomous, post-implementation testing policy.
---

## Testing Policy

**Goal:** Minimize token usage while ensuring all delivered changes work correctly.

1. **Test only after completion.** Finish the entire task/specification before running tests. Never test after individual edits, steps, or milestones.
2. **No approval required.** Run necessary tests automatically without asking for confirmation, unless execution is destructive or requires external authorization.
3. **Direct test execution.** The main agent must independently execute tests, analyze failures, and implement necessary fixes. Never spawn, invoke, or delegate testing-related tasks to subagents.
4. **Reuse existing tests.** Do not create new tests unless required to cover a confirmed bug, critical regression risk, or explicit acceptance criterion.
5. **Preserve test coverage.** Never delete existing tests without justification. Remove only obsolete, redundant, or irrelevant tests.
6. **Fix until green.** After implementation, run the smallest relevant test suite, fix detected regressions, and rerun only affected tests until they pass.
7. **Avoid waste.** No repetitive full-suite runs, unnecessary test generation, excessive logs, or repeated analysis of the same failure. Focus on relevant errors and actionable test output.
8. **Final verification.** Perform one relevant final verification after fixes. Report unresolved issues and any skipped checks.

**Execution order:** Implement entire specification → Run tests directly → Analyze failures → Fix failures → Verify → Report.

**Priority:** Correctness > Token efficiency > Test volume.