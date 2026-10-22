---
name: keyword-gap
description: Finds keywords one or more competitors rank for that your site doesn't, using OpenSEO ranking data. Use when the user asks for a keyword gap, a content gap, or what competitors rank for that they don't.
license: MIT
---

# Keyword gap analysis

Compare the keywords a site ranks for against a few competitors and list the ones the competitors rank for and the site doesn't. Needs the OpenSEO MCP server connected (https://openseo.so).

## Inputs

- The user's site. If an OpenSEO project already exists for it, use that project; call `list_projects` to find its `projectId`.
- Two or three competitor domains. If the user has none, ask for them, or offer to find them with `find_serp_competitors` on five to ten of the site's main keywords.

## Steps

1. **The site's keywords.** Call `get_ranked_keywords` with `target` set to the user's domain, `maxRank: 100` and `limit: 100`. Keep the keyword list.
2. **Each competitor's keywords.** Call `get_ranked_keywords` for each competitor with `maxRank: 20`, `minSearchVolume: 50` and `limit: 100`. Top-20 rankings show what the competitor is actually winning.
3. **Find the gaps.** A keyword is a gap when a competitor ranks for it in the top 20 and the user's site doesn't rank in the top 100. Match keywords case-insensitively. Drop branded terms for the competitor (its name or product names).
4. **Rank the gaps.** Put keywords more than one competitor ranks for first, then sort by search volume. Note the search intent from the rows; informational gaps usually mean a missing article, commercial or transactional gaps a missing product or landing page.
5. **Group them.** Put gaps that would be answered by the same page together, and name the page each group needs.

## Output

A short report:

- One line per competitor: how many keywords it ranks for in the top 20, and how many of those are gaps.
- A table of the top 20 gaps: keyword, monthly searches, intent, which competitors rank and at what position, and the competitor URL that ranks.
- Three to five page ideas, each covering one group of gaps, best opportunity first.

Then offer to save the top gaps to the project with `save_keywords`, tagged `keyword-gap`. Ask before saving or tagging.

## Notes

- Each `get_ranked_keywords` call uses OpenSEO credits. Three competitors plus the site is four calls. Tell the user before going past ten calls.
- Ranking data is an estimate for one market. Use the project's default market unless the user names another country.
