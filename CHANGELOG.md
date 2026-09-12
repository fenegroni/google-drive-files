# Changelog

All notable changes to this skill. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/). See the README for what a version
number means for a skill.

## [1.0.1] - 2026-09-12

### Changed

- The version is declared in the `version` frontmatter field, which the skill
  loader reads, instead of a line in the body. One place rather than two.

## [1.0.0] - 2026-09-10

### Added

- Reading with `download_file_content`, and why `read_file_content` corrupts
  YAML frontmatter.
- Creating files without silent conversion to Google Docs.
- Editing as create-then-archive, since the connector has no content-update
  tool, and why a `history/` folder replaces the trash.
- Search semantics: `title contains` matches tokens, `title =` is case-folded,
  and `parentId = 'root'` is a query alias only.
- Resolving by folder id, asserting exactly one match, and recovering loudly
  when a hardcoded id goes stale.
- A freshness check that tells the reader when the skill's own premise no
  longer holds.
- The single-anchor-folder pattern, with the id owned by the project.

[1.0.1]: https://github.com/fenegroni/google-drive-files/releases/tag/v1.0.1
[1.0.0]: https://github.com/fenegroni/google-drive-files/releases/tag/v1.0.0
