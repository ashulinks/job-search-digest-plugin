# Job Boards — Access Methods by Country

Checked against the live MCP registry: only **Indeed** currently has a dedicated, structured job-search connector relevant to these markets (`search_jobs`, `get_job_details`). Everything else below is web search + fetch. This isn't a permanent gap — re-check the registry periodically (`search_mcp_registry` with keywords like the board's name) in case a connector for LinkedIn, Seek, or the Gulf/Singapore boards is added later, and prefer it over the web-fallback method the moment it exists.

Two other MCP job connectors exist but are narrower-relevance: **ZipRecruiter** (predominantly US postings) and **Dice** (tech/IC roles specifically). Don't offer these as defaults for AU/UAE/SG searches generally; mention them only if the user's keywords are tech-IC-focused or they explicitly want wider coverage.

## Australia

| Board | Method | Notes |
|---|---|---|
| Indeed | MCP `search_jobs` | Filter by country/location; most reliable source |
| Seek | Web search + fetch | Dominant AU board for senior roles; site restricts scraping so treat coverage as partial |
| LinkedIn | Web search + fetch | Job pages are frequently login-gated; expect some fetches to fail — skip and log, don't force |

## UAE

| Board | Method | Notes |
|---|---|---|
| Indeed | MCP `search_jobs` | Most reliable source; set location to UAE/Dubai/Abu Dhabi |
| LinkedIn | Web search + fetch | Same login-gate caveat as above |
| Bayt | Web search + fetch | Largest Gulf-region board; generally fetchable |
| GulfTalent | Web search + fetch | Strong for senior financial-services roles |
| NaukriGulf | Web search + fetch | Broader volume, lower senior-role density |

## Singapore

| Board | Method | Notes |
|---|---|---|
| Indeed | MCP `search_jobs` | Most reliable source |
| LinkedIn | Web search + fetch | Same login-gate caveat |
| JobStreet | Web search + fetch | Largest SG/SEA board |
| MyCareersFuture | Web search + fetch | Government-run board; public pages, generally fetchable, good for regulated-industry roles |

## Query construction

**MCP (Indeed)**: pass the role keyword phrase(s) and a location/country filter directly to `search_jobs`; use `get_job_details` on promising hits to get the full description before scoring.

**Web fallback**: search `"<keyword phrase>" site:<board-domain>` using whatever keyword phrase the user gave in Step 1 (e.g. `"Head of Product" site:seek.com.au` if that's what they searched — the phrase itself always comes from the user, never a fixed default), run one query per keyword phrase per board rather than combining phrases — combined queries return shallower results. Then web_fetch each result URL to get the actual posting (title, company, location, posted date, full description, apply link). If a fetch is blocked or empty, log it as "unreachable — <board>" and move on; never build a digest entry from a search snippet alone.
