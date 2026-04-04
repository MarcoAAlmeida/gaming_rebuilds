# AGENTS.md — Knowledge Base for Agentic Systems

This document captures best practices, patterns, and guidelines for building agents
in this project. It serves as the shared knowledge base across all rebuilds.

---

## 1. What Is an Agent?

An agent is an autonomous unit that perceives its environment, reasons over it,
and takes actions to achieve a goal. In the context of this project, agents are
built using agentic frameworks (e.g., Claude Agent SDK, LangGraph, CrewAI) and
operate over game-domain tasks.

Key properties of a well-designed agent:
- **Goal-directed**: every action traces back to a defined objective
- **Observable**: the agent logs its reasoning and decisions
- **Bounded**: the agent knows what it should and should not do
- **Composable**: agents can be combined into multi-agent pipelines

---

## 2. Agent Design Principles

### 2.1 Keep Tools Focused

Each tool the agent uses should do exactly one thing. Avoid multipurpose tools
that mix concerns — they are harder to test, debug, and reason about.

```python
# Good: single-responsibility tool
def get_player_position(player_id: str) -> dict:
    ...

# Avoid: tool doing too much
def manage_player(player_id: str, action: str, value: Any) -> dict:
    ...
```

### 2.2 Prefer Declarative Goals Over Procedural Scripts

Give agents goals, not step-by-step procedures. Agents reason best when they
understand the *desired outcome* rather than a fixed sequence of steps.

### 2.3 Trust but Verify Tool Outputs

Never assume tool calls succeed. Always handle errors explicitly and design
agents to retry or escalate when they encounter unexpected results.

### 2.4 Minimal Authority Principle

Grant each agent only the tools and permissions it needs for its specific task.
An agent responsible for reading game state should not have write access.

---

## 3. Multi-Agent Patterns

### 3.1 Orchestrator / Subagent

An orchestrator agent breaks down a high-level task and delegates to specialized
subagents. Subagents report results back; the orchestrator synthesizes them.

```
Orchestrator
├── ResearchAgent   (reads game knowledge base)
├── PlannerAgent    (builds turn strategy)
└── ExecutorAgent   (applies actions to game state)
```

### 3.2 Parallel Fan-Out

When subagents are independent, run them concurrently. Merge results after all
complete.

```
Orchestrator ──┬── AgentA (concurrent)
               ├── AgentB (concurrent)
               └── AgentC (concurrent)
               └── [merge results]
```

### 3.3 Human-in-the-Loop

For irreversible or high-stakes actions, pause and request human confirmation
before proceeding. Design the pause point explicitly in the workflow.

---

## 4. Context and Memory

### 4.1 Context Window Discipline

- Keep system prompts concise and focused — every token costs reasoning capacity
- Pass only relevant context to subagents, not the entire parent context
- Summarize long histories before passing them downstream

### 4.2 Memory Tiers

| Tier | Mechanism | Use Case |
|---|---|---|
| In-context | Messages in current window | Short-term reasoning |
| Scratchpad | Tool writes to temp file/store | Inter-step state |
| Persistent | Database / vector store | Long-term knowledge |
| External | Git, files, APIs | Source of truth |

### 4.3 State Passing

Prefer explicit state passing over implicit shared globals. Each agent invocation
should receive everything it needs as input and return everything relevant as output.

---

## 5. Tool Design for Agents

### 5.1 JSON-Serializable I/O

All tool inputs and outputs must be JSON-serializable. Avoid returning objects
that cannot be represented in a tool result.

### 5.2 Idempotent Tools Where Possible

Design tools to be safe to call multiple times with the same arguments. This
makes retry logic safe.

### 5.3 Typed Schemas

Always declare full input schemas (using Pydantic or JSON Schema). This prevents
hallucinated parameters and enables static validation.

```python
class MovePlayerInput(BaseModel):
    player_id: str
    direction: Literal["north", "south", "east", "west"]
    steps: int = Field(ge=1, le=100)
```

### 5.4 Fail Loudly

Return structured error objects rather than silently failing or returning
empty results. Include an `error` field and a `message` explaining what went wrong.

---

## 6. Prompting Agents

### 6.1 System Prompt Structure

A good agent system prompt includes:
1. **Role** — who the agent is and what it's responsible for
2. **Context** — relevant background (game state, rules, constraints)
3. **Goal** — what success looks like
4. **Tools** — brief description of available tools (the model sees schemas, but
   a one-line summary in the prompt helps)
5. **Constraints** — what the agent must not do

### 6.2 Avoid Ambiguity

Vague instructions produce inconsistent behavior. Replace:

> "Handle the player's turn"

With:

> "Given the current game state, choose exactly one action from the available_actions
> list. Call the execute_action tool with your chosen action. If no valid actions
> exist, call report_stalemate."

### 6.3 Chain-of-Thought for Complex Reasoning

For tasks requiring multi-step reasoning (planning, strategy), instruct the agent
to reason before acting. Extended thinking or explicit scratchpad use improves
decision quality.

---

## 7. Evaluation and Testing

### 7.1 Unit Test Tools Independently

Every agent tool must have unit tests that run outside of any agent loop. Test
both happy paths and edge cases.

### 7.2 Trace Agent Runs

Log the full sequence of tool calls, inputs, and outputs for every agent run.
This is essential for debugging unexpected behavior.

### 7.3 Regression Tests via Fixtures

Record representative agent runs as fixtures. Run them as regression tests to
catch model behavior changes across updates.

### 7.4 Evals Over Examples

Use structured evaluation sets (input → expected outcome) rather than manual
spot-checking. Automate scoring where possible.

---

## 8. Safety and Guardrails

### 8.1 Output Validation

Validate agent outputs before acting on them. Never pass raw model output directly
into system calls, databases, or other agents without sanitization.

### 8.2 Action Reversibility

Prefer reversible actions. Before implementing irreversible operations, require
explicit confirmation (human or higher-trust agent).

### 8.3 Scope Limiting

Define explicit allowed/denied action sets per agent. Reject any action that
falls outside the defined scope, even if the model requests it.

### 8.4 Rate Limiting and Cost Controls

Set maximum token budgets and tool call limits per agent run. Unbounded loops
are a real risk in agentic systems.

---

## 9. Project-Specific Conventions

### 9.1 Rebuild Structure

Each game rebuild lives in `rebuilds/<game-name>/` as a git submodule.
The rebuild must include its own `AGENTS.md` extending this base document
with game-specific agent context.

### 9.2 Shared Tooling

Common tools shared across rebuilds (e.g., logging, state serialization)
are maintained in this meta-repository and imported by each rebuild.

### 9.3 MkDocs Documentation

All design decisions, agent architectures, and lessons learned are documented
in `docs/` and published via GitHub Pages (MkDocs Material theme).

---

## 10. References and Further Reading

- [Anthropic Agent SDK documentation](https://docs.anthropic.com)
- [Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents)
- [Claude Code AGENTS.md convention](https://docs.anthropic.com/en/docs/claude-code/memory)
- [LangGraph documentation](https://langchain-ai.github.io/langgraph/)
- [Evaluating LLM Agents](https://arxiv.org/abs/2310.07019)
