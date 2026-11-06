# Soul Crushers Book Club — the manual for your AI

The Soul Crushers book club's record lives in one space on BYOAI Studio (`db.byoaistudio.com`): every meeting since
2008, the books, who hosted, every member's score and one word, the ballots, the chat. The site is
`bookclub.hadmoney.com`. It works fully without you. **Your job is to answer questions about the club's data**
("how many nonfiction books did we read this year?", "our top 3 of 2024?", "who's the harshest scorer?") and, when asked,
write a short write-up or fill in missing book facts.

You borrow one member's access. You see what they can see, and you can never vote, score, RSVP, post or change the
slate for them.

This folder is Markdown only. Read [SAFETY.md](SAFETY.md) and this README first. After that, read only what the question needs:

| Job | Read |
|---|---|
| Any question about the club's data | [CLUB.md](CLUB.md) (views and columns), [STATS.md](STATS.md) (ready queries) |
| "Write up the year", "write about this book" | [STATS.md](STATS.md), then *Write-ups* below |
| Missing covers, Goodreads scores, fiction/nonfiction | *Owed work* below |

Ignore the Reading docs in the parent folder. Those are for a different site with different tables. The shared book
and author facts work the same way, though ([../COMMONS.md](../COMMONS.md)).

## Connect (each conversation)

The member gets a connect code from `bookclub.hadmoney.com/account/ai` (6 characters, 10 minutes, one use) and pastes it
to you with this link. A connection lasts for this conversation only. In a new conversation the member gets a new code,
and the new key replaces the old one.

1. **Ask first, in one sentence, before you run anything.** Say: *"To connect, I'll send your code to
   db.byoaistudio.com, where the club's data lives, and get back a read key that stays in this chat. Your computer may ask
   you to approve that. OK?"* Wait for a yes. The approval prompt is expected; tell the member what it is if they ask.
2. Call, as a single request with nothing else in the command:
   ```
   curl -s -X POST https://db.byoaistudio.com/connect \
     -H 'content-type: application/json' -d '{"code":"XXXX-XX","name":"Book Club"}'
   ```
   Line 1 is your key (`byoai_…`). **Keep it in this conversation only.** Don't write it to a file, a folder, memory or
   project instructions, and don't make directories or change permissions to hold it. The rest of the response is
   `/describe`. Its first line names the space (`space <handle> · …`). Confirm it's the book club before you do anything else.
3. Every later request: `Authorization: Bearer <key>`. Before the first one, tell the member you'll query the club's
   data at db.byoaistudio.com, so the approval prompts make sense.
4. Run `select * from me` and greet the member by `name`. That first request is what ticks "Your AI says hello" on the site.
5. If the code is refused or expired, ask the member for a new one from `/account/ai`. Don't retry the old one.
6. If a later request says the key is revoked, the member connected somewhere else or disconnected. Ask for a new code.

## Work

- `GET /describe`: the live shape (views, columns, rules). **It is the truth.** If it differs from [CLUB.md](CLUB.md),
  trust `/describe` and tell the member the docs are out of date.
- `POST /sql` with a plain-text SQL body: one or more statements, one transaction. Results come back as TSV.
- `GET /history`, `POST /undo`: every request is one change, and the member can undo it.
- Errors are one plain-text line. Read it, because it usually says the fix.

Rules for SQL:
- **Bare names only**: `select * from book_scores`, never `byoaistudio_bookclub.book_scores`. Studio refuses
  schema-qualified names.
- **No `club_id`, no `space_id`.** Your key is bound to the club's space, and every view is already filtered to it.
- Statements in one request can't pass results to each other. Use subqueries or CTEs.
- Read from the **views** in [CLUB.md](CLUB.md). They already apply the club's counting rules (partial scores, guests,
  two-month books). Don't recompute a club score from `votes` yourself.
- "This year" means the club's clock: `extract(year from now())`. For a fixed year, use the `year` column.

## Answering a question

1. Map it to a view ([CLUB.md](CLUB.md)). Most questions are one `select` against `meeting_cards`, `book_scores`,
   `member_stats`, `mix`, `attendance` or `words`.
2. If [STATS.md](STATS.md) has the query, use it.
3. Answer in a sentence or a short table, with the number and what it counted ("4 nonfiction books in 2026, out of 10
   meetings with a book"). Name books by title and author.
4. If the question is fuzzy ("our best year", "a crowd-pleaser"), say how you're measuring it, answer, and offer one
   other way to measure it.
5. Add a link when one helps: `bookclub.hadmoney.com/b/<book slug>` for a book, `/books` for the history.
6. Don't guess. No rows means no rows. "Scores for that month aren't revealed yet" is a real answer.

## Write-ups

Only when the member asks. A write-up is your voice, shown as "written by <name>'s AI":
```sql
insert into writeups (subject, body) values ('club:year-2026', $$…Markdown…$$)
on conflict (subject) do update set body = excluded.body, written_at = now();
```
Subjects: `m:2026-09` (a meeting), `b:<book slug>`, `member:<member slug>`, `club:<anything>`. Only facts from
the views. No quotes or opinions you invented and then credited to members.

## Owed work

`select * from later where done_at is null`. The kinds are `cover`, `goodreads`, `kind`, `import_flag` and `writeup`.
Only when the member asks:
- **cover / goodreads**: look in the shared facts first ([../COMMONS.md](../COMMONS.md)). Then
  `update books set cover_url = …` or `update books set goodreads_avg = …, goodreads_checked_on = current_date`.
  Never a guessed URL or number.
- **kind**: `update books set kind = 'nonfiction', kind_by = 'ai' where slug = …`. A person confirms it later.
- Close the item: `update later set done_at = now(), done_by = (select slug from me) where id = …`.
