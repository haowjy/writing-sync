# Sync commands

`sync` performs scoped Google reads and emits a combined folder/file plan with
a digest. It persists local source identities and ownership claims but performs
no Google writes. `sync --apply --plan-digest DIGEST` authorizes that exact
reviewed plan. A changed plan fails authorization. Both accept `--state-home`;
`--config-home` selects the private authentication directory.

Review every proposed operation. A pending result is not permission to repeat a
create. Use [recovery](recovery.md) for pending writes and conflicts.

Supported four-format returns and native annotation detection/blocking have
live evidence. That does not establish arbitrary rich-format fidelity or
preservation of annotations through updates.

## Scope

`sync --source SOURCE_UUID` (repeat `--source`) limits writes to those items.
Obtain UUIDs from the full sync preview. The scoped preview includes each
requested item and its required ancestor folders in execution order. Review
that entire plan, then repeat the same `--source` list with
`--apply --plan-digest DIGEST`.

A folder UUID selects that folder and its ancestors, not all descendants.
Select individual file UUIDs for content sync. Unrelated siblings are not
applied.

The same option works with `--initialize` and `--folders-only`; selected UUIDs
must be present in that mode's plan. Unknown UUIDs and an empty service-level
scope fail rather than falling back to syncing everything. Omit `--source` for
the whole index. Scope limits operations, not reads: index-wide ownership,
source, and inventory checks still run. Referenced but uninitialized link
targets are not silently added to the write scope.

## Initialize then populate

For a new converted source set, use [first sync](first-sync.md).
`sync --initialize` creates verified empty Doc identities and folders.
Ordinary `sync` populates them. Apply those two plans as separate digests.

`sync --folders-only` creates missing parent folders without creating source
Docs. Use it when a later rebind needs parents bound first.

## Reading results

Per-file results include `source_id`, `source`, `destination`, `operation`,
`status`, `reason`, `detail`, `action`, and the journal `operation_id` when
present. The destination is a remote ID when known, or a parent/name
description for an unconfirmed create. A successful process exit is not a
substitute for checking each status. `verified` with later `noop` is the
steady state. `pending` means follow the listed repair action.
