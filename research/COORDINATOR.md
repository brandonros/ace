**Use one execution API. My recommendation is to keep your existing workspace API and add a small coordinator around it.** Adopt `claude-code-openai-wrapper` as its replacement only if its compatibility and session behavior are better enough to justify migrating your existing integration.

You do not need both.

**Popular agent frameworks**

These are prominent options, with approximate GitHub stars observed now—not a ranking of production adoption.

| Framework | Stars | What it provides |
|---|---:|---|
| **[CrewAI](https://github.com/crewAIInc/crewAI)** | 59.7k | Role-based agent teams plus explicit, event-driven workflows called Flows. [GitHub](https://github.com/crewAIInc/crewAI?utm_source=chatgpt.com) |
| **[LangGraph](https://github.com/langchain-ai/langgraph)** | 43.2k | Stateful graphs, branching, checkpoints, and human intervention. Can orchestrate calls to an existing agent API. [GitHub](https://github.com/langchain-ai/langgraph?utm_source=chatgpt.com) |
| **[Agno](https://github.com/agno-agi/agno)** | 42.7k | Agent SDK, service runtime, and management interface. [GitHub](https://github.com/agno-agi/agno?utm_source=chatgpt.com) |
| **[Mastra](https://github.com/mastra-ai/mastra)** | 28.8k | TypeScript agents and workflows, including branching, parallel execution, and suspend/resume. Relevant to your Node stack. [GitHub](https://github.com/mastra-ai/mastra?utm_source=chatgpt.com) |
| **[Pydantic AI](https://github.com/pydantic/pydantic-ai)** | 20.6k | Typed Python agents, structured outputs, tools, and graph capabilities. [GitHub](https://github.com/pydantic/pydantic-ai?utm_source=chatgpt.com) |

Also relevant: **Google ADK** and **Microsoft Agent Framework**. Be careful with older AutoGen recommendations: its repository now says maintenance mode and directs new users to Microsoft Agent Framework. [GitHub](https://github.com/google/adk-python?utm_source=chatgpt.com)

**For your workflow coordinator, I would choose this:**

| Decision | My recommendation |
|---|---|
| Execution service | **Your existing Claude workspace API** |
| Trigger | **Keep the existing cron** |
| Coordination | **A small module in your existing Node service**, or one Python script if that is easier to maintain |
| State | Explicit run records; SQLite is a reasonable starting point for a single service instance |
| Verification | Existing CI plus task-specific checks |
| Agent framework | **None initially** |
| Durable workflow engine, if needed | **Temporal** |

The coordinator needs a few ordinary operations:

```text
discover eligible tickets
claim a work item
start or reconnect to an agent run
collect its result and verification
request a bounded repair or human clarification
record the outcome in Jira
```

**Claude already supplies your coding agent.** You have not described a missing capability that requires constructing another agent system in CrewAI, Pydantic AI, or Agno.

LangGraph or Mastra could implement that coordinator without replacing Claude. But I would introduce one only if its branching, checkpoints, or interactive workflow features remove code you otherwise need to maintain.

**If you want a standard coordinator instead of owning recovery machinery, my choice is Temporal.** Your difficult boundaries are Jira, remote execution, CI, retries, and human waits—exactly the durable workflow concerns Temporal addresses. Its service is an additional operational dependency, so I would use it immediately if your company already runs it; otherwise I would first establish that those problems justify introducing it. [docs.temporal.io](https://docs.temporal.io/ai?utm_source=chatgpt.com)

GitHub Actions is a reasonable home when most coordination follows repository and CI events. For your current Jira poll calling an existing internal service, moving the same script into Actions is optional.

**The next useful addition is a reliable `start → observe → resume/stop → verify` contract around one execution API. Another agent framework is not a prerequisite.**