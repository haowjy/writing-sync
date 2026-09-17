# Google OAuth setup

After [installing the independent CLI](installation.md), run from any directory:

```sh
drive-sync auth login --profile personal --client /external/client.json
drive-sync auth pick-folder --profile personal --client /external/client.json
drive-sync auth status --profile personal
drive-sync auth logout --profile personal
```

Use `/external/drive-sync-venv/bin/drive-sync` if that environment's executable
directory is not on `PATH`. Do not run login by importing code from the consumer
repository or writing credentials into the installed skill.

Login prints an `open_browser` JSON event containing the consent URL, then waits
for the callback. Open that URL in the user's browser. The final result confirms
whether authorization and account verification succeeded. Ctrl-C closes the
listener. No content is uploaded by login.

Tokens are stored in `$XDG_CONFIG_HOME/drive-sync/PROFILE/token.json`, falling
back to `~/.config/drive-sync/PROFILE/token.json`. `--config-home` overrides the
Drive Sync configuration directory itself. Status reports local credential
presence, not a fresh online validity check. Logout removes local credentials;
it does not revoke access at Google or delete client JSON supplied by the user.

1. Have the user create or select their Google Cloud project.
2. Enable Google Drive API and Google Docs API.
3. Configure the OAuth consent screen. For an external app in Testing, add the
   user's Google account as a test user.
4. Create an OAuth client of type **Desktop app**, not Web application. The JSON
   must contain one `installed` section, with no `web` section or duplicate keys.
   Do not combine downloaded client files or hand-edit their OAuth endpoints.
5. Download its JSON outside the skill/repository, for example
   `~/.config/drive-sync/personal/client.json`. Use directory permissions 0700
   and file permissions 0600. Ask for its path, never its contents.

The tool uses only `https://www.googleapis.com/auth/drive.file` for app-created
resources and folders the user opens with `auth pick-folder`. Do not substitute
broad Drive/Documents scopes, service accounts, rclone credentials, or a bare
API key. Existing research-probe tokens are not implicitly production profiles.
Pasting a folder URL does not grant access. Enable the Google Picker API in the
same Cloud project, then follow [destination parent](destination-parent.md).

Browser login uses a local loopback callback. If the CLI runs over SSH, the
browser's localhost is a different machine. On the user's machine, before
starting login or pick-folder:

```sh
ssh -o ExitOnForwardFailure=yes -L 127.0.0.1:PORT:127.0.0.1:PORT USER@HOST
```

Use the same `PORT` as `--port` on the CLI. Testing-mode refresh tokens can
expire; report reauthorization needs rather than repeatedly retrying writes.

Official setup: [Google's Python Docs quickstart](https://developers.google.com/workspace/docs/api/quickstart/python).
OAuth scope: [Drive per-file authorization](https://developers.google.com/workspace/drive/api/guides/api-specific-auth).
