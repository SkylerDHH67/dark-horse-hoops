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
Published` and `Players - Published`. `link-generator.html` takes those two
published links, pulls out the shared spreadsheet key and each tab's `gid`, and
packs `key|briefGid|playersGid` into one scrambled string -- that string is the
`?id=` value in the final board link. `board.html` reverses that exact process:
decode the id, rebuild the CSV URLs, fetch them, check `Publish = YES`, and render.

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
4. Publish the `Report Brief - Published` and `Players - Published` tabs to the
   web as CSV (individually, see above). Both are required.
5. Set `Publish = YES` when ready.
6. Paste the two published links into `link-generator.html`, generate the link, send it.

## Whitelisting

`board.html` reads a fixed, named list of columns for the Report Brief (see
`BRIEF_FIELDS` near the top of its `<script>`) -- that part is unchanged. For
players, Card view and the Dossier only ever read the same named fields they
always have; anything internal-only (Skyler Notes, Gemini prompts) never reaches
a Published tab in the first place, so it's structurally invisible to the site,
not just hidden by convention.

Compare view works differently on purpose: rather than reading whatever columns
happen to exist, it only ever looks for a small, fixed list of metrics that are
hardcoded in `board.html` itself (see `BUILT_IN_METRICS` near the top of its
`<script>`) -- Height, Age, PPG, RPG, APG, FG%. Those six are already present on
every `Players - Published` tab today, so Compare works on every existing and
future report with zero Google Sheet changes, ever.

## Compare / Heatmap view

Every board can show two views: **Cards** (unchanged) and **Compare**, a data
heatmap. Compare checks which of the built-in metrics actually have a matching
column on this report's `Players - Published` data and only shows those -- if a
report is ever missing one (or all) of them, that metric (or the whole Compare
toggle) just quietly doesn't appear. Cards always work regardless.

Each built-in metric has a display name, a category (PHYSICAL, SCORING,
SHOOTING, PLAYMAKING, REBOUNDING -- these group and order the columns on
screen), a format (`PERCENT`, `DECIMAL`, `COUNT`, or `FEET_INCHES`, controlling
how the raw sheet value is displayed), and a direction (`HIGHER_IS_BETTER`,
`LOWER_IS_BETTER`, or `NEUTRAL` -- neutral is for things like height or age
where "bigger" isn't automatically "better").

There is no sheet-driven config of any kind -- adding or changing a metric means
editing `BUILT_IN_METRICS` in `board.html`'s code, not touching any spreadsheet.

Real values are always shown (heat intensity is a visual layer on top, computed
relative to whatever's currently visible -- it never replaces the number).
Missing data always shows as `—`, never estimated or treated as zero. On mobile,
Compare becomes a "pick up to 4 players" selector with a stacked comparison,
rather than a shrunk table.

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
