# First session

The user has just connected you. Their Reading is empty. Goal: five books on their shelf, each with a note that sounds like it was written for them, in under ten minutes.

1. `GET /describe`. Check the first line says `apps: reading` (or includes it). Read [SAFETY.md](SAFETY.md).
2. Look at what's there: `select * from todo` and `select count(*) from books`. If there are books already, this isn't a first session: say what you see and ask what they want.
3. Ask **two questions**, in one message:
   - "What do you do, and what are you trying to get better at this year?"
   - "Name a few books that stuck with you, or that you keep meaning to read."
4. Save what you learned as their display name only if they gave one. Everything else you keep in your own project notes, not in the database: it's what makes each book's note personal.
5. Add the books they named, following [BOOKS.md](BOOKS.md). Up to five now; offer the rest.
6. Tell them to open **https://reading.hadmoney.com** and sign in with the same account.
7. Offer next steps, one line each: set up their ranking ([RANKING.md](RANKING.md)), import a Goodreads export, full descriptions for the books they'll read first.

Keep the questions short. Don't interview. If they answer one word, work with one word.

## Exact SQL
What's there:
```sql
select count(*) as books from books;
select * from todo limit 10;
select slug, name, home from shelves order by position;
```
Their display name (the profile has one row):
```sql
insert into profiles (display_name) values ('Sam');
select display_name from profiles;
```
