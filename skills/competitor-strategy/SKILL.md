---
name: competitor-strategy
description: Finds a site's competitors, then turns their top pages and backlinks into keywords to track, pages to create, and link themes, using OpenSEO data. Use when the user starts a new SEO project, or asks what to target based on what their competitors are doing.
license: MIT
---

# Competitor strategy

Study where a few competitors get their Google traffic and links, then turn that into a starting plan for the user's site: keywords to track, pages to create, and the links worth pursuing. It's built for new projects, where the website may be all you have, and it works on established sites too. Needs the OpenSEO MCP server connected (https://openseo.so).

For a full list of the keywords competitors rank for and the site doesn't, use `keyword-gap` instead.

## Inputs

- The user's site. Call `list_projects` to find its `projectId`, or `create_project` if it has no project yet. Ask only if you have no website to work from.
- Competitors. Call `get_project_context` and use any saved there. If there are none, find them in step 2. Don't ask the user for them.
- The market. Use the project's default market unless the site clearly targets another country, then pass `locationCode` and `languageCode`.

If OpenSEO isn't connected, say so and stop, or ask the user to paste exports of their competitors' top pages and backlinks. Don't estimate these numbers yourself.

## Steps

1. **Learn the business.** Read the project context, then the homepage and two to four product, service, pricing, or location pages with your web tools. Note what the site sells, who buys it, and whether customers come from one area. If they do, treat the business as local: get a `locationName` from `search_serp_locations` and pass it to `get_serp_results` and `research_keywords`. If you can't tell what the business does, ask for one sentence.
2. **Pick three or four competitors.** Use saved competitors first. Otherwise write five to ten searches a customer would make when ready to buy, with no brand names, and call `find_serp_competitors` with `excludeDomains` set to the site. Some searches have no data, so if the list comes back empty, call `get_serp_results` with `depth: 10` for the same searches and count how often each `domain` appears. For a local business, skip `find_serp_competitors`, which is country-level. Call `search_local_businesses` with `near` set to the business's city and radius, `categories` from `list_business_categories`, and `sortBy: "reviews"`, and take each row's `domain`. Local `get_serp_results` is a weaker source here: Maps rows have no `domain`, and organic rows can include other cities. Keep businesses selling the same thing to the same buyers, and drop publishers, directories, marketplaces, and forums.
3. **Size them.** Call `get_domain_overview` for the site and each competitor to get estimated organic traffic and keyword counts. Its backlink fields come back empty, so those come from step 6. Little data is normal for a new site or a small local business; say so rather than guessing. Swap out any competitor with almost no organic footprint. For a new or small site, keep the market leader and make sure one competitor is close to the site's size: the leader shows the demand, and the smaller site shows what's winnable.
4. **Find the pages earning traffic.** For each competitor, call `get_ranked_keywords` with `sortBy: "traffic_estimate"`, `maxRank: 20`, `resultTypes: ["organic"]`, `limit: 20`, and `excludeBrandTerms` set to its brand and product names. Rows run about 4,000 characters each, so keep the limit low. Group the rows by URL, add up the traffic, and label each page: product or service, comparison, free tool or template, guide, or location. Read the strongest page of each type before you describe it, and note which types repeat across competitors.
5. **Choose keywords to track.** Call `get_domain_keyword_suggestions` for each competitor, which returns its top 100 keywords by traffic in compact rows. Merge them and drop brand terms, off-topic terms, and anything past position 30. Put keywords two or more competitors rank for first, then weigh business fit, difficulty against the site's strength, and volume. For a new site, a low-difficulty term with a small site in the results beats a high-volume term held by giants. Treat a difficulty of 0 as no data, not as easy; it's the norm on local terms. Call `get_keyword_metrics` for intent on the shortlist, and `get_serp_results` with `depth: 10` on the top ten to confirm the intent and the format that wins. Trust `domain` and `title` in those rows, since `url` is often a Google redirect. The domains listed can change completely between checks, so don't report one check as who ranks. Re-check the query behind the first page at `depth: 20`, since a repeat at the same depth returns a 12-hour cache, and say if the results differ. For a local business, get local volume from `research_keywords` with `locationName`, which adds about 116 credits per seed. Mark what the site already covers with `get_search_console_performance` if connected, otherwise `get_domain_keyword_suggestions` on the site, plus `list_saved_keywords` and `get_rank_tracker`.
6. **Read the backlinks.** For each competitor, and for the site if it has links, call `get_backlinks_profile` with `mode: "one_per_domain"`, `sortField: "domainRank"`, `sortOrder: "desc"`, `pageSize: 100`, and `filters: { hideLost: true, hideBroken: true }`. Set the sort, because the default returns the newest links, not the strongest. `totalCount` is the number of live, non-spam referring domains. Then:
   - Remove links that domain rank flatters: URL shorteners, search result pages, free subdomains on hosts and site builders (such as `*.hostingersite.com` or `*.squarespace.com`), sites anyone can post on (GitHub, Medium, forums), the competitor's own app store and job listings, and affiliate placements (links carrying `ref` or `ga` parameters).
   - Find the common links. A domain linking to two or more competitors is a link the category tends to earn. Remove any the site already has.
   - Group what's left into themes by the kind of page linking: best-of lists, guides from vendors in adjacent categories, press citing original data, directories and marketplaces, associations and chambers, partner or integration pages, podcasts, and local sponsors.
   - Check which competitor pages attract the links. A study cited by the press or a free tool linked from tutorials is a page idea as well as a link source.
   - Open each linking page you plan to name, and confirm the link is live and sits in real content. Count a theme's competitors across all the rows, not only the links you opened.
7. **Build the plan.** Tie each keyword to a page, so the tracking list and the build list agree. Order pages by business value, then by how winnable the results look. Recommend a stronger angle than the competitor's page, not a copy of it. Start with link themes the site can act on now: for a new site that usually means listings and directories, because editorial links need a page worth citing.

## Output

A report in this order, with no more than a line or two of prose around each table:

- **The plan:** three lines naming the first page to build, the first link theme to work, and what the site's current size means for how quickly this can pay off.
- **Competitors:** a table of domain, saved or found, estimated organic traffic, ranking keywords, and referring domains, with the site in the first row.
- **Top competitor pages:** a table of URL, page type, estimated traffic from the rows you pulled, and top keyword.
- **Keywords to track:** a table of 15 to 30 keywords with monthly searches, difficulty, intent, which competitors rank, and the page each belongs to (an existing URL or "to create").
- **Pages to create:** three to seven, best first, each with its target keyword, the competitor pages ranking now, the format that wins, and the angle to take.
- **Backlink themes:** a table of theme, example domains, how many competitors have links from it, and how the site could earn one.
- **Most reputable links:** a table of 10 to 20 links with linking domain, domain rank, which competitors it links to, linking page, linked page, and dofollow or nofollow.
- **What to skip:** the keywords, pages, and links not worth chasing yet, and why.

Name any competitors you found at the top, so the user can correct them. Then offer to save them with `update_project_context` (`addCompetitors`), save the keywords with `save_keywords`, tagged `competitor-strategy`, or track them with `create_rank_tracker` and `add_rank_tracking_keywords`. Ask before any of these. Scheduled tracking keeps spending credits, so show the `estimate_rank_tracker_cost` figure first.

## Notes

- A run with three or four competitors is about 30 tool calls and 500 to 1,000 OpenSEO credits. Tell the user before going past five competitors or 2,000 credits. If credits run out, report what you have and say what's missing.
- If backlink data is unavailable, finish the rest of the report and say so.
- Don't use `research_project_website` or `save_project_website_setup` to find competitors. They're paid setup that needs the user's approval, and saving can start recurring AI tracking.
- Traffic and rankings are estimates for one market, so label them that way. Write `unknown` for any metric a tool didn't return.
- For a local business, keep organic results separate from Maps results.
