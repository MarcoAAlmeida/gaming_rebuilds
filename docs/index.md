# Gaming Rebuilds

This site is a knowledge base for a set of game rebuilds built using agentic
frameworks. It is also an onboarding resource for developers on this team who
are starting to work with AI agents and Claude Code.

If you have never built with agents before, start here.

---

## What Is an Agent?

An agent is a program that perceives its environment, reasons about what to do,
and takes action — then repeats. What makes it an agent rather than a script is
that it decides its own next step based on what it observes, rather than
following a fixed sequence of instructions.

You are already using one. Claude Code is an agent. When you give it a task, it
does not just generate text — it reads files, runs commands, edits code, checks
output, and decides what to do next. The loop continues until the task is done
or it needs to ask you something. That is the agent loop.

The infrastructure that makes this possible is being standardized by the
[Agentic AI Foundation (AAIF)](https://github.com/aaif), a Linux Foundation
project. The key protocol is [MCP (Model Context Protocol)](https://modelcontextprotocol.io),
an open standard that defines how agents connect to tools, data sources, and
external services. When Claude Code reads your filesystem, queries a database,
or calls an API, it does so through MCP-compatible tool definitions.

---

## Communicating with Agents — AGENTS.md

Agents are powerful, but they do not know your project. They cannot infer your
build system, your test strategy, your branching conventions, or which files
should never be touched. Without guidance, they make reasonable guesses —
which are often wrong.

**AGENTS.md** is the solution. It is a Markdown file you place at the root of
a repository to give any agent persistent, project-specific instructions. Think
of it as a README written for the agent rather than for a human developer.

A good AGENTS.md answers the questions an agent would otherwise have to guess:

- How do I install dependencies? (`pnpm install`, not `npm install`)
- How do I run the tests? (`./scripts/test.sh --watch=false`)
- What should I never modify? (`src/generated/`, `*.lock` files)
- What conventions matter? (branch naming, commit message format, PR checklist)

The format is intentionally simple. One concrete command beats three paragraphs
of explanation. Use direct, imperative language: *always run the linter before
committing*, *ask before modifying the database schema*, *never force-push to
main*.

The [AGENTS.md specification](https://github.com/agentsmd/agents.md) is an open
format supported by Claude Code, Copilot, Cursor, and other coding agents.
Augment Code has published a practical guide on
[how to write an effective AGENTS.md](https://www.augmentcode.com/guides/how-to-build-agents-md).

The `AGENTS.md` at the root of this repository is the shared knowledge base for
all rebuilds. Each individual rebuild is expected to have its own `AGENTS.md`
that extends it with game-specific context.

---

## Communicating with LLMs — llms.txt

Agent instructions live in repositories. But what about documentation sites,
APIs, and external projects? When an agent needs to understand a library or
service it has not seen before, it has to read the docs — and documentation
sites were designed for humans, not language models.

**llms.txt** addresses this. It is a lightweight convention: place a Markdown
file at the root of your documentation site that distills the content into a
form a language model can actually use. The format includes a brief summary of
what the project is, followed by a structured list of documentation sections
with one-sentence descriptions and links.

For large documentation sets, a companion `llms-full.txt` includes the complete
content of all pages in a single flat file — suitable for feeding directly into
a large context window.

The analogy is intentional: just as `robots.txt` tells search engine crawlers
what to index, `llms.txt` tells language models what your project contains and
how to navigate it. The specification is maintained at
[llmstxt.org](https://llmstxt.org/). A community-maintained directory of sites
that have adopted the standard is at
[llms-txt-hub](https://github.com/thedaviddias/llms-txt-hub).

If your team owns any documentation site, adding `llms.txt` is a low-effort
contribution that makes your docs useful to agents.

---

## Getting Started

Practical steps, in order:

1. **Install Claude Code** — [docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code)
2. **Add `AGENTS.md` to your repositories** — use the [AGENTS.md in this repo](AGENTS.md) as a starting template
3. **Run Claude Code on a real task** — read its reasoning, not just its output; understanding how it thinks is the fastest way to learn to work with it effectively
4. **Add `llms.txt` to any documentation site you own** — follow the spec at [llmstxt.org](https://llmstxt.org/)

---

## Rebuilds

The game rebuilds in this repository use agentic frameworks to model game logic,
turn management, and AI opponents as multi-agent systems. They are also
working examples of how to structure an agent-driven project.

| Game | Description | Status |
|---|---|---|
| [Karriers](rebuilds/karriers.md) | Carrier-based strategy game rebuild | In Progress |

---

## Further Reading

- [Agentic AI Foundation (AAIF)](https://github.com/aaif) — Linux Foundation project standardizing agentic AI infrastructure, including MCP and the AGENTS.md format
- [llms-txt-hub](https://github.com/thedaviddias/llms-txt-hub) — community directory of projects that have published llms.txt files
- [llmstxt.org](https://llmstxt.org/) — the llms.txt specification
- [How to build AGENTS.md](https://www.augmentcode.com/guides/how-to-build-agents-md) — practical guide from Augment Code
- [AGENTS.md specification](https://github.com/agentsmd/agents.md) — the open format reference
