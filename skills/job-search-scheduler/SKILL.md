---
name: job-search-scheduler
description: One-time setup helper that creates the recurring Cowork Scheduled Task which runs the job-search-digest skill automatically, always inside a dedicated Cowork Core project (never local or empty) so it keeps running in the cloud regardless of whether the computer is on. Requires job-search-digest's config.md and cv-profile.md to already exist in Drive — never gathers that configuration itself, since duplicating it would let the two skills drift apart; tells the user to run job-search-digest once first if either is missing. Checks Drive for a marker showing setup is already done and does nothing if so. Needs Cowork's scheduling capability active, which chat mode doesn't have even though chat and Cowork share the same window — but should still trigger from a plain chat turn, to redirect the user to switch modes. Use whenever the user wants to automate their job search or asks to "set up scheduling". Never use to run a search — that's job-search-digest — or tailor a single CV — that's cv-jd-tailor.
---

# Job Search Scheduler

A one-time setup skill, not a recurring one itself. Its only job is to create the Cowork Scheduled Task that will call `job-search-digest` on a cadence, inside a dedicated Cowork Core project. Once that task exists, this skill's work is done, and it should refuse to run again unless the existing task is genuinely gone.

## Step 1 — Check whether this is already set up

Read Drive for `Job Search/scheduler-status.md`. If it exists and says `Scheduled: yes`: tell the user plainly that a recurring task already exists — name it, give its cadence, project, and creation date — and stop there. Do not create a duplicate. If the user wants to change the existing schedule, direct them to Cowork's own Scheduled tab to edit or delete it directly; this skill only ever creates a task when none exists. Only proceed past this step if the marker file doesn't exist.

## Step 2 — Confirm this session can actually schedule something

Task creation depends on Cowork's own scheduling capability, which plain chat mode doesn't have — even though chat and Cowork now share the same window and the same message box. Still trigger on this request even from a plain chat turn; the point of catching it here is to redirect the user, not to stay silent.

If Cowork's scheduling capability isn't active in this exact session: don't try to work around it, and don't fabricate a "scheduled" confirmation. Instead:

1. **Tell the user plainly** that this specifically needs Cowork — and that it's a quick switch, not a separate app or a lost conversation. On desktop and web, that's the Cowork option next to the message box, in the same window; on mobile, it's the Cowork entry in the app's sidebar.
2. **Give the exact next action**: switch to Cowork there and ask the same thing again — e.g. "set up my recurring job search."
3. **Offer the manual alternative** for anyone who'd rather not switch right now: `configure.html` (from this same package).

Stop here in that case — don't proceed to Step 3 until Cowork's scheduling capability is confirmed active in this session.

## Step 3 — Confirm the prerequisites exist

Google Drive must be connected. If it isn't, say so and stop.

Check Drive for job-search-digest's `config.md` and `cv-profile.md`. **Both must already exist.** A scheduled run has no one to answer setup questions, so setup has to be done beforehand — and duplicating job-search-digest's own config-gathering logic here would let the two skills drift out of sync, so this skill never attempts it. If either file is missing: tell the user plainly to run job-search-digest once first (e.g. just ask it to search for jobs), which builds and saves both, then come back and run this skill again. Stop here in that case.

Once both exist, this skill only needs two things of its own — nothing job-search-digest's config already covers:
1. **Task name** — free text, default "Job Search Digest" if the user doesn't specify one.
2. **Cowork Core project** — mandatory (Step 4 explains why). Check whether the user already has a suitable one; reuse it if so. Otherwise, plan to create one in Step 4.

Don't ask for cadence — `config.md` already has it (set during job-search-digest's own Step 1); read it from there rather than asking again.

## Step 4 — Create the scheduled task inside a Cowork Core project

**Always use a Cowork Core project — never leave the project/folder field empty, and never choose a local folder.** This is a reliability requirement, not a preference: a local-folder or empty setting only runs while the computer is on, awake, and Cowork is open; a Core project runs in Anthropic's cloud regardless. A product that's supposed to remember what it's done can't silently skip days because a laptop was asleep.

If reusing an existing project (Step 3), use it as-is. If creating one, name it to match the task (e.g. "Job Search Digest") and set its **Instructions** field to:

```
This project runs a recurring, Drive-backed job search pipeline using the
job-search-digest, cv-jd-tailor, and job-search-scheduler skills. Persistent
state (config, CV profile, dedup log, tailored CVs, digest archive) lives
under the "Job Search/" folder in Google Drive — check there before asking
the user anything already saved. Never create a second scheduled task for
this pipeline if one already exists.
```

The task's own prompt can now be minimal, since config and profile are already guaranteed to exist in Drive by Step 3:

```
Run the job-search-digest skill.
```

Create the task using this session's own scheduling capability (`/schedule`, or describing it directly) with: the Step 3 task name, the cadence read from `config.md`, this minimal prompt, and the Core project from above. If Cowork's own flow prompts interactively for any of these mid-creation, answer with these values rather than re-asking the user.

Before moving to Step 5, confirm the task was actually created — check whatever confirmation this session's flow provides — and confirm it's genuinely running inside a Core project, not a local folder or empty setting. If creation failed, its success can't be confirmed, or it ended up outside a Core project, say so plainly and stop — do not proceed to Step 5.

## Step 5 — Record that this is done

Only once creation is genuinely confirmed, including that it's inside a Core project, write `Job Search/scheduler-status.md` to Drive:

```markdown
# Job Search Digest — Scheduler Status
- Scheduled: yes
- Task name: <name>
- Cadence: <daily | weekly>
- Project: <Core project name>
- Created: <date>
```

This is exactly what Step 1 checks on any future invocation — it's what makes running this skill twice safe rather than duplicative.

## Failure handling — never fail silently

Say so explicitly and stop, without writing the Step 5 marker, whenever: Drive isn't connected, `config.md` or `cv-profile.md` don't exist yet, this isn't a Cowork session or its scheduling capability isn't available, task creation fails, its success can't be confirmed, or it would end up running outside a Core project. Never write `Scheduled: yes` on anything less than full, confirmed success — a false positive here would permanently block a real setup from ever happening on a later run.
