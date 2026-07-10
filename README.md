# d2

## gh-bot

`bin/gh-bot` posts issue/PR comments as the `stapps-d2` GitHub App bot instead
of your own `gh auth login` identity. Useful so automated comments (Claude
Code, Codex, etc.) are visibly distinguishable from comments you post
yourself.

The App's installation ID is looked up per-repo at runtime (via the App's
own JWT), so the same App ID/private key work across any account or org the
App is installed on — e.g. both `github.com/stilliard/*` and
`github.com/wildfireinternet/*` — no per-org config needed.

### Setup

1. Install the `stapps-d2` GitHub App on whichever accounts/orgs you want it
   to comment in (`stilliard`, `wildfireinternet`, etc.) and on the specific
   repos within each.
2. Generate a private key for the App (GitHub App settings → "Generate a
   private key") and save the `.pem` file somewhere local (not in the repo).
3. Copy `.env.example` to `.env` and fill in:
   - `GH_BOT_APP_ID` — the App's ID
   - `GH_BOT_PRIVATE_KEY_PATH` — path to the `.pem` file

### Making it available on your PATH

`gh-bot` resolves its own real location (following symlinks) to find `.env`
next to it, so it's safe to symlink onto your `PATH` instead of always
calling it via `bin/gh-bot`:

```sh
ln -s "$(pwd)/bin/gh-bot" ~/.local/bin/gh-bot   # user-level, no sudo
# or
sudo ln -s "$(pwd)/bin/gh-bot" /usr/local/bin/gh-bot   # system-wide
```

(Avoid `/usr/bin` — that's reserved for distro-managed packages; use
`/usr/local/bin` for locally-installed system-wide tools.) Once linked, just
run `gh-bot comment ...` from anywhere.

### Usage

```sh
# General issue/PR comment
gh-bot comment --repo owner/repo --number 123 "Looks good to me."

# Pipe a longer body in via stdin
gh-bot comment --repo owner/repo --number 123 <<'EOF'
Multi-line
comment body.
EOF

# Reply to an existing PR review (line) comment thread
gh-bot reply --repo owner/repo --pr 123 --comment-id 987654321 \
  "Fixed in a1b2c3d, thanks!"
```

Run `gh-bot --help`, `gh-bot comment --help`, or `gh-bot reply --help` for
full option details.

Each invocation mints a fresh short-lived installation token (no caching),
so no bot credentials are ever persisted outside your `.env`/`.pem`.
