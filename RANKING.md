# Ranking

Every book gets one number, `rank`, and the site orders by it. The **reader** decides what the number means. You write their system down as a ranking template, score each book against it, and the database does the arithmetic.

One ranking system per space for now.

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
Write `rank_detail = {"scores": {<factor>: 1–10, …}}` with a score for **exactly** the template's factors, plus `rank_why`: one line naming the strongest evidence. The database computes `rank` and refuses scores that don't match the factors.

- Score against the reader's world, not the general public's. A book every founder quotes and nobody else reads is a
  9 for Maya.
- Books under five years old score at most 5 on `endurance`.
- Use the calibration note: the books there anchor the scale.

## Changing it
- "This book should be higher": change that book's scores.
- "Recent books matter more to me": change the template (weights or factors) and add a calibration line. Save with `apply_existing = true` and `todo` lists every book to re-score.

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
update books set rank_detail = '{"scores": {"talk": 9, "readers": 8, "current": 5, "canon": 8, "mentor": 9}}',
       rank_why = 'Named on most founder podcasts; mentors ask about it.' where slug = 'zero-to-one';
update books set rank_detail = '{"scores": {"talk": 8, "readers": 9, "current": 4, "canon": 8, "mentor": 7}}',
       rank_why = 'Assigned in nearly every accelerator program.' where slug = 'the-lean-startup';
update books set rank_detail = '{"scores": {"talk": 7, "readers": 7, "current": 3, "canon": 9, "mentor": 6}}',
       rank_why = 'On every decision-making syllabus.' where slug = 'thinking-fast-and-slow';
```
Check:
```sql
select slug, rank, rank_why from books order by rank desc nulls last;
select * from todo where kind like '%rank%';
```
