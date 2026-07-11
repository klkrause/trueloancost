# The True Cost — Loan Calculator

A loan-cost calculator with a daily-refreshed mortgage rate feed (via FRED/Freddie Mac)
and periodically-updated rates for auto, personal, and boat loans.

## What's in here

```
public/index.html     the calculator page (static, no build step)
api/rates.js           serverless function — the frontend calls this
api/refresh.js          serverless function — Vercel Cron calls this once a day
data/rates-manual.json  rates for loan types with no free public API — edit by hand
vercel.json             defines the daily cron schedule
package.json            the one dependency: @vercel/kv
```

## Deploy it (about 10 minutes)

**1. Get a free FRED API key**
Go to https://fredaccount.stlouisfed.org/apikeys, create an account, and generate a key.
This is what pulls live 30-year and 15-year mortgage rates. Free, no rate limits that matter here.

**2. Push this project to GitHub**
Create a new repo, add these files, commit, push. (If you're not comfortable with git yet,
GitHub's "upload files" button in the web UI works fine for a one-time push.)

**3. Create a Vercel account and import the repo**
- Go to https://vercel.com, sign up (free), click "Add New Project"
- Select your GitHub repo, click Deploy — it'll build successfully even before the next steps,
  it just won't have live rates yet

**4. Add a KV store**
- In your Vercel project, go to the **Storage** tab → **Create Database** → **KV**
- Connect it to this project — Vercel automatically adds the required environment
  variables (`KV_URL`, `KV_REST_API_URL`, etc.) for you

**5. Add two more environment variables**
Project **Settings → Environment Variables**:
- `FRED_API_KEY` — the key from step 1
- `CRON_SECRET` — any random string you make up (protects the refresh endpoint from
  being triggered by randoms; Vercel automatically sends it when Cron calls the endpoint)

**6. Redeploy**
Settings changes require a redeploy to take effect — go to **Deployments** and redeploy
the latest one, or just push an empty commit.

**7. Trigger the first refresh**
The cron (set in `vercel.json`, currently 9am ET / 13:00 UTC daily) will run automatically,
but you don't have to wait — go to **Settings → Cron Jobs** in the Vercel dashboard and hit
"Run" next to `/api/refresh` to populate the cache immediately.

**8. (Optional) Add a custom domain**
Project **Settings → Domains** → add your domain → follow the DNS instructions it gives you.
If you don't have a domain yet, any registrar works (Namecheap, Cloudflare, Google Domains,
or buy one straight through Vercel). Until then, your site is already live at the
`your-project.vercel.app` URL Vercel gives you after the first deploy.

## Updating the manual rates

Auto, personal, and boat loan rates have no free public API, so they live in
`data/rates-manual.json`. Check current averages every so often (Bankrate and Experian
publish weekly), edit the numbers, commit, and push — Vercel redeploys automatically and
the next cron run picks up the new values.

## Costs

Everything here fits comfortably in Vercel's free tier: one cron run a day, a handful of
KV reads/writes, and a static page. FRED's API is free with no practical rate limit for
one request a day.
