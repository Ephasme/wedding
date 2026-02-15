- Enable sub-agents (experimental) in your `settings.json`, then restart Gemini CLI. You can edit settings via the `/settings` UI or by editing the file directly.

  ```json
  {
    "experimental": {
      "enableAgents": true
    }
  }
  ```

- Create a new agent definition as a Markdown file with **YAML frontmatter** (the file **must** start with `---`). Put it either in your project (shared) or your user config (personal):

  - Project-level: `.gemini/agents/<your-agent>.md`
  - User-level: `~/.gemini/agents/<your-agent>.md`

- Use this minimal template (YAML frontmatter + Markdown body). The **body becomes the agent’s system prompt**.

  ```md
  ---
  name: security_auditor
  description: Reviews changes for security issues and suggests concrete fixes.
  kind: local
  # tools: ["<tool_name_1>", "<tool_name_2>"]  # optional
  # model: inherit  # optional (or a specific model, e.g. gemini-2.5-pro)
  ---

  You are a security auditor. Focus on:

  - Vulnerability discovery (auth, injection, secrets, supply chain).
  - Concrete patches and tests.
  - Minimal, safe changes.
  ```

  Notes:

  - `name` is the unique identifier and becomes the **tool name**; it must be a slug (lowercase letters, numbers, hyphens, underscores).
  - `tools` (optional) can restrict which tools the sub-agent can use; if omitted it may get a default set.
  - `model` (optional) can be set per-agent; default is `inherit` (use the main session model).

- Reload and enable it from inside Gemini CLI:

  - `/agents refresh` (reload agent registry after adding/editing files)
  - `/agents list` (see available agents)
  - `/agents enable security_auditor` (enable your agent)

- Use it by asking the main agent to delegate work to that specialist; sub-agents are exposed to the main agent “as a tool of the same name,” and the main agent will decide when to call them based on the description (you can also explicitly request it).

- Caution: sub-agents currently run in **“YOLO mode”** (they may execute tools without step-by-step confirmation), so be careful granting powerful tools like shell execution or file writes.
