---
name: create-skill
description: Create a new SKILL.md for an AI agent skill. Use when defining a reusable workflow, documenting an agent capability, converting a process into an executable skill, or standardizing how agents should decide, act, recover, and verify completion.
---

# Create Skill

## Overview

A good `SKILL.md` is an executable operating manual for an AI agent. It explains when the skill should be used, what principles guide the work, how to execute the workflow, how to handle failure, and how to verify the result.

The goal is to turn vague know-how into a repeatable procedure that another agent can follow without relying on hidden context.

**Triggerable:** The skill must clearly describe when it applies. The `description` field and `When to Use` section should make activation obvious.

**Actionable:** The skill should contain concrete steps, commands, templates, examples, and decision rules. It should help the agent do the work, not merely understand the topic.

**Verifiable:** The skill should end with checks that confirm the work was completed correctly.

## When to Use

- Creating a new skill for an AI coding agent
- Converting a repeated workflow into reusable instructions
- Standardizing project, engineering, review, deployment, or communication processes
- Writing a skill that agents should invoke automatically based on task context
- Improving an existing skill that is too vague, too theoretical, or hard to verify

## Skill Structure

Use this structure as the default skeleton:

```md
---
name: skill-name
description: Use when...
---

# Skill Title

## Overview

Explain the purpose, core idea, and guiding principles.

## When to Use

List the situations where this skill should be invoked.

## Core Principles

Define the decision rules that shape the workflow.

## Standard Workflow

Describe the normal execution path.

## Implementation Templates

Provide copyable commands, configs, snippets, checklists, prompts, or examples.

## Bundled Resources

Document any reusable files that belong with the skill, especially scripts and assets. Explain what each resource is for, when the agent should use it, and how to verify it works.

## Agent Feedback Loop

Explain how the agent should react to errors, failed checks, review comments, or incomplete output.

## Common Rationalizations

List common excuses or shortcuts, then define the correct response.

## Red Flags

List warning signs that the process is being misused or skipped.

## Verification

Provide a final checklist for completion.
```

## Frontmatter

Every skill starts with YAML frontmatter:

```yaml
---
name: create-skill
description: Create a new SKILL.md for an AI agent skill. Use when defining a reusable workflow, documenting an agent capability, converting a process into an executable skill, or standardizing how agents should decide, act, recover, and verify completion.
---
```

### `name`

Use a short, lowercase, kebab-case name.

Good examples:

```text
create-skill
ci-cd-and-automation
debugging-and-error-recovery
database-migration
pull-request-review
```

Avoid vague names:

```text
best-practices
misc
project-stuff
general-help
```

### `description`

The description is the activation rule for the agent. Write it as a practical trigger, not as marketing copy.

Good:

```text
Use when setting up or modifying CI pipelines, configuring quality gates, or debugging CI failures.
```

Weak:

```text
This skill explains CI/CD.
```

A strong description usually contains:

- The main action
- The target domain
- The situations that should trigger the skill
- Any important boundaries

## Overview

The overview should define the skill's purpose and judgment model.

It should answer:

- What is this skill trying to achieve?
- What problem does it prevent?
- What principles should guide decisions?
- What tradeoffs matter most?

Use bold principle labels when helpful:

```md
**Shift Left:** Catch problems as early as possible.

**Faster is Safer:** Smaller batches reduce release risk.

**Verify Before Merge:** Every change must pass automated checks before review.
```

The overview should be short enough to read quickly, but strong enough to shape the agent's behavior.

## When to Use

This section lists concrete trigger scenarios.

Good examples:

```md
## When to Use

- Setting up a new project's CI pipeline
- Adding or modifying automated checks
- Debugging CI failures
- Configuring deployment pipelines
- Making a process repeatable for future agents
```

Write triggers as situations, not abstract concepts.

Good:

```text
Debugging a failed database migration
```

Weak:

```text
Database knowledge
```

## Core Principles

Use this section when the skill needs a decision framework.

Examples:

```md
## Core Principles

- Prefer repeatable automation over manual steps.
- Prefer explicit checks over trust.
- Fix failing checks at the source.
- Keep the workflow small enough to run frequently.
```

Principles should be opinionated. They tell the agent what to prioritize when the task is ambiguous.

For skills that include bundled resources, add principles that control when the agent should rely on files instead of improvising.

Examples:

```md
## Core Principles

- Use scripts for fragile, repetitive, or deterministic operations.
- Use assets for output files, templates, sample projects, images, icons, fonts, or boilerplate that should be copied or adapted.
- Keep resource usage explicit: name the file, describe the purpose, and define the expected result.
- Test scripts before relying on them in the workflow.
```

## Standard Workflow

This is the main execution path.

Use numbered steps, diagrams, or process blocks:

```md
## Standard Workflow

1. Identify the task type and expected output.
2. Check whether an existing template applies.
3. Identify whether the skill needs bundled scripts or assets.
4. Draft the skill with frontmatter, overview, triggers, workflow, templates, bundled resources, red flags, and verification.
5. Remove vague advice and replace it with concrete actions.
6. Test reusable scripts or inspect reusable assets when they are included.
7. Add a verification checklist.
```

For process-heavy skills, use a diagram:

```text
Task arrives
    │
    ▼
Check trigger conditions
    │
    ▼
Apply standard workflow
    │
    ▼
Run verification checklist
    │
    ▼
Skill complete
```

## Implementation Templates

Templates are the most reusable part of a skill. Include anything the agent can copy and adapt:

- Commands
- Config files
- Code snippets
- Prompt templates
- Checklists
- Tables
- Example directory structures
- Example input/output pairs

Example:

````md
### Minimal Skill Template

```md
---
name: example-skill
description: Use when...
---

# Example Skill

## Overview

Purpose and principles.

## When to Use

- Trigger 1
- Trigger 2

## Standard Workflow

1. Step one.
2. Step two.
3. Step three.

## Verification

- [ ] Output exists
- [ ] Output matches the requested format
- [ ] Edge cases are handled
```
````

````

## Bundled Resources

Some skills need reusable files in addition to the `SKILL.md` instructions. Use bundled resources when the skill benefits from deterministic execution, reusable templates, or files that should be copied into the final output.

Use this directory shape when resources are useful:

```text
skill-name/
├── SKILL.md
├── scripts/
│   └── reusable-task.py
└── assets/
    └── reusable-template-or-file
````

### Scripts

Use `scripts/` for executable code that the agent should run or adapt. Scripts are appropriate when the same operation would otherwise be rewritten repeatedly, when exact behavior matters, or when a task is easy to describe but easy to implement incorrectly.

Good script candidates:

- File conversion or transformation
- PDF, image, spreadsheet, archive, or text processing
- Project scaffolding
- Validation checks
- Repetitive CLI workflows
- Data cleanup or migration helpers

For every script, document:

- The script path
- The task it performs
- Required inputs and outputs
- Example command
- Expected success signal
- Failure handling

Example:

````md
## Bundled Resources

### Scripts

- `scripts/rotate_pdf.py`: Rotate selected PDF pages and write a new PDF.

Usage:

```bash
python scripts/rotate_pdf.py --input input.pdf --output rotated.pdf --pages 1,3 --degrees 90
```
````

Expected result: `rotated.pdf` exists, opens successfully, and only the requested pages are rotated.

````

Script rules:

- Prefer scripts for deterministic operations over asking the agent to rewrite code from memory.
- Keep script interfaces small and explicit.
- Validate inputs and fail with actionable error messages.
- Test scripts before documenting them as part of the standard workflow.
- Mention environment assumptions, such as Python packages, shell tools, or OS-specific behavior.

### Assets

Use `assets/` for files that the agent should copy, modify, or use in the final output. Assets are appropriate when the skill depends on reusable output material rather than more instructions.

Good asset candidates:

- Document templates
- Slide templates
- Frontend or backend boilerplate
- Brand files, logos, icons, and images
- Sample configs
- Font files used internally by the skill
- Example project skeletons

For every asset or asset group, document:

- The asset path
- What the asset is used for
- Whether it should be copied, modified, referenced, or preserved unchanged
- Which parts the agent may edit
- Required output checks

Example:

```md
## Bundled Resources

### Assets

- `assets/frontend-template/`: Minimal frontend project skeleton. Copy this directory when creating a new frontend app, then modify the app name, routes, components, and configuration for the user's task.
- `assets/report-template.docx`: Base report template. Preserve styles and section structure; replace placeholder text with generated content.
````

Asset rules:

- Use assets when the output should preserve a known structure, style, or boilerplate.
- State which files are safe to modify.
- State which files should remain unchanged.
- Verify the final output still contains required template structure or branding.
- Do not add assets that are unrelated to the skill's normal outputs.

## Agent Feedback Loop

Skills should explain how the agent should respond when something fails.

Example:

```md
## Agent Feedback Loop

When a check fails:

1. Read the exact failure message.
2. Identify the smallest failing unit.
3. Fix the root cause.
4. Re-run the relevant check.
5. Continue only after the check passes.
```

For AI coding workflows, include explicit feedback prompts:

```text
"The CI pipeline failed with this error:
[paste exact error]

Fix the root cause and verify locally before pushing again."
```

This section turns the skill into a closed loop instead of a one-pass instruction.

## Common Rationalizations

This section prevents predictable shortcuts.

Use a table:

```md
| Rationalization                            | Correct Response                                                |
| ------------------------------------------ | --------------------------------------------------------------- |
| "This is obvious, no need to document it." | If agents need to repeat it, document the trigger and workflow. |
| "The skill is too small."                  | Small skills are useful when they are frequently triggered.     |
| "Examples are enough."                     | Examples need decision rules and verification checks.           |
| "We'll clean it up later."                 | A vague skill creates vague agent behavior immediately.         |
```

Keep this section practical. It should target the excuses that cause process failure.

## Red Flags

Red flags are warning signs that the skill is weak or misused.

Examples:

```md
## Red Flags

- The description does not clearly say when to use the skill
- The overview explains concepts but gives no action path
- The workflow depends on hidden project context
- The skill has no examples or templates
- Repetitive or fragile operations are described only as prose when a script would be safer
- Required output templates or boilerplate are described only in text when an asset would be more reliable
- Bundled scripts or assets are listed without usage instructions
- The verification section is missing
- The agent can complete the skill without producing a concrete artifact
- The skill contains broad advice that cannot be checked
```

## Verification

Every skill should end with a checklist.

Example:

```md
## Verification

- [ ] Frontmatter has a clear `name`
- [ ] Frontmatter has a trigger-oriented `description`
- [ ] `Overview` explains the purpose and principles
- [ ] `When to Use` lists concrete activation scenarios
- [ ] `Standard Workflow` gives executable steps
- [ ] Templates or examples are included where useful
- [ ] Scripts are included for repetitive or deterministic operations when useful
- [ ] Assets are included for reusable output templates, boilerplate, or media when useful
- [ ] Bundled resources have paths, usage rules, and verification checks
- [ ] Failure handling or feedback loop is defined
- [ ] Common shortcuts are addressed
- [ ] Red flags are listed
- [ ] Final checklist confirms completion
```

Verification should be specific enough that another agent can decide whether the skill is complete.

## Writing Rules

- Write for an agent that has no hidden context.
- Prefer concrete workflows over abstract advice.
- Use examples when a rule may be misunderstood.
- Keep sections skimmable.
- Put reusable commands and templates in code blocks.
- Use `scripts/` for repeatable executable operations.
- Use `assets/` for reusable output files, templates, media, and boilerplate.
- Make trigger conditions explicit.
- Make completion criteria testable.
- Remove anything that cannot affect the agent's behavior.

## Minimal Complete Example

```md
---
name: pull-request-review
description: Review pull requests for correctness, maintainability, security, test coverage, and merge readiness. Use when evaluating a PR before merge or responding to review requests.
---

# Pull Request Review

## Overview

Review the change as a production risk control. The goal is to catch defects before merge while keeping feedback specific, actionable, and tied to the changed code.

**Review the diff first:** Focus on what changed and what risks the change introduces.

**Block on correctness:** Bugs, security issues, broken tests, and missing critical coverage must be fixed before merge.

**Prefer actionable comments:** Every finding should explain the issue, impact, and expected fix.

## When to Use

- Reviewing a pull request before merge
- Checking AI-generated code changes
- Responding to requested review comments
- Re-reviewing after fixes

## Standard Workflow

1. Read the PR description and linked issue.
2. Inspect the diff file by file.
3. Identify correctness, security, migration, compatibility, and test risks.
4. Run or inspect relevant checks.
5. Leave specific comments on blocking issues.
6. Approve only when the change is safe to merge.

## Bundled Resources

### Scripts

- `scripts/check_pr.py`: Optional helper for collecting changed files and running repository-specific review checks. Use it when the repository includes this script and the review requires local verification.

### Assets

- `assets/review-comment-template.md`: Optional reusable comment format. Use it when leaving structured review comments. Preserve the issue/impact/fix structure.

## Common Rationalizations

| Rationalization                 | Correct Response                                                                 |
| ------------------------------- | -------------------------------------------------------------------------------- |
| "CI passed, so review is done." | CI is one signal; review still checks design, maintainability, and missed cases. |
| "The change is small."          | Small changes can still break production behavior.                               |
| "The author is trusted."        | Review the code, not the author.                                                 |

## Red Flags

- No tests for behavior changes
- Large unrelated changes in one PR
- Migration without rollback or compatibility plan
- Security-sensitive code without explicit review
- Review comments are vague or preference-only

## Verification

- [ ] Diff was reviewed
- [ ] Tests or checks were considered
- [ ] Relevant scripts or assets were used when present
- [ ] Blocking issues are clearly identified
- [ ] Comments are actionable
- [ ] Approval decision is explicit
```
