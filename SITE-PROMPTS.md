# Prompts from the site

The site shows the reader what's possible; **you do it**. Every control on Reading that builds or changes
something copies one short prompt for the reader to paste to you. Each prompt names the place by link (and the
book, author, shelf or section by name and id) and asks for **exactly one thing**.

When a message looks like one of these:
1. Do that one thing, nothing more. Read the page live first (the prompt carries no page text).
2. Follow the file the prompt names.
3. Check the preview (`select preview_url('<path>')`), then end with the link (`…#<section_id>` for a section).
4. "My template" means the reader's rules: every page of that kind. "This page" means one book or author.

What the reader still does directly on the site (you'll see it in `your_changes`): their notes and highlights,
reading status, rating, Kindle link and import, drill answers, news read/saved, deletes (with Undo), **Add to…** (putting
a book on a shelf or subshelf), and dragging a book on a shelf (it writes that score with `score_by = 'you'`).

## Later ([DAILY.md](DAILY.md))
| The prompt says | You do |
|---|---|
| "do the next 20 things in my Reading's Later list …" | a run: `owed_claim(20, 'live')`, do each, `owed_fail` what you can't, two-line summary ([DAILY.md](DAILY.md), *A run*) |
| a scheduled task: "every … do the next 20 things in my Reading's Later list, unattended …" | the same with `owed_claim(20, 'scheduled')`; nobody is watching |

## Pages ([PAGES.md](PAGES.md))
| The prompt says | You do |
|---|---|
| "add the section "X" (id `x`: …) to my book/author page template [as B (display `b`)], then write it for …" | add `{"id": "x"}` to the rules' sections (end of the list), with `"style": {"display": "b"}` when it names one, then write that section on that page in that shape |
| "help me design a new section for my book/author pages …" | the custom-section conversation (*The page conversation*): heading, purpose, look and accent, word cap → an `x_` section in the rules, then write it on that page |
| "write the section "X" (id `x`) for …" | write that one section on that page, in the template's look and shape |
| "write the section "X" on all my … pages that don't have it yet …" | `select slug from pages where kind = '<book\|author>' and has_page and 'x' = any(missing)`; one section per request; links at the end |
| "change the look of "X" (id `x`, now A) to B (display `b`)[, C accent][, D spacing] on my … template …" | set `"style": {"display": "b"}` on that rules entry (pages with their own override keep it: say how many), plus `accent` / `density` when named ("no accent" → drop `accent`) (tight → `compact`, roomy → `roomy`, normal → drop it); keep the rest; rewrite bodies into b's shape where `pages.style_warnings` says they don't fit, starting with the named page |
| "change the look of "X" (id `x`) to B (display `b`)… on "Book" only (a page override; my template stays) …" | `update book_sections` (or `author_sections`) `set style = '{"display": "b", …}'` on that one page and section; rewrite that body into the shape if it doesn't fit; the template is untouched |
| "reset the look of "X" (id `x`) on … to my … template (clear this page's override)" | `update book_sections` (or `author_sections`) `set style = null` for that page and section |
| "let's work on the section "X" of …" | read it, ask what to change, change only that section |
| "remove the section "X" (id `x`) from my … template …" | save the rules without it (it leaves every page in one change; they can undo) |

## Books ([BOOKS.md](BOOKS.md))
| The prompt says | You do |
|---|---|
| "add these N books to my shelf "S" (…): - … - …" | the add flow for each (lookup, note, put on that shelf, score it there), then each book page and the shelf |
| "add the book "Q" (find the right edition and author)" | find it; if unsure which one, ask with 2–3 options; then add it |
| "rank "B" on my shelf "S" …" (the book page's **Rank it there**, after Add to…) | the site already put it on the shelf; score it against that shelf's rule among the books already scored there, with `score_why` and `angle`; show where it lands, write on a yes ([RANKING.md](RANKING.md), *Ranking a shelf*) |
| "make drill cards for "B" …" | drill cards from its page and the reader's notes |
| "find how "B" connects to the other books and authors …" | [PLAYBOOK.md](PLAYBOOK.md): only connections with sources |

## Shelves ([SHELVES.md](SHELVES.md), [RANKING.md](RANKING.md))
| The prompt says | You do |
|---|---|
| "create a new shelf: ask me what it's for …" | ask, then name + blurb + ranking rule, suggest books from their library, add only the ones they pick |
| "help me write what my shelf "S" is for …" | name, blurb, ranking rule; then offer to re-rank |
| "suggest books for my shelf "S" …" | from their library first, then new ones; add only what they pick |
| "help me pick my next read (from my shelf "S") …" (an empty **Reading now**) | ask what they're in the mood for, suggest three with a one-line why each, set the chosen one to `status = 'reading'` |

The **Ranking strip** on a shelf's All tab. Each follows [RANKING.md](RANKING.md), *Ranking a shelf*: rule first,
pairwise against it, never silently over a `score_by = 'you'` score, full list, write on a yes, set `ranked_at`.

| The button | You do |
|---|---|
| **Rank this shelf** | every book on the shelf, the whole 1–100 range, `score_why` each, `angle` where empty |
| **Rank the new ones** | only books with no score, placed among the scored ones without moving those |
| **Re-space scores** | keep the order, spread the scores over 1–100 (no bunching, no ties unless they're truly equal); show before → after |
| **Set the rule** | ask what matters on this shelf, save one sentence as `ranking_rule`; then offer to rank |
| **Argue one book** (a book's shelf chip) | make the case up and down for that book against the rule; change its score only on a yes |
| **Compare two** (select two books on a shelf) | which of the two ranks higher on this shelf's rule, and why; swap or adjust only on a yes |
| **Write the story** | the shelf's (or the open subshelf's) story in `shelf_stories` ([PLAYBOOK.md](PLAYBOOK.md)) |
| **Suggest subshelves** | propose 3–6 subshelves, a one-line sentence each, and which books go where; create and place only what they agree to ([SHELVES.md](SHELVES.md)) |

## Authors ([AUTHORS.md](AUTHORS.md))
| The prompt says | You do |
|---|---|
| "find who A is connected to in my library …" | [PLAYBOOK.md](PLAYBOOK.md): teachers, students, critics, co-authors, with sources |
| "find recent news and articles about A …" | save the worth-reading ones with a one-line why |
| "make drill cards for A …" | face, big ideas, who said it |

## Playbook ([PLAYBOOK.md](PLAYBOOK.md))
| The prompt says | You do |
|---|---|
| "write the story of the "N" subshelf on my shelf "S" …" | the subshelf story (`shelf_stories`) |
| "let's work on the "N" story …" | ask what to change; change only that |
| "the "N" story … is missing someone …" | ask who, find how they connect, add them with sources |
| "update the "N" story … with this week's news …" | fold in the week's news about its authors |
| "explain how A and B connect …" / "is the connection between A and B right? …" | explain with sources / check each source and correct on a yes |
