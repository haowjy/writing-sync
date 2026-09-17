# Configuration commands

Use the [installed CLI](installation.md) from any working directory:

```sh
drive-sync --index /external/drive.yaml index create
drive-sync --index /external/drive.yaml index show
drive-sync --index /external/drive.yaml index preview
```

`index create` refuses to replace an existing index. `index validate` checks schema;
`index preview` additionally resolves sources. Commands emit JSON.

`index edit --expected-digest DIGEST --edits JSON` prepares and previews compound
changes. Add `--save` to commit configuration, not sync. Obtain DIGEST from `show`.
For example, this compound edit adds and selects a root:

```json
[
  {"operation":"set","path":["roots","notes"],"value":{"path":"/external/notes","destination_name":"Notes","destination_parent":"GRANTED_FOLDER_ID"}},
  {"operation":"append","path":["selection","include"],"value":{"root":"notes","path":".","glob":"**/*"}}
]
```

To place a root in an existing Drive folder, set `destination_parent` from
`auth pick-folder` as in [destination parent](destination-parent.md).

Edit operations are `set`/`remove` on mapping keys and `append` on lists. Replace a
whole list with `set` to remove or reorder its entries. Validation applies to the
compound result. Conflicting concurrent saves fail instead of overwriting changes.
Each edit must be an object with `operation` and a nonempty `path` list of string
keys. `set` and `append` require `value`; `remove` omits it. Unknown fields and
malformed shapes fail without saving. Services can use `parse_edits` to validate
the same representation without importing the CLI.

Command errors use JSON on stderr with a stable code, reason, safe repair action,
operation and known source/destination. `context` carries the requested index,
profile or source/operation IDs where applicable. Unknown destinations are empty.
Raw OS exceptions and JSON payloads are not echoed. Usage errors such as missing
`--plan-digest` use argparse's stderr help and fail before authentication; a
successful exit is not a substitute for checking each sync result's status.

Relative roots resolve from the index directory, never the working directory.
Exclusions win over explicit selections. Overrides change storage only; they do
not select files. Conflicting matching storage overrides fail. Selected unsupported
files require exclusion or explicit original storage. Symlinks and overlapping
roots are rejected.

## Conversion checks

`conversion check` resolves the index and reports format capability findings
without credentials or Google writes. It delegates to typed `check_conversion`
and `check_source_conversion` services. Invalid/unreadable files produce per-file
findings without hiding the remaining reports; invalid selection still aborts
because its scope is unknown. Original-file selections are checked for readable
bytes, not reported as conversion-fidelity successes.

Markdown currently renders headings,
paragraphs, flat lists, emphasis, code and external links through an AST. It
preserves frontmatter separately and omits hidden comments from reading text.
Local links require verified selected Doc targets; unresolved targets, images,
tables and unsupported structures block apply rather than silently disappearing. HTML has a semantic-subset reader; EPUB has a bounded package/spine reader using
that HTML model. Constrained source-preserving return patches exist for all four formats; rich
structural edits and asset/layout fidelity are not promised. See
[conversion limits](conversion-limits.md) before choosing converted storage.

HTML conversion preserves nonbreaking/Unicode spaces and distinguishes inline
code from preformatted blocks. Numbering overrides, direction/language attributes
and other unrepresented presentation semantics currently produce conversion
findings. Do not remove meaningful attributes just to pass validation; use original
storage when their rendered fidelity is required.

Uniquely mapped Markdown heading-text edits can return without changing heading
syntax: ATX markers (including closing markers), setext underlines, inline emphasis,
comments and line endings remain in the source. Repeated/ambiguous heading text
still blocks. This supports text edits inside headings, not arbitrary heading
level changes or structural document rewrites.

Unique literal text in flat Markdown list items can also return while preserving
bullet characters, numbered-list delimiters, whitespace, emphasis and comments.
This does not authorize inserting/removing/reordering items or flattening nested
lists. Repeated item text remains a correspondence conflict.

## Sync

Preview/apply, initialize, and `--source` scoping: [sync](sync.md).
First push of a new selection: [first sync](first-sync.md).
Pending writes and conflicts: [recovery](recovery.md).
