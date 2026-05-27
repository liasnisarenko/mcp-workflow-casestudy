# Daily 9 AM Standup: An Unattended Scheduled Workflow

The standup runs every morning at 9 AM Pacific. It produces a one-page brief I read with coffee. No manual trigger. Built on a Claude scheduled task with MCP access to Asana, Notion, my calendar, and a baseline notes file.

This document covers the workflow, the prompt, the trigger setup, and the design notes that explain why it's shaped this way.

---

## What the standup produces

A single message in my inbox at 9:01 AM with five sections:

1. **Top 5 tasks for today**, ranked by critical-path impact across all active projects.
2. **New since yesterday** - anything added to Asana or the personal-baseline notes file in the last 24 hours.
3. **Blockers** - tasks waiting on a third party, with the third party named.
4. **Calendar** - every meeting today, with the relevant prep-doc link.
5. **Sleep on it** - any task I've moved 3+ times. Either commit or kill.

About 200-300 words. Long enough to be useful, short enough to read in 90 seconds.

---

## The prompt

```
You are running my morning standup. Today is [Date].

Pull from these sources:

1. Asana
   - All projects I own, status not "Done"
   - For each task: title, project, due date, current owner, last update
   - Tag tasks as: today (due today), this-week (due in next 7 days),
     overdue (past due date), waiting (status = "Waiting on" or owner = not me)

2. Notion - "Daily Anchor" database
   - Yesterday's entry: priorities, what shipped, what got stuck
   - This week's themes
   - Personal-baseline file - what's on the meta-list that hasn't moved to Asana

3. Google Calendar
   - Every meeting today
   - For each meeting: title, time, attendees, location/link
   - Cross-reference each meeting against Notion docs - if a prep doc exists
     for this meeting, link it.

4. Diff against yesterday's standup (saved at /standups/)
   - Anything new in the task list that wasn't there yesterday
   - Anything that moved off the list since yesterday

Compose the brief in this exact format:

# Standup [Date]

## Top 5 today
1. [Task] - [Project] - [why this is top 5: critical-path / due today / blocking N other tasks]
2. ...

## New since yesterday
- [Task] (Asana, added [time]) - [Project]
- [Anchor entry] - [Notion]

## Blockers
- [Task] - waiting on [Person/Vendor] since [date]

## Calendar today
- [Time] [Meeting title] - prep: [link if exists]

## Sleep on it
- [Task moved 3+ times] - commit by [date] or kill

Save the brief to /standups/[Date].md before sending it to me.

Safety rails:
- Read-only across all platforms. Do not modify any task, doc, or event.
- If you can't find a source (Notion is down, Asana token expired, etc.),
  produce the brief from what you have and flag the missing source in a
  "Missing sources" section at the top.
- If you find more than 20 overdue tasks, do not list them all. Flag the
  count and recommend a triage session.
```

---

## The trigger setup

The prompt runs as a scheduled task in Claude with this cadence:

- **Cron:** `0 9 * * *` (every day at 9 AM Pacific)
- **Mode:** Unattended
- **Output:** Delivered to the Claude inbox (which sends a notification)
- **Failure handling:** If MCP connections fail, the task retries once with a 60-second delay. If the second attempt fails, it sends a "Standup failed" notification with the error message and skips the day rather than producing a misleading brief.

---

## Design notes

### Why diff against yesterday

The single most valuable line in the brief is "new since yesterday." It catches tasks I added in a hurry the night before, anything a collaborator pushed to me overnight, and the slow accretion of work I would otherwise miss. The diff is cheap and high-signal.

### Why "Sleep on it" exists

Tasks that have been rescheduled 3+ times are almost never going to get done in their current form. The brief surfaces them with a forced choice: commit a real date, or kill the task. This single section dropped my open-task count by ~40% in the first month.

### Why read-only is enforced explicitly

The standup runs unattended. If it ever modified an Asana task or a Notion doc, I might not notice until much later. The "read-only" instruction is enforced in the prompt and verified against the trigger config. Even if the model decides to be helpful and edit something, the trigger config doesn't grant write scopes for some platforms.

### Why "if you can't find a source, flag it"

The previous version of the standup would silently produce a brief with missing data. I'd see "no blockers" and trust it - when actually Asana had been down and there were three blockers I hadn't accounted for. The current version produces a brief with whatever sources are available and explicitly flags which sources failed. Trust the brief or fix the source - never trust a silent omission.

### Why the brief is short

Two reasons. First, attention - I read it once, fast, in the morning. A long brief gets skimmed. Second, the act of writing it short forces the agent to make ranking decisions. "Top 5" is a much more useful instruction than "list everything." The brief is shorter than my old morning routine and more action-driving.

---

## What this workflow demonstrates

If you're hiring someone to build AI workflows for your business, this is the kind of work I do day-one:

- Compose multiple data sources into a single useful artifact.
- Make the artifact short and ranked, not exhaustive.
- Run the workflow unattended with graceful failure handling.
- Enforce read-only where the workflow doesn't need writes.
- Surface omissions explicitly instead of hiding them.

It's not glamorous. It's the kind of small, reliable thing that compounds.
