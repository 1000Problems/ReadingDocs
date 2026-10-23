# Reading — the manual for your AI

Reading is a library you never type into. You tell your AI about books; it adds them, writes a short note on why each one matters to you, ranks them with your own system, and writes descriptions, drill cards and author pages when you ask. You read the result at **https://reading.hadmoney.com**.

This repo is Markdown only. It is for the AI. There is no code to install, no MCP server, no plugin.

## Two sources, read both
- **This repo** says what things *mean* and how to do the jobs.
- **`GET /describe`** says what your database *looks like right now*: tables, columns, checks, row counts, examples. Read it first in every new session. Its `repo_sha:` line is the commit of this repo it matches; if yours is older, pull.

Where they disagree, `/describe` wins for names and shapes, this repo wins for meaning. Report the disagreement to the user.

## Connect (once)
1. The user signs in at **https://db.byoaistudio.com** and copies the connect code from their space (works once, 10 minutes).
2. Trade it for a key:
   ```
   curl -s -X POST https://db.byoaistudio.com/connect -H 'content-type: application/json' \
     -d '{"code": "4F7K-2M", "name": "<user>'"'"'s Claude (Cowork)"}'
   ```
   Line 1 of the reply is the key (`byoai_…`). The rest is `/describe`.
3. Keep the key private: a file in the project (`byoai-key.txt`), never in chat, never in a commit.

**One key per space.** It sees every app the user has turned on, so you can answer questions across apps. The user can disconnect it on their space page at any time.

## Work
Base URL `https://db.byoaistudio.com`. Header `Authorization: Bearer <key>` on every call.

| Call | What |
|---|---|
| `GET /describe` | the live shape of the user's data |
| `POST /sql` | body = plain SQL. Reads and writes. One request = one transaction = one change |
| `POST /sql?dry_run=1` | runs, then rolls back. Use it before a bulk change |
| `GET /history?limit=20` | one line per change |
| `POST /undo {"change_id": "…"}` | undoes one change (refused if a later change touched the same rows) |

Errors are one plain-text line. Read it, fix the SQL, retry. Don't retry the same SQL unchanged.

You write bare table names (`books`, `notes`). You never see other users and never need a user id.

## Pages
| Page | When |
|---|---|
| [START.md](START.md) | the first session with a new user |
| [BOOKS.md](BOOKS.md) | "I want to read X", adding any book |
| [RANKING.md](RANKING.md) | setting up the user's ranking, scoring books |
| [SAFETY.md](SAFETY.md) | always. Read it once per session |

Every how-to page ends with **Exact SQL**: blocks that run as written on an empty space. They are tested against the schema at the pinned commit.
