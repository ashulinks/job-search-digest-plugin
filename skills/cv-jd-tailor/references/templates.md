# Output Templates — Australia, Singapore, UAE

Three default templates, all based on the user's own real CV at the time they were built — provided as literal starting documents under `assets/templates/` (`Australia_CV.docx`, `Singapore_CV.docx`, `UAE_CV.docx`). This file documents their shared structure, tone, and per-country deltas; the docx files themselves are the source of truth for exact formatting.

**Format templates, not a substitute for gathering the current CV.** Their baked-in content was real when built but goes stale as the candidate's experience changes. Never let it stand in for gathering the real, current CV each time (SKILL.md Step 1) — that input comes from what the user supplies this run, or a master profile already built earlier in the session (Step 2), never from reading the template file's own text as the answer.

## Selection rule

- **Australia** → `assets/templates/Australia_CV.docx`
- **Singapore** → `assets/templates/Singapore_CV.docx`
- **UAE** → `assets/templates/UAE_CV.docx`

Never cross-apply one country's file to another's output — the header photo rule and status line differ (below), and this is a hard rule, not a per-run judgment call. (A user-supplied template overrides these defaults — see SKILL.md Step 1.)

## How to use the asset templates (docx output)

Default for every run, not a fallback: never parse the source CV and regenerate a document from scratch (e.g. with docx-js) when a matching asset exists. The asset file already has the correct page 1 (header, capability matrix, education table) and page 2 (professional experience) layout — the job is to edit its real content in place.

1. Copy the matching file from `assets/templates/` as this run's working file — never start from a blank document.
2. Follow the docx skill's "Editing existing documents" flow: unzip, run `merge_runs.py`, edit `word/document.xml` in place to swap in the tailored content (reordered achievements per Step 5, localized regulatory references per Step 4, relabeled headers where applicable), then rezip and validate.
3. The structural description below documents what's already in the file — it's a guide to what's safe to edit and relabel, not a spec for rebuilding from scratch.
4. Only build from scratch if the user asks for a non-docx format or supplies their own template for this run — and disclose that deviation (SKILL.md's "never fail silently").

## Visual QA before finalizing (SKILL.md Step 6)

Reordering and rewriting can shift how content falls across the 2-page break in ways that aren't visible from the XML alone. Render every run to PDF (`soffice.py --headless --convert-to pdf`, then `pdftoppm -jpeg`, per the docx skill) and look at each page before scoring or delivering.

The most common failure mode: a role's title, `Employer | Location | Dates` line, or italic scope sentence ends up as the last line(s) on a page while its achievement bullets start on the next one — a company name should never be stranded alone at the bottom of a page, separated from the responsibilities under it. Fix it by setting `keep_with_next = True` (via python-docx's `paragraph.paragraph_format`) on the role title, the employer/dates line, and the scope sentence, so the whole block moves to the next page together rather than splitting. Re-render after the fix to confirm the block now sits together and the 2-page cap still holds — a keep-together fix that pushes the document to a third page needs content trimming (per SKILL.md Step 5's trimming priority), not a silent overflow.

Also check: a capability-matrix or education-table row split across the page boundary; a widowed single line at the top or bottom of a page; header/contact-block alignment and, for UAE, that the photo isn't cropped or distorted; and no structural residue from editing (stray empty paragraphs, a run that lost its bold/italic formatting during a rewrite). Fix anything found and re-render — don't score or send a file you haven't actually looked at.

**Page 1 table with empty space.** Editing (reordering/trimming bullets, relabeling headers, condensing or expanding a column) routinely leaves one column shorter than the others in the capability matrix or the Education/Certifications/Memberships table. If that leaves visible blank space at the bottom of a row or table on page 1 — the row is taller than its content needs — resize it down rather than leaving the gap: check each row's height (python-docx `row.height` / the `w:trHeight` XML) against the tallest cell's actual content after edits, and reduce a row set to a fixed/exact height that now exceeds what its content needs (switch to `WD_ROW_HEIGHT_RULE.AUTO` or lower the explicit value) so the table hugs its content. Re-render to confirm the gap is gone and no new page-break issue was introduced.

## Shared structure (all three templates)

Only the header's photo/status line, the Executive Summary's localization clause, and the Regulatory Frameworks ordering differ by market — everything else is identical.

### Header

| Target country | Photo | Columns |
|---|---|---|
| Australia | No | 2 (name/tagline/credential \| contact+status) |
| Singapore | No | 2 (name/tagline/credential \| contact+status) |
| UAE | Yes | 3 (photo \| name/tagline/credential \| contact+status) |

Fixed per market, not a per-run judgment call. Status line: Australia states citizenship/visa only ("Australian Citizen"); Singapore and UAE add a work-rights clause ("· Open to relocation, visa sponsorship required," "Singapore PR," "UAE Resident Visa (employer-sponsored, transferable)").

### Executive Summary

One dense paragraph, 4–6 sentences. Bold the load-bearing facts as they appear: years of experience, current employer + a scale metric, current title/scope, key credential(s), one differentiator. Close naming the seniority of stakeholder advised.

For Singapore and UAE, weave in one clause connecting an existing credential/framework to that market's own regulatory guidance (`regulatory-equivalents.md`) — only where genuinely transferable, framed as alignment, never as claimed local experience (Step 4's rule applies here first, since this is the most visible sentence on the page). Write the Singapore-market clause (MAS/PDPC/IMDA) fresh each time from the specific JD — don't reuse the UAE asset file's CBUAE/DFSA/FSRA wording.

### Capability matrix (three-column table)

Heading names the domain (e.g. "[DOMAIN] AND TECHNICAL EXPERIENCE"). The primary JD-keyword-matching surface — reorder/reselect its bullets first in Step 5, since it's what a recruiter or ATS scans first.

- **Column 1** — organisational/employer experience under bold sub-headers, each a short bulleted list.
- **Column 2** — Transformation & Key Programs under bold sub-headers by theme, each bullet ending with `[Employer]`.
- **Column 3** — functional/governance expertise under bold sub-headers by function, ending with Regulatory Frameworks and Stakeholder & Advisory sub-lists.

**Regulatory Frameworks ordering is country-sensitive**: Australia and Singapore lead with Australian frameworks (APRA, CPS 230/234/220, BCBS 239, APS 115, Privacy Act, PGPA Act) before international standards; UAE leads with UAE-market frameworks (CBUAE Responsible AI Guidance, DFSA/FSRA Enabling Technologies Guidelines, UAE AI Charter, PDPL). For a real Singapore JD, replace the ordering with Singapore-market frameworks (MAS, PDPA, MAS FEAT Principles, Model AI Governance Framework) first — the Singapore asset file's Australia-led ordering is a structural placeholder only.

#### Adapting the matrix to the candidate's actual domain

The asset files' heading and sub-headers reflect this user's own domain — AI and data governance in financial services. When the profile being tailored is in a materially different domain, relabel the heading and sub-headers to match its real terminology (e.g. an insurance claims profile: "AI Governance" → "Claims Management," heading → "INSURANCE AND CLAIMS EXPERIENCE").

- Relabel only as necessary — leave labels as-is when the candidate's domain already matches (true for most of this user's own applications).
- Preserve the shape exactly: column count, bold-sub-header-plus-bullets pattern, `[Employer]` tagging in Column 2. Relabeling changes words, never structure.
- A new label must describe a real category already in the master profile — never a category the source CV doesn't support. No genuine content for a domain-appropriate sub-header means dropping it, not inventing bullets to fill it.
- The Regulatory Frameworks and Stakeholder & Advisory sub-lists usually need the same treatment (e.g. insurance regulators/standards instead of AI/banking ones) — again, only ones genuinely held.

### Education, Certifications & Memberships (three-column table)

`Education | Certifications | Memberships & Publications` — identical across all three. No referees section by default. This user has genuine content for all three columns; a different profile may not have memberships or publications — see "No empty sections" below before leaving that column blank.

## No empty sections

No labeled section, column, or sub-header may render empty in the final output — never leave one blank just because this candidate's real content doesn't match the asset file's original categories. This generalizes the capability matrix's relabeling rule (above) to the whole document, most commonly relevant to the Memberships & Publications column.

1. **First choice — repurpose with real content.** Check the master profile for other genuine, ungrouped content that fits the space — professional development, notable projects, volunteering, languages, board or committee positions — and relabel the section to match what's actually going in it (e.g. "Memberships & Publications" → "Professional Development"). The content must already be genuinely in the master profile; this is relabel-and-fill, not licence to invent a membership or publication that doesn't exist.
2. **Last resort — remove the section.** Only if nothing in the master profile can legitimately fill it, remove the section or column entirely rather than showing it empty (e.g. collapse the three-column Education/Certifications/Memberships table to two columns and widen the rest, rather than leave a blank third column). This is a bigger structural change than the usual content edits here, so disclose it to the user (SKILL.md's "never fail silently") rather than doing it silently.

### Professional Experience (reverse chronological)

Bold role title; bold `Employer | Location | *Dates*` line (optional `| *Left: [reason]*` only if genuinely stated, never invented); italic one-sentence scope/mandate line; bullet achievements.

**"Earlier Career" condensing is a length-management tool, not a default.** The asset files condense roles older than roughly a decade into one "Earlier Career (YYYY–YYYY)" block (one short paragraph per employer, not full bullets) because that's what it took to fit their original content on 2 pages — it isn't a rule to always apply. Each run, check against the master profile's actual role list and this run's selected content:

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

This applies only to text actually being composed or rewritten (Executive Summary, Step 4 localization clauses, Step 5's improved bullet phrasing) — never to formatting already in the asset file. The file's own en dashes in date ranges (`2007–2015`) and "·" separators in the capability matrix are the candidate's real, pre-existing style; leave them exactly as they are.

- **Never introduce an em dash (—) in new prose.** Restructure into two sentences, or use a comma, colon, or parenthesis instead. This is the single most common tell that text was AI-written, and the asset files don't use it in their original prose — don't add it while rewriting.
- Avoid stock AI-sounding phrasing: transition filler ("Furthermore," "Moreover," "It's worth noting," "In today's..."), formulaic triads ("not only X, but also Y"), and generic corporate-speak verbs not already in the source CV's own vocabulary ("leverage," "foster," "robust," "seamless," "holistic" used as filler rather than a genuine term the candidate already uses).
- Before finalizing, reread every sentence you composed or rewrote this run specifically for an em dash or this kind of filler, and fix any you find — this is a final check, not just a drafting guideline.

## Per-market notes

- **Australia**: no photo, no DOB, no marital status (`country-rules.md`). 2-page cap; trim earlier-career detail and lowest-relevance matrix bullets first if over length.
- **Singapore**: no photo — resolves what would otherwise conflict with `country-rules.md`, which discourages a photo for Singapore specifically even though it allows one for UAE. No DOB, no marital status. Same cap and trimming priority.
- **UAE**: photo included in the asset file — keep as-is (`country-rules.md` accepts a photo for UAE). Nationality field only as stated, never inferred. No DOB, no marital status. Same cap and trimming priority.
