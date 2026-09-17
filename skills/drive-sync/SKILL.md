---
name: drive-sync
type: reference
description: >
  Load when syncing selected local files to a remote reading copy, or bringing
  remote edits back into those files. Google Drive/Docs is the current
  destination. Not for publishing to Royal Road or similar.
---

# Drive Sync

Syncs selected local files to a remote reading copy and can bring remote edits
back into those files. Markdown, text, HTML, and EPUB can convert; other
selected files stay in their original format. A converted file can still be
unsafe to return after a heavy remote rewrite — run conversion check before the
first write, and start with a small selection.

Google Drive/Docs is the current destination. Other destinations are not
implemented. Publishing platforms are a different skill.

Run the installed CLI. Do not rewrite YAML or call the remote API by hand. Keep
indexes, sources, credentials, and state outside this skill. Never ask for token
or client JSON contents. Saving the index does not write remotely. Preview, then
apply that digest.

For Google: use only `https://www.googleapis.com/auth/drive.file`. Do not widen
the scope. Pasting a folder URL does not grant access.

A pending result is not permission to retry a create.

## Load the resource for the job

- Install the CLI: [installation](resources/installation.md)
- OAuth, Picker API, SSH loopback: [Google setup](resources/google-auth-setup.md)
- Index create/edit: [configuration](resources/configuration.md)
- Index fields: [schema](resources/schema.md)
- First push of a new selection: [first sync](resources/first-sync.md)
- Everyday preview/apply/scope: [sync](resources/sync.md)
- Place a mirrored root in an existing Drive folder: [destination parent](resources/destination-parent.md)
- Whether converted storage is safe to choose: [conversion limits](resources/conversion-limits.md)
- Pending writes, conflicts, missing sources, rebind: [recovery](resources/recovery.md)
- Maintainer disposable Google tests: [integration testing](resources/integration-testing.md)
