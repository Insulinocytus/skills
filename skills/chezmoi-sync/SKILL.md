---
name: chezmoi-sync
description: Sync chezmoi dotfiles both ways: capture local changes, pull remote ones, resolve conflicts, apply, push.
disable-model-invocation: true
---

# chezmoi sync

Bring this machine and the remote to one state. Local changes are **captured** into the source repo and committed first; remote commits are then **integrated** with a rebase, so "local is newer", "remote is newer", and "both changed" all run the same steps. You decide only what needs a human: new files outside auto-add directories, local deletions, and true conflicts. Everything else runs unattended through to the push.

Run every chezmoi command that might prompt with `--no-tty`: a conflict then fails with an error you can read instead of blocking on a console prompt. Each `--force` names one target, never the whole tree.

## Steps

1. **Survey.**
   - Read the source repo's `AGENTS.md` and `README.md` (`chezmoi source-path`). They carry the repo's own rules: commit message style, hooks, files synced key-by-key, and **auto-add directories**, whose new entries are added without asking.
   - `chezmoi git -- fetch`, then `chezmoi git -- status -sb`: note ahead/behind and uncommitted source changes. A rebase or merge already in progress stops the run: report it.
   - `chezmoi status`. The first column is this machine against what chezmoi last wrote; the second is what `apply` would do:
     - `MM`: edited here (maybe also in source).
     - ` M`, ` A`: changed or new in source, not yet applied.
     - `DA`: deleted here.
   - New files: pass every managed directory (`chezmoi managed -i dirs -p absolute`) to `chezmoi unmanaged -p absolute`. Paths in `.chezmoiignore` never show up.

   Done when every status row, unmanaged entry, and ahead/behind count has a bucket.
2. **Capture.**
   - Batch one question for everything needing a decision, then act on the answers:
     - Unmanaged entries outside auto-add directories: **add** (`chezmoi add <path>`, recursive) or **ignore** (append a pattern to the source `.chezmoiignore`, relative to home; group siblings into one glob, e.g. `.omp/agent/*.db*`). Ignored paths are never asked again.
     - `DA` targets: **delete** (`chezmoi --no-tty forget --force <target>`; every machine loses it after the push) or **restore** (`chezmoi --no-tty apply --force <target>`).
   - `chezmoi add` every unmanaged entry under an auto-add directory.
   - `chezmoi re-add`. It skips `modify_` and template sources; the repo's docs say whether a hook covers them. An `MM` row that survives re-add is a local edit the source cannot absorb: show `chezmoi diff <target>` and ask whether to keep it (edit the source by hand) or drop it (`apply --force` that target).
   - Commit everything in the source tree (`chezmoi git -- add -A`, then commit) with a message in the repo's style describing the change.

   Done when no `chezmoi status` row has a non-space first column and the source tree is clean.
3. **Integrate.** Behind the remote: `chezmoi git -- pull --rebase --recurse-submodules`. For each conflicted file:
   - In a rebase, stage `:2:` is the remote side and `:3:` is the local commit being replayed.
   - Structured files (JSON, YAML, TOML), including generated ones such as a lock file or a shared config: read base `:1:`, `:2:` and `:3:` with `chezmoi git -- show :N:<path>` and merge key by key. Keys changed on one side take that side; the union of added entries is kept. Re-parse the result.
   - A key or text region changed differently on both sides: show both values and ask.
   - `chezmoi git -- add <path>`, then `chezmoi git -- rebase --continue`.

   Done when the rebase has finished and `HEAD` contains the remote branch.
4. **Apply.** `chezmoi --no-tty apply`. A "has changed since chezmoi last wrote it" error means Capture missed a target: return to step 2 for it.

   Done when `chezmoi status` prints nothing.
5. **Push.** Ahead of the remote: `chezmoi git -- push`.

   Done when `chezmoi git -- status -sb` shows neither ahead nor behind.
6. **Report:** commits pulled and pushed, files added, ignored, deleted, or restored, each conflict and how it was resolved, and any step that stopped with its error.
