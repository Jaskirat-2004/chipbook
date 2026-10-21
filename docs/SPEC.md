# Chipbook - v1 spec

A web app that keeps the books for friends' home poker games: buy-ins, cash-outs and settle-ups, approved by the host. It does not play poker and it does not move money. No download, no accounts.

Live target: `chipbook.kikilabs.in`.

## Rooms
- A **room** is one ledger. There are two types:
  - **Night**: code `JSN-XXXXX`, deleted 24 hours after creation.
  - **Table**: code `JST-XXXXX`, deleted 15 days after creation. (v2: paid extension.)
- `XXXXX` is 5 random characters drawn from an alphabet with no look-alikes (no 0/O, 1/I/L). Codes are never sequential.
- The creator is the **host**. The host sets the room name, the chip value (presets Rs 10 / 20 / 50 or custom), and the default buy-in in chips (20, editable). Chip value is fixed for the room.
- Joining is by code or QR (a link with the code in it). The QR is drawn in the browser by a small library, not by an online service.
- Before deletion, the room shows an "expires in X, export now" warning.

## Players and PIN
- A player's name is unique within a room. Each player has a 4-digit PIN, stored hashed.
- New name: the player enters a name and PIN, becomes **pending**, and the host approves them.
- Returning: the player enters an existing name and its PIN. After a correct PIN, the phone gets a session token, so the PIN is not asked again on that phone.
- Five wrong PINs lock the name. The host unlocks it or resets the PIN.
- When a room is deleted, its players, sessions and transactions are deleted with it.

## Transactions (all start pending; only approved ones count)
| Type | Posted by | Approved by |
|---|---|---|
| Buy-in (any number of times) | the player | the host |
| Cash-out (chips in hand when leaving) | the player | the host |
| Settle (paid outside the app, from payer to receiver) | the payer | the host AND the receiver |
| Any of the above posted by the host | the host | any one other active player (and the receiver, for a settle) |

- Rows are never edited. A mistake is fixed by a reversal row.
- A pending bar shows what is waiting. The host sees Approve / Reject.
- **Seated**: an approved buy-in seats a player, and an approved cash-out unseats them.

## Board
For each player: chips bought in, chips cashed out, net in chips and Rs (net is shown only once the player has cashed out), and settle-ups so far. Sorted by net.

## Chip check
Whenever nobody is seated, total chips cashed out must equal total chips bought in. If they differ, a warning shows the gap with two choices:
- **Recount**: players fix their cash-outs.
- **Force close**: host only, after a confirm. The gap is written to the log and the report, and no money is assigned to anyone for it.

## Settle suggestion
From the net balances, suggest the fewest "A pays B Rs N" payments: greedy, largest debtor pays largest creditor.

## Export
CSV of every transaction, plus a print-styled report page saved as PDF by the phone. Footer: Chipbook, by Jaskirat Singh.

## Stack
- Frontend: plain HTML, CSS and JavaScript (ES modules) in `public/`. No framework.
- Backend: Cloudflare Pages Functions in `functions/api/` (Workers runtime: `Request` / `Response`, not Node APIs).
- Database: Cloudflare D1 (SQLite). Plain SQL with `?` placeholders. Schema changes only through numbered files in `migrations/`.
- Live updates: each room has a `version` that goes up on every write. Phones poll it every 3 s and refetch only when it changed.
- Expiry: every request first deletes expired rooms; foreign keys cascade the delete.

## Schema (draft; it may change, through new migrations)
- `rooms` (id, code UNIQUE, type night|table, name, chip_value_rs, default_buyin_chips, host_player_id, version, created_at, expires_at)
- `players` (id, room_id, name, pin_hash, status pending|active, failed_pins, locked, seated, created_at, UNIQUE(room_id, name))
- `sessions` (token_hash, player_id, created_at)
- `txns` (id, room_id, player_id, type buyin|cashout|settle|reversal, chips, amount_rs, to_player_id, reverses_txn_id, status pending|approved|rejected, approved_by, receiver_ok, created_at)

IDs: integer primary keys, internal only (see the decision in the Matrix notes). Security comes from checking each request's session against the room, not from hidden IDs.

## Phases (discussed before each one; each one working, tested and committed)
0. Setup: wrangler, D1, first migration, local dev, a hello page deployed to the subdomain
1. Rooms + join: home, create, codes, QR, name + PIN, host approves joins, lock, expiry
2. Buy-ins: pending bar, approve/reject, polling, bought-in board
3. Cash-out: seated, chip check with Recount / Force close, net board
4. Settle: two-party approval, fewest-payments suggestion
5. Export: CSV, PDF report, expiry warning
6. Design pass: poker theme, installable web app, final deploy

## v2 (not now)
Paid table extension (needs HR's OK under the parallel-employment clause, plus a payment gateway), UPI pay buttons, majority-vote approval, stats per player.
