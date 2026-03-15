---
name: readme-creator
description: Generate a professional README.md for any project. Use this skill whenever the user wants to create, rewrite, or improve a README file, or says things like "crea un README", "genera el README", "hazme un README", "escribe el README", "write a README", "create a README", "generate a README", "update the README", "mejora el README". This skill supports two modes — writing from a user description, or analyzing the project to auto-generate it. Always use this skill even if the user doesn't explicitly say "README" but asks for project documentation, such as "documenta el proyecto", "document this project", "explain this project", or "quiero documentar esto".
---

# readme-creator

Generate polished, comprehensive README.md files for any project. Supports two operating modes that can be combined:

1. **Description Mode** — the user tells you what the project is about
2. **Analysis Mode** — you read the project's source code, config files, and structure to infer what it does

---

## Code Conventions

- **README content language**: Match the user's language. If they prompt in Spanish, write the README in Spanish. If in English, write in English. Ask if unsure.
- **Code blocks** in the README (install commands, usage examples) should always use the actual project's syntax, regardless of language.
- **File/folder names** referenced in the README must exactly match the real project names.

---

## Mode 1: From User Description

Use this mode when the user provides a verbal description of the project, such as:
> "Crea un README que explique que este proyecto contiene skills reutilizables para distintos proyectos siguiendo buenas prácticas"

### Step 1: Capture the User's Intent

Listen to what the user says and extract:

1. **Project purpose** — what the project does, who it's for
2. **Key features** — main capabilities or value propositions
3. **Tone** — formal/technical, friendly/casual, corporate, open-source community
4. **Audience** — developers, end users, clients, open-source community

### Step 2: Ask Clarifying Questions (if needed)

If the user's description is vague or missing critical details, ask **at most 3 focused questions**. Examples:

- "¿Quieres que el README incluya instrucciones de instalación o es más un documento descriptivo?"
- "¿El público objetivo son desarrolladores internos del equipo o es un proyecto open source?"
- "¿Hay alguna tecnología principal que quieras destacar?"

If the user gave a clear, complete description, **skip the questions and proceed directly**.

### Step 3: Check for Existing Project Context

Even in Description Mode, quickly scan the project root for context that enriches the README:

```
Look for:
- package.json / composer.json / pyproject.toml / Cargo.toml  → tech stack, scripts, name, version
- .env.example                                                 → configuration reference
- LICENSE                                                      → license type
- Existing directory structure                                 → architecture overview
```

Use any findings to complement the user's description, but **the user's words take priority** over inferred details.

### Step 4: Generate the README

Write the README following the **Output Structure** defined below.

---

## Mode 2: From Project Analysis

Use this mode when the user asks you to analyze the project and write the README based on what you find, such as:
> "Lee el proyecto y escríbeme un README"
> "Analyze this project and generate a README"

### Step 1: Scan the Project Structure

Start with a broad overview:

```bash
# 1. List the top-level structure
ls -la

# 2. Look for config/manifest files
cat package.json 2>/dev/null
cat composer.json 2>/dev/null
cat pyproject.toml 2>/dev/null
cat Cargo.toml 2>/dev/null
cat go.mod 2>/dev/null

# 3. Check for existing README
cat README.md 2>/dev/null

# 4. Look for documentation
ls docs/ 2>/dev/null
ls doc/ 2>/dev/null

# 5. Check git info
git log --oneline -5 2>/dev/null
git remote -v 2>/dev/null
```

### Step 2: Deep Dive into Key Files

Based on what you found, read the most important files to understand the project:

| What to Read | Why |
|---|---|
| Entry point (`server.js`, `index.ts`, `main.py`, `src/main.rs`, etc.) | Understand what the app does on startup |
| Config files (`.env.example`, `config/`, `settings.py`) | Document required configuration |
| Route definitions (`routes/`, `src/routes/`, `urls.py`) | Understand the API surface or page structure |
| Core modules/components | Understand architecture and key features |
| `Dockerfile`, `docker-compose.yml` | Deployment method |
| CI/CD files (`.github/workflows/`, `Jenkinsfile`) | Build/deploy pipeline |
| Test files (`test/`, `__tests__/`, `spec/`) | Testing approach |
| `CHANGELOG.md` | Recent changes and versioning |

**Don't read everything** — focus on understanding the project's purpose, architecture, and usage patterns. Read at most 10–15 key files.

### Step 3: Ask Clarifying Questions

After analyzing the project, ask the user **focused questions** about things you couldn't determine from the code alone. Examples:

- "He encontrado que el proyecto usa Express + Mongoose. ¿Es una API REST pública o un backend interno?"
- "Veo que hay un directorio `skills/` con subproyectos. ¿Cada skill es independiente o comparten dependencias?"
- "No encontré información sobre deploy. ¿Quieres que incluya instrucciones de despliegue?"
- "¿El README va dirigido a desarrolladores que contribuyen al proyecto, o a usuarios que lo consumen?"

**Rules for questions:**
- Ask between 2 and 5 questions maximum
- Only ask what you genuinely couldn't infer from the code
- Group related questions together
- Wait for the user's answers before generating the README

### Step 4: Generate the README

Write the README following the **Output Structure** defined below, using both the analysis results and the user's answers.

---

## Output Structure

Every generated README should follow this structure. **Include only sections that are relevant** — omit any section that doesn't apply to the project.

```markdown
# Project Name

Brief, compelling description of the project in 1–3 sentences. What it does, who it's for, and why it exists.

## ✨ Features

- Feature 1 — short description
- Feature 2 — short description
- Feature 3 — short description

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Node.js | Runtime |
| Express | Web framework |
| ... | ... |

## 📋 Prerequisites

- Node.js >= 18
- MongoDB (if applicable)
- Any other system dependencies

## 🚀 Getting Started

### Installation

Step-by-step installation commands.

### Configuration

Explain .env or config files. If .env.example exists, reference it.

### Running the Project

Development and production commands.

## 📁 Project Structure

Directory tree showing the key folders and files with brief annotations.

## 📖 Usage

How to use the project. API examples, CLI commands, or UI workflows.

## 🧪 Testing

How to run tests, if applicable.

## 🤝 Contributing

Contribution guidelines, if applicable.

## 📄 License

License info, referencing the LICENSE file if it exists.
```

### Structure Rules

1. **Emojis in headers** are optional — use them if the project tone is friendly/community-oriented; omit them for corporate or strictly technical projects.
2. **Tech Stack table** — always use a table, never a plain list.
3. **Project Structure** — use a code block with a tree-like format. Annotate key directories with `# comment`.
4. **Getting Started** — must include real, tested commands from the project's `package.json` scripts (or equivalent).
5. **Badges** — only add badges if the project has CI/CD, npm publishing, or other services that provide them. Never add fake or placeholder badges.

---

## Combining Both Modes

When the user provides both a description AND you have access to the project:

1. **User description takes priority** for the narrative sections (project purpose, features description, audience tone)
2. **Project analysis takes priority** for technical sections (tech stack, structure, commands, config)
3. **Merge conflicts** — if the user's description contradicts the code (e.g., "this is a Python project" but it's actually Node.js), ask the user to clarify before proceeding

---

## Quality Checklist

Before delivering the final README, verify:

- [ ] All commands mentioned actually work (cross-reference with `package.json` scripts or equivalent)
- [ ] File paths and directory names match the real project structure
- [ ] No placeholder text (e.g., "your project name here", "describe feature")
- [ ] No broken markdown (check headers, tables, code blocks, links)
- [ ] The tone is consistent throughout the document
- [ ] The README length is proportional to the project's complexity — don't write 500 lines for a 3-file project, and don't write 20 lines for a monorepo
- [ ] If an existing README was found, incorporate any valuable information from it (don't discard previous work without reason)
