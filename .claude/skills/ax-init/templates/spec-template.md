# [Project name]

**Version:** 1.0
**Last updated:** [date]

---

## Constraints
[Non-negotiable rules: stack choices, architectural limits, explicit prohibitions.
Agents treat this zone as read-only with higher authority than the rest of the spec.
Example: "All state lives in the database. No in-memory state between requests."
Example: "No new npm dependencies without operator approval."]

---

## Operator zone
[Vision, features, acceptance criteria.
Written and maintained by the operator.
Agents read this and execute against it. Agents never modify this zone.]

---

## System state
[Feature statuses: planned / in progress / complete.
Architecture descriptions. Current repo structure.
Updated by agents to reflect reality:
- ax-build marks features in-progress when picked up
- ax-build marks features complete when PR merges]
