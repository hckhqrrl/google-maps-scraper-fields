# Google Maps Business Scraper: Which Fields You Can Get, How Far One Search Reaches, and What It Costs (Plan Table + Credit Math)

A Google Maps business scraper returns public listing data: name, address, phone, website, category, rating, review count, hours, coordinates, photo URLs. Three limits shape every build. There are no email addresses on Maps. Every single search hits a depth cap, so a whole city needs to be tiled rather than paged through. And de-duplication only works on the place ID, never on the business name. ScraperAPI covers the collection step with a documented Maps Search endpoint that returns JSON, and Google requests bill at 25 credits each, which makes its 100,000-credit Hobby plan roughly 4,000 result pages a month.

> **Quick pick:** the 7-day trial hands you 5,000 credits with no card, about 200 Google Maps requests, enough to test two or three query phrasings against one real city. After that, Hobby's 100,000 credits cover 4,000 requests a month; solo lead-gen projects usually stay there, and teams running multi-city grids move up for the thread count.
> 👉 [Start the free trial and check current plan prices](https://scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, and what each credit allowance buys in Maps requests

ScraperAPI bills monthly or annually, and the live pricing page is the authoritative figure for each tier. What decides whether a plan fits your Maps project is the credit allowance, because Google requests are priced at a fixed 25 credits. The last column converts that for you.

| Plan | API credits / month | Google Maps requests at 25 credits | Concurrent threads | Get started |
| --- | --- | --- | --- | --- |
| Free | 1,000 | 40 | 5 | [ Free plan](https://scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 4,000 | 20 | [ Hobby pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 40,000 | 50 | [ Startup pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 120,000 | 100 | [ Business pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200,000 | 200 | [ Scaling pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 420,000 | 300 | [ Professional pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 860,000 | 500 | [ Advanced pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 880,000+ | 500+ | [ Enterprise and custom plans](https://scraperapi.com/pricing?fp_ref=coupons) |

The tiers differ in more than volume:

|  | Hobby / Startup | Business and above |
| --- | --- | --- |
| Geotargeting | US and EU | Country-level |
| Analytics history | Last 30 days | Unlimited |
| Pay-as-you-go overage | Not available | Scaling, Professional, Advanced, Enterprise |
| Auto-upgrade to next tier | Available | Available up to Professional |

Two behaviours matter when you run a large sweep. Hitting your concurrency ceiling returns a 429 rather than an extra charge; the request is turned away, not queued. And if you run out of credits mid-project, Hobby through Professional can auto-upgrade to the next tier, while pay-as-you-go at a fixed per-credit rate is reserved for Scaling and above.

Credit costs beyond the Google rate are worth knowing before you design a pipeline: a standard page is 1 credit, JS rendering adds 10, premium proxies add 10, ultra premium is 30, and bot-protection bypass adds 10. Geotargeting, sessions, device type and output format are free.

## What a Maps scraper can and cannot return

The field list is wider than most buyers expect, and the gaps follow a pattern once you know where Google stores each item.

A well-built scraper returns around 25 fields per place. The core set: `name`, `category` and `categories`, `address` and `fullAddress`, `neighborhood`, `latitude`, `longitude`, `phone`, `timezone`, `rating`, `description`, `hours` as a per-day array with open and close values, `photoCount`, `photoUrl`, `googleMapsUrl`, and `sourceQuery`, the keyword that surfaced the listing. Three of them carry more decision weight than the rest:

- `placeId`, the canonical `ChIJ...` identifier, is the only reliable join key across runs. Names and addresses change; place IDs generally do not.
- `claimed` tells you whether the owner has taken control of the Google Business Profile. Unclaimed profiles are the ones with nobody to ignore your outreach.
- `phoneE164` gives the number with a country code prefix, ready to drop into a dialer without reformatting.

**Email addresses are not on Google Maps at all.** No Maps scraper can read them off a listing, because Google does not publish them. Every tool that advertises email enrichment is doing a second pass over the business's own website, usually the home page plus Contact and About pages. Businesses without a website yield nothing, and one marketplace actor charges extra per place where an email is found. Plan for that as a separate enrichment step with its own hit rate, not as a field you tick on.

## Why one search never returns a whole city

Google caps how deep any single query goes. Vendor documentation disagrees on the number, quoting roughly 120 results per query in one place and about 200 places in another, without either stating how it counts. Treat the exact figure as unverified and the direction as certain: a single broad query tops out, and raising a `maxResults` parameter does not lift the ceiling.

The workaround everyone converges on is tiling. Split the city into cells by neighborhood, ZIP code or a coordinate grid, run the same keyword in each, then merge. One actor exposes this as a grid cell size from 0.5 to 50 km with up to 11 pages per cell; an open-source project ships a grid mode for the same reason.

Two more behaviours will bite you at merge time. Query phrasing changes the result set: `dentist in Austin TX` and `Austin dentist` return materially different lists, so test a couple of phrasings before committing to a large sweep. And the official Google Places API, the compliant route, also charges per request and caps result depth, so tiling is not a quirk of unofficial tools.

## Three ways to build it, and what each one costs you

The build choice decides your maintenance bill more than your hosting bill.

**Browser automation (Playwright, Selenium).** This is the only route that reaches the tabs inside a listing. A Playwright scraper can open Overview, Reviews and About, expand collapsed review text and owner replies, capture Popular Times by hour across the week, and pull photo and video URLs per album category. It is also the slowest and the most brittle. The author of one such project describes it as mimicking a human and time-taking, keeps every selector in a `SELECTORS` dictionary in `config.py` because Google redesigns break them, and pins `playwright==1.40.0` for Python 3.13 compatibility. Known failure modes include browser crashes under load and empty photo albums for some businesses.

**Direct calls to Google's internal endpoint.** A pure HTTP approach posts to the same internal `tbm=map` endpoint the Maps frontend uses, with a `pb` parameter encoding the query and pagination offset, plus a Chrome TLS fingerprint via curl_cffi. It is dramatically faster: one documented comparison puts a page of 20 results at about a second versus roughly 30 for the browser route. The author of that project is explicit that it accesses Google through undocumented internal endpoints, which violates Google's Terms of Service, and publishes it strictly for educational and research purposes.

**A managed scraping API.** You send a query and coordinates, you get structured JSON back, and the proxy rotation, rendering and blocking logic sit with the vendor. You pay per request rather than in engineering hours.

On legality, the two positions in circulation are not the same and should not be merged. The open-source author's stance: no platform officially allows scraping, and automated extraction of Google Maps data is only permitted through the official Places API. A marketplace vendor's stance: scraping publicly available data is broadly lawful in the US and EU, but the answer depends on what you collect and how you use it, with extra obligations under GDPR and CCPA for anything that counts as personal data. Both agree the official Places API is the sanctioned path and that it bills per request with a depth cap.

## Where ScraperAPI fits on that map

ScraperAPI exposes a Google Maps Search structured data endpoint. It takes a query plus latitude and longitude and returns JSON, either synchronously as a GET or asynchronously as a POST that accepts a `jobs` array of multiple query and coordinate pairs with a webhook callback for each. That batch form is what makes a grid sweep practical: one call submits the whole set of cells.

The published response sample shows each place with name, address components as an array, `stars`, `ratings` count, price level, a type array of categories, district, latitude and longitude, a formatted `address_line`, opening hours per weekday, image URLs, and a `url` field holding the business's listed link. A `next_page_url` comes back for pagination, and each additional page is another billed request.

Three practical notes before you commit:

- **Email is not in the response.** Budget the enrichment pass separately.
- **Confirm the identifier you need on your own test query.** The sample ScraperAPI documents does not visibly include a phone number or a place ID, and the sample is truncated, so run one query against a city you know and check the full payload before you architect a CRM join around either field.
- **Costs are visible per request.** Google requests are listed at 25 credits. Every response reports its own cost in the `sa-credit-cost` header, the dashboard's Domain Cost Estimator checks any URL before you send it, and `max_cost` caps a single scrape.

The honest limit: this endpoint returns listing-level data. It is not a review archive and not a photo album crawler. If your project needs full review threads, Popular Times or per-album media, a browser-driven scraper or a specialized actor remains the better tool, and you should expect to pay for that either in credits or in maintenance.

If the field list and the credit math line up with your project, 👉 [review the ScraperAPI plans and credit allowances](https://scraperapi.com/pricing?fp_ref=coupons) before you start a large sweep.

## Who should not buy this

Three cases where the money is better spent elsewhere.

You need verified emails as the primary output. A Maps endpoint gives you websites, and turning websites into emails is a second scraper with its own costs and hit rate. Buy a tool that bundles the enrichment, or accept a lower yield.

You need review text, sentiment or popular-times data. Listing-level fields will not get you there. Browser automation or a purpose-built reviews scraper is the fit.

You need a few hundred rows, once. A one-off manual export, or the official Places API with its introductory credit, likely costs less than any monthly plan.

## FAQ

**Can a Google Maps business scraper get email addresses?** Not from Maps. Google does not publish business emails on listings. Every tool that returns them is fetching the business's own website afterwards, so businesses without a site return nothing.

**How many places can I get from one city?** No single query returns all of them; the cap is real and vendors quote different numbers for it. Split by neighborhood or ZIP code, run the same keyword in each cell, and de-duplicate on place ID.

**Do I need a Google API key?** Not with a managed scraping API. The trade-off is that you inherit the vendor's compliance position instead of Google's terms, so read the section above rather than assuming either route is risk-free.

**What happens when Google blocks the requests?** On a managed API, the vendor absorbs it, and you will see it as failed requests against your credit balance rather than as bans on your own IPs. Self-built scrapers handle it with proxy rotation, per-proxy rate limits, escalating cooldowns and a circuit breaker that pauses everything when block rates spike.

**Is scraping Google Maps legal?** It depends on jurisdiction, what you collect and what you do with it. Business directory data about commercial entities sits in different territory from personal data, which carries GDPR and CCPA obligations. The official Places API is the one route Google sanctions, and it bills per request with a depth cap.

## How to decide

Match the plan to the request count, not to the marketing tier. Running one city a month with a handful of keywords: the free trial first, then Hobby at 4,000 Google requests. Running a multi-city grid with weekly refreshes: Startup at 40,000 requests, or Business if you need country-level geotargeting and unlimited analytics history. Above that, the request math in the table is the whole decision.

Two things to confirm before you pay. Run your actual query phrasing through the endpoint on the trial credits and check that the payload contains every field you plan to store, especially any identifier you intend to join on. Then map your monthly request volume, including pagination, against the credit column, because every extra results page is another 25 credits.

👉 [Compare the ScraperAPI plans against your monthly request volume](https://scraperapi.com/pricing?fp_ref=coupons).
