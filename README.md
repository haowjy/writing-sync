# writing-sync

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

Keep local Markdown in sync with the copies you actually use.

The copy that exists today is Google Docs. You select local files; their
folders show up in Drive; Markdown becomes a Doc. Text, HTML, and EPUB can
too. Everything else in the selection uploads as itself. Edits in Docs come
back into the original files when the mapping still holds.

## Install

Add the skill, then ask your agent to set it up. It will install what it
needs and walk you through Google login. Don't paste passwords, client
files, or token JSON into chat.

Mars:

```bash
meridian mars add haowjy/writing-sync
meridian mars sync
```

If you already use [creative-writing-skills](https://github.com/haowjy/creative-writing-skills),
add this package the same way. That repo's Claude / Cowork / Claude.ai plugin
does not include this skill; its harness install is in the
[creative-writing-skills Installation](https://github.com/haowjy/creative-writing-skills#installation)
section.

[skills.sh](https://skills.sh) (Claude Code, Cursor, Codex, OpenCode, and the
other agents listed there):

```bash
npx skills add haowjy/writing-sync
```

## CLI

The Drive implementation is a Python CLI the agent installs and runs. How
that code is laid out:
[skills/writing-sync/scripts/README.md](skills/writing-sync/scripts/README.md).

## Limitations

- **Comments do not sync.** `<!-- comments -->` in the file never appear in
  the Doc. Google comment threads and suggestions never come back into the
  file. If a Doc has comments or suggestions, sync blocks until you deal
  with them in Google.
- **No watcher.** Sync runs when you (or the agent) run it.
- **Both sides edited:** you pick keep-local or accept-remote. No merge.
- **Pull is not a rewrite.** Docs edits come back only when they still map
  onto the original file. Structural edits, new links, images, tables, and
  most Docs-only formatting do not round-trip.
- Converted Docs cover headings, paragraphs, flat lists, emphasis, code, and
  links. Frontmatter stays in the file. Paragraph spacing stays in Google.
- Extra tabs on a Doc block. Local folder layout wins; a remote move is a
  conflict. Leaving a path out of the selection does not delete either copy.
- It only sees Drive files it created or you picked. Pasting a folder URL
  does not grant access.

## Roadmap

[ ] Publishing platforms: keep the live chapter in sync as you update and write. They would be a different index so you can keep parts of it unpublished as well.
[ ] Better google docs support
  [ ] 2 way sync
  [ ] handle comments
  [ ] Keep formatting better translated
  [ ] https://github.com/CriticMarkup/CriticMarkup-toolkit
  [ ] better handling of gdocs comments
  [ ] Tab support?

## License

Apache License 2.0. See [LICENSE](LICENSE).
