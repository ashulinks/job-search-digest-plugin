# Digest, Storage, and Log Conventions

Google Drive is required for this skill — see SKILL.md Step 1. Everything below assumes it's connected.

## Drive folder structure

Config, the CV profile, and the dedup log are global — one job search, potentially spanning several countries, isn't three separate things. Only tailored CVs are genuinely country-specific (different template/format per country), so only they get per-country subfolders:

```
Job Search/
  config.md               — global config (keywords, countries, boards by country, delivery, cadence)
  cv-profile.md           — the parsed master CV profile, built once and reused every run
  seen-jobs.md            — global dedup log across all selected countries
  run-lock.md             — present only while a run is in progress; see "Run lock" below
  Digests/
    YYYY-MM-DD-digest.md  — one combined digest per run, sectioned by country
  <Country>/
    Tailored CVs/
      YYYY-MM-DD_Company_RoleTitle.docx
```

Use the country name as it appears in Step 1 (Australia / UAE / Singapore) for the `<Country>` folders. Create any part of this tree on first run if it doesn't exist.

## config.md

One file, not one per country:

```markdown
# Job Search Digest — Config
- Keywords: <comma-separated phrases>
- Countries: <selected countries>
- Boards by country:
  - Australia: <selected boards, if Australia is selected — never LinkedIn or SEEK, see job-boards.md>
  - UAE: <selected boards, if UAE is selected — never LinkedIn>
  - Singapore: <selected boards, if Singapore is selected — never LinkedIn>
- Delivery: <email | push | artifact>
- Cadence: <daily | weekly | hourly | any other cron-expressible interval>
- Last updated: <date>

## Employer Career Pages roster (optional, per country)

Only present for a country using the Employer Career Pages method (`job-boards.md`) — every
country using SEEK or LinkedIn's former slot needs one; any other country may add one too.

### Roster — <Country>
**Usual suspects (permanent):** <comma-separated companies>

**Discovered (auto-added):**
- <Company> — added <date> — snippet: "<title/snippet that justified the add>"

**Per-run cap:** <number, default 12–15 if unset>
```

**Cadence is free text, not a closed set.** Daily and weekly are the common defaults to offer in
Step 1, but anything a Cowork Scheduled Task can express (including hourly) is valid — write down
whatever the user actually asks for rather than coercing it to the nearest of the two defaults.

**Company-name normalization (roster dedup).** Before comparing a candidate name against the
existing roster, lowercase it and strip trailing legal-entity suffixes (Ltd, Limited, Group, Bank,
Inc, LLC, PLC, Pte, Co, Corporation, and similar) and punctuation. "Macquarie," "Macquarie Group,"
and "Macquarie Bank Limited" must normalize to the same key and therefore never produce three
roster entries for one employer. When genuinely unsure whether two names refer to the same
company, don't add the ambiguous one — surface it in that run's digest Notes for the user to
resolve manually instead of guessing.

**Pruning discovered companies.** Never delete a discovered entry outright — the point is to never
lose a lead. Instead, once a discovered company has gone 20 consecutive runs (not calendar days —
runs, so this scales correctly whether cadence is hourly or weekly) without producing a posting
that survived dedup, stop including it in the active per-run search rotation and mark it `dormant`
in its roster line. A dormant entry is skipped when filling per-run slots but still counts toward
"never forget a lead," and reactivates automatically (drop the `dormant` mark) the moment it's
mentioned again in a fresh search snippet. This is what keeps the active search list from growing
without bound while the historical roster itself can keep growing freely.

Both this file and `cv-profile.md` below are also read directly by the `job-search-scheduler` skill when it sets up the recurring task — it never re-asks for any of this, it just requires both files to already exist. Keep the field set here as the single source of truth; don't let any other skill restate or duplicate it.

## Run lock (overlap guard)

Before Step 1 of any run (scheduled or interactive), check Drive for `Job Search/run-lock.md`:

- **Doesn't exist, or exists but its `Started` timestamp is more than 2 hours old** (a lock that
  old means a previous run almost certainly crashed rather than being genuinely still active):
  write/overwrite it with `# Job Search Digest — Run Lock\n- Started: <current ISO timestamp>`,
  then proceed normally.
- **Exists and is 2 hours old or less**: a previous run is genuinely still in progress. Stop before
  doing any search or write work. On a scheduled run, end quietly — no digest, no notification,
  nothing appended to seen-jobs.md; this is expected behavior at short cadences, not a failure. On
  an interactive run, tell the user plainly that a previous run appears to still be in progress
  (with its start time) and ask whether to wait or proceed anyway; proceeding anyway overwrites the
  lock with a fresh timestamp exactly as above.

**Always delete `run-lock.md`** as the very last action of a run — after Step 8, and also on any
handled failure path (Step 8's "never fail silently" still applies; clearing the lock is separate
from whether the run succeeded). A run that crashes before reaching its own cleanup simply leaves
the lock to expire on its own via the 2-hour staleness check above, rather than blocking every
future run permanently.

## cv-profile.md

The master CV profile (SKILL.md Step 2), in the same structured form cv-jd-tailor uses — roles, dates, achievements, skills, education, citizenship/work-rights per country. Check for this file **before** asking the user for a CV: if it exists, load and reuse it without reparsing; only ask for a CV upload if this file doesn't exist yet, or the user explicitly says they want to update it. Overwrite it whenever the user supplies a genuinely updated CV.

## seen-jobs.md

One line per posting ever seen, appended-only (never delete or rewrite past entries), across every country:

```markdown
- 2026-08-30 | Australia | Indeed | Senior Product Manager | Example Bank | score:8 | tailored | https://...
- 2026-08-30 | UAE | Bayt | Product Analyst | Example Corp | score:4 | dropped
- 2026-08-30 | Singapore | JobStreet | Head of Product | Example Pte Ltd | score:6 | reviewed
```

The trailing link field is optional — include it whenever the posting had a real apply-link (most will); omit it only if none was ever retrieved.

Outcome is one of:
- `tailored` — score ≥7, cv-jd-tailor produced a CV
- `gap` — score ≥7 here, but cv-jd-tailor's own gate (its Step 5) declined to produce one
- `reviewed` — score 5–6, surfaced in the digest but never sent to tailoring
- `dropped` — score <5, never shown

Check this file at the start of Step 4 (dedup) by matching on (country, board, title, company), or on an exact link match when both postings have one — a posting already present under either match is not "new" regardless of when it was first seen.

## Digest template

One combined digest per run, covering every country searched that run — never one digest per country:

```markdown
# Job Search Digest — <Date>

<N> new postings found across <countries searched> on <boards searched>. <X> tailored, <Y> worth reviewing, <G> attempted but not tailored, <Z> below threshold (not shown).

## <Country>

### Tailored (score ≥7)
#### <Title> — <Company>
- Board: <board> | Posted: <date> | Score: <score>/10
- Link: <apply link>
- Tailored CV: <Drive path>

### Worth reviewing (score 5–6)
#### <Title> — <Company>
- Board: <board> | Posted: <date> | Score: <score>/10
- Link: <apply link>
- Why borderline: <one line>

### Attempted, not tailored (score ≥7 here, cv-jd-tailor's gate not met)
#### <Title> — <Company>
- Board: <board> | Posted: <date> | Score: <score>/10
- Link: <apply link>
- Gap: <what cv-jd-tailor's own Step 5 reported>

## <Next country, same structure, if more than one was searched>

## Notes
- Boards unreachable this run: <list, or "none">
- Employer Career Pages roster additions this run: <company, country, snippet that justified it — or "none">
- Assumptions made this run: <list, or "none">
```

Keep each entry to what's shown above — no extra commentary per posting. If a board returned nothing, still state that plainly under Notes rather than omitting it silently. Omit a section entirely (e.g. no "Attempted, not tailored" heading) rather than showing it empty.

Storage path: `Job Search/Digests/YYYY-MM-DD-digest.md` — always the combined file, never split per country.

## Push notification summary (Cowork Scheduled Task only)

Two lines max: total new-match count across every country searched this run, and the single highest-scoring posting's title, company, and country. Full detail lives in the digest artifact/email/Drive copy, not the push notification itself.
