# AUTHORS — author rows, pages and news

The same steps as [PAGES.md](PAGES.md) 5, 6 and 8, in the sections the reader chose, for when a run hands you an author's
Later row on its own (`author_page`, `news_check`). The page is written for this reader: who this person is *to them*, and
what to say about them in the reader's conversations.

## The row
- **One author, one page**, however many of their books the reader has. The database makes the row when a book
  names the author, and never a second: same person = same slug (case, accents, Dr./Jr. ignored). The site links a
  book to every author in it the same way. You never insert authors; spell each person the same way on every book.
- **Two people, one name.** Same slug means one page, so two different John Smiths would share it. When you know they
  are different people, spell the second one the way their books or Wikipedia tell them apart (a middle initial or
  full middle name) on every one of their books. Not sure they're different? Ask the reader.
- **A library from before the database did this** (books whose authors have no row): one request, safe to re-run:
  ```sql
  insert into authors (name)
  select distinct trim(a) from books, unnest(string_to_array(author, ',')) a where trim(a) <> ''
  on conflict (slug) do nothing;
  ```
  (It splits on commas only: rewrite `&` / `and` between co-authors to commas first.)
- Photo: per README, *Covers and author photos* (shared first; else the Wikipedia image, now; else empty).
- Facts (`wiki_url`, `photo_url`, `born`, `pronunciation`) only from a page you opened. Fill when empty or wrong;
  never overwrite with a guess. `author_facts.wikipedia` / `site` may already have them.
- `hook`: one line to remember them by.

## The page
Author pages are owed by themselves only once the reader has an author page format (`templates`, kind `author_page`):
then every author with 2+ of their books and no page is in Later (derived). Before that, one exists in Later only
because the reader said yes. A yes for later, or for many ("pages for every author I own 2+ books by"), is one row
each, then one line (*"12 author pages are in Later."*):
```
insert into owed (kind, subject)
select 'author_page', 'author:' || author_slug from book_authors
 group by author_slug having count(*) >= 2
on conflict do nothing;
```

Only after the reader said yes to it, in their `author_page` rules ([PAGES.md](PAGES.md), *The page conversation*),
one section per request through `author_sections`, only the sections they picked (a Sources section needs 1+ URL).
Write the whole first pass back to back, no drafts in chat, then one message and shape it with them
([PAGES.md](PAGES.md), *Write, then shape*).
Full names. Describe a face only after looking at the photo. Use the author's news in *How people talk about them*.

Recall cards: `author_cards` kinds `face`, `idea`, `who`, `book`, `taught`.

## News
1. Read first: `select url, headline, kind, published_on from author_articles where author_slug = '<slug>'`
   and `select * from news where author = '<slug>'`.
2. Search the web (last 12 months) **always before an author's first page** (no `summary`, no `news_checked_at`).
   After that, only when a run hands you its Later news check. Later has one only for authors with a page, 30 days
   after the last check.
3. Store finds in `author_news` (`on conflict (url) do nothing`). `author_id` is the main author; `news_authors` adds the
   others, `news_books` the books it's about. Then
   `update authors set news_checked_at = now()` (this closes the Later check and schedules the next one in 30 days).
   Private to this reader; the weekly job never reads it.
4. `why`: one sentence per `news_why` rules (10–40 words), tied to a book or idea on the shelf.
5. **News or article?** The site shows an item under **Articles** when `article_kind` is set, else under **News**.
   Something to sit down and read or hear (an essay, review, analysis, opinion piece, long profile, long written
   interview, podcast or video episode) is an article: `kind = 'news'` and `article_kind` = `essay`, `review`,
   `interview`, `excerpt`, `article` or `podcast` (the labels the site knows; a video episode is `podcast`).
   Something that happened (a book released, a talk given, a prize, funding, an event coming up) is news: `kind` =
   `news`, `talk`, `award`, `upcoming` or `interview` (a short one), with `article_kind` null.

## Exact SQL
```sql
insert into authors (name, hook) values ('Eric Ries', '…') on conflict (slug) do update set hook = excluded.hook;
insert into author_sections (author_slug, section_id, body) values ('eric-ries', 'who', '…');
insert into author_sections (author_slug, section_id, body) values ('eric-ries', 'sources',
  '- https://fortune.com/2026/05/26/eric-ries-lean-startup-incorruptible-big-tech-founders/');
select sections, missing from pages where kind = 'author' and slug = 'eric-ries';
select author, headline, published_on, mine, shared from news where author = 'eric-ries' order by published_on desc;
```
