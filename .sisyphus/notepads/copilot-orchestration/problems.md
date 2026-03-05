# Unresolved Problems

This notepad tracks blockers and unresolved issues.

---

## 2026-03-05 - Open Compliance Failure (F2)

- Unresolved: Must-NOT guardrail requires zero TODO/FIXME/placeholder matches in agent files.
- Current violation: `.github/agents/implementer.agent.md:99` contains `TODO` token.
- Required follow-up: edit agent wording to remove the literal TODO token, then re-run F2 Scenario 2 and refresh evidence.
