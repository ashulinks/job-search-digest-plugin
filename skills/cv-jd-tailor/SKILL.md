---
name: cv-jd-tailor
description: Tailor an existing CV to a specific job description for roles in the UAE, Australia, or Singapore — matching keywords, reordering achievements, and scoring the result against a rubric. Use this whenever the user provides (or references) an existing CV plus a job description/JD and wants it tailored, improved, scored, or checked for a specific role in one of these three countries. Also use when the user asks to "improve my CV," "rate my CV," "match my CV to this job," or wants their citizenship/work-rights status correctly reflected on a CV for UAE/Australia/Singapore. Do not use for CVs targeting any other country — flag that explicitly and stop rather than improvising rules for an unsupported market.
---

# CV/JD Tailor

Tailors an existing CV to a job description for UAE, Australia, or Singapore, preserves the user's own template/format and its tone and voice, states citizenship and work-rights status correctly, and scores the result against an explicit rubric before committing to producing it. Visually checks the rendered output — never just the edited XML — before scoring or delivering. Never fabricates experience. Never fails silently.

## Supported scope — hard boundary

Covers **UAE, Australia, and Singapore only**. For any other target country: stop before producing a tailored CV, explain that country-specific conventions (format norms, legal disclosures, visa framing) aren't covered, and offer a generic keyword/structure-only pass with no country-specific claims only if the user explicitly accepts that reduced scope.

## Step 1 — Gather required inputs

Do not proceed until all of these are confirmed. Ask for anything missing — never guess or infer silently. **Whenever a question has a bounded set of likely answers, ask it as clickable options (e.g. via the AskUserQuestion tool) rather than open-ended free text** — the user picks instead of typing. Reserve plain free-text questions for genuinely unbounded input (the CV itself, the JD text, the specific wording of a work-rights status):

1. **Existing CV** (file or pasted text — free text/upload, not a click choice). Gather this explicitly the first time you tailor a CV for someone in this conversation/session — never assume it. The three files under `assets/templates/` are format/structure references only; even though they contain real CV content, that content goes stale and must never stand in for asking. Once the user has supplied a CV and Step 2 has built a master profile from it, that stored profile — not the template file — is what later runs reuse.
2. **Job description** (file, pasted text, or URL — free text/upload).
3. **Target country**: offer UAE / Australia / Singapore as clickable options.
4. **Citizenship and current work rights/visa status** in the target country. Offer the country's common statuses as clickable options where `references/country-rules.md` gives them (e.g. for Australia: "Australian citizen" / "Permanent resident" / "Requires sponsorship"; adapt per target country), plus free text for anything more specific. Gating input, not optional. Never assume "likely a citizen" or "probably needs sponsorship"; an incorrect assumption here is a legal-exposure risk.
5. **Output template**: offer "Use the default template for [target country]" vs. "I'll provide my own template" as clickable options.

## Step 2 — Build or reuse the master profile

Reuse an existing structured master profile for this user in this conversation/session (roles, dates, achievements, skills, education, citizenship) if one exists — update it only with genuinely new information, never reparse from scratch. Otherwise, parse the CV supplied in Step 1 into that structure. The profile's content always comes from a CV the user actually supplied, never from a template's own baked-in content. All selection and reordering happens against this structure, not raw CV prose, or JD-matching quality will degrade across repeated runs.

## Step 3 — Apply country rules and select the template

Read `references/country-rules.md` for the target country: work-rights disclosure requirements, format/length conventions, and what's legal vs. prohibited to include. **2-page cap regardless of country.** Place the work-rights/citizenship disclosure where the user's template specifies, or in the header/contact block per the country's convention.

Use `assets/templates/CV_Template.docx` — the single standard template shared by all three countries, per `references/templates.md` — unless Step 1 already established a user-supplied template. This is the literal starting document for docx output, always: never regenerate from scratch (e.g. with docx-js) when the asset exists — edit a copy of it in place per that file's instructions, replacing every bracketed placeholder with the candidate's real content.

## Step 4 — Localize regulatory and standards references

Read `references/regulatory-equivalents.md`. Where the master profile's regulatory/standards experience is named in a different jurisdiction's terms than the target country's, surface the genuine equivalence explicitly (e.g. APRA prudential experience → analogous to CBUAE's framework) so a local reviewer recognizes it immediately — international standards (ISO/IEC 42001, etc.) are named as-is everywhere. **Never claim direct experience with a local regulator, law, or standard the candidate hasn't actually worked with** — frame it as transferable, not as if it were local experience. A JD-named local requirement with no real transferable basis is a genuine gap Step 5 must surface, not something to bridge with invented equivalence.

## Step 5 — Match against the JD, then gate on projected score before generating anything

Extract the JD's explicit requirements (must-have skills, keywords, years of experience, seniority signals) and check each honestly against the master profile: genuinely met, partially/transferably met, or not met. Do this analysis before touching any output document. Using `references/rubric.md`, project the achievable score on both axes (JD/keyword match; CV quality) from this analysis alone, assuming no fabrication — don't round up because the candidate "seems senior enough."

**Gate:** if either projected score is below **8/10**, stop here without editing or generating the document. Report the projected scores, what's driving them, and the gap list, then ask whether the user wants the full CV produced anyway (some do, for a deliberate stretch application) — offer this as clickable options ("Produce it anyway" / "Stop here"), not open-ended text. Only continue below if both axes project to 8+, or the user confirms they want it regardless — this avoids spending effort producing a polished document for a fit that was never going to be strong.

If continuing:
- Select, reorder, or rewrite existing achievements from the master profile to surface the strongest match — never write in achievements that aren't in the profile. Improve phrasing/specificity of buried or under-quantified real achievements.
- Match the template's existing tone and voice exactly when rewriting anything — see `references/templates.md`'s "Tone and voice," including its rule against em dashes and other AI-writing tells in any text you compose or rewrite this run (existing formatting in the asset file, like its date-range en dashes and "·" separators, is untouched). Don't introduce a different register, sentence structure, or vocabulary than what's already on the page.
- If the candidate's real domain differs materially from the asset file's own domain (AI/data governance in financial services), relabel the capability matrix's heading and sub-headers to match — see `references/templates.md`'s "Adapting the matrix to the candidate's actual domain." Relabel only as necessary, preserve the table's structure exactly, and never imply a category of expertise the profile doesn't genuinely support.
- **No section may render empty.** If the candidate's real content doesn't fill a labeled section, column, or sub-header (most often Memberships & Publications), repurpose it with other genuine master-profile content first, or remove it as a last resort — see `references/templates.md`'s "No empty sections." Never leave one visibly blank.
- **Never fabricate**: no invented skills, employers, dates, certifications, or metrics. An unaddressed JD requirement stays unaddressed.
- Preserve a user-supplied template's format exactly — no silent template, font, or structure changes.
- Enforce the 2-page cap; if trimming is needed, cut the weakest/least-relevant content first and disclose what was cut and why. Condensing older roles into "Earlier Career" is one trimming lever, not a default — only use it if full-detail roles wouldn't fit; if they do fit, expand them and drop that block, see `references/templates.md`'s "Professional Experience."

## Step 6 — Visual QA pass

Never treat the edited docx as finished straight off the XML edit. Render it to PDF and look at every page — see `references/templates.md`'s "Visual QA before finalizing" for the render command and what to check. The specific failure this catches: a role's title, employer/dates line, or scope sentence stranded alone at the bottom of a page while its bullets start on the next one — a company name must never be the last line on a page, separated from its responsibilities. Also check for a page 1 table (capability matrix or education table) left with visible empty space because a row is taller than its content needs after edits — resize the row down to fit rather than leaving the gap, see `references/templates.md`'s "Page 1 table with empty space." Fix any page-break, table-split, widow, formatting-residue, or oversized-row issue found, then re-render to confirm the fix held and the 2-page cap still holds, before moving to Step 7.

## Step 7 — Score against the rubric

Reached only if Step 5's gate cleared or was overridden, and Step 6's visual QA is clean. Score the actual produced document — not the Step 5 projection — on both rubric axes. Report both sub-scores (never blended), what's driving each, the gap list, and any drift from the Step 5 projection. Never inflate a final score to avoid an awkward conversation; if a strong score isn't achievable without fabrication, say so plainly (Step 5's gate should already have caught this).

## Failure handling — never fail silently

Say so explicitly, at every step, whenever something can't be done as requested — never produce a degraded result without comment. Covers: missing inputs, an unsupported country, a template that can't be preserved as specified, content cut to fit 2 pages, unmatched JD requirements, a regulatory claim that would require fabrication, stopping at Step 5's gate instead of generating a CV, a page-break/formatting defect caught in Step 6 that trimming can't cleanly fix, removing a section/column rather than leaving it empty, and any assumption made about ambiguous input — including treating a template's baked-in content as the user's current CV instead of asking.

## Reference files

- `references/country-rules.md` — per-country work-rights disclosure, format/length, and legal/prohibited content rules
- `references/templates.md` — the single standard template shared by all three countries, its structure, tone/voice, domain-filling rules, and the asset-file editing workflow
- `references/regulatory-equivalents.md` — regulatory/standards mapping across AU, UAE, and Singapore for genuine transferable-experience framing
- `references/rubric.md` — the two-axis scoring rubric and the Step 5 score gate
