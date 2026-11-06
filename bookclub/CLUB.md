# CLUB — the views you read

Every view is already filtered to the club and to what your member may see. Score columns are **null** (or rows are
missing) for meetings whose scoring isn't revealed yet. `GET /describe` is the live truth. This file explains what the
columns mean.

## How the club works (so the numbers make sense)

- One meeting per month. `slug = 'YYYY-MM'`. `status`: `planned` (future), `met`, `skipped`.
- At each meeting every member gives the book a **score 0–10** (up to 3 decimals) and **one word**.
- `attendance` per vote: `voted`, `partial` (read part, counts), `unfinished` (didn't finish, never counts), `missed`,
  `out`.
- **Club score** = average of the counting scores, shown to 1 decimal. Guests and partials count by club setting
  (`select partial_counts, guest_counts from club`).
- A book read over two months (Moby Dick) has its scores on the **last** meeting. The earlier one has `continues` set
  and is left out of rankings.
- `kind` is `fiction` / `nonfiction` / null (not tagged yet). `kind_by = 'ai'` means an AI suggested it and it isn't confirmed yet.
- `gr_x2` = Goodreads average × 2, so it sits on the club's 0–10 scale. `delta` = club score − `gr_x2` (positive: the club liked it more than Goodreads did).
- Ranks use `rank()`, so ties share a place.

## The main views

| View | One row per | Columns | Use for |
|---|---|---|---|
| `meeting_cards` | meeting | `slug, number, month, year, status, scoring, book, title, author, cover_url, kind, host, host_name, venue_label, club_score, n_scores, gr_x2, rank_all, rank_year` | "what did we read in …", counts by year/kind/author/host |
| `book_scores` | revealed meeting with a book | `meeting, book, title, author, month, year, host, club_score, club_score_raw, n_scores, gr_x2, delta, rank_all, rank_year, ranked_count` | top/bottom books, by year, vs Goodreads |
| `book_cards` | book | `slug, title, author, cover_url, kind, kind_by, goodreads_avg, gr_x2, meetings text[], times_on_slate` | a book's facts, books pitched but never picked |
| `member_stats` | member (not guests) | `member, name, status, n, mean, stdev, misses, unfinished, easy_rank, harsh_rank` | easiest/harshest scorer, most consistent |
| `host_stats` | member | `member, name, times_hosted, mean_pick_score, best_meeting, best_title` | who hosts most, whose picks score best |
| `attendance` | member | `member, name, attended, missed, out, unfinished, longest_streak` | attendance questions |
| `mix` | period | `period, period_kind ('year'/'era'), fiction, nonfiction, untagged, pct_fiction` | fiction/nonfiction balance by year or 5-year era |
| `words` | vote with a word | `meeting, book, title, member, name, word, score` | "what words did we use for …" |
| `meeting_votes` | vote | `meeting, member, name, status, attendance, score, word, source, corrected, entered_by` | one member's score on one book |
| `members` | member | `slug, name, status ('active'/'alumni'/'guest'), joined_on, left_on` | roster |
| `next_meeting` | 0–1 rows | `slug, month, meets_at, book, title, author, host_name, venue_label, host_line, bring, days_until, rsvp_yes, rsvp_maybe, rsvp_guests` | "when's the next meeting?" |
| `last_meeting` | 0–1 rows | `slug, month, title, author, host_name, club_score, rank_all, ranked_count, top_words` | "how did last month go?" |
| `ballot_results` | candidate on a closed slate | `slate, book, title, points, firsts, seconds, thirds, ballots, eligible, place, tied` | "how close was the vote?" (totals only) |
| `slate_cards` | candidate | `slate, book, title, author, pitch` | what's on the current slate |
| `club` | 1 row | name, founded_on, settings | club facts |
| `me` | 1 row | `slug, name, role, status` | who you're acting for |

Also readable: `rsvp_board` (who's coming), `meeting_directions` (members only; don't repeat addresses outside the
member's own chat), `club_posts` and `thread_posts` (the chat, if the club shares it with AIs), `writeups`, `later`.

## Joining

- Meeting ↔ book: `meeting_cards.book = book_cards.slug`. Scores: `book_scores.meeting = meeting_cards.slug`.
- Member slugs (`kent-l`) join `members.slug`, `member_stats.member` and `meeting_votes.member`.
- Authors are plain text in `author`. Several authors are joined in one string, so group by that string and say so.
  `book_authors` gives author slugs if you need them.
