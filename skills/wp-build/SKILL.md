---
name: wp-build
description: "Use when developing WordPress plugins with @wordpress/build. This skill is upstream-first: fetch current official/trunk docs before acting, then use this skill only for workflow, caveats, and verification gaps that upstream docs do not make obvious."
compatibility: "Treat @wordpress/build pages/routes/widgets as experimental unless current upstream docs and source say otherwise. Do not rely on version numbers, generated file names, PHP helper signatures, or package peer dependencies from this skill without verifying current upstream first."
---

# WP Build (@wordpress/build)

This skill is not a standalone copy of the `@wordpress/build` documentation.

Use the upstream docs as the source of truth. Use this skill only to decide what to verify, where upstream can lag, and which generated artifacts must be inspected before you trust an implementation.

## Upstream docs

Before creating, changing, reviewing, or debugging an `@wordpress/build` project, fetch the current upstream sources listed below:

1. GitHub trunk README:
   `https://raw.githubusercontent.com/WordPress/gutenberg/trunk/packages/wp-build/README.md`
2. GitHub trunk changelog:
   `https://raw.githubusercontent.com/WordPress/gutenberg/trunk/packages/wp-build/CHANGELOG.md`
3. GitHub trunk package metadata:
   `https://raw.githubusercontent.com/WordPress/gutenberg/trunk/packages/wp-build/package.json`
4. Official rendered docs:
   `https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/`

   Prefer the Markdown rendering when fetching for agent analysis:
   `https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/?output_format=md`

When the task involves generated PHP, page modes, route registration, boot behavior, or widgets, also inspect the relevant current source/templates in Gutenberg trunk rather than inferring from this skill.

Start with:

- `packages/wp-build/lib/build.mjs`
- `packages/wp-build/lib/route-utils.mjs`
- `packages/wp-build/lib/widget-utils.mjs`
- `packages/wp-build/templates/*.php.template`
- `packages/boot/src/` when page or route runtime behavior matters
- `packages/route/src/` when router behavior matters

If network access is unavailable, say that upstream could not be verified and treat all experimental behavior as uncertain.

## How to use this skill

Use this skill for:

- setting up or reviewing a plugin that uses `@wordpress/build`
- deciding whether `@wordpress/build` is appropriate instead of `@wordpress/scripts`
- working with experimental `wpPlugin.pages`, `routes/`, page modes, or `widgets/`
- debugging generated PHP, script-module registration, route loading, or boot/runtime behavior
- checking whether an implementation is relying on stale examples

Do not use this skill for:

- Gutenberg block development with `@wordpress/scripts` (use a block skill instead)
- stable WordPress plugin architecture unrelated to `@wordpress/build`
- copying official docs into a project without verifying the current package source

## Operating rules

- Prefer current upstream documentation for all ordinary setup details: directory names, package fields, examples, and command usage.
- Do not restate upstream docs in generated project documentation unless the project needs a local decision record.
- For experimental APIs, verify both docs and implementation. If README, official docs, and source disagree, explain the gap and follow the source for generated behavior.
- For generated PHP helpers, inspect the actual files under `build/` after running the build. Do not assume helper names or signatures from memory.
- For host/runtime dependencies, check the current `package.json`, changelog, and generated asset files. Do not rely on pinned compatibility text from this skill.
- Keep implementations minimal. Avoid adding `routes/`, `widgets/`, or page-mode machinery unless the project actually needs those surfaces.
- When debugging, reproduce with `wp-build` output and browser/PHP errors before changing architecture.

## Supplemental references

Read these only after checking upstream:

- `references/upstream-first.md` - source order and reconciliation workflow
- `references/implementation-caveats.md` - current known gaps and easy AI mistakes
- `references/verification.md` - build/runtime checks to run before declaring success

## Maintenance rule for this skill

Do not add large examples or full field references here if upstream already documents them clearly.

Only add local material when it meets at least one condition:

- upstream docs are ambiguous, outdated, or contradicted by source
- the point is an agent workflow rule rather than product documentation
- the point is a recurring failure mode not obvious from upstream docs
- the point tells the agent what generated files to inspect before trusting an answer

When adding a caveat, include why it exists and what upstream/source file should be checked next.
