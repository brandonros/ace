**No—you do not need all those as separate components. I made a small change sound like an entire platform.** Here is what I actually mean.

**“Claude workspace API” was my shorthand for the service you described:** your internal API that invokes Claude with your repositories, skills, agents, and tools. It is not a product name or a standard API.

**Your API and `claude-code-openai-wrapper` are alternatives to each other for exposing Claude execution.** They are not necessarily interchangeable: the wrapper would still need access to your workspace and configuration. There is no reason to replace your API just to run scheduled work.

Your concern about cost is the important one. These are different designs:

| Current pattern you described | Change I recommend |
|---|---|
| Cron invokes Claude’s `/thing` skill | Cron runs ordinary application code |
| Claude reads Jira and determines whether anything needs doing | Application code queries Jira and checks explicit eligibility |
| Claude reconciles progress on every invocation | Application code checks recorded run/PR/CI status |
| Agent spends tokens even when little has changed | Claude runs when there is actionable work |

**Keep the cron. Change what the cron invokes.**

For example, conceptually:

```python
for ticket in jira.find_automation_tickets():
    state = read_recorded_state(ticket)

    if state == "ready":
        start_claude(ticket, skill="do-my-ticket")

    elif state == "running":
        refresh_execution_status(ticket)  # ordinary API call

    elif state == "waiting_for_ci":
        check_ci_status(ticket)           # ordinary API call

    # Waiting for review, blocked, or finished: no Claude call.
```

That ordinary code **is what I meant by coordination**. It can live in your existing cron script or Node service. It does not require a new server, framework, or UI.

The skill can still research Jira and make implementation decisions **after it has been assigned work**. It doesn’t need to be the routine scheduler and status checker too.

**You also do not automatically need SQLite.** If Jira already records enough structured state—queued, running, blocked, PR link, execution ID—use that for the first version. Add another store when you have a concrete requirement Jira cannot handle adequately. Free-text comments alone are harder to interpret reliably without an LLM.

So my actual recommendation is just:

1. **Existing cron:** wakes up periodically.
2. **A little ordinary code:** checks Jira and execution status, then decides whether to invoke or continue Claude.
3. **Existing API and skill:** perform the engineering work.

A webhook could replace some polling later. It won’t fix waste if every event still triggers a full agent investigation.

**If your current cron already performs those checks in ordinary code, you already have the coordinator.** The next question is why it invokes Claude repeatedly—not which orchestration product to add.

**The “application” would be a small script you write, running in the container you already have.** I wasn’t referring to another product.

And your clarification matters: **you already have orchestration—it is implemented as instructions inside the giant skill.** Claude interprets those instructions every time the cron fires.

You have two implementation choices:

| Approach | Where the workflow lives | What happens every 15 minutes |
|---|---|---|
| **What you have** | Markdown skill interpreted by Claude | Claude queries Jira, examines state, decides what to do, and executes |
| **What I proposed** | A Python/Node script, plus the coding skill | The script queries Jira, checks explicit conditions, and invokes Claude when needed |

Concretely, your existing cron would invoke something like:

```bash
python /app/poll_jira.py
```

That file would contain ordinary code using Jira’s API and your existing agent API. **Someone has to extract those rules from the skill and implement them. There is no missing application you were supposed to know about.**

For example:

| Instruction currently inside the skill | Where I would put it |
|---|---|
| “Find tickets with label X and status Y” | Script calling Jira |
| “Skip tickets already running or awaiting review” | Script checking recorded state |
| “Read the ticket and find the relevant implementation” | Claude skill |
| “Implement the change and investigate test failures” | Claude skill |
| “Stop after two repair attempts” | Enforcement in code |
| “Record the execution ID and resulting PR” | Code handling the result |

**I would not rewrite the whole skill.** The smallest useful change is a filter in front of it:

1. Query Jira using ordinary code.
2. If there are no eligible tickets, exit without calling Claude.
3. For an eligible ticket, invoke the existing skill with that specific ticket.

That avoids paying an agent to repeatedly discover that nothing needs doing. It does **not** eliminate repeated investigation inside an active task; improving that requires recording progress and continuing the appropriate session.

Your current design may have been a reasonable way to prototype quickly. But encoding predictable scheduling and bookkeeping in a giant prompt means paying the model to interpret those rules repeatedly. **Extract the repetitive, mechanical parts when their cost or unreliability becomes material. Keep the engineering judgment in the skill.**