# 🍿 Movie Voter

Everyone can suggest a movie and vote for the ones they want to watch. No auth —
people just leave their name (honor system, votes aren't enforced or limited).

Same stack as [leangap-mentors](https://github.com/eddyvarelae/leangap-mentors):
a single static `index.html` (vanilla JS + CSS, no build step), data in Airtable,
deployed to Hostinger over FTP by GitHub Actions on every push to `main`.

## One-time setup

### 1. Airtable base

Create a base with **two tables**. The code references them by table id
(`AIRTABLE.moviesTable` / `AIRTABLE.votesTable` in `index.html`), so update those
ids if you recreate the base. Field names must match exactly:

**movies**
| Field | Type |
|---|---|
| `title` | Single line text |
| `submittedBy` | Single line text |
| `note` | Single line text |

**votes**
| Field | Type |
|---|---|
| `voter` | Single line text |
| `movieId` | Single line text (the Movies record id — links votes to a movie) |
| `movieTitle` | Single line text (readability only, so the Votes table is human-scannable) |

Delete any default fields you don't need (extra fields are harmless). Vote counts
are computed in the page by counting Votes rows per `movieId`.

Then create a **personal access token** at airtable.com/create/tokens with scopes
`data.records:read` + `data.records:write`, granted access to **only this base**.
The token ships in the page source (same accepted tradeoff as leangap-mentors),
so keep it scoped to nothing else.

### 2. GitHub secrets

Add these repository secrets (Settings → Secrets and variables → Actions):

| Secret | Value |
|---|---|
| `AIRTABLE_TOKEN` | the scoped personal access token |
| `AIRTABLE_BASE_ID` | the base id (`app…`, from airtable.com/api) |
| `FTP_SERVER` | Hostinger FTP host |
| `FTP_USERNAME` | Hostinger FTP user |
| `FTP_PASSWORD` | Hostinger FTP password |
| `FTP_SERVER_DIR` | remote dir, e.g. `/public_html/` (use a subfolder like `/public_html/movies/` to serve it next to another site) |

### 3. Deploy

Push to `main` (or run the workflow manually). The workflow `sed`-injects the
Airtable credentials into `index.html` and FTP-uploads everything except git/CI
files.

## Local development

Open `index.html` in a browser. With the `PASTE_*` placeholders still in place it
runs in demo mode with sample data and a banner; nothing is written anywhere.
