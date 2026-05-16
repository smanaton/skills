# Upstream proposal: `consolidate-docs` as a companion to `grill-with-docs`

Draft for discussion with the skills author **before** any PR. Nothing in the
vendored `setup-matt-pocock-skills` has been edited — the change to it is
proposed here, not applied.

## The gap

`grill-with-docs` is, by design, an append-only **producer**: *"update
CONTEXT.md right there. Don't batch these up."* Nothing in the skill set ever
compacts the result. Over many sessions the glossary's "Flagged ambiguities"
section becomes a changelog of `Resolved` entries and implementation detail
leaks into a doc that is contractually glossary-only. Because the skills read
`CONTEXT.md` *in full, up front, every session* (per `domain.md`) while ADRs are
read on demand, this is a **recurring hot-path tax**, not a one-off size issue.
Every grill-with-docs user pays it.

## The value, measured

`consolidate-docs` is the compaction counterpart. Its job is **not** to shrink
total documentation — it is to cut the repeated context cost of the always-read
path. From one real run on a drifted repo:

```
The consolidation did not primarily reduce total documentation size.
It reduced repeated cognitive/context cost by moving settled history
out of the always-read path.
```

- Always-read glossary: **18,618 → 12,061 bytes (−35%, ~1,639 est-tokens off
  every skill session)**; the flagged-ambiguities block alone 6,920 → 208 bytes.
- Net corpus: ≈ **−568 bytes** — essentially flat. The history was *relocated to
  on-demand cold-path artifacts, not deleted*. That is the whole point.

The skill **measures and reports this every run** (hot-path / cold-path /
net-corpus, with an explicit "reduced vs merely relocated" verdict).

## Design — ADR-first, not decision-log-first

The early shape made a `decision-log.md` the universal retirement home. The real
run showed that, for an ADR-disciplined repo, that mostly creates redundant
pointers and a second parallel decision system. Corrected policy:

> For an ADR-disciplined repo, the default is **fold-into-ADR/index-and-delete**,
> unless the resolved ambiguity contains useful process/history not captured by
> an ADR — only that exception goes to a cold-path decision log.

Per-item routing hierarchy:

1. **Already captured by an ADR** → remove from the glossary; the ADR index's
   "Retired ambiguities" map carries the *where*. No decision-log entry. Inferred
   linkage is permitted but the map row must read "inferred — confirm linkage".
1b. **Canonical-doc captured** → durably in the glossary/CONTEXT.md body and
   encodes no architectural trade-off, rejected alternative, or impl decision →
   delete the pointer, **propose no ADR** (stops ADR-spam for definitional
   cleanups).
2. **Architectural, not yet in an ADR** → hand back to `grill-with-docs` *only*
   when the rejection is a trade-off likely to recur or be challenged; the skill
   **never authors the ADR**.
3. **Useful process/rationale history**, not ADR-worthy and not already in the
   ADR map or canonical docs → cold-path decision-log. One entry is the norm,
   not a failure.
4. **Still unresolved** → stays in the glossary.

Other invariants: surfaces-never-decides (conflicts handed back, never
resolved); an eligibility gate (hedged `Resolved` / non-terminal ADRs are
skipped, not actioned); config-driven **paths** via
`docs/agents/consolidate-docs.md`, same pattern as `issue-tracker.md` /
`triage-labels.md` / `domain.md` — the *policy* is fixed, only paths vary.

Markdown-only, no new deps: `SKILL.md`, `DECISION-LOG-FORMAT.md`,
`ADR-INDEX-FORMAT.md`.

## The change to `setup-matt-pocock-skills` (proposed, not applied)

The ADR-first policy means there is **no universal new convention** to opt into
— the decision-log is now an exception path, not a default. So the setup
addition is light: a fourth section that records *paths* only.

> **Section D — Doc consolidation.** Explainer: `/grill-with-docs` appends to
> the glossary as decisions are made; over time the "Flagged ambiguities"
> section fills with resolved entries that are paid on every session.
> `/consolidate-docs` retires those into the strongest existing cold-path home
> (an ADR first; a `docs/decision-log.md` only for non-ADR-worthy process
> history) and never re-decides. Confirm: glossary path (default `CONTEXT.md`),
> ADR root (default `docs/adr/`), decision-log path (default
> `docs/decision-log.md`), and a domain-doc drift-check command if one exists
> (default `none` — the skill never invents one).

Plus a `### Doc consolidation` line in the `## Agent skills` block and
`docs/agents/consolidate-docs.md` in the step-3 draft list. A template ships
with the skill.

## Success metric (how to judge it)

```
Primary:   reduce always-read context without losing decision traceability.
Secondary: reduce total docs size when the ADR system already preserves history.
```

## For Matt — the ask, framed

Lead with evidence, not with a new convention:

```
I tested a consolidation counterpart to grill-with-docs on a real repo.
It produced a 35% reduction in always-read context, then an ADR-first
correction reduced cold-path duplication without creating a parallel
decision system. Here are the commits and the proposed skill policy.
```

Supporting one-liner if needed: the first run proved the value (−35%
always-read context); the second corrected the policy (ADR-backed history
belongs in the ADR index, not a parallel decision log — the log remains only
for non-ADR-worthy rationale that still deserves traceability).

## Honest validation status

- **Run 1** (real, drifted repo, decision-log-default policy): proved the
  hot-path win — 35% / ~1,639 est-tokens off every session; net corpus ~flat.
- **Run 2** (real, same repo, corrected ADR-first policy): re-sorted the 11-entry
  decision-log → **1 entry**; cold-path **−2,160 B**; net corpus genuinely
  **reduced, not merely relocated**; ADR index became the traceability surface.
  Two commits, history not rewritten, so the narrative stays honest.
- Policy itself hardened by run 2: it surfaced and fixed two real holes —
  **case 1b** (canonical-doc-captured, don't manufacture an ADR) and the
  **case 2/3 tie-breaker** (rejected alternatives aren't auto-ADRs).
- Also exercised on a synthetic fixture (skipped / contradiction / hand-back /
  non-terminal-ADR / config-driven-path branches).
- **Not** battle-tested across many repos — single repo, twice. That is the
  honest ceiling of the current evidence.
