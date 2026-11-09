---
name: read-only
description: "Read-only mode: inspect and propose; writes only inside a session scratch directory."
disable-model-invocation: true
---

# Read-only mode

Active until the user says to exit read-only mode. You are an observer: read, run, analyse, propose. The user decides what changes.

## Writes

A **write** is any action whose effect outlives its output:

- Files: creating, editing, moving, deleting, anywhere (workspace, `~`, dotfiles, global config).
- Git state: commit, checkout, switch, restore, reset, stash, merge, rebase, fetch, pull, push, branch/tag edits, `worktree add`.
- Installs: package managers, `mise use/install`, global tools.
- External side effects: non-GET HTTP, `gh` writes, MCP write tools, sending messages.

Everything else runs freely: reading files, search, `git log/diff/status/show/blame`, HTTP GET, MCP read tools, any command whose only effect is output.

## Scratch directory

On the first write you need, create one directory with `mktemp -d` (or the platform equivalent) and record its absolute path. Writes land inside it: scripts, notes, clones, a `cp -a` copy of the repo to edit and run, dependency installs scoped to it. Before each write, resolve the target path (following symlinks) and confirm it is under the scratch directory.

A write that cannot land there needs a grant (see below).

## Builds and tests

Run them when their output can be redirected into the scratch directory (`CARGO_TARGET_DIR=…`, `PYTHONDONTWRITEBYTECODE=1 pytest -p no:cacheprovider`), or run them inside a scratch copy of the repo. When neither works, ask first.

## Requests that imply changes

In this mode, "fix X" or "implement Y" means **propose**: explain the plan, or make the change in a scratch copy and show the diff (`git diff --no-index`). Then stop and wait.

A write outside the scratch directory needs an explicit **grant** from the user ("可以改了", "unlock", "apply it") that refers to the proposed change. Apply exactly that change, then return to read-only. A vague reply gets a clarifying question.

## Subagents

Pass these rules verbatim to every subagent you dispatch.
