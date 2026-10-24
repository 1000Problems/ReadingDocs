# START — the first session

**Goal:** when this session ends, the user opens Reading and it already looks like *theirs*:
their shelves, their way of judging books, books that fit their life, notes in their words.

The site has no AI and no labels of its own. It renders only what you write. A blank shelf is a blank site.

## 1. Start from what you know

Before asking anything, use what you already know about this person: their work, goals, projects, the people they
learn from, books or authors they have mentioned, what they are trying to get better at. Memory, earlier
conversations, files they have shared: all of it counts.

From that, draft their Reading:

- **Name** — what the site calls them (`profiles.display_name`).
- **3–5 shelves** — the areas of their life where reading matters (e.g. "Founder", "Health", "Spanish kitchen").
  One sentence each on what belongs there. One shelf is `home`.
- **Niches** — 2–5 per shelf, the sub-topics they actually care about.
- **A ranking** — how *they* judge whether a book matters. Pick 2–6 factors from `rules where kind = 'ranking'`
  and weights that sum to 100. Name it in their words ("Will a mentor ask?" beats "Importance").
- **5–10 first books** — ones they've mentioned, are reading, or that fit a goal you know of.

If you know very little, say so plainly and ask before drafting.

**Write the goal down.** What the reader reads for (e.g. "talk with people starting AI startups") is lost after the
session unless you store it. Until the profile has a field for it, put it in the home shelf's `sentence` and in the
`note` of the `book_note` and `drill_cards` rules. Every later AI reads it from `select kind, rules -> 'note' from rules`.

## 2. Show the plan, ask at most two questions

Show the draft in a few lines. Ask only what you can't infer: usually *"what are you reading now?"* and
*"what makes a book worth your time?"*. Don't interview them. Then wait for a yes or for corrections.

## 3. Write it

In this order, one request per step so each can be undone alone:

1. profile, shelves and niches
2. the ranking template
3. books, with a short note each (see [BOOKS.md](BOOKS.md)), then their scores

Then `select * from todo order by priority` and tell them, in one line each, what you could do next.

## 4. Keep it theirs

- Notes are written for this reader, from what they told you and what you know about them. Never generic.
- "This book should rank higher": about one book → change its scores. About how they judge → change the ranking
  template and say what moved.
- Learn as you go. When they react to a book, that tells you about their taste. Use it on the next one.

## Exact SQL

Profile, one home shelf with two niches, and a second shelf. A new space already has a home shelf `books`; unset it
first. `sentence` is the one line on what belongs on the shelf:

```sql
update shelves set home = false where home;
update profiles set display_name = 'Angel';
insert into profiles (display_name) select 'Angel' where not exists (select 1 from profiles);
insert into shelves (slug, name, position, home, blurb, niches) values
  ('founder', 'Founder', 0, true, 'Building a company: people, product, money.',
   '[{"id":"people","label":"People"},{"id":"product","label":"Product"}]'),
  ('kitchen', 'Spanish kitchen', 1, false, 'Cooking from home, proportions right.', '[]');
update shelves set sentence = blurb where sentence is null;
```

A ranking in the user's words: 3 factors, weights sum to 100:

```sql
insert into templates (kind, rules) values ('ranking', '{
  "score_name": "Will a mentor ask?",
  "score_sentence": "How likely someone I learn from asks whether I have read it.",
  "factors": [{"id":"mentor","weight":50},{"id":"talk","weight":30},{"id":"endurance","weight":20}],
  "note": "Zero to One = 9 on mentor: comes up in every founder conversation."
}') on conflict (kind) do update set rules = excluded.rules;
```
