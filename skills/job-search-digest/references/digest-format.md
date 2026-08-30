# Digest, Storage, and Log Conventions

Google Drive is required for this skill — see SKILL.md Step 1. Everything below assumes it's connected.

## Drive folder structure

Config, the CV profile, and the dedup log are global — one job search, potentially spanning several countries, isn't three separate things. Only tailored CVs are genuinely country-specific (different template/format per country), so only they get per-country subfolders:

```
Job Search/
  config.md               — global config (keywords, countries, boards by country, delivery, cadence)
  cv-profile.md           — the parsed master CV profile, built once and reused every run
  seen-jobs.md            — global dedup log across all selected countries
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
  - Australia: <selected boards, if Australia is selected>
  - UAE: <selected boards, if UAE is selected>
  - Singapore: <selected boards, if Singapore is selected>
- Delivery: <email | push | artifact>
- Cadence: <daily | weekly>
- Last updated: <date>
```

Both this file and `cv-profile.md` below are also read directly by the `job-search-scheduler` skill when it sets up the recurring task — it never re-asks for any of this, it just requires both files to already exist. Keep the field set here as the single source of truth; don't let any other skill restate or duplicate it.

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
- Assumptions made this run: <list, or "none">
```

Keep each entry to what's shown above — no extra commentary per posting. If a board returned nothing, still state that plainly under Notes rather than omitting it silently. Omit a section entirely (e.g. no "Attempted, not tailored" heading) rather than showing it empty.

Storage path: `Job Search/Digests/YYYY-MM-DD-digest.md` — always the combined file, never split per country.

## Push notification summary (Cowork Scheduled Task only)

Two lines max: total new-match count across every country searched this run, and the single highest-scoring posting's title, company, and country. Full detail lives in the digest artifact/email/Drive copy, not the push notification itself.
