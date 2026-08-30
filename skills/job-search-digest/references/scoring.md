# Quick Relevance Scoring

Purpose: a cheap 0–10 pre-filter run against every deduped posting, so the expensive cv-jd-tailor rubric (which renders and visually QAs a document) only runs on postings genuinely worth that effort. This is deliberately looser than cv-jd-tailor's two-axis rubric — it's a triage tool, not a final verdict.

## What to check against the master profile

Score each of the four factors 0–2.5 and sum for a 0–10 total:

1. **Title/seniority match** — does the posting's title and seniority level (Head of / Director / Manager / IC) align with the profile's target level? A senior-titled posting the profile is under-levelled for, or an IC-titled posting the profile is over-levelled for, scores low here even if the domain matches.
2. **Domain match** — does the posting's actual subject matter genuinely match the role keyword(s) the user gave in Step 1 of SKILL.md, not just a shared word? A posting that shares a keyword but sits in a clearly different function scores low (e.g. the user searched "Head of Risk" but the posting is a risk-adjacent individual-contributor role with no leadership scope). Read this from the user's own stated keywords each run — this skill has no fixed domain, so never assume one.
3. **Industry fit** — does the posting's industry match an industry genuinely present in the master profile (built from the user's actual CV in Step 2)? Score highest where it matches an employer's industry already in the profile or a closely adjacent one; moderate for a plausible transition; low for an unrelated industry. Read this from the profile each run — never assume a fixed industry list.
4. **Explicit requirement overlap** — scan the posting's stated must-haves (certifications, years of experience, named frameworks/regulations) against the profile and count genuine overlaps, not just keyword presence.

## Thresholds (see SKILL.md Step 5)

- **≥7**: strong match — proceed to full cv-jd-tailor tailoring.
- **5–6**: worth a look — surface in the digest with the score and a one-line reason, no tailored CV produced.
- **<5**: drop from the digest entirely, but still log it in the seen-jobs log so it's never re-scored or re-surfaced.

## Notes

- This score is never shown as if it were cv-jd-tailor's rubric score — label it clearly as a "relevance score" in the digest so the two aren't confused.
- If cv-jd-tailor's own gate (Step 5 of that skill) later projects below 8/10 for a posting that scored ≥7 here, that's expected and fine — the two checks measure different things (this one is breadth/fit triage, cv-jd-tailor's is achievable-document-quality). Report both outcomes honestly rather than reconciling them.
- Don't round up borderline scores because a posting "seems senior enough" — the whole point of this step is to protect the user's attention and the tailoring budget from weak fits.
