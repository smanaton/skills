---
name: consolidate-docs
description: Compaction counterpart to grill-with-docs — compacts the always-read hot path (the glossary) by retiring resolved ambiguity/history into the strongest existing cold-path artifact: ADR first, decision-log only when an ADR is not the right home. It reorganises and verifies losslessly; it never re-decides and never authors an ADR. Use when the glossary's flagged-ambiguities section has become a changelog of Resolved entries, after several grill-with-docs sessions, when the glossary or ADRs feel bloated, or when the user asks to consolidate / compact / tidy / prune the domain docs or ADRs.
---

<the-boundary>

grill-with-docs is the **producer**: it decides terminology and appends to the glossary in the moment, by design without batching. This skill is the **compaction counterpart**. Its purpose is **not** to shrink total documentation — it is to cut the *repeated* context cost of the always-read hot path by relocating settled history into the strongest cold-path home it already has.

- **Success — primary:** reduce always-read context **without losing decision traceability**.
- **Success — secondary:** reduce *total* docs size only when the ADR system already preserves the history.

Lossless reorganisation and verification only. It never re-litigates a decision, **never authors an ADR**, and never picks a winner when two docs disagree — it *surfaces* and hands back to grill-with-docs. If consolidating would require a new decision, stop and say so.

</the-boundary>

<retirement-policy>

For each **eligible** resolved ambiguity (see Eligibility), route it to the strongest *existing* cold-path artifact — there is no universal "move it to a decision-log" default:

1. **Already captured by an ADR** → remove it from the glossary; ensure the ADR index reflects it. **No decision-log entry.** If the ADR linkage is *inferred* rather than explicit, you may still retire the pointer provided the canonical concept is captured elsewhere — but the index-map row must say "inferred — confirm linkage", never presented as fully ADR-backed.
1b. **Canonical-doc captured** → if the resolution is durably in the canonical body of `CONTEXT.md`/the glossary or another authoritative domain doc, and it does **not** encode an architectural trade-off, rejected alternative, or implementation decision, retire the pointer **without proposing an ADR**. (Stops the skill manufacturing ADRs for definitional cleanups.)
2. **Architectural, not yet in an ADR** → do **not** author the ADR. Rejected alternatives do *not* automatically require one — propose/hand back to grill-with-docs only when the rejection reflects an architectural trade-off likely to recur or be challenged later. Leave the flag in place until the ADR exists.
3. **Useful process/rationale history** that is not ADR-worthy *and* not already well represented by the ADR index or canonical docs → keep in the cold-path `decision-log`. One such entry is not a failure — it defines the log's real, narrow purpose.
4. **Still unresolved** → leave it in the glossary.

The decision-log is the **exception home, not the default** — it exists so case 3 has a durable place to go without standing up a second parallel decision system alongside the ADRs.

</retirement-policy>

<configuration>

Repo-specific **paths** live in `docs/agents/consolidate-docs.md` (scaffolded by `/setup-matt-pocock-skills`): glossary doc + flagged-ambiguities heading; resolution marker + hedge tokens; ADR root + index path; decision-log path; domain-doc drift-check command (or `none`). The routing policy above is **fixed**, not a per-repo strategy choice. **Read the config first.** If it is absent, use documented defaults (glossary `CONTEXT.md`; heading `## Flagged ambiguities`; marker `Resolved (<date>)`; hedges `pending`/`provisional`/`revisit`/`TBD`/`if`/`once`; ADRs `docs/adr/` + `docs/adr/README.md`; decision-log `docs/decision-log.md`; drift `none`) but **say so up front** — never silently assume a repo's dialect; this skill rewrites the canonical glossary.

</configuration>

<workflow>

## Phase 1 — Scan (read-only)

Read the configured glossary doc, every ADR under the configured ADR root, and `docs/domain/*`. Identify, changing nothing: **(1)** resolved flags matching the configured marker with no configured hedge token; **(2)** leaked implementation detail in the glossary (module paths, `Implemented <date>: …`, code identifiers — but **keep** canonical source-of-truth anchors); **(3)** bloated/duplicate entries; **(4)** ADR hygiene gaps (missing index; missing Status supersession), only for ADRs whose Status is in the configured **terminal** set; **(5)** domain-doc drift via the configured command, or "no drift check configured" if `none`. Do not hand-eyeball drift.

## Phase 2 — Plan (present, get approval — do not edit yet)

One structured plan, lossless changes separated from contradictions, **including the projected measurement**:

- **Route per the retirement policy** — every eligible item tagged case 1/2/3/4 with its cold-path target named (ADR#, or decision-log, or "hand back: ADR-worthy", or "kept: unresolved").
- **Strip impl-detail** — quote each line; name where the fact still lives.
- **Compress entries** — before → after.
- **ADR hygiene** — index + Status/supersession lines (header metadata only).
- **⏸ Skipped — not yet final** — hedged `Resolved` / non-terminal ADRs, disqualifying wording quoted, left in place.
- **⚠ Contradictions found** — docs that disagree, or a resolution whose *why* no ADR records. Surface, never resolve.
- **Projected hot-path / cold-path / net-corpus impact** — see Measurement.

Wait for approval. Adjust on feedback.

## Phase 3 — Apply (after approval, one pass, one commit)

Execute the routing: case 1 → delete flag + update index; case 3 → append the configured decision-log per [DECISION-LOG-FORMAT.md](./DECISION-LOG-FORMAT.md); case 2 → left in place + handed back; case 4 → left. Rewrite the glossary (only open ambiguities remain; impl-detail stripped; entries compressed). Create/update the ADR index + Status lines per [ADR-INDEX-FORMAT.md](./ADR-INDEX-FORMAT.md) — **never edit an ADR body**. Leave `docs/domain/*` untouched. Emit the **realized measurement**.

</workflow>

<measurement>

Compute and report **every run** — projected in Phase 2, realized in Phase 3. Bytes exact; tokens = `bytes/4`, always labelled an estimate. Phase-2 projections come from terse drafts and **understate final size / overstate the saving**; the Phase-3 post-apply re-measure is authoritative.

```
Hot-path impact (the always-read cost, paid every skill session)
  - glossary doc: bytes / est-tokens — before → after (Δ, %)
  - flagged-ambiguities section: bytes — before → after
Cold-path impact (read on demand, not in the up-front read)
  - ADR index size; decision-log size; new/changed ADRs
Net corpus impact
  - total docs bytes — before → after
  - verdict: reduced, or merely relocated — state which, explicitly
```

</measurement>

<hard-rules>

- **Eligibility — terminal & unconditional only.** A flag matching the configured marker with no configured hedge token; an ADR whose Status is in the configured terminal set. Everything else → "⏸ Skipped — not yet final", left in place.
- **ADR-first routing.** The decision-log is the case-3 exception, never the default home for resolved history.
- **Inferred ADR linkage is marked, never hidden.** Retire an inferred-linkage pointer only when the canonical concept is captured elsewhere, and the ADR-index map row must read "inferred — confirm linkage". Never present inferred coverage as fully ADR-backed.
- **Never author a decision.** Case 2 (ADR-worthy, no ADR yet) is surfaced and handed back to grill-with-docs; the skill does not write the ADR.
- **Losslessness test.** A move or strip is lossless only if both the *what* and the *why* survive verbatim in a durable home: glossary body, an ADR, or git history for the diff. A retirement pointer is not the system of record.
- **Never re-decide.** Two docs disagree → surface both, pick neither.
- **ADR bodies are immutable.** Index + Status/supersession metadata only.
- **The decision-log is append-only.** Never rewrite or reorder past entries.
- **The glossary is a glossary.** Not a spec, scratch pad, or changelog.

</hard-rules>
