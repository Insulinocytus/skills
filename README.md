# Skills

This repository collects agent skills from other repositories.

Sources are managed in `sources.json`. Updates are installed with the `skills` CLI, copied into this repository, committed, and pushed by GitHub Actions.

## Add A Source

Add an entry to `sources.json`:

```json
{
  "sources": [
    {
      "package": "owner/repo",
      "skill": "foobar",
      "target": "skills/foobar"
    }
  ]
}
```

Fields:

- `package`: package or repository passed to `npx skills add`, for example `owner/repo` or a GitHub URL
- `skill`: skill name passed to `--skill`
- `target`: directory in this repository, defaults to `skills/<skill>`

## Updates

`.github/workflows/update-skills.yml` runs weekly and can also be triggered manually from GitHub Actions.

For each source, the workflow runs `npx skills add`, copies `.agents/skills/<skill>` to `target`, removes `.agents/skills/<skill>`, commits changed skills, and pushes the result.
