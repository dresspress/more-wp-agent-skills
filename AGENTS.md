# Agent Instructions — more-wp-agent-skills

This repo contains supplementary Claude Code skills for WordPress development.
Each skill lives under `skills/{skill-name}/` and follows the standard skill format.

## Updating a skill

When asked to update a skill (e.g. "更新 wp-build skill"):

1. **Find upstream sources** — read the skill's `SKILL.md` and look for the `## Upstream docs` section. It lists the authoritative URLs to fetch from.
2. **Fetch all upstream sources** — each skill lists both a GitHub trunk source and an official rendered doc. Fetch both: trunk gives the latest unreleased state; the official docs reflect what is available in stable releases. Note any gaps between the two.
3. **Read all current skill files** — `SKILL.md` plus every file under `references/`.
4. **Diff and align** — compare upstream content against the skill. Update for:
   - New features or fields added since the last sync
   - Breaking changes or renamed APIs
   - Experimental status changes (promoted to stable, or removed)
   - Corrected signatures, parameters, or behavior descriptions
   - New failure modes worth documenting
5. **Fix any errors found during review** — inaccurate signatures, missing required fields in examples, stale version numbers, formatting bugs.
6. **Commit** using Conventional Commits: `docs({skill-name}): ...` for content updates, `fix({skill-name}): ...` for corrections.

## Adding a new skill

1. Create `skills/{skill-name}/SKILL.md` with frontmatter (`name`, `description`, `compatibility`).
2. Add `references/` subdirectory with focused reference files.
3. Include an `## Upstream docs` section in `SKILL.md` listing the authoritative source URLs to fetch from for future updates.
4. Register the skill in the root `README.md` table.

## Conventions

- Each skill's `## Upstream docs` section should list both the GitHub trunk raw source and the official rendered doc site. Trunk is more current; the rendered docs reflect stable availability.
- When a feature exists in trunk but not yet in official docs, note it as Gutenberg-plugin-only and not yet available in stable WordPress core.
- Mark experimental APIs explicitly; note which version introduced or changed them.
- Reference files should be focused and cross-reference each other rather than duplicating content.
