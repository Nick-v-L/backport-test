# Backport-test

This repository runs automated backport tests when triggered via a `repository_dispatch` event (event type `backport-test`). It
creates test PRs in this repo that exercise the Nick-v-L/backport app (comment-based backport, label-based backport, and custom-branch backport), merges them, waits for backport PRs to be opened in this repo, and attempts to merge those backports.

## Quick overview

- Receiver workflow: `.github/workflows/receive-dispatch.yml` — listens for `repository_dispatch` events and runs the three test scenarios.
- Test file modified: `SAMPLE-CHANGE-COMMENT.txt` — the workflow appends a timestamped line to this file in each test PR.

## Setup

1. Install the receiver workflow in this repository (already present at `.github/workflows/receive-dispatch.yml`).
2. Decide how to trigger the test run from the Nick-v-L/backport repository:
   - Preferred: add a small workflow in `Nick-v-L/backport` that sends a `repository_dispatch` event to this repo on `push` to `main`.
   - Alternative: trigger manually using the `gh` CLI or `curl` (examples below).
3. (Optional) If you want the test run to create a status check on the triggering commit in `Nick-v-L/backport`, add a PAT with `repo` and `checks:write` permissions as a secret in this repo (suggested name: `BACKPORT_TEST_PAT`). The default behavior only needs the test repo's workflow permissions to create PRs inside this repo.

### Trigger examples (add to `Nick-v-L/backport`)

Below is an example workflow you can place in `Nick-v-L/backport` to dispatch a test run when `main` is pushed. Replace `Nick-v-L/backport-test` in `owner`/`repo` if different.

```yaml
name: Dispatch Backport Test

on:
  push:
    branches: [main]

jobs:
  dispatch-test:
    runs-on: ubuntu-latest
    steps:
      - name: Dispatch to backport-test repo
        uses: actions/github-script@v6
        with:
          script: |
            await github.repos.createDispatchEvent({
              owner: 'Nick-v-L',
              repo: 'backport-test',
              event_type: 'backport-test',
            });
```

## Manual dispatch examples

Using `gh`:

```bash
gh api repos/Nick-v-L/backport-test/dispatches -f event_type=backport-test
```

## How detection works

The receiver workflow tries to detect backport PRs by searching open PRs for either:

- a body that references the original PR number (e.g. `#123`),
- a title containing the word `backport`, or
- a head ref that includes the substring `backport`.

If your copy of the backport app uses a different naming convention, update the detection logic in `.github/workflows/receive-dispatch.yml` accordingly.

## Troubleshooting

- If the workflow fails to find backport PRs: check the app logs and make sure the backport app has permissions to create PRs in this repo.
- If merging backport PRs fails due to conflicts: the test will report failure — consider running tests on a protected branch or modify the test file to avoid conflicts.

## Contact

If you want me to adjust detection rules, add status-check creation, or harden retries/timeouts, tell me which option you'd prefer and I will implement it.
