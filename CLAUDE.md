# Chipbook - rules for Claude in this repo

What we are building is in `docs/SPEC.md`. Build against it, not against memory. If something is not in the spec, ask before inventing it.

## Mode
Guided build. Before every phase, discuss with him what to build and how, and name the concepts it uses. Then, per file, he chooses: he writes it himself with guidance, he copies it from you, or he asks you to write it. Never write a whole phase unasked. After every phase, explain what was built in five plain lines (files, what each does, the one concept worth knowing), so he can explain this app in an interview. He knows Python/FastAPI and SQL, but this is his first JavaScript: compare to FastAPI where it helps.

## Stack (do not change without his OK)
Plain HTML/CSS/JS frontend, Cloudflare Pages Functions backend, D1 database, plain SQL. No frameworks, no ORM. No new npm package without asking first.

## Rules that keep it solid
1. The server is the judge: approvals, who may approve, and the chip check are enforced in `functions/`, never only in the browser.
2. Money and chips are integers. No floats anywhere.
3. Rows in `txns` are never updated except for their approval fields, and never deleted one by one; corrections are reversal rows. Whole rooms are deleted at expiry (cascade).
4. Every SQL query uses `?` placeholders. Never build SQL from strings.
5. User text is rendered with `textContent`, never `innerHTML`.
6. State changes are guarded: approve only `WHERE status = 'pending'`, so a double-tap cannot apply twice.
7. PINs and session tokens are stored hashed (Web Crypto), never in plain text. Five wrong PINs lock the player.
8. Every write bumps the room's `version`.
9. Money logic (balances, chip check, settle-up suggestion) lives in plain functions with tests under `tests/`, run by `node --test`. Tests pass before a slice is called done.
10. Small files, one job each. Thin slices: one works end to end and is committed before the next starts.

## Git
Commits show only Jaskirat: no Co-Authored-By or "Generated with" lines. Never commit `.dev.vars`, `.wrangler/` or `node_modules/`.
