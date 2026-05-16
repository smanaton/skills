# ADR Index & Supersession Format

The ADR set is the **primary cold-path home** for resolved *architectural* history — most retired ambiguities should resolve to "already in an ADR, delete the flag, ensure the index reflects it" rather than to a decision-log.

ADR **bodies are immutable history** — `/consolidate-docs` never edits them, and never authors a new one. It adds only navigational metadata: a single index file, and supersession links expressed *in the `## Status` section*.

## The index — `docs/adr/README.md`

One table, ordered by number. Status is the live status (see below). Supersession shown inline so the reader never has to open a stale ADR to learn it was replaced.

```md
# Architecture Decision Records

| ADR | Title | Status |
|-----|-------|--------|
| [0001](ADR-0001-event-sourced-orders.md) | Event-sourced orders | Accepted |
| [0008](ADR-0008-charge-vs-payment.md) | Charge vs Payment split | Superseded by [0012](ADR-0012-payment-intents.md) |
| [0012](ADR-0012-payment-intents.md) | Payment intents | Accepted (supersedes [0008](ADR-0008-charge-vs-payment.md)) |
| [0003](ADR-0003-cart-order-split.md) | Cart and Order are distinct | Accepted (refines [0001](ADR-0001-event-sourced-orders.md)) |
```

Regenerate the whole table each run from the ADR files (it is derived, not append-only). It is the one ADR artefact that may be rewritten in full.

## Supersession in `## Status` (the only body change permitted)

The `## Status` line is metadata, not the decision narrative, so it may be updated. Nothing else in an ADR body is ever touched.

**Superseded / refined ADR** — append the back-reference, keep the original word:

```md
## Status

Accepted — superseded by ADR-0012
```
```md
## Status

Accepted — refined by ADR-0003
```

**Superseding / refining ADR** — append the forward-reference:

```md
## Status

Accepted (supersedes ADR-0008, refines ADR-0001)
```

## Distinguishing the relationships

- **Supersedes** — the later ADR *replaces* the earlier decision; following the old one now would be wrong (ADR-0012's payment-intent model vs the charge/payment split in ADR-0008).
- **Refines** — the earlier decision still holds; the later ADR sharpens or splits part of it (ADR-0003 splitting Cart out of the order model in ADR-0001).

If which one it is isn't unambiguous from the ADR texts, that is a **contradiction to surface**, not a call to make — route it to the user per the SKILL.md boundary.

## Retired-ambiguities map

When case-1 retirement deletes a pointer from the decision-log, its *where* must survive at the index. Append a small table so traceability is not lost:

```md
## Retired ambiguities (compacted from the glossary → captured here)

| Ambiguity | ADR | Note |
|-----------|-----|------|
| "north / north star" vs Intent | ADR-0011 | |
| Visibility Alignment vs Alignment Assessment | ADR-0003 + ADR-0011 | Inferred linkage; glossary carries the operation-not-table nuance |
```

The **Note** column is mandatory for any **inferred** linkage (`1`-with-inference) — it must say "inferred — confirm linkage" or equivalent. Case-1b items (captured in canonical body, no ADR) do **not** appear here — their home is the canonical doc itself, not the ADR map.
