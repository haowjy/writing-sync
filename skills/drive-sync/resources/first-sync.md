# First sync

Use after the CLI is installed, a profile is authorized, and the index
selection is saved. If the mirrored root should live inside an existing Drive
folder, finish [destination parent](destination-parent.md) first.

Keep `--index`, `--state-home`, and `--config-home` the same for every command
in this sequence. `--state-home` is optional when the default state directory
is acceptable; `--config-home` is required when the profile is not under the
default config directory.

1. Run `conversion check`. It is offline. `supported: true` is a local
   rendering gate, not Google fidelity. Read
   [conversion limits](conversion-limits.md) before enabling converted storage.
2. Preview `sync --initialize`. Apply that digest with
   `sync --initialize --apply --plan-digest DIGEST`. This creates verified empty
   Docs and folders, not reading copies. Do not call them synchronized content.
   If someone edits an incomplete Doc before population, the tool blocks rather
   than overwriting it.
3. Preview ordinary `sync`. Apply that second digest separately. This populates
   content. Review every proposed operation. Keep source backups.
4. Inspect every file result. `pending` means follow [recovery](recovery.md).
   Do not repeat initialize or create.
5. Preview `sync` again. Success is all `noop`.

A changed plan fails authorization. Preview again; do not reuse a stale digest.
