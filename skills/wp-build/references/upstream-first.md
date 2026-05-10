# Upstream-first workflow

This skill intentionally avoids duplicating the `@wordpress/build` README.

For each task, build the answer from current upstream sources first, then use local caveats only as a checklist for uncertainty.

## Source order

Use this order when facts conflict:

1. Current generated files in the user's project after running `wp-build`.
2. Current Gutenberg trunk source/templates for `packages/wp-build`, `packages/boot`, and `packages/route`.
3. Current Gutenberg trunk README/changelog/package metadata.
4. Official rendered docs on developer.wordpress.org.
5. This skill's supplemental caveats.

The generated files win because they are what the plugin will actually execute.

## Minimum upstream set

Always fetch:

- `packages/wp-build/README.md`
- `packages/wp-build/CHANGELOG.md`
- `packages/wp-build/package.json`
- official package docs on developer.wordpress.org

Fetch source/templates when the task touches behavior that docs often summarize:

- generated PHP entry points and callback names
- page mode differences
- route registration and lazy loading
- widget discovery and registration
- boot/init runtime behavior
- peer dependency or host-package assumptions

## Reconciliation rule

If upstream docs and source disagree:

- state the discrepancy plainly
- treat docs as intent and source/generated output as executable behavior
- avoid presenting the behavior as stable unless upstream marks it stable
- add a project-level comment or note only when the project needs future maintainers to understand the choice

## Offline rule

If current upstream cannot be fetched, do not pretend this skill is current. Say upstream was not verified and keep recommendations conservative.
