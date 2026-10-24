# PLAYBOOK — how the books and authors on a shelf connect

The same as [ADD_BOOK.md](ADD_BOOK.md) step 9, for rewriting a shelf's story on its own.

The Playbook is the shelf told as a story: who taught whom, where they disagree, where *this reader* should start.
It shows on `reading.hadmoney.com/authors`; each pair has its own page at `/connections/<pair>`. Only books and
authors **on this reader's shelf** get links. Everything else goes in a book page's *Connections*.

| Table | One row per | Rules |
|---|---|---|
| `niche_roles` | author in a niche | `role`: one line on why they matter there |
| `pairs` | two authors | `lineage`: a_teaches_b, b_teaches_a, peers, a_critic_of_b, b_critic_of_a, collaborators; `sources` 1+ URL; `weight` 1–3; either order, stored a_id < b_id |
| `refs` | one author naming another | `source` = `web` (with `source_url`) or `briefing` |
| `book_links` | two books | `relation`: related, sequel, contrast, covers |
| `niche_stories` | niche (`''` = whole shelf) | `playbook` rules: 120–250 words, full names, *The lineage* required; optional *Timeline*, *The debates*, *Where to start*, *Sources*; front matter `title:` / `headline:` |

## When
- After a full add, if the niche now has 2+ authors or books. `todo` asks (`story:`) once a niche has 3 books.
- When the reader asks "how do these connect?" or a news item shows a new link (a review, a joint talk).
- Don't write a lineage from one author. Wait, or write only *Where to start* after asking.

## Exact SQL
See ADD_BOOK.md step 9. Read first:
```sql
select slug, title, author, niche from books where shelf = 'founder';
select a.slug, r.niche, r.role from niche_roles r join authors a on a.id = r.author_id where r.shelf = 'founder';
select * from pairs;
select * from niche_stories where shelf = 'founder';
```
