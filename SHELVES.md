# SHELVES — the Library, shelves and subshelves

Note: niches are now subshelves.

**The Library is a shelf** (slug `library`). Every space has it; every book is on it (the database puts each new book
there); it has its own `ranking_rule` ("How much this book matters to me overall." until the reader changes it), its own
score per book, and its own subshelves. It is the home page. It can be renamed, never deleted, and its slug stays
`library`. **Other shelves** are the reader's own areas ("Founder", "Spanish kitchen"); a book can be on many.

**Every shelf is always in score order**, and **adding = scoring**: a book lands on a shelf with its score, in the same
request. A membership still unscored when the request ends fails the whole request ("score it on arrival"). Only you
(the AI) add books and put them on shelves; the site can't.

| View | One row per | What |
|---|---|---|
| `shelves` | shelf | `slug`, `name`, `position` (the Library is -1, first), `blurb` (the line the site shows), `sentence` (the same line for other AIs), `ranking_rule` (one sentence: what matters on this shelf), `ranked_at` (set it when you finish ranking) |
| `subshelves` | group inside a shelf | `shelf`, `slug`, `name`, `sentence`, `position`. One level, on any shelf including the Library. Placement only: no scores |
| `shelf_books` | (shelf, subshelf, book) | `subshelf = ''` is the shelf membership (**All**): it holds `score` 1–100, `score_why`, `score_by` (`ai` or `you`, set by the database), `angle` (how to read it on this shelf), `overlap` (which book here covers the same ground). A row with a subshelf places the book there |
| `shelf_stories` | shelf or subshelf | the playbook story (`subshelf = ''` = the whole shelf); [PLAYBOOK.md](PLAYBOOK.md) |
| `shelf_roles` | author on a shelf or subshelf | one line on why they matter there |

## The rules

- **All is the shelf.** Putting a book on a subshelf adds its All row for you (score that row in the same request).
  Deleting the All row takes the book off the shelf, and its subshelf rows go with it. The Library's All row can't be
  deleted: delete the book instead. Upsert key: `on conflict (shelf, subshelf, book_id)`.
- **One score per shelf**, on the All row, against that shelf's `ranking_rule` ([RANKING.md](RANKING.md), *Ranking a
  shelf*). A score on a subshelf row is refused: "scores live on the shelf". `score_by = 'you'` means the reader set it
  (drag on the site); never change one without saying so.
- **The Library score** comes from the book's `rank_detail` (the reader's ranking factors, [RANKING.md](RANKING.md)),
  or you write it like any other shelf score. A score the reader set is kept when `rank_detail` changes.
- **Subshelves are placement.** Make them when the reader agrees. A subshelf reaching 3 books with no story puts a
  `story` in Later (subject `subshelf:<shelf>/<subshelf>`; todo key `story:<shelf>:<subshelf>`).
- **Subshelves are areas** (how the reader organizes a shelf), never states. **Book states are a filter**: the site has
  a *Show* menu (Up next · Reading · Finished · Set aside) over every shelf and subshelf, and each cover shows its state
  (Reading outlined, Finished ✓, Set aside faded). The hero on top of any shelf is the top 3 by score of what is shown.
- **Automatic subshelves** left: Unplaced (Library only) and Recent, filters the site shows, never rows. Reserved slugs
  (so old links resolve): `all`, `reading`, `finished`, `unranked`, `unplaced`, `recent`, `abandoned`, `library`.
- Hints: `propose_shelves` (10+ books, at most one shelf besides the Library), `propose_subshelves` (10+ books on a
  shelf, no subshelves), `set_ranking_rule` (5+ books, no rule), `rank_unranked` (books on a shelf with a placeholder score:
  rank them). Offer them as `hints` says.

| Automatic | Filter (on any shelf, the Library included) |
|---|---|
| Unplaced | Library only: on no shelf but the Library |
| Recent | added to this shelf in the last 60 days |

| Show (filter) | Books |
|---|---|
| Up next / Reading / Finished | `status = 'want'` / `'reading'` / `'finished'` |
| Set aside | `set_aside`; unchecked hides set-aside books at any status |

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

Put a book on a shelf **with its score**, then place it on a subshelf (one request):

```sql
insert into shelf_books (shelf, book_id, score, score_why, angle) values
  ('money', (select id from books where slug = 'high-output-management'), 64,
   'Output thinking applies to cash as much as to people.', 'Read the output and leverage chapters as a budget lens.')
on conflict (shelf, subshelf, book_id) do update set score = excluded.score, score_why = excluded.score_why, angle = excluded.angle;
insert into shelf_books (shelf, subshelf, book_id) values ('money', 'mind', (select id from books where slug = 'high-output-management'))
on conflict (shelf, subshelf, book_id) do nothing;
```

Score or re-score a book on the Library directly (when you don't use `rank_detail`):

```sql
update shelf_books set score = 88, score_why = 'The book he quotes most; shaped how he runs 1:1s.'
 where shelf = 'library' and subshelf = '' and book_id = (select id from books where slug = 'high-output-management');
```

Set or change the rule; rename a subshelf (its placements, story and roles follow); move a book off a subshelf only:

```sql
update shelves set ranking_rule = 'Will it change a money decision I make this year or next?' where slug = 'money';
update subshelves set slug = 'mindset', name = 'Mindset' where shelf = 'money' and slug = 'mind';
delete from shelf_books where shelf = 'money' and subshelf = 'mindset'
   and book_id = (select id from books where slug = 'high-output-management');
```

Read the shelves (Library first), their subshelves and what each book is on, in score order:

```sql
select slug, name, blurb, ranking_rule, ranked_at from shelves order by position;
select shelf, slug, name, sentence from subshelves order by shelf, position;
select sb.shelf, sb.subshelf, b.slug, sb.score, sb.score_by from shelf_books sb join books b on b.id = sb.book_id
 order by sb.shelf, sb.subshelf, sb.score desc nulls last;
```

The automatic subshelves (any shelf; `library` for the Library):

```sql
-- What the site's hero shows with Show = Up next + Reading: top 3 by score, set-aside books out
select b.slug, b.title, sb.score from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'money' and sb.subshelf = '' and b.status in ('want', 'reading') and not b.set_aside
 order by sb.score desc limit 3;
-- Placeholders: rank these
select b.slug, b.title from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'library' and sb.subshelf = '' and sb.score_by = 'default';
-- Recent
select b.slug, b.title from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'money' and sb.subshelf = '' and sb.added_at > now() - interval '60 days' order by sb.added_at desc;
-- Unplaced (Library only): on no shelf but the Library
select b.slug, b.title from books b
 where not exists (select 1 from shelf_books sb where sb.book_id = b.id and sb.shelf <> 'library');
```

Take a book off a shelf (its subshelf rows go too), then delete the subshelf and the shelf (only when the reader
asks; the Library can't be deleted):

```sql
delete from shelf_books where shelf = 'money' and subshelf = ''
   and book_id = (select id from books where slug = 'high-output-management');
delete from subshelves where shelf = 'money' and slug = 'mindset';
delete from shelves where slug = 'money';
```
