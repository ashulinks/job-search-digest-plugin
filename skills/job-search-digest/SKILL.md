---
name: job-search-digest
description: Run or configure a recurring, multi-country job search and CV-tailoring pipeline for senior professional roles. Searches selected job boards for a given country and keyword set, dedupes against previously-seen postings, scores each new posting against the user's CV, tailors a CV for strong matches (via the cv-jd-tailor skill), saves results to Drive, and delivers a digest by email, push notification, or as a Claude artifact. Requires Google Drive to be connected — this skill relies on Drive to remember what it's already shown the user across runs. Use this whenever the user wants to set up a job search, run/refresh a job digest, scan boards for roles, or asks about their job search pipeline — including on a recurring/scheduled basis. Do not use this just to tailor a single already-known CV+JD pair — that's cv-jd-tailor directly.
---

# Job Search Digest

Turns a one-off "search for jobs" request into a repeatable pipeline: configure once, run daily/weekly (interactively or via a Cowork Scheduled Task), get a digest of genuinely new, relevant postings, with strong matches already tailored and saved for review.

This skill owns search, dedup, scoring, and delivery. It hands off to `cv-jd-tailor` for the actual CV tailoring — never duplicate that logic here.

## Step 1 — Gather configuration

Do not proceed until all of these are confirmed, and until Google Drive is connected — this skill has no fallback storage, since remembering what it's already shown the user across runs is core to what it does. If Drive isn't connected, say so plainly and stop rather than proceeding without persistence. Ask via clickable options wherever the set of answers is bounded; reserve free text for keywords and the CV itself. If this is a re-run and a saved config exists (see storage convention below), read it back and ask only "same config, or change something?" rather than re-asking everything.

1. **Role keyword(s)** — free text; ask for 2–5 phrases naming the roles or titles the user wants. Too narrow misses postings, too broad floods the digest. This skill is not tied to any field — offer a couple of illustrative examples spanning different domains so the user sees that, e.g. "AI Governance, Head of AI Risk" or "Senior Product Manager, Head of Product" or "Registered Nurse, Clinical Nurse Manager". Never default to any one domain if the user doesn't specify — always ask.
2. **Countries** — multi-select: Australia / UAE / Singapore (others can be added on request, but scoring and board coverage are only pre-built for these three).
3. **Job boards** — multi-select, offered per country from `references/job-boards.md`. Show the reliability note (MCP-backed vs best-effort) alongside each option so the user isn't surprised by gaps later.
4. **CV** — check Drive first for an existing `cv-profile.md` (Step 2 covers this). Only ask the user for a file or pasted CV if no saved profile exists yet, or the user says they want to update it.
5. **Delivery** — single-select: email / push notification (if running via Cowork Scheduled Task) / stored as a Claude artifact only.
6. **Cadence** — daily (default) or weekly. Note explicitly that this skill does the search-score-tailor-digest *work*; actually running it on a schedule requires a Cowork Scheduled Task calling this skill. Say so plainly if the user hasn't set that up — and if they want help doing it, point them to the `job-search-scheduler` skill rather than trying to create the schedule yourself here; that's a separate, one-time setup step this skill doesn't own.

Save the resulting config (keywords, countries, boards by country, delivery, cadence) to Drive as the single global `config.md` per the storage convention in `references/digest-format.md` — one file, not one per country — so re-runs and scheduled runs don't need to re-ask.

## Step 2 — Build or reuse the CV master profile

Check Drive first for `cv-profile.md` (see `references/digest-format.md`). If it exists, load and reuse it — never reparse the CV from scratch when a saved profile is available. If it doesn't exist, parse the Step 1 CV into the same structure cv-jd-tailor uses (roles, dates, achievements, skills, education, citizenship/work-rights per target country) and save it to Drive as `cv-profile.md` immediately — this is what lets the *next* run, including a scheduled one that starts a completely fresh session, skip re-asking for the CV entirely. This profile is what Step 5's scoring runs against, and what gets handed to cv-jd-tailor in Step 6.

## Step 3 — Search each selected board

For each (country, board) pair, follow `references/job-boards.md` for the access method:
- **MCP-backed boards** (currently Indeed): call the connector's job-search tool directly with the keyword set and country/location filter.
- **Best-effort boards** (LinkedIn, Seek, Bayt, GulfTalent, JobStreet, etc.): web_search with the keyword set plus a `site:` filter, then web_fetch each promising result to get real posting details (title, company, location, posted date, description, apply link). Never include a posting in the digest built only from a search snippet — fetch it first.
- If a board is unreachable, blocked, or returns nothing: log it and move on. Never fabricate a posting or fill in fields you couldn't actually retrieve.

**Efficiency — cap fetch volume.** Fetch full details for roughly the top 5–10 most promising results per (country, board, keyword) combination rather than exhaustively fetching everything a search returns. A recurring daily run with several countries, several boards, and several keyword phrases multiplies fast; widen past this only if the user explicitly asks for maximum coverage.

**Efficiency — cheap pre-dedup before fetching.** Before spending a fetch on a search result, check its snippet's title/company/URL against the seen-jobs log (Step 4) — skip the fetch entirely for anything that clearly matches an already-logged entry. This is purely a shortcut to avoid wasted fetches; it never substitutes for Step 4's authoritative check, and it never substitutes for the fetch-before-including rule above when a posting isn't already known to be seen.

Collect raw results per (country, board) before moving to dedup.

## Step 4 — Dedupe against the seen-jobs log

Read the global seen-jobs log (`references/digest-format.md` gives the path). Drop any posting whose (country, board, title, company) already appears there, or whose apply-link exactly matches a logged entry's link — these were surfaced in a previous run and shouldn't be re-presented as new. Keep the log itself for reference; don't remove old entries.

## Step 5 — Score new postings against the CV

For every posting that survives dedup, produce a quick 0–10 relevance score against the master profile, per `references/scoring.md`. This is a cheap pre-filter, not the full cv-jd-tailor rubric — its only job is deciding what's worth a human's attention and what's worth the cost of full tailoring.

- **≥7**: strong match → tailor (Step 6).
- **5–6**: worth reviewing → include in the digest, not tailored.
- **<5**: drop — don't include in the digest at all, but still record it in the seen-jobs log so it's never re-surfaced.

## Step 6 — Tailor CVs for strong matches

For each ≥7 match, invoke the `cv-jd-tailor` skill with: the master profile, the fetched JD, the target country, the user's citizenship/work-rights status for that country (ask once, reuse thereafter), and "use the default template for [country]" unless the user has specified their own. Respect cv-jd-tailor's own Step 5 gate — if its projected score comes in under 8/10 despite this pre-filter, let it stop and report the gap rather than forcing a document; put that outcome in the digest's "Attempted, not tailored" section (`references/digest-format.md`) rather than under "Tailored", and log it with outcome `gap` in Step 8.

Save every produced CV per `references/digest-format.md`'s Drive folder convention.

## Step 7 — Compile and deliver the digest

Build **one combined digest per run** — covering every selected country as its own section, never one digest per country — per the template in `references/digest-format.md`, then deliver it per the Step 1 choice:
- **Email**: send via Gmail, one digest per run, subject line includes date and new-postings count.
- **Push notification**: only meaningful inside a Cowork Scheduled Task run — summarize to a couple of lines (count of new matches across all countries, top one by score) rather than the full digest body.
- **Claude artifact**: render the full digest as a markdown artifact in-conversation.

Also save a dated copy of the combined digest to `Job Search/Digests/` in Drive per the same reference file, so there's a durable archive independent of the delivery channel.

## Step 8 — Update the seen-jobs log

Append every posting seen this run to the global seen-jobs log before finishing, with outcome `tailored`, `gap`, `reviewed`, or `dropped` (`references/digest-format.md` defines each) — so the next run's dedup is accurate. Do this last, after delivery succeeds — a failed delivery shouldn't silently mark postings as "already told the user."

## Failure handling — never fail silently

Say so explicitly whenever something can't be done as configured: a board that returned nothing or errored, a country/board combination with no coverage, a CV that couldn't be parsed, a delivery channel that failed (e.g. Gmail send error) — don't report a digest as "sent" if it wasn't. If this is a scheduled run with no one to ask a clarifying question, make the most reasonable documented assumption, proceed, and flag the assumption prominently at the top of the digest rather than blocking the whole run.

## Reference files

- `references/job-boards.md` — per-country board list, access method (MCP vs web fallback), and query construction guidance
- `references/scoring.md` — the quick 0–10 relevance scoring approach and thresholds
- `references/digest-format.md` — global config/profile/log conventions, the combined digest template, and Drive folder structure
