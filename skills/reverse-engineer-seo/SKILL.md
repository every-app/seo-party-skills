---
name: reverse-engineer-seo
description: Reverse-engineers how any website gets its Google traffic, using OpenSEO ranking and backlink data. Finds the page types that earn its non-brand traffic, the search patterns behind them, and the pages that attract its links, then writes the moves to copy. Use when the user asks how a company does SEO, wants an SEO teardown of a site, or wants to copy what works for a competitor or a site they admire.
license: MIT
---

# Reverse-engineer a site's SEO

Take one website and work out how it earns its Google traffic: which page types bring in searches that aren't for its brand, the search patterns those pages answer, and what attracts its links. Then turn that into moves the user can copy. Needs the OpenSEO MCP server connected (https://openseo.so).

For a plan for the user's own site built from several competitors, use `competitor-strategy` instead.

## Inputs

- The site to study. It doesn't have to be the user's. Ask if none was named.
- The user's own site or business, if they want the moves fitted to it. Without one, write the moves as patterns anyone can use.
- A `projectId` from `list_projects`. Any project works, since it only sets the market. Call `create_project` for the user's own site if there is none.
- The market. Use the project's default unless the user names another country, then pass `locationCode` and `languageCode`.

If OpenSEO isn't connected, say so and stop. Don't estimate these numbers yourself.

## Steps

1. **Size it.** Call `get_domain_overview` for estimated monthly organic traffic and ranking keywords, and `get_backlinks_overview` for referring domains, domain rank, and the 12-month trend.
2. **Split brand from non-brand.** Call `get_domain_keyword_suggestions`, which returns the top 100 keywords by traffic in compact rows. Add up the traffic of the rows that contain the brand or product names, and report it as a share of the 100. Note the search patterns that repeat in the rest, such as "[currency] to [currency]".
3. **Find the pages that earn the rest.** Call `get_ranked_keywords` with `sortBy: "traffic_estimate"`, `maxRank: 20`, `resultTypes: ["organic"]`, `limit: 10`, and `excludeBrandTerms` set to the brand and product names, then again with `offset: 10`. Rows run about 4,000 characters each, and twenty in one call overflow most clients. `totalCount` is every non-brand keyword the site ranks for in the top 20. Read position from `rank_absolute` and traffic from `etv`. Group the rows by URL, then by section: a subdomain, or the first folder that names a page type, skipping country and language folders. In `example.com/us/templates/invoice`, the section is `/us/templates/`.
4. **Measure each section.** For up to five sections, repeat the step 3 call with every parameter the same except `limit: 1`, no `offset`, and `target` set to the section. Pass a subdomain as `blog.example.com` and a folder as a full URL, `https://example.com/us/templates/`; a folder without `https://` is rejected. `totalCount` is the section's keywords, and the row is its top page. Divide by the site's total for the section's share. This shows what's past the first twenty rows without pulling thousands of them.
5. **Count AI Overview citations.** Call `get_ranked_keywords` with `resultTypes: ["ai_overview_reference"]`, `sortBy: "search_volume"`, and `limit: 5`. `totalCount` is how many searches, brand searches included, show the site as a source in Google's AI Overview, and the rows show which pages get cited.
6. **Read the template.** Open the top page of the three sections with the most keywords, using your web tools. Note the search it answers, whether it's written by hand or one template filled from data, its publish date if it shows one, and how it sends the reader to the product. If a page won't load, because it needs JavaScript or blocks automated readers, describe it from the `title` and `description` in its rows and say so.
7. **Find what earns the links.** Call `get_backlinks_profile` with `mode: "one_per_domain"`, `sortField: "domainRank"`, `sortOrder: "desc"`, `pageSize: 50`, and `filters: { hideLost: true, hideBroken: true }`. Set the sort, because the default returns the newest links, not the strongest, and keep the page at 50, because 100 rows can overflow. Skip URL shorteners, redirects, search result pages, app store listings and user profiles. Many of the rest point at a homepage and few other URLs repeat, so group `urlTo` by subdomain, section or page type. A type other than the homepage that keeps attracting strong links is a link magnet, such as articles cited for one figure, a free tool, or a yearly report.
8. **Write the moves.** Each move is one thing the site does that the numbers show working, sized down for a smaller site: the page type, the search pattern, and a first page to build. Say what won't copy, such as brand searches and years of links.

## Output

A report in this order, with no more than a line or two of prose around each table:

- **The playbook:** three lines on where the traffic comes from, what earns it, and the first move to copy.
- **Scorecard:** a table of estimated monthly organic traffic, ranking keywords, brand share of the top 100 keywords' traffic, non-brand keywords in the top 20, AI Overview citations, referring domains with their 12-month change, and domain rank.
- **Traffic engines:** a table of section, page type and how it's made (written by hand, or one template filled from data), non-brand keywords in the top 20, share of the site's total, top page, and its top keyword, with the rest of the site as the last row.
- **Top pages:** a table of 10 to 15 URLs with section, top keyword, monthly searches, position, and estimated monthly traffic added up across the page's rows.
- **Search patterns:** a table of pattern, example keywords, their monthly searches, and the section that answers them.
- **Link magnets:** a table of page type, what it is, how many of the strongest linking domains point at it, and examples.
- **Moves to copy:** three to five, best first, each with the evidence, how to do it on a smaller site, and a first page to build.
- **What won't copy:** the advantages that come from the brand or its age, and why.

Then offer to save the keywords behind the moves with `save_keywords`, tagged `reverse-engineer-seo`. Ask before saving.

## Notes

- A run is about 12 OpenSEO calls and 250 to 300 credits. Tell the user before studying more than three sites in one go.
- Traffic and rankings are estimates for one market, so label them that way. Write `unknown` for any metric a tool didn't return.
- A section's share is a share of ranking keywords, not of visits. Don't report it as a share of traffic.
- Search volume is a 12-month average. Before quoting a keyword whose volume looks too big for the site, check `monthly_searches` for a one-month spike.
