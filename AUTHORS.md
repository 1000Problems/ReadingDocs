# AUTHORS — author rows, pages and news

The same steps as [ADD_BOOK.md](ADD_BOOK.md) 5, 6 and 8, for when `todo` asks for an author on its own
(`author_page:<slug>`, `author_check:<slug>`). The page is written for this reader: who this person is *to them*, and
what to say about them in the reader's conversations.

## The row
- **One author, one page**, however many of their books the reader has. The database makes the row when a book
  names the author, and never a second: same person = same slug (case, accents, Dr./Jr. ignored). The site links a
  book to every author in it the same way. You never insert authors; spell each person the same way on every book.
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
`authors.summary`, `## ` sections from `select rules from rules where kind = 'author_page'`: who, remember,
big_ideas, books, how_people_talk, critiques, cheat_sheet, sources. Required: who, big_ideas, sources (1+ URL).
Full names. Describe a face only after looking at the photo. Use the author's news in *How people talk about them*.

Recall cards: `author_cards` kinds `face`, `idea`, `who`, `book`, `taught`.

## News
1. Read first: `select url, headline, kind, published_on from author_articles where author_slug = '<slug>'`
   and `select * from news where author = '<slug>'`.
2. Search the web (last 12 months) only if `todo` lists `author_check:<slug>`. It does only for authors with a page.
3. Store finds in `author_news` (`on conflict (url) do nothing`). `author_id` is the main author; `news_authors` adds the
   others, `news_books` the books it's about. Then
   `update authors set news_checked_at = now()`. Private to this reader; the weekly job never reads it.
4. `why`: one sentence per `news_why` rules (10–40 words), tied to a book or idea on the shelf.

## Exact SQL
```sql
insert into authors (name, hook) values ('Eric Ries', '…') on conflict (slug) do update set hook = excluded.hook;
update authors set summary = '## Who they are
…

## Big ideas
- …

## Sources
- https://fortune.com/2026/05/26/eric-ries-lean-startup-incorruptible-big-tech-founders/' where slug = 'eric-ries';
select author, headline, published_on, mine, shared from news where author = 'eric-ries' order by published_on desc;
```
