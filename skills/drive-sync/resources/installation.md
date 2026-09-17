# Installation

Requires Python 3.11+ on a POSIX system (advisory locks and directory fsync).
The skill is independent of its consumer repository. Copy the complete skill
folder, including `uv.lock`, resources and tests.

Install the locked CLI into a separate environment, not inside the skill:

```sh
UV_PROJECT_ENVIRONMENT=/external/drive-sync-venv uv sync --locked --no-editable --project /path/to/drive-sync
/external/drive-sync-venv/bin/drive-sync --help
```

After installation, runtime uses the environment's installed package, not imports
from the skill source directory. Keep indexes, source roots, state and OAuth
profiles external. Installing the package does not authorize any Google writes.

Run bundled tests with the environment's Python:

```sh
/external/drive-sync-venv/bin/python -m unittest discover -s /path/to/drive-sync/tests
```

OAuth tests bind only localhost and use synthetic credentials. Other tests run
without network. Passing installation tests does not establish fidelity for your
own files; check conversion support and review a small initial selection.

## Repeatable reinstall/state-isolation check

The bundled `tests/portability_check.py` uses a fake Google client, not OAuth.
Run from an unrelated working directory with the noneditable environment's Python
and no source `PYTHONPATH`. Supply a **new, nonexistent** external fixture directory:

```sh
/external/drive-sync-venv/bin/python /path/to/drive-sync/tests/portability_check.py seed /external/reinstall-fixture
# Reinstall the package without changing its external fixture/state directory.
UV_PROJECT_ENVIRONMENT=/external/drive-sync-venv uv sync --locked --no-editable --reinstall-package independent-drive-sync --project /path/to/drive-sync
/external/drive-sync-venv/bin/python /path/to/drive-sync/tests/portability_check.py verify /external/reinstall-fixture
```

The check creates two independent indexes and sources sharing a state home,
initializes and populates synthetic Docs, then verifies retained bindings and
no-op reruns after reinstall. It compares durable state/source hashes and checks
that no fake remote mutation occurs during verification. Fixture data is retained
for inspection. This checks same-version reinstall isolation, not future schema
migration compatibility or live Google behavior. Read-only runtime testing also
requires making the installed package read-only before each stage; reinstallers
may need its write permissions restored between stages.

Keep the fixture at its original absolute path for verification. Copying it to a
different directory changes filesystem identities and source-ownership claims;
that is a relocation experiment, not an unchanged-state reinstall check.

For minimum-version verification, install into a second external environment
using `uv sync --locked --no-editable --python 3.11 --project /path/to/drive-sync`
with a distinct `UV_PROJECT_ENVIRONMENT`. Run the same suite without a source
`PYTHONPATH`. The second interpreter can also run `portability_check.py verify`
against the first interpreter's fixture to check current-schema compatibility;
this is not a substitute for a future schema-migration test.
