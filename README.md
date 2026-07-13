# Fantasy Big Brother — Standings Site + Master Sheet

One website showing all six leagues, plus a **master Google Sheet** you edit once that feeds every league. Theme matches your sheets' light-blue palette.

Open `index.html` now — it runs in **demo mode** with all six of your real leagues (BBoos & Luci, Biddie Poo Brother, Boof Brother, The Pentagon [draft in progress], Max and Gerard, Ragan Fox's Fan Club) so you can see it before wiring anything.

---

## The big picture: how "edit once → everything updates" works

A plain Google Sheet **cannot push** values into other sheets. Data only flows the *other* way: a sheet can **pull** from another using `IMPORTRANGE`. So the setup is:

- **MASTER SHEET** — you edit this once each week (CompetitionResults + SeasonLongResults).
- Each **league sheet** pulls those cells from the master via `IMPORTRANGE`, so its own formulas recompute automatically. You can still open and browse each league sheet — it's always current.
- The **website** also reads the master (via two published CSV tabs) so players can check standings.

You edit the master's CompetitionResults + SeasonLongResults → every league sheet updates → the website updates. **Yes, you can still open any individual league sheet; it just gets its shared inputs from the master instead of you typing them in six times.**

What stays local to each league sheet: its rosters/draft, its point-value key, and any league-specific awards. Those differ per league and rarely change after the draft.

---

## Part A — Build the master sheet

1. Create one new Google Sheet named e.g. **BB28 MASTER**.
2. Give it two tabs, copied from any existing league workbook so the layout matches exactly:
   - **CompetitionResults** (the weekly table: Week, Double?, HOH, Have Nots, nominees, veto, evicted, jury, etc.)
   - **SeasonLongResults** (Award | Recipient | Point Value)
3. This is the only place you enter weekly results from now on.

## Part B — Point each league sheet at the master (one time per sheet)

In each of your six league workbooks, replace the CONTENTS of CompetitionResults and SeasonLongResults with a pull from the master.

1. Get the master's ID from its URL: docs.google.com/spreadsheets/d/**THIS_LONG_ID**/edit
2. In a league sheet's **CompetitionResults** tab, in the first data cell of the table (typically **A3** — leave the title/header rows alone), enter:

   `=IMPORTRANGE("PASTE_MASTER_ID_HERE","CompetitionResults!A3:P21")`

   The first time, the cell shows `#REF!` with an **Allow access** button — click it once to authorize.
3. Do the same on **SeasonLongResults**. In its first recipient cell, pull the Recipient column:

   `=IMPORTRANGE("PASTE_MASTER_ID_HERE","SeasonLongResults!B3:B38")`

   Only pull the columns the master owns (the Recipient names); keep each league's own point values local if they differ.
4. The rest of that league's tabs (CompetitionSummary, PointsDetails, rankings) keep their existing formulas and recompute from the imported data.

> Ranges above match the Max and Gerard layout (weeks rows 3–21, awards rows 3–38). Adjust per sheet if needed — send me one and I'll give exact ranges.

Repeat for all six. From then on: edit the master, every league updates.

## Part C — The website's data (two published tabs)

The website reads two simple published tabs (put them in the master sheet or a small companion sheet):

1. **SeasonPoints** — itemized point log: `Houseguest | Value | Description`. This is your workbook's **PointsDetails** content. Same season for all leagues, so one league's PointsDetails is the shared feed.
2. **Leagues** — every league's rosters: `League | Season | Alliance | Owner | Houseguest`, one row per houseguest. **Already done for you:** `Leagues_prefilled.csv` has all 77 roster rows from your six workbooks. Import it (File → Import → Upload). Add The Pentagon's rows once its draft is final.

Publish each tab: **File → Share → Publish to web → pick the tab → Comma-separated values (.csv) → Publish**, copy each link, and paste into the `CONFIG` block at the top of `index.html`:

    const CONFIG = {
      POINTS_CSV: "https://docs.google.com/.../pub?gid=...&output=csv",
      LEAGUES_CSV: "https://docs.google.com/.../pub?gid=...&output=csv"
    };

Leave them `null` to keep demo mode.

## Part D — Host on GitHub Pages

1. New repo (e.g. `fantasy-bb`).
2. Upload `index.html` to the root.
3. **Settings → Pages → Deploy from a branch → main → / (root)**. Save.
4. Live in ~1 min at `https://YOUR-USERNAME.github.io/fantasy-bb/`. Share with players.

---

## Weekly routine, once set up
Enter the week's results in the **master** CompetitionResults (and SeasonLong recipients). Every league sheet updates via IMPORTRANGE. Keep the SeasonPoints tab current, and the website shows new standings on next load.

## Notes
- Published tabs are readable by anyone with the link — fine for standings; keep nothing private there.
- `IMPORTRANGE` can take a moment to refresh and occasionally needs a re-open; normal Google Sheets behavior.
- Adding a league later = add its rows to the Leagues tab. It appears in the site automatically; no code changes.
