---
name: google-drive-files
description: How to read and write files in Google Drive through the Drive connector, where several tools do not do what their names suggest. Use whenever a task involves reading, creating, editing, listing or organising files in Google Drive — especially config or data files an agent owns, or markdown with YAML frontmatter. Covers a connector with no content-update tool, a reader that silently corrupts frontmatter, and search operators that match more than they appear to.
---

# Working with files in Google Drive

Version 1.0.0

Everything here was verified by testing the connector, not inferred from tool
descriptions. Each behaviour fails **silently** when you get it wrong — you get
plausible-looking wrong results, not an error.

## First: is this still true?

Check the available Drive tools for one that **updates file content**. As of
2026-09-09 there is none — only `create_file`, `copy_file`, `trash_file`,
`update_file` (title and parent only), `search_files`, `list_recent_files`,
`read_file_content`, `download_file_content`, `get_file_metadata`,
`get_file_permissions`, `share_file`.

If a content-update tool now exists, **most of the write guidance below is
obsolete ceremony** — edit in place instead, and re-verify the rest before
following it.

## Reading a file

Use **`download_file_content`** and base64-decode the result.

**Never use `read_file_content` on a file you intend to parse.** It returns a
"natural language representation" with markdown escaped, which destroys YAML
frontmatter:

| In the file | What `read_file_content` returns |
|---|---|
| `---` | `\---` |
| `duration_minutes: 90` | `duration\_minutes: 90` |
| `days: [mon, tue]` | `days: \[mon, tue\]` |
| `## Why` | `\#\# Why` |
| two-space indent | one space |

That is no longer a YAML document, and it may partially parse rather than error.
`download_file_content` returns the raw bytes and decodes byte-identical to the
original.

`read_file_content` is fine when you only want prose for a human.

## Creating a file

Pass `contentMimeType: text/markdown` (or whatever it really is) **and**
`disableConversionToGoogleType: true`.

Without the second flag, Drive converts the upload into a Google Doc, and the
next reader gets rich text instead of the markdown you wrote.

## Editing a file: create, then archive

There is no content-update tool, so an edit is a replacement:

1. **Create** the new file, with its final name, in the target folder.
2. **Archive** the old one: move it into a `history/` subfolder and rename it
   `<name>--<YYYY-MM-DDTHH-MM>.md`. A single `update_file` call changes both
   title and parent.

**In that order.** If the process dies in between, the folder briefly holds two
files with the same name, and a reader taking the newest by `createdTime` gets
the correct new content. Archiving first would leave a window where the file
appears deleted, which is far worse.

**Do not trash superseded versions.** Trash empties after 30 days. Drive's own
revision history does not apply here either, because each edit is a *new file*
rather than a new revision of one — so `history/` is the only version history
that exists. Archiving costs nothing and is the difference between having a trail
and having none.

Ignore `history/` when listing a folder's current contents.

## Finding things

Prefer resolving by **id**. Ids are unique and survive renames and moves.

Anchor on **folder** ids, never file ids. Editing a file mints a new file id, so
anything holding one across runs goes stale. A folder id survives any amount of
churn among its children.

When you must match by name, the operators do not behave as they look:

- **`title contains` matches word tokens, not substrings.** Searching `'illar'`
  finds nothing named `Pillars`. Searching `'Pillars'` finds both `Pillars` and
  `PillarsForMe`.
- **`title =` is genuine equality, but case-folded.** `'Pillars'` matches only
  `Pillars`; `'pillars'` also matches `Pillars`.

So: use `=`, scope it to a known `parentId`, and **assert exactly one result**.

```
0 results   → stop and tell the user
>1 results  → stop and ask which
```

Taking `files[0]` and continuing is how an agent quietly writes to the wrong
file. The assertion is what catches a case-variant duplicate or an interrupted
write.

One more: `parentId = 'root'` works as a query alias, but results report the real
root id, so never compare a returned `parentId` against the string `'root'`.

## When something is missing

Stop and tell the user. Do not create the folder, do not fall back to a local
directory, and do not proceed with an empty set.

A skill that cannot find its configuration and continues anyway will report that
everything is fine — the most misleading output it can produce.

If a hardcoded folder id no longer resolves, it was deleted and recreated, or the
setup was copied to another account. **Recover loudly**: search root for the
folder by title, assert exactly one, confirm it contains what you expect, then
tell the user the new id and that every skill holding it needs updating. A skill
that repairs itself quietly leaves every other skill broken.

## Anchoring a project

A setup that keeps shared configuration in Drive does well to put all of it under
one folder, and hardcode that folder's id wherever it is needed — a skill, an
agent's instructions, a project's `CLAUDE.md`. Resolve everything beneath it by
listing, matching names with `title =` scoped to that parent, asserting exactly
one result.

The id belongs to the project, not to this skill. Nothing here needs editing to
use a different folder.

There is no need for a registry file mapping names to ids. The folder hierarchy
already is that map, maintained by Drive, and a second copy only drifts from it.
