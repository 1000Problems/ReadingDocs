# SHELVES — the Library, shelves and subshelves

Note: niches are now subshelves.

**The Library** is every book in the space; it is the home page, ordered by `book_score` (shelf-blind, from
`rank_detail`; [RANKING.md](RANKING.md)). **Shelves** are the reader's own areas ("Founder", "Spanish kitchen"). There
are no built-in shelves: a new space has none until you make them with the reader. A book can be on many shelves.

| View | One row per | What |
|---|---|---|
| `shelves` | shelf | `slug`, `name`, `position`, `blurb` (the line the site shows), `sentence` (the same line for other AIs), `ranking_rule` (one sentence: what matters on this shelf; required before ranking), `ranked_at` (set it when you finish ranking) |
| `subshelves` | group inside a shelf | `shelf`, `slug`, `name`, `sentence`, `position`. Two levels only. Placement only: no scores |
| `shelf_books` | (shelf, subshelf, book) | `subshelf = ''` is the shelf membership (**All**): it holds `score` 1–100, `score_why`, `score_by` (`ai` or `you`, set by the database), `angle` (how to read it on this shelf), `overlap` (which book here covers the same ground). A row with a subshelf places the book there |
| `shelf_stories` | shelf or subshelf | the playbook story (`subshelf = ''` = the whole shelf); [PLAYBOOK.md](PLAYBOOK.md) |
| `shelf_roles` | author on a shelf or subshelf | one line on why they matter there |

## The rules

- **All is the shelf.** Putting a book on a subshelf adds its All row for you. Deleting the All row takes the book off
  the shelf, and its subshelf rows go with it. Upsert key: `on conflict (shelf, subshelf, book_id)`.
- **One score per shelf**, on the All row, against that shelf's `ranking_rule` ([RANKING.md](RANKING.md), *Ranking a
  shelf*). A score on a subshelf row is refused: "scores live on the shelf". `score_by = 'you'` means the reader set it
  (drag on the site); never change one without saying so.
- **Subshelves are placement.** Make them when the reader agrees (the site can make one too, even empty). A subshelf reaching 3 books with no
  story puts a `story` in Later (subject `subshelf:<shelf>/<subshelf>`; todo key `story:<shelf>:<subshelf>`).
- **Automatic subshelves** are filters the site shows on every shelf, never rows. Their slugs are reserved: `all`,
  `reading`, `finished`, `unranked`, `unplaced`, `recent`, `abandoned` (and a shelf can't be called `library` either).
  There is no "Want" filter.
- Hints: `propose_shelves` (10+ books, at most one shelf), `propose_subshelves` (10+ books on a shelf, no subshelves),
  `set_ranking_rule` (5+ books on a shelf, no rule). Offer them as `hints` says.

| Automatic | On a shelf | On the Library |
|---|---|---|
| Reading now | `status = 'reading'`, `started_at` desc | the same |
| Finished | `status = 'finished'`, `finished_at` desc | the same |
| Unranked | All-row `score` is null | unranked on some shelf |
| Unplaced | — | on no shelf |
| Recent | added to this shelf in the last 60 days | created in the last 60 days |
| Abandoned | `status = 'abandoned'` | the same |

## Exact SQL

Make a shelf with its rule, and a subshelf in it:

```sql
insert into shelves (slug, name, position, blurb, sentence, ranking_rule) values
  ('money', 'Money', 2, 'Books on how money actually works for a small company.',
   'Books on how money actually works for a small company.',
   'Will it change a money decision I make this year?')
on conflict (slug) do update set name = excluded.name, blurb = excluded.blurb,
  sentence = excluded.sentence, ranking_rule = excluded.ranking_rule;
insert into subshelves (shelf, slug, name, sentence) values
  ('money', 'mind', 'Mind', 'How founders think about risk and reward.')
on conflict (shelf, slug) do update set name = excluded.name, sentence = excluded.sentence;
```

Put a book on a shelf, place it on a subshelf (its All row is already there), score it there:

```sql
insert into shelf_books (shelf, book_id) values ('money', (select id from books where slug = 'high-output-management'))
on conflict (shelf, subshelf, book_id) do nothing;
insert into shelf_books (shelf, subshelf, book_id) values ('money', 'mind', (select id from books where slug = 'high-output-management'))
on conflict (shelf, subshelf, book_id) do nothing;
update shelf_books set score = 64, score_why = 'Output thinking applies to cash as much as to people.',
       angle = 'Read the output and leverage chapters as a budget lens.'
 where shelf = 'money' and subshelf = '' and book_id = (select id from books where slug = 'high-output-management');
```

Set or change the rule; rename a subshelf (its placements, story and roles follow); move a book off a subshelf only:

```sql
update shelves set ranking_rule = 'Will it change a money decision I make this year or next?' where slug = 'money';
update subshelves set slug = 'mindset', name = 'Mindset' where shelf = 'money' and slug = 'mind';
delete from shelf_books where shelf = 'money' and subshelf = 'mindset'
   and book_id = (select id from books where slug = 'high-output-management');
```

Read a shelf, its subshelves and what each book is on:

```sql
select slug, name, blurb, ranking_rule, ranked_at from shelves order by position;
select shelf, slug, name, sentence from subshelves order by shelf, position;
select sb.shelf, sb.subshelf, b.slug, sb.score, sb.score_by from shelf_books sb join books b on b.id = sb.book_id
 order by sb.shelf, sb.subshelf, sb.score desc nulls last;
```

The automatic subshelves, on a shelf:

```sql
-- Reading now / Finished / Abandoned (swap the status and the date)
select b.slug, b.title from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'money' and sb.subshelf = '' and b.status = 'reading' order by b.started_at desc;
-- Unranked
select b.slug, b.title from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'money' and sb.subshelf = '' and sb.score is null;
-- Recent
select b.slug, b.title from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'money' and sb.subshelf = '' and sb.added_at > now() - interval '60 days' order by sb.added_at desc;
```

And on the Library:

```sql
-- Unranked: unranked on some shelf
select b.slug, b.title from books b
 where exists (select 1 from shelf_books sb where sb.book_id = b.id and sb.subshelf = '' and sb.score is null);
-- Unplaced: on no shelf
select b.slug, b.title from books b where not exists (select 1 from shelf_books sb where sb.book_id = b.id);
-- Recent
select slug, title from books where created_at > now() - interval '60 days' order by created_at desc;
```

Take a book off a shelf (its subshelf rows go too), then delete the subshelf and the shelf (only when the reader
asks):

```sql
delete from shelf_books where shelf = 'money' and subshelf = ''
   and book_id = (select id from books where slug = 'high-output-management');
delete from subshelves where shelf = 'money' and slug = 'mindset';
delete from shelves where slug = 'money';
```
