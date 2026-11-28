# Mortis Action

Run Mortis Engine on GitHub Actions for pull request reviews, branch comparisons, and security scans. This revision installs [Mortis Engine 0.1.8](https://pypi.org/project/mortis-engine/0.1.8/) on a Linux runner and uses your AI provider connection. Engine analysis sends eligible source and diffs to that provider.

## Review pull requests

1. Copy [the review workflow](.github/workflows/mortis-review.yml) to `.github/workflows/mortis-review.yml` on your default branch.
2. Add the `MORTIS_API_KEY` repository secret and set the action's `provider` to an [Engine provider preset for the bundled workflow](https://pypi.org/project/mortis-engine/0.1.7/#connect-with-environment-variables). Set the `MORTIS_MODEL` repository variable if `mortis.toml` does not set a review model.
3. Pin the action to a reviewed commit SHA. The bundled workflow pins v0.2.3, which installs Mortis Engine 0.1.7.

**Public GitHub.com repositories.** Starting December 2, 2026, automatic reviews using `pull_request_target` require an applicable Actions event policy. In your repository or organization's workflow execution protections, allow that event specifically for `.github/workflows/mortis-review.yml`. Policy Insights shows which runs the default rule would block. To keep the default protection, retain the `issue_comment` trigger and have a maintainer post `/mortis review` in the PR conversation. [GitHub announcement](https://github.blog/changelog/2026-11-02-pull_request_target-blocked-by-default-december-2/) · [Workflow execution settings](https://docs.github.com/en/actions/how-tos/administer/control-workflow-execution)

Same-repository PRs are reviewed automatically. A user with repository write access can post `/mortis review` as a new PR comment to review the current head, including on forks. The `[review].automatic` setting controls automatic runs. All reviews skip drafts. Dependabot PRs skip automatic review. With v0.2.3, a maintainer can request a review of a ready Dependabot PR with `/mortis review` or `/mortis review --full`. Other bot-authored PRs require `include_bot_prs = true`.

With v0.2.3, use `/mortis review --full` to review every eligible change once under the configured review limits. Mortis restores available finding history, bypasses incremental and identical-tree reuse for that run, and saves the result for subsequent reviews.

### PR commands

With an action commit that installs Mortis Engine 0.1.5 or later, users with repository write access can post these commands on an open PR:

- `/mortis ask Why does this change the default?` answers from the current diff with links to the source lines. To ask about a specific thread, copy its review comment permalink from **Files changed**. Post the question in **Conversation** with the permalink on its own final line. Mortis replies in **Conversation** for both same-repository and fork PRs.
- `/mortis describe` updates the title and the Mortis summary block in the description. The rest of the description is preserved.

The bundled workflow handles commands through `issue_comment` using the workflow on the default branch. Ask and describe use the configured review model and the existing `contents: read` and `pull-requests: write` permissions.

With v0.2.3, `/mortis preview` shows selected files, exclusions and reasons, planned batches, and configured limits. It reads the full comparison scope using the target revision's policy and needs no model, provider key, or saved review history. A question with a review comment permalink includes available earlier messages from the selected thread. The newest question takes precedence within the Engine's 8,000-character limit.

- Set the `MORTIS_REVIEW_FORKS` repository variable to `true` to enable automatic fork reviews. Fork activity can then trigger provider calls.
- Set `MORTIS_UPLOAD_REPORTS` to `true` to retain JSON and Markdown as workflow artifacts for seven days. Reports can include source evidence.

The bundled workflow also reviews changes to the PR's target branch. Ordinary issue comments and unrelated edits skip the review job. The action fetches PR commits into a temporary bare Git repository and reads them as data. It compares the exact head to the merge base and loads `mortis.toml` from the **target commit**. Mortis updates one summary comment with the change overview, coverage, and confirmed findings. It also posts findings inline when GitHub can anchor them to added lines.

### Automatic descriptions and linked issues

Set either input under the Action step's `with` block.

| Input | Default | Behavior |
| --- | --- | --- |
| `auto-describe` | `false` | On opening a PR or marking it ready for review, copy the completed review overview into the Mortis block in the PR body. This uses the existing review result and adds no provider calls. |
| `check-linked-issues` | `false` | During an eligible automatic or manual review, check requirements from explicitly linked issues in the same repository. Grant `issues: read` to the review job. |

Link an issue in the author-written PR description with a closing reference such as `Closes #123`. Mortis reads up to three linked issues and assesses up to six requirements in document order. Use a Markdown task list or bullets under an **Acceptance criteria** heading for separate assessments. A prose-only issue receives one issue-level assessment.

Each fresh assessment sends the selected requirement and eligible PR diff to the configured provider. The PR summary includes the answer and source links, or shows **Unverified** for insufficient evidence and **Unavailable** when the check could not run. These assessments are advisory. Mortis reuses matching results for the same review inputs and checks edited issues on the next PR review, including an unchanged-code run.

When an Engine-supported replacement matches the complete line range in the current PR diff, the inline comment includes a GitHub suggestion. For workflow files, Mortis also checks that the resulting YAML parses and has unique mapping keys. Use **Commit suggestion** to apply the replacement to the PR branch.

Set `auto_resolve = true` under `[review]` in the target branch's `mortis.toml` to verify findings carried by incremental review history. Mortis resolves a matching GitHub thread only after the Engine verifies the fix against the current PR head. Resolving threads from GitHub comments alone requires an Engine release with `review --prior-findings`.

### Inline delivery and merge checks

These settings require v0.2.3. Inline findings are submitted as GitHub reviews with up to 25 comments each. Mortis leaves verified threads with replies open for the people discussing them. The sticky summary lists findings without a thread first and links overflow to the Actions run. If those entries exceed the job-summary limit, the run retains a sanitized Markdown index for seven days.

| Action input | Default | Behavior |
| --- | --- | --- |
| `inline-min-severity` | `low` | Set to `medium`, `high`, or `critical` to send lower-severity findings to the summary. |
| `publish-check` | `false` | Publish **Mortis Review (advisory)** on the exact PR head. Grant `checks: write` to the review job. |
| `fail-on-severity` | `none` | Set a minimum blocking severity (`low`, `medium`, `high`, `critical`). This enables the check and also requires `checks: write`. |

For branches that merge PRs directly, set `fail-on-severity`, let the check run, then select its exact name in the target branch's rules. For example, `high` on `main` publishes **Mortis Review (high) / main**. Replace any previous **Mortis Review** requirement and update the rule when the threshold changes.

For branches using a merge queue, keep Mortis advisory. The queue requires a check on the `merge_group` commit, and Mortis publishes its result on the PR head. [GitHub merge queue requirements](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks#status-checks-with-github-actions-and-a-merge-queue)

A complete review counts open findings from the current report and incremental history once per ID. A partial review, an unevaluated blocking policy, or a required check that GitHub cannot publish fails the job. A superseded check is cancelled on its original commit. For eligible failures without a published check, Mortis maintains one notice on the PR and clears it after recovery.

Existing `pull_request` workflows can continue reviewing same-repository PRs with their full checkout.

### Incremental pull request reviews

Set `incremental = true` under `[review]` in the target branch's `mortis.toml`. The first run and the first run after an Engine upgrade review the full PR. Subsequent runs can reuse coverage for unchanged files from a compatible, fully covered review. A changed tree with an eligible deletion or rename triggers a full review to check surviving callers, including in eligible unchanged files.

With Engine 0.1.8, reviews of deleted or renamed files can find callers through package-relative Python imports and shell paths with escaped spaces. Incremental reviews also recheck previously reviewed Python importers when a module-level function is removed, switches between sync and async, or changes how existing calls bind arguments. Adding a compatible optional parameter still allows reuse. Fully reverting a previously reviewed file change can trigger a fresh review of the remaining diff.

Set `skip_unchanged = true` to reuse compatible coverage when the head tree is identical, including complete caller coverage. The job summary and PR comment list reviewed and reused files and the reason for reuse. The PR comment keeps its previous change overview when identical content was already reviewed, along with earlier open findings from saved history.

Generate a key with `openssl rand -base64 32` and store it as the `MORTIS_STATE_KEY` repository secret. The bundled workflow passes the key to the action and grants `actions: read`. For a custom workflow, add those settings and serialize review jobs per PR using a concurrency group. Use an action revision that includes incremental review support.

Each PR's finalized review history is encrypted and stored as a workflow artifact for the repository's configured retention period. The first run starts with a full review. If the latest retained history cannot be read, Mortis still reviews the diff, preserves the saved history for retry, and marks a blocking policy `unevaluated`. Check the state key and the workflow's `actions: read` permission before retrying.

## Compare branches

This workflow compares two refs available in the full checkout. Use a trusted base ref such as `origin/main`.

```yaml
name: Mortis branch review
on:
  workflow_dispatch:
    inputs:
      head:
        description: Head ref to review, such as origin/my-branch
        required: true
        type: string
permissions:
  contents: read
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1
        with:
          fetch-depth: 0
          persist-credentials: false
      - uses: code-mortis/mortis-action@8958be81830c059c680bc293cdbf6724090b5f03 # v0.2.3
        with:
          base: origin/main
          head: ${{ inputs.head }}
          provider: openai
          model: ${{ vars.MORTIS_MODEL }}
        env:
          MORTIS_API_KEY: ${{ secrets.MORTIS_API_KEY }}
```

## Scan a revision

The v0.2.3 example below installs a checksum-verified Trivy 0.76.2 on Linux X64 and ARM64. Pass `ref` to select another local revision. The default is the commit that triggered the workflow. Scans in v0.2.0 fail because it installs Trivy 0.76.0, while Engine 0.1.5 requires 0.76.2.

```yaml
name: Mortis security scan
on:
  workflow_dispatch:
  schedule:
    - cron: '0 6 * * 1'
permissions:
  contents: read
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1
        with:
          fetch-depth: 0
          persist-credentials: false
      - uses: code-mortis/mortis-action@8958be81830c059c680bc293cdbf6724090b5f03 # v0.2.3
        with:
          command: scan
          provider: openai
          model: ${{ vars.MORTIS_MODEL }}
          # retain-scan-findings: "true"
        env:
          MORTIS_API_KEY: ${{ secrets.MORTIS_API_KEY }}
```

The v0.2.3 scan summary shows severity counts and coverage. Enable `retain-scan-findings` to keep a seven-day Markdown artifact listing each finding's severity, title, and location at the scanned revision. People with access to the run's artifacts can download the index. Raw JSON and full Markdown reports are separate runner-local outputs.

## Repository configuration and reports

Commit `mortis.toml` at your repository root. Reviews read it from the base commit. Scans read it from the scanned commit. For example:

```toml
[review]
automatic = true
include_bot_prs = false
language = "English"
exclusions = "vendor/**, generated/**"
effort = "medium"
max_files = 40

[scan]
exclusions = "vendor/**"
timeout_seconds = 600
```

The engine supplies defaults when the file is absent and rejects invalid configuration before contacting the provider. A model in the effective Engine settings takes precedence over the action’s `model` input. The provider preset and API key come from the workflow. Use the optional `protocol` and `base-url` inputs to override the preset’s connection settings.

The Actions summary shows status and coverage. The action exposes `status` (`completed`, `partial`, `failed`, or `skipped`), `run-id`, `report-json`, and `report-markdown`. Report outputs are runner-local paths for subsequent steps in the same job. Partial results keep their reports and return exit code 2. A later step can use `if: ${{ !cancelled() && steps.mortis.outputs.report-json != '' }}` to consume available reports, including after a failure. Reports may include source evidence. Successful analysis with findings exits 0.

Published PR reviews in v0.2.3 expose five delivery counts. `inline-posted` counts new inline comments, `inline-existing` counts findings with an existing comment, `summary-only` counts findings without an eligible location, and `inline-failed` counts delivery failures. The `inline-routed` output counts findings routed to the summary by severity. The separate `policy-result` output reports the severity decision (`advisory`, `passed`, `blocked`, or `unevaluated`). A completed analysis can produce `status=completed` and `policy-result=blocked` with a failing step. A partial run preserves exit code 2. Failure to publish the required summary or upload a requested index fails the job.

## Development

Use Python 3.15 and follow the [contribution guide](CONTRIBUTING.md) to install test dependencies and run the offline suite.

### Releasing the action

Use the project-local [release preparation skill](.agents/skills/mortis-action-release/SKILL.md),
`$mortis-action-release` in Codex, to check Engine compatibility, update the
Action guides, and write the release description in [CHANGELOG.md](CHANGELOG.md)
under `## X.Y.Z - YYYY-MM-DD`.

Include the changelog entry in the commit on `main` that you tag. Push the new `vX.Y.Z` tag to run CI and create a GitHub release draft from that version’s entry. Open the draft, select **Publish this Action to the GitHub Marketplace**, and publish the release.

## Security

Report vulnerabilities privately using the [security policy](SECURITY.md).

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
