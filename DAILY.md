# DAILY — every session after the first

The first session made the site theirs. Every later one should make it more theirs: a book they mentioned shows up,
a note gets sharper because you learned something, an author's news lands next to the book it's about.

## Start of every session (or scheduled run)

```sql
select rules ->> 'note' as goal from rules where kind = 'book_note';
select * from todo order by priority;
select * from hints;
```

Read the goal before writing anything. "Add X" is always the fast add ([BOOKS.md](BOOKS.md)); pages only when asked
or when `todo` lists them ([PAGES.md](PAGES.md)). Then:

- **The reader is here** → offer `todo` in one line each. Do what they pick.
- **Scheduled run, nobody watching** → do `news_why` and `author_check` items, and pages only for books being read
  when the reader has page rules (never a first page question, never an author page they didn't say yes to); leave anything that changes
  shelves, rankings or the reader's own words for when they're here. End with a two-line summary they'll see later.

## What they say is data

- "I just started X" → add it (`status = 'reading'`), or update it if it's there. `todo` then asks for its page;
  build it in their sections ([PAGES.md](PAGES.md)).
- "Finished X, loved it / it was thin" → `status = 'finished'`, and adjust its scores. Remember the reason: it tells you
  how they judge the next book.
- "X should rank higher" → about one book: change its scores. About how they judge: change the ranking (RANKING.md)
  and say what moved.
- "Not that shelf" → move it, and ask once whether the shelf needs a new niche.
- Their goal changes → rewrite the `book_note` note (START.md, merge), then offer to re-note the top books.

## A large library

For a reader with hundreds or thousands of books ([IMPORT.md](IMPORT.md)), scheduled runs enrich a little each time,
best first:

1. pages for books they're reading (`todo` `description:`), in their page rules ([PAGES.md](PAGES.md)); then notes and
   scores for books rated 5, then 4;
2. missing ISBNs, and covers/photos the shared facts still don't have (README, *Covers and author photos*);
3. author pages, one per author, most-owned first ([IMPORT.md](IMPORT.md) section 7). The author rows already exist
   with the books (the database makes one per author); never add a second row for an author who has one.

Keep each unattended run to 20 rows or fewer: that's the reader's standing limit for scheduled runs (in a session
with them, still ask before more than 5). Say in the run's summary what you changed. Never change shelves, statuses, ratings or
their own words on an unattended run.

## Keep it alive

The home page shows "Last updated by your AI" and warns after a week of silence. One small, real write a week (a
news why, a sharper note, a new book) is better than a big rewrite once a month.

## Updates to this manual

The manual is this repo's `main`, and it improves. At the start of every session, reread README (and the file for the
step you're on) from `main`. Where it differs from your notes or your memory, the manual wins. You never edit this
repo; it isn't yours or the reader's to change.
