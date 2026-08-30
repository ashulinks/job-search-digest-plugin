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

## Board coverage

LinkedIn (all countries) and SEEK (Australia) are excluded from fetching entirely —
both prohibit automated access in their terms and block it in practice, so
job-search-digest never fetches either, and never routes around the block with
browser automation. In their place, **Employer Career Pages** searches a per-country
roster of target companies' own career sites/ATS directly (Macquarie's own portal and
icare NSW's site, for example, work cleanly where LinkedIn does not). The roster starts
from a "usual suspects" seed (never removed) and grows automatically as new companies
surface in search results at a senior, keyword-matching level — every addition is
logged in the digest so the growth stays visible even without a manual approval step.
See `skills/job-search-digest/references/job-boards.md` for the full mechanics.

## Changelog

- **1.9.0** — Fixed two bugs found in real use. (1) Tailored CVs weren't reaching Drive when
  cv-jd-tailor was used directly/standalone (outside the job-search-digest pipeline) — only the
  pipeline itself had a save-to-Drive instruction, so a directly-tailored CV was only ever
  available as an in-chat download. cv-jd-tailor now owns saving its own output (new Step 8),
  every time, using the same `Job Search/<Country>/Tailored CVs/` convention and one-shot base64
  upload discipline, whether invoked directly or via the pipeline; it explains plainly and skips
  the save if Drive isn't connected, rather than failing silently. (2) A delivered CV lost all
  table shading and most run-level text colors versus the source template — traced to
  `references/templates.md` instructing python-docx's `Document(path)` + `.save()` for row-height,
  keep-with-next, and photo-insertion edits, which round-trips the whole file through python-docx's
  object model and doesn't reliably preserve every element of a hand-authored template. Every edit
  in that file (including those three) is now a direct `word/document.xml` XML edit instead, inside
  the same single-script workflow already used for text edits — and Step 6's visual QA gained a
  mandatory shading/color element-count check against the starting document, so a regression like
  this is caught before delivery, not after.
- **1.8.0** — Extended the Base CV cache (1.5.0) to cover a user-supplied template as well
  as the default one. Previously a persisted `Job Search/User Template.docx` bypassed the
  cache entirely — every tailoring re-did the full reconciliation pass from scratch, the
  same cost the cache was built to avoid. `Base CV.meta.md` now fingerprints both the
  source template and the profile, so it correctly rebuilds when either the profile
  changes, the user switches template preference, or they replace their supplied file —
  and reuses the cached base otherwise, regardless of which template is configured.
- **1.7.0** — job-search-digest now asks once, in its own Step 1, whether tailored CVs
  should use the default template or a template the user supplies — closing a gap where
  a recurring digest run always silently used the default with no way to actually record
  or honor a user's own template. The choice is stored in `config.md` (`Template:
  default | user-supplied`); a supplied file is saved to Drive as `Job Search/User
  Template.docx` and reused on every future run without re-asking. cv-jd-tailor's own
  Step 1 template question now only fires when it's used directly, outside the digest
  pipeline. Also fixed a stale reference to "three template files" left over from the
  1.4.0 consolidation to one template.
- **1.6.0** — Removed Indeed as a board. No MCP-backed job-search connector is used by
  default now; every country falls back to web search + fetch and/or Employer Career
  Pages. Australia in particular now has no default board at all — Employer Career Pages
  is its sole coverage unless the user adds one — since Indeed was its only non-excluded
  option alongside the already-excluded SEEK and LinkedIn.
- **1.5.0** — Cost optimization pass, based on real per-session token-cost data. Added a
  per-country **Base CV cache** (`<Country>/Base CV.docx` in Drive): the expensive part of
  tailoring — filling every placeholder in `CV_Template.docx` with the candidate's real
  content — now happens once per country per CV-profile version instead of once per job
  opening; every tailoring after that reuses the cached base and only does the cheap
  JD-specific reordering/trimming on top of it. Documented a single-script edit discipline
  (all XML edits for a run in one Python script, one Bash call, not incremental round
  trips) and a render-cost-aware visual-QA policy (one mandatory render per document, a
  second only to confirm an actual fix, never speculative re-renders). Clarified the
  correct one-shot base64 upload pattern for saving a docx to Drive, since Drive's upload
  tool has no "upload by path" option and a mishandled retry can be expensive. No cap was
  added to tailoring volume — kept at the user's explicit request — since these changes
  target the cost of each tailoring rather than how many happen.
- **1.4.0** — Consolidated to a single CV template. `Singapore_CV.docx` and `UAE_CV.docx`
  are removed; the former `Australia_CV.docx` is renamed `CV_Template.docx` and used as
  the one starting document for all three countries (country-specific deltas — work-rights
  wording, regulatory framework ordering, the optional UAE photo — are now applied at edit
  time per `country-rules.md`, not by picking a different file). Its content is now fully
  generic: every employer, program name, metric, certification, and framework name is a
  bracketed, intelligently-labeled placeholder (`[Employer A]`, `[Framework 1]`, `[N]+
  staff`) rather than one person's real career facts — cv-jd-tailor fills every placeholder
  with the candidate's own real content each run, same as it always has.
- **1.3.0** — The template's header identity fields (name, tagline, phone, email,
  LinkedIn, citizenship status) replaced with bracketed placeholders, so the shipped
  template carries no individual's personal information. No behavior change: the skill
  already always pulls the real header/contact details from the user's own supplied CV,
  never from the template file's text.
- **1.2.0** — Performance/reliability review: Employer Career Pages batches each
  employer's keywords into one query instead of one-per-keyword (falls back to
  per-keyword only if the batched query underperforms); added a run-lock so an hourly
  or otherwise fast cadence can't start a new run while a previous one is still going;
  defined a company-name normalization rule and a dormant/rotation policy so the
  roster's active search list can't grow without bound; cadence is now documented as
  free text (daily/weekly are just the two defaults offered) instead of implying it's
  limited to those two.
- **1.1.0** — Removed LinkedIn/SEEK fetching (ToS/robots.txt block); added Employer
  Career Pages as the replacement method, with an auto-growing, always-logged roster.
- **1.0.0** — Initial packaging.

## A note on how this plugin was packaged

`plugin.json` follows the documented Claude Code plugin manifest schema and
is built with confidence. `marketplace.json` is best-effort, assembled from
available documentation rather than tested directly — if adding this as a
git marketplace doesn't work cleanly, that file is the first place to check
against the current official schema.
