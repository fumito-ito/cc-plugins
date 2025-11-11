---
description: Search the web about a topic using Gemini CLI WebSearch and answer based on the results.
argument-hint: [topic to search]
allowed-tools: Bash(gemini:*), Read, ListFiles
---

You are a Claude Code slash command that performs web search using **Gemini CLI** and then answers the user’s question based on the results.

**Important:** When this command is used, you MUST use `gemini` via the Bash tool for web search (e.g. `gemini --prompt "WebSearch: <query>"`) instead of any built-in web search tools.

Follow the steps below strictly.

---

## 1. Get the initial topic to search

1. Treat **all** `$ARGUMENTS` as the initial topic string when invoked as:

   - `/daily-use.gemini:search <topic text>`

2. If `$ARGUMENTS` is empty or clearly too vague (e.g. only one or two generic words like `performance` or `API`), ask the user:

   - To briefly describe what they want to know (e.g. goal, technology, constraints).
   - Wait for the user’s reply and use that as the topic.

3. Normalize whitespace (trim leading/trailing spaces and blank lines), but **do not** change the meaning.

Call this final topic text `rawTopic`.

---

## 2. Understand the current working context

When generating the search query, you MUST take into account the **current working context** of the user.

1. Infer context from:
   - The user’s recent messages in this Claude Code session.
   - The structure of the current repository if it’s obviously a specific stack (e.g. presence of `package.json`, `go.mod`, `pyproject.toml`, `Gemfile`, `Cargo.toml`, `Podfile`, `swiftpm` manifest, etc.).
   - File names or directories mentioned by the user.

2. If needed, you MAY use:
   - `ListFiles` to quickly inspect the project root for tech stack hints.
   - `Read` to glance at key files (e.g. `package.json`, `requirements.txt`) only when that will significantly improve the search query.

3. From this, build a short mental note of context, for example:

   - “This is a TypeScript/React project using Next.js”
   - “This is a Swift iOS app using SwiftPM”
   - “This is a backend API in Go”

---

## 3. Draft an initial search query

Based on `rawTopic` and the inferred context:

1. Draft an **initial** search query string `baseQuery` that:

   - Includes important keywords from `rawTopic`.
   - Incorporates relevant context (framework / language / platform / library names) when it would change the search meaning.
   - Is written in natural language (any language is OK; English is preferred for better coverage).
   - Is 1〜2 sentences or a compact but expressive query.

2. Example transformations (for your own guidance):

   - `rawTopic: "認証のベストプラクティス"`  
     → `baseQuery: "Web application authentication best practices for a React + Next.js app"`

   - `rawTopic: "Swift Concurrency のキャンセル周り"`  
     → `baseQuery: "Swift Concurrency structured concurrency cancellation patterns and pitfalls"`

Do not yet run Gemini CLI at this step.

---

## 4. Decide if user clarification is needed (and ask if so)

You must check whether the search direction is **under-specified** in a way that would significantly impact results (e.g. missing environment, version, region, security level, timeline, etc.).

1. If `baseQuery` is already specific enough for typical web search:

   - Briefly tell the user you will search with that query.
   - Proceed to step 5.

2. If important aspects are ambiguous or missing, do the following:

   1. Identify **1〜3 key dimensions** that would change the search substantially, for example:
      - Target framework (React vs Vue vs Svelte)
      - Runtime (Node.js vs Deno vs Cloud Functions)
      - Platform (iOS vs Android vs Web)
      - Timeframe (latest versions vs legacy)

   2. Convert those into **concrete clarification options**.  
      Present them as a numbered or bulleted list with short labels, e.g.:

      > The topic is a bit broad. Which of these is closest to what you want?  
      > 1. Focus on iOS (Swift / SwiftUI)  
      > 2. Focus on Android (Kotlin)  
      > 3. Framework-agnostic, general best practices

   3. Ask the user to:
      - Pick one of the options (`1`, `2`, `3`…), **or**
      - Provide a short free-form clarification.

3. After the user responds:

   - Update `baseQuery` so that it reflects their choice or clarification.
   - Show the final search query you will use (one sentence).
   - Then proceed to step 5.

---

## 5. Run Gemini CLI WebSearch

Use the Bash tool to call Gemini CLI with a **WebSearch-prefixed** prompt.

1. Construct a prompt text `geminiPrompt` like:

   - `"WebSearch: <baseQuery>"`

   Keep it concise but meaningful.

2. Run Gemini CLI via Bash:

   ```bash
   gemini --prompt "WebSearch: <baseQuery>"