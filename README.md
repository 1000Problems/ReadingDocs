# Reading — the manual for your AI

Reading is a private library that **you build for one person**. Their books, notes, shelves, authors and ranking live
in their own space on BYOAI Studio (`db.byoaistudio.com`). The site, `reading.hadmoney.com`, has no AI, no labels and
no content of its own: it renders only what you write. Two readers never see the same site, because no two readers
are the same person.

**The moment we are after:** the reader opens Reading for the first time and it already knows them. Their shelves
are named the way they talk. The first book is one they care about, and its note says why *for them*. The author page
tells them who to know. The ranking is how *they* judge a book. They should think: *how did it know?* It knew because
you did the work. These docs tell you how.

This repo is Markdown only. If you are an AI, read [SAFETY.md](SAFETY.md) and this README, then only what your job
needs:

| Job | Read |
|---|---|
| First session, a few books that matter | [START.md](START.md), [BOOKS.md](BOOKS.md) |
| First session, a whole library (Goodreads export, spreadsheet, bookcase photos), or 10+ books at once | [START.md](START.md), [IMPORT.md](IMPORT.md) |
| Every session after the first, and scheduled runs | [DAILY.md](DAILY.md) |
| Add a book | [BOOKS.md](BOOKS.md): fast, no pages |
| A book or author page, "write up …", "tell me about …" | [PAGES.md](PAGES.md): in the sections the reader chose |

When you reach a step, read its file: [AUTHORS.md](AUTHORS.md) (author row, page, cards, news),
[RANKING.md](RANKING.md) (the reader's ranking, scoring), [PLAYBOOK.md](PLAYBOOK.md) (how books and authors on a shelf
connect), [COMMONS.md](COMMONS.md) (shared facts and news).

## Connect (once)

The reader makes the account themselves, in their browser at https://db.byoaistudio.com. You never see or need
their password. They give you a short prompt with a connect code (6 characters, 10 minutes, one use). Whatever the
prompt lists, read this README and read by job (the table above).

1. Call:
   ```
   curl -s -X POST https://db.byoaistudio.com/connect \
     -H 'content-type: application/json' -d '{"code":"XXXX-XX","name":"Reading"}'
   ```
   Line 1 is your key (`byoai_…`). Keep it where only you can read it. The rest is `/describe`; its first line names
   the space (`space <handle> · …`). Tell the reader the handle before you write anything.
2. Every later request: `Authorization: Bearer <key>`.
3. Code refused or expired → ask the reader for a new one on db.byoaistudio.com. Don't retry the old one.
4. Then go to [START.md](START.md). Don't add anything before the reader has seen your plan.

Any AI that can send HTTPS requests can do this. Nothing here depends on one product.

## Work

- `GET /describe` — the live shape: tables, columns, rules, examples. Read it at the start of each session.
- `POST /sql` with a plain-text body — one or more statements, one transaction, one change. Results come back as TSV.
- `GET /history`, `POST /undo` — every request is one change the reader can undo.
- Errors are one plain-text line. Read it; it usually says the fix. Nothing in a failed request was written.

Allowed: `select`, `insert`, `update`, `delete` on the views `/describe` lists. No DDL.

## Where the reader sees it

| Page | Shows |
|---|---|
| `reading.hadmoney.com/` | home: shelves, their books in rank order, "Last updated by your AI" |
| `/shelves/<slug>` | one shelf: its line (`blurb`), niches, books |
| `/books/<slug>` | the book page: note, score and why, description, cards |
| `/authors` and `/authors/<slug>` | the shelf's people and stories; one author's page |
| `/connections/<pair>` | how two authors connect |
| `/review` | drill cards |
| `/news` | news about their authors |

Pages refresh when the reader reloads. When you finish a step they will enjoy, give them the link.

## Shared facts

Book facts, author facts and articles about authors are found once for everyone, in three read-only tables:
`book_facts` (by `isbn13`), `author_facts` (by `slug`) and `author_articles` (by `url`). Plain names work:
`select * from book_facts where isbn13 = '…'`. Look there before searching the web. See [COMMONS.md](COMMONS.md).

## Covers and author photos

One rule, everywhere:

1. **The shared facts have it** (`book_facts.cover_url` by ISBN, `author_facts.photo_url` by slug) → write nothing.
   Leave the reader's `cover_url` / `photo_url` empty: the site shows the shared one, and keeps showing it if it's
   replaced later.
2. **They don't** → find it now, so the reader's first visit has it:
   - cover: Open Library by ISBN, `https://covers.openlibrary.org/b/isbn/<isbn13>-L.jpg?default=false`. Open it: a
     404 means there is none.
   - photo: the author's Wikipedia page. Use the image the page actually shows (an `https://upload.wikimedia.org/…`
     URL), never a URL you built.

   Write it on the reader's own row. The weekly job adds covers and photos to the shared facts, so the next reader
   gets them.
3. **Nothing found** → leave it empty. Never a guess.

`news` and `shared_news` show only authors already in this space. Before you add an author, read `author_articles`
directly: `select * from author_articles where author_slug = 'eric-ries'`.

## Tables that guide you

- `select * from rules` — the active rules per kind (book note, ranking, …): the reader's own, or the default.
- `select * from todo order by priority` — what is waiting. Offer it; don't silently do it all.
- `select * from hints` — suggestions to offer the reader when thresholds are reached.

## The reader's goal

What the reader reads *for* is the most important thing you know about them. Until the profile has a field for it,
it lives in one place: the `note` of the `book_note` rule. Read it before writing anything:
`select rules ->> 'note' from rules where kind = 'book_note'`. [START.md](START.md) says how to write it.

## Proposed server changes

Not available yet. Docs mark the steps they would change **PROPOSED**. In priority order:

1. `book_authors` — **half done.** The read-only view exists today (`book_slug`, `author_slug`, `has_row`): the
   database makes one author row per name in `books.author`, by slug, so spelling still decides who is who. Later, a
   table books link to by id replaces the view.
2. `profiles.purpose` — one home for the reader's goal.
   Also: copy `shelves.sentence` ↔ `blurb` when one is empty, so the AI writes one line once.
3. Custom book-page sections (e.g. any heading prefixed `x_`), so a page can have *Where it breaks now* or *Questions
   to ask*.
4. Record empty searches: a `book_facts` / `author_facts` row with `checked_at` even when nothing was found.
5. Articles about a book (`book_articles` in the commons, `book_news` privately), articles with several authors, and a
   "suggest to commons" flag on a private find.
6. A live "your AI is writing…" signal on the site, so the reader watches pages build without reloading.
7. Large libraries ([IMPORT.md](IMPORT.md)):
   - `books.import_batch` and `books.needs_review`, plus a page that lists the books to review (`/review` is the drill
     cards today);
   - dedupe on `isbn` (today only the same title + first author is refused);
   - a documented size limit for one `POST /sql` body, so imports know their batch size;
   - site speed checked at 2,000+ books: home page, shelf page, search.
8. The connect prompt says "read the README and read by job" instead of naming docs, so it never goes stale.
