---
name: free-tool
description: Finds the free tool a site should build to earn search traffic, such as a calculator, generator, checker or template, using OpenSEO keyword, search result and backlink data, then writes a spec your agent can build from. Use when the user asks what free tool to build, wants engineering-as-marketing ideas, or asks how to get search traffic to a new site.
license: MIT
---

# Free tool

Find the one free tool a site should build for search traffic: a calculator, generator, checker, converter or template its buyers already search for, on a first page it can win. Then spec it so the agent can build it. It works for brand-new sites, because it reads search demand and competitors rather than the site's own rankings. Needs the OpenSEO MCP server connected (https://openseo.so).

For a full plan of competitor keywords, pages and links, use `competitor-strategy` instead.

## Inputs

- The user's site. Call `list_projects` and take the project whose `domain` matches, or `create_project` if there is none. Ask only if you have no website to work from.
- Competitors and past research. Call `get_project_context` for saved competitors and the research log. If an earlier entry looked at free tools, say what it concluded.
- The market. Use the project's default unless the site clearly targets another country.

If OpenSEO isn't connected, say so and stop. Don't estimate search volumes yourself.

## Steps

1. **Learn the buyer's job.** Read the homepage, two or three product or pricing pages, and any free tools page. Write down what the product does, who buys it, and the numbers, documents or checks that buyer handles around it, such as fees, rates, taxes, rankings or invoices. List the free tools the site already has, and leave them out from here on.
2. **See which tools competitors rank with.** Pick two or three competitors that sell the same thing, from the saved ones or, if none fit, from `find_serp_competitors` run on five to ten unbranded searches a buyer makes, with `excludeDomains` set to the site and `resultTypes: ["organic"]`. Skip forums, publishers, directories and video sites. Call `get_domain_keyword_suggestions` for each. It returns a domain's top 100 keywords by traffic, without URLs or intent. Keep the keywords that name a tool, such as calculator, generator, template, checker, test, finder, converter or simulator.
3. **Add your own ideas.** Call `research_keywords` with two to four seeds built from step 1, such as "<job> calculator" or "<document> template". If a seed's own keyword isn't among the rows, there's no measured demand for it.
4. **Shortlist and check page one.** Keep only tools the buyer would use while doing the job the product helps with; competitor lists and research rows are full of others (password generators, mortgage calculators). Pick five to eight by fit, then volume, then difficulty. Call `get_serp_results` for them in batches of four, because eight keywords return about 80,000 characters. Read the rows seen in two or three reads (`seenIn`) as page one and quote `bestPosition` as a site's position. For each, note whether page one shows tools or articles and which domains hold it. Mark it **crowded** when most of those rows are small sites named after the tool, offering the same thing, since a new one would be another copy. Favor results held by a few established sites with a plain tool you could beat on inputs or outputs (bulk, state by state, exportable). Treat a difficulty of 0 as no data, not as easy.
5. **Pick one.** Choose the tool with the best mix of fit to the product, demand, an open first page and a clear way to be better than the top result.
6. **Check the leader's links.** Take the best-placed free tool page on the pick's first page (not an article or a product page) and find its URL by opening that site, because the result's `url` is often a Google redirect. Call `get_backlinks_profile` on it with `scope: "exact_url"`, `mode: "one_per_domain"`, `sortField: "domainRank"`, `sortOrder: "desc"` and `filters: { hideLost: true, hideBroken: true }`. `totalCount` is its referring domains. Note any "best free <tool>" roundups that link to it: they're the first places to ask for a listing once the tool ships.

## Output

A report in this order, with no more than a line or two of prose around each part:

- **Build this:** one line naming the tool, its target keyword, monthly searches and why it fits the product.
- **Candidates:** a table of tool, target keyword, monthly searches, difficulty, who holds page one (from the live check), crowded or open, and competitors ranking with it (from step 2, with their position).
- **Spec:** a page URL the site doesn't use yet and its title, the inputs, the outputs, what beats the current top result, how the tool leads into the product, the keyword variants the page should cover, and the build effort with any data source it needs. If each use calls a paid API, give the cost per use and the rate limit it needs.
- **Links to get:** the leading tool page's referring domains and the roundups or resource pages to ask for a listing.
- **Skip these:** crowded or off-fit tools, with one reason each.

Then ask whether to build the tool now in the user's codebase, or save the candidate keywords with `save_keywords`, tagged `free-tool`. Do neither without a yes.

## Notes

- A run is about 10 tool calls and 300 to 500 OpenSEO credits. Tell the user before going past five competitors or 2,000 credits. If credits run out, report what you have and say what's missing.
- `find_serp_competitors` returns no data for some searches. If it returns nothing usable, call `get_serp_results` for the same searches and count how often each `domain` appears.
- Search volumes, difficulty and positions are estimates for one market. Competitor positions from step 2 come from the provider's last crawl and can differ from the live check. Write `unknown` for any value a tool didn't return, and don't report one check of page one as a fixed ranking.
