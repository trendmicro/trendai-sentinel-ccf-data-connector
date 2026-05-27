<!--
Thanks for the PR! A few quick checks help us review faster.
See CONTRIBUTING.md for full guidance.
-->

## Summary

<!-- One or two sentences on *why* this change is needed. -->

## Type of change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature / connector enhancement
- [ ] Documentation only
- [ ] CI / workflow / repo hygiene
- [ ] Template refactor (no behavior change)
- [ ] Breaking change (requires re-deploy by existing users)

## Connector(s) affected

- [ ] Workbench
- [ ] OAT
- [ ] Both / shared
- [ ] Neither (docs or CI only)

## Validation

- [ ] `jq empty` passes on every changed JSON file
- [ ] `az deployment group validate` succeeds for changed `mainTemplate.json`
- [ ] Deployed to a test Sentinel workspace and confirmed data ingestion
- [ ] Updated README / ARCHITECTURE / connector docs if behavior changed
- [ ] No secrets, tokens, or workspace identifiers in the diff

## Test plan

<!--
Walk a reviewer through how you verified this PR. Include:
- Test workspace name / region (no real customer data)
- Az CLI command used to deploy
- Sample KQL run and the row count returned
-->

## Related issues

<!-- Use "Closes #123" to auto-close issues when merged. -->

Closes #
