# Recovery and conflicts

Use `state recover` after a pending or uncertain result. It reads current remote
state and verifies staged operations; it never repeats a remote write. Exactly
one matching object and the intended verified content are required to advance a
baseline. Zero/multiple ownership matches, mixed content, unreadable state or
missing preparation remain pending.

Repair apply commands support cooperative Ctrl-C cancellation. Before a
pending-create repair begins its write attempt, cancellation sends no write and
does not consume the one-attempt repair record. Once a request starts, its
outcome is still read back and journaled; cancellation is not a rollback. CLI
exit status `130` may therefore accompany a verified file result. Inspect both
the top-level `cancelled` flag and per-file statuses before deciding what remains.
Typed create-repair and batch-resume services also accept a cancellation token
and per-file progress callback; previews do not emit apply-progress events.

Use `conflict inspect --source UUID` to stage private local/base/remote review
files. Inspection first verifies the saved OAuth account/client and matching
index, then checks the observed remote source identity before staging evidence.
The command reports artifact paths and hashes, not source text. For native
Docs, `remote.txt` is semantic evidence, **not** an original-format replacement.
Do not paste it over annotated Markdown or treat it as an EPUB reconstruction.

Both commands require `--index`, and accept `--state-home` and `--config-home`.
Content-conflict resolution is described below. For a missing baseline with intact source/remote identities, preview
`state rebaseline --source UUID`, then apply its reviewed digest. This requires
local and remote content, formatting and layout to agree; it cannot reconstruct
a lost identity map. Restore a complete backup for that case. Never simulate
repair by deleting state or editing baseline hashes.
Back up the complete state directory: tokens alone cannot restore merge baselines.

## Explicit content-conflict resolution

`conflict resolve --source UUID --choice keep-local` previews a resolution plan.
`--choice accept-remote` previews the opposite direction. Apply only after review
with `--apply --plan-digest DIGEST`. Resolution handles two-sided content conflicts;
it cannot bypass missing state, moved objects, annotations or conversion findings.
Native accept-remote can still block where source-preserving mapping is ambiguous.
No resolution rewrites a baseline merely to suppress a conflict.

For a bound file or folder moved/renamed remotely, `--choice restore-layout` previews
restoring its local-authoritative name and parent without replacing content.
It uses a journaled conditional metadata write and read-back verification. Source relocation uses the separate rebind workflow below.

## Source relocation

After moving a local source, update selection if necessary, then use
`source rebind --source UUID --root ROOT --path NEW_RELATIVE_PATH`. The new path must be selected and its parent
folder already bound. Use a reviewed `sync --folders-only` plan to create missing
parents without creating source Docs. Review the plan, then repeat with `--apply --plan-digest`.
Both preview and apply require the old path to be absent. An existing file,
directory or dangling symlink blocks rebinding; a copy is not automatically a
relocation. If the old path reappears after review, preview again after resolving
which file owns the identity. Never delete either copy just to bypass the guard.

This updates local binding records recoverably while preserving remote identity
and old content baselines. It does not move local files or change Google content.

The binding then requires a separately reviewed `restore-layout` operation to
apply the new remote name/parent. A preview-allocated new identity may be replaced only when it has no binding,
pending write or owned remote object; the command verifies scoped Google inventory.
Already-created duplicate representations still require explicit review; do not
edit state manually to merge them.

Rebind account validation belongs to the Python service, not only the CLI. When
using a remote adapter to check placeholder identities, it must match this index's
saved Google account/client and index namespace. `wrong_account` or
`unverified_account` requires the original authorized profile; do not overwrite
`account.json` or switch profiles to bypass the check.

## Storage-mode replacement

`invalid_inventory` means the scoped Drive listing cannot establish which objects
exist. Restore API access and run a fresh preview; never interpret the error as
an empty folder, reset identities or repeat creation. Malformed ownership flags,
duplicate IDs across pages and wrong operation markers also block verification.

`invalid_metadata` or `invalid_document` means a bound-object read did not return
the requested identity, valid JSON, or a usable typed document revision. Retry
the read after checking API access. Do not treat the response as another object's
identity, edit state to match it, or recreate the bound object.

Read-back resolves the operation's remote identity before checking its layout.
`creation_not_found` means no operation-marked candidate was observed;
`ambiguous_identity` means multiple candidates prevent verification.
`remote_layout_mismatch` instead names the already-created object whose title or
parent differs from the staged plan. Never retry creation to repair a title: keep
the journal and inspect the identified object. A title/parent repair uses the reviewed command below. Content mismatch uses the separate reviewed content command below; deleting the journal is unsafe.

After editing a bound source's storage policy, preview
`conflict resolve --source UUID --choice replace-storage`. Apply the reviewed
digest to create a new representation, never to convert-overwrite or delete the
old object. The old binding/baselines are retained in private state and the old
remote object remains untouched. Subsequent sync manages the new binding only.
Conversion checks still apply; a pending migration needs recovery, not another
create. Both migration directions have disposable live evidence for the supported
fixtures; this does not bypass conversion findings on other sources.

## Repairing a missing content baseline

For an existing complete source/remote identity, preview `state rebaseline --source
UUID`. The tool requires agreement of source and remote content/formatting, the
expected remote layout, and no uncertain write. Account identity is checked by
the service; source identity/bytes/permissions, index digest and remote metadata
are rechecked after comparison. A race requires a fresh preview, not baseline
repair from mixed observations. Apply the displayed digest with
`--apply --plan-digest DIGEST`. This repairs local baseline records and archives
the prior binding; it never overwrites Google or source content to manufacture
agreement. Divergent versions require conflict review, not baseline suppression.

This does not reconstruct a completely lost source identity map. Restore that
map from backup; never adopt remote copies by matching titles alone.

## Missing source versus relocation

A missing old source plus a new selected path blocks new copy creation until the
ambiguity is resolved. Use `source rebind` for a move. If the old source was
intentionally removed and the new file is unrelated, preview
`state acknowledge-missing --source UUID`, then apply its digest. This records
acknowledged absence only: the remote object, identity and baselines remain.
The original configured root must still be available. Restore an unavailable
drive/folder before using this command; it is not a way to dismiss an offline
root. Index edits or root replacement invalidate a previously reviewed digest.
It cannot hide pending writes or acknowledge a path that still exists (including
a dangling symlink). Permission/I/O errors while checking the old path produce
`source_presence_unreadable`, not an assertion that the file is missing.

## Cancellation

During sync/initialization/resolution apply, Ctrl-C requests cancellation instead
of interrupting a write mid-reconciliation. The current bounded request completes
or times out, its outcome is checked, then no further mutation starts. A cancelled
CLI run exits with code 130. A forced process kill cannot run cleanup; use
`state recover` for any persisted pending operation after restarting.

## Reading results

Per-file results include `source_id`, `source`, `destination`, `operation`,
`status`, `reason`, `detail`, `action` and the journal `operation_id` when present.
The destination is a remote ID when known, or a parent/name description for an
unconfirmed create. Missing context remains empty rather than guessing an ID.
Safe validation details survive recovery; raw transport exceptions are redacted.
A `pending` result is not permission to repeat a write: follow its repair action.

Remote alignment, spacing, text direction and nondefault list numbering are not
currently expressible in the supported return model. A formatting conflict must
not be bypassed by accepting only the exported plain text. Review those edits in
Google and retain both versions; supported default-style normalization still
requires live validation.

## Closing an unchanged native update

`state close-unchanged --operation JOURNAL_UUID` previews closure of an unresolved
native-Doc content push. Apply with `--apply --plan-digest DIGEST` only after
review. The service rereads the full remote snapshot, including revision, and
requires exact equality with the saved pre-write snapshot. Changed revisions,
partial writes, missing preparation, original-file updates, layout effects and
creates are ineligible. Closure retains the intent and a separate audit record;
it changes neither content nor baselines. Preview normal sync afterward before
authorizing a new revision-guarded attempt. Never manually mark journals closed.

For native `accept-remote`, the candidate is based on the last verified source,
not the conflicting local prose. Current local bytes are retained as a backup.
The service checks that local differences are reversible readable edits; changed
comments, frontmatter or other protected structure block automatic resolution.
Reconcile such source-only changes manually instead of treating accept-remote as
permission to discard them.

## Resuming a partial large native push

Large native pushes use ordered, individually journaled batches. Each acknowledged
batch must be read back at the exact revision returned by Google before the next
batch can start. The file's baseline advances only after the complete intended
content and formatting verify. Cancellation stops at a checkpoint, not by
interrupting an in-flight write.

Preview `state resume-batches --operation JOURNAL_UUID`, then authorize the exact
plan with `--apply --plan-digest DIGEST`. Keep the original source and index
unchanged while resuming. Changed source identity, configuration, remote snapshot,
annotations or a stale digest block the command. Recovery of a saved receipt is
read-only; the remaining writes require the resume authorization.

A lost write response without a durable updated revision cannot be replayed
automatically. If a fresh observation exactly matches the pending batch's
pre-write snapshot (including revision, content, structure and annotations),
`resume-batches` offers a plan with `close_unchanged_attempt: true`. Review and
authorize that digest to archive the uncertain attempt, clear only its inflight
marker, and continue with revision-guarded batches. The archived sequence and
authorization remain in the operation's `.batches/unchanged-*.json` file; no
receipt is invented and no baseline advances at closure. A remote change before
closure or the next write blocks continuation. This works at intermediate
checkpoints as well as the first batch; it does not declare the whole push undone.

If the entire desired document already exists, normal `state recover` can still
verify the parent operation. Otherwise retain the journal and all source versions
for review; do not delete state or call resume repeatedly to bypass it.

### Replace a changed partial document

For a staged multi-batch native push whose partial content differs from both the
previous checkpoint and the intended final document, preview:

```sh
drive-sync --index /external/index.yaml state repair-batches --operation JOURNAL_UUID
```

This explicitly replaces the observed text/inline formatting with the original
staged source rendering. It is **not a merge**: review `before_text`, `desired_text`
and the full `observation`, especially any remote edits made since interruption.
Apply only with the returned `--plan-digest DIGEST` and `--apply`. Changed source,
configuration, observed remote version or repair history invalidates approval.
Annotations, unsupported formatting and mismatched identity/layout block repair.

Before writing, the service preserves the full observed Docs/Drive response and
authorized plan in `journal/OPERATION.batch-repairs/ATTEMPT.json`. Each attempt has
its own revision-guarded `.batches/sequence.json`; original checkpoints remain
untouched. Cancellation stops between batches. Only complete read-back of the
original intended content advances the original operation's baseline.

If repair itself is interrupted, use ordinary `state recover` first. If content
remains partial, preview `repair-batches` again and review the **new** replacement
plan. The previous digest cannot authorize another attempt because persisted
repair history has changed. This command never retries a sequence automatically,
creates a replacement Doc or removes earlier repair evidence.

## Local source permissions

Returned candidates retain the source's ordinary Unix permission bits; backups
and state files remain private (`0600`). A permission change between the source
snapshot and replacement blocks the commit. Special bits (setuid/setgid/sticky)
produce `unsupported_source_permissions`: inspect why they are present before
changing them; the agent must not automatically clear them to force a sync.
Source snapshots now capture UID/GID and readable extended attributes. Return
commits restore and verify them on the temporary candidate before replacing the
source path. Linux POSIX ACL preservation is tested; other platform-specific
metadata mechanisms (such as file flags or non-xattr ACLs) are not covered.
Attribute-reading or ownership/restoration failures block replacement rather than
silently discarding metadata. A filesystem reporting no xattr support is accepted;
a runtime without inspection support reports `source_metadata_unsupported`.
Private `.metadata-*.json` backup records retain ownership, mode, original mtime
and hex-encoded attributes alongside the content backup. Do not publish these
records or blindly apply their ownership/ACL values to unrelated files. Pause external editors during apply as described above.

## Ambiguous or invalid state JSON

State readers reject duplicate object keys, non-finite numbers (including numeric
overflow) and nesting deeper than 128 levels. `corrupt_state` identifies the record
path without echoing its contents. Restore the exact record from a trusted backup
or review it alongside the retained journal; do not pick a duplicate value, delete
the record, or regenerate identities to force recovery. Invalid serialization
leaves an existing record unchanged and reports `invalid_state`.

Conditional original-file and layout writes require one strong quoted Google
ETag. Wildcards, weak validators and lists of validators are rejected. If
`invalid_etag` occurs, acquire a fresh metadata guard; never replace it with `*`
or edit a journal to bypass the revision check.

`invalid_annotations` means a comments response could not establish safe annotation
state. It includes the destination and `read-comments` operation. Retry a fresh
preview later; do not discard comments, edit stored observations or interpret this
error as an empty comment list. Persistent failures require inspecting the API
response privately, not allowing an overwrite.

## Repairing an already-created Doc's title or parent

```sh
drive-sync --index /external/index.yaml state repair-created-layout --operation JOURNAL_UUID
drive-sync --index /external/index.yaml state repair-created-layout --operation JOURNAL_UUID --apply --plan-digest DIGEST
```

Use this only for a pending native file creation whose unique operation-marked
Doc already has the exact intended content and styles. The typed service checks
account, source, index, identity, annotations, current metadata and a fresh ETag.
It changes only the title/parent and records the attempt before sending. Normal
read-back reconciliation, not a successful write response, advances the original
creation baseline. No new object is created and no document text is replaced.

`repair_already_attempted` forbids another write, including after a lost response.
Run normal `state recover`; keep the nested `.repair/layout.json` attempt record.
If the write did not apply, retain the pending operation for manual review rather
than removing the attempt marker. `repair_content_mismatch` requires the separate content command below; do not
change the source or journal to bypass it.


## Repairing content in an already-created pending Doc

```sh
drive-sync --index /external/index.yaml state repair-created-content --operation JOURNAL_UUID
drive-sync --index /external/index.yaml state repair-created-content --operation JOURNAL_UUID --apply --plan-digest DIGEST
```

Review the current text and exact proposed replacement batch in the plan. This
explicitly replaces the pending Doc's supported content/styles with the payload
staged by the original create. Unexpected edits are not merged automatically.
Unsupported formatting, annotations, changed sources and stale plans block apply.
The full observed Doc and batch are preserved in `.repair/content.json` before
one required-revision write. It never creates a new Doc or alters prior retained
representations. Read-back, not the response, determines verification.

Do not delete an attempt record to retry an uncertain write. Use normal recovery;
if content verifies but the imported title differs, preview the separate layout
repair. A replacement exceeding the bounded single-batch limit remains blocked
pending multi-batch repair support; do not split requests or edit journals manually.


HTML and XHTML return patches can edit words inside uniquely identified
whitespace-collapsed text runs. Original indentation, tabs and CR/LF sequences
remain unchanged. Edits crossing that collapsed whitespace or repeated ambiguous
contexts still conflict rather than reformatting the source. Forward rendering
must reproduce the requested text before any candidate can replace the source.
