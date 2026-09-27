# Dark Horse Hoops

This repo is the entire public-facing side of Dark Horse Hoops. It is a static site
with no server, no database, and no build step. Google Sheets is the database;
this repo is just the display layer, and it never changes when a new client is added.

## Files

| File | What it is |
|---|---|
| `index.html` | The public landing page at the domain root. Marketing/contact only, no client data. |
| `board.html` | Renders one sourcing board. Reads a `?id=` link, decodes it, fetches the matching Google Sheet data, and displays it. With no `?id=`, it shows a built-in demo board. |
| `link-generator.html` | Private tool. Turns two Google Sheet tab links into one board link. Not linked from the public site -- bookmark it directly. |
| `CNAME` | Tells GitHub Pages this site should answer to darkhorsehoops.net. |

## How a board link actually works

Every report lives in its own Google Sheet, duplicated from the Dark Horse report
template. `link-generator.html` reads the link to the `Report Brief - Published`
tab and the `Players - Published` tab, pulls out the spreadsheet id/key and each
tab's `gid`, and packs `type:key|briefGid|playersGid` into one scrambled string --
that string is the `?id=` value in the final board link. `board.html` reverses
that exact process: decode the id, rebuild the CSV URLs, fetch them, check
`Publish = YES`, and render.

There is no database and no directory of clients anywhere in this repo or on the
web. Each link is self-contained. This is obscurity, not authentication. Anyone
with a link can view that board indefinitely. Client A's link cannot be used to
discover Client B.

**There are two ways to get CSV data out of the Sheet, and they have different
security properties. Pick one, deliberately, per report:**

- **Normal Sheets link (`type: "N"`).** Share the whole spreadsheet as *Anyone
  with the link -> Viewer*, then just copy the normal address-bar URL for each
  tab. Simplest to use, but once the whole file is link-shared, **every tab is
  fetchable by URL if someone has or guesses its `gid`** -- Internal tabs
  included. Only use this if there's nothing on the Internal tabs (Skyler
  Notes, Gemini prompts, etc.) you'd mind a stranger seeing.
- **"Publish to web" link (`type: "P"`).** File -> Share -> Publish to web,
  publish the *individual tab* (never "Entire document"). More setup per
  report, but Internal tabs stay completely unreachable regardless of anyone
  guessing a `gid`, because they were never published at all.

`link-generator.html` accepts either kind of link and auto-detects which one
you pasted. Mixing the two types across the Brief and Players link for the same
report isn't allowed -- pick one approach per report.

## Adding a new client report

No code changes, ever. The steps:

1. Duplicate the Dark Horse Sheet template in Google Drive.
2. Fill in Team Intel and Players on the Internal tabs.
3. Set `Publish = NO` and `Branding_Mode` while you work.
4. Decide which sharing approach you're using for this report (see above), and
   set it up: either share the whole Sheet as "Anyone with the link," or
   publish the two `- Published` tabs individually as CSV.
5. Set `Publish = YES` when ready.
6. Paste the two tab links into `link-generator.html`, generate the link, send it.

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
`<script>`) -- Position, Height, Fit Rating, Level Rating, Risk Level, Budget
Level, PPG, RPG, APG, 3pt Attempts per game, Stl Rate, Blk Rate. Those first six
came from columns Skyler added to the live template (M-Q and V-X on
`Players - Published`); older reports made before those columns existed just
quietly don't show that metric, nothing breaks.

## Compare / Heatmap view

Every board can show two views: **Cards** (unchanged) and **Compare**, one full
heatmap table. Compare checks which of the built-in metrics actually have a
matching column on this report's `Players - Published` data and only shows
those -- if a report is ever missing one (or all) of them, that metric (or the
whole Compare toggle) just quietly doesn't appear. Cards always work regardless.
Every metric the report has data for shows as its own column, all at once --
there's no tab that splits columns into groups; the position chips above the
table filter which *players* are shown, which is separate from that. Compare
sorts by Fit Rating (best first) by default; click any column header to resort.

Each built-in metric has a display name, a format (`PERCENT`, `DECIMAL`,
`COUNT`, `FEET_INCHES`, `SCALE`, or `CATEGORICAL`, controlling how the raw
sheet value is displayed), and a direction (`HIGHER_IS_BETTER`,
`LOWER_IS_BETTER`, or `NEUTRAL`) -- direction only matters now to mark
`NEUTRAL` columns (Position, Height) as non-heat; it no longer changes which
end of the scale looks hot. `SCALE` is for the four rating columns -- Gemini
may fill them in as a number (any scale) or a word (Low/Medium/High); either
works for sorting and heat-mapping, and the sheet's literal text is always
what's actually displayed. `NEUTRAL` columns (Height) are left out of the
heat scale entirely, since they aren't a magnitude.

Each cell's heat is computed strictly within its own column -- a value is
only ever compared against other rows in that same column, never against a
neighboring column, so each cell paints its own flat color rather than
blending into the cells next to it (a warm cell next to a cool cell is just
two unrelated columns, not one flowing scale). A higher number in a column
always reads warmer (orange, tracking the live `--accent-1` brand color) and
a lower number always reads cooler (blue), full stop, regardless of whether a
high or low number is actually the "good" outcome for that particular metric
-- so on Risk Level, for instance, a high risk number reads hot/red-ish and a
low one reads cool/blue, which is the intuitive read for a risk column.
`CATEGORICAL` (Position) is not part of the heat scale at all -- it gets a
fixed, stable color per distinct value instead (fully independent of the
other rows), so it reads as a position legend/tag rather than a score.
Hovering a row brightens it as a light-up affordance. There is no
sheet-driven config of any kind -- adding or changing a metric means editing
`BUILT_IN_METRICS` in `board.html`'s code, not touching any spreadsheet.

Real values are always shown (the color is a visual layer on top, computed
relative to whatever's currently visible -- it never replaces the number).
Missing data always shows as `—`, never estimated or treated as zero. On mobile,
Compare becomes a "pick up to 4 players" selector with a stacked comparison,
rather than a shrunk table.

Fit Rating also drives the gold meter bar at the bottom of each player card
(falls back to a rough PPG-based estimate if Fit Rating isn't filled in yet),
and a small "TOP PICK" badge appears on cards (and in the dossier) where
`Top Recommendation Badge` is `Yes`.

## Skill icons in the dossier

Each player can show up to four skill icons in the expanded dossier, driven by
four columns on `Players - Published`: `Skill 1`, `Skill 2`, `Skill 3`, `Skill 4`
(see `SKILL_FIELDS` near the top of `board.html`'s `<script>`). Unlike every
other player field, these columns don't hold display text -- each one holds a
direct image URL pointing at an icon in the `Skill Bank` tab's asset list, and
the icon artwork itself already has the skill name baked into the image, so no
separate text label is needed next to it.

A slot is only rendered if it looks like a real URL (`isLikelyImageUrl`); a
blank cell, or anything that isn't a URL, is silently skipped rather than
leaving a broken icon or a gap -- so a player can show fewer than four skills
with nothing looking wrong. If an icon URL is present but the image itself
fails to load, that one icon tile disappears rather than showing a broken
image.

Gemini fills in these four columns per player (see the Skill Bank tab's
instructions for exactly how it picks which four skills and resolves each to
an icon URL) -- `board.html` itself has no opinion on which skills exist or
what the icons look like, it just renders whatever URLs show up in these four
columns.

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
