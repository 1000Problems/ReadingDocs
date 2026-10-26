# Ranking

One kind of number: **a book's score on a shelf** (`shelf_books.score`, 1–100, on the All row), against that shelf's
`ranking_rule`. It orders the shelf. The Library is a shelf too ([SHELVES.md](SHELVES.md)), so every book has a Library
score. **Adding = scoring:** a book lands on a shelf with its score in the same request; an unscored membership fails
the request at commit.

The **ranking template** below is how the reader's own system scores the **Library**: you score each book's factors
in `rank_detail`, and the database turns them into its Library score. One ranking template per space.

## The template
One row in `templates` with `kind = 'ranking'`. Its `rules`:

| Key | What |
|---|---|
| `score_name` | what the reader calls the number: "Importance", "Ask", "Read next" |
| `score_sentence` | what it means, one sentence, in their words |
| `factors` | 2–6 of `[{"id", "weight"}]`; weights are whole numbers summing to 100 |
| `note` | calibration examples, at most 300 characters: "Zero to One = 9 on mentor: every founder conversation" |

Factors come from the catalog's list (`select factors from catalog where kind = 'ranking'`): `talk`, `readers`, `current`, `canon`, `mentor`, `influence`, `endurance`, `experts`, `reach`. Each has a label and a one-line meaning there. Custom factors come later.

Until the reader saves one, the default applies (`select rules from rules where kind = 'ranking'`): **Importance** = influence 30 · canon 25 · experts 20 · endurance 15 · reach 10.

The database refuses a template with unknown factors, fewer than 2 or more than 6, or weights that don't sum to 100. Every save keeps a version (`template_history`).

## Setting it up with the reader
In the first session, draft it from what you know (START.md). Revisit it once there are 10+ books, or when they ask
"why is X above Y?".
1. Ask: "When you pick the next book, what makes one win?" Listen for the factors in their answer: being asked about it (`talk`, `mentor`), everyone has read it (`readers`), it's new (`current`), it's a classic (`canon`, `endurance`), it changed its field (`influence`).
2. Propose a template in one message: name, sentence, 3–5 factors with weights. Show the top 5 books it would produce.
3. Adjust until they nod. Save it. Put the two or three books that defined the scale in `note`.
4. Score every book.

## Scoring a book
Write `rank_detail = {"scores": {<factor>: 1–10, …}}` with a score for **exactly** the template's factors, **in the
same request that adds the book** (or right after, same request). The database computes the parts and sets the book's
**Library score** from them (unless the reader set that score by hand), and refuses scores that don't match the
factors. A new book with neither `rank_detail` nor a Library score fails the request.

- Score against the reader's world, not the general public's. A book every founder quotes and nobody else reads is a
  9 for Maya.
- Books under five years old score at most 5 on `endurance`.
- Use the calibration note: the books there anchor the scale.

## Changing it
- "This book should be higher": change that book's scores.
- "Recent books matter more to me": change the template and add a calibration line.
  - **Only weights (or names, sentence, note) change:** every Library score is recomputed in the same change, from the
    scores the books already have. Nothing to re-score. Undo puts every score back, until another book is edited.
  - **The factors change:** the old scores no longer fit. Save with `apply_existing = true` and every scored book gets
    a `rescore` row in Later; runs re-score them, best first. Re-scoring a book closes its row.

## Ranking a shelf
One score per book per shelf, on its All row (`subshelf = ''`). Subshelves have no scores. When the reader asks to rank
a shelf (or the site's *Rank this shelf*):

1. Read the shelf's `ranking_rule` and `blurb`. No rule → ask the reader what matters on this shelf, save it as one
   sentence (`update shelves set ranking_rule = …`), then go on.
2. Read its books: `shelf_books where shelf = … and subshelf = ''` joined to `books` (`rank_detail`, the current
   `score`, `score_why`, `score_by`, `angle`), and each book's Library score.
3. Score pairwise against the rule: is this book above or below that one, *for this rule*? Use the whole 1–100 range;
   don't bunch everything at 70–90. The Library score is a hint, not the answer. The Library itself is ranked
   the same way, against its own rule.
4. Never change a `score_by = 'you'` row (the reader set it, e.g. by drag on the site) without saying so in the list.
5. Write or refresh `angle` where it's empty: how to read this book *on this shelf*. `score_why`: one line, the
   strongest evidence against the rule.
6. Show the full list (score, title, one-line why) and wait for a yes.
7. Write it in one request, and set `shelves.ranked_at = now()`.

"Rank the new ones" is the same for books with a null score only, placed among the existing scores without moving them.

## Exact SQL
What's on offer and what's active:
```sql
select factors, "default" from catalog where kind = 'ranking';
select source, version, rules from rules where kind = 'ranking';
```
Save the reader's template (an upsert: run it again to change it):
```sql
insert into templates (kind, rules, apply_existing) values ('ranking', '{
  "score_name": "Ask",
  "score_sentence": "How likely I am to be asked whether I have read it.",
  "factors": [
    {"id": "talk", "weight": 30}, {"id": "readers", "weight": 20}, {"id": "current", "weight": 15},
    {"id": "canon", "weight": 15}, {"id": "mentor", "weight": 20}
  ],
  "note": "Zero to One = 9 on mentor: every founder conversation. The Lean Startup = 9 on readers: assigned in every program."
}', true)
on conflict (kind) do update set rules = excluded.rules, apply_existing = excluded.apply_existing;
```
Score books (one change for the lot):
```sql
update books set rank_detail = '{"scores": {"talk": 9, "readers": 8, "current": 5, "canon": 8, "mentor": 9}}'
 where slug = 'zero-to-one';
update books set rank_detail = '{"scores": {"talk": 8, "readers": 9, "current": 4, "canon": 8, "mentor": 7}}'
 where slug = 'the-lean-startup';
update books set rank_detail = '{"scores": {"talk": 7, "readers": 7, "current": 3, "canon": 9, "mentor": 6}}'
 where slug = 'thinking-fast-and-slow';
```
Check:
```sql
select b.slug, sb.score from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'library' and sb.subshelf = '' order by sb.score desc nulls last;
select title, state from owed where kind = 'rescore' order by id;
```
Rank a shelf: read the rule and the books, then (after the reader's yes) write every score in one request:
```sql
select slug, name, blurb, ranking_rule, ranked_at from shelves where slug = 'founder';
select b.slug, b.title, lib.score as library_score, b.rank_detail, sb.score, sb.score_why, sb.score_by, sb.angle
  from shelf_books sb join books b on b.id = sb.book_id
  left join shelf_books lib on lib.book_id = b.id and lib.shelf = 'library' and lib.subshelf = ''
 where sb.shelf = 'founder' and sb.subshelf = '' order by sb.score desc nulls last, lib.score desc nulls last;
```
```sql
update shelf_books set score = 92, score_why = 'Named on most founder podcasts; mentors ask about it.',
       angle = 'Have a view on the monopoly chapter; founders will test it.'
 where shelf = 'founder' and subshelf = '' and book_id = (select id from books where slug = 'zero-to-one');
update shelf_books set score = 85, score_why = 'Assigned in nearly every accelerator program.'
 where shelf = 'founder' and subshelf = '' and book_id = (select id from books where slug = 'the-lean-startup');
update shelves set ranked_at = now() where slug = 'founder';
```
