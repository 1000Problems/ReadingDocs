# START — the first session

**Goal:** when this session ends, the reader opens Reading and it is unmistakably *theirs*: their shelves, their way
of judging books, books that fit their life, notes written to them. Nobody else has this site, because nobody else
is this person.

The site renders only what you write. A blank shelf is a blank site; a generic note is a generic site. Every line
you write is either about this reader or it shouldn't be there.

## Two kinds of first session

- **A reader who wants a few books to matter** (most readers): this page, from section 1.
- **A reader who arrives with a library** (hundreds or thousands of books, a Goodreads export, photos of their
  bookcases): connect, take their goal, then follow [IMPORT.md](IMPORT.md). Their first win is the whole library on
  the site, correct; rich pages come later.

If you can't tell which, ask one question: *"About how many books do you want on the site?"*

## They already asked for a book

Often the first thing a reader says is "add The Lean Startup", before any goal. If you know them, take the goal from
the request and what you know ("You're reading startup books; I'll write for talking with founders") and put it in
the plan in section 2 as one line they can correct. If you don't know them, ask for the goal in the plan (section 2).
Either way, make that book the first book, and write nothing until they've seen the plan.

## 1. Start from what you know

Before asking anything, use what you already know: their work, goals, projects, the people they learn from, books or
authors they've mentioned, what they're trying to get better at. Memory, earlier conversations, shared files: all
of it counts. This is what makes the first visit feel like magic, so spend effort here.

Draft their Reading:

- **Name** — what the site calls them (`profiles.display_name`).
- **Goal** — one or two sentences: what they read *for*, in their words. ("Talk credibly with people starting AI
  companies." "Cook from home without recipes.") Everything else is written for this.
- **3–5 shelves** — the areas of their life where reading matters. Name them the way they talk ("Founder",
  "Spanish kitchen", "Raising Lucía"). Never "Books", "Main" or "General". One line each on what belongs there. One
  shelf is `home`.
- **Niches** — the sub-topics they actually care about. Make a niche only when a book goes into it; no empty ones.
- **A ranking** — how *they* decide a book matters. 2–6 factors from the catalog, weights summing to 100, named in
  their words ("Will a mentor ask?" beats "Importance"). See [RANKING.md](RANKING.md).
- **The first book** — the one that will make them say "yes, that's me": a book they're reading, one they mentioned,
  or the obvious one for their goal. One book. Suggest more only in scene 4, and add them only on a yes.

Anything you don't know, ask; don't guess. If you know very little, say so plainly, draft what you can (the book
itself is a clue), and ask for the rest in the plan. Their name: use it if you know it, otherwise ask. Never put the
handle or a guess in `display_name`.

**They open with "add this book."** Plan it as the first book ([BOOKS.md](BOOKS.md)) on a shelf named for what the
book says about them. Show the plan first (section 2), then build it in scenes.

## 2. Show the plan, ask at most two questions

Show the draft in a few lines. Ask only what you can't infer, and never more than two questions. Then wait for a yes
or for corrections. Their corrections are the best data you'll get: use their exact words in the shelf lines and the
ranking.

- **You don't know them:** the two questions are their name and their goal. Save *"what are you reading now?"* and
  *"what makes a book worth your time?"* for after the first book (scene 4).
- **You know them:** usually *"what are you reading now?"* and *"what makes a book worth your time?"*.

If the goal is still unknown, don't guess a ranking: say in the plan that you'll set it once they've told you their
goal, and keep the space's default until then.

When you know little but have a goal, put the proposed ranking in the plan as one line they can correct: *Ranking:
"Will a founder ask?": a founder I advise asks about it 50, comes up in their conversations 30, still read in ten
years 20.* If the goal only arrives with their answers, say that line as you save it in scene 1. Never set a ranking
they haven't seen.

## 3. Build it in scenes

Each scene is one request (one change they can undo) and ends with a link to where it shows (README, *A link with
every change*). Tell them what to open, then go on.

**Scene 1 — the frame.** Profile, goal, shelves and niches, ranking (or the default, if the goal is still unknown). Link: `reading.hadmoney.com/`.

Shelf name and ranking on a first add: if the reader only said "add this book" and you don't know enough to plan
shelves, name the home shelf yourself from what you know and **show** it, don't ask: one line in the plan, or right
after the add (*"I called your shelf "Catching up"; rename it anytime. Scores use the default ranking until you tell
me what makes a book worth your time."*).

Adding books is fast; pages come after, the way the reader wants them. Don't write a page, search the web for news
or write an author page until the reader has said what they want to see.

**Scene 2 — the first book.** [BOOKS.md](BOOKS.md): the row, the note, the score, in one request, in under a minute.
Links: the book page and its shelf, `/books/<slug>` and `/shelves/<shelf slug>`. Make the note speak to their goal and to what they've actually told you. Never invent a detail
about them to sound personal.

**Scene 3 — what they want to see.** Ask the page question once ([PAGES.md](PAGES.md), *The page conversation*),
save their answer as their page rules (say the order back as you save), then write the first pass of **this one**
book page, every section, straight to the site. No drafts in chat. Then one message: *"Your page is up"*, the link,
one line per section, and an invitation to react in plain words. Shape it from what they say ([PAGES.md](PAGES.md),
*Write, then shape*). This is the page they'll judge you by. If they'd rather see pages later, nothing to write: a book they're reading
or rated 4–5 is already in Later (README, *Later*); for any other book, put its page there
(`insert into owed (kind, subject) values ('page', 'book:<slug>') on conflict do nothing`) and say so in one line.
Offer the author page once (*"Want a page for Eric Ries?"*); build it only on a yes, and a "later" puts it in Later.

**Scene 4 — more books, if they want.** Suggest 3–5 that fit their goal, in one line each. On a yes, the same fast
add, all in one request, no pages. Link: the home
page again, now ranked. Their pages wait until they ask, or until they start reading one. If one niche now has two authors, write its playbook
([PLAYBOOK.md](PLAYBOOK.md)).
If you saved the two book questions for later, ask them here, with the first book on screen.

## 4. The reveal

End with three or four lines, not a report:

- what you built, with the two or three best links;
- one thing you noticed about them from the books ("Four of your eight are about hiring. Want a People niche?");
- the day-two sentence: *"Next time just say 'add Zero to One' or 'update my reading'."*;
- if your product supports scheduled tasks, offer a weekly one that runs [DAILY.md](DAILY.md). The home page warns
  the reader when their AI hasn't written in over a week.

Don't list what's waiting: the site shows it under **Later** and offers things one at a time.

## Exact SQL

Scene 1, one request. A new space already has a home shelf `books`; unset it first. `blurb` is the line the site shows
under a shelf; `sentence` is the same line for other AIs. Write both.

```sql
update shelves set home = false where home;
update profiles set display_name = 'Maya';
insert into profiles (display_name) select 'Maya' where not exists (select 1 from profiles);
insert into shelves (slug, name, position, home, blurb, sentence, niches) values
  ('founder', 'Founder', 0, true,
   'Books the AI founders I advise expect me to have read.',
   'Books the AI founders I advise expect me to have read.',
   '[{"id":"idea","label":"Idea and validation","sentence":"Testing whether anyone wants it before building it."},
     {"id":"people","label":"People","sentence":"Hiring, managing, firing."}]'),
  ('kitchen', 'Spanish kitchen', 1, false,
   'Cooking from home, proportions right.', 'Cooking from home, proportions right.', '[]')
on conflict (slug) do update set home = excluded.home, blurb = excluded.blurb,
  sentence = excluded.sentence, niches = excluded.niches;

-- the goal: merged into the active book_note rules, so the other settings stay
insert into templates (kind, rules)
select kind, rules || '{"note": "Maya advises early-stage AI founders and wants to talk credibly with them about the books they quote. Write every note for that conversation."}'::jsonb
  from rules where kind = 'book_note'
on conflict (kind) do update set rules = excluded.rules;

-- the ranking, saved whole
insert into templates (kind, rules) values ('ranking', '{
  "score_name": "Will a founder ask?",
  "score_sentence": "How likely a founder I advise asks whether I have read it.",
  "factors": [{"id":"mentor","weight":50},{"id":"talk","weight":30},{"id":"endurance","weight":20}],
  "note": "Zero to One = 9 on mentor: comes up in every founder conversation."
}') on conflict (kind) do update set rules = excluded.rules;
```

Why the merge: a template replaces the default rules for its kind entirely. Writing only `{"note": …}` would drop
the length and style rules. Start from `rules` and add to it.
