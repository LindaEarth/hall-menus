# Weekly hall menus update — runbook

Run every Monday. Two runs: **main** (09:53 Europe/London) and **retry** (13:53 Europe/London).
Both run in the original Claude chat (not a fresh scheduled session), because only that chat can push to this repository.

Outputs (both must always show the same page):
- Claude artifact: https://claude.ai/artifact/YWQ62immEu5C5dRDX9Kqmz
- Public page: https://lindaearth.github.io/hall-menus/ (this repo's `index.html`)

## 0. Fetching rules (these caused past mistakes)

- Fetch each page with its **exact URL as written below. Never add query parameters** (`?refresh=…`, `?v=…`): the fetch tool then returns an old cached copy.
- **Always verify the date** before using any source:
  - Queens': the "Week Commencing" heading must equal this Monday.
  - Pembroke: the PDF header must say the expected `WEEK n`.
  - St Catharine's: the PDF header dates must include this Monday (dd.mm).
- If a source shows an older week, do **not** use it. Show the "isn't up yet / published yet" message for that college. The retry run tries again.
- Do not trust links listed on the Pembroke servery page (the fetch tool only sees an old copy). Use the fixed files below.

## 1. Start from the live page

Read the artifact (Artifact tool, action `read`). Keep everything except the menu data in `<script>` (from `/* WEEK_START` to `/* ---------- rendering`) and the Sources `<ul>`. Never change the design, the default filter ("Vegetarian & vegan"), the legend or the "What's behind the menu" cards.

Set `const WEEK_START="YYYY-MM-DD"` to this Monday. Only the current week is kept; replace all old data.

## 2. Sources and parsing

### Queens'
URL: https://www.queens.cam.ac.uk/life-at-queens/catering/dining-hall/weekly-menu/
Labels: (Vegan) → Vegan, (V) → Vegetarian, unlabelled mains → Meat & fish; (Halal) → `H("…")` tag. Soup and desserts are not shown; sides go in `sides`.
Sunday lunch = brunch (leave index out). Saturday lunch may be listed (e.g. "in The QBar"): include it as index 5.
Format: `QNS={lunch:[…],dinner:[7 days]}`, each `{vegan:[],veg:[],meat:[],sides:[]}`.
If not up yet: `const QNS_MSG={msgHtml:"This week's menu isn't up yet."}` and `qns()` returns it for every day.

### Pembroke (three files rotate until w/c 30 Nov 2026)
- Week 1 (w/c 5 Oct, 26 Oct, 16 Nov): https://www.pem.cam.ac.uk/sites/default/files/week_1_js_2.pdf
- Week 2 (w/c 12 Oct, 2 Nov, 23 Nov): https://www.pem.cam.ac.uk/sites/default/files/week_2_du.pdf
- Week 3 (w/c 19 Oct, 9 Nov, 30 Nov): https://www.pem.cam.ac.uk/sites/default/files/week_3_lc.pdf
From w/c 7 Dec: no known file. Show "This week's menu isn't published yet." and ask the user for the new PDF link.

Ask the fetch tool to "output every text line in order, keeping duplicates". Order: Main lunch Mon–Fri (5), Alternative lunch (5), Allergen-friendly lunch (5), Main dinner Mon–Sun (7), Alternative dinner (7), Allergen-friendly dinner (7), Pudding (7, not shown), Sides (5 per day Mon–Sun, each group starts with "Chips"). Sat/Sun lunch = brunch.
Diet: per meal, the one dish most likely vegan (usually the allergen-friendly dish, unless it is meat/fish) → Vegan; other meat-free → Vegetarian; meat/fish → Meat & fish. Never write "allergen friendly" on the page.
Format: `PEM_SIDES` (7 arrays), `PEM_L` (5 objects + `null,null`), `PEM_D` (7 objects).

### St Catharine's (four-week rotation)
Page: https://www.caths.cam.ac.uk/students/college-facilities-and-forms/catering-for-our-community/cafeteria
Pick the week whose PDF header includes this Monday. Read its Lunch, Dinner and Weekend PDFs.
Weekday text order: 3 dish+sides pairs = Friday; next 2 = Tuesday counters 2–3; next 3 = Thursday; after "MONDAY / WEDNESDAY": 3 Monday, 3 Wednesday, then 1 = Tuesday counter 1. Weekend PDF (dinner): first 3 = Saturday, next 3 = Sunday.
Vegan only if the name says vegan or plant-based; otherwise meat-free → Vegetarian.
Format: `CAT={lunch:[5 days],dinner:[7 days]}`, each day `[[dish,[sides],"veg|vegan|meat"],…]`.

### Peterhouse
Leave as is ("Not loaded. Check PETMENU").

## 3. Sources list
Update the Sources `<ul>` to this week's pages/PDFs; note any college that isn't up yet.

## 4. Publish (both, always)
1. Extract `<script>` and run `node --check`.
2. Publish the artifact (same URL). The artifact file has no doctype/html/head/meta tags.
3. In this repo: `index.html` = these four lines + the exact artifact HTML:
   ```
   <!doctype html>
   <html lang="en">
   <meta charset="utf-8">
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
   ```
   `git fetch origin main && git reset --hard origin/main` first, then commit "Update hall menus for week of <date>" and push. Confirm with `git ls-remote origin main`.

## 5. Retry run (13:53)
Re-check only colleges showing a "not up yet / published yet" message. If any filled in, publish both outputs. Always confirm GitHub has `WEEK_START="<this Monday>"`; if not, push.

## 6. Keep the schedule going
At the end of the **main** run, schedule (send_later into this chat):
- today 13:53 Europe/London: the retry message
- next Monday 09:53 Europe/London: the main message
(UK time is GMT from the last Sunday of October, BST from the last Sunday of March.)
At the end of the **retry** run, check (list_triggers, include_completed false) that next Monday's main run is scheduled; if not, schedule it.

## 7. Report
Tell the user in one short message: week loaded, any college missing, any dish that couldn't be placed, and that both the Claude page and GitHub were updated (quote any error).
