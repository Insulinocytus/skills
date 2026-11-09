---
name: forgejo-api
description: Forgejo REST API cheat sheet (curl + jq). Use when calling a Forgejo instance's API (e.g. Codeberg), including investigating failed Actions/CI runs and logs.
---

# Forgejo API

A digest of the Forgejo v17 API (`/api/v1`), meant for `curl` + `jq`. Work from this file; for anything it leaves out, see [When this file is not enough](#when-this-file-is-not-enough).

## Setup

- **Instance** (`FORGEJO_URL`): take the host from `git remote get-url origin` (`git@host:owner/repo.git`, `ssh://git@host[:port]/owner/repo.git` or `https://host/owner/repo.git`) and set `FORGEJO_URL=https://<host>`, without the SSH port; `OWNER`/`REPO` come from the same path. If there is no remote, or it is not Forgejo (`GET https://<host>/api/v1/version` fails), keep the user's exported `FORGEJO_URL`. If neither works, stop and ask the user for the instance URL.
- **Token**: read `FORGEJO_TOKEN`. If it is unset, stop and tell the user to create one under *Settings → Applications → Generate New Token* and export it as `FORGEJO_TOKEN`. Scopes are `read:`/`write:` per area (`repository` covers Actions, `issue`, `organization`, `user`, …); a 403 with a valid token means a missing scope, so name the scope the call needs.
- **Request template**:

  ```sh
  API="$FORGEJO_URL/api/v1"
  curl -sS --fail-with-body -H "Authorization: token $FORGEJO_TOKEN" "$API/repos/$OWNER/$REPO" | jq .
  # with a JSON body
  curl -sS --fail-with-body -H "Authorization: token $FORGEJO_TOKEN" -H 'Content-Type: application/json' \
    -X POST "$API/repos/$OWNER/$REPO/issues" -d "$(jq -n --arg t "$TITLE" --arg b "$BODY" '{title:$t, body:$b}')"
  ```

  Build bodies with `jq -n --arg`, so quotes and newlines in Markdown survive.

## Pagination

List endpoints take `page` (1-based) and `limit`. Send `limit=50`, the default `max_response_items` (an instance's real cap is at `GET /settings/api`). Keep following the `Link` header's `rel="next"` until it is absent. `x-total-count` holds the total, so `curl -sI` and read that header when only a count is needed.

## Repos, branches, files

- Repo: `GET /repos/{owner}/{repo}`; edit settings: `PATCH` the same path with any `EditRepoOption` field.
- Create: `POST /user/repos` or `POST /orgs/{org}/repos` with `{name, private?, description?, default_branch?, auto_init?}`.
- Fork: `POST /repos/{owner}/{repo}/forks` `{organization?, name?}` → 202 (forking finishes asynchronously).
- Branches: `GET /repos/{o}/{r}/branches`; create: `POST` the same path with `{new_branch_name, old_ref_name?}` (`old_ref_name` takes a branch, tag or SHA).
- Compare: `GET /repos/{o}/{r}/compare/{base}...{head}` → `{total_commits, commits[], files[]}`.
- Read a file:
  - `GET /repos/{o}/{r}/raw/{path}?ref=<ref>` returns the raw bytes.
  - `GET /repos/{o}/{r}/contents/{path}?ref=<ref>` returns metadata including `sha`, with `content` base64-encoded. On a directory it returns an array.
  - `GET /repos/{o}/{r}/contents?ref=` lists the root.
- Write a file:
  - Create: `POST /repos/{o}/{r}/contents/{path}` `{content, message?, branch?, new_branch?}`.
  - Update: `PUT` the same path `{content, sha, ...}`, where `sha` is the blob sha from `GET contents`.
  - Delete: `DELETE` the same path `{sha, message?}`.
  - `content` is always base64 (`base64 -w0 file`).
- Several files in one commit: `POST /repos/{o}/{r}/contents` `{files:[{operation:"create"|"update"|"delete", path, content?, sha?, from_path?}], message, branch?, new_branch?}`.

## Issues

- List: `GET /repos/{o}/{r}/issues?state=open|closed|all&type=issues|pulls&labels=a,b&q=<text>`. Without `type=issues`, the list includes PRs.
- Create: `POST /repos/{o}/{r}/issues` `{title, body?, labels?:[ids], milestone?:id, assignees?:[logins]}`.
- Edit, close or reopen: `PATCH /repos/{o}/{r}/issues/{index}` `{title?, body?, state?:"closed"|"open", milestone?, assignees?}`.
- Comments: `GET|POST /repos/{o}/{r}/issues/{index}/comments` (`POST` body `{body}`); edit or delete one: `PATCH|DELETE /repos/{o}/{r}/issues/comments/{id}`.
- Labels on an issue:
  - Add: `POST /repos/{o}/{r}/issues/{index}/labels` `{labels:[...]}`. The entries may be label IDs or label names.
  - Replace: `PUT` the same path. Clear: `DELETE` the same path. Remove one: `DELETE .../labels/{identifier}` (label ID or name).
- Repo labels: `GET|POST /repos/{o}/{r}/labels` (`POST` body `{name, color:"#rrggbb", description?}`).
- Milestones: `GET|POST /repos/{o}/{r}/milestones` (`POST` body `{title, due_on?, description?}`).

## Pull requests

PRs share index numbers with issues. Comments, labels and milestones on a PR all use the `issues/{index}` endpoints above.

- List: `GET /repos/{o}/{r}/pulls?state=open|closed|all&base=&head=`; one PR: `GET /repos/{o}/{r}/pulls/{index}` → `head.sha`, `head.ref`, `base.ref`, `mergeable`, `merged`.
- Create: `POST /repos/{o}/{r}/pulls` `{base, head, title, body?}`. For a PR from a fork, `head` is `forkowner:branch`.
- Edit: `PATCH /repos/{o}/{r}/pulls/{index}` (`title`, `body`, `base`, `state`, `allow_maintainer_edit`, …).
- Diff:
  - `GET /repos/{o}/{r}/pulls/{index}.diff` or `.patch` returns text.
  - `GET .../pulls/{index}/files` returns per-file stats.
  - `GET .../pulls/{index}/commits` returns the commits.
- Reviews: `GET|POST /repos/{o}/{r}/pulls/{index}/reviews`. The `POST` body is `{event:"APPROVED"|"REQUEST_CHANGES"|"COMMENT", body, commit_id?, comments?:[{path, body, new_position|old_position}]}`.
- Merge: `POST /repos/{o}/{r}/pulls/{index}/merge`:
  - Required: `Do`, one of `merge` | `rebase` | `rebase-merge` | `squash` | `fast-forward-only` | `manually-merged`.
  - Optional: `MergeTitleField`, `MergeMessageField`, `delete_branch_after_merge`, `head_commit_id` (refuse if head moved), `merge_when_checks_succeed` (auto-merge).
  - Status: `GET` the same path → 204 if merged, 404 if not. Cancel a scheduled auto-merge: `DELETE` the same path.

## Releases

- List: `GET /repos/{o}/{r}/releases?draft=&pre-release=&q=`; one release: `GET .../releases/{id}` or `GET .../releases/tags/{tag}`; latest: `GET .../releases/latest`.
- Create: `POST /repos/{o}/{r}/releases` `{tag_name, target_commitish?, name?, body?, draft?, prerelease?, hide_archive_links?}`. If the tag does not exist, it is created at `target_commitish`.
- Edit: `PATCH .../releases/{id}`. Delete: `DELETE .../releases/{id}` or `DELETE .../releases/tags/{tag}`. Deleting a release leaves the git tag in place; delete the tag with `DELETE .../tags/{tag}`.
- Upload an asset: `curl -F "attachment=@dist/app.tar.gz" "$API/repos/$O/$R/releases/$ID/assets?name=app.tar.gz"` → 201 `{id, name, size, browser_download_url}`. To link instead of uploading, send the form field `external_url=<url>` in place of `attachment`.
- Assets: `GET .../releases/{id}/assets`; edit: `PATCH .../releases/{id}/assets/{asset_id}` `{name}`; delete: `DELETE` the same path.

## Actions: investigating a failed CI run

Path params: a run's `run_id` is `ActionRun.id`. `index_in_repo` is the `#N` shown on the website; to look a run up by that number, use `GET .../actions/runs?run_number=N`.

Done when you can quote the failing step's log lines and name the cause.

1. **From a PR or commit to a run**:
   - Take the PR's `head.sha`.
   - `GET /repos/{o}/{r}/commits/{sha}/status` → `state` (`pending` | `success` | `error` | `failure` | `warning` | `skipped`). Each entry in `statuses[]` has `{context, status, description, target_url}`, and `target_url` links to the job page.
   - Or query the runs directly: `GET /repos/{o}/{r}/actions/runs?head_sha=<sha>`.
2. **List runs**: `GET /repos/{o}/{r}/actions/runs` → `{total_count, workflow_runs[]}`.
   - Filters: `status=failure` (values: `unknown` | `waiting` | `running` | `success` | `failure` | `cancelled` | `skipped` | `blocked`), `event=push|pull_request|workflow_dispatch`, `ref=refs/heads/main`, `workflow_id=<file>.yml`, `head_sha`, `run_number`.
   - Each run carries `{id, index_in_repo, status, commit_sha, prettyref, workflow_id, title, event, html_url, started, stopped}`.
3. **Jobs of a run**: `GET /repos/{o}/{r}/actions/runs/{run_id}/jobs` → an array of `{id, name, status, attempt, task_id, needs, runs_on, html_url}`.
4. **Steps of a job**: `GET /repos/{o}/{r}/actions/jobs/{job_id}` → `steps[]` of `{number, name, status, started, stopped}`. Step `0` is "Set up job" and the last step is "Complete job".
5. **Log**: `GET /repos/{o}/{r}/actions/jobs/{job_id}/logs` returns plain text for the latest attempt. Narrow it:
   - `step=N`: only that step's slice. Start with the failed step.
   - `q=<substring>`: filter lines; add `qi=true` for case-insensitive. Plain substring match, no regex.
   - `attempt=N`: an earlier attempt.
   - `format=ndjson`: one `{time, content}` object per line.
   - Without `q` or `format=ndjson`, the endpoint supports `Range:`. `-H 'Range: bytes=-20000'` fetches the tail of a long log.
6. **All logs at once**: `GET /repos/{o}/{r}/actions/runs/{run_id}/logs` returns a zip with every job's log (`-o logs.zip`).
7. **Artifacts**: `GET .../actions/runs/{run_id}/artifacts` → `{id, name, size_in_bytes, expired}`; download: `GET .../actions/artifacts/{id}/zip`.

Act on a run:

- Rerun: `POST .../actions/runs/{run_id}/rerun` (whole run) or `POST .../actions/jobs/{job_id}/rerun` (one job).
- Cancel: `POST .../actions/runs/{run_id}/cancel`. Delete: `DELETE .../actions/runs/{run_id}`.
- Manual trigger: `POST .../actions/workflows/{file}.yml/dispatches` `{ref, inputs?:{k:"v"}, return_run_info?:true}`. With `return_run_info`, the response is 201 with the run; otherwise 204.

If `/actions/runs` or `/logs` returns 404 on an existing repo, check `GET /version`: older instances only have `GET .../actions/tasks`, which lists runs without logs. In that case, give the user the run's web page `https://<host>/{o}/{r}/actions/runs/{index_in_repo}` for the log.

## Other endpoints

One line each; check the spec for the path before calling.

- Actions secrets and variables:
  - Repo secret: `PUT|DELETE /repos/{o}/{r}/actions/secrets/{name}` `{data}`.
  - Repo variable: `GET|POST|PUT|DELETE /repos/{o}/{r}/actions/variables/{name}` `{value}`.
  - The same shapes exist under `/orgs/{org}/actions/…` and `/user/actions/…`.
- Runners: `/repos/{o}/{r}/actions/runners`, `/orgs/{org}/actions/runners`, `/user/actions/runners`, `/admin/actions/runners`; a registration token is under `…/runners/registration-token`.
- Packages: `/packages/{owner}` (list), `/packages/{owner}/{type}/{name}/{version}`.
- Orgs and teams: `/orgs/{org}`, `/orgs/{org}/members`, `/orgs/{org}/teams`, `/teams/{id}/members/{user}`.
- Collaborators: `PUT|DELETE /repos/{o}/{r}/collaborators/{user}` `{permission:"read"|"write"|"admin"}`.
- Commit statuses: `POST /repos/{o}/{r}/statuses/{sha}` `{state, context, target_url?, description?}`.
- Admin: `/admin/users`, `/admin/orgs`, `/admin/cron`.
- Current user: `GET /user`; search: `GET /repos/search?q=`, `GET /users/search?q=`.

## When this file is not enough

The instance's own spec is authoritative. Endpoints are only compatible within one major version.

1. `GET $FORGEJO_URL/api/v1/version` gives the instance version.
2. Read the instance's spec:
   - Paths matching a word: `curl -s $FORGEJO_URL/swagger.v1.json | jq '.paths | keys[] | select(test("<word>"))'`.
   - Parameters of one path: `jq '.paths["/repos/{owner}/{repo}/…"]'`.
   - A body schema: `jq '.definitions.<Name>'`.
