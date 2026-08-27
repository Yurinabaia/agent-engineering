---
name: prime
description: Primes the agent with deep codebase understanding by analyzing structure, documentation, and key files. Use when starting work on a codebase, at the beginning of a session, or when you need a fast orientation before planning or implementing. Optionally pulls external task context from Jira issues and Confluence pages first.
argument-hint: [jira-issue-keys] [confluence-page-ids]
---

# Prime: Load Project Context

## Objective

Build comprehensive understanding of the codebase by analyzing structure, documentation, and key files. If external task references are provided, load them first so the codebase analysis is anchored to the actual work.

## Process

### Step 0: Load External Context

**Run this step BEFORE the codebase analysis.** It accepts optional arguments: `[jira-issue-keys] [confluence-page-ids]`.

- Jira keys may be a single key (`PROJ-12`) or comma-separated (`PROJ-12,PROJ-13`).
- Confluence page ids are numeric page ids.

**If Jira issue keys are provided:**

1. Call `mcp__atlassian__getAccessibleAtlassianResources` to obtain the `cloudId`.
2. For each Jira key, call `mcp__atlassian__getJiraIssue` with that `cloudId`, the issue key, and `responseContentFormat: "markdown"`.
3. Treat the returned issue summary, description, and acceptance criteria as the task context for everything that follows.

**If Confluence page ids are provided:**

1. Call `mcp__atlassian__getConfluencePage` for each page id with `contentFormat: "markdown"` (use the `cloudId` from above, fetching it via `mcp__atlassian__getAccessibleAtlassianResources` if it was not already retrieved).
2. Treat the returned page content as supporting context (specs, design docs, requirements).

**If no arguments are provided:** Skip this step entirely and proceed to Step 1.

Briefly summarize any external context loaded before continuing — this frames the rest of the priming.

### 1. Analyze Project Structure

List all tracked files:
!`git ls-files`

For a .NET solution, map the projects and the real dependency direction:

```bash
find . -name "*.sln" -maxdepth 3
find . -name "*.csproj" -not -path "*/bin/*" -not -path "*/obj/*"
grep -h "ProjectReference" **/*.csproj
```

The `ProjectReference` graph is the proof of which layer depends on which — compare it with the
layer rules in `CLAUDE.md` and report any violation.

### 2. Read Core Documentation

- Read CLAUDE.md (global rules — layers, conventions, error contract, workflow)
- Read README files at project root and major directories
- Read `.claude/references/clean-architecture-dotnet.md` for the per-layer templates
- Read `features/INDEX.md` to see which features are open and in which state

### 3. Identify Key Files

Based on the structure, identify and read:
- Composition root (`Program.cs`) and DI registration (`DependencyInjection.cs` in Application
  and Infrastructure)
- Project files (`*.csproj`) — target framework, packages, project references
- Persistence: `Data/ApplicationDbContext.cs`, `IRepository`, `IUnitOfWork`
- Error contract: `Filter/ErrorHandlingFilterAttribute.cs` + `Erros/ErrosException.cs`
- **One complete vertical slice** (entity → dto → mapper → service → controller → tests) as the
  template for everything you will write later

For work scoped to the API only, prefer the `prime-backend` skill — it loads this same slice
without the rest of the repo.

### 4. Understand Current State

Check recent activity:
!`git log -10 --oneline`

Check current branch and status:
!`git status`

## Output Report

Provide a concise summary covering:

### External Task Context (if loaded)
- Jira issue(s): key, title, one-line goal, acceptance criteria
- Confluence page(s): title and what they specify

### Project Overview
- Purpose and type of application
- Primary technologies and frameworks
- Current version/state

### Architecture
- Overall structure and organization
- Key architectural patterns identified
- Important directories and their purposes

### Tech Stack
- Languages and versions
- Frameworks and major libraries
- Build tools and package managers
- Testing frameworks

### Core Principles
- Code style and conventions observed
- Documentation standards
- Testing approach

### Feature Pipeline
- Open features in `features/INDEX.md`, their state and the next command for each

### Current State
- Active branch
- Recent changes or development focus, recent migrations
- Any immediate observations or concerns

**Make this summary easy to scan - use bullet points and clear headers.**
