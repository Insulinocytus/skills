---
name: nexusmods-api
description: Nexus Mods API cheat sheet (v3 REST plus v1/v2 gaps). Use when calling the Nexus Mods API, or publishing or updating a mod on Nexus Mods.
---

# Nexus Mods API

A compressed digest of <https://api.nexusmods.com/openapi.yaml> (v3, ~130 KB). Work from this file; open the spec only for what it leaves out (see [When this file is not enough](#when-this-file-is-not-enough)).

## Auth

Read the key from `NEXUS_API_KEY`, then `NEXUSMODS_API_KEY`. If neither is set, stop and tell the user to set one of them as an environment variable, with a key created at <https://www.nexusmods.com/settings/api-keys>.

Every v1, v2 and v3 call sends the key as header `apikey: <key>` and a descriptive `User-Agent` (e.g. `<tool-name>/<version>`). v3 bodies are JSON (`Content-Type: application/json`); responses wrap the payload in `data`.

## Writes need confirmation

Before any call that publishes or cannot be undone, list the exact request bodies, every optional field included with the value you will send or its default, and wait for the user's go-ahead: creating a file or file version, adding a changelog, editing the mod page, quarantining a version.

## IDs

v3 path `{id}`s are global IDs, not the numbers in site URLs.

| Thing | Site URL / v1 / v2 | v3 `{id}` | Resolve with |
|---|---|---|---|
| Mod | `/lastepoch/mods/23` → `23` | `18433999634455` | `GET /games/{game_domain}/mods/{game_scoped_id}` → `data.id`, `data.game_id` |
| Mod file | file id `228` on the Files tab | `8042586` | `GET /mods/{id}/files` → `data.mod_files[].id` |
| File version | — | `18433999634660` | `GET /mod-files/{id}/versions` → `data.versions[].id` |

v2 GraphQL takes the numeric `gameId` (e.g. `4292`, from `data.game_id`), never the domain name.

## Common v3 calls

Base URL `https://api.nexusmods.com/v3`. Most `mods`/`mod-files` endpoints carry an **Experimental** badge: on an unexpected 4xx, re-check that path in the spec.

### Read

- `GET /games/{game_domain}/mods/{game_scoped_id}` → `{id, game_scoped_id, game_id, name}`.
- `GET /mods/{id}/files` → `mod_files[]`: `{id, name, is_active, last_file_uploaded_at, versions_count, archived_count, removed_count}`.
- `GET /mod-files/{id}` → one mod file, same shape.
- `GET /mod-files/{id}/versions` → `versions[]`: `{id, file{id,name}, position, game_scoped_id, name, version, category, uploaded_at, is_primary}`.
- `GET /mod-file-versions/{id}`, `GET /games/{game_domain}/mod-file-versions/{game_scoped_id}` → one version, same shape.

### Upload (every new file or version starts here)

1. Create a session:
   - Single part, file ≤ 100 MiB: `POST /uploads` `{size_bytes, filename, md5}` → `{id, state, presigned_url}`. `md5` is the hex digest; it becomes required on 2026-12-01.
   - Multipart, any size (the official upload-action always uses this): `POST /uploads/multipart` `{size_bytes, filename}` → `{id, state, part_size_bytes, part_presigned_urls[], complete_presigned_url}`.
2. Send the bytes (no `apikey` header on presigned URLs):
   - Single part: `PUT presigned_url` with `Content-Type: application/octet-stream`, `Content-Disposition: attachment; filename="<filename>"`, `Content-MD5: <base64 of the same md5>`.
   - Multipart: `PUT` each `part_presigned_urls[n]` with part `n` (`part_size_bytes` each, last may be shorter) and `Content-Type: application/octet-stream`; keep each response `ETag` with its quotes stripped. Then `POST complete_presigned_url`, `Content-Type: application/xml`, body `<CompleteMultipartUpload><Part><PartNumber>1</PartNumber><ETag>…</ETag></Part>…</CompleteMultipartUpload>`.
3. `POST /uploads/{id}/finalise`.
4. Poll `GET /uploads/{id}` until `data.state` is `available` (states: `created`, `available`). upload-action backs off from 2 s ×1.5 up to 30 s, 60 attempts.

### Publish

- New version of an existing file (keeps the update chain mod managers follow): `POST /mod-files/{file_id}/versions`
  - Required: `upload_id`, `name`, `version`, `file_category` (`main` | `optional` | `miscellaneous`).
  - Optional: `description`, `update_mod_version` (bump the mod page version, default false), `archive_existing_file` (move the previous version to Archived, default false), `primary_mod_manager_download` (make this version the default download; send `true` when it replaces the main download), `allow_mod_manager_download`, `show_requirements_pop_up`, `previous_version_id`.
  - The previous version drops from `main` to `old_version` but keeps `is_primary` unless the new version claims it. No v3 endpoint changes `is_primary` afterwards; fixing it takes the website's Manage files page.
  - → `{file{id, game_scoped_id, name, file_category}, version{id, position}}`.
- Brand-new file beside the existing ones: `POST /mod-files` with the same fields plus required `mod_id` (v3 mod id), minus `archive_existing_file`/`previous_version_id`.
- Changelog: `POST /mods/{id}/changelogs` `{version, changelog}` → 201. Append-only.
- Mod page: `PATCH /mods/{id}` with `summary` (≤ 350 chars, whitespace collapsed) and/or `description` (BBCode); omitted fields stay. → 204.
- Rename a mod file: `PUT /mod-files/{id}` `{name}`. This is the only field it changes.

## Outside v3

Three common needs v3 cannot meet:

- Validate the key and get `user_id`: `GET https://api.nexusmods.com/v1/users/validate.json`. Source: [v1 SwaggerHub](https://app.swaggerhub.com/apis-docs/NexusMods/nexus-mods_public_api_params_in_form_data/1.0#/).
- List a user's mods: `POST https://api.nexusmods.com/v2/graphql` with `{ mods(filter:{uploaderId:[{value:"<user_id>", op:EQUALS}]}, count:50) { nodes { modId uid name version game { domainName } } } }`. `uid` is the v3 mod id. Source: [GraphQL docs](https://graphql.nexusmods.com/), `ModsFilter.uploaderId`.
- Read the current page text: `{ mod(modId: 23, gameId: 4292) { summary description version } }` on the same endpoint. Source: same GraphQL docs.

## Pitfalls

- **Presigned PUT signature**: it covers `content-type`, so the PUT must send `Content-Type: application/octet-stream`; any other value returns 403 `SignatureDoesNotMatch`. The spec omits this; upload-action sends it.
- **Single-part headers**: `Content-Disposition` must match the `filename` sent to `POST /uploads`, and `Content-MD5` is base64 while the body's `md5` is hex. Both are signed.
- **Description encoding**: `PATCH /mods/{id}` turns each `\n` into `<br />` and escapes existing `<br />`. v2 returns the description with `<br />` already inserted, so strip every `<br />` before sending it back, then re-read to confirm it round-trips.
- **Archiving**: no v3 endpoint archives an arbitrary file. `archive_existing_file` archives only the previous version in the same file's chain; older standalone files must be archived on the website.
- **Changelogs** are append-only: a second call for the same version adds text, nothing edits or deletes it.
- **User-Agent**: v2 GraphQL rejects library-default agents such as `Python-urllib/3.x` with 403 `error code: 1010` (Cloudflare); a custom `User-Agent` fixes it.
- **Name and version patterns**: `name` matches `^[a-zA-Z0-9 _'().-]+$`, `version` matches `^[a-zA-Z0-9.-]+$`, both ≤ 50 chars. Non-ASCII, `+` and `/` are rejected.
- **Deprecated**: `POST /mod-file-update-groups/{group_id}/versions` is replaced by `POST /mod-files/{id}/versions`.

## Other v3 endpoints

One line each; read the spec section for the path before calling.

- `GET /games/{game_domain}/trending-mods`: trending mods.
- `GET /games/{game_domain}/dlcs`: a game's DLCs.
- `PUT /mods/{id}/toggle-legacy-mod-requirements`: toggle legacy requirements.
- `POST /mods/batch`, `POST /mod-file-versions/batch`: batch details.
- `POST /mod-file-versions/{id}/quarantine`: quarantine a version.
- `POST /mod-file-versions/{id}/download-repacked`: download the repacked archive.
- `POST /mod-file-versions/move`, `POST /mod-file-versions/move-to-new-mod-file`: move versions between files.
- `GET|PUT /mod-file-versions/{id}/dependencies/ranges`, `GET|PUT /mod-file-versions/{id}/dependencies/dlc`, `GET /mod-file-versions/{id}/dependencies`, `GET /mod-file-versions/{id}/dependencies/ranges/materialized`, `POST /mod-file-versions/dependencies/ranges/materialized/batch`: dependencies.
- `POST /collections`, `PATCH /collections/{id}`, `POST /collections/{id}/revisions`: collections.
- `GET /vortex/extensions`, `POST /vortex/detected-games`: Vortex client data.

## When this file is not enough

In order:

1. <https://api.nexusmods.com/openapi.yaml>: authoritative v3 spec. Search it for the path or `operationId`.
2. [v1 SwaggerHub](https://app.swaggerhub.com/apis-docs/NexusMods/nexus-mods_public_api_params_in_form_data/1.0#/) (JSON: `https://api.swaggerhub.com/apis/NexusMods/nexus-mods_public_api_params_in_form_data/1.0`).
3. <https://graphql.nexusmods.com/>: v2 GraphQL schema.
4. [Nexus-Mods/upload-action `src/index.ts`](https://github.com/Nexus-Mods/upload-action/blob/main/src/index.ts): official reference implementation of upload and publish.
