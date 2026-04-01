# AI Skills Repository

A collection of reusable AI agent skills, instructions, and shared knowledge for GitHub Copilot and Claude. Each skill packages domain expertise, best practices, and step-by-step workflows so AI assistants produce consistent, high-quality output across projects.

---

## Repository Structure

```
ai-skills/
├── skills/                  # Reusable AI skills, organized by domain
│   ├── backend/
│   │   ├── dotnet/          # code-review, generate-crud, refactor-clean-architecture, write-unit-test
│   │   └── node/
│   ├── frontend/
│   │   ├── angular/
│   │   └── nextjs/          # generate-api-route, generate-page, refactor-component, sso-with-google
│   ├── cross/               # Language-agnostic skills
│   │   ├── code-review/
│   │   ├── create-user-story/
│   │   ├── debug/
│   │   └── explain-code/
│   └── workflow/            # Multi-step process skills
│       ├── build-feature/
│       ├── fix-bug/
│       └── full-code-review/
├── agents/                  # Agent configurations
│   ├── claude/              # Claude system prompts and templates
│   └── copilot/             # GitHub Copilot agent configs
├── shared/                  # Reference knowledge consumed by skills
│   ├── architecture/        # clean-architecture, microservices, monolith
│   ├── best-practices/      # error-handling, logging, validation
│   └── coding-standards/    # angular, dotnet, general, nextjs
├── schemas/                 # JSON/YAML schemas for skill validation
├── sdk/                     # Tooling for authoring and testing skills
└── examples/                # Example prompts and usage demos
```

---

## Available Skills

### Cross-platform

| Skill | Trigger | Description |
|-------|---------|-------------|
| `create-user-story` | Writing a Jira user story ticket | Creates a well-structured Jira user story following Product Team best practices — user perspective, value hypothesis, acceptance criteria |
| `code-review` | Reviewing code quality | — |
| `debug` | Diagnosing a bug | — |
| `explain-code` | Explaining what code does | — |

### Backend — .NET

| Skill | Description |
|-------|-------------|
| `code-review` | .NET-specific code review |
| `generate-crud` | Generate CRUD endpoints for a resource |
| `refactor-clean-architecture` | Refactor code to follow Clean Architecture layers |
| `write-unit-test` | Write unit tests for a class or method |

### Frontend — Next.js

| Skill | Trigger | Description |
|-------|---------|-------------|
| `sso-with-google` | Google SSO / NextAuth | Implement, fix, or review Google SSO using NextAuth.js v4 |
| `generate-api-route` | Creating a Next.js API route | — |
| `generate-page` | Creating a Next.js page | — |
| `refactor-component` | Refactoring a React component | — |

### Workflow

| Skill | Description |
|-------|-------------|
| `build-feature` | End-to-end workflow for building a new feature |
| `fix-bug` | Structured workflow for diagnosing and fixing a bug |
| `full-code-review` | Multi-pass comprehensive code review |

---

## Skill File Format

Each skill lives in its own folder as a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: skill-name
description: "When to use this skill. USE FOR: ... DO NOT USE FOR: ..."
---

# Skill Title

[Step-by-step instructions for the AI agent]
```

The `description` field is the **discovery surface** — the agent reads it to decide whether to load the skill. Use clear trigger phrases in `USE FOR`.

---

## How to Use a Skill

### GitHub Copilot

Copy the skill folder to `.github/skills/<skill-name>/SKILL.md` in your project, or reference it from your `copilot-instructions.md`.

### Claude

Use skills as system prompt additions or include them in the `agents/claude/` configuration.

---

## Adding a New Skill

1. Create a folder under the appropriate domain: `skills/<domain>/<skill-name>/`
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`) and step-by-step instructions
3. Reference shared knowledge from `shared/` where applicable
4. Register the skill in `index.yaml`

### Skill Quality Checklist

- [ ] `name` matches the folder name
- [ ] `description` includes clear `USE FOR` and `DO NOT USE FOR` triggers
- [ ] Instructions are written as directives to the AI, not to the human
- [ ] Includes a concrete example or output template
- [ ] No project-specific secrets or hardcoded paths

---

## Shared Knowledge

The `shared/` folder contains reference documents that skills can import for consistent standards:

- **`architecture/`** — Clean Architecture, Microservices, Monolith patterns
- **`best-practices/`** — Error handling, Logging, Input validation
- **`coding-standards/`** — Language and framework conventions (Angular, .NET, Next.js, General)
