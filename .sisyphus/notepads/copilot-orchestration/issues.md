# Issues & Gotchas

This notepad tracks problems encountered and their solutions.

---

## 2026-03-05 - Task F2 Must-NOT Guardrail Violation

- Issue: Strict forbidden-pattern scan found `TODO` in `.github/agents/implementer.agent.md:99`.
- Detection command: `grep -RIn --include="*.agent.md" -E "TODO|FIXME|placeholder" .github/agents`
- Impact: F2 QA Scenario "No Must NOT Have violations" failed.
- Workaround used: Recorded as compliance failure in `.sisyphus/evidence/task-F2-must-not-have.md` with file:line evidence.
