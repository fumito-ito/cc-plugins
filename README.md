# Claude Code Plugin Marketplace

A collection of custom commands and plugins for extending Claude Code capabilities with integrations for Gemini CLI and gh CLI. This repository is structured as a plugin marketplace, allowing you to enhance your Claude Code experience with additional productivity tools.

## Features

### Gemini CLI Integration (`daily-use.gemini`)
- **Gemini Discuss** (`/discuss-with-gemini`): Conduct in-depth discussions about your current work with Gemini AI, enhancing Claude Code's accuracy through multi-perspective analysis and iterative refinement
- **Gemini Search** (`/gemini-search`): Use Google's Gemini CLI for web searches instead of Claude's built-in web search tool

### gh CLI Integration (`daily-use.gh`)
- **GitHub Issue Creator** (`/daily-use.gh:create-issue`): Create GitHub issues from text with automatic validation, clarification, and translation capabilities

## Prerequisites

To use these commands and plugins, you'll need to have:

### For Gemini Commands
- Gemini CLI installed and available in your PATH
- Authentication configured via `gcloud auth application-default login`

### For gh Commands
- gh CLI (`gh`) installed and available in your PATH
- gh CLI authenticated with your GitHub account

## Installation

### Setup Marketplace

This repository is configured as a Claude Code plugin marketplace, allowing you to access all plugins at once.

```
$ claude
$ /plugin
=> Select `Add Marketplace`
=> Enter marketplace source as: `fumito-ito/cc-plugins`
=> Select `Browse and install plugins`
```

## Commands Reference

### /daily-use.gemini:discuss

This command facilitates detailed discussions with Google's Gemini AI about your current work.

**Process:**
1. Collects information about your current work from Git
2. Prepares discussion topics on architecture, performance, maintainability, security, etc.
3. Initiates a discussion with Gemini using a comprehensive prompt
4. Conducts iterative refinement through 3-5 rounds of discussion
5. Generates an actionable plan with prioritized items
6. Saves the discussion log for future reference

**Arguments:**
- No arguments: Analyzes current Git changes and work context
- Topic (e.g., "GraphQL schema optimization"): Focuses the discussion on a specific topic
- File path: Analyzes a specific file
- `--deep`: Conducts a deeper analysis with more discussion rounds

### /daily-use.gemini:search

A command that uses the Gemini CLI for web searches, bypassing Claude's built-in search functionality.

**Usage:**
```
/gemini-search <search query>
```

### /daily-use.gh:create-issue

Creates a GitHub issue with validation and translation capabilities.

**Process:**
1. Takes issue body as input
2. Saves it to a temporary file
3. Checks for ambiguity and proposes improvements
4. Infers a GitHub issue title
5. Translates content to English if needed
6. Confirms with you before creating the issue

**Arguments:**
```
/daily-use.gh:create-issue [issue body]
```

## Contributing

Contributions are welcome! Feel free to submit pull requests or open issues to suggest improvements or add new features.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.