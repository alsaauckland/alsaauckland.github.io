# ALSA Auckland Website — Handover Guide

Everything you need to maintain and extend the ALSA Auckland website. Read this
once before making changes.

> **Keep this document current.** It is the source of truth for how the site
> works. Whenever you (human or AI) make a change that alters the site's
> structure, forms/backends, reusable systems, or conventions, update the
> relevant section here in the same commit. Don't log one-off content edits.
> AI assistants: this rule is also in `CLAUDE.md`.

---

## 1. What this is

- **Live site:** https://alsaauckland.org
- **Repo:** `alsaauckland.github.io` (GitHub Pages)
- **Custom domain:** set in `CNAME` (`alsaauckland.org`)
- **Analytics:** Google Analytics, tracking ID `G-1M6M9RLGB6` (in every page `<head>`)
- **~50 HTML pages**, static, no framework, no build step.

The site is a **static site**: plain HTML files, one `index.html` per URL folder.
There is **no build system, no npm, no bundler**. What you see in the file is what
ships. CSS lives in an inline `<style>` block inside each page's `<head>` (there is
no shared stylesheet — each page carries its own copy of the styles).

---

## 2. How deployment works

GitHub Pages builds and publishes automatically on every push to the `main` branch.

```
edit files  ->  git add/commit  ->  git push origin main  ->  live in ~1-2 min
```

- No manual deploy step. The push *is* the deploy.
- After pushing, the CDN can serve a stale copy for a minute; hard-refresh
  (Cmd+Shift+R) or wait a moment.
- To check build status: repo → **Settings → Pages**, or the **Actions** tab.

**Commit message convention:** end commits authored with AI assistance with a
`Co-Authored-By:` trailer (see git history for the pattern).

---

## 3. Repository layout

```
/                         Home page (index.html)
/404.html                 Custom not-found page
/CNAME                    Custom domain
/robots.txt, /sitemap.xml SEO
/images/                  All images, grouped by area (see below)

Pages (each is a folder with an index.html):
/about        /contact      /join         /partners     /programmes
/events       /gallery      /newsletters  /spotlight    /competition
/careers      /card         /privacy      /conduct
/hub          Linktree-style page (podcast, socials, newsletter, join);
              last item in the top nav, right of Contact

/mentoring            Mentoring hub
/mentoring/why        Why the programme matters
/mentoring/apply      Mentee/mentor application form

/cultural-competency  Event registration page (Chinese Cultural Competency)
/technology-networking  Event registration page (draft)

/team                 Team hub
/team/<person>        Individual profile pages (jayden-lin, cayla-huang, ...)
/team/apply           Recruitment hub
/team/roles/<role>    One page per role (president, treasurer, ...)
/team/cabinet, /team/standards, /team/why-join

/newsletters/issue-1  A published newsletter issue (dated archive — don't edit
                      for current info; it's a snapshot)
```

**Images** live in `/images/` grouped by area: `home/`, `about/`, `team/`,
`portraits/`, `programmes/`, `sponsors/`, `newsletters/`, `contact/`, `gallery/`.

> **Important: paths are root-absolute.** Always reference assets as
> `/images/...` (leading slash), never `images/...` or `../images/...`. This is
> why opening a file directly in the browser looks broken — see §9 on local preview.

---

## 4. Design system

Match these when editing or building — every page uses them so the site stays
consistent.

**Colours**
| Token | Hex | Use |
|-------|-----|-----|
| Brand maroon | `#5b0f0f` | Primary buttons, headings accents, links |
| Dark maroon | `#2b0e0e` | Dark section backgrounds, deep contrast |
| Near-black | `#1a1a1a` | Footers, dark bands, body headings |
| Gold | `#C9A84C` | Eyebrows, accents, italic emphasis |
| Cream | `#fdfaf4` | Page background |
| Warm sand | `#f5f0e8` | Alternate section background, callout boxes |

**Fonts** (loaded from Google Fonts in every `<head>`)
- **Playfair Display** (serif) — headings, `<h1>`/`<h2>`, event titles
- **Lora** (serif) — body text, everything else

**Recurring patterns**
- **Eyebrow**: tiny uppercase gold label above a heading (letter-spacing ~4px).
- **Italic emphasis**: headings often use `<em>` styled italic + gold for a key word.
- **Callout box**: `#f5f0e8` background with a 3px `#C9A84C` left border.
- **Nav + footer**: nearly identical across pages — copy them from an existing
  page when creating a new one.
- **Mobile nav**: a hamburger (`#hamburger`) toggling `.nav-links.open`; the same
  small script sits at the bottom of every page.

---

## 5. Editing an existing page

1. Open the page's `index.html` (e.g. `/about/index.html`).
2. Edit the HTML/CSS directly. Styles are in the `<style>` block up top.
3. Preview locally (see §9), then `git add` / `commit` / `push`.

**House style for copy** (the site's established voice):
- **NZ / British spelling** (organise, programme, colour, centre).
- **No em dashes** — use commas, full stops, or restructure.
- **No AI filler** ("delve", "moreover", "in today's world", etc.). Be specific
  and plain.
- **Bullet lists**: bold lead sentence, then plain explanatory text.
- **Confident, not defensive.** Say what a thing *is*, not what it "aims to" be.

---

## 6. Creating a new page

The fastest, safest way is to **clone an existing page** that's closest to what you
want, then change the content:

1. Make a new folder for the URL, e.g. `/my-new-page/`.
2. Copy an existing `index.html` into it (pick one with a matching layout).
3. Update: `<title>`, `<meta name="description">`, the `og:` tags (incl.
   `og:url`), the `<h1>`, and the body content.
4. Keep the nav and footer identical to other pages.
5. Add the URL to `sitemap.xml` (optional but good for SEO).
6. Link to it from wherever it should be discoverable (nav, a listing page, etc.).
   A page with **no inbound links** is only reachable by typing the URL — that's
   sometimes intentional (e.g. a soft-launch registration page).

There are no partials/includes, so the nav and footer are duplicated in each
file. If you change the nav sitewide, you change it in every page (find-and-replace).

---

## 7. Forms and the Apps Script backend (the important part)

Several pages have forms that submit to **Google Apps Script web apps** (serverless
Google scripts that write to Google Sheets and send branded emails). There is **no
traditional server or database** — Google Sheets *is* the database.

**Pages with forms:** `join`, `contact`, `partners`, `spotlight`, `competition`,
`mentoring/apply`, `cultural-competency`, `technology-networking`.

### The source of truth for scripts

The Apps Script code lives on disk at:

```
~/Documents/alsa-email-scripts/
    application.gs              (team recruitment applications)
    competition.gs             (legal writing competition)
    contact.gs                 (contact form)
    cultural-competency.gs     (event registration + waitlist + auto-close)
    membership-and-premium.gs  (join / membership)
    mentoring.gs               (mentee + mentor applications)
    spotlight.gs               (professional spotlight nominations)
    tech-networking.gs         (tech networking registration)
```

**Always edit the `.gs` file here, then paste it into the Apps Script editor** —
this folder is the canonical copy. Keep it in sync whenever a script changes.

### How a form works end to end

1. The page's `<script>` has an endpoint constant, e.g.
   `const CC_ENDPOINT = 'https://script.google.com/macros/s/.../exec'`.
2. On submit, JS builds a data object and `fetch()`es it to the endpoint with
   `mode: 'no-cors'` (so the browser can't read the reply — the page shows a
   generic "thanks" and the real result comes by email).
3. The Apps Script `doPost(e)` parses the JSON, appends a row to a Google Sheet
   tab, emails an internal notification, and emails the person a branded
   confirmation.
4. A shared `alsaEmailShell_(...)` function in each script renders the branded
   email (ALSA crest, maroon header, Instagram/LinkedIn footer).

### Deploying / updating a script (critical — people get this wrong)

Saving the code in the Apps Script editor is **not** enough. The live `/exec` URL
runs the last **deployed version**, not your latest save. To push changes live:

```
Paste code -> Cmd+S (save)
Deploy -> Manage deployments -> Edit (pencil) -> New version -> Deploy
```

"Manage deployments → New version" keeps the **same `/exec` URL** (so the website
keeps working). Use "New deployment" only for a brand-new form (it creates a new
URL you then paste into the page).

**Web app settings** when deploying: *Execute as: Me*, *Who has access: Anyone*.
After first deploy (or when scopes change), run the `testEmail()` function once
from the editor and authorise it — that grants Sheet + Gmail permissions.

### The master events Sheet

Event registrations (Cultural Competency, etc.) write to one master Google Sheet
called **"ALSA 2026 Events"**, with **one tab per event**. Each event has its own
standalone Apps Script project (separate `/exec` URL) that points at the same
Sheet by ID but writes to its own tab. The scripts auto-create their tab and
header row.

### GOTCHA: never copy script code out of a chat window

Copying code from a rendered chat/markdown block **drops spaces and mangles
quotes** (e.g. "you have" becomes "youhave", straight quotes become curly quotes
that break the script). Always copy from the `.gs` file or a plain-text editor.
Validate with `node --check yourscript.gs` (rename to `.js` first if needed).

---

## 8. Special systems to know about

### Events calendar (`/events/`)
Event cards carry a `data-date="YYYY-MM-DD"`. A small script at the bottom of the
page **auto-hides events whose date has passed**. So old events disappear on their
own — you don't have to remove them manually. The home page shows the **next four**
upcoming events using the same mechanism.

### Gallery (`/gallery/`) and the shared gallery data
Album data lives in **`/gallery-data.js`** (repo root) as `window.ALSA_GALLERY`,
a single source of truth loaded by **both** the Gallery page and the Home page.
Each album is an object: `slug`, `title`, `date`, `dateISO`, `description`,
`basePath`, `folder`, and a `photos: [...]` list of filenames. The **first photo**
in each list is the album cover. Photos load from `basePath`
(`/images/gallery/<folder>/`). To add an album: drop images in a new folder and
add an object to `gallery-data.js`. Missing image files auto-hide, so the grid
never shows broken tiles.

The Home page **"Moments" strip** is generated from the same `gallery-data.js`
(newest albums first, a few photos each), so adding albums/photos there updates
the home strip automatically. It links each thumbnail to `/gallery/?event=<slug>`
and strips a leading "Inaugural " from captions for brevity. **Edit
`gallery-data.js`, not the inline data** — the gallery page now reads from it.

### Deadline auto-close pattern
Registration/application pages with a deadline should **close themselves**. The
Cultural Competency page is the reference implementation: a `DEADLINE` JS constant
(`new Date('...T21:00:00+12:00')`, NZ time) that on page load hides the form and
shows a "registrations closed" notice once passed, and blocks late submissions.
**Standing rule: deadlines are 9:00pm, and the page must close automatically** —
don't rely on someone taking the form down by hand. (Learned the hard way: the
Legal Writing Competition kept accepting entries past its deadline because it was
only static text with no date logic.)

### Waitlist pattern (Cultural Competency script)
The registration scripts can cap confirmed places and overflow to a waitlist
(e.g. 50 confirmed + 10 waitlist = 60 total). The row's `Status` column records
Confirmed vs Waitlist; the confirmation email differs accordingly; and once full,
late registrants get a polite "full" email. Counts are derived from the Sheet's
last row — so **delete rows cleanly** ("Delete rows", not the Delete key), or
phantom blank rows inflate the count.

---

## 9. Local preview (before you push)

Because paths are root-absolute (`/images/...`), double-clicking an HTML file
won't load styles/images correctly. Run a local server from the repo root:

```
python3 -m http.server 8080 --bind 127.0.0.1
```

Then open http://127.0.0.1:8080/ (and any page path, e.g.
http://127.0.0.1:8080/events/). This is local-only and private. Hard-refresh
(Cmd+Shift+R) to bust cache after edits.

> Note: forms on a locally-served page still POST to the **live** Apps Script, so
> a test submission writes a real row and sends real emails. Fine for testing —
> just delete test rows afterwards.

---

## 10. Quick checklists

**Change some text or an image**
1. Edit the page's `index.html`  2. Preview locally  3. commit + push  4. hard-refresh.

**Add a new event to the calendar**
1. Copy an event card in `/events/index.html`, set `data-date`, tag, name, desc.
2. (Optional) mirror it on the home page and `/programmes/`.  3. push. Past events auto-hide.

**Add a new registration page + form**
1. Clone `/cultural-competency/index.html` (has form + waitlist + auto-close).
2. Rename the JS ids/consts, set date/venue/deadline copy.
3. Clone the matching `.gs` script, set its tab name + caps, deploy as a Web app,
   copy the `/exec` URL into the page's endpoint constant.
4. Add the "Register" link where it should be found.  5. push.

**Update an email/confirmation**
1. Edit the relevant `.gs` in `~/Documents/alsa-email-scripts/`.
2. Paste into Apps Script (from the file, not chat), save.
3. Deploy → Manage deployments → New version → Deploy.

---

## 11. Things that will trip you up (summary)

- **Push = deploy.** There is no separate deploy button for the website.
- **Apps Script save ≠ deploy.** Redeploy a *new version* for script changes to go live.
- **Root-absolute paths.** Always `/images/...`. Use a local server to preview.
- **No shared CSS/partials.** Nav, footer, and styles are duplicated per page.
- **Don't copy code from chat.** It corrupts spaces/quotes. Use the `.gs` files.
- **Sheets are the database.** Registrations/nominations live in Google Sheets.
- **Newsletter issues are dated archives.** Don't edit them for current info.
