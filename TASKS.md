# Remaining work — Apollo LinkedIn

Read [AGENTS.adoc](AGENTS.adoc) before working. Behavior is defined in
[SPEC.adoc](docs/SPEC.adoc); task details are in
[EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md).

Keep only remaining work here. Remove a line only after implementation **and
verification**. Keep blocked lines with the missing evidence. Git preserves
history. Task IDs are stable; do not renumber after removing an item.

- [ ] BOOT — Verify documentation, public MIT repository publication, and the execution handoff. No application implementation is part of this bootstrap.
- [ ] T00 — Verify account access and provider contracts using [ACCESS.adoc](docs/ACCESS.adoc). Regional eligibility is confirmed; API credentials, Gmail consent, payload shapes, current sequence membership, LinkedIn freshness, and Sales Navigator message coverage are still unverified. Owner-controlled setup is required for live checks; independent offline tasks can proceed.
- [ ] T01 — Bootstrap the Python package, test/lint/build commands, lockfile, and minimal CLI help with test-first behavior.
- [ ] T02 — Implement validated records, date precision, URL/email normalization, and coverage states with failing tests first.
- [ ] T03 — Implement identity matching, email maxima, invitation/connection evidence, and recent-message projections with failing tests first.
- [ ] T04 — Implement SQLite persistence, atomic source commits, idempotency, mapping, and local deletion with failing tests first.
- [ ] T05 — Implement private configuration and the bounded, redacted HTTP layer with failing tests first.
- [ ] T06 — Implement Apollo list, contact, verified current sequence membership, and email reads. Live contract verification depends on T00; synthetic tests alone cannot close it.
- [ ] T07 — Implement Gmail desktop OAuth, private token storage, refresh, and re-consent handling. Live consent remains owner-controlled.
- [ ] T08 — Implement direct Gmail SENT metadata reads and combine them with Apollo email evidence; both sources are required.
- [ ] T09 — Implement LinkedIn Member Portability snapshots with per-domain paging, readiness, freshness, and coverage checks.
- [ ] T10 — Implement explicit LinkedIn and Apollo CSV import fallback against confirmed export schemas; keep unverified formats visibly unsupported.
- [ ] T11 — Implement on-demand orchestration, partial results, budgets, and transaction-safe refreshes.
- [ ] T12 — Implement private HTML/CSV tables and network-activity output, escaping, formula protection, and freshness display.
- [ ] T13 — Connect CLI commands, manual-match review, local forgetting/reset, and an offline end-to-end workflow.
- [ ] T14 — Resolve and implement the LinkedIn freshness path: use verified snapshots if sufficient; otherwise implement supported changelog reconciliation. Unproven event coverage remains a stated blocker, never a fictional live-sync pass.
- [ ] T15 — Add offline CI, finish setup documentation, run private live acceptance, and open the implementation PR with truthful evidence. Keep missing live capabilities on this list until resolved.
