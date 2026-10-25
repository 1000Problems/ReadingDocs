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
reading status, rating, Kindle link and import, drill answers, news read/saved, and deletes (with Undo).

## Later ([DAILY.md](DAILY.md))
| The prompt says | You do |
|---|---|
| "do the next 20 things in my Reading's Later list …" | a run: `owed_claim(20, 'live')`, do each, `owed_fail` what you can't, two-line summary ([DAILY.md](DAILY.md), *A run*) |
| a scheduled task: "every … do the next 20 things in my Reading's Later list, unattended …" | the same with `owed_claim(20, 'scheduled')`; nobody is watching |

## Pages ([PAGES.md](PAGES.md))
| The prompt says | You do |
|---|---|
| "add the section "X" (id `x`: …) to my book/author page template, then write it for …" | add `{"id": "x"}` to the rules' sections (end of the list), then write that section on that page |
| "help me design a new section for my book/author pages …" | the custom-section conversation (*The page conversation*): heading, purpose, word cap → an `x_` section in the rules, then write it on that page |
| "write the section "X" (id `x`) for …" | write that one section on that page, in the template's look and shape |
| "write the section "X" on all my … pages that don't have it yet …" | `select slug from pages where kind = '<book\|author>' and has_page and 'x' = any(missing)`; one section per request; links at the end |
| "change the look of "X" (id `x`, now A) to B on my … template …" | set `"style": {"display": "B"}` on that rules entry (keep accent/density); rewrite bodies into B's shape where `pages.style_warnings` says they don't fit, starting with the named page |
| "let's work on the section "X" of …" | read it, ask what to change, change only that section |
| "remove the section "X" (id `x`) from my … template …" | save the rules without it (it leaves every page in one change; they can undo) |

## Books ([BOOKS.md](BOOKS.md))
| The prompt says | You do |
|---|---|
| "add these N books to my shelf "S" (…): - … - …" | the add flow for each (lookup, note, rank on that shelf), then each book page and the shelf |
| "add the book "Q" (find the right edition and author)" | find it; if unsure which one, ask with 2–3 options; then add it |
| "move "B" … to my shelf "S" … and rank it there" | set its shelf (and niche if the shelf has them), rank it against that shelf's rule ([RANKING.md](RANKING.md)) |
| "why is "B" ranked where it is on my shelf "S"? …" | make the case up and down against the rule; change the rank only on a yes |
| "make drill cards for "B" …" | drill cards from its page and the reader's notes |
| "find how "B" connects to the other books and authors …" | [PLAYBOOK.md](PLAYBOOK.md): only connections with sources |

## Shelves ([BOOKS.md](BOOKS.md), [RANKING.md](RANKING.md))
| The prompt says | You do |
|---|---|
| "create a new shelf: ask me what it's for …" | ask, then name + blurb + ranking rule, suggest books from their library, add only the ones they pick |
| "rank my shelf "S" against its rule. If it has no rule yet, write one with me first …" | rule first if missing; show the new order with one-line whys; apply on a yes |
| "help me write what my shelf "S" is for …" | name, blurb, ranking rule; then offer to re-rank |
| "suggest books for my shelf "S" …" | from their library first, then new ones; add only what they pick |
| "help me split my shelf "S" into niches …" | propose 2–5 niches and where each book goes; apply only what they agree to |

## Authors ([AUTHORS.md](AUTHORS.md))
| The prompt says | You do |
|---|---|
| "find who A is connected to in my library …" | [PLAYBOOK.md](PLAYBOOK.md): teachers, students, critics, co-authors, with sources |
| "find recent news and articles about A …" | save the worth-reading ones with a one-line why |
| "make drill cards for A …" | face, big ideas, who said it |

## Playbook ([PLAYBOOK.md](PLAYBOOK.md))
| The prompt says | You do |
|---|---|
| "write the story of the "N" niche on my shelf "S" …" | the niche story |
| "let's work on the "N" story …" | ask what to change; change only that |
| "the "N" story … is missing someone …" | ask who, find how they connect, add them with sources |
| "update the "N" story … with this week's news …" | fold in the week's news about its authors |
| "explain how A and B connect …" / "is the connection between A and B right? …" | explain with sources / check each source and correct on a yes |
