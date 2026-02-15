## Gemini CLI: create a “rule” you can call with a slash command

### 1) Choose where the command lives

- **User-scoped (global):** `~/.gemini/commands/`  
   Available in all projects.
- **Project-scoped (repo-only):** `<project>/.gemini/commands/`  
   Available only within that project.
- **Precedence:** if both define the same command name, the **project** command wins.

### 2) Name the file to define the slash command

- Command name comes from the file path **relative to** the `commands` directory.
- `commands/test.toml` → `/test`
- Subfolders become **namespaces** using `:`:
  - `commands/git/commit.toml` → `/git:commit`
- File names are **case sensitive**.

### 3) Create a “rules” command (example)

Create one of these files:

- Global: `~/.gemini/commands/rules/strict.toml`
- Repo-only: `<project>/.gemini/commands/rules/strict.toml`

Paste:

```toml
description = "Apply our strict engineering rules to the current task"
prompt = """
You must follow these rules while answering:
-  Prefer small, reviewable changes.
-  Do not change public APIs without calling it out.
-  Add/adjust tests when behavior changes.
-  Explain tradeoffs and edge cases.

Context (project guidelines):
@{docs/engineering-rules.md}

User request:
{{args}}
"""
```

Run it in Gemini CLI:

- `/rules:strict Refactor this module without changing behavior`

### 4) Arguments behavior (important)

- If your `prompt` contains `{{args}}`, Gemini CLI replaces it with whatever you type after the command.
- If your `prompt` does **not** contain `{{args}}`:
  - If you pass arguments (e.g. `/cmd foo`), Gemini CLI appends the full invocation to the end of the prompt (after two newlines).
  - If you pass no arguments (e.g. `/cmd`), the prompt is sent as-is.

### 5) Optional: add dynamic context (git diff, etc.)

You can run a shell command and inject its output using `!{...}` (Gemini CLI will ask for confirmation):

Create `~/.gemini/commands/rules/review.toml`:

```toml
description = "Review staged changes against our rules and propose fixes"
prompt = """
Apply our engineering rules and review the staged diff.

Rules:
@{docs/engineering-rules.md}

Staged diff:
!{git diff --staged}

Focus area (optional):
{{args}}
"""
```

Run it:

- `/rules:review`
- `/rules:review auth flow`
