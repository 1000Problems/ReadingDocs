# DAILY — every session after the first

The first session made the site theirs. Every later one should make it more theirs: a book they mentioned shows up,
a note gets sharper because you learned something, an author's news lands next to the book it's about.

## Start of every session (or scheduled run)

```sql
select rules ->> 'note' as goal from rules where kind = 'book_note';
select * from todo order by priority;
select * from hints;
```

Read the goal before writing anything. Then:

- **The reader is here** → offer `todo` in one line each. Do what they pick.
- **Scheduled run, nobody watching** → do `news_why`, `author_check` and `author_page` items; leave anything that changes
  shelves, rankings or the reader's own words for when they're here. End with a two-line summary they'll see later.

## What they say is data

- "I just started X" → add it (`status = 'reading'`), or update it if it's there. `todo` will ask for its description.
- "Finished X, loved it / it was thin" → `status = 'finished'`, and adjust its scores. Remember the reason: it tells you
  how they judge the next book.
- "X should rank higher" → about one book: change its scores. About how they judge: change the ranking (RANKING.md)
  and say what moved.
- "Not that shelf" → move it, and ask once whether the shelf needs a new niche.
- Their goal changes → rewrite the `book_note` note (START.md, merge), then offer to re-note the top books.

## A large library

For a reader with hundreds or thousands of books ([IMPORT.md](IMPORT.md)), scheduled runs enrich a little each time,
best first:

1. books they're reading, then rated 5, then rated 4: a note and a score;
2. missing covers and ISBNs, from `book_facts` or a source you opened;
3. authors they own five or more books by: offer the author page (don't add author rows unattended).

Keep each unattended run to 5 rows or fewer (SAFETY: bulk changes need a yes). If the reader gives a standing yes
("fill up to 25 covers a run"), follow it and say so in the run's summary. Never change shelves, statuses, ratings or
their own words on an unattended run.

## Keep it alive

The home page shows "Last updated by your AI" and warns after a week of silence. One small, real write a week (a
news why, a sharper note, a new book) is better than a big rewrite once a month.

## Updates to this manual

`/describe` names the commit (`repo_sha`) the server was tested against. If it's newer than your copy, tell the
reader in one line and offer to read the changes. Don't switch silently.
