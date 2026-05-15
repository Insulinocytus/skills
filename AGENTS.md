# AGENTS.md

## Project Overview

This repository is a curated mirror of agent skills from external repositories. The root project stores source definitions in `sources.json`, mirrored skill directories in `skills/`, and a GitHub Actions workflow that periodically refreshes those skills.

The repository is documentation and asset heavy. There is no root application package, development server, database, or production build pipeline.

Key files and directories:

- `sources.json`: canonical list of upstream repositories and skill names to install.
- `skills/<skill-name>/`: mirrored skill contents, normally copied from `.agents/skills/<skill-name>` by the update workflow.
- `.github/workflows/update-skills.yml`: weekly/manual/push-triggered synchronization workflow.
- `README.md`: short human-facing repository overview and source-editing instructions.

## Setup Commands

- No root dependency installation is required for normal documentation edits.
- Use Node.js 18 or newer when working in `skills/react-components`, because that skill has its own `package.json` and validator dependency.
- Install the `react-components` skill dependencies only when validating or editing that skill: `npm install --prefix skills/react-components`.
- The update workflow uses `npx -y skills add ... --agent opencode --copy -y`; local runs require network access and the npm `skills` CLI.
- The workflow also uses `jq` on Linux. Install it before reproducing workflow logic outside GitHub Actions.

## Development Workflow

- To add or remove mirrored skills, edit `sources.json`; do not hand-edit generated mirrors unless the task explicitly targets a local skill change.
- Preserve the `sources.json` shape: each item has `repository` and `skill`, where `skill` is an array of skill names passed as repeated `--skill` arguments.
- Generated skill directories are replaced wholesale by the update workflow. If a change belongs upstream, make it in the upstream skill source instead of only in this mirror.
- Keep skill directory names stable and lowercase where possible. Note that upstream names may contain special forms such as `react:components` in `sources.json`, while the mirrored directory is `skills/react-components`.
- Avoid running broad formatting over `skills/`; many files are vendored reference material from different sources.

## Updating Skills

GitHub Actions is the source of truth for automated refreshes. It runs on:

- Manual dispatch from Actions.
- Pushes to `main` that change `sources.json` or `.github/workflows/update-skills.yml`.
- Weekly schedule: `0 0 * * 0`.

Workflow behavior:

- Reads each source from `sources.json` with `jq`.
- Runs `npx -y skills add "$repository" --skill ... --agent opencode --copy -y`.
- Copies `.agents/skills/<skill>` to `skills/<skill>`.
- Removes the temporary `.agents/skills/<skill>` directory.
- Commits each changed mirrored skill as `update <skill>` and pushes changes.

When changing the workflow, verify that it still stages only the intended `skills/<skill>` paths and does not commit temporary `.agents/` content.

## Testing And Validation

There is no root test suite. Use targeted checks based on the files you changed.

- Validate a skill folder with the bundled validator when Python and PyYAML are available: `python skills/skill-creator/scripts/quick_validate.py skills/<skill-name>`.
- Validate the `create-agentsmd` skill metadata example: `python skills/skill-creator/scripts/quick_validate.py skills/create-agentsmd`.
- Validate `react-components` generated TSX components from that package: `npm run --prefix skills/react-components validate -- <path-to-component.tsx>`.
- After editing `sources.json`, confirm it parses as JSON. Prefer a parser check such as `python -m json.tool sources.json` or `jq empty sources.json`.
- After editing `.github/workflows/update-skills.yml`, inspect the workflow syntax and command quoting carefully; there is no local CI wrapper in this repository.

Do not document or assume root-level `npm test`, `npm run build`, `pnpm`, `yarn`, or `make` commands unless those files are added later.

## Code Style

- Use standard Markdown for repository documentation and skill files.
- Keep instructions agent-focused, specific, and executable.
- Prefer ASCII punctuation for new repository-level documentation unless the surrounding file already uses non-ASCII text.
- Keep `sources.json` two-space indented and human-readable.
- Preserve upstream skill content and style inside `skills/<skill-name>/` unless the task is specifically to modify that skill.
- For shell snippets in workflow-related docs, match the existing Bash style used in `.github/workflows/update-skills.yml`.
- For JavaScript under `skills/react-components`, preserve the existing ESM style, semicolons, single quotes for imports, and Node built-in `node:` imports.

## Repository Conventions

- Treat `skills/` as mirrored vendor content. Make surgical edits and avoid unrelated cleanup.
- Do not add large generated dependency directories such as `node_modules/`.
- Do not commit temporary `.agents/` directories created by local `skills add` runs.
- Keep root documentation concise; detailed skill behavior belongs in each skill's `SKILL.md` and support files.
- If adding a new source, update `README.md` only if the process or schema changes; adding another `sources.json` entry usually does not require README changes.

## Security Considerations

- Do not commit API tokens, npm auth tokens, GitHub tokens, Cloudflare credentials, or local environment files.
- Be cautious when running `npx skills add` from external repositories; it fetches and installs third-party skill content.
- Review vendored skill updates for unexpected executable scripts before committing or approving generated changes.
- The GitHub Actions workflow has `contents: write` permission so it can commit refreshed skills. Avoid broadening permissions without a concrete need.

## Pull Request Guidelines

- For source list changes, summarize which upstream repository and skill names were added, removed, or changed.
- For mirrored skill changes, identify whether the change was generated from upstream or manually edited in this repository.
- Before finishing a change, run the targeted validation commands that match the changed files and report any checks that could not be run.
- Do not create commits from automation or agent work unless the user explicitly asks for a commit.

## Troubleshooting

- If `npx skills add` produces no `.agents/skills` directory, check the repository name, skill names, network access, and CLI output.
- If a skill name in `sources.json` does not match the mirrored directory name, inspect the generated `.agents/skills` directory after a local run; upstream tooling may normalize names.
- If `quick_validate.py` fails with `No module named yaml`, install PyYAML in the active Python environment or use an environment that already includes it.
- If `npm run --prefix skills/react-components validate -- <file>` fails before parsing a component, run `npm install --prefix skills/react-components` first.
