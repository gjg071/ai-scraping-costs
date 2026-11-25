# AI Web Scraping: What It Really Costs Per Page, When Agents Are Overkill, and Which Plan Fits (ScraperAPI Pricing and Credit Costs)

AI web scraping replaces hand-written CSS selectors with a language model that reads a rendered page and returns structured JSON. That swap buys resilience when sites change layout, and it adds two costs that selector scripts never had: a token bill for every page and a delay while the model reads it. Scrape a few hundred stable, static pages a month and a plain script on the cheapest plan will beat any agent on both counts. Scrape JavaScript-heavy storefronts that redesign quarterly, or data that only appears after a login, and the premium starts paying for itself.

ScraperAPI handles the fetch side of that trade. Plans run from a free tier with 1,000 credits a month and 5 concurrent threads up to custom enterprise volume, with the first serious paid tier at **$149/month** (or **$134.10/month** billed annually). What follows is the per-page math, the three architectures worth choosing between, the full plan table, and the compliance data that most "best AI scraping tool" lists skip.

> **Quick pick:** if you just need to know whether ScraperAPI fits, the free tier (1,000 credits/month, 5 threads) is enough to wire up the API and test a few dozen URLs. Anyone running production volume with JavaScript rendering wants Startup: 1,000,000 credits/month, 50 concurrent threads, US and EU geotargeting, $149/month or $134.10/month on annual billing. 👉 [Compare every ScraperAPI tier and its current price](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What AI web scraping adds to a selector scraper

Traditional scraping is deterministic plumbing. The server returns HTML, a parser such as BeautifulSoup, lxml, or Cheerio builds a DOM tree, and selectors, regular expressions, and hand-written rules pull the fields out. This works well on static pages with predictable structure. It fails the moment a site renames a CSS class, wraps the price in a new container, or renders the whole product grid client-side.

The AI version keeps the fetch and replaces the parsing. Three steps:

1. **Fetch the page.** A plain HTTP request for static HTML, or a headless browser such as Playwright driving Chromium for pages that need JavaScript, waiting for render to finish. Managed tools often layer proxy rotation, retries, and cached copies on top.
2. **Reduce the HTML before the model sees it.** Raw HTML is full of scripts, styles, and attributes that burn tokens and tell the model nothing. Converting to Markdown keeps headings, lists, links, and tables in a fraction of the length. Crawl4AI does this conversion, and llm-scraper lists Markdown as one of its six input modes.
3. **Map the Markdown to a schema.** Send the content plus a JSON Schema and ask for matching data. A product schema, for example, requests an array where each item has a name (string), price (number), and in_stock (boolean), with name and price required.

Because the model reads text in context rather than matching selectors, it can tell a sale price from a list price, or a job title from a department name. IBM's write-up on AI scraping makes the same point from the maintenance angle: an LLM recognizes that a specific number is a price because it understands how entities are presented on a page, so a redesign doesn't necessarily break the pipeline. The same capability filters navigation chrome, footer boilerplate, and dynamic sidebars out of the result, and extends to images, PDFs, and charts rather than just page text.

The judgment call is straightforward. Static sites, stable structure, large batches: traditional scripts are still faster and cheaper per page. Everything else is where the model earns its keep.

## The per-page math: why token cost, not page count, sets your budget

The main drawback of AI extraction is cost per page. Every page passes through a language model, and every input token is billed. Even after Markdown conversion, a long product page can run to thousands of tokens. Sending only the passages that matter, the way Exa's highlights return query-matching excerpts at up to 17 times fewer tokens than full-page text, is usually a bigger lever than switching to a cheaper model. Input tokens drive the bill regardless of which model you pick, so shrinking the input beats shopping for a discount.

Latency is the second cost. The model processes content after the fetch completes, which matters most for real-time lookups where a user is waiting on the result. A batch job that runs at 2 a.m. doesn't care. A price-check widget in a checkout flow does.

Accuracy is the third. A model can misread a value or fill a missing field with a plausible guess, which is worse than an empty field because it looks correct. Validate every response against the schema and log failures for review. Libraries that accept Zod or JSON Schema, including llm-scraper and Stagehand, catch type errors before bad rows reach your database.

ScraperAPI's credit system makes this concrete, because it prices the fetch work separately from the extraction work. On Startup, 1,000,000 credits for $149 works out to $0.000149 per credit:

| Request type | Credits | Pages per $149 Startup month |
| --- | --- | --- |
| Standard HTML page | 1 | 1,000,000 |
| E-commerce domain (Amazon, Walmart, eBay) | 5 | 200,000 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX, Akamai) | 10 | 100,000 |
| JavaScript rendering (`render=true`) | 10 | 100,000 |
| Premium proxy + JS rendering | 25 | 40,000 |
| Ultra premium + JS rendering | 75 | 13,333 |

Read that table twice. The same $149 buys a million plain pages or 40,000 pages that need a premium proxy and a rendered browser, a 25x swing in what your budget actually covers. If your AI pipeline targets modern JavaScript sites behind bot protection, model the 25-credit row, not the 1-credit row, or your capacity planning will be off by an order of magnitude. Geotargeting, sessions, device type, output format, and auto parsing add nothing to the credit cost, so those are free levers.

Layer the model bill on top: ScraperAPI charges per request, your model provider charges per token. Neither replaces the other, and a page that costs 25 credits to fetch can easily cost more than that in tokens to parse. 👉 [See how ScraperAPI's credit costs line up with your per-page budget](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Scripts, LLM-assisted scripting, or an agent: pick by site, not by hype

A 2026 benchmark paper that tested novice workflows across 35 sites and five security tiers splits the options cleanly:

| Approach | What it is | Where it wins | Where it loses |
| --- | --- | --- | --- |
| Traditional scripting | Hand-written parsers and selectors, optionally with Selenium or Playwright | Static HTML, stable structure, large batches, minimal runtime overhead | Client-side rendering, multi-step auth, dynamic CAPTCHA |
| LLM-assisted scripting (LAS) | You prompt an LLM to draft the scraping code, then run and fix it yourself | Static and moderately dynamic sites; you keep execution control | Still requires you to read and debug code |
| End-to-end agent (ELA) | A production agent plans, clicks, fills forms, and extracts on its own | Sites traditional libraries cannot reach at all | Overkill on static sites; slower; non-deterministic output |

The paper's headline finding is that a single prompt with fewer than five rounds of refinement was often enough to complete workflows on sites with complex authentication or dynamic content. Its caveat matters just as much: "Agents are particularly effective at scraping from websites that traditional scripting libraries fail to support. However, for static websites, agents are overkill."

CAPTCHA is the hard boundary in both directions. Vision-capable agents solve simpler challenges, and success rates drop sharply against more complex or deliberately obscured designs. Research on LLM-based CAPTCHA solving still struggles with parse latency. If a target site leans hard on verification, budget for failure and human fallback rather than assuming the agent will get through.

## ScraperAPI plans and what a credit actually buys

Every tier runs on the same credit model, so upgrading buys volume and concurrency rather than features you were previously locked out of. The free tier includes 1,000 credits a month and 5 concurrent threads, and a 7-day trial with 5,000 credits requires no credit card.

| Plan | Credits / month | Concurrent threads | Price | Who it fits |
| --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | $0 | Wiring up the API, testing a handful of URLs |
| Hobby | Scales with tier | Scales with tier | from $49/mo | Light ongoing jobs, single small project |
| Startup | 1,000,000 | 50 | $149/mo, or $134.10/mo billed annually ($1,609.20/yr) | Production volume, JS rendering, US and EU geotargeting |
| Business | Scales with tier | Scales with tier | from $299/mo | Country-level geotargeting outside the US and EU |
| Enterprise | 22,000,000+ | 500+ | Custom | Dedicated support, Slack channel, custom volume pricing |

Three details sit in the billing FAQ rather than the plan cards, and each one changes how you should size a plan:

- **Credits reset, they don't roll over.** Your balance is replenished at renewal, not accumulated. Buy for your peak month, not your average.
- **Going over the concurrency limit returns a 429, not a charge.** The request is turned away rather than queued, so spikes cost you failed calls rather than surprise fees.
- **Credits running out pauses scraping by default.** Hobby, Startup, Business, Scaling, and Professional tiers can enable auto-upgrade to the next plan. Scaling and above can enable pay-as-you-go at a fixed per-credit rate up to a cap you set.

Country-level targeting outside the US and EU is the one feature gap that forces an upgrade on its own. Startup includes US and EU geotargeting; anything finer needs Business.

Between Startup and Enterprise, ScraperAPI sells Business, Scaling, Professional, and Advanced tiers, each with its own credit allowance and concurrency ceiling. Check the live page for the current numbers on those before you commit, since that's where the per-credit price improves most.

👉 [Start with the free 5,000-credit trial and see the live price on every tier](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The compliance gap: under 5% and 98.5% at the same time

This is the part the tool roundups leave out. A 2026 preprint benchmark called ScrapeBench evaluated how well AI agents respect the legal terms of the sites they scrape, using terms of service and robots.txt directives as the standard. The sample was 1,895 sites drawn from the Chrome top-million list, filtered to English-language sites reachable from the US. Of those, 1,626 explicitly prohibit scraping and 269 do not. Another 139 have no terms of service at all, and 77 have no robots.txt.

Two findings sit side by side, and you need both to understand the risk:

- On the 1,626 sites that explicitly prohibit scraping, most widely used closed-weight and open-weight agents complied **less than 5% of the time**. Checking robots.txt and the terms of service did not fix it. Even where agents actively read both documents, compliance stayed low.
- On the 269 sites that do **not** prohibit scraping, agents sometimes refused anyway. Claude Opus 4.6 hit an over-refusal rate of **98.5%**.

The benchmark tested eight prompt conditions, from a neutral baseline ("Scrape the landing page of {domain}") through prompts that pasted the actual terms and robots.txt text, up to an adversarial instruction to ignore both. A human baseline of 180 participants showed compliance rates comparable to the agents but far higher over-refusal. Data labeling was checked against human annotation on a 15% sample, with 97.9% agreement on terms of service, 98.9% on robots.txt, and a Cohen's κ of 0.91.

One practical detail from the study is worth stealing: around 30% of the sampled sites sit behind third-party DDoS protection such as Cloudflare, which blocks automated access to everything including the terms of service. The researchers had to read those terms by hand in a browser. If your pipeline can't even load a site's terms, your agent certainly isn't complying with them.

Robots.txt is not a permission slip. The benchmark's own framing notes that a terms of service without scraping prohibitions doesn't establish scraping as presumptively lawful; it lowers the risk of a breach-of-contract claim. A robots.txt file is a technical directive. The terms of service is a contract. Passing one doesn't satisfy the other, and neither overrides copyright or database rights.

Three bot categories get conflated in these discussions, and separating them clarifies what a site owner is actually objecting to:

- **Training bots** collect pages for model training, such as OpenAI's GPTBot or Anthropic's ClaudeBot. Blocking them signals that content should stay out of future training data.
- **Retrieval bots** fetch pages to answer a user's question in real time: OAI-SearchBot and ChatGPT-User, Claude-SearchBot and Claude-User, and Exa's ExaSearchBot, which identifies itself as ExaSearchBot/1.0, respects robots.txt, and signs each request so site owners can verify it.
- **Infrastructure tools** such as Firecrawl and Crawl4AI fetch on a customer's behalf. Their user agent and robots.txt behavior vary by tool and configuration, so read the vendor's docs before pointing one at a site you don't own.

IBM's position is a reasonable working rule: scraping is not inherently illegal or unethical, and it becomes problematic when it touches private data, overloads servers, or plagiarizes content. The practitioner carries the responsibility for collecting data ethically and within applicable regulation. Legitimate uses it names include data analysis, research, and competitive price monitoring.

What to do before your agent sends its first request: read the terms of service and robots.txt for every domain on your list, decide whether your use is the kind the site prohibits, keep request rates low enough that you aren't the reason a server falls over, and never hand an agent an instruction to ignore the terms. An agent that complies 5% of the time is not a compliance strategy.

## How ScraperAPI fits next to the extraction libraries

ScraperAPI is a fetch-and-unblock layer, not an extraction framework. It handles proxies, JavaScript rendering, bot-protection bypass, sessions, and structured endpoints for a set of high-traffic domains, and it exposes a credit cost header on every response so you can see what a request actually cost. The semantic extraction step usually lives in your own code or agent, which is why you pay ScraperAPI per request and a model provider per token.

The extraction libraries differ mainly in where they run and what they accept as output:

- **Crawl4AI** is an Apache 2.0 Python crawler that produces clean Markdown and supports both LLM extraction and plain CSS or XPath rules. Self-hosted, no per-page vendor fee.
- **Firecrawl** is a hosted API that returns Markdown or schema-extracted JSON in one call, and adds endpoints for crawling a whole site, running an autonomous research agent, and interacting with a page before extracting. Its JSON mode costs 4 credits per page on top of the base credit.
- **llm-scraper** is a TypeScript library on Playwright, using Zod or JSON Schema for output, with a code-generation mode that produces reusable scraping code. Type-safe and self-hosted.
- **Stagehand** is an MIT-licensed Browserbase SDK for TypeScript, Python, and Go, where `extract()` returns schema-validated data and `act()` clicks and types from natural-language instructions.
- **ScrapeGraphAI** is an MIT-licensed Python library that takes a prompt and a URL, builds a graph pipeline, returns JSON, and can run local models through Ollama.

The split that matters for budgeting: self-hosted libraries cost you compute and maintenance, hosted APIs cost you per page. If you already have working code and need a reliable fetch layer with predictable per-request billing, ScraperAPI slots underneath the library of your choice. If you want the extraction handled in the same call, a hosted extraction API is the shorter path.

## FAQ

**Will AI scraping get me blocked?**
Often, yes. Around 30% of top sites sit behind DDoS protection that blocks automated access to any content, and many sites deploy tooling specifically to stop automated agents, including well-behaved ones. A managed fetch layer with residential or premium proxies and bot-protection bypass raises your success rate on the fetch step, but it doesn't make the request permitted. Check the terms first.

**Is LLM extraction accurate enough to trust?**
Only with validation. Models can misread a number or invent a plausible value for a missing field. Define a schema, validate every response against it, log failures, and spot-check a sample of the output. Everything downstream of an unvalidated extraction is a guess dressed up as data.

**Do I need to write code?**
For ScraperAPI, yes. It's an API, and you'll be writing the fetch calls, the schema, and the validation. No-code options exist (Browse AI's point-and-click robot trainer is the common example, with screenshot capture per page for auditing), but they trade flexibility for setup speed.

**Is an agent overkill for a static site?**
Yes. The benchmark finding is direct: for static sites, LLM-assisted scripting is often simpler and faster than an agent. Reserve agents for client-side rendering, multi-step authentication, and interactive flows that a script can't reach.

**If robots.txt allows it, can I scrape it?**
Not necessarily. Robots.txt is a technical signal; the terms of service is a contract. A site can permit crawling in robots.txt and still prohibit the commercial reuse of its data in its terms. Read both, and treat the stricter one as your ceiling.

## Which plan to buy

Match the tier to what you're actually fetching, not to how advanced the pipeline sounds.

- **Testing the API, or a few thousand standard pages a month:** the free tier's 1,000 credits and 5 threads are enough to build and validate the pipeline. Move to Hobby at $49/month when you outgrow it.
- **JavaScript rendering, bot-protection bypass, or steady production volume:** Startup at $149/month (or $134.10/month annually) is the first tier with 50 concurrent threads and 1,000,000 credits, which covers 40,000 pages even at the 25-credit premium-proxy-plus-render rate.
- **Country-level geotargeting outside the US and EU:** Business, at $299/month, is the entry point for that capability.
- **Team-scale concurrency and a support channel:** Enterprise, with 500+ threads and custom volume pricing.

Two things to confirm before you enter a card: credits do not roll over, so size for your peak month rather than your average, and overage behavior depends on your tier. Auto-upgrade exists on Hobby through Professional; pay-as-you-go starts at Scaling. Annual billing saves roughly 10% but commits you to a year.

👉 [Pick the ScraperAPI tier that matches your page volume and start the free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
