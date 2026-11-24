---
name: file-findings
description: Investigate a task, write the findings up, file a report ticket with a tracker, and break out the top N findings as related tickets
user-invocable: true
argument-hint: [top-n | verified | all] [task...]
---

The user wants an investigation turned into tracked work: one report ticket that holds the findings and a tracker, and one ticket for each selected finding, related back to the report.

`$ARGUMENTS` carries the selection and the task. A leading number N selects the top N findings (default 10). `verified` selects every finding you re-read yourself. `all` selects every finding. The rest names the task: a review, an audit, a comparison, a bug hunt. With no task, use the investigation already in this conversation. If there is none, ask what to investigate and stop.

File in the tracker this session can write to: a connected tracker tool first (Linear, Jira), else `gh issue` when the repository is on GitHub. With neither, say so and stop after Step 2.

Invoking this skill is the request to file. File only what it describes.

## Step 1 — Investigate

Skip this step when the findings are already in the conversation.

Read; do not change the code under review. Work from a named commit so every line reference stays true.

Fan out to subagents only when the user asked for agents. Give each a disjoint scope, the evidence rules below, and what is already known or already fixed, so it does not report it again. A subagent's report arrives as confident as a finding and may be wrong: re-read the code behind the top findings yourself before you rank them.

Every finding carries:

- **Where** — `path:line` at the named commit.
- **Mechanism** — what executes, how often, and why that costs or breaks.
- **Smallest fix** — the least change that removes it. Name any standing decision it would overturn.
- **How to confirm** — the test, measurement or reproduction that would prove it.
- **Status** — `Checked` (you read the cited code and it holds), `Agent` (a subagent traced it, you did not re-read it), or `Inferred` (say which part nobody confirmed).

Keep the suspects you examined and cleared. They save the next reader the same search.

## Step 2 — Write the findings document

One markdown file, outside the repository under review unless told otherwise. It contains, in order:

1. How the work was done and what was not built, run or measured.
2. A summary a reader can stop after.
3. The ranked "do these first" table.
4. Every finding, grouped by area, each with a stable ID: an area letter and a number (`H1`, `M3`). An ID is never renumbered. It is the join key for every step below.
5. What was checked and found fine.
6. Limits: what was not examined, and whose reading each unconfirmed claim is.

## Step 3 — Look before filing

Search the tracker for tickets that overlap each selected finding. A finding an existing ticket already covers gets no new ticket. It is recorded against that ticket in the tracker table.

Pick the team or project that plainly owns the work; ask only when two would fit. Use labels that already exist. Set no assignee, priority or estimate, and mention nobody, unless the user asked.

## Step 4 — File the report ticket

The description is a summary, not the document: what was found, the "do these first" table, a placeholder for the tracker, and the limits. State in the first paragraph what was not run.

Attach the findings document twice where the tracker allows: as a native document a reader can open in place, and as the markdown file. Relate the report to the overlapping tickets found in Step 3.

## Step 5 — File the finding tickets

One ticket per selected finding. They are independent, so send them in one batch.

The title states the problem as a sentence, not the fix and not the finding ID. The body stands alone for a reader who has not seen the report:

- A first line naming the finding ID, the report ticket, and the commit the paths refer to.
- **Problem** — what goes wrong and who notices.
- **Where** — the references.
- **Proposed change** — the smallest fix, and what it would overturn.
- **How to confirm** — or how to reproduce.
- **Notes** — the status from Step 1 in plain words, what was not run, and related findings.

Relate each ticket to the report ticket. Use a plain relation, not a sub-issue, unless the user asked for sub-issues: the report should be closable without waiting on every fix. Add a blocked-by link where one fix has to land before another is worth measuring.

A ticket claims only what its status supports. An `Agent` or `Inferred` finding says so in its notes.

## Step 6 — Close the loop

Replace the placeholder in the report ticket with the tracker: one row for every finding in the document, selected or not.

```
| ID | Finding | Rank or status | Ticket |
```

The ticket column holds the new ticket, `Existing ticket <id>`, `Folded into <id>`, or `Not ticketed`. End with a totals line, and count it from the table rather than from memory: new, existing, folded and not ticketed must sum to the number of findings.

Write the same ticket ids into the findings document and attach the updated file in place of the stale one. When a finding also appears in an earlier report's tracker, update that tracker too.

## Step 7 — Report

Tell the user:

- the report ticket, with its link;
- the new tickets as a table of finding ID, ticket and title;
- which findings were left without a ticket, and why;
- every choice made on their behalf: the team, the relation type, the labels, and how an ambiguous selection was read;
- any number you stated earlier that the final count corrected.
