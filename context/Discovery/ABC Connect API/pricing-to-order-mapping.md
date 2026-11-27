# Pricing to order handoff

These are proposed adapter mappings grounded in the endpoint fields. A successful quote is not a reservation or a final-price guarantee.

| Price response/context | Order request | Handoff rule |
| --- | --- | --- |
| bound shipToNumber | shipTo.number | Same authorized job account; delivery name/address/contacts supplied separately. |
| bound branchNumber | branchNumber | Same allowed branch as the price request. |
| lines[].id | lines[].id | Join by ID and verify item/context; never rely on array position. |
| lines[].itemNumber | lines[].itemNumber | Must match selected ABC item. |
| Get Item itemDescription | lines[].itemDescription | Use supplier description; no LLM rewrite in wire field. |
| selected quantity + successful quoted quantity | lines[].orderedQty.value | Verify exact requested quantity; no integer rounding. |
| priced lines[].uom | lines[].orderedQty.uom and unitPrice.uom | Only when pricing result is confirmed for this ordered UOM; differing UOM requires a verified conversion. |
| lines[].unitPrice | lines[].unitPrice.value | Successful pricing line only. Price may be updated by branch processing. |
| lines[].currency.code | currency | One coherent currency per local proposal; reject mixed currency. |
| dimensional length + validated product variation | lines[].dimensions.length | Required for dimensional lines; absent for non-dimensional lines. |
| local price instructions | lines[].unitPrice.instructions | Required string; empty when unused. |
| local correlation ledger | requestId | Tracking ID; provider deduplication is not established. |

Pricing permits 50 lines per call, while orders permit 99 lines. A 99-line local order needs at least two pricing calls for the same bound context, with all results reconciled before authorization. Place Orders accepts an array; this pack uses one order per request as a local default.

The draft schema allows only known fields. Dynamic authorization, item/branch matching, UOM conversion, dimensionality and quote freshness are runtime checks. The synthetic example demonstrates structure only and must not be sent as a real order.
