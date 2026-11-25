# SERP tracking API: what 1,000 rank checks really cost, how to reach top 100, and when ScraperAPI is worth it (credit math + plan comparison)

If you plan to pull Google rankings through ScraperAPI, budget **25 credits per request**, not one. ScraperAPI prices Google and Bing (including their subdomains) at 25 credits a request, so the free 1,000 credits that come with a new account buy you 40 Google searches, not a thousand. That single number decides whether the service fits your rank tracking project, and it is the thing most "best SERP API" roundups skip.

Below is the credit math for a realistic tracking job, every plan ScraperAPI currently lists and what changes between them, the endpoint parameters you actually get, and the cases where a per-SERP provider is the cheaper answer.

> **Quick verdict:** ScraperAPI makes sense when you already run scraping through it, want one API key and one bill for Google SERP JSON plus page scraping, and need country/TLD/location targeting without a separate contract. It is a poor fit if your only workload is tens of thousands of cheap rank checks a day. 👉 [Check the current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How ScraperAPI charges for a Google search

One credit equals a standard page request. Harder targets carry a fixed multiplier, and the multiplier is published rather than hidden in a quote.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon | 5 |
| Google or Bing (including subdomains) | 25 |
| LinkedIn | 30 |
| Site behind Cloudflare, DataDome or PerimeterX | +10 on top of the base cost |

Two things follow from that table. A rank tracking job costs 25× a normal scrape of the same volume, and a Google search that hits bot protection could cost more again. ScraperAPI gives you a Domain Cost Estimator in the dashboard and a `max_cost` parameter per scrape, which is how you keep a surprise target from eating a day's allowance.

The free tier is 1,000 credits with a maximum of 5 concurrent connections. At 25 credits a Google search, that is 40 searches. Enough to verify the JSON shape, the location parameters and your parsing logic. Not enough to run a pilot on 200 keywords.

## Every ScraperAPI plan

ScraperAPI's monthly price per plan moves with the credit allowance and the billing cycle, and those numbers change on the pricing page. I am not going to paste a table of figures that will be wrong by the time you read it. What does not change quickly is the structure, and the structure is what determines what happens when you run out of credits mid-month.

| Plan | Auto-upgrade when credits run out | Pay-as-you-go past the limit | What it is for |
| --- | --- | --- | --- |
| Free | No | No | 1,000 credits, 5 concurrent connections, endpoint testing |
| Hobby | Yes | No | First real tracking runs on a small keyword set |
| Startup | Yes | No | Small agency or in-house team, single product |
| Business | Yes | No | Multiple projects sharing one key |
| Scaling | Yes | Yes | Volume that fluctuates month to month |
| Professional | Yes | Yes | Heavy scheduled tracking plus scraping |
| Advanced | No (PAYG instead) | Yes | Sustained high volume with a spending cap |
| Enterprise | No (PAYG instead) | Yes | Custom volume and terms |

The distinction that matters: on Hobby through Professional you can switch on auto-upgrade, and your account climbs to the next plan when credits hit zero. On Scaling, Professional, Advanced and Enterprise you can turn on pay-as-you-go instead and keep scraping at a fixed per-credit rate up to a monthly cap you set yourself. Annual plans can renew early for a fresh batch of credits.

Going over your concurrency limit does not cost extra. You get a 429, the request is refused rather than queued, and you retry. For a rank tracker that fires a batch at a fixed hour, that behaviour is workable but you need retry logic with backoff, because a refused request is a missing data point in your history table.

👉 [Compare the plan allowances on ScraperAPI's pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The multipliers nobody puts in the pricing calculator

This is where a rank tracking budget actually breaks, and it applies to every provider, not just this one.

**Top 100 stopped being one request.** Google retired the `num=100` parameter in September 2025. A page of results is 10 organic entries. To see positions 1–100 you now paginate, roughly ten requests per keyword. At ScraperAPI's 25 credits per Google request, a top-100 check for one keyword on one device costs about **250 credits**. Ten keywords a day on desktop and mobile is 5,000 credits a day, before you have tracked anything interesting.

**Every device is a separate search.** Mobile rankings are not a filter on the desktop result. Tracking 500 keywords on desktop and mobile means 1,000 checks, not 500. With a credit-based API that doubles the bill. Some rank trackers count a mobile keyword as an extra tracked keyword; others bundle both devices into one price. Confirm which model you are buying before comparing headline numbers.

**Depth is a product decision, not a technical one.** Most teams only need top 100 for a short list of money keywords and top 10 or 20 for everything else. If you paginate blindly across the whole portfolio, the credit bill scales with ambition rather than value.

Run your own numbers before you commit: keywords × devices × pages per keyword × 25 credits, times your check frequency. If the monthly total lands near a plan boundary, price the next tier up as well, because auto-upgrade will take you there anyway.

## What ScraperAPI's Google SERP endpoint gives you

The endpoint is `https://api.scraperapi.com/structured/google/search` and it returns parsed JSON (CSV is also available) rather than raw HTML you have to clean.

Documented parameters include `query`, `country_code` for geo targeting, `tld` for the Google domain (com, co.uk, ca, de, es, fr, it, co.jp, in, com.sg, com.mx, ae, com.br, nl, com.au, com.tr, sa, se, pl and others), `hl` for host language, `gl` for country boosting, `uule` for a encoded region, and `start` for the result offset.

There is also an async version at `async.scraperapi.com/structured/google/search`. It accepts a `queries` array for batch submission and a webhook callback, which is the right shape for scheduled rank checks: you fire the batch, the results come back to your endpoint, you write them to your own history table.

Two gaps to know about before you build on it. The parameter list I checked has **no device switch**, so desktop and mobile rankings are not selectable the way they are on dedicated rank tracking APIs. And the response schema does not document an AI Overview block, so if AI Overview visibility is part of your reporting you will be parsing that out of the returned content yourself or confirming current support with the vendor.

ScraperAPI also does not store your ranking history. It returns the current result for a query. Scheduling, deduplication, position diffing and alerts are your code. That is normal for a SERP API and it is the reason rank tracker APIs and SERP APIs get compared to each other even though they do different jobs.

👉 [Start with the free credits and test the Google SERP endpoint](https://www.scraperapi.com/?fp_ref=coupons)

## Where ScraperAPI stops being the right answer

If your product is a rank tracker, your dominant cost is Google requests and your margin lives in the per-request price. A general scraping API that charges 25 credits a Google search is not built to win that comparison, and pretending otherwise wastes your evaluation time.

The same applies if you need any of these without writing them yourself: pixel-level position data, AI Overview extraction as a first-class field, per-keyword check scheduling, stored ranking history with diffs, or a device parameter for mobile versus desktop. Those exist on dedicated rank tracking APIs and you will end up building them on top of a raw SERP feed.

Where ScraperAPI holds up: your stack already calls it for other targets, so SERP checks become one more endpoint on an existing key and invoice. You need Google results from a specific country domain with a specific `uule`, paired with scraping the pages that rank, and you would rather not manage two vendors and two billing relationships. Or your volumes are modest enough that the 25-credit multiplier is a rounding error next to engineering time.

## How the alternatives price the same job (information only)

| Provider | Model | Published entry price | Depth and targeting notes |
| --- | --- | --- | --- |
| DataForSEO | Pay per SERP | $0.60 per 1,000 SERPs, standard queue | Depth up to 700 results, billed per 10; location code, name or coordinates; desktop and mobile with OS selection |
| SerpApi | Pay per search | $25/month for 1,000 searches, 250 free | City, coordinates or `uule`; desktop, tablet, mobile; AI Overview block returned |
| SE Ranking | Usage-based credits | $50 per 250K credits | Daily checks, full top 100, 35+ SERP features including AI Overview; 5 requests per second per key |
| Ahrefs Rank Tracker | Subscription | $129/month Lite, 750 tracked keywords | Weekly updates; country, state or city; desktop and mobile |
| Semrush Position Tracking | Subscription plus API units | $549/month Advanced for API access | Daily, top 100, postal-code level, filter by SERP feature |

Each of those carries its own gaps. DataForSEO's queue timing is documented inconsistently across its own pages, so treat any single turnaround figure as indicative. Ahrefs updates weekly by default, which is too slow for day-trading on rankings. Semrush charges API units on top of the subscription for creating campaigns and pulling reports.

## Which one to pick

Strip it down to your actual workload.

- **You already scrape through ScraperAPI** and want Google SERP JSON on the same key and invoice: stay. Test the endpoint with the free credits, confirm the location and `uule` behaviour for your target markets, then size the plan from the credit math above.
- **Your only workload is daily rank checks at volume**: a per-SERP provider at roughly $0.60 to $2 per 1,000 SERPs will beat a 25-credit-per-search model by a wide margin. This is arithmetic, not a knock on the product.
- **You need stored history, scheduled checks, pixel positions or AI Overview reporting out of the box**: buy a rank tracker API and skip the build.
- **You need rankings plus the pages behind them in one pipeline**: a scraping API with structured SERP output removes a vendor, which is often worth more than the per-request difference.

Before you enter card details, confirm four things with whichever vendor you choose: whether failed or blocked requests are billed, what one top-100 check costs end to end at your required depth, whether mobile counts as a second check, and what happens at the credit limit (refused request, auto-upgrade, or pay-as-you-go). Those four answers change the monthly total more than the headline price does.

If ScraperAPI is the one, 👉 [open the pricing page and pick the plan that matches your credit math](https://www.scraperapi.com/pricing?fp_ref=coupons), then run a single day of real volume on the free tier before you commit to a year.
