# BOOKS — adding a book

## Steps

1. **Is it already there?** `select slug, title, author from books where title ilike '%…%'`, or `select * from search('…')`.
   Adding the same book twice is refused and the error names the existing slug. Update that one.
2. **Look it up** if you can: exact title, author(s), ISBN, cover URL. Never invent an ISBN. Leave it null.
3. **Shelf and niche.** `select slug, name, niches from shelves`. Pick from what exists. No shelf → the home shelf.
   A niche must be on the shelf; the error lists the valid ones.
4. **The short note** (`note`): one paragraph, why *this* book for *this* reader, from what they said and what you know
   of them. Rules: `select rules from rules where kind = 'book_note'` (default 40–90 words, plain prose, no invented
   quotes or facts).
5. **Status:** `want` (default), `reading`, `finished`, `abandoned`. Dates fill themselves.
6. **Score it** (optional now, expected soon): `rank_detail = '{"scores": {…}}'` with a 1–10 score for every factor
   of their ranking. `rank` is computed; you can't set it. One line in `rank_why`: the strongest evidence.

`slug` is made from the title if you leave it out. Many books in one request is fine; for imports, batches of 25.

## Exact SQL

```sql
insert into books (title, author, status, shelf, niche, note) values
  ('High Output Management', 'Andrew S. Grove', 'want', 'founder', 'people',
   'Grove wrote the manual for running a team when he ran Intel. You keep asking how to give feedback without slowing people down; his one-on-one and output chapters are the practical answer, and every operator you learn from has read it.');
```

```sql
update books set rank_detail = '{"scores": {"mentor": 9, "talk": 8, "endurance": 9}}',
                 rank_why = 'Named by most operators as the management book.'
 where slug = 'high-output-management';
select slug, rank, rank_detail -> 'name' as score from books where slug = 'high-output-management';
```
