# Adding a book

The one procedure for adding a book, from "I want to read X" or from a list.

## Steps
1. **Already there?** `select * from search('<title>')`. Typos and partial titles match. If it's there, say so and stop.
2. **Look it up.** Open Library gives author, ISBN and cover:
   `https://openlibrary.org/search.json?limit=3&fields=title,author_name,isbn,cover_i&q=<title>`
   Cover URL: `https://covers.openlibrary.org/b/id/<cover_i>-M.jpg`. Ask the user only if two different books match.
3. **Author name.** Before you write `author`, check how this author is already written: `select slug, name from authors where name ilike '%<surname>%'` and `select distinct author from books where author ilike '%<surname>%'`. Use the existing spelling exactly. "Kahneman" and "Daniel Kahneman" must not both exist.
4. **The note** (`books.note`). One paragraph, **40–90 words**, plain prose, no headings: why this book, for *this* reader, from what they told you and what you know of them. No invented facts. The database checks the length and refuses outside it. This is cheap on purpose, so adding fifty books stays cheap.
5. **Status.** `want` unless they said they're reading (`reading`) or done (`finished`, then offer a rating 1–5). The database stamps `started_at`/`finished_at`.
6. **Shelf and niche.** Leave `shelf` empty and the book goes to the home shelf. If they have more shelves (`select slug, name, niches from shelves`), pick the one that fits; the niche must be one of that shelf's `niches[].id`. Unsure: home shelf, and say so.
7. **Insert.** One `insert … returning slug`. The slug comes from the title. The same title + first author twice is refused with the existing slug; that's your duplicate check too.
8. **Rank** if the user has set up ranking: see [RANKING.md](RANKING.md). If not, skip it. `todo` will remind you.
9. **Reply** with one line per book: title, author, and the note. Then: "Full description and drill cards when you want them."

Full descriptions, drill cards and author pages are separate jobs, done on request or when `todo` lists them. `select kind, what from catalog` shows what each kind of writing can contain.

## Several books
One `/sql` request can carry many inserts; it's one change, so one undo removes them all. Batches of up to 25. If a batch is refused, the whole batch rolled back: redo it one book at a time and report which ones the database refused and why.

## Optional, any time
- Who recommended it: `update books set why_read = 'Recommended by …' where slug = '…'`.
- Kindle edition: `kindle_asin` = the 10 characters after `/dp/` in the Amazon link.
- Tags: 1–3 lowercase words, `tags = array['startups','strategy']`.

## Exact SQL
Look first:
```sql
select type, book_slug, title, author from search('zero to one') limit 5;
select slug, name from authors where name ilike '%thiel%';
select distinct author from books where author ilike '%thiel%';
```
Add one book (one request, one change):
```sql
insert into books (title, author, isbn, cover_url, status, tags, note)
values ('Zero to One', 'Peter Thiel', '9780804139298', 'https://covers.openlibrary.org/b/id/8227654-M.jpg', 'want',
        array['startups', 'strategy'],
        'You said you want to stop building features nobody asked for. Thiel argues the opposite of copying: find the one thing few people agree with you on, and build a small monopoly around it. Read it before your next roadmap review; the chapter on secrets is the one to argue with.')
returning slug;
```
Add several at once (batch of up to 25):
```sql
insert into books (title, author, status, note) values
  ('The Lean Startup', 'Eric Ries', 'finished',
   'You already run experiments without calling them that. Ries gives the vocabulary: build, measure, learn, and the minimum viable product. Useful mostly as a shared language with your team, so a disagreement about a feature becomes a disagreement about which test to run first. Skim the second half.'),
  ('Thinking, Fast and Slow', 'Daniel Kahneman', 'want',
   'You make a lot of quick calls on hiring and pricing. Kahneman explains the two speeds of thinking and the predictable ways the fast one fools you. Long and dense; read the parts on anchoring and the planning fallacy first, since those are the mistakes you said cost you most last year.')
returning slug;
```
Small edits after:
```sql
update books set status = 'reading' where slug = 'zero-to-one';
update books set why_read = 'Recommended by a founder friend', kindle_asin = 'B00J6YBOFQ' where slug = 'zero-to-one';
```
