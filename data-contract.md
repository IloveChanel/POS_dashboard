# Order Data Contract

This is the exact shape of one order. The website's checkout, the per-store
container (database), and the dashboard must all read and write orders in
this exact structure — same field names, same types, no exceptions.

If any one of the three pieces uses a different shape, orders will either
fail to save or fail to display. This file is the single source of truth
for that shape. When the real backend gets built, this becomes the schema.

```json
{
  "id": "1042",
  "storeId": "HARRISON-001",
  "time": "6:14 PM",
  "date": "2026-09-11",
  "customer": "Michelle Vance",
  "phone": "(586) 555-0142",
  "address": null,
  "email": "mvance@email.com",
  "subscribed": true,
  "fulfill": "pickup",
  "items": [
    {
      "name": "2x Detroit Style Pizza",
      "mods": ["Extra cheese", "Extra sauce", "Light on the pepperoni"]
    }
  ],
  "total": 58.97,
  "status": "open"
}
```

## Field rules

- `id` — string, unique per order. Never reused, even after an order is deleted.
- `storeId` — string, must match the format from signup (`LOCATIONNAME-####`). This is what keeps one store's orders from ever appearing on another store's dashboard.
- `time` / `date` — kept separate on purpose: `time` is for display, `date` is `YYYY-MM-DD` for filtering/export/accounting. Never combine into one field.
- `address` — `null` when `fulfill` is `"pickup"`. Never an empty string — empty string and "no address" are different things and get treated differently by delivery logic.
- `email` / `subscribed` — `subscribed` is `false` unless the customer explicitly checked an opt-in box at checkout. Never default this to `true`.
- `items[].mods` — always an array, even when empty (`[]`), never `null`. Code that loops over modifiers shouldn't have to check for null first.
- `total` — **number**, not a string with a dollar sign. `58.97`, not `"$58.97"`. Formatting with a `$` happens only at display time, in whichever page is showing it — never store the formatted version.
- `fulfill` — only ever `"pickup"` or `"delivery"`, lowercase, nothing else.
- `status` — only ever `"open"`, `"printed"`, or `"fulfilled"`.

## Why this matters now, before the backend exists

Right now the dashboard, menu-items, and export pages each have their own
sample data typed in separately — which is fine for a mockup, but it's
exactly how contract drift happens in a real build: someone tweaks one
page's data shape while testing and forgets the other two. Once a real
website checkout starts sending live orders into an actual database, this
document is what the backend's insert logic, the dashboard's read logic,
and the export's CSV logic all get written against — so there's one
definition to change, not three places to keep in sync by memory.
