# Job Search Digest

A recurring, multi-country job search and CV-tailoring pipeline, packaged as
one Claude plugin. Install once — no separate file-by-file setup.

## What's inside

- **job-search-digest** — searches job boards (Australia, UAE, Singapore),
  dedupes against what it's already surfaced, scores postings against your
  CV, and hands strong matches to the tailoring skill. Owns all
  configuration (keywords, countries, boards, delivery, cadence) and your CV
  profile — saved once to Drive, never re-asked or duplicated elsewhere in
  this plugin.
- **cv-jd-tailor** — tailors your CV to a specific job description for
  Australia, UAE, or Singapore, scored against a rubric before it commits to
  producing anything.
- **job-search-scheduler** — a one-time setup skill: run it inside Cowork and
  it creates the recurring scheduled task for you, inside a dedicated
  **Cowork Core project** (never a local folder, so it actually runs daily
  regardless of whether your computer is on), using the config and CV
  profile job-search-digest already saved. Checks first whether this has
  already been done and won't create a duplicate.

## Install

**From a file:** Claude → Customize → Plugins → upload this package.

**From git:** Claude → Customize → Plugins → Add marketplace → paste this
repository's URL (or `owner/repo` on GitHub). Then install `job-search-digest`
from the marketplace it adds.

Either way, all three skills install together — no need to handle them one
at a time.

## Setup after installing

### 1. Connect Google Drive

Required, not optional — it's the single source of truth for your
configuration, CV profile, dedup log, tailored CVs, and past digests.
Connect **Gmail** too if you want digests delivered by email.

### 2. Run job-search-digest once, interactively

Just ask it to search for jobs. This gathers your configuration and builds
your CV profile, saving both to Drive. Everything after this depends on
those two files already existing.

### 3. Set up the schedule

Inside Cowork (chat and Cowork share the same window — there's a Cowork
toggle next to the message box on desktop/web, or a Cowork entry in the
sidebar on mobile), say: **"set up my recurring job search."**

That triggers `job-search-scheduler`, which confirms your configuration
exists, creates (or reuses) a Cowork Core project with the right standing
instructions, creates the scheduled task with a one-line prompt (`Run the
job-search-digest skill.` — everything else comes from Drive), and records
that setup is done so it won't ever create a duplicate.

## A note on how this plugin was packaged

`plugin.json` follows the documented Claude Code plugin manifest schema and
is built with confidence. `marketplace.json` is best-effort, assembled from
available documentation rather than tested directly — if adding this as a
git marketplace doesn't work cleanly, that file is the first place to check
against the current official schema.
