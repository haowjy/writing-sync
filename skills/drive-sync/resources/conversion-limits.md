# Conversion and return limits

Run these commands before enabling converted storage for a new source set:

```sh
drive-sync conversion capabilities
drive-sync --index /external/drive.yaml conversion check
```

Both are offline. `capabilities` describes implemented routes, not a guarantee
about arbitrary files. `check` reads the selected files and reports forward
rendering findings. Its `supported: true` means that source passes the local
rendering gate; it does **not** approve every possible Google edit or verify the
account's import support. For converted files, `return_policy` remains
`candidate_verification_required`. Every actual return must pass mapping,
formatting and preservation checks before replacing the source.

## Implemented routes

All four converted formats use a local readable model. Creation uses an HTML
bridge, or separately authorized empty-Doc initialization followed by population.
Updates use revision-guarded Docs requests. Read-back uses the Docs structural
model with annotation observations; it does not replace source files with a
freshly generated Google `files.export` download. Text and supported style
coverage are compared semantically; raw remote snapshots retain change evidence.

### Google-only paragraph presentation (0.1.1)

Paragraph space before/after, line spacing and spacing mode stay in Google Docs;
they are not encoded as Markdown blank lines or inserted into plain text. A
spacing-only edit is a sync no-op. If local prose also changed, the planner can
push it without declaring a false two-sided content conflict. Real text/style
conflicts, layout changes and annotation guards still apply.

On a return, the candidate must reproduce content and supported source styles,
while spacing remains untouched remotely. On a push, spacing is captured from
the fresh guarded observation, mapped onto resulting paragraphs, explicitly
restored after content/style updates and checked during read-back. It is not
merely excluded from verification. The staged overlay is retained for uncertain
writes and reviewed batch repair; old journals retain their stricter verification.

Uniform spacing can carry over paragraph insertions/deletions. Mixed spacing
requires unambiguous paragraph correspondence; incompatible replacements and
repeated paragraphs with conflicting spacing block. Mapping is bounded to 2048
paragraphs per side and 4 million combined text characters. Inherited named-style
edits, fonts, alignment and other unrepresented formatting remain protected—not
silently treated as spacing. Supported direct paragraph fields are `spaceAbove`,
`spaceBelow`, `lineSpacing` and `spacingMode`.

Google supports [Markdown import](https://developers.google.com/workspace/drive/api/guides/manage-uploads)
and [Markdown export](https://developers.google.com/workspace/drive/api/guides/ref-export-formats).
The current sync still uses the routes above, not whole-file Markdown
export/re-import. An exported reading copy cannot reconstruct craft comments
that were deliberately never uploaded. Re-import alone also does not establish
preservation of Google-only presentation during an update. Source-preserving
return mapping and revision-controlled updates remain necessary in this tool.

| Format | Original-format return | Protected material and blocking cases |
| --- | --- | --- |
| Markdown | Uniquely mapped literal-text edits in the original bytes | Frontmatter, invisible comments, markup and local link syntax remain local. Ambiguous correspondence, structural changes and edits crossing protected boundaries block. |
| Plain text | UTF-8 text with the source newline convention | Literal Unicode remains content. Formatting that cannot be represented in plain text blocks rather than disappearing silently. |
| HTML | Uniquely mapped text edits within the original HTML | Preserve tags, attributes, comments and unmapped whitespace. Scripts, CSS, assets and unsupported structures are not silently flattened. A regenerated DOM is not a preservation strategy. |
| EPUB | Mapped XHTML text edits inside the original package | Preserve spine order and unrelated members; deliberately update package modification metadata. Unsupported assets/layout/reading behavior, invalid packages and ambiguous mapping block. |

Supported source formatting is determined by the renderer's findings, not by an
agent's guess based on file extension. A readable source may still produce an
unreturnable edit: splitting/reordering paragraphs, changing styles or introducing
new links is not equivalent to replacing an unambiguous word. The service checks
the candidate by rendering it forward and comparing the resulting text/styles.
No general lossless rich-document merge is promised.

## Review a return candidate before applying

```sh
drive-sync --index /external/drive.yaml source preview-return --source SOURCE_UUID
```

This prepares a conflict-free pull through the same validation service as apply,
but writes only private review artifacts: `before.source`, `candidate.source`,
`changes.json` and a `source.diff` when a bounded UTF-8 diff is available. EPUB
reports list every member's added/removed/modified/unchanged status and hashes;
binary or large files retain complete artifacts with an explicit diff-omission
note. Artifact directories are `0700`, files `0600`. The command prints paths and
hashes, not source text, and never changes the source or Google object.

For a two-sided content conflict, add `--accept-remote` to preview that explicit
resolution's candidate. This is still inspection, not consent to replace local
edits. The same conflict/preservation rules apply; unsupported resolutions block.

After reviewing a normal pull, use the returned digest with
`sync --source SOURCE_UUID --apply --plan-digest DIGEST`. For an accept-remote
review, use `conflict resolve --source SOURCE_UUID --choice accept-remote
--apply --plan-digest DIGEST`. Repeat the same `--index` and state/profile options.
Changes after review invalidate the material plan. Apply regenerates and
revalidates the candidate; it does not trust a manually edited review artifact.

EPUB modification metadata is stamped when the candidate is prepared. A later
apply can therefore change that timestamp and dependent ZIP checksums; the
review explicitly lists this exception in `candidate_may_change_at_apply`.
Content edits and unrelated member preservation remain guarded. Every real pull
also saves its **actual staged** change report and before/candidate blob references
under `journal/OPERATION.review/changes.json` before local replacement, including
pulls performed without the separate preview command.

## Links, assets and annotations

- Managed local links need verified destination identities and persisted reverse
  provenance. For first-time cross-links, use reviewed `sync --initialize` before
  population. Unselected targets, fragments and unrecognized new links may block;
  never substitute a `file://` URL or silently expand write scope.
- Converters do not fetch external resources or execute scripts. Do not describe
  retained EPUB ZIP members as proof that their visual effect survived in Docs.
  Rich assets/CSS/layout need separate fidelity evidence and remain capability
  blockers where not represented.
- Comments, suggestions, extra tabs and unrepresentable remote structures/styles
  can block writes or returns. Removing them is a human decision, not an automatic
  repair. An unreadable annotation response is not evidence of no annotations.

## Resource budgets

Application rendering, return matching and candidate reports run in a fresh
fixed-operation Python worker. Each operation has these defaults:

| Budget | Limit |
| --- | --- |
| Worker CPU time | 10 seconds |
| Parent-enforced wall time | 20 seconds |
| Worker address space | 512 MiB |
| Worker result file | 64 MiB |
| Serialized request | 256 MiB |

The worker sets OS resource limits before parsing the request, disables core
dumps, receives no OAuth/profile environment variables or inherited application
file descriptors, and uses a private temporary directory. The parent kills and
waits for an over-time worker and removes temporary files. Resource failures
produce `conversion_resource_limit`; no source replacement is authorized from
an incomplete result. OS enforcement is tested on Linux; unavailable limits
fail closed rather than falling back to unbounded in-process conversion.

These are per-worker resource bounds, **not an OS security sandbox** or a claim
that the parent process uses at most 512 MiB. Serialized input/output are bounded
separately. No arbitrary commands, plugins or network operations are accepted by
the worker. Index fields cannot raise the policy limits. Large or pathological
inputs may fail even if their format is otherwise supported; retain both copies
and inspect the finding rather than stripping protected content to force success.

The implementation uses Python's documented [resource limits] and
[subprocess timeout behavior]. Low-level parser helpers are internal; call the
application/conversion entry points to retain the worker boundary.

## Safe response to a blocked conversion

Show the source path, finding code/reason and suggested action. Preserve the
source and remote copy. Use [recovery](recovery.md) for a pending operation or
reviewed conflict resolution. Do not strip comments, regenerate an EPUB or
flatten HTML merely to make a test pass.

If the user wants original-file storage, make an explicit
[index override](schema.md); never change representation as a silent fallback.
For an already-bound file, use the reviewed storage-replacement workflow, which
retains the old object. Original-file mode transfers bytes under the same
conflict guards, but does not provide an editable Google Doc reading copy.

See [integration testing](integration-testing.md) for the four-format and EPUB
conformance fixtures. Passing those fixtures establishes their tested behavior,
not unrestricted import/export fidelity for arbitrary books or web pages.

[resource limits]: https://docs.python.org/3/library/resource.html
[subprocess timeout behavior]: https://docs.python.org/3/library/subprocess.html#subprocess.run
