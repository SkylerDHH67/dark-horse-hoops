# Dark Horse Hoops

This repo is the entire public-facing side of Dark Horse Hoops. It is a static site
with no server, no database, and no build step. Google Sheets is the database;
this repo is just the display layer, and it never changes when a new client is added.

## Files

| File | What it is |
|---|---|
| `index.html` | The public landing page at the domain root. Marketing/contact only, no client data. |
| `board.html` | Renders one sourcing board. Reads a `?id=` link, decodes it, fetches the matching Google Sheet data, and displays it. With no `?id=`, it shows a built-in demo board. |
| `link-generator.html` | Private tool. Turns two Google Sheet "publish to web" links into one board link. Not linked from the public site -- bookmark it directly. |
| `CNAME` | Tells GitHub Pages this site should answer to darkhorsehoops.net. |

## How a board link actually works

Every report lives in its own Google Sheet, duplicated from the Dark Horse report
template. Two tabs in that sheet get published to the web as CSV: `Report Brief -
Published` and `Players - Published`. `link-generator.html` takes both of those
published links, pulls out the shared spreadsheet key and each tab's `gid`, and
packs `key|briefGid|playersGid` into one scrambled string -- that string is the
`?id=` value in the final board link. `board.html` reverses that exact process:
decode the id, rebuild both CSV URLs, fetch them, check `Publish = YES`, and render.

There is no database and no directory of clients anywhere in this repo or on the
web. Each link is self-contained. This is obscurity, not authentication. Anyone
with a link can view that board indefinitely. Client A's link cannot be used to
discover Client B.

**Security-relevant step that must happen in Google Sheets, not here:** when
publishing a report's tabs to the web, always publish the *individual tab*
("Report Brief - Published" or "Players - Published" specifically), never
"Entire document." Publishing the whole document would make the Internal tabs
fetchable by anyone who guessed their `gid`, even though nothing links to them.

## Adding a new client report

No code changes, ever. The steps:

1. Duplicate the Dark Horse Sheet template in Google Drive.
2. Fill in Team Intel and Players on the Internal tabs.
3. Set `Publish = NO` and `Branding_Mode` while you work.
4. Publish the two `- Published` tabs to the web as CSV (individually, see above).
5. Set `Publish = YES` when ready.
6. Paste both published links into `link-generator.html`, generate the link, send it.

## Whitelisting

`board.html` only ever reads a fixed, named list of columns (see `BRIEF_FIELDS`
and `PLAYER_FIELDS` near the top of its `<script>`). Anything else in the sheet --
Skyler Notes, Gemini prompts, internal-only columns -- is structurally invisible
to the frontend, not just hidden by convention. If a future column needs to reach
the site, it has to be added to one of those two arrays on purpose.

## Deployment

This site is served by GitHub Pages directly from this repo (Settings -> Pages).
`CNAME` points it at **darkhorsehoops.net**. DNS for that domain needs an A record
(apex) pointing at GitHub's Pages IP addresses, set at whichever registrar the
domain lives at. If Pages ever needs to be reconfigured, that's the one setting
that matters -- everything else is just static files.

## One placeholder to update

`index.html`'s contact link points to `hello@darkhorsehoops.net`. That inbox
doesn't necessarily exist yet -- set up an actual mailbox or forwarding rule for
it (or swap in whatever address you actually want to use) before treating the
site as fully live.
