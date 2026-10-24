# BOOKS — add a book

Every book is added this way: one request, under a minute. The reader sees it on their shelf right away, with its
cover (from the shared facts), a note on why it's for them, and a score. The database makes the author row.

**No pages here.** No book page, no author page, no news search, no cards. Those come after, in the sections the
reader chose ([PAGES.md](PAGES.md)). Ten or more books at once: [IMPORT.md](IMPORT.md).

Read the reader's goal first: `select rules ->> 'note' from rules where kind = 'book_note'`.

## Steps

1. **Is it already there?** `select slug, title, author from books where title ilike '%…%'`, or `select * from search('…')`.
   Adding the same book twice is refused and the error names the existing slug. Update that one.
2. **Look it up.** Shared facts first: `select * from book_facts where title ilike '%…%'` (ISBN-13, cover, Kindle ASIN,
   year). Then the web for what is missing: exact title, author(s), ISBN. Never invent an ISBN. Leave it null. Cover: README,
   *Covers and author photos*.
3. **Shelf and niche.** `select slug, name, niches from shelves`. Pick from what exists. No shelf → the home shelf.
   A niche must be on the shelf; the error lists the valid ones. To add one:
   `update shelves set niches = niches || '[{"id":"validation","label":"Idea and validation","sentence":"…"}]'::jsonb where slug = 'founder';`
4. **Author.** Full name(s), first author first, co-authors separated by commas. The database gives each author one
   row (and page) the moment the book is written, matched by slug, so **spell an author the same way on every book**:
   "Andrew Grove" and "Andrew S. Grove" are two people to it. Check first: `select slug, name from authors where name ilike '%grove%'`.
5. **The short note** (`note`): one paragraph, why *this* book for *this* reader, from their goal and what you know
   of them. A note that would fit anyone is wrong. Personal details only from what they told you or what's in your
   memory of them; if you have none, write to their goal. Never invent a detail to sound personal. Rules: `select rules from rules where kind = 'book_note'` (default 40–90 words, plain prose, no invented
   quotes or facts).
6. **Status:** `want` (default), `reading`, `finished`, `abandoned`. Dates fill themselves.
7. **Score it** (optional now, expected soon): `rank_detail = '{"scores": {…}}'` with a 1–10 score for every factor
   of their ranking. `rank` is computed; you can't set it. One line in `rank_why`: the strongest evidence.

Other columns the site shows: `tags` (text[]), `kindle_asin`, `cover_url`, `founder_angle`, `overlap` (which book on the
shelf covers the same ground), `why_read` (the reader's own reason only, never yours).

`slug` is made from the title if you leave it out.

## After the add

Give the link (`reading.hadmoney.com/books/<slug>`). If the reader has no page rules yet
(`select source from rules where kind = 'book_description'` says `default`), ask the page question once
([PAGES.md](PAGES.md), *The page conversation*). If they have rules, offer in one line: *"Want its page?"*

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
