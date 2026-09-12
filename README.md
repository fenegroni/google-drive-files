# google-drive-files

A Claude skill for reading and writing files in Google Drive through the Drive
connector, where several tools do not do what their names suggest.

Everything in it was verified by testing the connector. Each behaviour it covers
fails silently — you get plausible wrong results, not errors:

- `read_file_content` escapes markdown and corrupts YAML frontmatter.
- There is no content-update tool, so editing a file means create-then-archive.
- `title contains` matches word tokens; `title =` is case-folded.
- Uploads become Google Docs unless conversion is disabled.

The skill is generic. It holds no folder ids and needs no editing — a project
that anchors its configuration on a Drive folder keeps that id itself.

## Install

Download the latest release, or this repo as a ZIP, and upload it as a skill.
`SKILL.md` sits at the root, so the downloaded folder is the skill folder.

## Versioning

[Semantic Versioning](https://semver.org/). The version is declared in the `version`
field of `SKILL.md`'s frontmatter, which the skill loader reads, so an
installed copy can tell you which one it is. A skill's interface is its
guidance, so the numbers mean:

- **MAJOR** — guidance changes such that following the old version would now be
  wrong. The likeliest cause: the connector gains a content-update tool, and
  create-then-archive stops being the right way to edit.
- **MINOR** — a newly verified behaviour or procedure is added. Nothing existing
  changes.
- **PATCH** — wording, clarity, examples. Same behaviour.

Changes are recorded in [CHANGELOG.md](CHANGELOG.md).

## Licence

[MIT](LICENSE).
