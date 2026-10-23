# Movie Voter — Build Log

A one-page movie night voting site: anyone suggests a movie, votes with just
their name, no accounts. Single static `index.html` (vanilla JS, no build
step), data in Airtable, auto-deployed to Hostinger over FTP by GitHub
Actions. Live at movievoter.eddy.mx. Built with Claude Code.

---

## 1. The ask

> Make a movie voting site for movie night. Everyone can suggest a movie and
> vote for the ones they want to watch. No auth — people just leave their
> name, honor system. Same stack as leangap-mentors: one static index.html,
> Airtable as the backend, deployed to Hostinger over FTP on every push.

Claude built the whole thing in one pass:

- **Two Airtable tables** — `movies` (title, submittedBy, note) and `votes`
  (voter, movieId, movieTitle). Vote counts are computed in the page by
  counting vote rows per movie.
- **The page** — name field on top, "Suggest a movie" panel, and the ballot:
  cards sorted by votes with a big count, voter name pills, and a ▲ Vote
  button.
- **Demo mode** — with the credential placeholders still in place the page
  runs on sample data and shows a banner, so it's testable locally without
  touching Airtable.
- **Deploy pipeline** — GitHub Actions workflow that `sed`-injects the
  Airtable token and base id from repo secrets into `index.html`, then
  FTP-uploads to Hostinger. The token ships in the page source by design —
  it's scoped to only this base.

Then `git init`, `gh repo create`, six secrets, push — live.

---

## 2. Airtable 404s

First load: `Airtable 404`. The code said `Movies` and `Votes`; the base
said `movies` and `votes`. Claude matched the lowercase names, then went one
better — switched the code to reference the tables by **table id** instead
of name, so renaming a table in Airtable can never break the site again.

---

## 3. Unvote + edit your own movie

> If I voted from my phone I want to be able to remove my vote from the same
> phone. And let me fix typos in my own submission.

No accounts, so the browser is the identity. Claude used localStorage:

- Each vote stores its Airtable record id locally — the same browser can
  click ✓ Voted again to **unvote** (deletes that exact record).
- The browser that submitted a movie gets an ✎ Edit button: the **note** is
  editable anytime, the **title** only within 60 seconds of submitting —
  after that it's locked so the ballot can't be rug-pulled mid-vote.
- Nothing is enforced server-side. It's a courtesy, like the whole site.

---

## 4. Auto-vote your own suggestion

Suggesting a movie now drops your own vote on it in the same breath —
nobody submits a movie they don't want to watch.

---

## 5. Open the card → poster + intro

> When I open the card the movie poster should appear and Marianna's intro,
> you can use 2A7D4604-ED8B-42DB-97D2-8B9269B55837 from my downloads.

That opaque filename turned out to be the Howl's Moving Castle poster
(360×540 JPG). Claude:

- Copied the poster and Marianna's photo into `assets/` so they deploy with
  the page.
- Added an `EXTRAS` map keyed by lowercased title — any movie with an entry
  gets a click-to-open **modal**: poster on the left, intro on the right
  ("Suggested by Marianna", photo, a short pitch). One more entry = one more
  movie with a card.
- Cards with extras show a "▸ Open the card" hint; clicking Vote or Edit
  still works normally and never triggers the modal. Close with ✕, Esc, or
  clicking outside.

---

## 6. "Don't put the mentors info on the card"

> Don't put the mentors info on the card, like the dnd stuff.

First draft of the intro reused Marianna's mentor bio from leangap-mentors —
role, email-marketing credentials, DnD, the cat. Cut. The intro is now just
a two-line pitch for the movie itself.

---

## 7. Winner badge

> When the card is opened show the label winner with x votes.

A gold **🏆 Winner · 5 votes** pill under the title — computed from the
live standings, not hardcoded. It only says Winner while that movie
actually leads the ballot; a non-leading movie's card gets a neutral vote
count pill instead. Stays honest if the votes flip before movie night.

---

## 8. "Shouldn't be a circle"

> Shouldn't be a circle, the framing doesn't work unless the whole img is
> displayed.

The intro photo started as a classic 72px avatar circle. Problem: the
source image is 1080×600 landscape — the circle was throwing away most of
the frame. Now it renders **uncropped at natural proportions**, 160px wide
with rounded corners. Sometimes the standard component is the wrong one.

---

## 9. The flaky deploys

Two of the last three deploys died with `Error: Timeout (control socket)` —
Hostinger's FTP just not picking up. Claude caught both by watching the
workflow run after each push, re-ran them with `gh run rerun --failed`, and
verified the fix landed by `curl`-ing the live page and grepping for the
new code. Zero-downtime debugging from the terminal.

---

## Final stack

```
index.html        the whole app — markup, CSS, and vanilla JS in one file
assets/           howls-poster.jpg, marianna.png
.github/          deploy.yml — secret injection + FTP upload on push
```

**No build step. No framework. No server.** Airtable is the database, the
API, and the admin panel; GitHub Actions is the deploy pipeline; the
browser's localStorage is the auth system.

Live at **movievoter.eddy.mx** — Howl's Moving Castle is winning.
