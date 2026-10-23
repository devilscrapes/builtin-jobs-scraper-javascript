<p align="center"><a href="https://apify.com/DevilScrapes/builtin-jobs-scraper"><img src="assets/banner.svg" alt="BuiltIn Jobs Scraper — JavaScript example" width="100%"></a></p>

# BuiltIn Jobs Scraper for JavaScript and Node.js: tech listings with salary fields

<p align="center">
<a href="https://apify.com/DevilScrapes/builtin-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/builtin-jobs-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/builtin-jobs-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Built In (builtin.com) is where many tech companies and startups advertise roles, and its cards carry more structure than most boards: pay band, level, remote or hybrid status, categories and skills. This repository wires the **[BuiltIn Jobs Scraper](https://apify.com/DevilScrapes/builtin-jobs-scraper)** into Node.js with the official `apify-client`. You send a keyword, a city or a category; the Actor walks the result pages on Apify's infrastructure; your code receives objects with integer salary bounds and normalized work mode, ready for a database insert or an API response. The sample is a single ES module that also writes `results.json` and `results.csv`, but the four-statement core is all a backend needs.

> **Actor:** [BuiltIn Jobs Scraper](https://apify.com/DevilScrapes/builtin-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [builtin-jobs-scraper-python](https://github.com/devilscrapes/builtin-jobs-scraper-python)

## 🔥 What you get

- **Three filters that compose**: a keyword, a city and a category can be combined in one run. Free text such as `Austin, TX` or `Data Science` is turned into the slug form builtin.com uses, so you do not have to look it up.
- **Salary as numbers, not just text**: a band like `142K-204K Annually` becomes `salary_min` 142000 and `salary_max` 204000, while `salary_raw` keeps the original wording including the pay period.
- **Multi-city cards unfolded**: when a card only says `3 Locations`, the `locations` array lists each city from the card's tooltip instead of the count.
- **A timestamp you can sort**: relative labels such as `5 Hours Ago` or `Reposted Yesterday` are converted to an ISO-8601 estimate at run time, with the raw label kept beside it.
- **Work mode in a fixed vocabulary**: Remote, Hybrid, Onsite, or `Not specified` when the card is ambiguous, so filtering never depends on how the site phrased it.
- **Clean stopping and no duplicates**: paging ends at your limit, on an empty page, or when a page repeats the previous one, and a job seen twice in the run is kept once.
- **A visible outcome**: a search that matches nothing finishes with a status message saying so, and pages that keep failing after retries make the run fail instead of returning a quietly short dataset.

## 🎯 Use cases

### Feed a niche job board

A board for, say, remote data roles or Austin startups can refresh its inventory from a scheduled worker. Upsert on `job_id`, store `url` as the outbound link, and use `work_mode`, `seniority` and `categories` as ready-made facets.

### Send salary-filtered alerts

Run a search every morning, keep listings whose `salary_min` clears a threshold, and post them to a webhook in Slack, Discord or your own app. Integer fields make the comparison a single expression, with no regex on currency strings.

### Power a compensation widget

Expose an endpoint that returns the median band for a role and city. A serverless function can trigger a run, aggregate `salary_min` and `salary_max`, and cache the answer for a day.

### Spot accounts that are hiring

HR-tech and developer-tool vendors can group listings by `company_name` and push the counts into a CRM, so sales sees which prospects are opening engineering seats right now. `company_url` gives a stable key for matching.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/builtin-jobs-scraper-javascript.git
cd builtin-jobs-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [BuiltIn Jobs Scraper](https://apify.com/DevilScrapes/builtin-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Take away argument parsing and file output and this is what remains. `call()` waits for the run; `listItems()` fetches the dataset rows into memory.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/builtin-jobs-scraper').call({"search": "engineer", "maxResults": 10, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` reads `input.json` and passes it through untouched. The default asks for ten listings that match `engineer`, small enough to see the shape of the data at almost no cost. Add `"city": "New York"` or `"category": "Data Science"` to focus it; plain wording is converted for you, and `maxResults` accepts up to 500.

```json
{
  "search": "engineer",
  "maxResults": 10,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `search` | string | no | `"engineer"` | — |
| `city` | string | no | — | — |
| `category` | string | no | — | — |
| `maxResults` | integer | no | `100` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Every object stands for one listing card. Array fields (`locations`, `categories`, `top_skills`) are always arrays, possibly empty, so you can iterate without guards. `salary_min` and `salary_max` are integers or `null`, and `work_mode` is always one of four fixed strings.

| Field | Type | Always present |
|---|---|---|
| `job_id` | string | yes |
| `title` | string | yes |
| `url` | string | yes |
| `company_name` | string | yes |
| `company_url` | string or null | no |
| `company_logo_url` | string or null | no |
| `location` | string | yes |
| `locations` | array | yes |
| `work_mode` | string | yes |
| `posted_text` | string | yes |
| `posted_at_estimate` | string or null | no |
| `salary_raw` | string or null | no |
| `salary_min` | integer or null | no |
| `salary_max` | integer or null | no |
| `seniority` | string or null | no |
| `categories` | array | yes |
| `top_skills` | array | yes |
| `description_snippet` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| result | $0.0015 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/builtin-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This script uses only Node.js built-ins. It filters remote listings above a salary floor from an environment variable, tallies the most requested skills, and writes a `digest.json` body you can POST to any webhook.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const jobs = JSON.parse(await readFile('results.json', 'utf8'));
const floor = Number(process.env.SALARY_FLOOR ?? 150000);

const matches = jobs
    .filter((job) => job.work_mode === 'Remote' && (job.salary_min ?? 0) >= floor)
    .sort((a, b) => (b.salary_max ?? 0) - (a.salary_max ?? 0));

const skillCounts = new Map();
for (const job of jobs) {
    for (const skill of job.top_skills) skillCounts.set(skill, (skillCounts.get(skill) ?? 0) + 1);
}

const digest = {
    text: `${matches.length} remote roles start at $${floor.toLocaleString('en-US')} or more`,
    roles: matches.map(({ title, company_name, salary_raw, url }) => ({ title, company_name, salary_raw, url })),
    topSkills: [...skillCounts].sort((a, b) => b[1] - a[1]).slice(0, 10),
};
await writeFile('digest.json', JSON.stringify(digest, null, 2));
console.log(digest.text);
```

## ❓ FAQ

### Can I start a run from a webhook handler without blocking it?

Yes. Swap `call()` for `start()`, respond right away, and register an Apify webhook on run success that hits a second endpoint. That endpoint reads `defaultDatasetId` from the payload and pulls the items.

### What if my city or category does not exist on Built In?

The run still succeeds, with zero rows and a status message saying nothing matched. A misspelt filter therefore shows up as an empty result rather than an exception, so check the count in your code.

### Can I use it from TypeScript?

Yes. The client package includes its own types. Describe the listing fields in an interface and cast `items` to it to get autocompletion on `salary_min`, `top_skills` and the rest.

### How many listings can one run return?

Up to 500 through `maxResults`. Built In shows about 25 cards per page, so a full run touches roughly twenty pages and stops earlier if the results run out.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/builtin-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [BuiltIn Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/builtin-jobs-scraper)
- [Python version of this example](https://github.com/devilscrapes/builtin-jobs-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Dice Tech Jobs Scraper](https://apify.com/DevilScrapes/dice-tech-jobs-scraper) — another tech-focused job board, heavier on contract and enterprise roles.
- [Wellfound Startup Jobs Scraper](https://apify.com/DevilScrapes/wellfound-jobs-scraper) — startup roles with funding-stage context.
- [RemoteOK Jobs Scraper](https://apify.com/DevilScrapes/remoteok-jobs-scraper) — remote-only tech postings from a different source.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/builtin-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Built In (builtin.com).
