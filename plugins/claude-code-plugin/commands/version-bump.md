---
description: Increment the version of Claude Code plugins developed in the current branch
allowed-tools: Bash(git *), Read, Edit
---

# Version Bump Command

Increment the version of Claude Code plugins developed in the current branch.

## Task Steps

1. Use `git diff` and `git status` commands to check all changes on the current branch and get a list of directories under the `plugins` directory that contain modified files

2. Process each directory that contains modified files:
   - If directory `foo` contains modified files, verify that `plugins/foo/.claude-plugin/plugin.json` exists
   - Check the value of the `version` property in `plugin.json`
   - Present the directory name and current `version` property value to the user, and ask whether the changes correspond to `major`, `minor`, or `patch`
   - Update the `version` property value according to the user's selection:
     - For `major`: Increment the x part of `x.0.0`
     - For `minor`: Increment the y part of `x.y.0`
     - For `patch`: Increment the z part of `x.y.z`

3. After processing all directories, ask the user if they want to commit the changes. If YES, commit the changes with an appropriate commit message

## Notes

- Assumes the working file tree is for developing Claude Code Plugins
- Assumes the working file tree is version-controlled with git and development is being done on a branch
- Skip directories where `plugin.json` does not exist
