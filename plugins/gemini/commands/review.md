---
description: Review current code changes using Gemini CLI, classify findings, and optionally apply fixes.
argument-hint: [optional: scope or note for the review]
allowed-tools: Bash(gemini:*), Bash(git status:*), Bash(git diff:*), Bash(git branch:*), ListFiles, Read, Write
---

You are a Claude Code slash command that uses **Gemini CLI** to review the current code changes and help apply fixes.

You must:
- Build a rich **context** (prompts, investigations, git changes, referenced docs),
- Ask Gemini CLI for a **code review**,
- Classify the findings into three categories:
  - Must fix
  - Needs discussion/consideration
  - No need to fix
- Interactively resolve the “needs discussion” items with the user,
- Apply approved fixes,
- Run a **second review** with updated context and show the final results.

Gemini CLI is assumed to be installed and already usable.

Follow the steps below strictly.

---

## 1. Generate the context to share with Gemini

The context must include:

- The prompt interactions and investigations related to the current change (in this Claude Code session).
- Changes stacked on the current Git branch.
- Any referenced documentation that is relevant to the change.

Do the following:

1. **Summarize prompt interactions & investigations**

   - From the recent conversation in this Claude Code session, create a short narrative summary including:
     - What feature or bug is being worked on.
     - Any constraints, design decisions, or tradeoffs discussed.
     - Any important investigation results (e.g. “we found that X is caused by Y”).
   - Keep this summary within a few paragraphs.

2. **Identify the current Git branch & status**

   - Use Bash:

     ```bash
     git branch --show-current
     git status --short
     ```

   - Note the current branch name and whether there are staged/unstaged changes.

3. **Collect diffs for the current change**

   - Prefer to capture both staged and unstaged changes.

   - Use something like:

     ```bash
     git diff --cached
     git diff
     ```

   - If there are too many files, you may mention that you’re focusing on the most relevant ones (based on the recent conversation context).

4. **Collect referenced documentation (optional but recommended)**

   - If the user or code references specific docs (e.g. `docs/api.md`, `README.md`, design docs), use:
     - `ListFiles` to locate them (if needed),
     - `Read` to extract only the relevant sections (not the entire file if it’s huge).
   - Summarize each doc briefly (what it defines and how it relates to the change).

5. **Assemble a structured context text**

   Construct a context text with sections like:

   ```text
   [High-level Summary]
   ... (summary of the current task / bug / feature)

   [Conversation & Investigations]
   ... (key points from recent prompts and investigations)

   [Git Branch]
   current branch: <branchName>
   git status:
   <git status --short output>

   [Code Changes (Diffs)]
   --- staged changes (git diff --cached) ---
   <output or summarized snippets>

   --- unstaged changes (git diff) ---
   <output or summarized snippets>

   [Referenced Docs]
   - docs/api.md: <short summary or key excerpts>
   - README.md: <short summary or key excerpts>