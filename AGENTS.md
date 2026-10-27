# writing-sync

Mars source package. The skill is `skills/writing-sync`: selected local files to
a remote reading copy and back. Google Drive/Docs is implemented. Other
destinations (Royal Road and similar) belong here later as **separate skills**.

Operational (CLI + workflow resources), not craft. `creative-writing-skills`
depends on this package with no include filter (`[dependencies.writing-sync]`),
so its plugin ships the `writing-sync` skill. Whether creative-writing-skills
should ship it is not decided, and the choice lives in that repo's `mars.toml`,
not here.

```bash
meridian mars check
```
