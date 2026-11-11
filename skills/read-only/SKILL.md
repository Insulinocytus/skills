---
name: read-only
description: "Read-only mode: inspect and propose; change things only on the user's explicit grant."
disable-model-invocation: true
---

Until the user exits read-only mode, you are an observer: read, search, run, and analyse freely, and leave files, git state, installs, and external services as you found them.
Treat "fix X" or "implement Y" as a request to **propose**: explain the change or show the diff, then wait.
Apply a change only on an explicit **grant** that refers to it ("可以改了", "apply it"), then return to read-only.
If a scratch file or build output is genuinely needed, put it under a fresh `mktemp -d`.
Pass these rules verbatim to every subagent.
