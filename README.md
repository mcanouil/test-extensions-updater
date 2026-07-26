# Test Repository for Quarto Extensions Updater

Fixture repository for [`mcanouil/quarto-extensions-updater`](https://github.com/mcanouil/quarto-extensions-updater), a GitHub Action that keeps Quarto extensions up to date in the same way Dependabot keeps packages up to date.

This repository holds no real content.
It exists so that each input of the action can be exercised against a workspace whose extensions are deliberately out of date.

> [!IMPORTANT]
> Do not update the extensions under `_extensions/`.
> They are pinned to old versions on purpose, so every test run finds something to update.
> Any pull request or branch produced by a test run is disposable.

## Layout

```text
_extensions/            Quarto extensions pinned to outdated versions.
.github/workflows/      One workflow per action input or combination of inputs.
```

## Fixtures

Each extension carries a `source` field, which is what the action reads to work out where updates come from.

| Extension | Pinned version | Source |
| --- | --- | --- |
| `mcanouil/elevator` | 1.1.0 | `mcanouil/quarto-elevator` |
| `mcanouil/highlight-text` | 1.0.0 | `mcanouil/quarto-highlight-text` |
| `mcanouil/iconify` | 3.0.1 | `mcanouil/quarto-iconify` |
| `mcanouil/invoice` | 1.2.4 | `mcanouil/quarto-invoice` |
| `mcanouil/mcanouil` | 0.1.5 | `mcanouil/quarto-mcanouil` |
| `mcanouil/preview-colour` | 1.0.0 | `mcanouil/quarto-preview-colour` |
| `mcanouil/remember` | 1.0.0 | `mcanouil/quarto-remember` |
| `mcanouil/version-badge` | 1.0.0 | `mcanouil/quarto-badge` |

## Test workflows

Every workflow is `workflow_dispatch` only, so nothing runs on a schedule or on push.

| Workflow | What it covers |
| --- | --- |
| `test-basic.yml` | Default behaviour with `create-pr: true`, one pull request per extension. |
| `test-minimal-pr.yml` | The smallest useful input set. |
| `test-dry-run-basic.yml` | `dry-run: true`, reports updates without creating anything. |
| `test-dry-run-with-issue.yml` | Dry run followed by issue creation from the action outputs. |
| `test-grouped-updates.yml` | `group-updates: true`, a single pull request for every extension. |
| `test-custom-pr-options.yml` | Custom branch prefix, commit prefix, title prefix, labels, assignees, and reviewers. |
| `test-branch-configuration.yml` | Custom `branch-prefix` and `commit-message-prefix`. |
| `test-include-exclude.yml` | `include-extensions` and `exclude-extensions` filtering. |
| `test-update-strategy-all.yml` | `update-strategy: all`. |
| `test-update-strategy-minor.yml` | `update-strategy: minor`. |
| `test-update-strategy-patch.yml` | `update-strategy: patch`. |
| `test-auto-merge-all.yml` | `auto-merge: true` with strategy `all`. |
| `test-auto-merge-minor.yml` | `auto-merge: true` with strategy `minor`. |
| `test-auto-merge-patch.yml` | `auto-merge: true` with strategy `patch`. |
| `test-registry-url.yml` | A custom `registry-url`. |
| `test-workspace-path.yml` | An explicit `workspace-path`. |
| `test-outputs.yml` | The `updates-available` and `update-count` outputs. |
| `test-error-handling.yml` | Behaviour when a filter matches no extension. |
| `quarto-extensions-updater.yml` | The plain usage a consumer would copy. |
| `quarto-extensions.yml` | The same usage driven by a GitHub App token rather than `GITHUB_TOKEN`. |

## Running a test

Dispatch a workflow from the Actions tab, or from the command line:

```sh
gh workflow run test-basic.yml --repo mcanouil/test-extensions-updater
```

To exercise a branch of the action rather than the pinned reference, create a branch here, repoint the `uses:` reference in the workflows, and dispatch against it:

```sh
gh workflow run test-basic.yml --repo mcanouil/test-extensions-updater --ref <branch>
```

Auto-merge workflows need at least one required status check or branch protection rule on `main`, otherwise GitHub rejects the auto-merge request.

## Cleaning up after a run

Test runs leave pull requests and branches behind.
Clear them before the next run so results stay readable:

```sh
REPO=mcanouil/test-extensions-updater

for n in $(gh pr list --repo "${REPO}" --state open --limit 100 --json number --jq '.[].number'); do
  gh pr close "${n}" --repo "${REPO}"
done

for b in $(gh api "repos/${REPO}/branches" --paginate --jq '.[].name' | grep -vx main); do
  gh api -X DELETE "repos/${REPO}/git/refs/heads/${b}"
done
```

## Licence

MIT.
