# Job Boards — Access Methods by Country

No MCP-backed job-search connector is used by default for these markets — every board below is web search + fetch, or Employer Career Pages. This isn't a permanent state — re-check the registry periodically (`search_mcp_registry` with keywords like the board's name) in case a relevant connector becomes available, and prefer it over the web-fallback method the moment it does; a structured connector is generally more reliable than search-and-fetch when one genuinely fits.

Two other MCP job connectors exist but are narrower-relevance: **ZipRecruiter** (predominantly US postings) and **Dice** (tech/IC roles specifically). Don't offer these as defaults for AU/UAE/SG searches generally; mention them only if the user's keywords are tech-IC-focused or they explicitly want wider coverage.

## Excluded boards — do not fetch, on any country

**LinkedIn** and **SEEK** are excluded from fetching entirely, for every country, always — never offer either as a selectable board in Step 1, and never fetch an individual posting page from either, regardless of what a search result surfaces. Both explicitly prohibit automated access in their terms (LinkedIn's User Agreement bars bots/automated methods generally, authenticated or not; SEEK's terms bar "any robot, spider, scraper, or other automated means to access the Site"), and both block it in practice — LinkedIn job pages return `ROBOTS_DISALLOWED` on fetch, SEEK returns empty content or 403s. This is not a reliability gap to route around with retries, browser automation, or a different fetch tool — deliberately bypassing a site's access controls is out of scope for this skill regardless of how the user asks. If a search snippet surfaces a LinkedIn or SEEK posting title/company, it may still inform the **Employer Career Pages** roster below (see "Auto-add discovered companies") — but the posting itself is never fetched, scored, or placed in the digest from that snippet alone.

Use **Employer Career Pages** (below) as the replacement for both, in every country. For Australia specifically, since SEEK and LinkedIn were previously the only two boards offered and Indeed is no longer used, Employer Career Pages is that country's sole method unless the user adds a board of their own.

## Australia

| Board | Method | Notes |
|---|---|---|
| Seek | **Excluded** | See "Excluded boards" above — use Employer Career Pages instead |
| LinkedIn | **Excluded** | See "Excluded boards" above — use Employer Career Pages instead |

No default board remains for Australia — Employer Career Pages (below) is the whole of this country's coverage unless the user names a board of their own to add.

## UAE

| Board | Method | Notes |
|---|---|---|
| LinkedIn | **Excluded** | See "Excluded boards" above — use Employer Career Pages instead |
| Bayt | Web search + fetch | Largest Gulf-region board; generally fetchable |
| GulfTalent | Web search + fetch | Strong for senior financial-services roles |
| NaukriGulf | Web search + fetch | Broader volume, lower senior-role density |

## Singapore

| Board | Method | Notes |
|---|---|---|
| LinkedIn | **Excluded** | See "Excluded boards" above — use Employer Career Pages instead |
| JobStreet | Web search + fetch | Largest SG/SEA board |
| MyCareersFuture | Web search + fetch | Government-run board; public pages, generally fetchable, good for regulated-industry roles |

## Employer Career Pages

The replacement method for the excluded boards above, and available as an additional method for any country even when other boards are working fine. Instead of searching a job board, search each target employer's own career site or ATS (Workday, Greenhouse, Lever, SmartRecruiters, SuccessFactors, iCIMS, Taleo, etc.) directly — those are the employer's own domain, not a restricted board, so ordinary fetch rules apply with no ToS conflict.

**Roster, per country**, stored in `config.md` — schema, normalization rule, and pruning policy are defined in `references/digest-format.md`; this section covers only how the roster gets *searched*, not how it's stored.

**Per run — query batching.** For each roster employer, try one combined query first: `("<keyword 1>" OR "<keyword 2>" OR "<keyword 3>") <employer> careers` (or the employer's known career-site domain directly). This cuts search volume roughly 3x versus one query per keyword, which matters a lot here — unlike a job board, a single employer rarely has more than one or two relevant postings open at once, so there's little coverage lost by asking for all keywords at once. **Fall back to one query per keyword phrase, for that employer only,** if the combined query errors, or returns zero results where a per-keyword query might plausibly do better (e.g. the search tool appears to have ignored the OR grouping). Don't fall back pre-emptively "just in case" — only when the combined query demonstrably underperforms.

Search up to a per-country cap defined in `config.md` (default 12–15 if the user hasn't specified one). Always include every usual suspect before filling remaining slots with discovered companies; if discovered companies exceed the remaining slots, prioritise ones that have actually produced a scored posting before, and rotate the rest across runs rather than letting the same few crowd out others indefinitely.

**Fetching**: fetch postings directly, same as any other web-fallback board. Some ATS platforms (Workday in particular) render client-side and return only metadata to a fetch — log that as "unreachable — client-rendered" and move on. If a specific employer's own career page turns out to block fetching (its own robots.txt, a 403, an empty response pattern like SEEK's) — treat it exactly like the excluded boards above: log it, move on, and never fall back to browser automation to force it open. That boundary applies to every page this skill touches, not just the two boards named explicitly.

## Query construction

**MCP (if a suitable connector is later added)**: pass the role keyword phrase(s) and a location/country filter directly to its search tool; use its detail-lookup tool on promising hits to get the full description before scoring.

**Web fallback**: search `"<keyword phrase>" site:<board-domain>` using whatever keyword phrase the user gave in Step 1 (e.g. `"Head of Product" site:seek.com.au` if that's what they searched — the phrase itself always comes from the user, never a fixed default), run one query per keyword phrase per board rather than combining phrases — combined queries return shallower results. Then web_fetch each result URL to get the actual posting (title, company, location, posted date, full description, apply link). If a fetch is blocked or empty, log it as "unreachable — <board>" and move on; never build a digest entry from a search snippet alone.
