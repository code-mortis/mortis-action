# Changelog

## Unreleased

## 0.2.4 - 2026-11-27

Mortis Action now installs Mortis Engine 0.1.8. When upgrading from v0.2.3, keep your existing workflow permissions and `MORTIS_STATE_KEY`. The first incremental PR review checks the full diff to establish a fresh baseline and carries forward open findings from saved history.

- Subsequent reviews recheck previously reviewed Python importers when a module-level function is removed, switches between sync and async, or changes how existing calls bind arguments. Adding a compatible optional parameter still allows reuse.
- Reviews of deleted or renamed files find more callers in unchanged files, including package-relative and multiline Python imports, calls through aliases, and shell paths with escaped spaces.

## 0.2.3 - 2026-11-26

Upgrading from v0.2.2 keeps Mortis Engine 0.1.7, Trivy 0.76.2, and the Linux runner requirement. Keep your existing provider, model, and `MORTIS_STATE_KEY`. To pick up the new event filters, update your copy of the bundled review workflow as well as the Action pin.

- Use `/mortis review --full` for a full review, or `/mortis preview` to see scope and exclusions before making a provider call. Ask about a review thread by posting a question and its comment permalink in the PR conversation. Maintainers can also request reviews on Dependabot PRs there.
- Review PR retargets and filter unrelated comments before allocating a runner. Public GitHub.com repositories using automatic reviews must explicitly allow their workflow's `pull_request_target` event beginning December 2, 2026. Maintainers can also request reviews through a PR conversation.
- Enable `auto-describe` to fill the Mortis description block from the opening review. Enable `check-linked-issues` and grant `issues: read` to check linked acceptance criteria with the configured provider.
- Receive inline findings in groups of up to 25. Set `inline-min-severity` to route lower severities to the summary. Mortis protects discussions with replies and makes overflow findings available in the job summary or a seven-day Markdown index. Delivery counts are available as Action outputs.
- Enable `publish-check` for an advisory check on the reviewed head, or set `fail-on-severity` to enforce a threshold across current and carried-forward open findings. Blocking check names include the threshold and target branch, such as **Mortis Review (high) / main**. Both options require `checks: write`. PR workflows can read `policy-result` separately from the analysis status.
- When the latest saved review history cannot be read, Mortis reviews the diff and preserves the history for retry. An enforced severity policy reports `unevaluated`. The review message tells you when to set `MORTIS_STATE_KEY` and when to check the existing key and `actions: read` permission.
- Scan summaries show severity counts. Set `retain-scan-findings: true` to retain a sanitized seven-day index of titles and locations.

## 0.2.2 - 2026-11-26

Mortis Action now installs Mortis Engine 0.1.7. The first incremental PR review after upgrading reviews the full diff to establish a new baseline. Keep your existing `MORTIS_STATE_KEY` and workflow permissions.

- Incremental reviews recheck previously reviewed Python importers when a module-level function’s required parameters change or it gains or loses `*args` or `**kwargs`. Fully reverting a previously reviewed file change triggers a fresh review of the remaining diff when prior coverage would otherwise be reused.
- When a Python module under `src/` is deleted or renamed, reviews can find absolute imports of that module in unchanged files.
- GitHub release drafts now use the tagged version’s changelog entry as their description.

## 0.2.1 - 2026-11-26

- Upgrade to Mortis Engine 0.1.6 and retain caller coverage in encrypted review history. Deleting or renaming an eligible file triggers a full comparison so the review includes surviving callers.
- Restore security scans on Linux x64 and ARM64 by installing the Engine’s required Trivy version, 0.76.2.

## 0.2.0 - 2026-11-24

- Add opt-in incremental pull request reviews. Set `incremental = true` and configure the `MORTIS_STATE_KEY` secret to reuse coverage and carry open findings forward in encrypted per-PR history.
- Security scans in this version fail because it installs Trivy 0.76.0 and the Engine requires 0.76.2.

## 0.1.0 - 2026-11-24

- Run Mortis Engine on GitHub Actions for pull request reviews and branch comparisons.
- Publish PR summaries and inline findings, answer `/mortis ask` questions, and update PR descriptions with `/mortis describe`.
- Introduce the `scan` command. Its Trivy version mismatch is fixed in 0.2.1.
