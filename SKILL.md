---
name: deep-research
description: Parallel fan-out research and synthesis. Use when a decision needs evidence from more sources than one read can hold.
---

# Deep research

Decompose the question into sub-questions that can be answered independently.
Fan out across them; synthesize only after the reads come back.

Rules of evidence:

- Primary source over summary. Read the thing itself before reading about it.
- Separate what is established from what is claimed, and say which is which.
- Record what you searched that came back empty. A search with no hits is
  still evidence; a negative result is a finding, and the next reader
  should not pay for it twice.
- On conflicting sources, state both and name which is better sourced and why.
- Cite where every finding came from. A finding with no source is a claim.

Stop when new sources stop changing the answer. Reading past that point is
motion, not progress.

## Lanes

Route each sub-question to the lane that fits its shape, not to one default
search engine. A question with two shapes goes to two lanes.

| Shape of the sub-question | Lane | Fallback |
|---|---|---|
| Library, framework, or API behavior | context7: resolve the library id, then query docs | firecrawl developer search, then the vendor docs page |
| Issues, pull requests, READMEs | firecrawl developer search | exa search scoped to the code host |
| Discovery: what exists, who does this, similar to X | exa search | firecrawl search |
| Recent or dated: within a window, from named domains | exa advanced search with date and domain filters | firecrawl search |
| Read a known URL | firecrawl scrape | exa fetch |
| Map or read a whole site | firecrawl map, then scrape the pages that matter | firecrawl crawl (tier 1) |
| Academic papers and citation trails | firecrawl research papers: search, inspect, related, read | exa search for the paper, then scrape |
| US government, legal, regulatory | firecrawl gov search | firecrawl scrape of the primary .gov source |
| Structured records from one platform: reviews, listings, social, maps, jobs, app stores | apify: search actors, read details, then call (tier 1) | firecrawl scrape of the public page |
| Multi-step research that parallel lanes could not close | exa agent or firecrawl agent (tier 2) | another round of lanes with sharper sub-questions |
| Watch a source for change over time | firecrawl monitors (tier 2) | a dated note to re-check |

Context7 is the first stop for anything a library's own docs answer. Training
memory of an API is a claim, not a source.

## Cost tiers

- Tier 0, use freely: context7, exa search and fetch, firecrawl search, scrape,
  map, and the developer, gov, and paper lanes, apify actor search and details.
- Tier 1, metered per run: apify actor calls, firecrawl crawl. Read the
  actor's pricing in its details first, then state the expected cost and the
  input bounds (result limits, page caps) in the same turn as the call.
- Tier 2, long-running or persistent: the exa and firecrawl agents, firecrawl
  monitors. Ask the operator before starting one, with the question it will
  answer and why tier 0 could not.

Never call a delete tool from any research provider. Research reads; it does
not mutate.

## When a lane is down

Before the fan-out, check which providers this session actually has. If a
lane's provider is missing or erroring, take its fallback and say so in the
report: name the lane, the provider that was absent, and what the fallback
cost in coverage. A silently narrowed search reads as a complete one.

When no research provider is present, use the built-in web search and fetch,
and say the lanes were unavailable.

Tool names, parameters, and per-provider gotchas: references/providers.md,
next to this file. Read it when a lane's call shape is unclear.
