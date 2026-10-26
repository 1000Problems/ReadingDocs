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
3. **Shelves.** `select slug, name, ranking_rule from shelves`. Put it on the shelf(s) it belongs to, and on a
   subshelf if one fits (`select shelf, slug, name from subshelves`): one `shelf_books` row each ([SHELVES.md](SHELVES.md)).
   Every book is on the Library already (the database puts it there). No other shelves yet → make the first one with
   the reader (START.md).
4. **Author.** Full name(s), first author first, co-authors separated by commas. The database gives each author one
   row (and page) the moment the book is written, matched by slug, so **spell an author the same way on every book**:
   "Andrew Grove" and "Andrew S. Grove" are two people to it. Check first: `select slug, name from authors where name ilike '%grove%'`.
5. **The short note** (`note`): one paragraph, why *this* book for *this* reader, from their goal and what you know
   of them. A note that would fit anyone is wrong. Personal details only from what they told you or what's in your
   memory of them; if you have none, write to their goal. Never invent a detail to sound personal. Rules: `select rules from rules where kind = 'book_note'` (default 40–90 words, plain prose, no invented
   quotes or facts).
6. **Status:** `want` (default), `reading`, `finished`, `abandoned`. Dates fill themselves.
7. **Score it, in the same request (required).** Adding = scoring. `rank_detail = '{"scores": {…}}'` with a 1–10
   score for every factor of their ranking: that sets its **Library** score. On each other shelf it's on, a score
   1–100 against that shelf's rule, with one line in `score_why` and an `angle` (how to read it on this shelf), on
   the All row ([RANKING.md](RANKING.md), *Ranking a shelf*). A book or membership still unscored when the request
   ends fails the whole request ("score it on arrival"); nothing is kept.

Other columns the site shows: `tags` (text[]), `kindle_asin`, `cover_url`, `why_read` (the reader's own reason only,
never yours); on `shelf_books`: `angle`, `overlap` (which book on the shelf covers the same ground).

`slug` is made from the title if you leave it out.

## After the add

Give two links: the book page and the shelf it's on (`reading.hadmoney.com/books/<slug>` and
`reading.hadmoney.com/shelves/<shelf slug>`). You did a lot of work in one request; the reader should see it. Every
later change gets its own link too (README, *A link with every change*).

**The first add on an empty space** (no plan agreed yet, or the reader only said "add this book"): say in the same
message, in one line each, the shelf name you chose and that scores use the default ranking until they tell you what
makes a book worth their time. The shelf name is shown, not asked: *"I called your shelf "Catching up"; rename it
anytime."* They correct it in plain words.

If the reader has no page rules yet
(`select source from rules where kind = 'book_description'` says `default`), ask the page question once
([PAGES.md](PAGES.md), *The page conversation*). If they have rules, don't offer the page. A book they're reading or
rated 4–5 is put in Later by the database: end with one line, *"Its page is in Later; say "write it up" for it now."*
Any other book: say nothing about pages. "Write it up" means now ([PAGES.md](PAGES.md), *Write, then shape*).

## Exact SQL

One request: the book with its factor scores (its Library score), then its shelf with that shelf's score, then the
subshelf.

```sql
insert into books (title, author, status, note, rank_detail) values
  ('High Output Management', 'Andrew S. Grove', 'want',
   'Grove wrote the manual for running a team when he ran Intel. Founders hiring their first managers quote his one-on-one and output chapters; read them so you can tell which advice still fits a ten-person AI company and where to push back.',
   '{"scores": {"mentor": 9, "talk": 8, "endurance": 9}}');
insert into shelf_books (shelf, book_id, score, score_why, angle)
values ('founder', (select id from books where slug = 'high-output-management'), 88,
        'The management book founders name first when they hire managers.',
        'Read the one-on-one chapters before your first manager hire.')
on conflict (shelf, subshelf, book_id) do update set score = excluded.score, score_why = excluded.score_why, angle = excluded.angle;
insert into shelf_books (shelf, subshelf, book_id)
values ('founder', 'people', (select id from books where slug = 'high-output-management'))
on conflict (shelf, subshelf, book_id) do nothing;
```

```sql
select b.slug, sb.shelf, sb.score, sb.score_why from books b
  join shelf_books sb on sb.book_id = b.id and sb.subshelf = '' where b.slug = 'high-output-management';
```
