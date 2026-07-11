# The True Cost — Loan Calculator

A loan-cost calculator with a daily-refreshed mortgage rate feed (via FRED/Freddie Mac)
and periodically-updated rates for auto, personal, and boat loans.

## What's in here

```
index.html              the calculator page (static, no build step) — MUST stay at repo root
api/rates.js             serverless function — the frontend calls this
api/refresh.js           serverless function — Vercel Cron calls this once a day
data/rates-manual.json   rates for loan types with no free public API — edit by hand
vercel.json              defines the daily cron schedule
package.json             the one dependency: @vercel/kv
```

**Important:** `index.html` needs to live at the repo root, not inside a `public/` or `src/`
folder — Vercel's zero-config static hosting looks for it there by default.

## Redeploying this fixed version

If you already have the project connected in Vercel from before:

1. In your GitHub repo, delete the old files and replace them with everything in this folder
   (in particular, make sure `index.html` ends up at the root, not inside `public/`)
2. Commit and push — Vercel redeploys automatically
3. Check **Deployments** in the Vercel dashboard; the latest one should show a green "Ready" status
4. Your `your-project.vercel.app` URL and `loantruecost.com` should both work once that's green

## Deploy it from scratch (about 10 minutes)

**1. Get a free FRED API key**
https://fredaccount.stlouisfed.org/apikeys — free account, generate a key.

**2. Push this project to GitHub**
Create a new repo, add these files (index.html at the root), commit, push.

**3. Create a Vercel account and import the repo**
https://vercel.com → Add New Project → select the repo → Deploy.
It will deploy successfully even before the next steps — it just won't have live rates yet.

**4. Add a KV store**
Project → **Storage** tab → **Create Database** → **KV** → connect it to this project.
Vercel automatically adds the required environment variables for you.

**5. Add two more environment variables**
Project **Settings → Environment Variables**:
- `FRED_API_KEY` — the key from step 1
- `CRON_SECRET` — any random string you make up

**6. Redeploy**
Environment variable changes need a redeploy — go to **Deployments**, redeploy the latest one.

**7. Trigger the first refresh**
**Settings → Cron Jobs** → hit "Run" next to `/api/refresh` to populate the cache immediately,
rather than waiting for the next scheduled run (9am ET / 13:00 UTC daily by default).

**8. Custom domain**
Since you bought `loantruecost.com` through Vercel directly, this part should be close to
automatic — Project **Settings → Domains**, add it if it isn't already attached to this
project. Domains bought through Vercel skip the usual DNS-propagation wait.

## If it's still not working

Check **Deployments → (latest) → Build Logs** for red error text — that tells you exactly
what failed, and is worth pasting back here if something looks unfamiliar. The two most common
issues at this stage:
- **Build fails outright**: usually a typo or missing file — the log will point at the exact line
- **Function crashes at runtime (500 on /api/rates or /api/refresh)**: check
  **Deployments → (latest) → Functions** tab, click the function, and look at its logs —
  a missing `FRED_API_KEY` or an unconnected KV store are the most common causes

## Updating the manual rates

Auto, personal, and boat loan rates have no free public API, so they live in
`data/rates-manual.json`. Check current averages every so often (Bankrate and Experian
publish weekly), edit the numbers, commit, and push.

## Costs

Everything here fits comfortably in Vercel's free tier.
