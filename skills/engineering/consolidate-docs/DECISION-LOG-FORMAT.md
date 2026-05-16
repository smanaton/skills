# Decision Log Format

The decision-log is the **case-3 exception home, not the default**. Resolved ambiguities go to the strongest *existing* cold-path artifact: if an ADR already captures it → just delete the flag (no entry here); if it is architectural but ADR-less → hand back to grill-with-docs. Only **useful process / rationale history that is genuinely not ADR-worthy and not already well represented by the ADR index or the canonical docs** lands here — so the repo never grows a second decision system parallel to its ADRs. One entry is the norm, not a failure: it defines this file's narrow scope.

It is **append-only** and **dated**, a pointer index rather than the system of record: each entry still cites where the substance lives (an ADR, or a glossary section).

## File header (first creation only)

```md
# Decision Log

Retired ambiguities, compacted out of `CONTEXT.md` by `/consolidate-docs`.
Append-only. Each entry points to the ADR or glossary section that holds the
substance — this file records *that it was decided and where*, not the reasoning.
```

## Entry format

Group by the date the ambiguity was **resolved** (the `Resolved (<date>)` date from the original flag, not the consolidation date). One line per retired flag.

```md
## <YYYY-MM-DD>

- **<short name of the ambiguity>** — <one clause: the resolution outcome>. → <ADR-00NN | CONTEXT.md §<section>>
```

## Example

```md
## 2026-02-10

- **`charge` vs `payment`** — `charge` is the gateway attempt, `payment` the settled record; not synonyms. → ADR-0008; CONTEXT.md §Billing
- **"Order" vs "Cart"** — a Cart becomes an Order only at checkout; distinct lifecycles. → ADR-0003; CONTEXT.md §Ordering

## 2026-02-03

- **`pending` Shipment state** — removed; `created` + `awaiting_stock` already express it. → CONTEXT.md §Shipment states
- **"account" vs "Customer"** — legacy term for Customer; never use in new prose. → ADR-0005; CONTEXT.md §Identity
```

## Rules

- **Append-only.** Never rewrite, reorder, or delete past entries. New consolidation runs add entries (and may add a new date heading); they never touch existing ones.
- **Must cite a durable home.** If an entry would have nothing to point to (no ADR, not in the glossary body), it is **not** ready to retire — that resolution is decision-worthy and unfinished. Leave the flag in `CONTEXT.md` and route it to grill-with-docs instead.
- **One line, one ambiguity.** No embedded history, no rationale prose — the cited ADR holds that.
