---
name: github
description: "Interact with GitHub using the `gh` CLI. Use `gh issue`, `gh pr`, `gh run`, and `gh api` for issues, PRs, CI runs, and advanced queries."
metadata:
  {
    "openclaw":
      {
        "emoji": "🐙",
        "requires": { "bins": ["gh"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gh",
              "bins": ["gh"],
              "label": "Install GitHub CLI (brew)",
            },
            {
              "id": "apt",
              "kind": "apt",
              "package": "gh",
              "bins": ["gh"],
              "label": "Install GitHub CLI (apt)",
            },
          ],
      },
  }
---

# GitHub Skill

Use the `gh` CLI to interact with GitHub. Always specify `--repo owner/repo` when not in a git directory, or use URLs directly.

## Issues

Create:

```bash
gh issue create --repo owner/repo --title "Bug: ..." --body "Description" --label "bug"
```

List and filter:

```bash
gh issue list --repo owner/repo --state open --label "bug" --limit 20
gh issue list --repo owner/repo --assignee @me
```

Comment:

```bash
gh issue comment 42 --repo owner/repo --body "Working on this"
```

Close with reason:

```bash
gh issue close 42 --repo owner/repo --reason completed --comment "Fixed in #45"
```

## Pull Requests

Create:

```bash
gh pr create --repo owner/repo --title "fix: description" --body "Closes #42" --base main
```

Check CI status:

```bash
gh pr checks 55 --repo owner/repo
```

Review:

```bash
gh pr review 55 --repo owner/repo --approve --body "LGTM"
gh pr review 55 --repo owner/repo --request-changes --body "Please fix X"
```

Merge:

```bash
gh pr merge 55 --repo owner/repo --squash --delete-branch
```

## CI/CD Debugging

List recent workflow runs:

```bash
gh run list --repo owner/repo --limit 10
```

View a specific run:

```bash
gh run view <run-id> --repo owner/repo
```

View logs for failed steps only:

```bash
gh run view <run-id> --repo owner/repo --log-failed
```

Re-run failed jobs:

```bash
gh run rerun <run-id> --repo owner/repo --failed
```

Trigger a workflow manually:

```bash
gh workflow run ci.yml --repo owner/repo --ref main
```

Download artifacts:

```bash
gh run download <run-id> --repo owner/repo --dir /tmp/artifacts
```

## Releases

Create:

```bash
gh release create v1.0.0 --repo owner/repo --title "v1.0.0" --generate-notes
```

With assets:

```bash
gh release create v1.0.0 dist/*.tar.gz --repo owner/repo --title "v1.0.0"
```

List:

```bash
gh release list --repo owner/repo --limit 5
```

## Labels

```bash
gh label list --repo owner/repo
gh label create "priority/high" --repo owner/repo --color "FF0000"
```

## API for Advanced Queries

The `gh api` command accesses data not available through other subcommands.

Get PR with specific fields:

```bash
gh api repos/owner/repo/pulls/55 --jq '.title, .state, .user.login'
```

GraphQL (e.g., get all open PRs with labels):

```bash
gh api graphql -f query='
  query {
    repository(owner: "owner", name: "repo") {
      pullRequests(states: OPEN, first: 10) {
        nodes { number title labels(first: 5) { nodes { name } } }
      }
    }
  }
'
```

## JSON Output

Most commands support `--json` for structured output. Use `--jq` to filter:

```bash
gh issue list --repo owner/repo --json number,title --jq '.[] | "\(.number): \(.title)"'
gh pr list --repo owner/repo --json number,title,statusCheckRollup --jq '.[] | "\(.number): \(.title) [\(.statusCheckRollup | map(.conclusion) | join(","))]"'
```
