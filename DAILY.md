# DAILY — every session after the first

The first session made the site theirs. Every later one should make it more theirs: a book they mentioned shows up,
a note gets sharper because you learned something, an author's news lands next to the book it's about.

## Start of every session

```sql
select rules ->> 'note' as goal from rules where kind = 'book_note';
select id, at, what from your_changes where new order by id;
select * from hints;
```

Read the goal before writing anything. "Add X" is always the fast add ([BOOKS.md](BOOKS.md)); pages only when asked
or on a run ([PAGES.md](PAGES.md)). Follow up on `your_changes` in one line ([PAGES.md](PAGES.md), *What the reader
changed on the site*).

**Later** (README, *Later*) is not something to offer. Look at it when the reader asks ("what's waiting?",
`select title from owed_next order by priority limit 5`) or says "do what's waiting", and on runs.

## A run

The reader pastes the site's **Run now** opener ("Do the next 20 things in my Reading's Later list …"), says "do
what's waiting", or a scheduled task fires. Same recipe each time:

1. `select * from owed_claim(20, 'live')` (a scheduled task: `'scheduled'`). You get up to 20 rows, best first, each
   with `say` (exactly what to write) and your `run_id`. They're held for you for 2 hours; nobody else takes them.
2. Do each row as `say` says, in the reader's rules ([PAGES.md](PAGES.md), [AUTHORS.md](AUTHORS.md),
   [PLAYBOOK.md](PLAYBOOK.md)). The write closes the row; nothing else to mark.
3. Can't do one: `select owed_fail(<id>, '<one line why>')`, and go on.
4. Stop at 20 writes (one section, one why, one check, one ISBN = one write). A page you didn't finish stays in Later
   with its remaining sections.
5. End: `update runs set finished_at = now(), summary = '<what you did>. <how many left; next title>' where id = <run_id>`,
   and say the same two lines to the reader.

On a scheduled run nobody is watching: never ask the page question, never write an author page nobody said yes to
(Later only has the ones they did), and never change shelves, statuses, ratings or the reader's own words.

## What they say is data

- "I just started X" → add it (`status = 'reading'`), or update it if it's there. Its page goes in Later by itself
  (once they have page rules); "write it up" means now, in their sections ([PAGES.md](PAGES.md)).
- "Finished X, loved it / it was thin" → `status = 'finished'`, and adjust its scores. Remember the reason: it tells you
  how they judge the next book.
- "X should rank higher" → about one book: change its scores. About how they judge: change the ranking (RANKING.md)
  and say what moved.
- "Not that shelf" → move it, and ask once whether the shelf needs a new niche.
- Their goal changes → rewrite the `book_note` note (START.md, merge), then offer to re-note the top books.

## A large library

For a reader with hundreds or thousands of books ([IMPORT.md](IMPORT.md)), runs enrich a little each time, best first,
and Later already holds the order: pages for books they're reading or rated 4–5, news and whys, stories, then ISBNs
and covers last. Author pages are there only for authors the reader said yes to.

Keep each unattended run to 20 writes or fewer: that's the reader's standing limit for scheduled runs. In a session
with them, do up to 5 and put the rest in Later, unless they say "all of them now". Say in the run's summary what you
changed. Never change shelves, statuses, ratings or their own words on an unattended run.

## Keep it alive

The home page shows "Last updated by your AI" and warns after a week of silence. One small, real write a week (a
news why, a sharper note, a new book) is better than a big rewrite once a month.

## Exact SQL

What's waiting, best first:

```sql
select priority, id, kind, title, say from owed_next order by priority limit 20;
```

The reader said "later" to an author page (nothing else owes those):

```sql
insert into owed (kind, subject) values ('author_page', 'author:andrew-s-grove') on conflict do nothing;
select kind, title, state, added_by from owed where subject = 'author:andrew-s-grove';
```

A run: claim, do one, fail one, finish (each block is one request):

```sql
select * from owed_claim(20, 'live');
```

```sql
update books set isbn = '9780679762881' where slug = 'high-output-management';
```

```sql
select owed_fail((select id from owed where kind = 'cover' and subject = 'book:high-output-management' and state = 'open'),
                 'no free cover found (Open Library has none for this ISBN)');
```

```sql
update runs set finished_at = now(),
       summary = 'Added the ISBN of High Output Management. 1 left; next: Andrew S. Grove page'
 where id = (select max(id) from runs);
select kind, title, state, attempts, note from owed order by id;
```

## Updates to this manual

The manual is this repo's `main`, and it improves. At the start of every session, reread README (and the file for the
step you're on) from `main`. Where it differs from your notes or your memory, the manual wins. You never edit this
repo; it isn't yours or the reader's to change.
