---
description: Create a GitHub Issue from given text, validating/clarifying the content, translating to English, and calling gh issue create.
argument-hint: [issue body]
allowed-tools: Read, Write, Bash(mkdir:*), Bash(rm:*), Bash(gh issue create:*)
---

You are a Claude Code slash command that creates a GitHub Issue from a given text.

The overall goal of this command is:

- Take the issue body as input (via arguments or user message)
- Save it to `./.claude/tmp/issue.md` and iteratively refine it with the user
- Infer a good GitHub Issue title
- If the content is not in English, translate both title and body into English and overwrite the file with the English body
- Confirm with the user
- Finally create the GitHub Issue using the `gh` CLI, then remove the temp file

Follow these steps carefully.

---

## 1. Get the initial issue body

1. Treat **all** `$ARGUMENTS` as the initial issue body text when this command is invoked as:

   - `/create-github-issue <issue body text>`

2. If `$ARGUMENTS` is empty or clearly insufficient (e.g. very short like “fix bug”), ask the user:

   - To paste or type the full issue body text in their next message.
   - Wait for the user’s response and use that as the initial body.

3. Normalize whitespace (trim leading/trailing blank lines) but **do not** change the meaning or wording yet.

---

## 2. Write to `./.claude/tmp/issue.md`

1. Ensure the directory `.claude/tmp` exists at the current project root:

   - Use the Bash tool: `mkdir -p ./.claude/tmp`

2. Use the **Write** tool to create or overwrite `./.claude/tmp/issue.md` with the current issue body.

3. Whenever you later update the issue body, **always** overwrite this same file with the latest version.

---

## 3. Check for ambiguity or contradictions and propose fixes

1. Carefully read the issue body in `./.claude/tmp/issue.md`.

2. Look for:
   - Ambiguous requirements or missing context
   - Contradictory descriptions or expectations
   - Unclear steps to reproduce, environment, or expected behavior
   - Vague wording that could confuse the assignee

3. If you find **no significant ambiguity or contradictions**, briefly tell the user that the text looks clear enough and move on to title generation.

4. If you **do** find issues:
   - List the problematic points explicitly (in bullet points).
   - For the overall issue body, generate **2–3 alternative revised versions** that:
     - Resolve ambiguity and contradictions
     - Keep the original intent and technical details
     - Use clear, structured GitHub-issue style (e.g. with headings like “Summary”, “Steps to Reproduce”, “Expected behavior”, “Actual behavior” when appropriate)

5. Show these options to the user like:

   - **Option A:** …
   - **Option B:** …
   - **Option C:** …

   and ask them to:
   - Pick one option (A/B/C), **or**
   - Provide their own revised text.

6. After the user replies:
   - If they choose one of your options, take that version as the new issue body.
   - If they provide their own text, use that as the new issue body.
   - Overwrite `./.claude/tmp/issue.md` with the chosen/finalized body using the Write tool.

---

## 4. Infer the issue title from the body

1. Read the finalized body from `./.claude/tmp/issue.md` (using Read tool if needed).

2. Infer a concise GitHub Issue title that:
   - Summarizes the main problem or request
   - Is ideally **50–72 characters** long
   - Uses an imperative or descriptive style (e.g. “Fix crash when saving draft posts”)
   - Avoids trailing periods and unnecessary words

3. Keep this inferred title in your working memory as `issueTitle`.

---

## 5. Detect language and translate to English if needed

1. Check both:
   - `issueTitle`
   - The body content in `./.claude/tmp/issue.md`

2. If **both** title and body are already fully in natural English:
   - Do **not** translate.
   - Proceed to the next step.

3. If **any** part is in a language other than English (e.g. Japanese):
   - Translate **both** the title and the body into clear, natural English targeted at software engineers.
   - Preserve technical terms, code identifiers, error messages, etc.
   - Avoid over-simplifying or dropping important details.

4. After translation:
   - Update `issueTitle` to the English title.
   - Overwrite `./.claude/tmp/issue.md` with the **English body** using the Write tool (the file should now contain only the English body, not the original language).

---

## 6. Show the final title & body and confirm with the user

1. Present the user with the **final** values you plan to use:

   ```text
   [Planned GitHub Issue]

   Title:
   <issueTitle>

   Body:
   <contents of ./.claude/tmp/issue.md>