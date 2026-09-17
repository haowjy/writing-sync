# Disposable integration testing

This is a maintainer test, not ordinary user sync. Obtain explicit permission to
create and edit disposable Google resources before any Google stage. Never use
real source selections or the human-annotated research Doc as a test fixture.

The bundled `tests/live_integration.py` uses the production application services
with a synthetic index. It reads a separately authorized research token; it does
not migrate that token to the production profile or print its contents. Normal
users should follow [Google setup](google-auth-setup.md), not this token path.

With the skill installed and its dependencies available:

```sh
python /path/to/drive-sync/tests/live_integration.py seed --run /external/new-fixture
python /path/to/drive-sync/tests/live_integration.py initialize --run /external/new-fixture --research-token /private/probe-token.json --apply
python /path/to/drive-sync/tests/live_integration.py sync --run /external/new-fixture --research-token /private/probe-token.json --apply
python /path/to/drive-sync/tests/live_integration.py edit-remote --run /external/new-fixture --research-token /private/probe-token.json --apply
python /path/to/drive-sync/tests/live_integration.py sync --run /external/new-fixture --research-token /private/probe-token.json --apply
```

`seed` refuses an existing directory. Later stages reuse that directory and index,
never regenerate identities to evade pending operations. `edit-remote` records
its intended text edit before sending and will not blindly retry an uncertain
write. It changes only the synthetic index's owned Docs, without changing local
sync baselines. Save each stage's result before the next same-named stage.

Inspect per-file statuses; a successful process exit alone does not prove sync.
If any operation is pending, use the runner's `inspect` stage for scoped reads
and `recover` for production read-only reconciliation. Do not repeat initialize
as an attempted repair. Once returns verify, run `sync` again and require all
operations to be no-ops. Compare original-format bytes, retained HTML comments,
text line endings and EPUB member payloads locally.

The runner does not delete test resources. Its plain fixture proves neither
arbitrary rich-format fidelity nor full EPUB conformance. Keep those release gates
separate, and retain the fixture/state directory for recovery and inspection.

For representative supported formatting, use a **different new directory** and
the sequence `seed-styled`, `initialize`, `sync`, `edit-styled`, `sync`, `sync`,
`verify-styled`, with the same arguments as above. This fixture contains HTML
and EPUB/XHTML with headings, bold, italics, a list and an external link.
`edit-styled` changes one word in each disposable Doc using a required revision.
`verify-styled` checks exact HTML bytes apart from that word, unchanged EPUB
member payloads except the edited chapter and package timestamp, matching
rendered styles and an all-no-op final plan. It writes `styled-verification.json`.
Run EPUBCheck separately on both source and returned EPUB; style equality does
not establish package conformance. Unsupported CSS/assets and arbitrary rich
edits remain outside this fixture's evidence.

After the simple four-format round trip is entirely no-op, `rebind-file` renames
the disposable `notes/a.md` to `notes/a-renamed.md`. It calls the reviewed rebind
service, applies a reviewed `restore-layout` plan, and checks the same Doc ID,
unchanged content/source bytes, unchanged inventory IDs and a final all-no-op
preview. Its attempt marker is written before the local rename; reruns are refused
even after success. If interrupted, inspect the saved plans and use normal state
recovery rather than deleting the marker or restarting the probe. This stage does
not rename the file back and must not be used on real writing.

`replace-original` and then `replace-doc` exercise explicit storage-mode changes
on that renamed synthetic source. Each stage records its attempt before editing
the fixture index, requests a reviewed `replace-storage` operation, and verifies
retention of the old remote object and complete binding. A successful stage ends
with an all-no-op preview. Each direction is one-shot: an uncertain or unverified
create requires normal recovery, never deleting the marker or repeating the stage.
The reverse live fixture required a reviewed content repair after an earlier
import newline mismatch. A fixed serializer alone never authorizes replay or
automatically repairs an existing object.

For original-file mode, `seed-original` adds the disposable `e.bin` selection and
an explicit original-storage override without Google access. Sync its initial
bytes, change only that fixture to the runner's expected local-edit bytes, sync
again, then use `edit-original` and sync to test the return. The edit stage records
its intent before the conditional request and refuses uncertain replay. This is
a separate path from Google Docs conversion and must have separate evidence.

`make-conflict` diverges the disposable `b.txt` locally and remotely. Ordinary sync
must report `both_changed` without overwriting either version; `resolve-conflict`
then exercises an explicitly reviewed accept-remote plan. These stage names have
fixed synthetic fixture expectations; do not point them at a user's project.

`add-comment` creates one unanchored API comment on disposable `c.html` and changes
its local HTML. Sync must report `annotations_protected` while retaining the
comment and both content versions. The comment is left in place. This does not
replace native editor suggestion/anchored-comment tests or authorize touching
any earlier human-annotated research Doc.

For the large Unicode fixture, `cancel-large` changes the disposable text source
and requests cancellation immediately after the first batch response. Verify one
checkpoint and the previous file baseline before continuing. `resume-large
--operation JOURNAL_UUID` previews and applies the exact remaining sequence through
the production resume service. Compare the saved first checkpoint before/after:
it must not be replaced or replayed. These stages require the established large
fixture and retain partial state; they are not general-purpose user commands.

After a fresh simple four-format round trip ends in all no-ops, the one-shot
`lost-batch-repair` stage expands only synthetic `b.txt` to a multi-batch payload.
It deliberately discards the second successful Google batch response before the
executor can persist its receipt. The original push must remain pending. The
stage then previews and explicitly applies `repair-batches`, checks read-back
verification and requires a final all-no-op preview. It preserves the original
sequence, partial observation and repair evidence, and refuses to rerun once its
attempt marker exists. It leaves the expanded synthetic file in place. This
fault injection exercises an applied-but-unacknowledged batch, not a transport
outage simulator; no old probe Doc or real writing belongs in its index.

For `folder-layout`, seed a **new** four-format fixture. Before initialization,
create `notes/left` and `notes/right`; move only its synthetic `a.md` and `b.txt`
into `left`, and `c.html` and `d.epub` into `right`. Initialize and populate all
seven objects, then run the `folder-layout` stage. It moves/renames `left` under
`right`, requires a folder conflict, and restores it through reviewed
`restore-layout`. It separately renames/restores the managed root. All child Doc
snapshots and inventory IDs must remain unchanged, and the final seven operations
must be no-ops. The attempt marker prevents replay after any interruption; retain
the fixture and repair plans rather than rerunning or deleting test resources.

`stale-original` uses the saved pre-edit ETag from the disposable binary test for
one intentionally stale update. It requires HTTP 412 and unchanged bytes **and**
metadata afterward. The result is persisted and the attempt cannot be repeated by
the runner. If any check fails, stop original-file writes and investigate; a
failure response alone is not evidence that content remained unchanged. This
probe must never target a native Google Doc or an irreplaceable original file.

`seed-nested` adds `notes/level1/level2/g.md`, linking back to `../../a.md`.
After reviewed initialization and sync, `verify-nested` performs read-only Google
checks of the parent chain, rendered text and link destination, then checks saved
link provenance and unchanged local bytes. It saves `nested-verification.json`.
The fixture must already contain the selected, verified `a.md` destination. This
checks one cross-file link, not first-time cyclic-link initialization.

## Independent EPUB conformance

The optional `tests/epub_conformance.py` generates a source EPUB, two original-format
return candidates and a deliberately invalid navigation fixture. It invokes the
[official EPUBCheck distribution](https://www.w3.org/publishing/epubcheck/docs/installation/)
with [warning-sensitive exit status and JSON reports](https://www.w3.org/publishing/epubcheck/docs/cli/).
Java and EPUBCheck are maintainer-only external tools, not sync dependencies.
Keep the complete distribution together; the JAR needs its bundled `lib/` files.

```sh
/external/venv/bin/python /path/to/drive-sync/tests/epub_conformance.py \
  --java /external/jre/bin/java \
  --jar /external/epubcheck/epubcheck.jar \
  --output /external/new-conformance-results
```

Use a nonexistent output directory. No credentials or Google requests are involved.
The gate requires clean reports for all three valid EPUBs and the expected missing
navigation-property error for the negative control, not merely a nonzero process
exit. It retains EPUBs, logs, JSON reports and a version/hash summary. Passing these
fixtures does not certify arbitrary EPUB assets, CSS, reading order or edit fidelity.

The EPUB runtime capability check also inspects package metadata and spine
properties, not only chapter XHTML. Fixed-layout constraints, non-default
presentation overrides, non-linear/right-to-left page progression, synchronized
narration and declared active/specialized resources produce explicit findings.
Known rendition vocabulary aliases are resolved before checking. Ordinary
reflowable content remains supported. A capability finding blocks unattended
conversion/return; it is not permission to strip metadata or switch storage mode
without reviewing the index change. These checks are separate from
[EPUB structural conformance](https://www.w3.org/TR/epub-33/).

For the existing nested fixture, `move-nested` performs one conditional remote
move/rename of `g.md` to `level1/g-moved.md`, verifies unchanged Doc content and
leaves sync baselines untouched. It persists intent first and refuses a second
attempt. Ordinary `sync` must report `remote_structure_changed`, not restore the
layout automatically. Save that result, then use `restore-nested` for the exact
reviewed production `restore-layout` plan. Require verified read-back before
running `verify-nested` again. Pending restoration requires normal recovery,
never replaying the move probe. These stages affect only the synthetic fixture.


`repair-created-layout --operation JOURNAL_UUID` previews and applies the production
metadata-only recovery service for an already-created disposable Doc. Use it only
when its exact content verified but its imported title/parent did not. Plans,
results and the service's durable attempt marker are retained. This stage does
not create another object and refuses a repeated repair attempt; use `recover`
after an uncertain response. Do not use it to bypass content mismatches.


`repair-created-content --operation JOURNAL_UUID` invokes the production reviewed
content recovery service on an already-created synthetic Doc. It saves the exact
plan, preserves the observed Doc in the service journal, and uses a required-revision
batch rather than repeating the create. After verification, run `sync` and require
all operations to be no-op. Retained storage representations must remain intact.

## First-time cyclic links

Use `seed-cyclic --run /external/new-cyclic-fixture` for a separate, nonexistent
fixture directory. It creates two synthetic Markdown files pointing to each
other. Run `initialize`, inspect all three verified results (folder plus two
empty identities), then run `sync` to populate both Docs. Do not run ordinary
content sync first or rerun initialization to evade a pending creation.

`verify-cyclic` then independently reads both links, checks exact source bytes
and stored provenance against the target IDs, and requires an all-no-op preview.
It performs no remote writes and saves `cyclic-verification.json`. Google stages
use the same explicit token/permission flags as the other examples. A missing or
wrong link fails verification; do not overwrite it merely to make the check pass.
