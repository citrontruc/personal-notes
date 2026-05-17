# Claude code

## Table of content

- [Claude code](#claude-code)
  - [Table of content](#table-of-content)
  - [Models](#models)
    - [Tips on claude](#tips-on-claude)
  - [Claude.md](#claudemd)
  - [Skills](#skills)
    - [What is it?](#what-is-it)
    - [How to use them?](#how-to-use-them)
    - [Additional skill informations](#additional-skill-informations)
    - [Order of precedence](#order-of-precedence)
    - [Entreprise](#entreprise)
    - [Plugins](#plugins)
  - [Subagents](#subagents)
  - [Hooks](#hooks)
  - [MCP servers](#mcp-servers)
  - [MCP](#mcp)
    - [Tools: Model-Controlled](#tools-model-controlled)
    - [Resources: App-Controlled](#resources-app-controlled)
    - [Prompts: User-Controlled](#prompts-user-controlled)
  - [Example prompts](#example-prompts)
    - [Architecture](#architecture)
    - [Specification](#specification)
    - [Code quality](#code-quality)
    - [Onboarding](#onboarding)

## Models

- Opus: highet level of intelligence. Also most expensive. Good for long running tasks. Good for advanced software.
- Sonnet: middle.
- Haiku: less expensive, less performant.

### Tips on claude

We have some functions to help with the generation of formatted content. We can add an assistant message to define the beginning of the answer and we can ask to stop the message whenever a specific sequence is generated (stop_sequence). For example, you can define as an assitance message of \```json and stop sequence on a ```. Frackly, it seems you are better of using langchain.

## Claude.md

file loaded everytime when you make a request. It concerns your personal preferences.

## Skills

### What is it?

- Skills are folders of instructions that Claude Code can discover and use to handle tasks more accurately. Each skill lives in a SKILL.md file with a name and description in its frontmatter
- Claude uses the description to match skills to requests. When you ask Claude to do something, it compares your request against available skill descriptions and activates the ones that match
- Personal skills go in ~/.claude/skills and follow you across all projects. Project skills go in .claude/skills inside a repository and are shared with anyone who clones it. Correct directory is .claude/skills/your folder name/SKILL.md
- Skills load on demand — unlike CLAUDE.md (which loads into every conversation) or slash commands (which require explicit invocation), skills activate automatically when Claude recognizes the situation
- If you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written

IMPORTANT: when creating a skill, write a name and description describing what to do and when to use it. These are loaded when you make a request to Claude and lets him choose the most appropriate behaviour. You get a confirmation prompt before Claude loads the full skill content into context.

### How to use them?

- Personal skills go in ~/.claude/skills (your home directory). These follow you across all your projects — your commit message style, your documentation format, how you like code explained.
- Project skills go in .claude/skills inside the root directory of your repository. Anyone who clones the repo gets these skills automatically. This is where team standards live, like your company's brand guidelines, preferred fonts, and colors for web design.

### Additional skill informations

Allowed-tools to specify what claude can use and model to specify which model to use when reasoning. In allowed-tools don't forget to say "read" when claude does not have the right to modify.

- scripts/ — Executable code
- references/ — Additional documentation
- assets/ — Images, templates, or other data files

You can add assets and references folder and write linkss to these files in your claude description. Progressive disclosure: keep SKILL.md under 500 lines and link to supporting files (references, scripts, assets) that Claude reads only when needed

### Order of precedence

In case of conflicts in skill, here is the priority list Entreprise > Personal > Project > plugins. Write explicit names if you have different needs for similar looking actions.

### Entreprise

You can have entreprise wide skills and share skills with your whole company.

### Plugins

Plugins are shared skills. They are accessible through the marketplace.

## Subagents

Subagents run in a separate context. They receive a task, work on it independently, and return results. They're isolated from the main conversation. SubAgents don't automatically see skills. In order to let your agent access your skills, add a skills field in your agent description with the skills you want to use. Only custom subagents can access skills.

## Hooks

Hooks when you need to schedule a task to run only on specific triggers. Skills can be used in hooks.

## MCP servers

Are for integration with other tools.

Hooks are defined in Claude settings files. You can add them to:

- Global - ~/.claude/settings.json (affects all projects)
- Project - .claude/settings.json (shared with team)
- Project (not committed) - .claude/settings.local.json (personal settings)

Hooks are mostly used to describe rules before using tools for example to exclude files that your agents should not read / manipulate.

There are way to describe behaviours pre-tool use and after tool use. Here is an example:

```json
"PreToolUse": [
  {
    "matcher": "Read",
    "hooks": [
      {
        "type": "command",
        "command": "node /home/hooks/read_hook.js"
      }
    ]
  }
]

"PostToolUse": [
  {
    "matcher": "Write|Edit",
    "hooks": [
      {
        "type": "command", 
        "command": "node /home/hooks/edit_hook.js"
      }
    ]
  }
]
```

## MCP

### Tools: Model-Controlled

Tools are controlled entirely by Claude. The AI model decides when to call these functions, and the results are used directly by Claude to accomplish tasks.

Tools are perfect for giving Claude additional capabilities it can use autonomously. When you ask Claude to "calculate the square root of 3 using JavaScript," it's Claude that decides to use a JavaScript execution tool to run the calculation.

### Resources: App-Controlled

Resources are controlled by your application code. Your app decides when to fetch resource data and how to use it - typically for UI elements or to add context to conversations.

In our project, we used resources in two ways:

- Fetching data to populate autocomplete options in the UI
- Retrieving content to augment prompts with additional context

Think of the "Add from Google Drive" feature in Claude's interface - the application code determines which documents to show and handles injecting their content into the chat context.

### Prompts: User-Controlled

Prompts are triggered by user actions. Users decide when to run these predefined workflows through UI interactions like button clicks, menu selections, or slash commands.

Prompts are ideal for implementing workflows that users can trigger on demand. In Claude's interface, those workflow buttons below the chat input are examples of prompts - predefined, optimized workflows that users can start with a single click.

## Example prompts

### Architecture

```claude.md
# CLAUDE.md

## Overview
[Project name] - Modular Monolith Web API

## Tech Stack
- .NET 10, ASP.NET Core Minimal APIs
- EF Core 10 with PostgreSQL (snake_case naming convention)
- FluentValidation for request validation
- Result<T> pattern for error handling (no exceptions for business logic)
- JWT Authentication with Refresh Tokens
- OpenTelemetry for tracing and metrics
- Serilog for structured logging
- Swagger/OpenAPI for API documentation
- Docker for containerization

## Architecture
- Modular Monolith with Vertical Slice Architecture
- Clean Architecture: domain is isolated from infrastructure
- Each module: Domain, Core, Infrastructure, PublicApi projects
- CQRS with manual handlers (no MediatR, no interfaces)
- Manual mapping with extension methods

## Project Structure
- src/Modules.[Name].Domain/ - Entities, value objects, enums
- src/Modules.[Name].Core/ - Endpoints, handlers, validators
- src/Modules.[Name].Infrastructure/ - DbContext, EF config, migrations
- src/Modules.[Name].PublicApi/ - Cross-module contracts
- src/Modules.Common.API/ - Shared endpoint abstractions, error handling
- src/Modules.Common.Domain/ - Result<T>, IHandler, IEvent interfaces
- src/ModularMonolith.Host/ - Program.cs, DI composition

## File Naming
- Features/[UseCaseName]/[UseCaseName].Endpoint.cs
- Features/[UseCaseName]/[UseCaseName].Handler.cs
- Features/[UseCaseName]/[UseCaseName].Validator.cs
- Features/[UseCaseName]/[UseCaseName].Mapping.cs (if 2+ mappings)
- Features/Shared/Routes/RouteConsts.cs
- Features/Shared/Errors/[Module]Errors.cs

## Code Conventions
- Positional records for request/response DTOs
- File-scoped namespaces
- Primary constructors for dependency injection
- Sealed classes for implementations
- Internal by default, public only for contracts

## Patterns We Use
- Result<T> pattern for all handler return types
- IHandler marker interface for auto-registration
- IApiEndpoint for endpoint registration
- RouteConsts for centralized route definitions
- FluentValidation validators per use case
- Bogus for test data seeding
- EF Core DbContext directly in handlers

## Patterns We Do NOT Use
- Repository pattern
- AutoMapper or any mapping library
- MediatR or any mediator library
- Exceptions for business logic flow
- [FromServices] attribute

## Testing
- xUnit for test framework
- Moq for mocking
- Testcontainers for integration tests (PostgreSQL)
- Respawn for database cleanup between tests
- NetArchTest.Rules for architecture tests
- Test naming: [Method]_[Scenario]_[ExpectedResult]

## DI Registration
- Each module: Add[Module]Module(services, configuration)
- Auto-scan handlers: RegisterHandlersFromAssemblyContaining
- Auto-scan validators: AddValidatorsFromAssembly
- Auto-scan endpoints: RegisterApiEndpointsFromAssemblyContaining
```

### Specification

```claude.md
Based on the architecture we designed, create a detailed Backend
Implementation Specification as a Markdown file.

The specification must include:

1. Entity Definitions
   - All entities as C# classes with property types
   - Value objects and enums
   - Entity relationships

2. Database
   - EF Core mapping configuration for each entity
   - EF Core Migrations
   - Seeding strategy with sample data using Bogus

3. API Endpoints
   - For each endpoint: HTTP method, route, request body,
     response body with C# record definitions
   - Business rules and validation rules per endpoint
   - Error scenarios and expected error responses

4. Authentication and Authorization
   - Which endpoints require authentication
   - Which endpoints require specific policies or roles
```

### Code quality

```claude.md
Review the generated code with three passes:

Pass 1 - Code Quality:
- Are naming conventions consistent across all files?
- Are all handlers following the same pattern?
- Are there any unused imports or dead code?
- Is the code DRY without being over-abstracted?

Pass 2 - Performance:
- Are there any N+1 query issues in EF Core calls?
- Are there any unnecessary database round-trips?
- Are all async methods properly awaited?
- Are CancellationTokens passed through the entire call chain?

Pass 3 - Security:
- Are all endpoints properly authorized?
- Are all user inputs validated before processing?
- Are there any SQL injection or XSS vulnerabilities?
- Are sensitive data fields excluded from responses?
- Are sensitive data fields excluded from logging?

For each issue found, show the file, the problem, and the fix.
```

### Onboarding

```claude.md
You are onboarding me to this repository.

Tasks:
- Identify the top-level architecture (apps/services/libraries), and what each does.
- Produce a directory map: top 10 directories with responsibilities.
- Identify key runtime boundaries: API layer, domain layer, persistence, async jobs, config.
- Show a dependency diagram (Mermaid) using the repo’s actual module/package boundaries.
- Cite exact files for each claim (paths + brief evidence).

Constraints:
- Do not edit files.
- Prefer reading docs first (README, AGENTS.md, CLAUDE.md, docs/, CONTRIBUTING, ADRs) and then code.
- If the repo is a monorepo, explain the workspace/tooling setup.

Diagram:
Create a mermaid diagram with your findings
```
