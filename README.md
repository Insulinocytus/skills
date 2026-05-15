# Skills

This repository collects agent skills from other repositories.

Sources are managed in `sources.json`. Updates are installed with the `skills` CLI, copied into this repository, committed, and pushed by GitHub Actions.

## Add A Source

Add an entry to `sources.json`:

```json
[
  {
    "package": "owner1/repo1",
    "skill": [
      "foobar1",
      "foobar2"
    ]
  },
  {
    "package": "owner2/repo2",
    "skill": [
      "foobar3",
      "foobar4"
    ]
  }
]
```

Fields:

- `package`: package or repository passed to `npx skills add`, for example `owner/repo` or a GitHub URL
- `skill`: skill names passed to `--skill`

## Updates

`.github/workflows/update-skills.yml` runs weekly and can also be triggered manually from GitHub Actions.

For each source, the workflow runs `npx skills add`, copies `.agents/skills/<skill>` to `skills/<skill>`, removes `.agents/skills/<skill>`, commits changed skills, and pushes the result.
