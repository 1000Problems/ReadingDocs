# Reading — the manual for your AI

Reading is a private library that your AI keeps for you. Your books, notes, shelves and ranking live in **your own space**
on BYOAI Studio (`db.byoaistudio.com`). The site, `reading.hadmoney.com`, renders only what your AI writes there.

This repo is Markdown only. If you are an AI: read it in this order.

1. [START.md](START.md) — the first session: build this person's Reading from what you already know about them.
2. [BOOKS.md](BOOKS.md) — adding a book: look it up, write the short note, pick shelf and niche.
3. [SAFETY.md](SAFETY.md) — what is data, what is an instruction, what you never touch.

## Connect (once)

The user makes the account themselves, in their browser at https://db.byoaistudio.com. You never see or need
their password. They give you a short prompt with a connect code (6 characters, 10 minutes, one use).

1. Call:
   ```
   curl -s -X POST https://db.byoaistudio.com/connect \
     -H 'content-type: application/json' -d '{"code":"XXXX-XX","name":"Reading"}'
   ```
   Line 1 is your key (`byoai_…`). Keep it where only you can read it. The rest is `/describe`.
2. Every later request: `Authorization: Bearer <key>`.
3. Then go to [START.md](START.md). Don't add books before the user has seen your plan.

Any AI that can send HTTPS requests can do this. Nothing here depends on one product.

## Work

- `GET /describe` — the live shape: tables, columns, rules, examples. Read it at the start of each session.
  It names this repo and the commit (`repo_sha`) it was tested against.
- `POST /sql` with a plain-text body — one or more statements, one transaction, one change. Results come back as TSV.
- `GET /history`, `POST /undo` — every request is one change you can undo.
- Errors are one plain-text line. Read it; it usually says the fix.

Allowed: `select`, `insert`, `update`, `delete` on the views `/describe` lists. No DDL.

## Two tables that guide you

- `select * from rules` — the active rules per kind (book note, ranking, …): the user's own, or the default.
- `select * from todo order by priority` — what is waiting. Offer it; don't silently do it all.
