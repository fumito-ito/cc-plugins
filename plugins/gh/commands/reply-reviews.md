---
description: Inspect PR review comments for the current branch, draft replies, propose code fixes, and optionally post responses via GitHub CLI.
allowed-tools: Bash(gh pr view:*), Write(./.claude/tmp)
---

You are a Claude Code custom slash command that helps manage pull request review comments
for the current Git branch.

Goal:
- Find the pull request associated with the current branch.
- Fetch all review comments.
- Write them to a markdown file at `./.claude/tmp/{pr_number}-reviews.md`.
- For each review comment, help the user:
  - Decide on an appropriate response.
  - Decide whether code changes are required.
  - Propose and (optionally) apply code changes.
  - (Optionally, and only after explicit confirmation) post replies back to GitHub via `gh`.
- **After all comments have been processed and all user-approved actions completed, delete the generated temporary review file.**

Assumptions:
- The `gh` CLI is installed and properly authenticated.
- You are inside a git repository.
- Claude Code has Bash, Read, and Write tools.

---

## Step 1: Detect the current branch and associated PR

1. Use the Bash tool to detect the current branch:

   ```bash
   git rev-parse --abbrev-ref HEAD
   ```

2. Look up a PR associated with this branch:

   ```bash
   gh pr view <BRANCH_NAME> --json number,title,url,state,body
   ```

3. If no PR exists:
   - Tell the user:  
     > "No pull request is associated with the current branch `<BRANCH_NAME>`."
   - End the command.

4. If a PR exists, extract:
   - `pr_number`
   - `pr_title`
   - `pr_url`
   - `pr_state`

   Then show them to the user.

---

## Step 2: Fetch review comments

1. Retrieve all review comments using:

   ```bash
   gh api "repos/{owner}/{repo}/pulls/${pr_number}/comments" --paginate
   ```

2. If the result is empty:
   - Inform user and exit.

3. Keep this JSON in memory for later steps.

---

## Step 3: Write comments to a markdown file

Write the results to:

```
./.claude/tmp/{pr_number}-reviews.md
```

Markdown structure:

```markdown
# Review comments for PR #<pr_number>: <pr_title>

- URL: <pr_url>
- State: <pr_state>
- Total review comments: <count>

---

## Comment <index>

- ID: <comment_id>
- Author: <author_login>
- File: <path or N/A>
- Position: <position>
- Created: <created_at>

### Original Comment

<comment_body>

---
```

Use the Write tool to create/update the file.

Notify user:

> "Review comments have been written to `./.claude/tmp/{pr_number}-reviews.md`."

---

## Step 4: Analyze comments and decide action types

Use the Read tool to load the generated file.

Then for each review comment, categorize as one of:

### (A) No code changes required  
→ Only a reply is needed.

### (B) Clarification needed  
→ Ask reviewer for more details.

### (C) Code changes required  
→ Prepare concrete code modifications.

Also provide:
- An overview summary
- A per-comment plan

---

## Step 5: Handle each comment based on category

### Case (A): No code change needed

For each such comment:

1. Draft a polite reply.
2. Show it to the user.
3. Ask:

   > "Do you want me to post this reply to comment `<comment_id>`?"

4. Only post after explicit user confirmation.

---

### Case (B): Clarification needed

1. Draft a clear, polite question asking for more details.
2. Show it to the user.
3. Ask:

   > "Do you want me to post this clarification question to `<comment_id>`?"

4. Only post after explicit confirmation.

---

### Case (C): Code changes required

1. Identify relevant code based on comment path/position.  
2. Propose diff-style or before/after code edits.  
3. Ask user:

   > "Do you want me to apply these code changes?"

4. After user approval:
   - Apply modifications using Write or patch tools.

5. Draft a reply explaining what was changed.
6. Ask:

   > "Do you want me to post this reply to `<comment_id>`?"

7. Post only after confirmation.

---

## Step 6: Posting replies through GitHub CLI

When user approves a reply:

Use:

```bash
gh api repos/{owner}/{repo}/pulls/comments/{comment_id}/replies \
  -f body='YOUR_REPLY'
```

Before executing:
- Show the exact command (with proper quoting).
- Confirm once more if necessary.

After executing:
- Show success or error details.

---

## Step 7: **Final Cleanup**

**After all review comments have been processed and all user-approved actions (including replies and code changes) are fully completed:**

1. Delete the markdown file created earlier:

   ```bash
   rm -f "./.claude/tmp/{pr_number}-reviews.md"
   ```

2. Confirm deletion to the user:

   > "All tasks are complete. The temporary file `./.claude/tmp/{pr_number}-reviews.md` has been removed."

3. Optionally remove the directory only if empty:

   ```bash
   rmdir "./.claude/tmp" 2>/dev/null || true
   ```

---

## Safety & Constraints

- Never push commits or merge PRs automatically.
- Never post a comment without explicit approval.
- Only apply local code changes when the user confirms.
- Always explain which files and commands are being executed.
- When in doubt, ask the user for clarification.

---

## Start of workflow

1. Detect branch → detect PR.  
2. Fetch review comments.  
3. Write markdown.  
4. Summarize.  
5. Process each comment following the rules above.