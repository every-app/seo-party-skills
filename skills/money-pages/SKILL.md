---
name: money-pages
description: Finds the pages a SaaS needs for the searches buyers make right before they pay, using OpenSEO keyword and search result data. Covers "<competitor> alternatives", "X vs Y", competitor pricing and "<category> for <who>" pages, puts a dollar value on each from what advertisers pay per click, checks who holds page one, and lists what to build first. Use when the user runs a SaaS or software product and asks how to get sign-ups from Google, which comparison or alternatives pages to build, or for a SaaS SEO plan.
license: MIT
---

# Money pages

A SaaS earns its best search sign-ups from four page types: "<competitor> alternatives", "<you> vs <competitor>", "<competitor> pricing" and "<category> for <who>", where <who> is an industry, team, business size or device. This skill finds the searches behind each for one product, prices them, checks who holds page one, and lists the pages to build. Needs the OpenSEO MCP server connected (https://openseo.so).

For keywords competitors rank for across their whole site, use `keyword-gap` instead.

## Inputs

- The product's website. Call `list_projects` and take the project whose `domain` matches, or `create_project` if there is none.
- Competitors, only if the product is too new to show up in "vs" searches (step 2). Take three that sell the same kind of product from `get_project_context`, or ask for three.
- The market. Use the project's default unless the user names another country.

If OpenSEO isn't connected, say so and stop. Don't estimate volumes or prices yourself.

## Steps

1. **Read the product.** Read the homepage and pricing page. Find the sitemap in `robots.txt`, since it's often not at `/sitemap.xml`, and list the pages whose path contains `compare`, `vs`, `versus`, `alternative`, `pricing` or `solutions`, blog posts included. Sites name these folders differently (`/compare-against/`, `/blog/x-vs-y`). Write down the category in the buyer's words ("scheduling software", not "scheduling automation platform") and who the site says it's for.
2. **Find who buyers compare it with.** Call `research_keywords` with the wildcard seeds `<brand> vs *` and `* vs <brand>`. Each pair appears in only one word order. The competitors are the five names with the most comparison searches that sell the same kind of product; skip tools the product works with rather than replaces, like Google Calendar for a booking app. If buyers compare it in two categories (notes and projects), keep both. If both seeds come back nearly empty, the product is too small to be compared yet: take competitors from the inputs and run the same two seeds on the best known one. Its "X vs Y" searches between two competitors are the ones to win.
3. **Measure the other page types.** Call `research_keywords` with `<category> for *` for each category. Use wildcards: a plain seed drifts ("asana alternatives" returned yoga poses), and even a wildcard returns other meanings of the category, so drop industries, devices and uses the product doesn't serve. Then call `get_keyword_metrics` with `includeMonthlyTrends: false` on `<name> alternatives`, `<name> alternative` and `free <name> alternative` for the brand and each competitor, and `<name> pricing` for each competitor. Keywords with no data are left out of the response; write them as `unknown`.
4. **Price each page.** Click value is monthly searches times CPC: what advertisers pay Google each month for those clicks. Group the searches one page would answer ("x alternative", "x alternatives", "free x alternative"), lead with the one with the most searches, and value the page by that keyword alone. Rank by click value, not volume. Treat a null CPC as unknown and a difficulty of 0 or null as no data. Mark a CPC over $100 as noisy and leave that page out of totals. Searches for the brand's own alternatives aren't pages to build; they go in "Who's using your name".
5. **Check page one.** Call `get_serp_results`, four per call since five can pass 50,000 characters, for the eight pages with the highest click value among those with at least 100 searches, plus the two biggest searches containing the brand's name. Read rows seen in two or three reads (`seenIn`) as page one and quote `bestPosition`; if no row is seen twice, call page one unsettled and say to recheck before building. Label who holds it: other software vendors' own pages, publishers and review sites (G2, Capterra, Zapier), small blogs and forums, or the brand. `url` is a Google redirect, so name each row by `domain`, or by `title` when `domain` is null. Where the site has the page but isn't on page one, open it and note what the ranking pages have that it lacks, such as a list of several tools where it compares only one.
6. **Pick the build order.** Favor high click value and a page one held by vendors' own pages or small blogs rather than big publishers. A page the site has that misses page one counts as a rework.

## Output

A report in this order, with a line or two of prose around each part:

- **The short version:** three lines: the click value of the missing pages (each page's lead keyword once, noisy CPCs left out), the one page to build first, and which competitors rank for searches with the brand's name.
- **Money pages:** a table of page type, page to build (a URL path that fits the site's folders, or its existing URL), target keyword, monthly searches, CPC, monthly click value, difficulty, who holds page one, and status: missing, exists but not on page one, or ranking (with position).
- **Who's using your name:** a table of each search containing the brand, its monthly searches, CPC, the competitors on page one with their positions, and the brand's own position, or `not checked`. If no search contains the brand yet, say so in one line.
- **Build these first:** three to five pages to create or rework, best first, each with its target and variant keywords, the angle that beats page one, and what the page must include, such as a pricing table, a feature grid or a migration guide.
- **Skip these:** searches that looked right but don't fit, with one reason each.

Then offer to save the target keywords with `save_keywords`, tagged `money-pages` and the page type. Ask before saving.

## Notes

- A run is about 8 OpenSEO calls and 200 to 300 credits. Ask before checking more than ten searches on page one or adding competitors.
- Volumes, CPCs and positions are estimates for one market, so click value is an estimate too. Don't report one check of page one as a fixed ranking.
- Comparison and pricing pages must be fair to competitors: check their current prices on their own sites, state their real strengths, and date the page.
