# Why We Lose Bookings — Dashboard

A static HTML/JS prototype for a "revenue lost to cancellations" report,
built on top of the [booking_project_pipeline](https://github.com/jameslimpin1/booking_project_pipeline)
dbt/DuckDB analytics pipeline (synthetic hotel booking + guest-chat data).

This repo holds only the dashboard — no build step, no dependencies, no
data pipeline. All chart and drill-down data is embedded directly in the
HTML as static JSON and refreshed from the pipeline's warehouse by a script
that lives in that repo (see [Data & refresh](#data--refresh) below).

## Pages

- **`dashboard_prototype.html`** — executive summary. Headline stat (1 in 7
  bookings cancel, 14.8% baseline), revenue lost to cancellation, where in
  the guest-conversation journey bookings are usually lost, and the
  "unanswered opening message" finding (guests whose first message never
  gets a reply cancel at nearly 1 in 4 — the single clearest fixable driver
  in the data).
- **`dashboard_prototype_page2.html`** — explorer / drill-down. Month ×
  channel × loyalty-tier slicers over the same data, a "what happened"
  impact heatmap (event pattern × month), top complaint/everyday-question
  cohorts, the price-quintile revenue-risk chart, and a conversation-level
  drill-down (event journey → full message transcript) for a curated set of
  216 conversations: every unanswered-opening case, the 50 highest
  cancellation-risk bookings, and 80 typical conversations for contrast.

## Viewing locally

No build step — open either file directly in a browser:

```bash
open dashboard_prototype.html
```

Or serve both pages together (needed for the page 1 → page 2 link to resolve
via a local server instead of `file://`):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/dashboard_prototype.html
```

## Data & refresh

The data embedded in both pages (`<script type="application/json">` blocks)
is generated from the marts in
[booking_project_pipeline](https://github.com/jameslimpin1/booking_project_pipeline)'s
DuckDB warehouse — it is not queried live, so it goes stale whenever the
pipeline's underlying data or dbt models change.

To refresh it, from the **pipeline** repo (checked out as a sibling
directory to this one):

```bash
cd ../booking_project_pipeline
scripts/refresh_pipeline.sh
```

This rebuilds the pipeline's warehouse (`dbt build`), checks source
freshness, then re-queries the marts and rewrites this repo's two HTML
files' JSON blocks in place — everything else (layout, JS, styling) is left
untouched. If this repo isn't checked out as `../booking_project_dashboard`
relative to the pipeline repo, pass its path explicitly:

```bash
scripts/refresh_pipeline.sh --dashboard-dir /path/to/booking_project_dashboard
```

The refresh only edits files on disk — it doesn't commit or push. Review the
diff and commit here separately once you're happy with the result:

```bash
git status
git add dashboard_prototype.html dashboard_prototype_page2.html
git commit -m "Refresh dashboard data"
git push
```

See the pipeline repo's README (["Keeping the dashboard
fresh"](https://github.com/jameslimpin1/booking_project_pipeline#keeping-the-dashboard-fresh))
for what the refresh script actually queries and how the drill-down
conversation set is chosen.

## Data note

The underlying dataset is synthetic (see the pipeline repo's
`generate_hotel_chat_data.py`), so findings validate the analysis
*approach*, not real business figures — treat this as "the analysis we'd
run on production data," not as live results.
