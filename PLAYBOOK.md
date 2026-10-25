# PLAYBOOK — how the books and authors on a shelf connect

The same as [PAGES.md](PAGES.md) step 9, for rewriting a shelf's story on its own.

The Playbook is the shelf told as a story: who taught whom, where they disagree, where *this reader* should start.
It shows on `reading.hadmoney.com/authors`; each pair has its own page at `/connections/<pair>`. Only books and
authors **on this reader's shelf** get links. Everything else goes in a book page's *Connections*.

| Table | One row per | Rules |
|---|---|---|
| `shelf_roles` | author on a shelf or subshelf (`''` = whole shelf) | `role`: one line on why they matter there |
| `pairs` | two authors | `lineage`: a_teaches_b, b_teaches_a, peers, a_critic_of_b, b_critic_of_a, collaborators; `sources` 1+ URL; `weight` 1–3; either order, stored a_id < b_id |
| `refs` | one author naming another | `source` = `web` (with `source_url`) or `briefing` |
| `book_links` | two books | `relation`: related, sequel, contrast, covers |
| `shelf_stories` | subshelf (`''` = whole shelf) | `playbook` rules: 120–250 words, full names, *The lineage* required; optional *Timeline*, *The debates*, *Where to start*, *Sources*; front matter `title:` / `headline:` |

## When
- **Links** (`shelf_roles`, `pairs`, `refs`, `book_links`): once the subshelf has 2+ authors.
- **The story** (`shelf_stories`): on request, or on a run when Later has it. Later gets it once a subshelf has 3 books;
  writing the story closes it.
- When the reader asks "how do these connect?" or a news item shows a new link (a review, a joint talk).
- Don't write a lineage from one author.

## Exact SQL
See PAGES.md step 9. Read first:
```sql
select b.slug, b.title, b.author, sb.subshelf from shelf_books sb join books b on b.id = sb.book_id
 where sb.shelf = 'founder' order by sb.subshelf, b.slug;
select a.slug, r.subshelf, r.role from shelf_roles r join authors a on a.id = r.author_id where r.shelf = 'founder';
select * from pairs;
select * from shelf_stories where shelf = 'founder';
```
