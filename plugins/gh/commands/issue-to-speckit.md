---
description: Select or fetch a GitHub Issue and pass it to /speckit.specify
argument-hint: "[issue-number]"
allowed-tools: Bash(gh issue list:*), Bash(gh issue view:*)
---

You are a helper that bridges GitHub Issues and Spec Kit.

Your job:

1. Use the GitHub CLI (`gh`) against the **current repository**.
2. Either:
   - If the user provided an issue number (e.g. `/speckit.issue 123`), fetch that single Issue.
   - If no issue number is provided (just `/speckit.issue`), fetch a **list of recent Issues**, show them 10 at a time, and let the user choose which one to use.
3. Once an Issue is chosen, pass its content to `/speckit.specify` as an argument so that Spec Kit can generate a spec for that Issue.

---

## 1. Input handling

- If the command was invoked with an argument, treat the **first argument** as the target Issue:
  - Accept either:
    - A plain issue number like `123`
    - Or a full GitHub Issue URL
- If **no argument** was given:
  - Enter **interactive selection mode**:
    - Fetch up to 50 Issues (newest first, based on creation time).
    - Present them **10 at a time** as a table with:
      - Issue number
      - Title
      - Labels
      - Created date (YYYY-MM-DD)
    - After showing each page, clearly tell the user they can:
      1. Select one of the listed Issues (by index or issue number)
      2. Enter any other Issue number manually
      3. Type `"more"` to see the next 10 (up to 50 total)
      4. Type `"cancel"` to abort

When you are in interactive mode, keep track of which page you are on and which Issues have already been shown. Do **not** re-call `gh` unnecessarily; reuse the JSON you already fetched whenever possible.

---

## 2. Fetching Issues with gh

Use the Bash tool with the `gh` CLI commands below. Always assume it is already authenticated and running inside a cloned repository.

### 2-1. Listing Issues (interactive mode, when no number is given)

Call Bash with:

- Command (current repo, newest by creation time, all states):

```bash
gh issue list \
  --state all \
  --limit 50 \
  --json number,title,labels,createdAt \
  --search "sort:created-desc"
```

From the JSON:

- Sort by `createdAt` descending (newest first).  
  (The `sort:created-desc` search qualifier should already give you this order, but you can treat the JSON as the source of truth.)
- Paginate in the **chat UI**:
  - First page: items 1–10
  - Second page: items 11–20
  - … up to 50
- For each page, render a compact table, for example:

| # | Title | Labels | CreatedAt |
|---|-------|--------|-----------|
| 123 | Fix login bug | bug, backend | 2025-11-10 |
| 122 | Add audit log | feature | 2025-11-09 |

- After showing a page, explicitly prompt:

> Enter the **Issue Number** of the issue you want to select.
> To specify a different issue number, enter that number.
> To view the next 10 issues, enter `"more"`.
> To cancel, enter `"cancel"`.

Then:

- If the user chooses an index (1–10 for that page), map it back to the corresponding Issue number from the list.
- If they provide an issue number, use that number directly.
- If `"more"`, show the next 10 Issues (from the same JSON).
- If `"cancel"`, stop and explain that the command was aborted.

Once you have a final Issue number, continue with step 2-2.

### 2-2. Viewing a single Issue (when the number is known)

When you have a specific Issue number or URL (either from the command argument or from the interactive selection), call Bash:

```bash
gh issue view "<ISSUE_ID_OR_URL>" \
  --json number,title,body,labels,author,state,url,createdAt,updatedAt
```

Treat the returned JSON as the canonical representation of the Issue.

---

## 3. Preparing the JSON payload for Spec Kit

From the `gh issue view` JSON, create a **clean, compact JSON object** that will be passed to `/speckit.specify`.  
You can reuse most fields directly, but normalize labels and author for easier reading.

Use a structure like:

```json
{
  "source": "github-issue",
  "issue": {
    "number": 123,
    "title": "Issue title ...",
    "body": "Raw markdown body from the issue...",
    "labels": ["bug", "frontend"],
    "state": "OPEN",
    "url": "https://github.com/owner/repo/issues/123",
    "author": "octocat",
    "createdAt": "2025-11-01T12:34:56Z",
    "updatedAt": "2025-11-10T09:00:00Z"
  }
}
```

Notes:

- `labels` should be rendered as an array of **label names** only.
- `author` should be the login / display name string.
- Keep `body` as-is (markdown or text) so `/speckit.specify` has full context.
- You may also include additional fields (e.g. `comments`, `assignees`) if available and helpful, but keep the JSON relatively compact and focused.

---

## 4. Invoking `/speckit.specify`

Once you have constructed the JSON object above:

1. Render it as pretty-printed JSON (so the user can read it).
2. Then **invoke `/speckit.specify`** using that JSON as its main argument.

Use a pattern like:

```text
/speckit.specify Please create a specification based on the contents of the following GitHub Issue.

```json
<Embed the JSON constructed above here>
```
```

Guidance:

- The argument to `/speckit.specify` should:
  - Clearly tell Spec Kit that the following JSON describes a GitHub Issue.
  - Ask it to derive requirements, user stories, and acceptance criteria from the Issue content.
  - Be in Japanese by default, since Spec Kit is commonly used that way in this project.
- Make sure the final message you send contains **both**:
  - A short natural-language instruction (1–3 sentences)
  - The JSON block

At the end of your response for this slash command, **show the exact `/speckit.specify` invocation** you used (including the JSON) so the user can copy, tweak, or re-run it if needed.

If anything goes wrong (e.g. `gh` returns an error, or the issue doesn’t exist), explain what happened in Japanese and ask the user how they’d like to proceed instead of silently failing.