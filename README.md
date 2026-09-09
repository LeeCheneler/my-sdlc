# My AI-Assisted SDLC

I am a principal developer working across a portfolio of services and more than 20 repositories. My job is not confined to writing code in one codebase: I move between development, technical review, organisation, Jira backlog building, documentation, maintenance, patching and incident follow-up. Quite often, a single outcome crosses several repositories and requires multiple pull requests to land in a particular order.

That shapes how I use AI. I do not treat it as an autocomplete tool attached to one repository, or as an autonomous engineer to which I hand an unconstrained problem. I have built an operating model around it: shared context at the portfolio level, repository-specific working agreements, isolated workspaces, deliberate human approval points and automation for work that should not require repeated manual effort.

The result is less a conventional linear SDLC and more a portfolio-level engineering loop.

## One workspace for the whole portfolio

My local setup is deliberately simple. I have a single `<company>/` directory containing local clones of all the repositories I work with. A `CLAUDE.md` file at that root provides shared context and instructions, and I almost always start Claude Code from there rather than from inside an individual repository.

That choice reflects the actual shape of the work. Cross-repository context is the norm rather than the exception. While changing one service, I may need to inspect another repository for an established pattern, understand a shared contract, coordinate related changes or prepare a chain of dependent pull requests. Starting from the common parent lets one working session operate at that broader level instead of pretending every task has a single-repository boundary.

The root instructions are not a substitute for good judgement, and they need maintenance—the current file is probably due an update. They are useful because they make the default working method available in every session. They encode the stable parts of how I operate so I do not have to restate them each time.

## Worktrees make parallel work routine

When I work on a repository, I use Git worktrees almost exclusively. That isolation matters when several pieces of work are moving at once, especially when one outcome spans multiple repositories.

I have two small utility commands:

- `new-worktree ...` creates and prepares a worktree;
- `rm-worktree ...` cleans it up when the work is finished.

They do more than wrap `git worktree add` and `git worktree remove`. They also automate setup such as dependency installation and symlinking local environment files. This removes the recurring toil that otherwise makes isolated environments feel expensive.

The root Claude instructions describe this worktree flow, so Claude already knows how a piece of work should be created, prepared and eventually cleaned up. Isolation is therefore the default rather than something I need to remember to request.

This is a recurring theme in my setup: if a good practice is important, make it the path of least resistance for both the human and the agent.

## Each repository defines its own development contract

The portfolio-level instructions describe how I work generally. Individual repositories then add their own `CLAUDE.md` files containing the local development loop, commands, conventions and constraints.

The essential loop is:

1. Read the ticket or request and investigate the repository.
2. Present an understanding of the problem and an implementation plan, including the intended commit sequence, for human sign-off.
3. Work through one commit at a time, presenting the completed change before committing it.
4. After the commits are approved, open the pull request.

This is deliberately more controlled than simply asking an agent to “implement the ticket.” The plan checkpoint tests whether the problem has been understood before code is changed. The proposed commit list forces the work into reviewable units. Commit-by-commit approval keeps the implementation legible and gives me opportunities to redirect it before mistakes compound.

The controls work because they are part of a repeated operating discipline, supported by versioned repository instructions, tools and human review.

## AI is involved across the lifecycle, not only in implementation

Coding is only one part of how I use AI. Because I work from the portfolio root, I can use the same environment for much of the surrounding engineering work:

- investigating requests and tracing behaviour across services;
- comparing patterns between repositories;
- decomposing work and preparing Jira backlog items;
- designing ordered changes across several repositories;
- implementing and testing those changes;
- drafting or updating documentation;
- investigating CI failures and preparing fixes;
- auditing maintenance and security work across the portfolio.

When a pipeline fails, I can tell Claude that CI has failed and let it locate the relevant run, or provide the failed run URL directly to save a step. CI diagnosis is particularly well suited to an agent: the evidence is usually available in logs, the repository and the diff, and the proposed correction can go back through the same test and review loop.

The important point is that AI does not sit in a separate “code generation” stage. It helps carry context between discovery, planning, implementation, validation and maintenance.

## Backlog creation is part of the engineering loop

A significant part of my role is managing and shaping the team’s backlog in Jira. I use Claude Code for this too, with access to Jira and Confluence through the Atlassian MCP. Because I still work from the `<company>/` directory, backlog planning can draw on the same portfolio-wide context as implementation: existing Jira and Confluence material, the related repositories, established patterns and the current state of the services involved.

My preference is to organise work into bounded, fully deliverable units—usually epics in Jira. An epic should describe an outcome that can actually be completed, not become a permanent container for loosely related activity. I do not like epics remaining partially complete for long periods, because the backlog gradually becomes cluttered and loses its sense of direction.

The process is collaborative rather than a one-shot generation exercise:

1. I discuss the intended deliverable with Claude and supply the business, Jira or Confluence context it needs.
2. Claude investigates the relevant repositories where technical context is required.
3. We shape a clear, completable epic around the outcome.
4. We ideate on the work beneath it, initially producing only a succinct bullet-point ticket list.
5. We draft the first ticket in detail and iterate until its scope, intent and level of detail are right.
6. Once that establishes the pattern, Claude drafts the remaining tickets in the epic.
7. I review the complete set before it goes to the team for refinement, prioritisation and eventual delivery.

Starting with a compact ticket list matters. It lets me test the proposed decomposition before investing effort in detailed tickets. Perfecting the first ticket then establishes a concrete standard for the rest, allowing Claude to do the repetitive drafting without losing the structure and tone we have agreed.

This is particularly effective for large epics containing substantial but repetitive work. Cyber Essentials and patching work across many repositories, or a portfolio-wide migration from New Relic to Datadog, can require a large number of similarly structured tickets with important repository-specific differences. Claude handles the investigation and drafting overhead, while I retain control of the outcome, decomposition and final backlog. The team still refines the work before delivery; AI removes the administrative weight of getting it to that point.

## Automate the deterministic work first

I have experimented with large collections of skills and elaborate agent setups. I went through the phase of creating dozens of skills and trying to make everything agentic. I have since reduced that to about three, and I rarely need to invoke them explicitly. Models have improved, but the more important lesson is that a small number of clear operating rules and good tools usually beats a large layer of prompt machinery.

I now start by separating two kinds of work:

1. **Deterministic work** that normal software can perform reliably.
2. **Judgement-heavy work** where an AI model can interpret evidence, investigate a codebase or produce a context-sensitive change.

AI should not be inserted merely because it can be. Deterministic code is cheaper to run, easier to test and easier to trust. The useful pattern is to automate collection and control flow conventionally, then introduce AI only where interpretation or implementation genuinely benefits from it.

## Patching is the clearest example

Patching used to consume a meaningful amount of time. As the number of repositories grew, repeatedly checking their security posture, understanding alerts, applying updates and creating pull requests became an increasingly expensive portfolio-wide chore.

Today, most of that process is automated through two workflows.

The first is a deterministic audit. It inspects the organisation’s repositories in GitHub for Dependabot, secret-scanning and vulnerability or code-scanning concerns, then produces a detailed report.

The second is the patching workflow. It takes the work identified by the audit and invokes Claude Code programmatically using `claude -p` to investigate repositories, apply patches and open pull requests.

This division is intentional:

- finding and enumerating alerts is deterministic;
- understanding the repository and producing a suitable change may require judgement;
- tests and CI provide repeatable validation;
- the pull request creates an auditable boundary before integration.

I use my own tool, [Kiri](https://kiri.build), as the harness around these workflows. It gives me a UI for running and following them, while retaining session and workflow history locally in the SQLite database backing my installation. Repeatable tasks become reusable workflows, and their results remain available rather than disappearing with a terminal session.

This has turned patching from a recurring manual sweep into an exception-driven process. Humans spend less time finding routine work and more time deciding what genuinely needs attention.

## Human review is the current boundary

At present, pull-request review and merging are still human-controlled. AI can implement the work, investigate failed checks and prepare follow-up changes, but a person reviews the pull request and decides whether to merge it.

I want to experiment with moving that boundary, particularly for low-risk classes of work. Patching is an obvious candidate, as are small README or documentation-only changes. A possible model is:

1. The author labels a pull request as a candidate for automated review or approval.
2. Automation verifies objective conditions such as change scope, affected paths, test results and repository policy.
3. An AI reviewer classifies the change and examines the diff in context.
4. The system either approves and potentially merges it, or explains why human review is required.

The label would be a nomination, not a bypass. Nor should “CI passed” be the only condition: CI demonstrates that the checks we wrote passed, not that the change is necessarily safe. The decision should also account for the type of change, blast radius, sensitive paths, ownership, reversibility and confidence in the available validation.

The aim is not to remove humans from engineering. It is to reserve human attention for changes where judgement, risk or organisational context makes that attention valuable. Routine, well-bounded changes should not automatically incur the same process cost as architectural or production-sensitive work.

## The operating principles

Several principles underpin the whole approach:

**Work at the level of the outcome.** If an outcome crosses repositories, the agent’s workspace and plan should be able to cross repositories too.

**Encode the development loop near the code.** Shared instructions establish portfolio-wide defaults; repository instructions add the commands and constraints that apply locally.

**Make isolation cheap.** Worktrees and bootstrap scripts remove the cost of creating a clean working environment, so parallelism does not mean state contamination.

**Plan before changing.** Investigation, explicit understanding and an agreed commit sequence come before implementation.

**Keep changes reviewable.** Small, intentional commits provide better human checkpoints and better recovery points for an agent.

**Use deterministic automation wherever possible.** Reserve model calls for tasks that benefit from interpretation, reasoning or codebase-specific implementation.

**Treat AI as a participant in the control system, not the control system itself.** Instructions, permissions, tests, CI, pull requests and risk-based review remain essential.

**Optimise human attention rather than maximising autonomy.** The goal is not the largest possible number of agent actions. It is to reduce toil while preserving—or improving—engineering confidence.

## What my SDLC actually is

My SDLC is therefore not “ask Claude to write some code.” It is a structured loop for operating across a large repository estate:

- maintain portfolio-level context;
- investigate at the right scope;
- shape outcomes into bounded epics and coherent tickets;
- refine and prioritise that work with the team;
- isolate each unit of work;
- agree the plan and commit structure;
- implement with progressive human checkpoints;
- validate with repository tooling and CI;
- use AI to diagnose failures and iterate;
- integrate through pull requests;
- automate recurring portfolio maintenance;
- retain the history and reusable process locally;
- gradually move low-risk work toward policy-driven, agent-assisted approval.

The AI model is an important component, but it is only one component. The larger productivity gain comes from the system around it: the filesystem layout, instructions, worktrees, scripts, deterministic audits, workflows, approval points and retained context. Together, those let me operate coherently across many repositories without treating every task as a fresh conversation or every service as an island.
