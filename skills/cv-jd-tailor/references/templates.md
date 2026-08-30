# Output Template — Australia, Singapore, UAE

One standard template, `assets/templates/CV_Template.docx`, used as the literal starting document for every country and every candidate domain. This file documents its structure, tone, and how to apply per-country deltas at edit time; the docx file itself is the source of truth for exact formatting.

**Fully de-identified and content-generic.** Every fact that would identify a real person or describe one candidate's specific career — name, contact details, employers, program/product names, metrics, certifications, publications, regulatory framework names — has been replaced with a bracketed, intelligently-labeled placeholder (`[FULL NAME]`, `[Employer A]`, `[Framework 1]`, `[N]+ staff`, and so on). What survives unbracketed is pure structure: section and sub-header labels generic enough to apply to most professional domains (`Functional Domain`, `Business Products`, `Regulatory Frameworks`, `Education`, `Certifications`), the table layout, the bold/italic conventions, and the bullet phrasing pattern (full sentences, active verbs, quantified outcomes). There is no baked-in "real CV" to accidentally treat as an answer.

**Format template, not a substitute for gathering the current CV.** Every bracketed placeholder must be replaced with the candidate's own real, current content — never partially filled or left in the output. That input comes from what the user supplies this run, or a master profile already built earlier in the session (SKILL.md Step 1/Step 2), never from the template file's own text.

## Selection rule

`CV_Template.docx` is the base asset for Australia, Singapore, and UAE alike — there is no per-country file to choose between. Country-specific deltas (work-rights status wording, regulatory framework ordering, the optional photo) are applied at edit time per `country-rules.md` and the "Per-market notes" below, not by picking a different asset. In practice, most runs start one step further along than the raw asset — see "How to use the asset template" below for the Base CV cache that skips redundant placeholder-filling. (A user-supplied template overrides both defaults — see SKILL.md Step 1.)

## How to use the asset template (docx output)

Default for every run, not a fallback: never parse the source CV and regenerate a document from scratch (e.g. with docx-js) when a starting document exists. The file already has the correct page 1 (header, capability matrix, education table) and page 2 (professional experience) layout — the job is to replace every remaining bracketed placeholder with the candidate's real content in place.

1. **Pick the starting document per `digest-format.md`'s "Base CV cache"**: if a current `<Country>/Base CV.docx` exists (fingerprint matches `cv-profile.md`), copy that — every placeholder is already filled with the candidate's real content, so this run only needs Step 5's JD-specific reordering/trimming/relabeling on top of it. Otherwise copy `CV_Template.docx` from `assets/templates/`, do the full placeholder-fill this run, and save the result back as the new `Base CV.docx` (per `digest-format.md`) before continuing with JD-specific tailoring. Either way, never start from a blank document.
2. Follow the docx skill's "Editing existing documents" flow: unzip, run `merge_runs.py`, then make **every** text edit for this run — placeholder-filling if starting from the raw template, and/or JD-specific reordering/trimming/relabeling — in **one Python script executed in a single Bash call**, not a series of separate `Edit`/`Bash` round trips. Each additional tool-call round repeats fixed per-turn overhead on top of the edit itself, so planning the full set of changes before touching the file and applying them in one pass is meaningfully cheaper than iterating edit-by-edit.
3. Rezip and validate, then do the Step 6 visual QA render — always at least once per output document, never skipped, regardless of which starting document was used (see "Visual QA before finalizing" below for the render-cost-aware version of this pass).
4. The structural description below documents what's already in the file — it's a guide to what's safe to edit and relabel, not a spec for rebuilding from scratch.
5. Only build from scratch if the user asks for a non-docx format or supplies their own template for this run — and disclose that deviation (SKILL.md's "never fail silently").

## Visual QA before finalizing (SKILL.md Step 6)

Reordering and rewriting can shift how content falls across the 2-page break in ways that aren't visible from the XML alone. Render every run to PDF (`soffice.py --headless --convert-to pdf`, then `pdftoppm -jpeg -r 100`, per the docx skill) and look at each page before scoring or delivering — **this first render is mandatory for every tailored CV, never skipped, regardless of how small the edit looked in the XML.**

**Render-cost-aware re-render policy.** A fix found during QA needs a re-render to confirm it held — but only that one fix needs re-checking, not a full re-QA from scratch:
- Plan Step 5's reordering/trimming/relabeling decisions in full before touching the document, so the first render is checking a finished edit, not a rough draft — this minimizes how often a second render is needed at all.
- If the first render is clean, stop there — don't render again "just to be sure."
- If it isn't clean, make the targeted fix (`keep_with_next`, row-height adjustment, etc.), then re-render **once** to confirm; only repeat beyond that if the fix genuinely didn't hold, and diagnose why before trying a third time rather than re-rendering speculatively.
- `-r 100` (already the default above) is deliberately modest — this pass is checking layout/structure, not final print quality, so there's no need for a higher-resolution render here.

The most common failure mode: a role's title, `Employer | Location | Dates` line, or italic scope sentence ends up as the last line(s) on a page while its achievement bullets start on the next one — a company name should never be stranded alone at the bottom of a page, separated from the responsibilities under it. Fix it by setting `keep_with_next = True` (via python-docx's `paragraph.paragraph_format`) on the role title, the employer/dates line, and the scope sentence, so the whole block moves to the next page together rather than splitting. Re-render after the fix to confirm the block now sits together and the 2-page cap still holds — a keep-together fix that pushes the document to a third page needs content trimming (per SKILL.md Step 5's trimming priority), not a silent overflow.

Also check: a capability-matrix or education-table row split across the page boundary; a widowed single line at the top or bottom of a page; header/contact-block alignment and, for UAE, that an added photo isn't cropped or distorted; and no structural residue from editing (stray empty paragraphs, a leftover `[bracketed placeholder]` that should have been replaced, a run that lost its bold/italic formatting during a rewrite). Fix anything found and re-render — don't score or send a file you haven't actually looked at. **Before Step 6 is considered clean, grep the edited `document.xml` for a literal `[` — any surviving bracket is either a placeholder that didn't get replaced or a genuine JD/profile detail the candidate wants shown in brackets; confirm which before finalizing.**

**Page 1 table with empty space.** Editing (reordering/trimming bullets, relabeling headers, condensing or expanding a column) routinely leaves one column shorter than the others in the capability matrix or the Education/Certifications/Memberships table. If that leaves visible blank space at the bottom of a row or table on page 1 — the row is taller than its content needs — resize it down rather than leaving the gap: check each row's height (python-docx `row.height` / the `w:trHeight` XML) against the tallest cell's actual content after edits, and reduce a row set to a fixed/exact height that now exceeds what its content needs (switch to `WD_ROW_HEIGHT_RULE.AUTO` or lower the explicit value) so the table hugs its content. Re-render to confirm the gap is gone and no new page-break issue was introduced.

## Shared structure (all three countries)

Only the header's photo/status line, the Executive Summary's localization clause, and the Regulatory Frameworks ordering differ by market — everything else is identical.

### Header

Default layout is 2 columns (name/tagline/credential | contact+status), no photo — this is what the asset file ships with, for every country including UAE.

| Target country | Photo | Columns |
|---|---|---|
| Australia | No | 2 (name/tagline/credential \| contact+status) |
| Singapore | No | 2 (name/tagline/credential \| contact+status) |
| UAE | Only if the candidate's own current CV already includes one | 3 if a photo is added (photo \| name/tagline/credential \| contact+status), else 2 |

Per `country-rules.md`, a UAE candidate's photo is included only if their own existing CV already has one — never add one that wasn't there. When one applies, add a third header-table column and insert the image cell (python-docx `add_picture` into the new cell, then rebalance the three column widths) rather than assuming a baked-in photo column exists — there isn't one in the shared asset. Fixed per market otherwise, not a per-run judgment call. Status line: Australia states citizenship/visa only ("Australian Citizen"); Singapore and UAE add a work-rights clause ("· Open to relocation, visa sponsorship required," "Singapore PR," "UAE Resident Visa (employer-sponsored, transferable)").

### Executive Summary

One dense paragraph, 4–6 sentences. Bold the load-bearing facts as they appear: years of experience, current employer + a scale metric, current title/scope, key credential(s), one differentiator. Close naming the seniority of stakeholder advised.

For Singapore and UAE, weave in one clause connecting an existing credential/framework to that market's own regulatory guidance (`regulatory-equivalents.md`) — only where genuinely transferable, framed as alignment, never as claimed local experience (Step 4's rule applies here first, since this is the most visible sentence on the page). Write each market's regulatory clause fresh each run from the specific JD and the candidate's real profile — the asset file's bracketed placeholder gives no market-specific wording to reuse.

### Capability matrix (three-column table)

Heading names the domain: the asset ships with the literal placeholder `[DOMAIN] AND TECHNICAL EXPERIENCE` — replace `[DOMAIN]` with the candidate's real field (e.g. "BANKING AND TECHNICAL EXPERIENCE," "INSURANCE AND CLAIMS EXPERIENCE," "PRODUCT AND TECHNICAL EXPERIENCE"). This is the primary JD-keyword-matching surface — reorder/reselect its bullets first in Step 5, since it's what a recruiter or ATS scans first.

- **Column 1** — organisational/employer experience under bold sub-headers, each a short bulleted list.
- **Column 2** — Transformation & Key Programs under bold sub-headers by theme, each bullet ending with `[Employer]`.
- **Column 3** — functional/governance expertise under bold sub-headers by function, ending with Regulatory Frameworks and Stakeholder & Advisory sub-lists.

**Regulatory Frameworks ordering is country-sensitive**: lead with the target market's own frameworks before international standards — e.g. for Australia/Singapore-market financial-services roles, jurisdiction-specific prudential/privacy frameworks before ISO/NIST-style international ones; for UAE, UAE-market frameworks (CBUAE, DFSA/FSRA, PDPL, etc.) first. The asset's `[Framework 1]`…`[Framework 19]` placeholders carry no market ordering of their own — decide the order fresh each run from `regulatory-equivalents.md` and the candidate's real profile.

#### Filling in the matrix for the candidate's actual domain

Every capability-matrix bullet in the asset is a bracketed placeholder — there is no baked-in domain to relabel away from. Fill each one with the candidate's real, domain-specific content, and treat the organizing sub-header labels already in the file (`Functional Domain`, `Business Products`, and the bracketed `[Functional Area 1]`/`[Program Theme 1]`-style bold sub-headers) as a starting shape, not fixed wording:

- Rename a sub-header when the candidate's domain calls for different categories (e.g. `Business Products` → `Practice Areas` for a legal profile; a `[Functional Area]` sub-header → "Claims Management" for an insurance profile).
- Preserve the shape exactly: column count, bold-sub-header-plus-bullets pattern, `[Employer]` tagging in Column 2. Relabeling changes words, never structure.
- A filled-in label or bullet must describe a real category from the master profile — never invent one the source CV doesn't support. No genuine content for a placeholder sub-header means dropping it, not inventing bullets to fill it.
- The Regulatory Frameworks and Stakeholder & Advisory sub-lists take the candidate's real regulators/standards/forums — omit either sub-list entirely if the candidate's field genuinely has none (see "No empty sections" below) rather than inventing generic-sounding ones.

### Education, Certifications & Memberships (three-column table)

`Education | Certifications | Memberships & Publications` — identical across all three countries. No referees section by default. Every cell is a bracketed placeholder; a candidate without genuine memberships or publications gets that column repurposed or removed — see "No empty sections" below before leaving it blank or filling it with content the profile doesn't support.

## No empty sections

No labeled section, column, or sub-header may render empty in the final output — and no bracketed placeholder may survive into the delivered file unfilled. This generalizes the capability matrix's relabeling rule (above) to the whole document, most commonly relevant to the Memberships & Publications column.

1. **First choice — repurpose with real content.** Check the master profile for other genuine, ungrouped content that fits the space — professional development, notable projects, volunteering, languages, board or committee positions — and relabel the section to match what's actually going in it (e.g. "Memberships & Publications" → "Professional Development"). The content must already be genuinely in the master profile; this is relabel-and-fill, not licence to invent a membership or publication that doesn't exist.
2. **Last resort — remove the section.** Only if nothing in the master profile can legitimately fill it, remove the section or column entirely rather than showing it empty or leaving its placeholder text in place (e.g. collapse the three-column Education/Certifications/Memberships table to two columns and widen the rest, rather than leave a blank or placeholder-filled third column). This is a bigger structural change than the usual content edits here, so disclose it to the user (SKILL.md's "never fail silently") rather than doing it silently.

### Professional Experience (reverse chronological)

Bold role title; bold `Employer | Location | *Dates*` line (optional `| *Left: [reason]*` only if genuinely stated, never invented); italic one-sentence scope/mandate line; bullet achievements.

**"Earlier Career" condensing is a length-management tool, not a default.** The asset condenses roles older than roughly a decade into one "Earlier Career ([YYYY]–[YYYY])" block (one short paragraph per employer, not full bullets) — that's the pattern to reuse when full-detail treatment of every role wouldn't fit on 2 pages, not a rule to always apply. Each run, check against the master profile's actual role list and this run's selected content:

- **If every role fits with full title/employer/scope/bullets treatment within the 2-page cap, drop the Earlier Career block entirely** and give the older roles the same full treatment as recent ones.
- **Only condense into Earlier Career if full-detail treatment would breach the cap**, and condense the oldest roles first (Step 5's trimming priority) before cutting bullets from more recent, more JD-relevant roles.
- Either way, confirm the outcome at Step 6's visual QA re-render — condensing or expanding shifts the page break.

## Tone and voice

Match whichever register a section already uses — never introduce a different tone, sentence structure, or vocabulary when writing a localization clause (Step 4), rewriting an achievement, or relabeling a header (Step 5).

- **Capability matrix**: compressed noun-phrase fragments joined by "·", no verbs or articles (e.g. "Agentic control plane · MCP, AWS Bedrock AgentCore/Gateways, Strands Agents, CDK guardrails"), Column 2 ending in `[Employer]`.
- **Executive Summary and Professional Experience bullets**: full sentences, active verbs — present tense for the current role ("Own," "Chair," "Embed," "Lead"), past tense for every prior role ("Provided," "Led," "Drove," "Built"). Implied third person throughout: no "I," "my," or "we."
- **Register**: formal, board/exec-level, outcome-led — quantify impact where the profile has real numbers, never pad with unevidenced marketing adjectives ("passionate," "dynamic," "world-class," "results-driven").
- **Conventions**: ALL-CAPS section headings, Title Case sub-headers, bold key credentials/scale-metrics/program names as the asset file already does.

### No AI-writing tells in newly written text

This applies only to text actually being composed or rewritten (Executive Summary, Step 4 localization clauses, Step 5's improved bullet phrasing) — never to formatting already in the asset file. The file's own en dashes in date ranges and "·" separators in the capability matrix are structural style to keep exactly as they are.

- **Never introduce an em dash (—) in new prose.** Restructure into two sentences, or use a comma, colon, or parenthesis instead. This is the single most common tell that text was AI-written — don't add it while rewriting.
- Avoid stock AI-sounding phrasing: transition filler ("Furthermore," "Moreover," "It's worth noting," "In today's..."), formulaic triads ("not only X, but also Y"), and generic corporate-speak verbs not already in the source CV's own vocabulary ("leverage," "foster," "robust," "seamless," "holistic" used as filler rather than a genuine term the candidate already uses).
- Before finalizing, reread every sentence you composed or rewrote this run specifically for an em dash or this kind of filler, and fix any you find — this is a final check, not just a drafting guideline.

## Per-market notes

- **Australia**: no photo, no DOB, no marital status (`country-rules.md`). 2-page cap; trim earlier-career detail and lowest-relevance matrix bullets first if over length.
- **Singapore**: no photo — resolves what would otherwise conflict with `country-rules.md`, which discourages a photo for Singapore specifically even though it allows one for UAE. No DOB, no marital status. Same cap and trimming priority.
- **UAE**: photo added only if the candidate's own current CV already includes one (`country-rules.md`) — see "Header" above for how to add the column; never add a photo that wasn't already there. Nationality field only as stated, never inferred. No DOB, no marital status. Same cap and trimming priority.
