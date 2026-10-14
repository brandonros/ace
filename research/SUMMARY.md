I read the README and all six files under `research/`. **The main gap is a short, authoritative “where we landed and what to investigate next.”** You’ve preserved the conversation, including recommendations that changed as you clarified the situation.

Here’s what I would add or correct, in priority order.

**1. Put the actual objective at the top of the README**

[ENGINEERING_JUDGEMENT.md](https://github.com/brandonros/ace/blob/master/research/ENGINEERING_JUDGEMENT.md) captures the most important finding, but the README still introduces this primarily as command-center research.

The objective is:

> Increase completed engineering work per human hour by reducing the intervention required for selected workflows. Determine which decisions can be automated, how results are verified, and where exceptions require an engineer.

That distinction determines whether tomorrow should be spent on a UI, workflow code, test infrastructure, or task selection.

**2. Add a “current understanding” section that resolves the contradictory advice**

[COORDINATOR.md](https://github.com/brandonros/ace/blob/master/research/COORDINATOR.md) recommends a coordinator and SQLite. [JIRA.md](https://github.com/brandonros/ace/blob/master/research/JIRA.md) contains the later clarification: orchestration already exists inside the skill, and another database is optional.

Record these explicitly:

| Subject | Current understanding |
|---|---|
| Execution API | We want one: the existing service **or** a replacement based on the wrapper |
| Orchestration | Already implemented substantially inside the giant skill |
| Cron | Not identified as the problem |
| Ordinary workflow code | Proposed way to extract mechanical work from the skill; not yet proven necessary |
| SQLite / Temporal / agent framework | Options, not selected dependencies |
| Open WebUI | An interactive interface candidate; not required for background execution |
| Cost of repeated reconciliation | Suspected overhead that needs measurement |

Some of the confusion came from my earlier answers. Don’t preserve those as equally current architectural decisions.

**3. Add evidence from the actual system**

This is the largest practical omission. There are extensive product references, but no concrete example of your existing execution path.

Tomorrow, capture:

- The current cron command or trigger.
- The relevant parts of the giant skill.
- One example API request and response.
- One successful run and two unsuccessful runs.
- Where each run spent tokens, elapsed time, and human attention.

Without that, we are still designing around a description.

**Do not assume the giant skill is expensive merely because it is large.** Measure the idle reconciliation cost, repeated research, failed attempts, and review effort separately. Extracting bookkeeping into code might save little if implementation retries dominate spending.

**4. Define what “without an engineer” actually means**

The judgment document explains the principle, but the target boundary remains unspecified:

| Outcome | Has the system completed its assigned responsibility? |
|---|---|
| Research findings posted | For investigation tasks, possibly |
| Draft PR opened | Implementation proposed |
| Required checks pass | Defined verification completed |
| PR merged | Integration decision made |
| Change deployed and observed | Operational outcome evaluated |

Your boss might mean the last two while your skill targets the second. **That mismatch would explain dissatisfaction even if the skill performs exactly as instructed.**

Write down the intended endpoint, required evidence, and who owns exceptions.

**5. Add the missing large-project problem: integration**

Your notes cover running separate tickets better than delivering one large project.

For larger work, somebody or something must handle:

- Breaking the project into tasks with stable interfaces.
- Ordering dependencies between tasks.
- Detecting overlapping or incompatible changes.
- Testing the combined changes.
- Revisiting plans when an earlier assumption fails.

**Five individually passing PRs can still produce an incorrect combined system.** More isolated workspaces do not solve that. This deserves an explicit open question, given your boss’s ambition.

**6. Give tomorrow a bounded investigation**

I would make the next actions:

1. Trace one real run from cron through Jira to PR.
2. Annotate the skill’s steps: **mechanical rule / model judgment / verification / external action**.
3. Identify the largest observed source of human effort or failure.
4. Choose one change that addresses it.
5. State what result would justify keeping that change.

For example: “Test whether a plain Jira eligibility check avoids enough unnecessary agent invocations to justify maintaining it.” That is an experiment, not a commitment to build a coordinator platform.

Finally, a few documentation fixes:

- Link the research files from the README, with a suggested reading order.
- Mark saved assistant responses as research notes, with dates and unverified claims.
- The README mixes `/v1/...` provider-style endpoints with application-specific `/api/...` routes; your longer dated note already explains this more accurately.
- Fix “Coding harnessess.”
- The AutoGen maintenance claim in `COORDINATOR.md` links to Google ADK; link to the AutoGen repository instead.

**I would stop expanding the product catalog for now. The most useful addition is a one-page handoff connecting the business objective to an observed failure in your current system and the next experiment.**