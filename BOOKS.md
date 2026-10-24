# BOOKS — adding a book

This is the quick add: one row, a note, a shelf, a score. For the full add (author, news, book page, playbook) see
[ADD_BOOK.md](ADD_BOOK.md). The first book for a new reader is always a full add.

Read the reader's goal first: `select rules ->> 'note' from rules where kind = 'book_note'`.

## Steps

1. **Is it already there?** `select slug, title, author from books where title ilike '%…%'`, or `select * from search('…')`.
   Adding the same book twice is refused and the error names the existing slug. Update that one.
2. **Look it up.** Shared facts first: `select * from book_facts where title ilike '%…%'` (ISBN-13, cover, Kindle ASIN,
   year). Then the web for what is missing: exact title, author(s), ISBN, cover URL. Never invent an ISBN. Leave it null.
3. **Shelf and niche.** `select slug, name, niches from shelves`. Pick from what exists. No shelf → the home shelf.
   A niche must be on the shelf; the error lists the valid ones. To add one:
   `update shelves set niches = niches || '[{"id":"validation","label":"Idea and validation","sentence":"…"}]'::jsonb where slug = 'founder';`
4. **Author.** Full name(s), first author first. **Write `books.author` exactly as `authors.name`**: that name match is
   the only link from a book to its author page. Check: `select name from authors where name ilike '%ries%'`.
5. **The short note** (`note`): one paragraph, why *this* book for *this* reader, from their goal and what you know
   of them. A note that would fit anyone is wrong. Personal details only from what they told you or what's in your
   memory of them; if you have none, write to their goal. Never invent a detail to sound personal. Rules: `select rules from rules where kind = 'book_note'` (default 40–90 words, plain prose, no invented
   quotes or facts).
6. **Status:** `want` (default), `reading`, `finished`, `abandoned`. Dates fill themselves.
7. **Score it** (optional now, expected soon): `rank_detail = '{"scores": {…}}'` with a 1–10 score for every factor
   of their ranking. `rank` is computed; you can't set it. One line in `rank_why`: the strongest evidence.

Other columns the site shows: `tags` (text[]), `kindle_asin`, `cover_url`, `founder_angle`, `overlap` (which book on the
shelf covers the same ground), `why_read` (the reader's own reason only, never yours).

`slug` is made from the title if you leave it out. Many books in one request is fine; for imports, batches of 25.

## Exact SQL

```sql
insert into books (title, author, status, shelf, niche, note) values
  ('High Output Management', 'Andrew S. Grove', 'want', 'founder', 'people',
   'Grove wrote the manual for running a team when he ran Intel. Founders hiring their first managers quote his one-on-one and output chapters; read them so you can tell which advice still fits a ten-person AI company and where to push back.');
```

```sql
update books set rank_detail = '{"scores": {"mentor": 9, "talk": 8, "endurance": 9}}',
                 rank_why = 'The management book founders name first when they hire managers.'
 where slug = 'high-output-management';
select slug, rank, rank_why from books where slug = 'high-output-management';
```
