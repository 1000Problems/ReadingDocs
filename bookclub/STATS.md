# STATS — ready queries

Copy, adjust the year or name, and send as the `POST /sql` body. If one fails, check `/describe` and tell the member.

## Counting books

```sql
-- How many nonfiction books have we read this year?
select count(distinct book) filter (where kind = 'nonfiction') as nonfiction,
       count(distinct book) filter (where kind is null)         as untagged,
       count(distinct book)                                     as all_books
from meeting_cards
where status = 'met' and book is not null and year = extract(year from now());
-- distinct book: a two-month book counts once

-- Fiction vs nonfiction, every year
select period, fiction, nonfiction, untagged, pct_fiction from mix where period_kind = 'year' order by period;

-- Books read in a year
select month, title, author, host_name, club_score from meeting_cards
where year = 2024 and status = 'met' order by month;

-- Total books ever
select count(distinct book) from meeting_cards where status = 'met' and book is not null;
```
Always say how many books are **untagged** (`kind` null) when you answer a fiction/nonfiction question.

## Top and bottom books

```sql
-- Top 3 books of 2024
select rank_year, title, author, club_score, n_scores from book_scores
where year = 2024 order by rank_year limit 3;

-- All-time top 10 / bottom 5
select rank_all, title, author, year, club_score from book_scores order by rank_all limit 10;
select rank_all, title, author, year, club_score from book_scores order by club_score_raw asc limit 5;

-- Best book each year
select year, title, author, club_score from book_scores where rank_year = 1 order by year;

-- Club vs Goodreads: most over- and under-rated
select title, club_score, gr_x2, round(delta, 1) as delta from book_scores
where gr_x2 is not null order by delta desc limit 5;     -- asc for the ones we liked less

-- Most divisive book (widest spread of scores)
select title, round(stddev(score), 2) as spread, count(*) as n from meeting_votes
where attendance in ('voted','partial') group by meeting, title order by spread desc limit 5;
```
Ties share a rank (`rank_year` 2, 2, 4), so "top 3" can be more than 3 rows. Show all the tied ones.

## Authors

```sql
-- Author with the most books
select author, count(distinct book) as books, string_agg(distinct title, ' · ') as titles
from meeting_cards where status = 'met' and author is not null
group by author order by books desc limit 5;

-- Best-scoring authors (2+ books)
select author, count(*) as books, round(avg(club_score_raw), 1) as avg_score
from book_scores group by author having count(*) >= 2 order by avg_score desc limit 5;
```

## Members

```sql
-- Easiest and harshest scorers
select name, n, mean, easy_rank, harsh_rank from member_stats order by mean desc;

-- Most consistent (lowest spread)
select name, n, stdev from member_stats where n >= 10 order by stdev asc limit 5;

-- Attendance and streaks
select name, attended, missed, longest_streak from attendance order by attended desc;

-- One member's highest scores
select title, score, word from meeting_votes
where member = 'kent-l' and score is not null order by score desc limit 5;

-- Who hosts most, whose picks score best
select name, times_hosted, mean_pick_score, best_title from host_stats order by times_hosted desc;
```
Find a member's slug with `select slug, name from members where name ilike '%kent%'`.

## Words

```sql
-- Words used for a book
select name, word, score from words where title ilike '%piranesi%' order by score desc;

-- Most-used words ever
select lower(word) as word, count(*) from words group by 1 order by 2 desc limit 10;
```

## Fuzzy questions

Say how you're measuring, answer, and offer one other way to measure it.
- **"Our best year"**: highest average `club_score_raw` by `year` in `book_scores` (needs 3+ books). Or: the most
  top-10 books.
- **"Crowd-pleaser"**: high `club_score` with low spread. **"Hidden gem"**: high `delta` against Goodreads.
- **"Who agrees with whom"**: correlate two members' scores over the meetings they both scored (`meeting_votes`
  self-join). Say how many meetings it's based on.
