# Cases — Index

> One-row-per-case index. The orchestrator scans THIS file — not individual case files — to look up existing case IDs on a new inbound. Keeps lookup O(1) as the team's case volume grows.

## Format

One row per case. Most recent first.

| case_id | client | property | status | agent | last_updated |
|---------|--------|----------|--------|-------|--------------|
| 2026-04-28-Henderson-buyer | James + Sarah Henderson | 4521 Speedway Ave, Austin 78751 | option_period | Diana | 2026-05-13 |
| 2026-05-13-Patel-buyer | Tom + Priya Patel | 78704 (Bouldin/South Lamar focus) | closed | Diana | 2026-05-13 |

## How This File Is Maintained

- **New case opened** — orchestrator prepends a row when the case is first routed to 01.
- **Status change** — transaction coordinator updates `status` column: `active` → `under_contract` → `option_period` → `pending_close` → `closed` | `terminated`.
- **No human edits** to this file outside of those two triggers. The index reflects live deal state.

## Status definitions

| Status | Meaning |
|--------|---------|
| `active` | Lead qualified, in research/comm stage — no contract yet |
| `under_contract` | Offer accepted, effective date set, in TC tracking |
| `option_period` | Within the buyer's termination option window |
| `pending_close` | Option expired, financing contingency cleared, closing scheduled |
| `closed` | Transaction complete |
| `terminated` | Deal ended before close (buyer terminated, fell through, etc.) |

## Cold-start state

Until at least one real case has been opened, this file shows only the EXAMPLE-patel-2026 row above. Diana on day one will see this and `EXAMPLE-patel-2026.md` — the convention template for how a case archive file should look.
