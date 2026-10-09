# deep-research

A Claude Code skill for research that has to hold up. It splits a question
into sub-questions, sends each one to the search provider built for its
shape, and synthesizes only after the reads come back, with every finding
cited.

It runs with zero setup. With no providers connected it uses Claude Code's
built-in web search and says so. Each provider you add opens more lanes.

## Install

Clone it into your personal skills folder:

    git clone https://github.com/LV-262/deep-research-skill ~/.claude/skills/deep-research

Or into one project's `.claude/skills/deep-research` to scope it there.
Update with `git pull`. Claude Code picks it up on the next session; type
`/deep-research` or ask a research question and it loads on its own. See
[Claude Code skills](https://code.claude.com/docs/en/skills).

## Providers

All four are optional, and three of the four work without an account.
Connect the ones that match the research you do.

| Provider | Adds these lanes | Without a key | Key or account | Cost |
|---|---|---|---|---|
| [Context7](https://github.com/upstash/context7) | Library, framework, and API docs at the current version | Works, lower rate limit | Free key at [context7.com/dashboard](https://context7.com/dashboard) | Free |
| [Exa](https://github.com/exa-labs/exa-mcp-server) | Discovery search, date and domain filtered search, page fetch, multi-step agent | Search and fetch work, rate-limited | OAuth or a key at [dashboard.exa.ai](https://dashboard.exa.ai/api-keys); needed for the agent | [Pricing](https://exa.ai/pricing) |
| [Firecrawl](https://github.com/firecrawl/firecrawl-mcp-server) | Scrape, map, crawl; GitHub issues and PRs; US government sources; academic papers; agent; monitors | Scrape and search only, rate-limited | Key at [firecrawl.dev/app/api-keys](https://www.firecrawl.dev/app/api-keys) | [Pricing](https://firecrawl.dev/pricing) |
| [Apify](https://github.com/apify/apify-mcp-server) | Structured records from one platform: reviews, listings, social, maps, jobs, app stores | Search and inspect Actors only | Token from [Apify Console](https://console.apify.com/settings/integrations) | [Pay per Actor run](https://apify.com/pricing) |

### Connect them

Each command registers the server under the name the skill expects. Replace
the placeholder with your key, or drop the `--header` line where a key is
optional.

    claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp \
      --header "Authorization: Bearer YOUR_CONTEXT7_KEY"

    claude mcp add --scope user --transport http exa \
      "https://mcp.exa.ai/mcp?tools=web_search_exa,web_fetch_exa,web_search_advanced_exa,agent_run" \
      --header "Authorization: Bearer YOUR_EXA_KEY"

    claude mcp add --scope user --transport http firecrawl https://mcp.firecrawl.dev/v2/mcp \
      --header "Authorization: Bearer YOUR_FIRECRAWL_KEY"

    claude mcp add --scope user --transport http apify "https://mcp.apify.com?tools=actors,docs" \
      --header "Authorization: Bearer YOUR_APIFY_TOKEN"

Keep the `tools` lists in the Exa and Apify URLs. Exa leaves advanced search
and its agent off by default, and Apify's defaults pull in tools that
duplicate the other lanes. Exa and Apify also support OAuth: drop the header
and sign in from `/mcp` if your client offers it.

Use `--scope user`. A project-scoped server lands in `.mcp.json`, which is
made to be committed, and your key would go with it. Run `claude mcp list` to
confirm all four connect. Full reference:
[Claude Code MCP docs](https://code.claude.com/docs/en/mcp).

## How it works

Ask a question that needs more than one read. The skill:

1. Splits it into sub-questions that can be answered independently.
2. Routes each one by shape: library behavior to Context7, "what exists" to
   Exa, a known URL to a Firecrawl scrape, a regulation to Firecrawl's
   government index, app reviews to an Apify Actor. A question with two
   shapes goes to two lanes.
3. Runs the lanes in parallel, primary sources first.
4. Synthesizes once the reads are back: what is established versus claimed,
   which source wins a conflict and why, and what came back empty.
5. Stops when new sources stop changing the answer.

Example: "Should we move our Next.js app from the pages router to the app
router this quarter?" goes to Context7 for the current migration docs,
Firecrawl developer search for open issues on the features you use, and
Exa for recent write-ups from teams who made the move. The report cites
each one.

### Cost tiers

The skill spends money only on purpose.

| Tier | What | Rule |
|---|---|---|
| 0 | Search, scrape, map, docs, the developer, government, and paper indexes, Actor search | Used freely |
| 1 | Apify Actor runs, Firecrawl crawls | Reads the price first, states the expected cost and limits in the same turn |
| 2 | Exa and Firecrawl agents, Firecrawl monitors | Asks you first, with the question and why tier 0 fell short |

It never calls a delete tool. Research reads; it does not mutate.

### When a provider is missing

The skill checks which providers the session actually has before it fans
out. A missing lane takes its fallback, and the report names the lane, the
missing provider, and what the fallback cost in coverage. A search that
quietly narrowed never passes for a complete one.

## Files

- `SKILL.md`: the instructions Claude follows
- `references/providers.md`: tool names, parameters, and per-provider
  details, loaded when a lane's call shape is unclear

## License

MIT. See [LICENSE](LICENSE).
