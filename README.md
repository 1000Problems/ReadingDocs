# Reading — the manual for your AI

Reading is a private library that **you build for one person**. Their books, notes, shelves, authors and ranking live
in their own space on BYOAI Studio (`db.byoaistudio.com`). The site, `reading.hadmoney.com`, has no AI, no labels and
no content of its own: it renders only what you write. Two readers never see the same site, because no two readers
are the same person.

**The moment we are after:** the reader opens Reading for the first time and it already knows them. Their shelves
are named the way they talk. The first book is one they care about, and its note says why *for them*. The author page
tells them who to know. The ranking is how *they* judge a book. They should think: *how did it know?* It knew because
you did the work. These docs tell you how.

This repo is Markdown only. If you are an AI, read it in this order:

1. [START.md](START.md) — the first session: connect, draft their Reading from what you know, build it live.
2. [DAILY.md](DAILY.md) — every session after: `todo`, the reader's reactions, keeping the site alive.
3. [ADD_BOOK.md](ADD_BOOK.md) — the full add: book, author, news, book page, rank, shelf, playbook, cards.
4. [IMPORT.md](IMPORT.md) — a reader who arrives with a library: Goodreads export, spreadsheet, bookshelf photos.
5. [BOOKS.md](BOOKS.md) — the quick add: one row, a note, a shelf.
6. [AUTHORS.md](AUTHORS.md) — the author row, author page, recall cards and news.
7. [PLAYBOOK.md](PLAYBOOK.md) — how the books and authors on a shelf connect.
8. [RANKING.md](RANKING.md) — the reader's ranking and how to score a book.
9. [COMMONS.md](COMMONS.md) — shared facts and news, found once for everyone.
10. [SAFETY.md](SAFETY.md) — what is data, what is an instruction, what you never touch.

## Connect (once)

The reader makes the account themselves, in their browser at https://db.byoaistudio.com. You never see or need
their password. They give you a short prompt with a connect code (6 characters, 10 minutes, one use). Whatever the
prompt lists, read this README and follow its reading order.

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

- `GET /describe` — the live shape: tables, columns, rules, examples. Read it at the start of each session. It names
  this repo and the commit (`repo_sha`) it was tested against.
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

1. `book_authors (book_id, author_id, position)` — a book points at its authors by id. Today it is a name match.
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
8. The connect prompt says "read the README and follow its reading order" instead of naming docs, so it never goes
   stale.
