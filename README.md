# more-wp-agent-skills

Supplementary agent skills for WordPress development, extending the official [WordPress/agent-skills](https://github.com/WordPress/agent-skills) repository.

## Skills

| Skill | Description |
|-------|-------------|
| [wp-build](./skills/wp-build/SKILL.md) | Upstream-first guidance for `@wordpress/build`, focused on verification workflow, experimental caveats, and generated-output checks |

## Usage

These skills follow the same format as the official WordPress agent-skills and can be used alongside them.

## Why "more"?

The official WordPress agent-skills repository focuses on stable, widely-adopted workflows. This repository covers:

- Emerging tools and experimental workflows (e.g., `@wordpress/build`, especially `routes/` / `wpPlugin.pages`)
- Specialized workflows not yet covered upstream
- Upstream-first supplements for workflows where official docs and trunk source can drift

## Contributing

Skills should follow the [WordPress agent-skills format](https://github.com/WordPress/agent-skills#skill-format). Consider contributing mature skills upstream to the official repository.

## License

GPLv2 or later, consistent with WordPress.
